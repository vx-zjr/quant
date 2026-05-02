# Alpha Vantage 数据接口详解

**版本**: 2024年
**官方文档**: https://www.alphavantage.co/documentation/
**官方Python库**: pip install alpha-vantage

---

## 目录

- [简介](#简介)
- [API Key申请](#api-key申请)
- [核心股票API](#核心股票api)
- [指数数据API](#指数数据api-premium)
- [期权数据API](#期权数据api)
- [Alpha Intelligence](#alpha-intelligence)
- [基本面数据](#基本面数据)
- [外汇与加密货币](#外汇与加密货币)
- [大宗商品](#大宗商品)
- [经济指标](#经济指标)
- [技术指标](#技术指标)
- [Python示例代码](#python示例代码)

---

## 简介

Alpha Vantage是一家提供金融市场数据的领先API服务提供商，为投资者、量化交易者和开发者提供实时及历史市场数据。

### 主要特点
- 支持股票、外汇、加密货币、大宗商品等多种资产类别
- 提供日内、日、周、月多种时间频率的数据
- 内置50+技术指标
- 支持JSON和CSV格式输出
- 提供官方Python库和MCP服务器
- 支持全球100,000+交易品种

---

## API Key申请

**申请地址**: https://www.alphavantage.co/support/#api-key

### 免费套餐限制
- 每分钟5次请求
- 每日500次请求
- 部分高级功能需要付费订阅

### 基础URL
```
https://www.alphavantage.co/query
```

### 通用参数
| 参数 | 说明 |
|------|------|
| function | API函数名称 |
| symbol | 交易品种代码 |
| apikey | 你的API密钥 |
| datatype | 返回格式(json/csv)，默认json |
| outputsize | compact(最近100条) 或 full(全部历史) |

---

## 核心股票API

### 1. 日内数据 (TIME_SERIES_INTRADAY)

返回指定股票的交易日内分钟级数据。

**函数名**: TIME_SERIES_INTRADAY

**参数**:
| 参数 | 必填 | 说明 |
|------|------|------|
| function | 是 | TIME_SERIES_INTRADAY |
| symbol | 是 | 股票代码，如 IBM |
| interval | 是 | 1min, 5min, 15min, 30min, 60min |
| outputsize | 否 | compact(最近100条) 或 full(全部) |
| apikey | 是 | 你的API密钥 |

**示例URL**:
```
https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=IBM&interval=5min&apikey=demo
```

### 2. 日线数据 (TIME_SERIES_DAILY)

返回股票的日线数据，包含开盘价、最高价、最低价、收盘价和成交量。

**函数名**: TIME_SERIES_DAILY

**参数**: 同日内数据

**返回字段**: timestamp, open, high, low, close, volume

### 3. 调整后日线数据 (TIME_SERIES_DAILY_ADJUSTED)

返回调整后的日线数据，包括拆分和股息调整。

**返回字段**: open, high, low, close, adjusted_close, volume, dividend_amount, split_coefficient

### 4. 周线/月线数据
- TIME_SERIES_WEEKLY (周线)
- TIME_SERIES_WEEKLY_ADJUSTED (调整后周线)
- TIME_SERIES_MONTHLY (月线)
- TIME_SERIES_MONTHLY_ADJUSTED (调整后月线)

覆盖20+年历史数据。

### 5. 实时报价 (GLOBAL_QUOTE)

获取股票的实时价格信息。

**函数名**: GLOBAL_QUOTE
**参数**: function, symbol, apikey

### 6. 股票搜索 (SYMBOL_SEARCH)

根据关键字搜索股票代码和公司信息。

**函数名**: SYMBOL_SEARCH
**参数**: function, keywords, apikey

### 7. 全球市场状态 (MARKET_STATUS)
查看全球主要市场的开闭市状态。
**函数名**: MARKET_STATUS

---

## 指数数据API (Premium)

提供主要市场指数的历史数据，需要Premium订阅。

### 支持的指数
| 指数名称 | 代码 |
|----------|------|
| 道琼斯工业平均指数 | ^.DJI |
| 标普500 | ^GSPC |
| 纳斯达克综合指数 | ^IXIC |
| 纳斯达克100 | ^NDX |
| CBOE VIX波动率指数 | ^VIX |
| 罗素2000小盘股指数 | ^RUT |

---

## 期权数据API

### 1. 实时期权数据 (OPTION_DATA)
获取股票期权的实时数据，包括看涨期权和看跌期权链。

**函数名**: OPTION_DATA
**参数**: function, symbol, interval (realtime/15min/30min/60min), apikey

### 2. 历史期权数据 (HISTORICAL_OPTIONS)
**函数名**: HISTORICAL_OPTIONS
**参数**: function, symbol, datatype (json/csv), apikey

### 3. 看跌/看涨比率
- 实时: PUT_CALL_RATIO&interval=realtime
- 历史: HISTORICAL_PUT_CALL_RATIO

---

## Alpha Intelligence

提供新闻情绪分析、财报电话会议记录、内部人交易等数据。

### 1. 新闻情绪 (NEWS_SENTIMENT)
获取金融新闻和情绪分析数据。

**函数名**: NEWS_SENTIMENT
**参数**: tickers, topics, sort (latest/popular), limit, apikey

### 2. 财报电话会议记录 (EARNINGS_CALL_TRANSCRIPT)
**函数名**: EARNINGS_CALL_TRANSCRIPT

### 3. 内部人交易 (INSIDER_TRANSACTIONS)
获取公司内部人员的股票交易信息。

### 4. 机构持仓 (INSTITUTIONAL_HOLDINGS)

### 5. Top Gainers & Losers
获取当日涨幅最大和跌幅最大的股票。
**函数名**: TOP_GAINERS_LOSERS

---

## 基本面数据

### 1. 公司概览 (COMPANY_OVERVIEW)
获取公司的基本信息、财务数据和估值指标。

**函数名**: COMPANY_OVERVIEW
**返回字段**: Symbol, Name, Description, Sector, Industry, MarketCapitalization, EBIT, EBITDA, Revenue, GrossProfit, NetIncome, EPS, PE, PEG, DividendPerShare, DividendYield

### 2. 利润表 (INCOME_STATEMENT)

### 3. 资产负债表 (BALANCE_SHEET)

### 4. 现金流量表 (CASH_FLOW)

### 5. 每股收益历史 (EARNINGS)

### 6. 盈利预期 (EARNINGS_ANALYTICS)

### 7. 流通股数 (SHARES_OUTSTANDING)

### 8. 财报日历 (EARNINGS_CALENDAR)

### 9. IPO日历 (IPO_CALENDAR)

### 10. 股息数据 (DIVIDENDS) 和 股票拆分 (SPLITS)

---

## 外汇与加密货币

### 外汇 (Forex)
- CURRENCY_EXCHANGE_RATE: 汇率查询
- FX_INTRADAY: 日内外汇数据 [Premium]
- FX_DAILY: 日线外汇数据
- FX_WEEKLY: 周线外汇数据
- FX_MONTHLY: 月线外汇数据

### 加密货币
- CRYPTO_EXCHANGE_RATE: 加密货币汇率查询
- CRYPTO_INTRADAY: 日内加密货币数据 [Premium]
- CRYPTO_DAILY: 日线加密货币数据
- CRYPTO_WEEKLY: 周线加密货币数据
- CRYPTO_MONTHLY: 月线加密货币数据

---

## 大宗商品

- GOLD_SILVER_HISTORY: 黄金白银历史数据
- WTI: WTI原油价格
- BRENT: 布伦特原油价格
- NATURAL_GAS: 天然气价格
- 其他: 铜、铝、小麦、玉米、棉花、糖、咖啡等

---

## 经济指标

| 指标 | 函数名 |
|------|--------|
| 实际GDP | REAL_GDP |
| 人均GDP | REAL_GDP_PER_CAPITA |
| 国债收益率 | TREASURY_YIELD |
| 联邦基金利率 | FEDERAL_FUNDS_RATE |
| CPI | CPI |
| 通货膨胀 | INFLATION |
| 零售销售 | RETAIL_SALES |
| 耐用品订单 | DURABLE_GOODS |
| 失业率 | UNEMPLOYMENT |
| 消费者信心 | CONSUMER_CONFIDENCE |

---

## 技术指标

Alpha Vantage提供50+技术指标。

### 常用技术指标

| 指标 | 函数名 | 说明 |
|------|--------|------|
| SMA | SMA | 简单移动平均 |
| EMA | EMA | 指数移动平均 |
| VWAP | VWAP | 成交量加权平均价 |
| BBANDS | BBANDS | 布林带 |
| MACD | MACD | 移动平均收敛/发散 |
| RSI | RSI | 相对强弱指数 |
| ADX | ADX | 平均方向指数 |
| CCI | CCI | 商品通道指数 |
| ATR | ATR | 平均真实波幅 |
| OBV | OBV | 能量潮 |

---

## Python示例代码

### 安装官方库
```bash
pip install alpha-vantage
```

### 获取日线数据
```python
from alpha_vantage.timeseries import TimeSeries

print(data)
```

### 使用requests库
```python
import requests

url = 
  https://www.alphavantage.co/query?function=TIME_SERIES_DAILY_ADJUSTED
  &symbol=IBM&outputsize=full&apikey=demo
r = requests.get(url)
data = r.json()
print(data)
```

### 获取实时报价
```python
from alpha_vantage.foreignexchange import ForeignExchange

print(data)
```

### 获取基本面数据
```python
from alpha_vantage.fundamentaldata import FundamentalData

print(overview)
```

---

## MCP服务器集成

Alpha Vantage提供官方MCP服务器，可用于LLM和AI Agent集成。

**MCP服务器地址**: https://mcp.alphavantage.co/


---

## 错误处理

### 常见错误码

| 代码 | 说明 | 解决方案 |
|------|------|----------|
| 200 | 请求成功 | - |
| 429 | 请求频率超限 | 降低请求频率或升级套餐 |
| 403 | API密钥无效 | 检查API密钥是否正确 |
| 404 | 股票代码不存在 | 验证股票代码 |
| 500 | 服务器错误 | 重试请求 |

### 限流提示

免费用户: 每分钟5次，每天500次

建议实现:
- 请求间隔控制
- 缓存机制
- 错误重试逻辑

---

## 相关资源

- 官方文档: https://www.alphavantage.co/documentation/
- API Key申请: https://www.alphavantage.co/support/#api-key
- Python库: https://github.com/RomelTorres/alpha_vantage
- MCP服务器: https://mcp.alphavantage.co/

---

## 附录: API调用示例补充

### 使用alpha-vantage库获取日线数据
```python
from alpha_vantage.timeseries import TimeSeries

ts = TimeSeries(key="YOUR_API_KEY", output_format="pandas")
data, meta_data = ts.get_daily(symbol="IBM", outputsize="full")
print(data)
```

### 使用alpha-vantage库获取实时汇率
```python
from alpha_vantage.foreignexchange import ForeignExchange

fx = ForeignExchange(key="YOUR_API_KEY")
data = fx.get_currency_exchange_rate("BTC", "USD")
print(data)
```

### 使用alpha-vantage库获取基本面数据
```python
from alpha_vantage.fundamentaldata import FundamentalData

fd = FundamentalData(key="YOUR_API_KEY")
overview = fd.get_company_overview("IBM")
print(overview)
```

---

## 附录: 全球股票代码格式

### 美国市场
- 直接使用股票代码，如 IBM, AAPL, MSFT

### 国际市场格式
- 英国伦敦证券交易所: TSCO.LON
- 日本东京证券交易所: 7203.TYO
- 德国法兰克福交易所: BMW.DE
- 印度孟买证券交易所: RELIANCE.BSE
- 中国上海证券交易所: 600104.SHH
- 中国深圳证券交易所: 000002.SHZ


---
文档创建完成
最后更新: 2024年
