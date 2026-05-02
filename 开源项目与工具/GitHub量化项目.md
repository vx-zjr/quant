# GitHub量化项目大全

## 一、头部量化公司开源

### 1. 腾讯开源项目
| 项目 | 链接 | Stars | 说明 |
|------|------|-------|------|
| AngelSlim | github.com/Tencent/AngelSlim | 3k+ | 大模型压缩工具包，支持LLM量化、剪枝 |
| ncnn | github.com/Tencent/ncnn | 15k+ | 移动端神经网络框架 |
| FastMoE | github.com/Tencent/FASTMOE | 2k+ | MoE分布式训练系统 |
| RapidJSON | github.com/Tencent/rapidjson | 11k+ | 高性能JSON解析库 |
| phosphor-svelte | github.com/Tencent/phosphor-svelte | 2k+ | 图标组件库 |

### 2. 阿里开源项目
| 项目 | 链接 | Stars | 说明 |
|------|------|-------|------|
| EasyNLP | github.com/alibaba/EasyNLP | 5k+ | 简单易用的NLP工具包 |
| Graph-Learn | github.com/alibaba/graph-learn | 4k+ | 图神经网络框架 |
| PredictServe | github.com/alibaba/predict-serve | 2k+ | 模型推理服务 |
| MaaT | github.com/alibaba/MaaT | 1k+ | 蚂蚁图学习平台 |

### 3. 其他大厂开源
| 公司 | 项目 | 说明 |
|------|------|------|
| Amazon | GluonTS | 时序预测库 |
| Amazon | Chronos | 时序基础模型 |
| Google | TF-Ranking | 学习排序 |
| Meta | Kats | 时间序列分析 |
| Microsoft | FLAML | 自动机器学习 |

---

## 二、量化交易框架

### Python框架

#### 1. vnpy
```yaml
名称: vnpy
链接: github.com/vnpy/vnpy
Stars: 20k+
语言: Python
描述: 最流行的开源量化交易框架
特点:
  - 支持股票、期货、期权、外汇
  - 事件驱动回测引擎
  - 丰富的数据服务
  - 社区活跃，文档完善

安装: pip install vnpy

核心模块:
  - vnpy.app: 策略应用模块
  - vnpy.data: 数据服务
  - vnpy.gateway: 交易接口
  - vnpy.algorithm: 算法交易
```

#### 2. backtrader
```yaml
名称: backtrader
链接: github.com/mementum/backtrader
Stars: 8k+
描述: 成熟的事件驱动回测框架

特点:
  - 简洁的策略开发API
  - 内置多个数据源
  - 支持多种订单类型
  - 可视化分析

代码示例:
  from backtrader import Strategy, bt
  class MyStrategy(Strategy):
      def __init__(self):
          self.sma = bt.indicators.SMA(self.data.close, period=15)
      
      def next(self):
          if self.data.close > self.sma:
              self.buy()
          elif self.data.close < self.sma:
              self.sell()
```

#### 3. zipline
```yaml
名称: zipline
链接: github.com/quantopian/zipline
Stars: 7k+
描述: Quantopian开发的回测框架

特点:
  - 与Quantopian研究平台集成
  - 内置多个数据源
  - 算法交易支持
  - 性能优化

安装: pip install zipline
```

#### 4.QuantConnect(Lean)
```yaml
名称: Lean
链接: github.com/QuantConnect/Lean
Stars: 10k+
描述: QuantConnect量化平台的引擎

特点:
  - 支持C#、Python
  - 多市场支持
  - 实时交易
  - 云端回测
```

### C++框架

#### 1. QuantLib
```yaml
名称: QuantLib
链接: github.com/lballabio/QuantLib
描述: 专业量化金融C++库

功能:
  - 期权定价模型
  - 利率模型
  - 固定收益
  - 风险计量

安装: conda install -c conda-forge quantlib
```

#### 2. Strata
```yaml
名称: Strata
链接: github.com/OpenGamma/Strata
描述: OpenGamma开发的量化Java库

应用:
  - 定价引擎
  - 风险分析
  - 市场数据
```

### JavaScript框架

#### 1. ccxt
```yaml
名称: ccxt
链接: github.com/ccxt/ccxt
Stars: 18k+
描述: 加密货币交易所统一API

支持交易所:
  - Binance, Coinbase, Kraken
  - Bybit, OKX, Bitget
  - 100+交易所

代码示例:
  import ccxt
  binance = ccxt.binance()
  ohlcv = binance.fetch_ohlcv('BTC/USDT', '1h')
```

---

## 三、机器学习量化库

### 深度学习框架

#### 1. PyTorch
```yaml
官网: pytorch.org
GitHub: github.com/pytorch/pytorch
Stars: 80k+
说明: Facebook开源的深度学习框架

量化应用:
  - 神经网络策略
  - 时间序列预测
  - NLP因子挖掘
```

#### 2. TensorFlow
```yaml
官网: tensorflow.org
GitHub: github.com/tensorflow/tensorflow
Stars: 180k+
说明: Google开源的ML框架

量化应用:
  - 量化策略开发
  - 模型部署
  - TF-Ranking用于排序
```

#### 3. JAX
```yaml
官网: jax.dev
GitHub: github.com/google/jax
说明: Google高性能ML框架

特点:
  - 自动微分
  - GPU/TPU加速
  - 函数式编程
```

### 时序预测库

#### 1. GluonTS
```yaml
名称: GluonTS
链接: github.com/awslabs/gluonts
Stars: 5k+
说明: Amazon开源的时序预测库

功能:
  - 概率预测
  - 模型组合
  - 数据处理
```

#### 2. Prophet
```yaml
名称: Prophet
链接: github.com/facebook/prophet
Stars: 18k+
说明: Facebook时序预测工具

特点:
  - 易于使用
  - 季节性分解
  - 异常检测
```

#### 3. Kats
```yaml
名称: Kats
链接: github.com/facebookresearch/kats
Stars: 8k+
说明: Meta的时间序列分析工具

功能:
  - 预测
  - 异常检测
  - 特征提取
```

### 强化学习库

#### 1. RLlib
```yaml
名称: RLlib
链接: github.com/ray-project/ray
Stars: 30k+
说明: Ray强化学习库

功能:
  - 多算法支持
  - 分布式训练
  - 环境集成
```

#### 2. Stable-Baselines3
```yaml
名称: Stable-Baselines3
链接: github.com/DLR-RM/stable-baselines3
Stars: 8k+
说明: 强化学习基线算法

算法:
  - PPO, A2C, SAC
  - DQN, TD3
```

---

## 四、金融数据处理

### 数据获取库

#### 1. AKShare
```yaml
名称: AKShare
链接: github.com/akfamily/akshare
Stars: 8k+
说明: 纯免费Python财经数据接口

功能:
  - A股数据
  - 期货数据
  - 宏观数据
  - 实时行情

安装: pip install akshare

代码示例:
  import akshare as ak
  df = ak.stock_zh_a_hist(symbol="000001", period="daily", start_date="20230101")
```

#### 2. Tushare
```yaml
名称: Tushare
链接: tushare.pro
说明: 专业财经数据接口

功能:
  - A股完整数据
  - 财务数据
  - 基金数据
  - 需要积分
```

#### 3. yfinance
```yaml
名称: yfinance
链接: github.com/ranaroussi/yfinance
Stars: 12k+
说明: Yahoo Finance数据接口

功能:
  - 美股数据
  - 港股数据
  - 加密货币

代码示例:
  import yfinance as yf
  data = yf.download("AAPL", start="2020-01-01")
```

### 数据库

#### 1. DolphinDB
```yaml
名称: DolphinDB
链接: dolphindb.com
说明: 国产高性能时序数据库

特点:
  - 超高写入性能
  - 内置量化分析函数
  - 支持Python/Java API
```

#### 2. InfluxDB
```yaml
名称: InfluxDB
链接: influxdata.com
说明: 开源时序数据库

特点:
  - 高效时序存储
  - InfluxQL查询
  - 丰富生态
```

---

## 五、量化策略项目

### 1. FinRL
```yaml
名称: FinRL
链接: github.com/AI4Finance-Foundation/FinRL
Stars: 8k+
说明: 深度强化学习量化框架

特点:
  - 集成多种RL算法
  - 支持股票、加密
  - 完整教程

算法:
  - DQN, PPO, SAC
  - A2C, TD3
```

### 2. FinGPT
```yaml
名称: FinGPT
链接: github.com/AI4Finance-Foundation/FinGPT
Stars: 5k+
说明: 开源金融大模型

功能:
  - 金融NLP
  - 舆情分析
  - 研报解读
```

### 3. QuantConnect Samples
```yaml
名称: LEAN Samples
链接: github.com/QuantConnect/Lean
说明: QuantConnect策略示例

示例策略:
  - 双均线策略
  - 配对交易
  - 统计套利
```

### 4. trading-bot
```yaml
名称: trading-bot
链接: github.com/robsim378/syscrypto
Stars: 3k+
说明: 多交易所加密交易机器人

功能:
  - 网格交易
  - 均线策略
  - 跟单交易
```

---

## 六、因子研究项目

### 1. AlphaFactor
```yaml
名称: AlphaFactor
链接: 内部开发
说明: 多因子框架

功能:
  - 因子计算
  - IC分析
  - 组合优化
```

### 2. alphalens
```yaml
名称: alphalens
链接: github.com/quantopian/alphalens
Stars: 3k+
说明: 因子分析工具

功能:
  - 因子IC分析
  - 分组收益
  - 预测分析
```

### 3. pyfolio
```yaml
名称: pyfolio
链接: github.com/quantopian/pyfolio
Stars: 4k+
说明: 组合分析工具

功能:
  - 绩效归因
  - 风险分析
  - 图表可视化
```

---

## 七、风控工具

### 1. riskfolio-lib
```yaml
名称: riskfolio-lib
链接: github.com/jumping-lip/riskfolio-lib
Stars: 2k+
说明: 投资组合风险优化

功能:
  - 组合优化
  - 风险平价
  - Black-Litterman
```

### 2. PyPortfolioOpt
```yaml
名称: PyPortfolioOpt
链接: github.com/robertmartin8/PyPortfolioOpt
Stars: 5k+
说明: 组合优化库

功能:
  - 均值方差
  - 有效前沿
  - 风险平价
```

---

## 八、可视化工具

### 1. mplfinance
```yaml
名称: mplfinance
链接: github.com/matplotlib/mplfinance
Stars: 2k+
说明: 金融图表绘制

功能:
  - K线图
  - 技术指标
  - 自定义样式
```

### 2. finplot
```yaml
名称: finplot
链接: github.com/matplotlib/finplot
说明: 高性能金融图表

特点:
  - WebGL加速
  - 实时更新
  - 交互性强
```

---

## 九、GitHub搜索技巧

### 搜索量化项目
```bash
# 搜索回测框架
site:github.com backtrader OR zipline

# 搜索机器学习量化
site:github.com "machine learning" trading

# 搜索金融数据
site:github.com "financial data" python

# 搜索强化学习交易
site:github.com "reinforcement learning" trading
```

### 常用标签
| 标签 | 说明 |
|------|------|
| quantitative-trading | 量化交易 |
| algorithmic-trading | 算法交易 |
| backtesting | 回测系统 |
| financial-analysis | 金融分析 |
| time-series | 时序分析 |
| machine-learning | 机器学习 |
| deep-learning | 深度学习 |
| reinforcement-learning | 强化学习 |

---

## 十、项目推荐榜单

### 回测框架Top 5
1. **vnpy** - 20k Stars - 功能全面
2. **backtrader** - 8k Stars - 简洁易用
3. **zipline** - 7k Stars - Quantopian出品
4. **Lean** - 10k Stars - QuantConnect引擎
5. **backtesting.py** - 3k Stars - 轻量级

### 机器学习Top 5
1. **PyTorch** - 80k Stars - 深度学习
2. **TensorFlow** - 180k Stars - 工业级
3. **FinRL** - 8k Stars - RL量化
4. **FinGPT** - 5k Stars - 金融LLM
5. **scikit-learn** - 55k Stars - 传统ML

### 数据处理Top 5
1. **ccxt** - 18k Stars - 加密API
2. **akshare** - 8k Stars - 免费A股
3. **yfinance** - 12k Stars - 美股数据
4. **pandas** - 40k Stars - 数据分析
5. **DolphinDB** - 商业 - 时序数据库

### 风控优化Top 5
1. **riskfolio-lib** - 2k Stars - 组合优化
2. **PyPortfolioOpt** - 5k Stars - 组合优化
3. **QuantLib** - 6k Stars - 风险计量
4. **pyfolio** - 4k Stars - 绩效分析
5. **alphalens** - 3k Stars - 因子分析

---

## 十一、快速入门指南

### 搭建量化环境
```bash
# 创建环境
conda create -n quant python=3.10
conda activate quant

# 安装核心库
pip install numpy pandas scikit-learn
pip install xgboost lightgbm catboost
pip install backtrader zipline

# 安装数据库
pip install akshare yfinance

# 安装可视化
pip install matplotlib plotly seaborn

# 安装深度学习(可选)
pip install torch tensorflow

# 安装强化学习(可选)
pip install stable-baselines3 ray
```

### 推荐学习路径
```
第1步: 学习Python基础
  └── 用backtrader开发简单策略

第2步: 学习数据分析
  └── 用pandas处理金融数据
  └── 用akshare获取数据

第3步: 学习机器学习
  └── 用sklearn构建因子模型
  └── 用xgboost选股

第4步: 学习深度学习
  └── 用PyTorch开发神经网络策略

第5步: 学习强化学习
  └── 用FinRL开发RL策略

第6步: 生产部署
  └── 用vnpy实盘交易
```

---

*最后更新: 2026-04-26*