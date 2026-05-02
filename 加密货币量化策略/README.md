# 加密货币量化策略完整文档

---
## 目录

1. [项目概述](#项目概述)
2. [交易所API接口](#交易所api接口)
3. [做市策略](#做市策略)
4. [套利策略](#套利策略)
5. [趋势跟踪策略](#趋势跟踪策略)
6. [统计套利](#统计套利)
7. [DeFi量化](#defi量化)
8. [链上分析](#链上分析)
9. [风险管理](#风险管理)
10. [配置说明](#配置说明)

---

## 项目概述

本项目提供了一套完整的加密货币量化交易系统。

### 技术栈

- Python 3.9+
- CCXT (统一交易所API)
- WebSocket (实时数据流)
- NumPy/Pandas (数据分析)

---

## 交易所API接口

### Binance交易所实现

```python
class BinanceExchange:
    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret
        self.base_url = "https://api.binance.com"

    async def fetch_ticker(self, symbol):
        # 获取24小时行情
        pass

    async def create_order(self, symbol, side, price, quantity):
        # 创建订单
        pass
```

---

## 做市策略

### 订单簿分析

```python
class MarketMakerStrategy:
    def __init__(self, exchange, symbol, spread_pct=0.001):
        self.exchange = exchange
        self.symbol = symbol
        self.spread_pct = spread_pct

    async def calculate_orders(self):
        # 基于价差计算挂单价格
        pass
```

---

## 套利策略

### 三角套利

```python
class TriangularArbitrage:
    def __init__(self, exchange):
        self.exchange = exchange
        self.paths = []

    def find_opportunities(self):
        # 查找三角套利机会
        pass

    async def execute(self, path):
        # 执行套利交易
        pass
```

---

## 趋势跟踪策略

### CTA策略

```python
class CTAStrategy:
    def __init__(self, symbols, lookback=20):
        self.symbols = symbols
        self.lookback = lookback
        self.position = {}

    def generate_signals(self, candles):
        # 生成交易信号
        pass
```

---

## 统计套利

### 配对交易策略

```python
class PairsTradingStrategy:
    def __init__(self, symbol1, symbol2, lookback=60):
        self.symbol1 = symbol1
        self.symbol2 = symbol2
        self.lookback = lookback
        self.hedge_ratio = 0
        self.spread_mean = 0
        self.spread_std = 0

    def calculate_spread(self, price1, price2):
        # 计算价差
        pass

    def generate_signals(self):
        # 生成套利信号
        pass
```

---

## DeFi量化

### DEX套利

```python
class DEXArbitrage:
    def __init__(self, routers):
        self.routers = routers  # Uniswap, SushiSwap, etc.

    def find_arbitrage(self, token_in, path):
        # 查找DEX套利机会
        pass

    async def execute_swap(self, router, path, amount):
        # 执行交换
        pass
```

### 流动性挖矿

```python
class LiquidityMining:
    def __init__(self, protocols):
        self.protocols = protocols
        self.positions = {}

    def calculate_apy(self, pool):
        # 计算年化收益率
        pass

    def allocate_capital(self, pools):
        # 分配资金
        pass
```

---

## 链上分析

### Whale追踪

```python
class WhaleTracker:
    def __init__(self, threshold_usd=1000000):
        self.threshold = threshold_usd
        self.alerts = []

    def monitor_transactions(self):
        # 监控大额转账
        pass

    def generate_signals(self, txs):
        # 生成跟随信号
        pass
```

### Gas优化

```python
class GasOptimizer:
    def __init__(self):
        self.gas_history = []

    def estimate_optimal_gas(self):
        # 估算最优Gas价格
        pass

    def wait_for_gas(self, target_gas):
        # 等待合适Gas
        pass
```

---

## 风险管理

### 风险管理系统

```python
class RiskManager:
    def __init__(self, max_position_pct=0.1, max_drawdown=0.2):
        self.max_position_pct = max_position_pct
        self.max_drawdown = max_drawdown
        self.current_drawdown = 0

    def check_position(self, size, price):
        # 检查仓位是否合规
        pass

    def check_drawdown(self, equity):
        # 检查回撤限制
        pass
```

---

## 配置说明

```yaml
# config.yaml
exchange:
  binance:
    api_key: "your_api_key"
    api_secret: "your_secret"
    testnet: false

strategy:
  market_maker:
    spread_pct: 0.001
    order_size_pct: 0.01

risk:
  max_position: 0.1
  max_drawdown: 0.2
  stop_loss: 0.05
```

---

*文档结束*

## 做市策略详细实现```python
class MarketMakerStrategy:
    def __init__(self, exchange, symbol, config):
        self.exchange = exchange
        self.symbol = symbol
        self.spread_pct = config.get("spread_pct", 0.001)
        self.order_size_pct = config.get("order_size_pct", 0.01)
        self.active_orders = []
        self.position = 0
    
    async def calculate_spread(self, mid_price):
        # 基于波动率调整价差
        pass
    
    async def place_orders(self, bid_price, ask_price, size):
        # 挂单
        pass
    
    async def cancel_orders(self):
        # 取消订单
        pass
```---
## 套利策略详细实现

### 跨交易所套利

```python
class CrossExchangeArbitrage:
    def __init__(self, exchange_a, exchange_b, symbol):
        self.exchange_a = exchange_a
        self.exchange_b = exchange_b
        self.symbol = symbol
        self.min_profit_pct = 0.001  # 最小利润阈值
    
    async def check_opportunity(self):
        # 检查套利机会
        price_a = await self.exchange_a.fetch_ticker(self.symbol)
        price_b = await self.exchange_b.fetch_ticker(self.symbol)
        profit_pct = abs(price_a - price_b) / min(price_a, price_b)
        return profit_pct > self.min_profit_pct
    
    async def execute(self):
        # 执行套利
        pass
```
---
## 趋势跟踪策略详细实现

### 双均线策略

```python
class DualMovingAverageStrategy:
    def __init__(self, fast_period=10, slow_period=30):
        self.fast_period = fast_period
        self.slow_period = slow_period
        self.position = 0
    
    def calculate_ma(self, prices, period):
        # 计算移动平均
        return sum(prices[-period:]) / period
    
    def generate_signal(self, prices):
        fast_ma = self.calculate_ma(prices, self.fast_period)
        slow_ma = self.calculate_ma(prices, self.slow_period)
        if fast_ma > slow_ma and self.position <= 0:
            return 1  # 做多信号
        elif fast_ma < slow_ma and self.position >= 0:
            return -1  # 做空信号
        return 0
```
---
## 统计套利详细实现

### 协整套利策略

```python
import numpy as np
from sklearn.linear_model import LinearRegression

class CointegrationArbitrage:
    def __init__(self, symbol1, symbol2, lookback=100):
        self.symbol1 = symbol1
        self.symbol2 = symbol2
        self.lookback = lookback
        self.hedge_ratio = 0
        self.spread_mean = 0
        self.spread_std = 0
        self.entry_threshold = 2.0  # 进场阈值(标准差)
        self.exit_threshold = 0.5  # 出场阈值
    
    def calculate_spread(self, prices1, prices2):
        # 计算对冲比例
        model = LinearRegression()
        model.fit(np.array(prices1).reshape(-1, 1), prices2)
        self.hedge_ratio = model.coef_[0]
        spread = prices2 - self.hedge_ratio * np.array(prices1)
        self.spread_mean = np.mean(spread)
        self.spread_std = np.std(spread)
        return spread
    
    def generate_signal(self, current_spread):
        z_score = (current_spread - self.spread_mean) / self.spread_std
        if z_score > self.entry_threshold:
            return -1  # 做空价差
        elif z_score < -self.entry_threshold:
            return 1  # 做多价差
        elif abs(z_score) < self.exit_threshold:
            return 0  # 平仓
        return None
```
---
## DeFi量化详细实现

### AMM套利机器人

```python
from web3 import Web3

class AMMArbitrage:
    def __init__(self, web3_url):
        self.w3 = Web3(Web3.HTTPProvider(web3_url))
        self.uniswap_router = "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"
        self.sushiswap_router = "0xd9e1cE17f2641f24aE83637ab66a2cca9C378B9F"
        self.pancakeswap_router = "0x05fF2B0DB69458A0750badebc4f3e32a9C0cE7C1"
    
    async def get_amounts_out(self, router, path, amount_in):
        # 获取输出数量
        pass
    
    def find_arbitrage_opportunity(self, token_in, token_out, amount):
        # 查找最优路径
        routers = [self.uniswap_router, self.sushiswap_router, self.pancakeswap_router]
        results = []
        for router in routers:
            amount_out = self.get_amounts_out(router, [token_in, token_out], amount)
            results.append((router, amount_out))
        return max(results, key=lambda x: x[1])
    
    async def execute_arbitrage(self, router, path, amount_in):
        # 执行套利交易
        pass
```
---
## 链上分析详细实现

### Whale追踪系统

```python
class WhaleTracker:
    def __init__(self, web3_url, threshold_usd=1000000):
        self.w3 = Web3(Web3.HTTPProvider(web3_url))
        self.threshold_usd = threshold_usd
        self.tracked_addresses = set()
        self.alerts = []
        self.price_cache = {}
    
    async def get_token_price(self, token_address):
        # 获取代币价格
        pass
    
    async def parse_transfer_log(self, log):
        # 解析转账日志
        pass
    
    def check_whale_transaction(self, tx_value_usd, from_address, to_address):
        # 检查是否为鲸鱼交易
        if tx_value_usd >= self.threshold_usd:
            alert = {
                "from": from_address,
                "to": to_address,
                "value_usd": tx_value_usd
            }
            self.alerts.append(alert)
            return True
        return False
    
    async def monitor_pending_transactions(self):
        # 监控待处理交易
        pass
```
---
## 风险管理详细实现

### 综合风险管理器

```python
class RiskManager:
    def __init__(self, config):
        self.max_position_pct = config.get("max_position_pct", 0.1)
        self.max_drawdown = config.get("max_drawdown", 0.2)
        self.max_daily_loss = config.get("max_daily_loss", 0.05)
        self.stop_loss_pct = config.get("stop_loss_pct", 0.02)
        self.take_profit_pct = config.get("take_profit_pct", 0.04)
        self.max_leverage = config.get("max_leverage", 3)
        self.equity = 0
        self.peak_equity = 0
        self.daily_pnl = 0
        self.positions = {}
    
    def calculate_position_size(self, price, stop_loss_pct):
        # 计算仓位大小
        risk_amount = self.equity * 0.01  # 每笔交易风险1%
        position_size = risk_amount / (price * stop_loss_pct)
        return position_size
    
    def check_position_limits(self, symbol, size, price):
        # 检查仓位限制
        position_value = size * price
        max_position_value = self.equity * self.max_position_pct
        if position_value > max_position_value:
            return False
        return True
    
    def check_drawdown_limits(self):
        # 检查回撤限制
        self.peak_equity = max(self.peak_equity, self.equity)
        drawdown = (self.peak_equity - self.equity) / self.peak_equity if self.peak_equity > 0 else 0
        if drawdown > self.max_drawdown:
            return False
        return True
    
    def check_daily_loss_limit(self):
        # 检查每日亏损限制
        if abs(self.daily_pnl) > self.equity * self.max_daily_loss:
            return False
        return True
    
    def should_stop_out(self, position):
        # 检查是否应该止损
        unrealized_pnl = (position["current_price"] - position["entry_price"]) * position["size"]
        pnl_pct = unrealized_pnl / (position["entry_price"] * position["size"])
        if position["side"] == "long" and pnl_pct < -self.stop_loss_pct:
            return True
        if position["side"] == "short" and pnl_pct < -self.stop_loss_pct:
            return True
        return False
    
    def update_equity(self, pnl):
        # 更新权益
        self.equity += pnl
        self.daily_pnl += pnl
```
---
## 完整示例

### 主程序示例

```python
import asyncio
from exchange.binance import BinanceExchange
from strategies.trend import DualMovingAverageStrategy
from risk.manager import RiskManager

async def main():
    # 初始化交易所
    exchange = BinanceExchange(
        api_key="your_api_key",
        api_secret="your_secret"
    )
    
    # 初始化策略
    strategy = DualMovingAverageStrategy(fast_period=10, slow_period=30)
    
    # 初始化风险管理
    risk_mgr = RiskManager({
        "max_position_pct": 0.1,
        "max_drawdown": 0.2
    })
    
    # 主循环
    async with exchange:
        while True:
            # 获取数据
            candles = await exchange.fetch_candles("BTCUSDT", "1h", limit=100)
            prices = [c.close for c in candles]
            
            # 生成信号
            signal = strategy.generate_signal(prices)
            
            # 执行交易
            if signal != 0 and risk_mgr.check_position_limits(...):
                # 下单
                pass
            
            await asyncio.sleep(60)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 部署说明

1. **环境配置**: Python 3.9+, 安装依赖包
2. **API配置**: 配置交易所API密钥
3. **运行测试**: 使用回测验证策略
4. **实盘部署**: 逐步增加仓位

---

*文档结束*