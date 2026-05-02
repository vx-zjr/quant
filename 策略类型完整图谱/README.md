# 策略类型完整图谱

## 目录
- [Alpha策略](./Alpha策略.md)
- [CTA与趋势跟踪](./CTA与趋势跟踪.md)
- [统计套利](./统计套利.md)
- [期权量化策略](./期权量化策略.md)
- [高频交易策略](./高频交易策略.md)
- [事件驱动策略](./事件驱动策略.md)

---

## 一、Alpha策略

### 多因子模型
| 因子类别 | 代表因子 | 描述 |
|----------|----------|------|
| 价值因子 | PE、PB、PCF | 基本面估值 |
| 动量因子 | 1M/3M/12M收益 | 价格动量 |
| 质量因子 | ROE、ROA、毛利率 | 盈利质量 |
| 规模因子 | 市值对数 | 市值因子 |
| 波动率因子 | 残差波动率 | 风险因子 |
| 成长因子 | 营收增长、利润增长 | 成长性 |

### 因子研究框架
```python
# 因子研究示例
import pandas as pd
import numpy as np
from scipy import stats

def factor_analysis(factor_data, returns, periods=20):
    """
    因子分析框架
    
    参数:
    factor_data: 因子值矩阵
    returns: 收益率矩阵
    periods: 持有期
    """
    results = []
    
    for factor_name in factor_data.columns:
        factor = factor_data[factor_name]
        
        # 因子IC值 (Information Coefficient)
        ic = stats.spearmanr(factor, returns)[0]
        
        # 因子收益分组
        quintiles = pd.qcut(factor, 5, labels=['Q1', 'Q2', 'Q3', 'Q4', 'Q5'])
        portfolio_returns = returns.groupby(quintiles).mean()
        
        # 多空组合收益
        long_short = portfolio_returns['Q5'] - portfolio_returns['Q1']
        
        # 换手率
        turnover = calculate_turnover(quintiles)
        
        results.append({
            'factor': factor_name,
            'IC': ic,
            'IC_IR': ic / factor.std(),  # IC的IR
            'Q5_return': portfolio_returns['Q5'].mean(),
            'Q1_return': portfolio_returns['Q1'].mean(),
            'long_short': long_short.mean(),
            'turnover': turnover.mean()
        })
    
    return pd.DataFrame(results)
```

### 机器学习因子
| 方法 | 应用 |
|------|------|
| XGBoost | 非线性因子组合 |
| 神经网络 | 特征自动提取 |
| 随机森林 | 特征重要性 |
| 遗传算法 | 因子搜索 |

---

## 二、CTA与趋势跟踪

### 趋势指标
| 指标 | 描述 |
|------|------|
| 移动平均 | MA、EMA |
| 趋势线 | 通道突破 |
| ADX | 趋势强度 |
| 唐奇安通道 | 突破系统 |

### 经典趋势系统
```python
# 双均线趋势策略
def dual_moving_average_strategy(prices, fast_ma=20, slow_ma=60):
    """
    双均线趋势策略
    
    逻辑:
    - 快速均线从上穿慢速均线 → 做多
    - 快速均线从下穿慢速均线 → 做空
    """
    signals = pd.DataFrame(index=prices.index)
    signals['fast_ma'] = prices['close'].rolling(fast_ma).mean()
    signals['slow_ma'] = prices['close'].rolling(slow_ma).mean()
    
    # 交易信号
    signals['signal'] = 0
    signals.loc[signals['fast_ma'] > signals['slow_ma'], 'signal'] = 1
    signals.loc[signals['fast_ma'] < signals['slow_ma'], 'signal'] = -1
    
    # 信号变化点
    signals['positions'] = signals['signal'].diff()
    
    return signals

# Turtle Trading策略
def turtle_trading(prices, entry_period=20, exit_period=10, 
                   atr_period=20, atr_multiplier=2):
    """
    海龟交易法则
    
    参数:
    entry_period: 入场周期
    exit_period: 出场周期
    atr_period: ATR周期
    atr_multiplier: ATR倍数
    """
    signals = pd.DataFrame(index=prices.index)
    
    # ATR计算
    high_low = prices['high'] - prices['low']
    high_close = np.abs(prices['high'] - prices['close'].shift())
    low_close = np.abs(prices['low'] - prices['close'].shift())
    tr = pd.concat([high_low, high_close, low_close], axis=1).max(axis=1)
    atr = tr.rolling(atr_period).mean()
    
    # 入场信号
    rolling_high = prices['high'].rolling(entry_period).max()
    rolling_low = prices['low'].rolling(entry_period).min()
    
    signals['entry'] = prices['close'] > rolling_high
    signals['exit'] = prices['close'] < rolling_low
    
    # 止损
    signals['stop'] = atr * atr_multiplier
    
    return signals
```

### 策略评估
| 指标 | 计算 | 理想值 |
|------|------|--------|
| 年化收益 | Mean * 252 | > 10% |
| 夏普比率 | Mean/Std * sqrt(252) | > 1.5 |
| 最大回撤 | Equity peak - trough | < 20% |
| 胜率 | Win trades / Total | > 40% |
| 盈亏比 | Avg win / Avg loss | > 1.5 |

---

## 三、统计套利

### 配对交易
```python
# 协整配对交易
def cointegration_pairs_trading(stock1, stock2, window=60):
    """
    协整配对交易策略
    
    步骤:
    1. 检测两只股票的协整关系
    2. 计算价差的z-score
    3. 当z-score偏离阈值时入场
    """
    # 计算价差
    spread = stock1 - hedge_ratio * stock2
    
    # 价差的均值和标准差
    mean = spread.rolling(window).mean()
    std = spread.rolling(window).std()
    
    # z-score
    z_score = (spread - mean) / std
    
    # 交易信号
    signals = pd.DataFrame(index=stock1.index)
    signals['z_score'] = z_score
    signals['signal'] = 0
    signals.loc[z_score > 2, 'signal'] = -1  # 价差高估，做空价差
    signals.loc[z_score < -2, 'signal'] = 1   # 价差低估，做多价差
    signals.loc[z_score.abs() < 0.5, 'signal'] = 0  # 回归时平仓
    
    return signals
```

### 均值回归
| 方法 | 描述 |
|------|------|
| 布林带策略 | 价格偏离布林带回归 |
| RSI策略 | 超买超卖回归 |
| Z-score策略 | 标准化偏离回归 |

---

## 四、期权量化策略

### 波动率交易
```python
# 波动率套利策略
def volatility_arbitrage(iv, rv, spot, strike, expiry, option_type='call'):
    """
    波动率套利策略
    
    比较隐含波动率与实际波动率
    - IV > RV: 卖出期权 (overpriced)
    - IV < RV: 买入期权 (underpriced)
    """
    from scipy.stats import norm
    
    # BS定价公式
    def black_scholes(S, K, T, r, sigma, option_type):
        d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
        d2 = d1 - sigma*np.sqrt(T)
        
        if option_type == 'call':
            return S*norm.cdf(d1) - K*np.exp(-r*T)*norm.cdf(d2)
        else:
            return K*np.exp(-r*T)*norm.cdf(-d2) - S*norm.cdf(-d1)
    
    # 计算期权价值
    T = expiry / 365
    option_price = black_scholes(spot, strike, T, 0.03, iv, option_type)
    
    # 计算vega
    d1 = (np.log(spot/strike) + (0.03 + 0.5*iv**2)*T) / (iv*np.sqrt(T))
    vega = spot * np.sqrt(T) * norm.pdf(d1) / 100
    
    # 波动率偏差
    vol_diff = iv - rv
    
    return {
        'option_price': option_price,
        'vega': vega,
        'vol_diff': vol_diff,
        'position': 'sell' if iv > rv else 'buy'
    }
```

### Greeks管理
| Greek | 描述 | 管理方法 |
|-------|------|----------|
| Delta | 价格敏感度 | 动态对冲 |
| Gamma | Delta变化率 | 高Gamma风险 |
| Theta | 时间衰减 | 时间价值 |
| Vega | 波动率敏感 | IV风险管理 |

---

## 五、高频交易策略

### 做市策略
```python
# 简单做市策略
class MarketMaker:
    def __init__(self, spread_pct=0.001, inventory_limit=1000):
        self.spread_pct = spread_pct
        self.inventory_limit = inventory_limit
        self.inventory = 0
        self.position = 0
        
    def quote(self, mid_price):
        """
        生成报价
        """
        # 基于库存调整价差
        if abs(self.inventory) > self.inventory_limit * 0.8:
            # 减少持仓方向报价
            spread = self.spread_pct * 2
        else:
            spread = self.spread_pct
        
        bid_price = mid_price * (1 - spread/2)
        ask_price = mid_price * (1 + spread/2)
        
        return bid_price, ask_price
    
    def update_inventory(self, trade_side, quantity):
        """
        更新库存
        """
        if trade_side == 'buy':
            self.inventory += quantity
        else:
            self.inventory -= quantity
```

### 套利策略
| 策略 | 描述 |
|------|------|
| 跨交易所套利 | CEX间价格差 |
| 三角套利 | 同交易所多币种 |
| 期现套利 | 合约与现货 |
| 统计套利 | 相关品种 |

---

## 六、事件驱动策略

### 财报事件
| 事件 | 信号 |
|------|------|
| 业绩预告 | 预增/预减 |
| 正式财报 | 超预期/低于预期 |
| 分红 | 分红除权 |
| 并购 | 重组公告 |

### 宏观事件
| 事件 | 交易逻辑 |
|------|----------|
| 美联储加息 | 利率敏感资产 |
| CPI数据 | 通胀预期 |
| GDP数据 | 经济周期 |
| 地缘政治 | 避险资产 |

### 自然语言处理事件
```python
# 新闻事件驱动
def news_event_strategy(news_df, stock_returns):
    """
    新闻事件驱动策略
    
    1. 提取新闻情绪
    2. 检测突发事件
    3. 建立短期仓位
    """
    from transformers import pipeline
    
    sentiment_analyzer = pipeline("sentiment-analysis", 
                                   model="ProsusAI/finbert")
    
    # 新闻情绪
    news_df['sentiment'] = news_df['text'].apply(
        lambda x: sentiment_analyzer(x)[0]
    )
    
    # 事件信号
    news_df['signal'] = 0
    news_df.loc[news_df['sentiment'] == 'positive', 'signal'] = 1
    news_df.loc[news_df['sentiment'] == 'negative', 'signal'] = -1
    
    # 短期收益
    event_returns = stock_returns.rolling(5).sum().shift(-5)
    
    return news_df[['date', 'signal', 'returns']].dropna()
```

---

*最后更新: 2026-04-26*
