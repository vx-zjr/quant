# 开源项目与工具

## 目录
- [GitHub量化项目](./GitHub量化项目.md)
- [量化框架汇总](./量化框架汇总.md)
- [机器学习工具库](./机器学习工具库.md)
- [数据处理工具](./数据处理工具.md)

---

## 一、GitHub热门量化项目

### 头部量化公司开源

#### 腾讯开源项目
| 项目 | GitHub | 特点 |
|------|--------|------|
| AngelSlim | Tencent/AngelSlim | 大模型压缩工具包，支持量化 |
| ncnn | Tencent/ncnn | 移动端神经网络框架 |
| HPC-Ops | - | 高性能算子库 |
| FastMoE | Tencent/FASTMOE | MoE分布式训练 |

#### 其他大厂项目
| 项目 | 公司 | 特点 |
|------|------|------|
| FinRL | - | 深度强化学习量化框架 |
| FinRL-DeepDow | - | 深度学习量化投资 |
| GluonTS | Amazon | 时序预测库 |
| Kats | Meta | 时间序列分析工具包 |

---

## 二、量化框架汇总

### Python量化框架
| 框架 | GitHub | Stars | 特点 |
|------|--------|-------|------|
| vnpy | vnpy/vnpy | 20k+ | 国产量化框架，功能全面 |
| backtrader | backtrader/backtrader | 8k+ | 事件驱动回测 |
| zipline | quantopian/zipline | 7k+ | Quantopian回测框架 |
| pyfolio | quantopian/pyfolio | 2k+ | 组合分析工具 |
| alphalens | quantopian/alphalens | 1.5k+ | 因子分析工具 |

### C++量化框架
| 框架 | 特点 |
|------|------|
| QuantLib | 定量金融C++库，衍生品定价 |
| TA-Lib | 技术分析库 |
| OpenCL Algo | GPU并行算法 |

### Java量化框架
| 框架 | 特点 |
|------|------|
| Strata | OpenGamma量化库 |
| JQuantLib | QuantLib Java版 |

---

## 三、机器学习工具库

### 深度学习框架
| 框架 | 官网 | 量化应用 |
|------|------|----------|
| PyTorch | pytorch.org | 深度学习量化策略 |
| TensorFlow | tensorflow.org | 神经网络模型 |
| JAX | jax.com | 高性能ML研究 |
| MXNet | mxnet.apache.org | 自动微分 |

### 专用ML库
| 库 | 特点 |
|------|------|
| scikit-learn | 传统机器学习 |
| XGBoost | 梯度提升树 |
| LightGBM | 高效梯度提升 |
| CatBoost | 类别特征处理 |
| Prophet | Facebook时序预测 |
| NeuralProphet | 神经时序预测 |

### 强化学习库
| 库 | 特点 |
|------|------|
| RLlib | Ray强化学习库 |
| Stable-Baselines3 | 强化学习基线 |
| OpenAI Baselines | 深度RL实现 |

---

## 四、数据处理工具

### 时序数据库
| 数据库 | 特点 |
|--------|------|
| InfluxDB | 开源时序数据库 |
| TimescaleDB | PostgreSQL时序扩展 |
| KDB+ | 高性能时序数据库(商业) |
| DolphinDB | 国产高性能时序数据库 |
| Prometheus | 监控时序数据 |

### 数据处理库
| 库 | 特点 |
|------|------|
| Pandas | 数据分析基础库 |
| Polars | 高性能DataFrame |
| Dask | 大规模并行处理 |
| Vaex | 大数据框处理 |
| Modin | Pandas加速版 |

### 量化专用数据API
```python
# 常用数据获取
import akshare as ak    # 免费财经数据
import tushare as ts    # A股数据
import yfinance as yf    # 美股数据
import ccxt             # 加密货币交易所
```

---

## 五、数据可视化

### 图表库
| 库 | 特点 |
|------|------|
| Plotly | 交互式图表 |
| Bokeh | Web可视化 |
| Altair | 声明式统计可视化 |
| Matplotlib | Python基础绘图 |
| Seaborn | 统计图表 |

### 量化专用图表
| 库 | 特点 |
|------|------|
| mplfinance | 金融图表 |
| chartpy | 多库统一接口 |
| finplot | 高性能金融图表 |

---

## 六、风控工具

### 风险管理库
| 库 | 特点 |
|------|------|
| PyVaR | VaR计算工具 |
| riskfolio-lib | 组合风险优化 |
| QuantLib | 风险计量 |
| frisk | 金融风险工具 |

### 性能分析
| 库 | 特点 |
|------|------|
| line_profiler | 代码行级分析 |
| memory_profiler | 内存分析 |
| cProfile | 性能剖析 |

---

## 七、部署工具

### 容器化
| 工具 | 特点 |
|------|------|
| Docker | 容器化部署 |
| Kubernetes | 容器编排 |
| Docker Compose | 多容器编排 |

### 模型服务
| 工具 | 特点 |
|------|------|
| TensorFlow Serving | TF模型服务 |
| TorchServe | PyTorch模型服务 |
| Triton | NVIDIA推理服务 |
| BentoML | ML模型部署 |

### 交易API
| API | 支持品种 |
|------|----------|
| Interactive Brokers | 全球多市场 |
| Alpaca | 美股零佣金API |
| Binance API | 加密货币 |
| OANDA | 外汇 |

---

## 八、工具推荐配置

### 量化开发环境
```yaml
# docker-compose.yml 示例
version: '3'
services:
  jupyter:
    image: jupyter/scipy-notebook
    ports:
      - "8888:8888"
    volumes:
      - ./:/home/jovyan/work
    environment:
      - JUPYTER_TOKEN=your-token
      
  postgresql:
    image: postgres:14
    environment:
      - POSTGRES_PASSWORD=your-password
      
  redis:
    image: redis:7
    ports:
      - "6379:6379"
```

### Anaconda环境配置
```bash
# 创建量化环境
conda create -n quant python=3.10
conda activate quant
pip install numpy pandas scikit-learn xgboost lightgbm
pip install pyfoliobacktest ta-lib
pip install akshare tushare
```

---

*最后更新: 2026-04-26*
