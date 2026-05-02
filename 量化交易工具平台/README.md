# 量化交易工具与平台

## 目录
- [国内量化平台](./国内量化平台.md)
- [国际量化平台](./国际量化平台.md)
- [开源量化框架](./开源量化框架.md)
- [数据源与API](./数据源与API.md)
- [编程语言与工具](./编程语言与工具.md)

---

## 一、国内主流量化平台

### 聚宽 (JoinQuant)
- **官网**: https://www.joinquant.com
- **特点**: 界面友好，数据全面，回测功能强大
- **功能**: 策略回测、模拟交易、实盘连接、因子研究
- **优势**: 中文社区活跃，教程丰富，适合入门

### 米筐 (RiceQuant)
- **官网**: https://www.ricequant.com
- **特点**: 支持AI写策略，自然语言输入
- **功能**: 策略开发、回测分析、风险分析
- **特色**: 输入"近5日涨幅前10且换手率>5%"自动生成Python代码

### 优矿 (Uqer)
- **官网**: https://uqer.io
- **特点**: 国内量化平台领军者
- **功能**: 从研究到实盘一站式服务
- **优势**: 数据质量高，因子库丰富

### 掘金量化 (Myquant)
- **官网**: https://www.myquant.com
- **特点**: 支持多语言(C++/Python/Java)
- **功能**: 策略开发、回测、实盘
- **优势**: 性能优秀，支持高频策略

### 宽德量化
- **官网**: https://www.quant.cn
- **特点**: 专业量化服务商
- **产品**: 投资服务、技术服务、教育培训

### vn.py
- **官网**: https://www.vnpy.com
- **特点**: 开源量化交易框架
- **语言**: Python
- **优势**: 社区活跃，文档完善

---

## 二、国际量化平台

### QuantConnect
- **官网**: https://www.quantconnect.com
- **特点**: 全球最大的量化社区平台
- **语言**: Python/C#/F#
- **优势**: 支持多市场(股票/期权/期货/加密)

### Quantopian
- **官网**: https://www.quantopian.com
- **特点**: 社区驱动的量化平台
- **功能**: 策略开发、回测、众包研究
- **注意**: 已停止运营，代码可迁移

### Backtrader
- **官网**: https://www.backtrader.com
- **特点**: Python开源回测框架
- **优势**: 灵活、轻量级、易扩展

### Zipline
- **官网**: https://zipline.io
- **特点**: Quantopian开发的回测框架
- **语言**: Python
- **优势**: 事件驱动，与Pandas集成

### Amibroker
- **官网**: https://www.amibroker.com
- **特点**: 专业级技术分析软件
- **语言**: AFL公式语言
- **优势**: 图表功能强大

### MetaTrader 5 (MT5)
- **官网**: https://www.metatrader5.com
- **特点**: 外汇/差价合约交易平台
- **语言**: MQL5
- **优势**: 全球最流行的外汇量化平台

---

## 三、开源量化框架

### Python生态
```
vnpy          - 国产开源量化框架
backtrader    - 事件驱动回测
zipline       - Quantopian回测框架
quantstats    - 量化统计分析
pyfolio       - 组合分析
empyrical     - 风险指标计算
alphalens     - 因子分析
quantconnect  - LEAN引擎开源
```

### C++生态
```
QuantLib      - 定量金融库
TA-Lib        - 技术分析库
```

### Julia生态
```
QuantLib.jl   - Julia版QuantLib
```

---

## 四、数据源与API

### 收费数据源
| 数据源 | 官网 | 特点 |
|--------|------|------|
| Wind | https://www.wind.com.cn | 国内最全，数据质量高 |
| Bloomberg | https://www.bloomberg.com | 全球金融数据标准 |
| Refinitiv | https://www.refinitiv.com | 路透金融数据 |
| FactSet | https://www.factset.com | 机构级数据 |

### 免费/低价数据源
| 数据源 | 官网 | 特点 |
|--------|------|------|
| Tushare | https://tushare.pro | 免费A股数据 |
| AKShare | https://akshare.akfamily.net | 免费财经数据 |
| Yahoo Finance | https://finance.yahoo.com | 美股免费数据 |
| Alpha Vantage | https://www.alphavantage.co | 免费API |
| Quandi | https://www.quandl.com | 经济数据 |

---

## 五、编程语言与工具

### 主要语言
- **Python**: 量化研究首选，生态丰富
- **C++**: 高频交易，低延迟
- **Rust**: 新兴系统编程，高性能
- **Java**: 机构级应用，稳定
- **Julia**: 科学计算，高性能
- **R**: 统计分析，学术研究

### 开发工具
```
Jupyter Notebook/Lab  - 交互式编程环境
VS Code               - 代码编辑器
PyCharm               - Python IDE
Docker                - 容器化部署
Git                   - 版本控制
```

---

## 六、云服务与算力

### 国内云服务
- 阿里云量化服务
- 腾讯云量化平台
- 华为云金融云
- 百度智能云

### 国际云服务
- AWS (Amazon Web Services)
- Azure (Microsoft Azure)
- GCP (Google Cloud Platform)

### GPU算力
- NVIDIA A100/H100
- 算力租赁平台

---

*最后更新: 2026-04-26*
