# FinBERT与自然语言处理(NLP)量化应用

## 一、FinBERT概述

FinBERT是基于BERT架构的金融领域专用预训练语言模型，专门针对金融文本（财报、新闻、分析师报告等）进行优化。

### 模型对比
| 对比维度 | 通用BERT | FinBERT |
|---------|---------|---------|
| 训练数据 | 通用语料 | 金融语料 |
| 金融术语理解 | 一般 | 深入 |
| 情感分析准确率 | 较低 | 显著提升 |

---

## 二、FinBERT模型生态

| 模型 | 开发者 | 特点 |
|------|--------|------|
| ProsusAI/finbert | Prosus AI | 最流行的金融情感分析模型 |
| finbert-zh | 中文金融模型 | 中文金融情感分析 |

```python
from transformers import AutoModelForSequenceClassification, AutoTokenizer

model_name = "ProsusAI/finbert"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)
```

---

## 三、金融NLP核心任务

### 3.1 情感分析

```python
from transformers import pipeline
import pandas as pd

sentiment_analyzer = pipeline(
    "sentiment-analysis",
    model="ProsusAI/finbert"
)

def analyze_batch_news(news_list, batch_size=32):
    results = []
    for i in range(0, len(news_list), batch_size):
        batch = news_list[i:i+batch_size]
        sentiments = sentiment_analyzer(batch)
        for news, sentiment in zip(batch, sentiments):
            results.append({
                "text": news,
                "label": sentiment["label"],
                "score": sentiment["score"]
            })
    return pd.DataFrame(results)
```

### 3.2 命名实体识别(NER)

```python
from transformers import AutoTokenizer, AutoModelForTokenClassification

class FinancialNER:
    def __init__(self, model_name="dslim/bert-base-NER"):
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        self.model = AutoModelForTokenClassification.from_pretrained(model_name)

    def extract_entities(self, text):
        inputs = self.tokenizer(text, return_tensors="pt", truncation=True)
        outputs = self.model(**inputs)
        predictions = torch.argmax(outputs.logits, dim=2)
        tokens = self.tokenizer.convert_ids_to_tokens(inputs["input_ids"][0])
        labels = [self.model.config.id2label[p.item()] for p in predictions[0]]
        return [{"text": t, "type": l[2:]} for t, l in zip(tokens, labels) if l.startswith("B-")]
```

### 3.3 文本摘要

```python
from transformers import pipeline

summarizer = pipeline("summarization")

def summarize_research_report(report_text, max_length=150):
    cleaned_text = report_text.replace("\\\\n", " ").strip()
    return summarizer(cleaned_text, max_length=max_length, min_length=30)[0]["summary_text"]
```

---

## 四、NLP量化因子构建

### 舆情因子体系

```python
import pandas as pd
import numpy as np

class SentimentFactorBuilder:
    def __init__(self, sentiment_model):
        self.model = sentiment_model

    def calculate_daily_sentiment(self, news_df):
        news_df["text"] = news_df["title"].fillna("") + " " + news_df["content"].fillna("")
        sentiments = [self.model(t)[0] for t in news_df["text"]]
        label_map = {"positive": 1, "neutral": 0, "negative": -1}
        news_df["sentiment_value"] = news_df["sentiment_label"].map(label_map)
        return news_df.groupby("date").agg({"sentiment_value": "mean", "sentiment_score": "mean"})

    def build_rolling_sentiment(self, daily_sentiment, windows=[3, 5, 10, 20]):
        factors = pd.DataFrame(index=daily_sentiment.index)
        for w in windows:
            factors[f"sentiment_ma{w}"] = daily_sentiment["sentiment_value"].rolling(w).mean()
            factors[f"sentiment_std{w}"] = daily_sentiment["sentiment_value"].rolling(w).std()
            factors[f"sentiment_sum{w}"] = daily_sentiment["sentiment_value"].rolling(w).sum()
        return factors

    def calculate_ic(self, merged_df, factor_col, return_col="returns"):
        return merged_df[factor_col].corr(merged_df[return_col])
```

### 新闻影响力因子

```python
class NewsImpactFactor:
    def __init__(self):
        self.source_weights = {"Reuters": 1.0, "Bloomberg": 1.0, "WSJ": 0.9, "FT": 0.9, "券商研报": 0.8, "财经网站": 0.5, "社交媒体": 0.3}

    def calculate_news_score(self, news_item):
        sentiment = news_item.get("sentiment", 0)
        source = self.source_weights.get(news_item.get("source", ""), 0.5)
        engagement = min((news_item.get("likes", 0) + news_item.get("comments", 0) * 2) / 1000, 1.0)
        return sentiment * source * (0.7 + 0.3 * engagement)
```

---

## 五、研报分析与自动解读

### 研报情感分析

```python
class ResearchReportAnalyzer:
    def __init__(self):
        self.sentiment_model = pipeline("sentiment-analysis", model="ProsusAI/finbert")

    def extract_key_sections(self, report_text):
        sections = {}
        if "摘要" in report_text:
            sections["摘要"] = report_text[report_text.find("摘要"):report_text.find("摘要")+500]
        return sections

    def analyze_report(self, report_text):
        sections = self.extract_key_sections(report_text)
        return {k: self.sentiment_model(v)[0] for k, v in sections.items()}
```

---

## 六、社交媒体情绪分析

```python
class SocialMediaSentiment:
    def __init__(self):
        self.model = pipeline("sentiment-analysis", model="ProsusAI/finbert")

    def clean_text(self, text):
        import re
        text = re.sub(r"@[\\\\w]+", "", text)
        text = re.sub(r"http\\\\S+", "", text)
        return re.sub(r"\\\\s+", " ", text).strip()

    def aggregate_daily_sentiment(self, posts_df):
        analyzed = []
        for _, post in posts_df.iterrows():
            cleaned = self.clean_text(post["content"])
            if len(cleaned) >= 10:
                sentiment = self.model(cleaned)[0]
                analyzed.append({"date": post["date"], "sentiment": sentiment})
        if not analyzed: return pd.DataFrame()
        df = pd.DataFrame(analyzed)
        label_map = {"positive": 1, "neutral": 0, "negative": -1}
        df["value"] = df["sentiment"].apply(lambda x: label_map.get(x["label"], 0))
        return df.groupby("date").agg({"value": "mean", "sentiment": "count"})
```

---

## 七、完整NLP量化策略

```python
class NLPStrategy:
    def __init__(self, sentiment_model):
        self.model = sentiment_model
        self.factor_builder = SentimentFactorBuilder(sentiment_model)
        self.top_n = 50

    def generate_signals(self, all_factors):
        signals = []
        for stock, factors in all_factors.items():
            if factors is None or factors.empty: continue
            for col in ["sentiment_ma5", "sentiment_ma10", "sentiment_ma20"]:
                if col in factors.columns:
                    ic = self.factor_builder.calculate_ic(factors, col)
                    if ic > 0.05 and factors[col].iloc[-1] > 0:
                        signals.append({"stock": stock, "factor": col, "ic": ic, "signal": 1})
                    elif ic < -0.05 and factors[col].iloc[-1] < 0:
                        signals.append({"stock": stock, "factor": col, "ic": ic, "signal": -1})
        return pd.DataFrame(signals)
```

---

## 八、模型选择指南

| 场景 | 推荐模型 | 优点 |
|------|---------|------|
| 英文金融情感 | ProsusAI/finbert | 专用预训练 |
| 中文金融情感 | finbert-zh/ChatGLM | 中文理解 |
| 快速原型 | DistilBERT | 速度快 |
| 多语言 | mBERT/XLM-RoBERTa | 多语言支持 |

---

## 九、工具与资源

| 库 | 用途 |
|---|------|
| transformers | BERT/FinBERT模型 |
| huggingface_hub | 模型托管 |
| torch | 深度学习框架 |
| spacy | NLP工具 |
| jieba | 中文分词 |

---

## 十、总结

FinBERT和NLP技术在量化投资中的主要应用：

1. **舆情因子**：分析新闻、研报、社交媒体生成情感因子
2. **另类数据挖掘**：从非结构化文本提取市场情绪信号
3. **智能投研**：自动化解析研报和财报会议
4. **策略增强**：NLP因子与传统量化因子结合

**关键要点**：
- FinBERT在金融情感分析上显著优于通用BERT
- 文本预处理和去重对结果质量影响很大
- IC分析和回测验证是因子筛选的必要步骤

**未来趋势**：
- 多模态融合：文本+图表+视频联合分析
- 大模型应用：GPT-4等在金融理解上的应用
- 实时流处理：实时新闻情感监控

---

*最后更新: 2026-04-26*
