# 市场微观结构

## 目录
- [订单簿建模](./订单簿建模.md)
- [交易成本分析](./交易成本分析.md)
- [市场深度分析](./市场深度分析.md)
- [高频数据处理](./高频数据处理.md)

---

## 一、订单簿建模

### 订单簿结构
| 概念 | 描述 |
|------|------|
| 限价订单簿 | 买卖盘口数据结构 |
| 市价订单簿 | 成交簿 |
| 订单流 | 订单到达过程 |

### 订单簿模型
| 模型 | 特点 |
|------|------|
| Glosten-Milgrom | 知情交易者模型 |
| Kyle模型 | 批量竞价模型 |
| Ho-Stoll模型 | 动态做市 |

### 订单簿动力学
```python
# 订单簿分析示例
import pandas as pd
import numpy as np

def calculate_orderbook_imbalance(depth_data):
    """
    计算订单簿不平衡度
    """
    bid_volume = depth_data['bid_volume'].sum()
    ask_volume = depth_data['ask_volume'].sum()
    return (bid_volume - ask_volume) / (bid_volume + ask_volume)

def estimate_spread(depth_data):
    """
    估计有效价差
    """
    best_bid = depth_data['bid_price'].max()
    best_ask = depth_data['ask_price'].min()
    return best_ask - best_bid
```

---

## 二、交易成本分析(TCA)

### 成本构成
| 类型 | 描述 | 计算 |
|------|------|------|
| 佣金 | 交易所收费 | 固定费用 |
| 价差 | 买卖价差 | 滑点 |
| 市场冲击 | 大单冲击 | 价格偏移 |
| 机会成本 | 未成交损失 | 延迟成本 |

### TCA指标
| 指标 | 公式 | 优化方向 |
|------|------|----------|
| 实施差距 | (执行价格-基准价格)/基准价格 | 越小越好 |
| 有效价差 | 2×(执行价-中间价)/(中间价) | 越小越好 |
| 市场冲击 | 大单对价格的影响 | 越小越好 |
| 到达价格 | 执行价格vs到达价格 | 接近到达价 |

### TCA工具
```python
# TCA分析框架
class TCAnalyzer:
    def __init__(self, market_data):
        self.market_data = market_data
        
    def calculate_implementation_shortfall(self, order, execution):
        """
        计算实施差距
        """
        decision_price = self.market_data.loc[order.decision_time, 'price']
        avg_exec_price = np.average(execution['price'], 
                                    weights=execution['shares'])
        return (avg_exec_price - decision_price) / decision_price
    
    def analyze_market_impact(self, order, market_data):
        """
        分析市场冲击
        """
        # 临时性冲击 vs 永久性冲击
        pass
```

---

## 三、市场深度分析

### 深度指标
| 指标 | 计算 | 含义 |
|------|------|------|
| 深度厚度 | 各价位挂单量之和 | 支撑/阻力强度 |
| 深度梯度 | 相邻价位变化率 | 流动性分布 |
| 订单流失衡 | 买卖单净流入 | 短期价格方向 |

### 流动性度量
| 度量 | 公式 | 说明 |
|------|------|------|
| Amihud IL | \|收益\|/成交量 | 非流动性 |
| Pastor-Stambaugh | 收益反转系数 | 流动性风险 |
| Roll模型 | 价差估计 | 有效价差 |

### 流动性异常
| 现象 | 描述 | 策略 |
|------|------|------|
| 流动性枯竭 | 极端波动时流动性消失 | 降低仓位 |
| 价差扩大 | 买卖价差急剧扩大 | 等待流动性恢复 |
| 价格冲击加剧 | 大单影响放大 | 拆单交易 |

---

## 四、高频数据处理

### 数据类型
| 类型 | 频率 | 内容 |
|------|------|------|
| Tick数据 | 微秒级 | 逐笔成交 |
| Level2 | 秒级 | 订单簿快照 |
| K线 | 自定义 | 聚合数据 |

### 数据处理框架
```python
# 高频数据处理
import pandas as pd
import numpy as np
from collections import deque

class TickProcessor:
    def __init__(self):
        self.tick_buffer = deque(maxlen=10000)
        
    def process_tick(self, tick):
        """处理单笔Tick"""
        self.tick_buffer.append(tick)
        
    def calculate_features(self):
        """计算Tick级特征"""
        ticks = pd.DataFrame(self.tick_buffer)
        return {
            'spread': ticks['ask'] - ticks['bid'],
            'mid_price': (ticks['ask'] + ticks['bid']) / 2,
            'volume': ticks['volume'].sum(),
            'vwap': (ticks['price'] * ticks['volume']).sum() / ticks['volume'].sum()
        }

class OrderBookProcessor:
    def __init__(self, depth=10):
        self.depth = depth
        self.bid_levels = []
        self.ask_levels = []
        
    def update(self, bid_df, ask_df):
        """更新订单簿"""
        self.bid_levels = bid_df.head(self.depth)
        self.ask_levels = ask_df.head(self.depth)
```

### 数据存储
| 数据库 | 适用场景 | 特点 |
|--------|---------|------|
| KDB+ | 高频数据 | 高性能时序 |
| DolphinDB | 高频数据 | 国产高性能 |
| InfluxDB | 时序数据 | 开源易用 |
| TimescaleDB | 时序数据 | PostgreSQL扩展 |

---

## 五、市场结构差异

### A股vs美股
| 维度 | A股 | 美股 |
|------|-----|------|
| 交易时间 | 4小时 | 6.5小时 |
| T+1制度 | 是 | 否 |
| 涨跌停 | 10%/20% | 无 |
| 订单类型 | 限价为主 | 多种多样 |
| 量化比例 | 20-30% | 60%+ |

### 市场机制对比
| 机制 | 中国 | 美国 |
|------|------|------|
| 集合竞价 | 开盘前15分钟 | 开盘前30分钟 |
| 连续交易 | 是 | 是 |
| 暗池 | 有限 | 发达 |
| 高频监管 | 加强中 | 成熟 |

---

## 六、最优执行

### 交易算法
| 算法 | 特点 | 适用场景 |
|------|------|----------|
| VWAP | 成交量加权 | 大宗交易 |
| TWAP | 时间加权 | 流动性好 |
| POV | 百分比成交量 | 降低冲击 |
| IS | 实施差距 | 主动执行 |
| Almgren-Chriss | 最优执行 | 风险控制 |

### 交易策略
```python
# Almgren-Chriss最优执行
def almgren_chriss(S0, T, N, sigma, eta, gamma, lambda_):
    """
    最优交易执行路径
    
    参数:
    S0: 初始价格
    T: 交易周期
    N: 总交易量
    sigma: 波动率
    eta: 临时性冲击系数
    gamma: 永久性冲击系数
    lambda_: 风险厌恶
    """
    kappa = np.sqrt(lambda_ / (eta * gamma))
    
    def x(t):
        """交易路径"""
        return N * (1 - np.sinh(kappa * (T - t)) / np.sinh(kappa * T)) / 2
    
    return x
```

---

*最后更新: 2026-04-26*
