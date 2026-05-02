# Yahoo Finance ���ݽӿ����

> **����**: 2026-04-26 | **�汾**: yfinance

---

## һ������

Yahoo Finance��Yahoo�ṩ����ѽ�������ƽ̨���ṩȫ���Ʊ���ڻ�����㡢���ܻ��ҵ����ݡ�**yfinance**��Python�������е�Yahoo Finance���ݻ�ȡ�⡣
### ��Ҫ�ص�

- ��ȫ��ѣ�����API Key
- ֧�����ɡ��۹ɡ�A�ɡ��ڻ�����㡢���ܻ���
- �������ͷḻ��K�ߡ��������ݡ���Ȩ���������ֲֵ�
- Pythonic API������ʹ��

### ��װ����

```bash
pip install yfinance
```

---

## ��������APIʹ��

### 2.1 �����۸����ݻ�ȡ

```python
import yfinance as yf

# ����1��ͨ��Ticker����
ticker = yf.Ticker("AAPL")
df = ticker.history(start="2024-01-01", end="2025-12-31")
print(df.head())

# ����2��ͨ��download����������ȡ
df = yf.download("AAPL", start="2024-01-01", end="2025-12-31")
```

**���������У�**
- Open/High/Low/Close (OHLC)
- Volume (�ɽ���)
- Dividends (�ֺ�)
- Stock Splits (���)

### 2.2 ��ͬʱ����������

```python
# �������� (Ĭ��)
df = yf.download("AAPL", period="1y")

# ���Ӽ�������
# interval: 1m, 2m, 5m, 15m, 30m, 60m, 90m, 1h, 1d, 5d, 1wk, 1mo
df_5m = yf.download("AAPL", interval="5m", period="5d")
df_1h = yf.download("AAPL", interval="1h", period="7d")
```

### 2.3 period �� interval ����˵��

| ���� | ��ѡֵ | ˵�� |
|------|----------|------|
| period | 1d, 5d, 1mo, 3mo, 6mo, 1y, 2y, 5y, 10y, ytd, max | ����ʱ�䷶Χ |
| interval | 1m, 2m, 5m, 15m, 30m, 60m, 90m, 1h, 1d, 5d, 1wk, 1mo | K������ |

**ע�⣺** ���Ӽ������ݽ��������730��

---

## ����������ȡ������

```python
# �����Ʊ�����ÿո�ָ�
tickers = "AAPL MSFT GOOGL AMZN"
df = yf.download(tickers, period="1mo")

# Ҳ�������б�
df = yf.download(["AAPL", "MSFT", "GOOGL"], period="1mo")

# ʹ�� group_by ����Ʊ����
df = yf.download(["AAPL", "MSFT"], period="1mo", group_by="ticker")

```

---

## �ġ���Ʊ������Ϣ (info)

```python
ticker = yf.Ticker("AAPL")
info = ticker.info

# �����ֶ�
```
print(info['longName'])
print(info['currentPrice'])
print(info['marketCap'])
print(info['trailingPE'])
print(info['fiftyTwoWeekHigh'])

 
```

**info 主要字段：**
- basicInfo: shortName, longName, symbol, sector, industry
- priceInfo: currentPrice, previousClose, open, dayHigh, dayLow, volume
- valuation: marketCap, peRatio, pegRatio, psRatio, pbRatio
- dividend: dividendRate, dividendYield, exDividendDate
- financial: totalRevenue, netIncomeToCommon, profitMargins
- institutional: institutionalOwnership, heldPercentInsiders
-52Week: fiftyTwoWeekHigh, fiftyTwoWeekLow, fiftyTwoWeekChange

---

## 五、财务报表数据

```python
ticker = yf.Ticker("AAPL")

# 利润表 (Income Statement)
income_stmt = ticker.financials
print(income_stmt)

# 资产负债表 (Balance Sheet)
balance_sheet = ticker.balance_sheet

# 现金流量表 (Cash Flow)
cashflow = ticker.cashflow
```

---

## 六、期权数据

```python
ticker = yf.Ticker("AAPL")

# 获取所有到期日的期权链
options = ticker.options

# 获取指定到期日的期权链
opt_chain = ticker.option_chain("2025-05-16")
print(options)

# Get option chain for specific expiry
opt_chain = ticker.option_chain(date)

# Call options
calls = opt_chain.calls
# Put options
puts = opt_chain.puts
```

**期权链字段：**
- contractSymbol, lastTradeDate, strike, lastPrice, bid, ask, volume, openInterest, impliedVolatility

---

## 七、分红和拆股数据

```python
ticker = yf.Ticker("AAPL")

# 分红历史
dividends = ticker.dividends
print(dividends.tail(10))

# 拆股历史
splits = ticker.splits
print(splits)

```

---

## 八、机构持仓数据

```python
ticker = yf.Ticker("AAPL")

# 机构持仓 (顶级持仓者)
institutional = ticker.institutional_holders
print(institutional.head())

# 内部人持仓
insider = ticker.insider_holders

```

---

## 九、分析师预测和建议

```python
ticker = yf.Ticker("AAPL")

# 分析师推荐历史
recommendations = ticker.recommendations
print(recommendations.tail())

# 分析师目标价
target = ticker.target_price

# 盈利预测
earnings = ticker.earnings

```

---

## 十、股票走势和趋势

```python
ticker = yf.Ticker("AAPL")

# 可持续投资 (社会责任投资数据)
sustainability = ticker.sustainability
print(sustainability)

# 收益日历 (财报日期)
calendar = ticker.calendar

```

---

## 十一、新闻和事件

```python
ticker = yf.Ticker("AAPL")

# 获取新闻
news = ticker.news
for item in news:

```

---

## 十二、持仓成本和主要股东

```python
ticker = yf.Ticker("AAPL")

for item in news:
    print(item[title], item[pubDate])

---

## 十二、持仓成本和主要股东

```python
ticker = yf.Ticker("AAPL")

# 获取major_holders
holders = ticker.major_holders
print(holders)

```

---

## 十三、港股和A股数据

```python
import yfinance as yf

# 港股: 代码后面加.HK
hk_ticker = yf.Ticker("0700.HK")  # 腾讯控股
df = hk_ticker.history(start="2024-01-01")

# A股: 代码后面加.SS(上海) 或 .SZ(深圳)
cn_ticker = yf.Ticker("600000.SS")  # 浦发银行(上海)
df = cn_ticker.history(start="2024-01-01")

```

---

## 十四、期货数据

```python
import yfinance as yf

# 黄金期货
gc = yf.Ticker("GC=F")  # Gold Futures
gold = gc.history(period="1mo")

# 原油期货
cl = yf.Ticker("CL=F")  # Crude Oil Futures
oil = cl.history(period="1mo")
# 其他期货代码
# SI=F 白银, NG=F 天然气, ES=F 标普500期货, NQ=F 纳斯达克期货
```

---
oil = cl.history(period="1mo")

# Other futures codes
# SI=F Silver, NG=F Natural Gas, ES=F S&P500 Futures, NQ=F Nasdaq Futures

```

---

## 十五、外汇数据

```python
import yfinance as yf

# 美元/欧元
eurusd = yf.Ticker("EURUSD=X")
df = eurusd.history(period="1mo")

# 常用外汇代码
# USDJPY=X 美元/日元, GBPUSD=X 英镑/美元, USDCHF=X 美元/瑞郎
# USDCNH=X 美元/离岸人民币, AUDUSD=X 澳元/美元
```

---

## 十六、加密货币数据

```python
import yfinance as yf

# 比特币 (BTC/USD)
btc = yf.Ticker("BTC-USD")
btc_data = btc.history(period="1mo")

# 以太坊 (ETH/USD)
eth = yf.Ticker("ETH-USD")
eth_data = eth.history(period="1mo")

# Other crypto: ETH-USD, BNB-USD, SOL-USD, XRP-USD

```

---

## 十七、常用代码对照表

| 市场 | 代码格式 | 示例 |
|-----|----------|-----|
| 美股 | 直接代码 | AAPL, MSFT, GOOGL |
| 港股 | .HK后缀 | 0700.HK, 9988.HK |
| A股上海 | .SS后缀 | 600000.SS, 688001.SS |
| A股深圳 | .SZ后缀 | 000001.SZ, 300001.SZ |
| 期货 | =F后缀 | GC=F, CL=F, ES=F |
| 外汇 | XXXYYY=X | EURUSD=X, USDJPY=X |
| 加密货币 | -USD后缀 | BTC-USD, ETH-USD |

---

## 十八、Ticker对象完整属性

| 属性 | 说明 |
|-----|-----|
| .info | 基本信息字典 |
| .history() | 价格历史数据 |
| .dividends | 分红历史 |
| .splits | 拆股历史 |
| .financials | 利润表 |
| .balance_sheet | 资产负债表 |
| .cashflow | 现金流量表 |
| .options | 所有期权到期日 |
| .option_chain() | 期权链数据 |
| .institutional_holders | 机构持仓 |
| .insider_holders | 内部人持仓 |
| .major_holders | 主要股东 |
| .recommendations | 分析师推荐 |
| .earnings | 盈利预测 |
| .sustainability | ESG/社会责任数据 |
| .calendar | 收益日历/财报日期 |
| .news | 新闻 |
| .target_price | 分析师目标价 |

---

## 十九、实用代码示例

### 19.1 下载并保存数据到本地

```python
import yfinance as yf
import pandas as pd

def download_stock_data(symbol, start, end, save_path):
    ticker = yf.Ticker(symbol)
    df = ticker.history(start=start, end=end)
    df.to_csv(save_path)
    print(f"Saved {symbol} data to {save_path}")

# Usage
download_stock_data("AAPL", "2020-01-01", "2025-12-31", "aapl_data.csv")

```

### 19.2 批量下载多只股票

```python
import yfinance as yf

symbols = ["AAPL", "MSFT", "GOOGL", "AMZN", "META"]
data = yf.download(symbols, period="1y")

# Extract close prices
closes = data[Close] if Close in data.columns else data[Close.1]
print(closes.head())

```

### 19.3 计算股票收益率

```python
import yfinance as yf
import pandas as pd

ticker = yf.Ticker("AAPL")
df = ticker.history(period="1y")

# Daily returns
df[Return] = df[Close].pct_change()

# Cumulative returns
df[CumReturn] = (1 + df[Return]).cumprod() - 1

print(df.tail())

```

### 19.4 获取期权链并计算Greeks

```python
import yfinance as yf

ticker = yf.Ticker("AAPL")
opt = ticker.option_chain(ticker.options[0])

# Calls and Puts DataFrames
calls = opt.calls
puts = opt.puts

print(f"Available strikes: {len(calls)} call options")
```

---

## 二十、常见问题与解决方案

### Q1: 数据获取失败或超时？
```python
# 增加重试次数
df = yf.download("AAPL", period="1mo", retry_count=3)
```

### Q2: 如何获取前复权数据？
```python
# yfinance默认返回的就是前复权-adjusted数据
# Adj Close列即为复权后的收盘价
df = ticker.history(period="5y", auto_adjust=False)
```

### Q3: 如何设置代理？
```python
import os
os.environ["HTTP_PROXY"] = "http://proxy.example.com:8080"
os.environ["HTTPS_PROXY"] = "http://proxy.example.com:8080"
```

### Q4: 如何获取特定日期的数据？
```python
# 使用start和end参数指定日期范围
df = yf.download("AAPL", start="2024-01-01", end="2024-12-31")
```

---

## 二十一、相关资源

- GitHub: github.com/ranaroussi/yfinance
- Documentation: pypi.org/project/yfinance
- Yahoo Finance: finance.yahoo.com

---

## 二十二、完整封装类

```python
import yfinance as yf
import pandas as pd

class YahooFinanceAPI:
    """Yahoo Finance API封装类"""

    def __init__(self, symbol):
        self.ticker = yf.Ticker(symbol)
        self.symbol = symbol

    def get_history(self, start=None, end=None, period=None, interval="1d"):
        """获取价格历史"""
        return self.ticker.history(start=start, end=end, period=period, interval=interval)

    def get_info(self):
        """获取股票基本信息"""
        return self.ticker.info

    def get_dividends(self):
        """获取分红历史"""
        return self.ticker.dividends

    def get_financials(self):
        """获取财务报表"""
        return {
            "income": self.ticker.financials,
            "balance_sheet": self.ticker.balance_sheet,
            "cashflow": self.ticker.cashflow
        }

    def get_options(self, date=None):
        """获取期权链"""
        if date is None:
            date = self.ticker.options[0]
        return self.ticker.option_chain(date)


# Usage example
api = YahooFinanceAPI("AAPL")
df = api.get_history(period="1y")
info = api.get_info()
```

---

*文档结束*
