# 数据源与API接口

> **更新**: 2026-04-26 | **收集范围**: 股票/期货/期权/另类数据/API接口/数据服务商

---

## 一、数据类型总览

### 1.1 金融数据类型分类

```
┌──────────────────────────────────────────────────────────────────┐
│                        金融数据类型                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  市场数据        │  │  基础数据        │  │  另类数据       │  │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤  │
│  │ Tick数据        │  │ 财务数据         │  │ 卫星图像        │  │
│  │ K线数据         │  │ 股东信息         │  │ 舆情数据        │  │
│  │ 订单簿          │  │ 行业分类         │  │ ESG评分         │  │
│  │ 交易量          │  │ 公司公告         │  │ 供应链          │  │
│  │ 资金流向        │  │ 分析师预测       │  │ 电商数据        │  │
│  │ 融资融券        │  │ 估值指标         │  │ 专利数据        │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  宏观数据        │  │ 另类市场数据     │  │ 衍生数据        │  │
│  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤  │
│  │ GDP/CPI/PPI    │  │ 加密货币         │  │ 因子数据        │  │
│  │ 利率/汇率       │  │ 大宗商品         │  │ 风险因子        │  │
│  │ 贸易数据        │  │ 房地产           │  │ 预期收益        │  │
│  │ 财政数据        │  │ 艺术品           │  │ 波动率曲面      │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 1.2 数据频率与延迟

| 数据类型 | 更新频率 | 延迟 | 存储需求 |
|----------|----------|------|----------|
| 日线数据 | 日频 | T+1 | 低 |
| 分钟数据 | 分钟频 | ~5min | 中 |
| Tick数据 | 实时 | <1s | 高 |
| 资金流 | 日频 | T+1 | 低 |
| 订单簿 | 实时 | <100ms | 极高 |
| 新闻/舆情 | 实时 | <1min | 中 |

---

## 二、A股数据源

### 2.1 Tushare (免费/Pro)

```python
"""
Tushare数据接口

官方网址: tushare.pro
免费版token获取: tushare.pro/register
"""

import tushare as ts
import pandas as pd

class TushareAPI:
    """
    Tushare数据接口封装
    
    需要注册获取token
    """
    
    def __init__(self, token='YOUR_TOKEN'):
        self.pro = ts.pro_api(token)
        
    def get_daily(self, ts_code, start_date, end_date):
        """
        获取日线数据
        
        参数:
            ts_code: 股票代码 (如: 000001.SZ)
            start_date: 开始日期 (YYYYMMDD)
            end_date: 结束日期
        """
        df = self.pro.daily(
            ts_code=ts_code,
            start_date=start_date,
            end_date=end_date
        )
        return df
    
    def get_basket_constituent(self, index_code):
        """
        获取指数成分股
        
        示例: 上证50 = '000016.SH'
        """
        df = self.pro.index_weight(
            index_code=index_code,
            trade_date=pd.Timestamp.now().strftime('%Y%m%d')
        )
        return df
    
    def get_financial_data(self, ts_code, ann_date):
        """
        获取财务报表
        
        包含: 资产负债表、利润表、现金流量表
        """
        # 利润表
        df = self.pro.fina_indicator(
            ts_code=ts_code,
            start_date=ann_date
        )
        return df
    
    def get_money_flow(self, trade_date):
        """
        获取资金流向
        
        包含: 个股资金流向、板块资金流
        """
        df = self.pro.moneyflow_hsgt(
            trade_date=trade_date
        )
        return df
    
    def get_limit_list(self, trade_date):
        """
        获取涨停板数据
        
        用于事件驱动策略
        """
        df = self.pro.limit_list_d(
            trade_date=trade_date
        )
        return df
    
    def get_stk_rewards(self, ts_code):
        """
        获取个股融资融券
        
        包含: 融资余额、融券余额
        """
        df = self.pro.ssq_basic(
            ts_code=ts_code
        )
        return df

# 使用示例
def main():
    token = 'YOUR_TOKEN_HERE'
    api = TushareAPI(token)
    
    # 获取日线数据
    df = api.get_daily('000001.SZ', '20230101', '20260126')
    print(df.head())
    
    # 计算收益率
    df['return'] = df['close'].pct_change()
    
    return df

# Tushare Pro高级功能
class TusharePro:
    """
    Tushare Pro API
    
    需要付费订阅更多数据
    """
    
    def __init__(self, token):
        self.pro = ts.pro_api(token)
        
    def get_tick_data(self, ts_code, date):
        """
        获取分笔数据
        
        Pro权限
        """
        df = self.pro.tick(
            ts_code=ts_code,
            trade_date=date
        )
        return df
    
    def get_minute_data(self, ts_code, freq='1min'):
        """
        获取分钟数据
        
        freq: 1min, 5min, 15min, 30min, 60min
        """
        df = self.pro.security_list(trade_date='20230101')
        return df
    
    def get_realtime_quote(self, ts_codes):
        """
        获取实时行情
        
        订阅模式
        """
        # 需要websocket订阅
        pass
```

### 2.2 AKShare (免费)

```python
"""
AKShare - 纯免费Python金融数据接口

官方文档: akshare.akfamily.net
GitHub: github.com/akfamily/akshare
"""

import akshare as ak
import pandas as pd

class AKShareAPI:
    """
    AKShare数据接口封装
    
    无需注册，完全免费
    """
    
    @staticmethod
    def get_stock_zh_a_hist(symbol='000001', period='daily', 
                            start_date='20230101', end_date='20260126'):
        """
        获取A股历史数据
        
        参数:
            symbol: 股票代码
            period: 日线/周线/月线
            start_date/end_date: 日期范围
        """
        df = ak.stock_zh_a_hist(
            symbol=symbol,
            period=period,
            start_date=start_date,
            end_date=end_date
        )
        return df
    
    @staticmethod
    def get_realtime_quote(symbol='000001'):
        """
        获取实时行情
        """
        df = ak.stock_zh_a_spot_em()
        return df
    
    @staticmethod
    def get_stock_financial_report(symbol='000001'):
        """
        获取财务报表
        """
        # 利润表
        income = ak.stock_financial_report_sina_by_report_type(
            symbol=symbol,
            type='1'  # 1=年报, 2=季报
        )
        
        # 资产负债表
        balance = ak.stock_financial_report_sina_by_report_type(
            symbol=symbol,
            type='2'
        )
        
        return income, balance
    
    @staticmethod
    def get_money_flow(symbol='000001'):
        """
        获取资金流向
        """
        df = ak.stock_money_flow_sina()
        return df
    
    @staticmethod
    def get_index_zh_a_hist(index_code='000001'):
        """
        获取指数历史数据
        
        示例: 上证指数 '000001', 沪深300 '000300'
        """
        df = ak.stock_zh_index_daily_em(
            symbol=f"sh{index_code}"
        )
        return df
    
    @staticmethod
    def get_futures_hist(symbol='IF8888'):
        """
        获取期货历史数据
        """
        df = ak.futures_zh_daily_sina(
            symbol=symbol
        )
        return df
    
    @staticmethod
    def get_option_hist():
        """
        获取期权数据
        """
        df = ak.opt_market_detail_sse()
        return df

# 使用示例
def main():
    api = AKShareAPI()
    
    # 获取日线数据
    df = api.get_stock_zh_a_hist('000001')
    print(df.head())
    
    # 获取实时行情
    spot = api.get_realtime_quote()
    print(spot.head())
```

### 2.3 聚宽数据 (免费/付费)

```python
"""
聚宽 JoinQuant 数据接口

官方文档: joinquant.com
"""

import jqdatasdk as jq
from jqdatasdk import opt

class JoinQuantAPI:
    """
    聚宽数据接口
    
    需要注册获取账号
    """
    
    def __init__(self, username, password):
        self.username = username
        self.password = password
        self.authenticate()
        
    def authenticate(self):
        """登录认证"""
        jq.auth(self.username, self.password)
        
    def get_price(self, securities, start_date, end_date, freq='daily'):
        """
        获取价格数据
        
        Args:
            securities: 股票代码列表
            start_date/end_date: 日期范围
            freq: daily/minute/tick
        """
        df = jq.get_price(
            securities=securities,
            start_date=start_date,
            end_date=end_date,
            frequency=freq
        )
        return df
    
    def get_fundamentals(self, security, date=None):
        """
        获取基本面数据
        
        返回: 财务报表+估值指标
        """
        if date:
            q = jq.query(jq.finance.STK_FINANCE_INCOME).filter(
                jq.finance.STK_FINANCE_INCOME.pubDate <= date
            )
        else:
            q = jq.query(jq.finance.STK_FINANCE_INCOME)
            
        df = jq.finance.run_query(q)
        return df
    
    def get_index_stocks(self, index_code):
        """
        获取指数成分股
        """
        stocks = jq.get_index_stocks(index_code)
        return stocks
    
    def get_industry分类(self, date=None):
        """
        获取行业分类
        
        包含: 申万行业、证监会行业
        """
        industries = jq.get_industries(name='sw_l1')
        return industries
```

### 2.4 米筐RiceQuant

```python
"""
米筐 RiceQuant 数据接口

官方文档: ricequant.com
"""

from rqdatac import *

class RiceQuantAPI:
    """
    米筐数据接口
    """
    
    def __init__(self, username, password):
        init(username, password)
        
    def get_dailybars(self, order_book_ids, start_date, end_date):
        """
        获取日线数据
        
        order_book_ids: 合约代码列表
        """
        df = dailybars(
            order_book_ids=order_book_ids,
            start_date=start_date,
            end_date=end_date
        )
        return df
    
    def get_minutebars(self, order_book_id, start_date, end_date, freq='1m'):
        """
        获取分钟数据
        
        freq: 1m, 5m, 15m, 30m, 60m
        """
        df = minutebars(
            order_book_id=order_book_id,
            start_date=start_date,
            end_date=end_date,
            frequency=freq
        )
        return df
    
    def get_financials(self, order_book_ids, fields=None):
        """
        获取财务数据
        """
        df = get_financials(
            order_book_ids=order_book_ids,
            fields=fields
        )
        return df
    
    def get_factor(self, factor_name, order_book_ids, start_date, end_date):
        """
        获取因子数据
        
        内置因子: 估值、动量、质量等
        """
        df = get_factor(
            factor=factor_name,
            entities=order_book_ids,
            start_date=start_date,
            end_date=end_date
        )
        return df
```

---

## 三、美股数据源

### 3.1 Yahoo Finance

```python
"""
Yahoo Finance 数据接口

免费数据源
"""

import yfinance as yf
import pandas as pd

class YahooFinanceAPI:
    """
    Yahoo Finance API
    
    完全免费，数据丰富
    """
    
    @staticmethod
    def get_stock_data(symbol, start='2020-01-01', end='2026-01-01'):
        """
        获取股票数据
        
        包含: OHLCV、分红、拆股
        """
        ticker = yf.Ticker(symbol)
        df = ticker.history(start=start, end=end)
        return df
    
    @staticmethod
    def get_options(symbol):
        """
        获取期权链
        """
        ticker = yf.Ticker(symbol)
        opt = ticker.option_chain()
        return opt
    
    @staticmethod
    def get_info(symbol):
        """
        获取股票基本信息
        
        包含: 估值、财务、机构持仓
        """
        ticker = yf.Ticker(symbol)
        info = ticker.info
        return info
    
    @staticmethod
    def get_financials(symbol):
        """
        获取财务报表
        """
        ticker = yf.Ticker(symbol)
        
        income_stmt = ticker.financials
        balance_sheet = ticker.balance_sheet
        cash_flow = ticker.cashflow
        
        return income_stmt, balance_sheet, cash_flow
    
    @staticmethod
    def get_batch_quotes(symbols):
        """
        批量获取行情
        
        用于多股票筛选
        """
        data = yf.download(
            tickers=symbols,
            period='5d',
            interval='1d',
            group_by='ticker'
        )
        return data
    
    @staticmethod
    def get_earnings_calendar(start_date, end_date):
        """
        获取财报日历
        """
        # 使用其他数据源
        pass

# 使用示例
def main():
    api = YahooFinanceAPI()
    
    # 获取苹果数据
    aapl = api.get_stock_data('AAPL')
    print(aapl.head())
    
    # 计算收益率
    aapl['return'] = aapl['Close'].pct_change()
    
    # 获取期权链
    options = api.get_options('AAPL')
    print(options.calls.head())
```

### 3.2 Alpha Vantage

```python
"""
Alpha Vantage API

免费API: alphavantage.co
每日限制: 25次/分钟, 500次/天
"""

import requests
import pandas as pd

class AlphaVantageAPI:
    """
    Alpha Vantage API
    
    需要申请免费API Key
    """
    
    BASE_URL = 'https://www.alphavantage.co/query'
    
    def __init__(self, api_key):
        self.api_key = api_key
        
    def get_daily(self, symbol, outputsize='compact'):
        """
        获取日线数据
        
        outputsize: compact(最近100天)/full(全量)
        """
        url = self.BASE_URL
        params = {
            'function': 'TIME_SERIES_DAILY_ADJUSTED',
            'symbol': symbol,
            'apikey': self.api_key,
            'outputsize': outputsize
        }
        
        response = requests.get(url, params=params)
        data = response.json()
        
        # 解析数据
        time_series = data.get('Time Series (Daily)', {})
        df = pd.DataFrame.from_dict(time_series, orient='index')
        df.index = pd.to_datetime(df.index)
        df = df.astype(float)
        
        return df
    
    def get_intraday(self, symbol, interval='5min', outputsize='compact'):
        """
        获取分钟数据
        
        interval: 1min, 5min, 15min, 30min, 60min
        """
        params = {
            'function': 'TIME_SERIES_INTRADAY',
            'symbol': symbol,
            'interval': interval,
            'apikey': self.api_key,
            'outputsize': outputsize
        }
        
        response = requests.get(self.BASE_URL, params=params)
        data = response.json()
        
        key = f'Time Series ({interval})'
        time_series = data.get(key, {})
        
        df = pd.DataFrame.from_dict(time_series, orient='index')
        df.index = pd.to_datetime(df.index)
        df = df.astype(float)
        
        return df
    
    def get_forex_daily(self, from_symbol, to_symbol):
        """
        获取外汇数据
        """
        params = {
            'function': 'FX_DAILY',
            'from_symbol': from_symbol,
            'to_symbol': to_symbol,
            'apikey': self.api_key
        }
        
        response = requests.get(self.BASE_URL, params=params)
        data = response.json()
        
        time_series = data.get('Time Series FX (Daily)', {})
        df = pd.DataFrame.from_dict(time_series, orient='index')
        
        return df
    
    def get_sector_performance(self):
        """
        获取行业表现
        """
        params = {
            'function': 'SECTOR',
            'apikey': self.api_key
        }
        
        response = requests.get(self.BASE_URL, params=params)
        return response.json()
    
    def get_crypto_daily(self, symbol, market='USD'):
        """
        获取加密货币数据
        """
        params = {
            'function': 'DIGITAL_CURRENCY_DAILY',
            'symbol': symbol,
            'market': market,
            'apikey': self.api_key
        }
        
        response = requests.get(self.BASE_URL, params=params)
        data = response.json()
        
        time_series = data.get('Time Series (Digital Currency Daily)', {})
        df = pd.DataFrame.from_dict(time_series, orient='index')
        
        return df

# 使用示例
def main():
    api = AlphaVantageAPI('YOUR_API_KEY')
    
    # 获取苹果日线
    df = api.get_daily('AAPL')
    print(df.head())
```

### 3.3 其他美股数据源

```python
"""
其他美股数据源
"""

class OtherUSDataSources:
    """
    其他美股数据源
    """
    
    @staticmethod
    def get_from_finviz(symbol):
        """
        Finviz - 股票筛选
        
        网址: finviz.com
        """
        import finviz
        stock = finviz.Snapshot(symbol)
        return stock.to_dict()
    
    @staticmethod
    def get_from_polygon(symbol, api_key):
        """
        Polygon API
        
        专业级美股数据
        网址: polygon.io
        """
        pass
    
    @staticmethod
    def get_from_nasdaq_api():
        """
        NASDAQ API
        """
        pass
```

---

## 四、期货与期权数据

### 4.1 国内期货数据

```python
"""
国内期货数据接口
"""

class ChinaFuturesAPI:
    """
    国内期货数据
    
    数据源: 中金所/上期所/大商所/郑商所
    """
    
    def __init__(self):
        self.exchanges = {
            'CFFEX': '中金所',
            'SHFE': '上期所',
            'DCE': '大商所',
            'CZCE': '郑商所',
            'INE': '能源中心'
        }
        
    def get_futures_daily(self, symbol, start_date, end_date):
        """
        获取期货日线数据
        
        symbol格式: IF2301 (主力连续用 IF0)
        """
        import akshare as ak
        
        # 使用AKShare
        df = ak.futures_zh_daily_sina(symbol=symbol)
        
        return df
    
    def get_main_contract_mapping(self, exchange='CFFEX'):
        """
        获取主力合约映射
        """
        import akshare as ak
        
        df = ak.futures_main_sina(exchange=exchange)
        return df
    
    def get_futures_holdings(self, date):
        """
        获取持仓排名
        """
        import akshare as ak
        
        df = ak.futures_position_rank(
            trade_date=date
        )
        return df
    
    def get_option_chain(self, underlying, exchange='SSE'):
        """
        获取期权链
        
        支持: 50ETF, 300ETF, 铜期权
        """
        import akshare as ak
        
        if '50' in underlying:
            # 50ETF期权
            df = ak.opt_sina_option_sse(symbol='510050')
        elif '300' in underlying:
            # 300ETF期权
            df = ak.opt_sina_option_sse(symbol='510300')
            
        return df
    
    def calculate_basis(self, futures_price, spot_price):
        """
        计算基差
        """
        basis = futures_price - spot_price
        basis_ratio = basis / spot_price
        
        return basis, basis_ratio

# 商品期货数据
class CommodityFutures:
    """
    商品期货数据
    """
    
    def get_metals_data(self):
        """贵金属数据"""
        pass
    
    def get_energy_data(self):
        """能源数据"""
        pass
    
    def get_agricultural_data(self):
        """农产品数据"""
        pass
```

### 4.2 期权数据

```python
"""
期权数据接口
"""

class OptionsData:
    """
    期权数据处理
    """
    
    def __init__(self):
        self.greeks_calc = GreeksCalculator()
        
    def parse_option_chain(self, chain_df):
        """
        解析期权链数据
        
        返回: 整理后的期权链
        """
        # 提取call和put
        calls = chain_df[chain_df['type'] == 'call']
        puts = chain_df[chain_df['type'] == 'put']
        
        # 按行权价匹配
        merged = pd.merge(
            calls, puts,
            on='strike',
            suffixes=('_call', '_put')
        )
        
        return merged
    
    def calculate_implied_volatility(self, option_price, S, K, T, r, is_call=True):
        """
        计算隐含波动率
        
        使用牛顿法迭代
        """
        from scipy.stats import norm
        
        def black_scholes_price(S, K, T, r, sigma, is_call=True):
            d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
            d2 = d1 - sigma*np.sqrt(T)
            
            if is_call:
                price = S*norm.cdf(d1) - K*np.exp(-r*T)*norm.cdf(d2)
            else:
                price = K*np.exp(-r*T)*norm.cdf(-d2) - S*norm.cdf(-d1)
                
            return price
        
        def vega(S, K, T, r, sigma):
            d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
            return S * np.sqrt(T) * norm.pdf(d1)
        
        # 牛顿法
        sigma = 0.3  # 初始值
        for _ in range(100):
            price = black_scholes_price(S, K, T, r, sigma, is_call)
            vega_val = vega(S, K, T, r, sigma)
            
            if vega_val < 1e-10:
                break
                
            diff = option_price - price
            sigma += diff / vega_val
            
        return sigma
    
    def build_volatility_smile(self, option_chain):
        """
        构建波动率微笑
        
        用于期权定价和策略开发
        """
        strikes = option_chain['strike']
        ivs = []
        
        for _, row in option_chain.iterrows():
            iv = self.calculate_implied_volatility(
                row['price'], row['S'], row['strike'],
                row['T'], row['r'], row['is_call']
            )
            ivs.append(iv)
            
        return pd.DataFrame({'strike': strikes, 'iv': ivs})

class GreeksCalculator:
    """
    Greeks计算器
    
    用于期权风险管理和定价
    """
    
    @staticmethod
    def black_scholes_greeks(S, K, T, r, sigma, is_call=True):
        """
        计算Black-Scholes Greeks
        
        返回: Delta, Gamma, Theta, Vega, Rho
        """
        from scipy.stats import norm
        
        d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
        d2 = d1 - sigma*np.sqrt(T)
        
        # Delta
        if is_call:
            delta = norm.cdf(d1)
        else:
            delta = norm.cdf(d1) - 1
            
        # Gamma (call和put相同)
        gamma = norm.pdf(d1) / (S * sigma * np.sqrt(T))
        
        # Theta
        term1 = -S * norm.pdf(d1) * sigma / (2 * np.sqrt(T))
        if is_call:
            term2 = -r * K * np.exp(-r*T) * norm.cdf(d2)
            theta = (term1 + term2) / 365
        else:
            term2 = r * K * np.exp(-r*T) * norm.cdf(-d2)
            theta = (term1 + term2) / 365
            
        # Vega (call和put相同)
        vega = S * norm.pdf(d1) * np.sqrt(T) / 100
        
        # Rho
        if is_call:
            rho = K * T * np.exp(-r*T) * norm.cdf(d2) / 100
        else:
            rho = -K * T * np.exp(-r*T) * norm.cdf(-d2) / 100
            
        return {
            'delta': delta,
            'gamma': gamma,
            'theta': theta,
            'vega': vega,
            'rho': rho
        }
    
    @staticmethod
    def portfolio_greeks(positions):
        """
        计算组合Greeks
        
        positions: List of {symbol, quantity,greeks}
        """
        total_greeks = {
            'delta': 0,
            'gamma': 0,
            'theta': 0,
            'vega': 0,
            'rho': 0
        }
        
        for pos in positions:
            for g in total_greeks:
                total_greeks[g] += pos['quantity'] * pos['greeks'][g]
                
        return total_greeks
```

---

## 五、另类数据源

### 5.1 卫星数据

```python
"""
卫星图像数据源

用于测量经济活动的另类数据
"""

class SatelliteData:
    """
    卫星图像数据
    
    常用数据源:
    - RS Metrics
    - Planet Labs
    - ShadowLight
    - Spaceflight Industries
    """
    
    def estimate_gdp_via_lights(self, region):
        """
        通过夜间灯光估算GDP
        
        灯光强度与经济活动高度相关
        """
        # 归一化灯光指数
        ntl_index = self._calculate_ntl(region)
        
        # GDP相关性模型
        gdp_estimate = self._model_gdp(ntl_index)
        
        return gdp_estimate
    
    def count_vehicles(self, parking_lot):
        """
        计算停车场车辆数
        
        用于零售销售预测
        """
        # 需要计算机视觉处理
        pass
    
    def estimate_ship_traffic(self, port):
        """
        估算港口船只数量
        
        用于贸易数据
        """
        pass

class SatelliteProviders:
    """
    卫星数据提供商
    """
    
    PROVIDERS = {
        'Planet Labs': {
            'url': 'planet.com',
            'data_type': '每日卫星图像',
            'cost': '订阅制'
        },
        'Maxar': {
            'url': 'maxar.com',
            'data_type': '高分辨率图像',
            'cost': '按任务'
        },
        'Iceye': {
            'url': 'iceye.com',
            'data_type': 'SAR图像(穿透云层)',
            'cost': '按任务'
        }
    }
```

### 5.2 舆情数据

```python
"""
舆情数据源

用于情绪分析和事件驱动策略
"""

class SentimentData:
    """
    舆情数据接口
    """
    
    def __init__(self):
        self.sentiment_models = {}
        
    def get_news_sentiment(self, symbol, start_date, end_date):
        """
        获取新闻情绪数据
        
        需要:
        - 新闻API (NewsAPI, GDELT, RavenPack)
        - 情绪分析模型
        """
        pass
    
    def get_social_media_sentiment(self, keyword):
        """
        获取社交媒体情绪
        
        数据源:
        - Twitter API
        - StockTwits
        - 微博API
        - 东财股吧
        """
        pass
    
    def calculate_sentiment_score(self, text):
        """
        计算文本情绪得分
        
        方法:
        1. 词典方法 (Loughran-McDonald)
        2. ML方法 (FinBERT)
        3. 混合方法
        """
        # 使用预训练模型
        from transformers import pipeline
        
        classifier = pipeline(
            'sentiment-analysis',
            model='Prosailor/FinBERT'
        )
        
        result = classifier(text)
        
        return result

class SocialMediaData:
    """
    社交媒体数据
    """
    
    @staticmethod
    def get_twitter_sentiment(keyword, since, until):
        """
        Twitter情绪数据
        
        需要Twitter API访问权限
        """
        pass
    
    @staticmethod
    def get_stocktwits_posts(symbol):
        """
        StockTwits帖子数据
        """
        import requests
        
        url = f'https://api.stocktwits.com/api/2/streams/symbol/{symbol}.json'
        response = requests.get(url)
        
        return response.json()
    
    @staticmethod
    def get_eastmoney_posts(stock_code):
        """
        东财股吧帖子
        
        免费数据
        """
        import akshare as ak
        
        df = ak.stock_board_industry_em()
        
        return df
```

### 5.3 ESG数据

```python
"""
ESG数据源

用于ESG投资和风险评估
"""

class ESGData:
    """
    ESG数据接口
    """
    
    PROVIDERS = {
        'MSCI': {
            'url': 'msci.com',
            'coverage': '全球10000+公司',
            'update': '日频'
        },
        'Sustainalytics': {
            'url': 'sustainalytics.com',
            'coverage': '全球12000+公司',
            'update': '日频'
        },
        'Refinitiv': {
            'url': 'refinitiv.com',
            'coverage': '全球7000+公司',
            'update': '日频'
        },
        'Bloomberg': {
            'url': 'bloomberg.com',
            'coverage': '全球12000+公司',
            'update': '日频'
        }
    }
    
    @staticmethod
    def get_esg_scores(symbol):
        """
        获取ESG评分
        """
        # 整合多个数据源
        pass
    
    @staticmethod
    def calculate_esg_portfolio_score(holdings):
        """
        计算组合ESG得分
        """
        total_esg = 0
        total_weight = 0
        
        for holding in holdings:
            weight = holding['weight']
            esg = holding['esg_score']
            
            total_esg += weight * esg
            total_weight += weight
            
        return total_esg / total_weight
```

### 5.4 其他另类数据

```python
"""
其他另类数据

包含: 供应链、专利、电商等
"""

class AlternativeData:
    """
    另类数据整合
    """
    
    # 供应链数据
    SUPPLY_CHAIN = {
        'Quandl': '供应链情绪',
        'Panjiva': '进出口数据',
        'ImportGenius': '海关数据'
    }
    
    # 专利数据
    PATENT = {
        'PatSnap': '全球专利数据库',
        'Derwent': '专利引用分析',
        'Google Patents': '免费专利检索'
    }
    
    # 电商数据
    ECOMMERCE = {
        'SimilarWeb': '网站流量分析',
        'Semrush': '营销数据分析',
        'Helium10': '亚马逊产品数据'
    }
    
    # 信用卡数据
    CREDIT_CARD = {
        'MasterCard Economics': '消费数据',
        'JP Morgan Chase': '消费数据',
        'Facteus': '匿名消费数据'
    }
```

---

## 六、宏观数据源

### 6.1 中国宏观数据

```python
"""
中国宏观数据接口
"""

class ChinaMacroData:
    """
    中国宏观数据
    """
    
    def __init__(self):
        self.data_sources = {
            'NBS': '国家统计局',
            'PBOC': '中国人民银行',
            'SAFE': '外管局',
            'MOF': '财政部'
        }
        
    def get_gdp_data(self):
        """GDP数据"""
        import akshare as ak
        
        df = ak.macro_china_gdp()
        return df
    
    def get_ppi_cpi_data(self):
        """PPI和CPI数据"""
        import akshare as ak
        
        ppi = ak.macro_china_ppi()
        cpi = ak.macro_china_cpi()
        
        return ppi, cpi
    
    def get_fx_data(self):
        """外汇储备和汇率"""
        import akshare as ak
        
        fx_reserve = ak.macro_china_fx_reserves()
        usd_cny = ak.currency_boc_sina()
        
        return fx_reserve, usd_cny
    
    def get_money_supply(self):
        """货币供应量 M0/M1/M2"""
        import akshare as ak
        
        df = ak.macro_china_money_supply()
        return df
    
    def get_interest_rates(self):
        """利率数据"""
        import akshare as ak
        
        lpr = ak.lpr_sina()
        repo = ak.macro_china_repo_rate()
        
        return lpr, repo
```

### 6.2 国际宏观数据

```python
"""
国际宏观数据源
"""

class GlobalMacroData:
    """
    全球宏观数据
    """
    
    def get_fred_data(self, series_id):
        """
        从FRED获取美联储数据
        
        网址: fred.stlouisfed.org
        """
        import pandas_datareader as pdr
        
        df = pdr.get_data_fred(series_id, start='2010-01-01')
        return df
    
    def get_world_bank_data(self, indicator, country='CN'):
        """
        世界银行数据
        """
        import wbdata
        
        data = wbdata.get_dataframe({indicator: indicator}, country=country)
        return data
    
    def get_trading_economics_data(self, indicator):
        """
        Trading Economics数据
        
        需要API Key
        """
        pass
    
    def get_bloomberg_macro(self, tickers):
        """
        Bloomberg宏观数据
        
        需要Bloomberg终端
        """
        pass

# 常用FRED数据序列
FRED_SERIES = {
    'GDP': 'GDP',
    'CPI': 'CPIAUCSL',
    'PPI': 'PPIACO',
    'Unemployment': 'UNRATE',
    'Fed Rate': 'DFF',
    '10Y Yield': 'DGS10',
    'VIX': 'VIXCLS',
    'Initial Claims': 'ICSA',
    'Retail Sales': 'RRSFS',
    'ISM PMI': 'ISMINDPMI',
    'Consumer Confidence': 'CONSCONF',
    'Housing Starts': 'HOUST',
    'Existing Home Sales': 'EXHOSLUSM495S'
}
```

---

## 七、数据存储与处理

### 7.1 数据存储架构

```python
"""
金融数据存储架构
"""

import pandas as pd
import sqlite3
from pathlib import Path

class DataStorage:
    """
    数据存储管理器
    
    支持: SQLite/Parquet/HDF5
    """
    
    def __init__(self, db_path='quant_data.db'):
        self.db_path = db_path
        self.conn = sqlite3.connect(db_path)
        
    def save_to_sql(self, df, table_name, if_exists='append'):
        """
        保存到SQLite
        """
        df.to_sql(table_name, self.conn, if_exists=if_exists, index=True)
        
    def read_from_sql(self, query):
        """
        从SQL读取
        """
        df = pd.read_sql(query, self.conn)
        return df
    
    def save_to_parquet(self, df, path):
        """
        保存到Parquet
        
        优点: 压缩率高, 支持嵌套结构
        """
        df.to_parquet(path, compression='snappy')
        
    def save_to_hdf5(self, df, key):
        """
        保存到HDF5
        
        适合存储大量时间序列
        """
        df.to_hdf(key, key)
        
    def read_hdf5(self, key):
        """读取HDF5"""
        return pd.read_hdf(key)

class DataPipeline:
    """
    数据处理流水线
    """
    
    def __init__(self):
        self.transforms = []
        
    def add_transform(self, func):
        """添加数据转换函数"""
        self.transforms.append(func)
        
    def process(self, df):
        """处理数据"""
        for transform in self.transforms:
            df = transform(df)
        return df
    
    @staticmethod
    def clean_price_data(df):
        """清洗价格数据"""
        # 去除异常值
        df = df.replace([np.inf, -np.inf], np.nan)
        df = df.fillna(method='ffill')
        
        # 去除停牌数据
        df = df[df['volume'] > 0]
        
        return df
    
    @staticmethod
    def align_dates(dfs):
        """对齐多个数据源的日期"""
        # 取所有日期的交集
        common_dates = dfs[0].index
        for df in dfs[1:]:
            common_dates = common_dates.intersection(df.index)
            
        return [df.loc[common_dates] for df in dfs]
```

### 7.2 数据质量检查

```python
"""
数据质量检查模块
"""

class DataQuality:
    """
    数据质量检查
    """
    
    @staticmethod
    def check_missing_values(df):
        """检查缺失值"""
        missing = df.isnull().sum()
        missing_pct = missing / len(df) * 100
        
        return pd.DataFrame({
            'missing_count': missing,
            'missing_pct': missing_pct
        })
    
    @staticmethod
    def check_outliers(df, n_std=5):
        """检查异常值"""
        outliers = {}
        
        for col in df.select_dtypes(include=[np.number]).columns:
            mean = df[col].mean()
            std = df[col].std()
            
            outlier_mask = np.abs(df[col] - mean) > n_std * std
            outliers[col] = outlier_mask.sum()
            
        return outliers
    
    @staticmethod
    def check_continuity(df, freq='D'):
        """检查数据连续性"""
        expected_dates = pd.date_range(
            df.index.min(), df.index.max(), freq=freq
        )
        
        missing_dates = expected_dates.difference(df.index)
        
        return missing_dates
    
    @staticmethod
    def check_price_consistency(df):
        """检查价格逻辑一致性"""
        # high >= low
        high_low_check = (df['high'] >= df['low']).all()
        
        # high >= close/open
        high_check = ((df['high'] >= df['close']) & 
                     (df['high'] >= df['open'])).all()
        
        # low <= close/open
        low_check = ((df['low'] <= df['close']) & 
                    (df['low'] <= df['open'])).all()
        
        return {
            'high_low_valid': high_low_check,
            'high_valid': high_check,
            'low_valid': low_check
        }
    
    @staticmethod
    def generate_quality_report(df):
        """生成数据质量报告"""
        report = {
            'shape': df.shape,
            'date_range': (df.index.min(), df.index.max()),
            'missing': DataQuality.check_missing_values(df),
            'outliers': DataQuality.check_outliers(df),
            'continuity': DataQuality.check_continuity(df),
            'price_consistency': DataQuality.check_price_consistency(df)
        }
        
        return report
```

---

## 八、关键资源汇总

### 8.1 数据源一览

| 数据源 | 类型 | 覆盖范围 | 费用 |
|--------|------|----------|------|
| Tushare | 股票/期货/期权 | A股/港股 | 免费/Pro |
| AKShare | 股票/期货/宏观 | A股/港股 | 免费 |
| 聚宽 | 股票/期货/因子 | A股/期货 | 免费/付费 |
| 米筐 | 股票/期货 | A股/期货 | 免费/付费 |
| Yahoo Finance | 股票/期货/外汇 | 美股/全球 | 免费 |
| Alpha Vantage | 股票/外汇/加密 | 美股/外汇 | 免费/付费 |
| FRED | 宏观数据 | 美国宏观 | 免费 |
| Wind | 全品种 | 全球全品种 | 昂贵 |
| Bloomberg | 全品种 | 全球全品种 | 昂贵 |

### 8.2 API对比

| API | 易用性 | 数据质量 | 覆盖范围 | 费用 |
|-----|--------|----------|----------|------|
| Tushare | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | A股为主 | 免费 |
| AKShare | ⭐⭐⭐⭐ | ⭐⭐⭐ | A股为主 | 免费 |
| 聚宽 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | A股为主 | 中等 |
| Yahoo Finance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 美股为主 | 免费 |
| Alpha Vantage | ⭐⭐⭐ | ⭐⭐⭐⭐ | 美股为主 | 免费 |
| Polygon | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 美股为主 | 付费 |

### 8.3 数据下载命令

```bash
# 安装数据获取库
pip install akshare yfinance tushare pandas_datareader

# 安装数据处理库
pip install pyarrow fastparquet hdf5plugin

# 安装数据库
pip install sqlalchemy duckdb

# 安装可视化
pip install matplotlib seaborn plotly
```

---

## 九、最佳实践

### 9.1 数据获取策略

1. **分层获取**: 免费库优先，付费库补充
2. **本地缓存**: 常用数据本地存储，减少API调用
3. **增量更新**: 定期增量更新，避免全量拉取
4. **异常处理**: 网络不稳定时重试机制
5. **合规使用**: 遵守各平台使用条款

### 9.2 数据存储策略

```python
# 推荐存储格式
STORAGE_FORMAT = {
    'tick数据': 'Parquet (列压缩)',
    '日线数据': 'Parquet (按股票分区)',
    '因子数据': 'HDF5 (快速读写)',
    '元数据': 'SQLite (结构化查询)'
}
```

### 9.3 数据质量监控

```python
# 监控指标
MONITORING_RULES = {
    'price_zero': 'close > 0',
    'volume_nonneg': 'volume >= 0',
    'return_reasonable': '|return| < 0.20',  # 日涨跌不超过20%
    'high_low_valid': 'high >= low',
    'no_future_dates': 'date <= today'
}
```

---

*文档持续更新中*