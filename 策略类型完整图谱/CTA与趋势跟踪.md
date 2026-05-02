# CTA与趋势跟踪策略

## 一、CTA策略概述

### 什么是CTA
- **CTA定义**: Commodity Trading Advisor，商品交易顾问
- **核心逻辑**: 趋势跟踪，顺势而为
- **市场范围**: 期货、期权、股票、外汇、加密货币

### CTA策略分类
```
CTA策略
├── 趋势跟踪
│   ├── 日间趋势
│   ├── 日内趋势
│   └── 突破策略
├── 均值回归
│   ├── 收敛交易
│   └── 回归策略
├── 套利策略
│   ├── 跨品种套利
│   ├── 跨市场套利
│   └── 期现套利
└── 复合策略
    ├── 多周期策略
    └── 多策略组合
```

---

## 二、经典趋势跟踪系统

### 1. 移动平均系统

#### 双均线系统
```python
def dual_ma_system(prices, fast_period=20, slow_period=60):
    """
    双均线趋势跟踪系统
    
    规则:
    - 快速均线从上穿越慢速均线 -> 做多
    - 快速均线从下穿越慢速均线 -> 做空
    """
    signals = pd.DataFrame(index=prices.index)
    
    # 计算均线
    signals['fast_ma'] = prices['close'].rolling(fast_period).mean()
    signals['slow_ma'] = prices['slow_ma'].rolling(slow_period).mean()
    
    # 生成信号
    signals['raw_signal'] = 0
    signals.loc[signals['fast_ma'] > signals['slow_ma'], 'raw_signal'] = 1
    signals.loc[signals['fast_ma'] < signals['slow_ma'], 'raw_signal'] = -1
    
    # 避免频繁交易
    signals['signal'] = signals['raw_signal']
    
    # 交易点位
    signals['positions'] = signals['signal'].diff()
    
    return signals

# 三均线系统
def triple_ma_system(prices, short=10, medium=50, long=200):
    """
    三均线趋势系统
    
    多头信号: 短 > 中 > 长
    空头信号: 短 < 中 < 长
    """
    signals = pd.DataFrame(index=prices.index)
    
    signals['short_ma'] = prices['close'].rolling(short).mean()
    signals['medium_ma'] = prices['close'].rolling(medium).mean()
    signals['long_ma'] = prices['close'].rolling(long).mean()
    
    # 多头排列
    signals['bullish'] = (signals['short_ma'] > signals['medium_ma']) & \
                          (signals['medium_ma'] > signals['long_ma'])
    
    # 空头排列
    signals['bearish'] = (signals['short_ma'] < signals['medium_ma']) & \
                          (signals['medium_ma'] < signals['long_ma'])
    
    signals['signal'] = 0
    signals.loc[signals['bullish'], 'signal'] = 1
    signals.loc[signals['bearish'], 'signal'] = -1
    
    return signals
```

#### 指数移动平均系统
```python
def ema_crossover_system(prices, fast_period=12, slow_period=26):
    """
    EMA交叉系统
    """
    signals = pd.DataFrame(index=prices.index)
    
    # 计算EMA
    signals['fast_ema'] = prices['close'].ewm(span=fast_period, adjust=False).mean()
    signals['slow_ema'] = prices['close'].ewm(span=slow_period, adjust=False).mean()
    
    # MACD指标
    signals['macd'] = signals['fast_ema'] - signals['slow_ema']
    signals['signal_line'] = signals['macd'].ewm(span=9, adjust=False).mean()
    signals['histogram'] = signals['macd'] - signals['signal_line']
    
    # 交易信号
    signals['signal'] = 0
    signals.loc[signals['macd'] > signals['signal_line'], 'signal'] = 1
    signals.loc[signals['macd'] < signals['signal_line'], 'signal'] = -1
    
    return signals
```

### 2. 唐奇安通道系统

```python
def donchian_channel_system(high, low, close, entry_period=20, exit_period=10):
    """
    唐奇安通道突破系统（海龟交易法基础）
    
    入场规则:
    - 价格突破20日高点 -> 做多
    - 价格跌破20日低点 -> 做空
    
    出场规则:
    - 价格跌破10日低点 -> 平多
    - 价格突破10日高点 -> 平空
    """
    signals = pd.DataFrame(index=close.index)
    
    # 计算通道
    signals['upper_channel'] = high.rolling(entry_period).max()
    signals['lower_channel'] = low.rolling(entry_period).min()
    signals['exit_upper'] = high.rolling(exit_period).max()
    signals['exit_lower'] = low.rolling(exit_period).min()
    
    # 入场信号
    signals['entry_long'] = close > signals['upper_channel'].shift(1)
    signals['entry_short'] = close < signals['lower_channel'].shift(1)
    
    # 出场信号
    signals['exit_long'] = close < signals['exit_lower'].shift(1)
    signals['exit_short'] = close > signals['exit_upper'].shift(1)
    
    # 仓位状态
    position = 0
    positions = []
    
    for i in range(len(close)):
        if position == 0:
            if signals['entry_long'].iloc[i]:
                position = 1
            elif signals['entry_short'].iloc[i]:
                position = -1
        elif position == 1:
            if signals['exit_long'].iloc[i]:
                position = 0
        elif position == -1:
            if signals['exit_short'].iloc[i]:
                position = 0
        
        positions.append(position)
    
    signals['position'] = positions
    
    return signals

# 改进版唐奇安系统
def enhanced_donchian_system(high, low, close, atr_period=20):
    """
    增强版唐奇安系统 - 加入止损
    
    1. 突破20日高点入场
    2. 2ATR止损
    3. 10日低点出场
    """
    signals = pd.DataFrame(index=close.index)
    
    # ATR计算
    tr = np.maximum(high - low, 
                    np.maximum(abs(high - close.shift(1)),
                               abs(low - close.shift(1))))
    atr = tr.rolling(atr_period).mean()
    
    # 通道
    signals['entry_high'] = high.rolling(20).max()
    signals['exit_low'] = low.rolling(10).min()
    
    # 入场
    signals['entry_long'] = close > signals['entry_high'].shift(1)
    
    # 止损
    entry_price = None
    stop_loss = None
    positions = []
    
    for i in range(len(close)):
        if entry_long[i] and position == 0:
            entry_price = close.iloc[i]
            stop_loss = entry_price - 2 * atr.iloc[i]
            position = 1
        
        if position == 1:
            if close.iloc[i] < stop_loss:
                position = 0
                entry_price = None
        
        positions.append(position)
    
    return pd.DataFrame({'position': positions})
```

### 3. 布林带系统

```python
def bollinger_band_system(prices, period=20, num_std=2):
    """
    布林带突破系统
    
    规则:
    - 价格向上突破上轨 -> 做多
    - 价格向下突破下轨 -> 做空
    - 价格回归中轨 -> 平仓
    """
    signals = pd.DataFrame(index=prices.index)
    
    # 计算布林带
    signals['middle'] = prices['close'].rolling(period).mean()
    signals['std'] = prices['close'].rolling(period).std()
    signals['upper'] = signals['middle'] + num_std * signals['std']
    signals['lower'] = signals['middle'] - num_std * signals['std']
    
    # 信号
    signals['signal'] = 0
    signals.loc[prices['close'] > signals['upper'], 'signal'] = 1
    signals.loc[prices['close'] < signals['lower'], 'signal'] = -1
    
    return signals

# 布林带回归策略
def bollinger_mean_reversion(prices, period=20, num_std=2, lookback=5):
    """
    布林带均值回归策略
    
    规则:
    - 价格触及下轨 -> 买入
    - 价格触及上轨 -> 卖出
    """
    signals = pd.DataFrame(index=prices.index)
    
    # 计算布林带
    signals['middle'] = prices['close'].rolling(period).mean()
    signals['std'] = prices['close'].rolling(period).std()
    signals['upper'] = signals['middle'] + num_std * signals['std']
    signals['lower'] = signals['middle'] - num_std * signals['std']
    
    # 回归信号
    signals['position'] = 0
    
    # 价格触及下轨且前几日下跌
    signals.loc[(prices['close'] <= signals['lower']) & 
                (prices['close'].pct_change(lookback) < -0.05), 'position'] = 1
    
    # 价格触及上轨且前几日上涨
    signals.loc[(prices['close'] >= signals['upper']) & 
                (prices['close'].pct_change(lookback) > 0.05), 'position'] = -1
    
    # 回归中轨时平仓
    signals.loc[prices['close'] <= signals['middle'], 'position'] = 0
    
    return signals
```

---

## 三、趋势指标详解

### 1. 趋势强度指标

```python
def calculate_adx(high, low, close, period=14):
    """
    计算ADX (Average Directional Index)
    
    ADX > 25: 趋势强
    ADX < 20: 趋势弱
    """
    # +DM和-DM
    plus_dm = high.diff()
    minus_dm = -low.diff()
    
    plus_dm[plus_dm < 0] = 0
    minus_dm[minus_dm < 0] = 0
    
    # True Range
    tr1 = high - low
    tr2 = abs(high - close.shift(1))
    tr3 = abs(low - close.shift(1))
    tr = pd.concat([tr1, tr2, tr3], axis=1).max(axis=1)
    
    # 平滑
    atr = tr.rolling(period).mean()
    plus_di = 100 * (plus_dm.rolling(period).mean() / atr)
    minus_di = 100 * (minus_dm.rolling(period).mean() / atr)
    
    # DX和ADX
    dx = 100 * abs(plus_di - minus_di) / (plus_di + minus_di)
    adx = dx.rolling(period).mean()
    
    return adx, plus_di, minus_di

def trend_strength_signal(adx, plus_di, minus_di, threshold=25):
    """
    基于ADX的趋势信号
    """
    signal = 0
    
    if adx > threshold:
        if plus_di > minus_di:
            signal = 1  # 上升趋势
        else:
            signal = -1  # 下降趋势
    else:
        signal = 0  # 无趋势
    
    return signal
```

### 2. 趋势线交易

```python
def trend_line_system(prices, lookback=50):
    """
    趋势线突破系统
    """
    from scipy.signal import argrelextrema
    
    signals = pd.DataFrame(index=prices.index)
    
    # 找到局部极值
    highs = prices['high']
    lows = prices['low']
    
    local_highs = highs.iloc[argrelextrema(highs.values, np.greater, order=lookback)[0]]
    local_lows = lows.iloc[argrelextrema(lows.values, np.less, order=lookback)[0]]
    
    # 绘制趋势线
    # ...
    
    # 突破信号
    signals['resistance_break'] = prices['close'] > local_highs.shift(1)
    signals['support_break'] = prices['close'] < local_lows.shift(1)
    
    signals['signal'] = 0
    signals.loc[signals['resistance_break'], 'signal'] = 1
    signals.loc[signals['support_break'], 'signal'] = -1
    
    return signals
```

---

## 四、仓位管理

### 1. ATR仓位管理

```python
def calculate_position_size(capital, entry_price, stop_loss, atr):
    """
    基于ATR的仓位管理
    
    规则: 每笔交易风险敞口不超过账户的2%
    """
    # 风险金额
    risk_amount = capital * 0.02
    
    # ATR止损距离
    risk_per_share = atr
    
    # 仓位数量
    shares = risk_amount / risk_per_share
    
    # 资金占用
    capital_used = shares * entry_price
    
    # 仓位比例
    position_pct = capital_used / capital
    
    return {
        'shares': int(shares),
        'capital_used': capital_used,
        'position_pct': position_pct,
        'risk_per_share': risk_per_share
    }

# 海龟仓位公式
def turtle_position_unit(capital, atr, max_position_pct=0.02, max_units=4):
    """
    海龟交易法仓位单位
    
    N = ATR
    Unit = 账户1% / N
    最大持仓 = 4个单位
    """
    risk_per_unit = capital * 0.01 / atr
    
    return {
        'unit_size': int(risk_per_unit),
        'max_units': max_units,
        'max_position': int(risk_per_unit * max_units),
        'max_position_pct': max_position_pct * max_units
    }
```

### 2. 波动率仓位管理

```python
def volatility_position_sizing(returns, capital, target_vol=0.15, min_vol=0.05):
    """
    波动率仓位管理
    
    目标: 使策略波动率保持在恒定水平
    """
    # 计算已实现波动率
    realized_vol = returns.std() * np.sqrt(252)
    
    # 计算目标波动率仓位
    if realized_vol > min_vol:
        position_size = target_vol / realized_vol
    else:
        position_size = 1.5  # 低波动时增加仓位
    
    # 限制最大仓位
    position_size = min(position_size, 2.0)
    
    return {
        'position_size': position_size,
        'realized_vol': realized_vol,
        'vol_ratio': target_vol / realized_vol if realized_vol > 0 else 0
    }

class VolatilityTargeting:
    """波动率目标管理"""
    
    def __init__(self, target_vol=0.15, lookback=60):
        self.target_vol = target_vol
        self.lookback = lookback
    
    def compute_positions(self, returns, base_position):
        """
        计算目标仓位
        """
        # 已实现波动率
        realized_vol = returns.iloc[-self.lookback:].std() * np.sqrt(252)
        
        if realized_vol > 0:
            vol_scalar = self.target_vol / realized_vol
        else:
            vol_scalar = 1.0
        
        # 限制scalar范围
        vol_scalar = np.clip(vol_scalar, 0.5, 3.0)
        
        # 目标仓位
        target_position = base_position * vol_scalar
        
        return target_position
```

---

## 五、策略评估与优化

### 绩效指标

```python
def evaluate_cta_strategy(returns, risk_free_rate=0.03):
    """
    评估CTA策略绩效
    """
    results = {}
    
    # 收益指标
    results['total_return'] = (1 + returns).prod() - 1
    results['annual_return'] = (1 + returns.mean()) ** 252 - 1
    
    # 风险指标
    results['volatility'] = returns.std() * np.sqrt(252)
    results['max_drawdown'] = calculate_max_drawdown(returns)
    
    # 风险调整收益
    results['sharpe_ratio'] = (returns.mean() - risk_free_rate/252) / returns.std() * np.sqrt(252)
    results['sortino_ratio'] = calculate_sortino_ratio(returns, risk_free_rate)
    results['calmar_ratio'] = results['annual_return'] / abs(results['max_drawdown'])
    
    # 交易指标
    results['win_rate'] = (returns > 0).mean()
    results['avg_win'] = returns[returns > 0].mean()
    results['avg_loss'] = returns[returns < 0].mean()
    results['profit_factor'] = abs(results['avg_win'] / results['avg_loss']) if results['avg_loss'] != 0 else 0
    
    # 趋势跟踪特定指标
    results['up_capture'] = calculate_capture_ratio(returns, benchmark, direction='up')
    results['down_capture'] = calculate_capture_ratio(returns, benchmark, direction='down')
    
    return results

def calculate_max_drawdown(returns):
    """计算最大回撤"""
    equity = (1 + returns).cumprod()
    peak = equity.cummax()
    drawdown = (equity - peak) / peak
    return drawdown.min()

def calculate_sortino_ratio(returns, risk_free_rate=0.03):
    """计算Sortino比率"""
    excess_returns = returns - risk_free_rate / 252
    downside_returns = returns[returns < 0]
    
    if len(downside_returns) > 0:
        downside_std = downside_returns.std() * np.sqrt(252)
        return (excess_returns.mean() * 252) / downside_std
    return 0

def calculate_capture_ratio(strategy_returns, benchmark_returns, direction='up'):
    """计算捕获比"""
    if direction == 'up':
        up_months = benchmark_returns > 0
        if up_months.sum() > 0:
            return strategy_returns[up_months].mean() / benchmark_returns[up_months].mean()
    else:
        down_months = benchmark_returns < 0
        if down_months.sum() > 0:
            return strategy_returns[down_months].mean() / benchmark_returns[down_months].mean()
    return 0
```

### 参数优化

```python
def optimize_donchian_parameters(high, low, close, entry_range=range(10, 60, 5),
                                  exit_range=range(5, 30, 5)):
    """
    优化唐奇安通道参数
    """
    results = []
    
    for entry in entry_range:
        for exit_period in exit_range:
            if exit_period >= entry:
                continue
            
            # 运行策略
            signals = donchian_channel_system(high, low, close, entry, exit_period)
            returns = calculate_returns(signals, close)
            
            # 计算绩效
            performance = evaluate_cta_strategy(returns)
            
            results.append({
                'entry_period': entry,
                'exit_period': exit_period,
                'sharpe': performance['sharpe_ratio'],
                'annual_return': performance['annual_return'],
                'max_drawdown': performance['max_drawdown']
            })
    
    results_df = pd.DataFrame(results)
    
    # 找出最优参数
    best_sharpe = results_df.loc[results_df['sharpe'].idxmax()]
    best_return = results_df.loc[results_df['annual_return'].idxmax()]
    
    return {
        'all_results': results_df,
        'best_sharpe': best_sharpe,
        'best_return': best_return
    }

# 步行前进优化
def walk_forward_optimization(high, low, close, train_window=252, test_window=60):
    """
    步行前进优化
    """
    results = []
    
    for i in range(train_window, len(close) - test_window, test_window):
        train_data = slice(i - train_window, i)
        test_data = slice(i, i + test_window)
        
        # 训练集优化
        train_results = optimize_donchian_parameters(
            high[train_data], low[train_data], close[train_data]
        )
        
        best_params = {
            'entry': train_results['best_sharpe']['entry_period'],
            'exit': train_results['best_sharpe']['exit_period']
        }
        
        # 测试集评估
        test_signals = donchian_channel_system(
            high[test_data], low[test_data], close[test_data],
            best_params['entry'], best_params['exit']
        )
        test_returns = calculate_returns(test_signals, close[test_data])
        test_performance = evaluate_cta_strategy(test_returns)
        
        results.append({
            'period': i,
            'best_params': best_params,
            'train_sharpe': train_results['best_sharpe']['sharpe'],
            'test_sharpe': test_performance['sharpe_ratio']
        })
    
    return pd.DataFrame(results)
```

---

## 六、实战案例

### 完整CTA策略示例

```python
class CTAStrategy:
    """CTA趋势跟踪策略"""
    
    def __init__(self, config):
        self.entry_period = config.get('entry_period', 20)
        self.exit_period = config.get('exit_period', 10)
        self.atr_period = config.get('atr_period', 20)
        self.max_position_pct = config.get('max_position_pct', 0.02)
        
    def calculate_indicators(self, data):
        """计算指标"""
        df = pd.DataFrame()
        
        # 唐奇安通道
        df['upper'] = data['high'].rolling(self.entry_period).max()
        df['lower'] = data['low'].rolling(self.entry_period).min()
        
        # 出场通道
        df['exit_upper'] = data['high'].rolling(self.exit_period).max()
        df['exit_lower'] = data['low'].rolling(self.exit_period).min()
        
        # ATR
        tr = np.maximum(
            data['high'] - data['low'],
            np.maximum(
                abs(data['high'] - data['close'].shift(1)),
                abs(data['low'] - data['close'].shift(1))
            )
        )
        df['atr'] = tr.rolling(self.atr_period).mean()
        
        return df
    
    def generate_signals(self, data):
        """生成交易信号"""
        indicators = self.calculate_indicators(data)
        
        signals = pd.DataFrame(index=data.index)
        
        # 入场信号
        signals['entry_long'] = data['close'] > indicators['upper'].shift(1)
        signals['entry_short'] = data['close'] < indicators['lower'].shift(1)
        
        # 出场信号
        signals['exit_long'] = data['close'] < indicators['exit_lower'].shift(1)
        signals['exit_short'] = data['close'] > indicators['exit_upper'].shift(1)
        
        return signals
    
    def calculate_position(self, capital, atr):
        """计算仓位"""
        risk_amount = capital * self.max_position_pct
        return int(risk_amount / atr)
    
    def run_backtest(self, data, initial_capital=1000000):
        """运行回测"""
        signals = self.generate_signals(data)
        
        capital = initial_capital
        position = 0
        entry_price = 0
        entry_bar = 0
        
        trades = []
        equity_curve = [initial_capital]
        
        for i in range(len(data)):
            current_price = data['close'].iloc[i]
            atr = signals['atr'].iloc[i] if 'atr' in signals.columns else current_price * 0.02
            
            # 入场
            if position == 0:
                if signals['entry_long'].iloc[i]:
                    position_size = self.calculate_position(capital, atr)
                    if position_size > 0:
                        position = position_size
                        entry_price = current_price
                        entry_bar = i
                        capital -= position * current_price
                        
                elif signals['entry_short'].iloc[i]:
                    position_size = self.calculate_position(capital, atr)
                    if position_size > 0:
                        position = -position_size
                        entry_price = current_price
                        entry_bar = i
                        capital += position * current_price
            
            # 出场
            elif position > 0:
                if signals['exit_long'].iloc[i] or signals['entry_short'].iloc[i]:
                    capital += position * current_price
                    pnl = capital - initial_capital
                    trades.append({'pnl': pnl, 'bars': i - entry_bar})
                    position = 0
                    
            elif position < 0:
                if signals['exit_short'].iloc[i] or signals['entry_long'].iloc[i]:
                    capital += position * current_price
                    pnl = capital - initial_capital
                    trades.append({'pnl': pnl, 'bars': i - entry_bar})
                    position = 0
            
            # 更新权益
            if position != 0:
                portfolio_value = capital + position * current_price
            else:
                portfolio_value = capital
            equity_curve.append(portfolio_value)
        
        return {
            'final_capital': capital,
            'total_return': (capital - initial_capital) / initial_capital,
            'trades': pd.DataFrame(trades),
            'equity_curve': equity_curve
        }
```

---

## 七、组合多个CTA策略

```python
class CTAPortfolio:
    """CTA策略组合"""
    
    def __init__(self, strategies):
        self.strategies = strategies
        self.weights = None
    
    def equal_weight(self):
        """等权重分配"""
        n = len(self.strategies)
        self.weights = [1.0 / n] * n
    
    def risk_parity(self, historical_returns):
        """
        风险平价权重
        """
        volatilities = historical_returns.std() * np.sqrt(252)
        inv_vol = 1 / volatilities
        self.weights = (inv_vol / inv_vol.sum()).values.tolist()
    
    def optimize_weights(self, returns, method='max_sharpe'):
        """
        优化权重
        """
        from scipy.optimize import minimize
        
        def objective(weights):
            portfolio_return = (returns * weights).sum(axis=1)
            if method == 'max_sharpe':
                return -(portfolio_return.mean() / portfolio_return.std())
            elif method == 'min_vol':
                return portfolio_return.std()
        
        constraints = [
            {'type': 'eq', 'fun': lambda w: np.sum(w) - 1},
            {'type': 'ineq', 'fun': lambda w: w}  # 非负
        ]
        
        bounds = [(0, 1) for _ in range(len(self.strategies))]
        
        result = minimize(
            objective,
            np.ones(len(self.strategies)) / len(self.strategies),
            method='SLSQP',
            bounds=bounds,
            constraints=constraints
        )
        
        self.weights = result.x.tolist()
    
    def run_combined_backtest(self, data):
        """组合回测"""
        all_returns = []
        
        for strategy in self.strategies:
            result = strategy.run_backtest(data)
            returns = pd.Series(result['equity_curve']).pct_change().dropna()
            all_returns.append(returns)
        
        returns_df = pd.concat(all_returns, axis=1)
        portfolio_returns = (returns_df * self.weights).sum(axis=1)
        
        return evaluate_cta_strategy(portfolio_returns)
```

---

## 八、常见问题与解决方案

### 1. 趋势跟踪的困境

```python
# 问题: 大多数趋势跟踪策略在低波动市场表现不佳
# 解决方案: 多周期、多品种组合

def multi_timeframe_strategy(data, timeframes=[60, 240, 1440]):
    """
    多周期趋势策略
    
    规则: 仅在所有周期趋势一致时入场
    """
    signals = []
    
    for tf in timeframes:
        tf_data = resample_data(data, tf)
        tf_signal = donchian_channel_system(
            tf_data['high'], tf_data['low'], tf_data['close']
        )
        signals.append(tf_signal['position'])
    
    # 多数周期一致
    combined_signal = sum(signals) / len(signals)
    
    return combined_signal
```

### 2. 过度拟合问题

```python
# 解决方案: 限制参数搜索空间，使用样本外测试

def robust_parameter_selection(high, low, close):
    """
    稳健参数选择
    
    1. 使用较宽的参数范围
    2. 步行前进验证
    3. 要求样本外稳定性
    """
    # 参数范围
    entry_range = [20, 30, 40, 50]  # 较宽的步长
    exit_range = [10, 15, 20]  # 较宽的步长
    
    # 要求至少3个样本外周期有正收益
    valid_params = []
    
    for entry in entry_range:
        for exit_period in exit_range:
            wfa_results = walk_forward_optimization(
                high, low, close,
                train_window=252, test_window=60,
                entry=entry, exit=exit_period
            )
            
            positive_periods = (wfa_results['test_sharpe'] > 0).sum()
            
            if positive_periods >= 3:
                valid_params.append({
                    'entry': entry,
                    'exit': exit_period,
                    'avg_sharpe': wfa_results['test_sharpe'].mean()
                })
    
    return valid_params
```

---

*最后更新: 2026-04-26*