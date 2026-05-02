# 因子信号系统完整指南

> **更新**: 2026-04-26 | **收集范围**: 因子挖掘/信号生成/因子组合/Alpha提取

---

## 一、模块导入与基础配置

```python
import pandas as pd
import numpy as np
from typing import Dict, List, Tuple, Optional, Union
from dataclasses import dataclass, field
from abc import ABC, abstractmethod
from datetime import datetime
from enum import Enum
import warnings
warnings.filterwarnings("ignore")


class FactorCategory(Enum):
    VALUE = "value"
    MOMENTUM = "momentum"
    QUALITY = "quality"
    SENTIMENT = "sentiment"
    VOLATILITY = "volatility"
    SIZE = "size"
    MACRO = "macro"
    ALTERNATIVE = "alternative"


@dataclass
class FactorConfig:
    name: str
    category: str
    lookback_period: int = 20
    decay_half_life: int = 20
    neutralization: List[str] = field(default_factory=list)
    winsorize_pct: float = 0.01
    min_samples: int = 100
    description: str = ""
    author: str = ""
    created_at: str = field(default_factory=lambda: datetime.now().isoformat())
```

---

## 二、因子挖掘模块

### 2.1 因子挖掘基类

```python
class FactorMiningBase(ABC):
    "因子挖掘基类"
    
    def __init__(self, name: str, category: FactorCategory):
        self.name = name
        self.category = category
        self.config = FactorConfig(
            name=name,
            category=category.value,
            lookback_period=20,
            decay_half_life=20,
            neutralization=[],
            winsorize_pct=0.01
        )
        self.statistics = {}
        
    @abstractmethod
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        pass
    
    def preprocess(self, factor: pd.Series) -> pd.Series:
        factor = self._winsorize(factor, self.config.winsorize_pct)
        factor = self._standardize(factor)
        return factor
    
    def _winsorize(self, series: pd.Series, pct: float = 0.01) -> pd.Series:
        lower = series.quantile(pct)
        upper = series.quantile(1 - pct)
        return series.clip(lower, upper)
    
    def _standardize(self, series: pd.Series) -> pd.Series:
        mean = series.mean()
        std = series.std()
        if std == 0:
            return series - mean
        return (series - mean) / std


class FactorRegistry:
    "因子注册表"
    
    def __init__(self):
        self._factors: Dict[str, FactorMiningBase] = {}
        self._categories: Dict[FactorCategory, List[str]] = {}
        
    def register(self, factor: FactorMiningBase):
        self._factors[factor.name] = factor
        if factor.category not in self._categories:
            self._categories[factor.category] = []
        self._categories[factor.category].append(factor.name)
        
    def get(self, name: str) -> Optional[FactorMiningBase]:
        return self._factors.get(name)
    
    def list_by_category(self, category: FactorCategory) -> List[str]:
        return self._categories.get(category, [])
    
    def list_all(self) -> List[str]:
        return list(self._factors.keys())
```

### 2.2 价值因子挖掘

```python
class ValueFactorMining(FactorMiningBase):
    "价值因子挖掘"
    
    def __init__(self, name: str = "value"):
        super().__init__(name, FactorCategory.VALUE)
        self.config.description = "基于估值指标的价值因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        value_factors = pd.DataFrame(index=data.index)
        
        if "pe" in data.columns:
            value_factors["ep"] = 1 / data["pe"].replace(0, np.nan)
            value_factors["ep"] = value_factors["ep"].replace([np.inf, -np.inf], np.nan)
        
        if "pb" in data.columns:
            value_factors["bp"] = 1 / data["pb"].replace(0, np.nan)
            value_factors["bp"] = value_factors["bp"].replace([np.inf, -np.inf], np.nan)
        
        if "ps" in data.columns:
            value_factors["sp"] = 1 / data["ps"].replace(0, np.nan)
            value_factors["sp"] = value_factors["sp"].replace([np.inf, -np.inf], np.nan)
        
        if "pcf" in data.columns:
            value_factors["pcf"] = 1 / data["pcf"].replace(0, np.nan)
            value_factors["pcf"] = value_factors["pcf"].replace([np.inf, -np.inf], np.nan)
        
        valid_cols = [c for c in value_factors.columns if value_factors[c].notna().sum() > self.config.min_samples]
        
        if len(valid_cols) > 0:
            value_score = value_factors[valid_cols].rank(axis=1, na_option="keep").mean(axis=1)
        else:
            value_score = pd.Series(index=data.index)
        
        return self.preprocess(value_score)


class PEGrowthFactor(FactorMiningBase):
    "PEG增长因子"
    
    def __init__(self, name: str = "peg"):
        super().__init__(name, FactorCategory.VALUE)
        self.config.description = "PEG增长因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        if not all(x in data.columns for x in ["pe", "eps_growth"]):
            return pd.Series(index=data.index)
        
        peg = data["pe"] / (data["eps_growth"].replace(0, np.nan) * 100)
        peg = peg.replace([np.inf, -np.inf], np.nan)
        peg = self._winsorize(peg, 0.01)
        peg_inv = 1 / peg.replace(0, np.nan)
        
        return self.preprocess(peg_inv)


class BookToMarketFactor(FactorMiningBase):
    "账面市值比因子"
    
    def __init__(self, name: str = "btm"):
        super().__init__(name, FactorCategory.VALUE)
        self.config.description = "账面市值比因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        if "pb" not in data.columns:
            return pd.Series(index=data.index)
        
        btm = 1 / data["pb"].replace(0, np.nan)
        btm = btm.replace([np.inf, -np.inf], np.nan)
        
        return self.preprocess(btm)
```

### 2.3 动量因子挖掘

```python
class MomentumFactorMining(FactorMiningBase):
    "动量因子挖掘"
    
    def __init__(self, name: str = "momentum", lookback_periods: List[int] = None):
        super().__init__(name, FactorCategory.MOMENTUM)
        self.config.description = "综合动量因子"
        self.lookback_periods = lookback_periods or [5, 10, 20, 60, 120]
        self.weights = {"short": 0.3, "medium": 0.4, "long": 0.3}
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        if "close" not in data.columns:
            return pd.Series(index=data.index)
        
        returns = data["close"].pct_change()
        momentum_scores = pd.DataFrame(index=data.index)
        
        for period in [5, 10, 20]:
            momentum_scores[f"mom_{period}d"] = returns.rolling(period).sum()
        
        for period in [30, 60]:
            momentum_scores[f"mom_{period}d"] = returns.rolling(period).sum()
        
        for period in [120, 240]:
            if len(data) >= period:
                momentum_scores[f"mom_{period}d"] = returns.rolling(period).sum()
        
        valid_cols = [c for c in momentum_scores.columns if momentum_scores[c].notna().sum() > self.config.min_samples]
        
        if len(valid_cols) > 0:
            momentum = momentum_scores[valid_cols].rank(axis=1, na_option="keep").mean(axis=1)
        else:
            momentum = pd.Series(index=data.index)
        
        return self.preprocess(momentum)


class RelativeStrengthFactor(FactorMiningBase):
    "相对强弱因子 (RSI)"
    
    def __init__(self, name: str = "relative_strength"):
        super().__init__(name, FactorCategory.MOMENTUM)
        self.config.description = "相对强弱因子"
        self.lookback = 20
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        if "close" not in data.columns:
            return pd.Series(index=data.index)
        
        returns = data["close"].pct_change()
        delta = returns.fillna(0)
        
        gain = delta.where(delta > 0, 0)
        loss = (-delta).where(delta < 0, 0)
        
        avg_gain = gain.rolling(self.lookback).mean()
        avg_loss = loss.rolling(self.lookback).mean()
        
        rs = avg_gain / avg_loss.replace(0, np.nan)
        rsi = 100 - (100 / (1 + rs))
        rsi = rsi.replace([np.inf, -np.inf], np.nan)
        
        return self.preprocess(rsi)


class EarningsMomentumFactor(FactorMiningBase):
    "盈利动量因子"
    
    def __init__(self, name: str = "earnings_momentum"):
        super().__init__(name, FactorCategory.MOMENTUM)
        self.config.description = "盈利动量因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        earnings_factors = pd.DataFrame(index=data.index)
        
        if "eps" in data.columns:
            earnings_factors["eps_chg"] = data["eps"].pct_change()
        
        if "revenue" in data.columns:
            earnings_factors["revenue_chg"] = data["revenue"].pct_change()
        
        if "net_profit" in data.columns:
            earnings_factors["profit_chg"] = data["net_profit"].pct_change()
        
        valid_cols = [c for c in earnings_factors.columns if earnings_factors[c].notna().sum() > self.config.min_samples]
        
        if len(valid_cols) > 0:
            em_score = earnings_factors[valid_cols].rank(axis=1, na_option="keep").mean(axis=1)
        else:
            em_score = pd.Series(index=data.index)
        
        return self.preprocess(em_score)
```

### 2.4 质量因子挖掘

```python
class QualityFactorMining(FactorMiningBase):
    "质量因子挖掘"
    
    def __init__(self, name: str = "quality"):
        super().__init__(name, FactorCategory.QUALITY)
        self.config.description = "综合质量因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        quality_factors = pd.DataFrame(index=data.index)
        
        if all(x in data.columns for x in ["net_profit", "equity"]):
            quality_factors["roe"] = data["net_profit"] / data["equity"].replace(0, np.nan)
        
        if all(x in data.columns for x in ["net_profit", "total_assets"]):
            quality_factors["roa"] = data["net_profit"] / data["total_assets"].replace(0, np.nan)
        
        if all(x in data.columns for x in ["gross_profit", "revenue"]):
            quality_factors["gross_margin"] = data["gross_profit"] / data["revenue"].replace(0, np.nan)
        
        if all(x in data.columns for x in ["revenue", "total_assets"]):
            quality_factors["asset_turnover"] = data["revenue"] / data["total_assets"].replace(0, np.nan)
        
        valid_cols = [c for c in quality_factors.columns if quality_factors[c].notna().sum() > self.config.min_samples]
        
        if len(valid_cols) > 0:
            quality_score = quality_factors[valid_cols].rank(axis=1, na_option="keep").mean(axis=1)
        else:
            quality_score = pd.Series(index=data.index)
        
        return self.preprocess(quality_score)


class ProfitabilityFactor(FactorMiningBase):
    "盈利质量因子"
    
    def __init__(self, name: str = "profitability"):
        super().__init__(name, FactorCategory.QUALITY)
        self.config.description = "盈利质量因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        profit_factors = pd.DataFrame(index=data.index)
        
        if "net_profit" in data.columns:
            profit_vol = data["net_profit"].rolling(8).std() / data["net_profit"].rolling(8).mean().replace(0, np.nan)
            profit_factors["profit_stability"] = -profit_vol
        
        if all(x in data.columns for x in ["operating_cf", "net_profit"]):
            cf_ratio = data["operating_cf"] / data["net_profit"].replace(0, np.nan)
            profit_factors["cf_ratio"] = cf_ratio
        
        if all(x in data.columns for x in ["net_profit", "operating_cf", "total_assets"]):
            accrual = (data["net_profit"] - data["operating_cf"]) / data["total_assets"].replace(0, np.nan)
            profit_factors["accrual"] = -accrual
        
        valid_cols = [c for c in profit_factors.columns if profit_factors[c].notna().sum() > self.config.min_samples]
        
        if len(valid_cols) > 0:
            prof_score = profit_factors[valid_cols].rank(axis=1, na_option="keep").mean(axis=1)
        else:
            prof_score = pd.Series(index=data.index)
        
        return self.preprocess(prof_score)
```

### 2.5 情绪因子挖掘

```python
class SentimentFactorMining(FactorMiningBase):
    "情绪因子挖掘"
    
    def __init__(self, name: str = "sentiment"):
        super().__init__(name, FactorCategory.SENTIMENT)
        self.config.description = "综合情绪因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        sentiment_factors = pd.DataFrame(index=data.index)
        
        if all(x in data.columns for x in ["close", "volume"]):
            amount = data["close"] * data["volume"]
            sentiment_factors["money_flow"] = amount.pct_change()
            sentiment_factors["mf_ma20"] = amount.rolling(20).mean().pct_change()
        
        if "volume" in data.columns:
            vol_ma = data["volume"].rolling(20).mean()
            sentiment_factors["vol_anomaly"] = data["volume"] / vol_ma.replace(0, np.nan)
        
        valid_cols = [c for c in sentiment_factors.columns if sentiment_factors[c].notna().sum() > self.config.min_samples]
        
        if len(valid_cols) > 0:
            sentiment_score = sentiment_factors[valid_cols].rank(axis=1, na_option="keep").mean(axis=1)
        else:
            sentiment_score = pd.Series(index=data.index)
        
        return self.preprocess(sentiment_score)


class MoneyFlowFactor(FactorMiningBase):
    "资金流向因子"
    
    def __init__(self, name: str = "money_flow"):
        super().__init__(name, FactorCategory.SENTIMENT)
        self.config.description = "资金流向因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        if not all(x in data.columns for x in ["high", "low", "close", "volume"]):
            return pd.Series(index=data.index)
        
        mf_multiplier = ((data["close"] - data["low"]) - (data["high"] - data["close"])) / (data["high"] - data["low"]).replace(0, np.nan)
        mf_volume = mf_multiplier * data["volume"]
        cmf = mf_volume.rolling(20).sum() / data["volume"].rolling(20).sum()
        
        typical_price = (data["high"] + data["low"] + data["close"]) / 3
        raw_mf = typical_price * data["volume"]
        
        positive_flow = raw_mf.where(typical_price.diff() > 0, 0).rolling(14).sum()
        negative_flow = raw_mf.where(typical_price.diff() < 0, 0).rolling(14).sum()
        
        mf_ratio = positive_flow / negative_flow.replace(0, np.nan)
        mfi = 100 - (100 / (1 + mf_ratio))
        
        money_flow = cmf.rank() + mfi.rank()
        
        return self.preprocess(money_flow)
```

### 2.6 波动率因子挖掘

```python
class VolatilityFactorMining(FactorMiningBase):
    "波动率因子挖掘"
    
    def __init__(self, name: str = "volatility"):
        super().__init__(name, FactorCategory.VOLATILITY)
        self.config.description = "波动率因子"
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        vol_factors = pd.DataFrame(index=data.index)
        
        if "close" not in data.columns:
            return pd.Series(index=data.index)
        
        returns = data["close"].pct_change()
        
        for window in [10, 20, 60]:
            vol_factors[f"hvol_{window}d"] = returns.rolling(window).std() * np.sqrt(252)
        
        vol_factors["vol_skew"] = returns.rolling(20).skew()
        vol_factors["vol_kurtosis"] = returns.rolling(20).kurt()
        
        valid_cols = [c for c in vol_factors.columns if vol_factors[c].notna().sum() > self.config.min_samples]
        
        if len(valid_cols) > 0:
            vol_score = -vol_factors[valid_cols].rank(axis=1, na_option="keep").mean(axis=1)
        else:
            vol_score = pd.Series(index=data.index)
        
        return self.preprocess(vol_score)


class IdiosyncraticVolatilityFactor(FactorMiningBase):
    "特质波动率因子"
    
    def __init__(self, name: str = "idio_vol"):
        super().__init__(name, FactorCategory.VOLATILITY)
        self.config.description = "特质波动率因子"
        self.lookback = 60
        
    def calculate(self, data: pd.DataFrame) -> pd.Series:
        if not all(x in data.columns for x in ["close", "market_close"]):
            return pd.Series(index=data.index)
        
        stock_returns = data["close"].pct_change()
        market_returns = data["market_close"].pct_change()
        
        residual_vol = pd.Series(index=data.index)
        
        for i in range(self.lookback, len(data)):
            window_stock = stock_returns.iloc[i-self.lookback:i]
            window_market = market_returns.iloc[i-self.lookback:i]
            X = np.column_stack([np.ones(len(window_market)), window_market.values])
            y = window_stock.values
            try:
                coef = np.linalg.lstsq(X, y, rcond=None)[0]
                residuals = y - X @ coef
                residual_vol.iloc[i] = np.std(residuals)
            except:
                pass
        
        return self.preprocess(-residual_vol)


class RollingICCalculator:
    "滚动IC计算器"
    
    def __init__(self, window: int = 20):
        self.window = window
        self.ic_series = None
        
    def calculate(self, factor: pd.Series, returns: pd.Series) -> pd.Series:
        combined = pd.DataFrame({"factor": factor, "returns": returns}).dropna()
        ic = combined["factor"].rolling(self.window).corr(combined["returns"])
        self.ic_series = ic
        return ic
    
    def get_ir(self) -> Dict[str, float]:
        if self.ic_series is None:
            return {}
        return {
            "mean_ic": self.ic_series.mean(),
            "std_ic": self.ic_series.std(),
            "ir": self.ic_series.mean() / self.ic_series.std() if self.ic_series.std() > 0 else 0,
            "cum_ic": self.ic_series.cumsum().iloc[-1],
        }
```

class QuantileSignalGenerator(SignalGenerator):
    """分位数信号生成器"""
    
    def __init__(self, name: str = "quantile_signal"):
        super().__init__(name)
        self.params = {"long_quantile": 0.2, "short_quantile": 0.2}
    
    def generate(self, factor: pd.DataFrame, threshold: float = None) -> pd.DataFrame:
        signals = pd.DataFrame(index=factor.index, columns=factor.columns, data=0)
        for col in factor.columns:
            factor_series = factor[col]
            long_threshold = factor_series.quantile(1 - self.params["long_quantile"])
            short_threshold = factor_series.quantile(self.params["short_quantile"])
            signals[col] = np.where(factor_series >= long_threshold, 1, np.where(factor_series <= short_threshold, -1, 0))
        return signals
```

class ZScoreSignalGenerator(SignalGenerator):
    """Z-score信号生成器"""
    
    def __init__(self, name: str = "zscore_signal"):
        super().__init__(name)
        self.params = {"long_threshold": 1.5, "short_threshold": -1.5}
    
    def generate(self, factor: pd.DataFrame, threshold: float = None) -> pd.DataFrame:
        signals = pd.DataFrame(index=factor.index, columns=factor.columns, data=0)
        for col in factor.columns:
            factor_series = factor[col]
            zscore = (factor_series - factor_series.mean()) / factor_series.std()
            signals[col] = np.where(zscore >= self.params["long_threshold"], 1, np.where(zscore <= self.params["short_threshold"], -1, 0))
        return signals


class ThresholdSignalGenerator(SignalGenerator):
    """阈值信号生成器"""
    
    def __init__(self, name: str = "threshold_signal"):
        super().__init__(name)
        self.params = {"long_threshold": 0.5, "short_threshold": -0.5}
    
    def generate(self, factor: pd.DataFrame, threshold: float = None) -> pd.DataFrame:
        signals = pd.DataFrame(index=factor.index, columns=factor.columns, data=0)
        long_t = threshold if threshold is not None else self.params["long_threshold"]
        short_t = -threshold if threshold is not None else self.params["short_threshold"]
        for col in factor.columns:
            signals[col] = np.where(factor[col] >= long_t, 1, np.where(factor[col] <= short_t, -1, 0))
        return signals


class ContinuousSignalGenerator(SignalGenerator):
    """连续信号生成器"""
    
    def __init__(self, name: str = "continuous_signal"):
        super().__init__(name)
        self.params = {"min_weight": 0.0, "max_weight": 1.0}
    
    def generate(self, factor: pd.DataFrame, threshold: float = None) -> pd.DataFrame:
        signals = pd.DataFrame(index=factor.index, columns=factor.columns, data=0.0)
        for col in factor.columns:
            series = factor[col]
            min_val, max_val = series.min(), series.max()
            normalized = (series - min_val) / (max_val - min_val) if max_val > min_val else 0.5
            signals[col] = normalized * (self.params["max_weight"] - self.params["min_weight"]) + self.params["min_weight"]
        return signals


class CrossSectionalSignalGenerator(SignalGenerator):
    """横截面信号生成器"""
    
    def __init__(self, name: str = "cross_sectional_signal"):
        super().__init__(name)
        self.params = {"n_groups": 10, "long_groups": 2, "short_groups": 2}
    
    def generate(self, factor: pd.DataFrame, threshold: float = None) -> pd.DataFrame:
        signals = pd.DataFrame(index=factor.index, columns=factor.columns, data=0)
        for date in factor.index:
            ranks = pd.qcut(factor.loc[date], q=self.params["n_groups"], labels=False, duplicates="drop")
            signals.loc[date] = np.where(ranks <= self.params["short_groups"] - 1, -1, np.where(ranks >= self.params["n_groups"] - self.params["long_groups"], 1, 0))
        return signals
```

---

## 四、因子组合模块

### 4.1 因子组合基类

```python
class FactorCombinator(ABC):
    """因子组合基类"""
    
    def __init__(self, name: str = "factor_combinator"):
        self.name = name
        self.weights = None
    
    @abstractmethod
    def combine(self, factors: pd.DataFrame, returns: pd.Series = None) -> Tuple[pd.Series, np.ndarray]:
        pass


class EqualWeightCombinator(FactorCombinator):
    """等权组合器"""
    
    def combine(self, factors: pd.DataFrame, returns: pd.Series = None) -> Tuple[pd.Series, np.ndarray]:
        self.weights = np.ones(len(factors.columns)) / len(factors.columns)
        return factors.rank(axis=1, na_option="keep").mean(axis=1), self.weights


class ICWeightCombinator(FactorCombinator):
    """IC加权组合器"""
    
    def __init__(self, name: str = "ic_weight"):
        super().__init__(name)
        self.ic_values = {}
    
    def combine(self, factors: pd.DataFrame, returns: pd.Series = None) -> Tuple[pd.Series, np.ndarray]:
        if returns is None:
            return EqualWeightCombinator().combine(factors, returns)
        ic_values = [factors[col].corr(returns) if not np.isnan(factors[col].corr(returns)) else 0 for col in factors.columns]
        self.ic_values = dict(zip(factors.columns, ic_values))
        abs_ic = np.abs(ic_values)
        self.weights = abs_ic / abs_ic.sum() if abs_ic.sum() > 0 else np.ones(len(ic_values)) / len(ic_values)
        return sum(factors[col] * w for col, w in zip(factors.columns, self.weights)), self.weights


class RiskParityCombinator(FactorCombinator):
    """风险平价组合器"""
    
    def combine(self, factors: pd.DataFrame, returns: pd.Series = None) -> Tuple[pd.Series, np.ndarray]:
        vol = factors.std()
        inv_vol = 1 / vol.replace(0, np.nan)
        self.weights = inv_vol.values / inv_vol.sum()
        return factors.rank(axis=1, na_option="keep").mean(axis=1), self.weights
```

class MeanVarianceCombinator(FactorCombinator):
    """均值方差组合器"""
    
    def __init__(self, name: str = "mean_variance"):
        super().__init__(name)
        self.params = {"risk_aversion": 0.5, "min_weight": 0.0, "max_weight": 1.0}
    
    def combine(self, factors: pd.DataFrame, returns: pd.Series = None) -> Tuple[pd.Series, np.ndarray]:
        from scipy.optimize import minimize
        n = len(factors.columns)
        if returns is None:
            return EqualWeightCombinator().combine(factors, returns)
        mu = np.array([factors[col].corr(returns) for col in factors.columns])
        cov = factors.cov().values
        def objective(w):
            return -(np.dot(mu, w) - self.params["risk_aversion"] * np.dot(w, np.dot(cov, w)) / 2)
        constraints = {"type": "eq", "fun": lambda w: np.sum(w) - 1}
        bounds = [(self.params["min_weight"], self.params["max_weight"]) for _ in range(n)]
        result = minimize(objective, np.ones(n)/n, method="SLSQP", bounds=bounds, constraints=constraints)
        self.weights = result.x
        return sum(factors[col] * w for col, w in zip(factors.columns, self.weights)), self.weights


class FactorOrthogonalizer:
    """因子正交化"""
    
    def gram_schmidt(self, factors: pd.DataFrame) -> pd.DataFrame:
        ortho = pd.DataFrame(index=factors.index)
        names = factors.columns.tolist()
        ortho[names[0]] = factors[names[0]]
        for i in range(1, len(names)):
            new_f = factors[names[i]].copy()
            for prev in ortho.columns:
                coef = new_f.cov(prev) / prev.var() if prev.var() > 0 else 0
                new_f = new_f - coef * prev
            ortho[names[i]] = new_f
        return ortho
```

### 5.2 Alpha统计与分析

```python
class AlphaStatistics:
    """Alpha统计分析"""
    
    def calculate(self, alpha: pd.Series, returns: pd.Series = None) -> Dict:
        stats = {"mean": alpha.mean(), "std": alpha.std(), "sharpe": self._calculate_sharpe(alpha), "max_drawdown": self._calculate_max_drawdown(alpha), "win_rate": (alpha > 0).sum() / len(alpha.dropna())}
        if returns is not None:
            stats["corr_with_returns"] = alpha.corr(returns)
        return stats
    
    def _calculate_sharpe(self, alpha: pd.Series) -> float:
        return alpha.mean() / alpha.std() * np.sqrt(252) if alpha.std() > 0 else 0
    
    def _calculate_max_drawdown(self, alpha: pd.Series) -> float:
        cumulative = (1 + alpha).cumprod()
        running_max = cumulative.expanding().max()
        return ((cumulative - running_max) / running_max).min()


class AlphaTurnoverAnalysis:
    """Alpha换手率分析"""
    
    def calculate(self, signals: pd.DataFrame, quantile: int = 10) -> pd.Series:
        ranks = signals.rank(axis=1, ascending=False)
        turnover = ranks.diff().abs().mean(axis=1) / (quantile - 1)
        return turnover
```

---

## 七、关键概念与公式

### 7.1 核心公式

| 概念 | 公式 | 说明 |
|------|------|------|
| IC | Correlation(F, R) | 因子与收益的相关系数 |
| IR | IC_mean / IC_std | 信息比率 |
| 夏普比率 | (Rp - Rf) / σp | 风险调整收益 |
| 最大回撤 | max(Peak - Trough) / Peak | 最大跌幅 |

### 7.2 因子类别总结

| 类别 | 代表因子 | 预测方向 |
|------|----------|----------|
| 价值 | EP, BP, SP | 低估值 -> 高收益 |
| 动量 | 20日收益, RSI | 强者恒强 |
| 质量 | ROE, ROA, 盈利质量 | 优质公司 -> 高收益 |
| 情绪 | 资金流向, 分析师评级 | 正向情绪 -> 高收益 |
| 波动 | 低波动率 | 低波动 -> 高收益 |

### 7.3 系统架构图

```
+------------------+     +------------------+     +------------------+
|   因子挖掘模块   |     |   信号生成模块   |     |   因子组合模块   |
|                  |     |                  |     |                  |
| - ValueFactorMining     | - QuantileSignalGenerator     | - ICWeightCombinator     |
| - MomentumFactorMining | - ZScoreSignalGenerator        | - RiskParityCombinator    |
| - QualityFactorMining  | - CrossSectionalSignalGenerator| - MeanVarianceCombinator  |
| - SentimentFactorMining+------->   信号    +------->   组合因子   |
+------------------+     +------------------+     +------------------+
                                                        |
                                                        v
                                                +------------------+
                                                |   Alpha提取模块   |
                                                |                  |
                                                | - ReturnBasedAlphaExtractor   |
                                                | - ICBasedAlphaExtractor      |
                                                | - RiskAdjustedAlphaExtractor  |
                                                +------------------+
```

---
*文档更新: 2026-04-26*
