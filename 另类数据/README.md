# 另类数据资源

## 目录
- [卫星图像数据](./卫星图像数据.md)
- [消费数据](./消费数据.md)
- [舆情情绪数据](./舆情情绪数据.md)
- [设备行为数据](./设备行为数据.md)
- [供应链数据](./供应链数据.md)
- [ESG数据](./ESG数据.md)

---

## 一、卫星图像数据

### 主要提供商
| 提供商 | 官网 | 数据类型 | 应用场景 |
|--------|------|----------|----------|
| Planet Labs | planet.com | 每日地球影像 | 停车场车辆计数 |
| Orbital Insight | orbitalinsight.com | 商业活动分析 | 零售客流分析 |
| Spaceflight Industries | spaceflight.com | 船舶跟踪 | 国际贸易 |
| RS Metrics | rsmetrics.com | 大宗商品监测 | 钢铁、原油产量 |
| Descartes Labs | descarteslabs.com | 卫星图像AI分析 | 农业、零售 |

### 应用案例
| 应用 | 数据 | 交易逻辑 |
|------|------|----------|
| 零售客流 | 停车场车辆数 | 预测零售企业营收 |
| 石油库存 | 储油罐卫星图 | 预测原油库存变化 |
| 农作物 | 农田卫星图 | 预测农产品产量 |
| 船舶 | 港口船舶数 | 预测贸易量 |

### 获取方式
| 方式 | 说明 | 成本 |
|------|------|------|
| 直接购买 | 从提供商购买 | 高 |
| API接入 | 通过API获取 | 中 |
| 数据聚合 | 从第三方获取 | 低 |

---

## 二、消费数据

### 信用卡消费数据
| 提供商 | 特点 | 数据延迟 |
|--------|------|----------|
| Second Measure | 实时消费数据 | 1-2个月 |
| FactSet | 信用卡数据 | 1个月 |
| YipitData | 订阅数据 | 1个月 |

### 电商数据
| 数据源 | 内容 | 应用 |
|--------|------|------|
| 京东/淘宝 | 电商GMV | 消费预测 |
| 美团 | 餐饮数据 | 餐饮行业 |
| 拼多多 | 低价消费 | 消费降级 |

### POS数据
| 应用 | 数据类型 | 说明 |
|------|----------|------|
| 门店客流 | POS交易量 | 实体零售 |
| 支付数据 | 支付金额 | 消费趋势 |

---

## 三、舆情情绪数据

### 社交媒体数据
| 平台 | 数据类型 | API |
|------|----------|-----|
| Twitter/X | 推文、情绪 | 有API |
| Reddit | 讨论、评论 | 有API |
| StockTwits | 股票讨论 | 有API |
| 微博 | 中文社交 | 有限制 |

### 新闻数据
| 提供商 | 特点 | 覆盖范围 |
|--------|------|----------|
| RavenPack | 新闻NLP分析 | 全球新闻 |
| Bloomberg | 财经新闻 | 专业财经 |
| Refinitiv | 新闻数据 | 全球覆盖 |
| GDELT | 全球新闻事件 | 免费 |

### 搜索趋势
| 数据源 | 应用 |
|--------|------|
| Google Trends | 搜索趋势 |
| Baidu Index | 百度搜索 |
| 微信指数 | 微信热词 |

### 情绪量化指标
```python
# 情绪因子构建示例
import pandas as pd
from textblob import TextBlob
import numpy as np

def calculate_sentiment_score(texts):
    """
    计算文本情绪得分
    """
    scores = []
    for text in texts:
        blob = TextBlob(text)
        scores.append(blob.sentiment.polarity)
    return np.array(scores)

def build_sentiment_factor(news_df, stock_df):
    """
    构建舆情因子
    """
    # 情绪得分
    news_df['sentiment'] = news_df['text'].apply(
        lambda x: calculate_sentiment_score([x]).mean()
    )
    
    # 按日期聚合
    daily_sentiment = news_df.groupby('date')['sentiment'].mean()
    
    # 与股票收益合并
    factor = pd.merge(stock_df, daily_sentiment, on='date')
    return factor
```

---

## 四、设备行为数据

### 移动设备数据
| 数据类型 | 应用 | 提供商 |
|----------|------|--------|
| GPS位置 | 门店客流 | SafeGraph |
| APP使用 | 用户行为 | App Annie |
| 流量数据 | 广告效果 | Oracle Data Cloud |

### 地理位置数据
| 数据源 | 应用场景 |
|--------|----------|
| 手机信令 | 人口流动 |
| 停车场 | 零售客流 |
| 交通 | 经济活动 |

### 网络流量数据
| 数据类型 | 应用 |
|------|------|
| 网站流量 | 流量排名 |
| 搜索流量 | 需求变化 |
| 广告点击 | 消费意图 |

---

## 五、供应链数据

### 供应链监控
| 数据类型 | 说明 |
|----------|------|
| 物流数据 | 运输时效 |
| 港口数据 | 贸易流量 |
| 库存数据 | 企业库存 |

### 航运数据
| 数据源 | 应用 |
|--------|------|
| AIS数据 | 船舶跟踪 |
| 港口拥堵 | 供应链瓶颈 |
| 集装箱数据 | 贸易量 |

### 供应链金融
| 指标 | 应用 |
|------|------|
| 应付账款 | 企业现金流 |
| 存货周转 | 销售状况 |
| 物流时效 | 经济活力 |

---

## 六、ESG数据

### ESG评级
| 提供商 | 特点 |
|--------|------|
| MSCI ESG | 国际权威 |
| Sustainalytics | 风险评级 |
| 彭博ESG | 综合数据 |
| 商道融绿 | 国内ESG |

### 碳排放数据
| 数据源 | 应用 |
|--------|------|
| Carbon Disclosure Project | 碳排放披露 |
| 卫星监测 | 全球变暖 |
| 碳交易所 | 碳价格 |

### ESG因子应用
| 因子 | 描述 |
|------|------|
| 环境得分 | 环保表现 |
| 社会得分 | 社会责任 |
| 公司治理 | 治理质量 |

---

## 七另类数据获取平台

### 数据市场
| 平台 | 网址 | 特点 |
|------|------|------|
| DataHub | datahub.io | 综合数据 |
| Quandl | quandl.com | 金融数据 |
| Alpha Vantage | alphavantage.co | 免费API |
| Numerai | numer.ai | 预测数据 |

### Kaggle数据集
| 数据集 | 内容 |
|--------|------|
| Two Sigma数据 | 市场预测 |
| Jane Street | 匿名特征 |
| Individual Investors | 投资者情绪 |

### 另类数据评估
| 评估维度 | 内容 |
|----------|------|
| 覆盖度 | 数据覆盖范围 |
| 时效性 | 数据更新频率 |
| 准确性 | 数据准确度 |
| 独特性 | 数据稀缺性 |

---

*最后更新: 2026-04-26*
