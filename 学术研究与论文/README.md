# 学术研究与论文资源

> 最后更新：2026年1月

---

## 目录

- [前沿研究方向2024-2026](./前沿研究方向2024-2026.md)
- [顶级会议论文精选](./顶级会议论文精选.md)
- [研究方法论趋势](./研究方法论趋势.md)
- [产学研转化案例](./产学研转化案例.md)
- [顶级学术会议](./顶级学术会议.md)
- [顶级金融期刊](./顶级金融期刊.md)
- [预印本平台](./预印本平台.md)
- [经典量化著作](./经典量化著作.md)

---

## 核心内容

### 2024-2026年前沿研究方向

1. **时序基础模型**: Chronos, TimeGPT, Lag-Llama
2. **金融大模型**: BloombergGPT, FinGPT
3. **因果推断**: CausalML, DoWhy
4. **强化学习交易**: FinRL, 世界模型
5. **多模态金融**: 文本+图表+时序融合

### 顶级会议

| 会议 | 官网 | 量化论文方向 |
|------|------|-------------|
| NeurIPS | neurips.cc | 深度学习、强化学习 |
| ICML | icml.cc | 机器学习应用 |
| ICLR | iclr.cc | 表示学习 |
| ICAIF | ai-for-finance.org | 量化金融AI |
| KDD | kdd.org | 数据挖掘 |

### 论文发现工具

- arXiv q-fin: arxiv.org/list/q-fin/recent
- Semantic Scholar: AI驱动的论文搜索
- Connected Papers: 论文引用图谱

---

## 快速导航

### 论文搜索

```python
# Semantic Scholar API
import requests

def search_papers(query, limit=20):
    url = "https://api.semanticscholar.org/graph/v1/paper/search"
    params = {
        "query": query,
        "fields": "title,authors,year,venue,citationCount",
        "limit": limit
    }
    return requests.get(url, params=params).json()

# 推荐搜索
search_papers("time series foundation model finance")
search_papers("reinforcement learning trading")
search_papers("causal inference quantitative")
```

### 推荐阅读

**入门必读**
1. BloombergGPT (arXiv:2303.17564)
2. Chronos (arXiv:2403.07890)
3. FinRL (arXiv:2011.08695)

**进阶必读**
1. NeurIPS Quant Workshop 论文集
2. ICAIF 最佳论文集
3. JFE/JF 机器学习专刊

---

*版本: v2.0 | 2026年1月更新*
