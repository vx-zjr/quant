# AKShare 数据接口详解

## 一、概述

**AKShare** 是基于 Python 的开源财经数据接口库，旨在为量化投资者和金融研究者提供简洁、高效的数据获取方案。AKShare 取 开源（Open Source）和 量化（Quantitative）之意，由 Albert King 和 Yaojie Zhang 等开发者维护。

### 官方资源
- GitHub：https://github.com/akfamily/akshare
- 文档：https://akshare.akfamily.xyz/
- PyPI：https://pypi.org/project/akshare/


### 核心特点

| 特点 | 说明 |
|------|------|
| 简单易用 | 一行代码即可获取数据 |
| 数据丰富 | 覆盖股票、期货、期权、债券、基金、指数、宏观等多品类 |
| 持续更新 | 活跃的社区维护，频繁的功能迭代 |
| 完全开源 | MIT 开源协议，可自由使用与二次开发 |
| 配套工具 | 提供 HTTP API（AKTools）和 Docker 镜像 |

### 适用场景
- 量化研究与策略回测
- 金融数据分析与可视化
- 自动化交易系统开发
- 学术研究与教学演示

---

## 二、安装指南

### 环境要求
- **Python**：64 位 Python 3.9 及以上版本
- **操作系统**：Windows / Linux / macOS


### 安装命令

**标准安装：**
```
pip install akshare --upgrade
```

**国内镜像安装（推荐国内用户）：**
```
pip install akshare -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com --upgrade
```

### Docker 部署
```
docker pull registry.cn-shanghai.aliyuncs.com/akfamily/aktools:jupyter
docker run -it registry.cn-shanghai.aliyuncs.com/akfamily/aktools:jupyter python
```

---

## 三、快速入门

### 基础示例：获取 A 股历史行情
```python
import akshare as ak

# 获取平安银行（000001）历史日线数据
stock_zh_a_hist_df = ak.stock_zh_a_hist(
    symbol="000001",
    period="daily",
    start_date="20170301",
    end_date="20231022",
    adjust=""
)
print(stock_zh_a_hist_df)
```

### 绘制 K 线图示例
```python
import akshare as ak
import mplfinance as mpf

# 获取苹果公司美股日线数据
stock_us_daily_df = ak.stock_us_daily(symbol="AAPL", adjust="qfq")
stock_us_daily_df = stock_us_daily_df.set_index(["date"])
stock_us_daily_df = stock_us_daily_df["2020-04-01": "2020-04-29"]

# 绘制蜡烛图
mpf.plot(stock_us_daily_df, type="candle", mav=(3, 6, 9), volume=True, show_nontrading=False)
```

---

## 四、数据分类详解

### 1. 股票市场（Stock）

#### A 股市场
| 功能函数 | 说明 |
|----------|------|
| stock_zh_a_hist | A 股历史行情 |
| stock_zh_a_spot_em | A 股实时行情 |
| stock_zh_a_stocks_spot | A 股实时行情（新浪） |
| stock_zh_stock_info | 股票基本信息 |
| stock_zh_a_cash_flow | 股票资金流向 |
| stock_zh_a_limit_data | 涨跌停数据 |

#### 科创板与北交所
| 功能函数 | 说明 |
|----------|------|
| stock_zh_star_board_spot_em | 科创板实时行情 |
| stock_zh_bj_spot_em | 北交所实时行情 |
| stock_zh_bj_hist | 北交所历史行情 |


#### 港股与美股
| 功能函数 | 说明 |
|----------|------|
| stock_hk_spot | 港股实时行情 |
| stock_hk_hist | 港股历史行情 |
| stock_us_spot | 美股实时行情 |
| stock_us_hist | 美股历史行情 |

#### 股票板块
| 功能函数 | 说明 |
|----------|------|
| stock_board_industry_spot_em | 行业板块行情 |
| stock_board_concept_spot_em | 概念板块行情 |

### 2. 期货市场（Futures）

| 功能函数 | 说明 |
|----------|------|
| futures_zh_spot | 国内期货实时行情 |
| futures_zh_hist | 国内期货历史行情 |
| futures_zh_position_rank | 期货持仓排名 |
| futures_zh_warrant | 期货仓单数据 |
| futures_comex_so_spot | COMEX 贵金属 |
| futures_ice_b_spot | 布伦特原油 |

### 3. 期权市场（Options）

#### 商品期权
| 功能函数 | 说明 |
|----------|------|
| option_zh_spot | 商品期权实时行情 |
| option_zh_hist | 商品期权历史行情 |

#### ETF 期权
| 功能函数 | 说明 |
|----------|------|
| option_50etf_spot | 50ETF 期权 |
| option_50etf_hist | 50ETF 期权历史 |
| option_300etf_spot | 300ETF 期权 |
| option_300etf_hist | 300ETF 期权历史 |
| option_1000etf_spot | 1000ETF 期权 |
| option_1000etf_hist | 1000ETF 期权历史 |

### 4. 债券市场（Bond）

| 功能函数 | 说明 |
|----------|------|
| bond_zh_spot | 中国债券实时行情 |
| bond_zh_hist | 中国债券历史行情 |
| bond_zh_above | 上交所债券 |
| bond_zh_trade | 债券现货交易 |

### 5. 基金市场（Fund）

#### 公募基金
| 功能函数 | 说明 |
|----------|------|
| fund_open_fund_spot_em | 开放式基金实时行情 |
| fund_open_fund_hist | 开放式基金历史净值 |
| fund_etf_spot_sina | ETF 基金实时行情 |
| fund_etf_hist_sina | ETF 基金历史净值 |
| fund_lof_spot_sina | LOF 基金实时行情 |

#### 私募基金
| 功能函数 | 说明 |
|----------|------|
| fund_private_spot_em | 私募基金管理人信息 |
| fund_private_sector_em | 私募基金产品信息 |

### 6. 指数数据（Index）

#### A 股指数
| 功能函数 | 说明 |
|----------|------|
| index_zh_a_spot | A 股指数实时行情 |
| index_zh_a_hist | A 股指数历史行情 |

#### 宽基与行业指数
| 功能函数 | 说明 |
|----------|------|
| index_stock_info | 指数基本信息 |
| index_stock_weight | 指数成分股权重 |
| index_sw_spot | 申万行业指数行情 |
| index_sw_hist | 申万行业指数历史 |

### 7. 宏观数据（Macro）

#### 中国宏观
| 功能函数 | 说明 |
|----------|------|
| macro_china_gdp | GDP 数据 |
| macro_china_cpi | CPI 数据 |
| macro_china_ppi | PPI 数据 |
| macro_china_m2 | 货币供应量 |
| macro_china_lpr | LPR 利率 |

#### 国际宏观
| 功能函数 | 说明 |
|----------|------|
| macro_usa_cpi | 美国 CPI |
| macro_usa_gdp | 美国 GDP |
| macro_usa_unemployment | 美国失业率 |
| macro_usa_interest_rate | 美联储利率 |

### 8. 加密货币（Crypto）

| 功能函数 | 说明 |
|----------|------|
| crypto_js_spot | 加密货币实时行情 |
| crypto_hist | 加密货币历史行情 |
| crypto_rank | 加密货币排行榜 |


---

## 五、数据来源

| 数据源 | 说明 |
|--------|------|
| 东方财富 | 股票、基金、期货等实时行情 |
| 新浪财经 | 股票、基金等数据 |
| 上海证券交易所 | 股票、期权等官方数据 |
| 深圳证券交易所 | 股票、期权等官方数据 |
| 北京证券交易所 | 北交所数据 |
| 中国金融期货交易所 | 股指期货数据 |
| 上海期货交易所 | 商品期货数据 |
| 大连商品交易所 | 商品期货数据 |
| 郑州商品交易所 | 商品期货数据 |
| 金十数据 | 宏观数据、国际市场 |
| 申万指数 | 行业指数数据 |

---

## 六、常用函数速查

### 行情数据
```python
# A 股历史行情
ak.stock_zh_a_hist(symbol="000001", period="daily", start_date="20230101", end_date="20231231")

# 港股历史行情
ak.stock_hk_hist(symbol="00700", start_date="20230101", end_date="20231231")

# 美股历史行情
ak.stock_us_hist(symbol="AAPL", start_date="20230101", end_date="20231231")
```

### 实时数据
```python
# A 股实时行情
ak.stock_zh_a_spot_em()

# 板块实时行情
ak.stock_board_industry_spot_em()

# 期权实时行情
ak.option_50etf_spot()
```

---

## 七、使用声明与注意事项

### 使用声明
1. AKShare 提供的数据仅用于学术研究目的
2. 数据仅供参考，不构成任何投资建议
3. 基于 AKShare 研究决策时应注意数据风险
4. 部分接口可能因不可控因素被移除
5. 请遵守各数据源的开源协议

### 最佳实践
- **错误处理**：使用 try-except 处理网络异常
- **缓存机制**：频繁访问建议实现本地缓存
- **时间间隔**：避免过于频繁的请求
- **数据验证**：获取后验证数据完整性

---

## 八、相关资源

| 资源 | 链接 |
|------|------|
| GitHub 仓库 | https://github.com/akfamily/akshare |
| 官方文档 | https://akshare.akfamily.xyz/ |
| PyPI 主页 | https://pypi.org/project/akshare/ |
| 问题反馈 | https://github.com/akfamily/akshare/issues |
| HTTP API (AKTools) | https://github.com/akfamily/aktools |


---

**版本信息**
- 当前版本：1.x（持续更新中）
- 最低 Python 版本：3.9
- 开源协议：MIT

*文档更新时间：2024年*