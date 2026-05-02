# Backtrader 完整教程

## 目录

1. Backtrader 简介
2. 安装与配置
3. 核心概念
4. 数据加载与处理
5. 策略开发
6. 指标计算
7. 订单与交易
8. 信号系统
9. 分析器与绩效评估
10. 可视化与输出
11. 高级特性
12. 实战案例
13. 常见问题与解决方案

---

## 1. Backtrader 简介

Backtrader 是一个 Python 语言的量化交易回测框架，由 Daniel Rodriguez 开发。
主要特点：简洁 API、模块化架构、丰富指标库、多种数据源支持、Cerebro 引擎。
---

## 2. 安装与配置

pip install backtrader
pip install matplotlib yfinance pandas numpy
---

## 3. 核心概念

Cerebro 是核心引擎：
import backtrader as bt
cerebro = bt.Cerebro()
cerebro.addstrategy(MyStrategy)
cerebro.adddata(data)
cerebro.broker.setcash(100000)
results = cerebro.run()
cerebro.plot()
---

## 4. 数据加载

CSV格式: datetime,open,high,low,close,volume
GenericCSVData: dataname, fromdate, todate, dtformat, datetime, open, high, low, close, volume
YahooFinanceData: dataname=AAPL
---

## 5. 策略开发

class MyStrategy(bt.Strategy):
    params = (("period", 20),)
    def __init__(self): self.sma = bt.indicators.SMA(self.data.close, period=20)
    def next(self):
        if self.sma[0] > self.data.close[0]: self.buy()
        elif self.sma[0] < self.data.close[0]: self.sell()
---

## 6. 指标计算

SMA, EMA: bt.indicators.SMA/EMA(self.data.close, period=20)
MACD: bt.indicators.MACD(self.data.close)
RSI: bt.indicators.RSI(self.data.close, period=14)
ATR: bt.indicators.ATR(self.data, period=14)
布林带: bt.indicators.BollingerBands(self.data.close, period=20)
---

## 7. 订单与交易

self.buy() / self.sell() - 市价单
self.buy(exectype=bt.Order.Limit, price=100.0) - 限价单
self.buy(exectype=bt.Order.Stop, price=95.0) - 止损单
notify_order() - 订单状态回调
---

## 8. 信号系统

CrossOver: 金叉/死叉信号
self.crossover = bt.indicators.CrossOver(fast_ma, slow_ma)
if crossover > 0: buy()
elif crossover < 0: sell()
---

## 9. 分析器

SharpeRatio, DrawDown, TradeAnalyzer, SQN
cerebro.addanalyzer(bt.analyzers.SharpeRatio, _name="sharpe")
results = cerebro.run()
strat = results[0]
sharpe = strat.analyzers.sharpe.get_analysis()
SQN评级: <1无用, 1-1.9一般, 2-2.9一般+, 3-3.9良好, 4-4.9优秀, 5-5.9卓越, >=6圣杯
---

## 10. 可视化

cerebro.plot(style="candlestick", vol=True, figsize=(16,8))
---

## 11. 高级特性

多数据源: cerebro.adddata(data1, name="AAPL")
参数优化: cerebro.optstrategy(MyStrategy, period=range(10,50,5))
---

## 12. 实战案例 - 双均线策略

class SMACrossStrategy(bt.Strategy):
    params = (("fast_period", 10), ("slow_period", 30))
    def __init__(self):
        self.fast_ma = bt.indicators.SMA(self.data.close, period=10)
        self.slow_ma = bt.indicators.SMA(self.data.close, period=30)
        self.crossover = bt.indicators.CrossOver(self.fast_ma, self.slow_ma)
    def next(self):
        if not self.position and self.crossover > 0: self.buy()
        elif self.position and self.crossover < 0: self.sell()
---

## 13. 常见问题

Q: CSV加载失败 - 检查列索引和日期格式
Q: 指标数据不足 - 使用 prenext/nextstart
Q: 订单未成交 - 检查 self.order 状态管理
---

## 附录
指标别名: SMA, EMA, MACD, RSI, ATR, BBands, CCI
资源: https://www.backtrader.com/
文档更新时间: 2024

---
完整策略示例代码
import backtrader as bt
from datetime import datetime
class CompleteStrategy(bt.Strategy):
    params = (fast_period=10, slow_period=30, rsi_period=14)
    def __init__(self):
        self.fast_ma = bt.indicators.SMA(self.data.close, period=10)
        self.slow_ma = bt.indicators.SMA(self.data.close, period=30)
        self.order = None
    def next(self):
        if self.order: return
        if not self.position and self.fast_ma[0] > self.slow_ma[0]: self.buy()
        elif self.position and self.fast_ma[0] < self.slow_ma[0]: self.sell()
cerebro = bt.Cerebro()
cerebro.addstrategy(CompleteStrategy)
data = bt.feeds.YahooFinanceData(dataname=AAPL, fromdate=datetime(2020,1,1), todate=datetime(2023,12,31))
cerebro.adddata(data)
cerebro.broker.setcash(100000)
cerebro.run()
cerebro.plot()
---
更多指标: OBV, Stochastic, CCI, Momentum
经纪商配置: cerebro.broker.setcommission(commission=0.001)
文档更新时间: 2024