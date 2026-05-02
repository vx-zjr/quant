# 时间序列分析完全指南

> 量化金融领域的时间序列分析实战手册，涵盖经典统计模型到现代机器学习方法

---

## 目录

1. 时间序列分析基础
2. ARIMA模型
3. SARIMA季节模型
4. 状态空间模型与卡尔曼滤波
5. 时序预测实战
6. 异常检测
7. 波动率模型
8. 多元时间序列
9. 性能评估与模型选择
10. 实战案例：加密货币价格预测

---

## 一、时间序列分析基础

### 1.1 时间序列基本概念

时间序列是按时间顺序排列的一组观测值，广泛应用于金融、生物、气象等领域。量化交易中常见的时间序列包括价格、成交量、收益率、波动率等。

**核心特征：**
- **趋势(Trend)**：长期变化趋势
- **季节性(Seasonality)**：固定周期的循环波动
- **周期性(Cycle)**：非固定周期的波动
- **残差(Residual)**：随机波动

### 1.2 Python环境准备

```python
"""
时间序列分析环境配置
"""
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime, timedelta
import warnings
warnings.filterwarnings("ignore")

# 统计分析库
from scipy import stats
from scipy.stats import jarque_bera, shapiro, kstest
from statsmodels.tsa.stattools import adfuller, kpss, acf, pacf
from statsmodels.tsa.seasonal import seasonal_decompose, STL
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

# 模型库
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.tsa.statespace.sarimax import SARIMAX
from statsmodels.tsa.statespace.kalman_filter import KalmanFilter
from statsmodels.tsa.holtwinters import ExponentialSmoothing, SimpleExpSmoothing

# 机器学习库
from sklearn.preprocessing import StandardScaler, MinMaxScaler
from sklearn.metrics import mean_squared_error, mean_absolute_error
import itertools

# 设置中文显示
plt.rcParams["font.sans-serif"] = ["SimHei", "DejaVu Sans"]
plt.rcParams["axes.unicode_minus"] = False

print("环境配置完成!")
print(f"NumPy版本: {np.__version__}")
print(f"Pandas版本: {pd.__version__}")
```

### 1.3 生成模拟时间序列数据

```python
def generate_financial_timeseries(
    start_date: str = "2020-01-01",
    periods: int = 500,
    initial_price: float = 100.0,
    mu: float = 0.0002,
    sigma: float = 0.02,
    trend: float = 0.0001,
    seasonality_amplitude: float = 0.01,
    seasonality_period: int = 20,
    seed: int = 42
) -> pd.DataFrame:
    """生成模拟金融市场时间序列数据"""
    np.random.seed(seed)
    dates = pd.bdate_range(start=start_date, periods=periods)
    random_returns = np.random.normal(mu, sigma, periods)
    trend_component = trend * np.arange(periods)
    seasonal_component = seasonality_amplitude * np.sin(
        2 * np.pi * np.arange(periods) / seasonality_period)
    combined_returns = random_returns + trend_component + seasonal_component
    prices = initial_price * np.exp(np.cumsum(combined_returns))
    data = {
        "date": dates,
        "close": prices,
        "open": prices * (1 + np.random.uniform(-0.005, 0.005, periods)),
        "high": prices * (1 + np.abs(np.random.uniform(0, 0.01, periods))),
        "low": prices * (1 - np.abs(np.random.uniform(0, 0.01, periods))),
        "volume": np.random.uniform(1000000, 5000000, periods)
    }
    df = pd.DataFrame(data).set_index("date")
    df["returns"] = df["close"].pct_change()
    df["log_returns"] = np.log(df["close"] / df["close"].shift(1))
    df["rolling_mean"] = df["close"].rolling(window=20).mean()
    df["rolling_std"] = df["close"].rolling(window=20).std()
    df["rolling_vol"] = df["returns"].rolling(window=20).std() * np.sqrt(252)
    return df.dropna()

def generate_seasonal_timeseries(
    start_date: str = "2020-01-01",
    periods: int = 730,
    freq: str = "D"
) -> pd.DataFrame:
    """生成带季节性模式的时间序列"""
    np.random.seed(123)
    dates = pd.date_range(start=start_date, periods=periods, freq=freq) if freq == "D" else pd.bdate_range(start=start_date, periods=periods)
    t = np.arange(periods)
    trend = 0.02 * t / periods
    annual_seasonal = 0.15 * np.sin(2 * np.pi * t / 365)
    day_of_week = dates.dayofweek
    weekly_seasonal = 0.05 * (day_of_week < 5).astype(float)
    noise = np.random.normal(0, 0.03, periods)
    value = 100 + trend * 100 + annual_seasonal * 100 + weekly_seasonal + noise
    df = pd.DataFrame({"date": dates, "value": value}).set_index("date")
    return df
```

### 1.4 平稳性检验

**平稳性**是时间序列分析的核心概念。只有平稳序列才能使用大多数统计模型。

```python
def test_stationarity(
    series: pd.Series,
    max_lags: int = None
) -> dict:
    """综合平稳性检验：ADF检验和KPSS检验"""
    results = {}
    adf_result = adfuller(series.dropna(), maxlag=max_lags, autolag="AIC")
    results["ADF"] = {
        "statistic": adf_result[0],
        "p_value": adf_result[1],
        "lags_used": adf_result[2],
        "n_obs": adf_result[3],
        "critical_values": adf_result[4],
        "is_stationary": adf_result[1] < 0.05
    }
    try:
        kpss_result = kpss(series.dropna(), regression="c", nlags="auto")
        results["KPSS"] = {
            "statistic": kpss_result[0],
            "p_value": kpss_result[1],
            "is_stationary": kpss_result[1] > 0.05
        }
    except:
        results["KPSS"] = {"error": "KPSS检验失败"}
    adf_stationary = results["ADF"]["is_stationary"]
    kpss_stationary = results.get("KPSS", {}).get("is_stationary", None)
    if adf_stationary and kpss_stationary:
        results["conclusion"] = "平稳"
    elif not adf_stationary and not kpss_stationary:
        results["conclusion"] = "非平稳(需差分)"
    else:
        results["conclusion"] = "差分平稳"
    return results
```

### 1.5 时间序列分解

```python
def decompose_timeseries(
    series: pd.Series,
    model: str = "additive",
    period: int = None,
    method: str = "classical"
) -> dict:
    """时间序列分解：经典分解和STL分解"""
    if method == "STL":
        stl = STL(series, period=period or 12, robust=True)
        decomposition = stl.fit()
        return {
            "trend": decomposition.trend,
            "seasonal": decomposition.seasonal,
            "residual": decomposition.resid,
            "method": "STL"
        }
    else:
        decomposition = seasonal_decompose(
            series.dropna(), model=model, period=period or 12)
        return {
            "trend": decomposition.trend,
            "seasonal": decomposition.seasonal,
            "residual": decomposition.resid,
            "observed": decomposition.observed,
            "method": "Classical"
        }
```

---\
\\
## 二、ARIMA模型
\\
### 2.1 ARIMA模型原理
\\
ARIMA (AutoRegressive Integrated Moving Average) 是时间序列预测中最经典的模型之一。
\\
参数说明:
- **p**: AR(自回归)阶数
- **d**: I(差分)阶数
- **q**: MA(移动平均)阶数
### 2.2 确定ARIMA阶数

```python
def find_optimal_arima_order(series, max_p=5, max_d=2, max_q=5, ic="aicc"):
    results = []
    for p, d, q in itertools.product(range(max_p+1), range(max_d+1), range(max_q+1)):
        try:
            model = ARIMA(series, order=(p, d, q))
            fitted = model.fit(disp=False)
            results.append({"order": (p,d,q), ic: getattr(fitted, ic.upper()), "aic": fitted.aic})
        except:
            continue
    return pd.DataFrame(results).sort_values(by=ic.upper()).iloc[0]["order"]
```

### 2.3 ARIMAModel封装类

```python
class ARIMAModel:
    def __init__(self, order=(1,1,1)):
        self.order = order
        self.model = None
        self.fitted_model = None
    def fit(self, series, display_summary=True):
        self.model = ARIMA(series, order=self.order)
        self.fitted_model = self.model.fit(disp=False)
        self.residuals = self.fitted_model.resid
        if display_summary:
            print(self.fitted_model.summary())
        return self
    def forecast(self, steps=10, alpha=0.05):
        fr = self.fitted_model.get_forecast(steps=steps)
        ci = fr.conf_int(alpha=alpha)
        return pd.DataFrame({"forecast": fr.predicted_mean, "lower": ci.iloc[:,0], "upper": ci.iloc[:,1]})
    def diagnostics(self):
        self.fitted_model.plot_diagnostics(figsize=(14, 10))
        plt.show()
    def residual_tests(self):
        from statsmodels.stats.diagnostic import acorr_ljungbox
        lb = acorr_ljungbox(self.residuals.dropna(), lags=[10], return_df=True)
        jb_stat, jb_pvalue, skew, kurtosis = jarque_bera(self.residuals.dropna())
        return {"ljung_box_pvalue": lb.iloc[0,1], "jarque_bera_pvalue": jb_pvalue}
```

### 2.4 滚动预测

```python
def rolling_forecast(series, train_size=None, order=(1,1,1), horizon=1):
    if train_size is None:
        train_size = len(series) // 2
    predictions, actuals, dates = [], [], []
    for i in range(train_size, len(series) - horizon + 1):
        try:
            train = series.iloc[:i]
            model = ARIMA(train, order=order).fit(disp=False)
            forecast = model.forecast(steps=horizon)
            predictions.append(forecast.iloc[-1])
            actuals.append(series.iloc[i + horizon - 1])
            dates.append(series.index[i + horizon - 1])
        except:
            continue
    return pd.DataFrame({"actual": actuals, "predicted": predictions}, index=dates)
```
---

## 三、SARIMA季节模型

### 3.1 SARIMA模型原理

**SARIMA** (Seasonal ARIMA) 在ARIMA基础上加入季节性项:

SARIMA(p,d,q)(P,D,Q)_m

其中:
- **p,d,q**: 非季节性AR、差分、MA阶数
- **P,D,Q**: 季节性AR、差分、MA阶数
- **m**: 季节周期长度(年数据m=4, 月数据m=12, 周数据m=52, 日数据m=5)

### 3.2 季节性检测

```python
def detect_seasonality(series, max_period=365):
    max_lags = min(max_period, len(series) // 2 - 1)
    acf_values = acf(series.dropna(), nlags=max_lags, fft=True)
    threshold = 1.96 / np.sqrt(len(series))
    peaks = []
    for i in range(1, len(acf_values) - 1):
        if acf_values[i] > threshold and acf_values[i] > acf_values[i-1] and acf_values[i] > acf_values[i+1]:
            peaks.append({"lag": i, "acf": acf_values[i]})
    common_periods = [5, 7, 12, 21, 52, 252]
    for peak in peaks:
        for period in common_periods:
            if abs(peak["lag"] - period) <= 2:
                return period
    return 1
```

### 3.3 SARIMA模型拟合

```python
class SARIMAModel:
    def __init__(self, order, seasonal_order):
        self.order = order
        self.seasonal_order = seasonal_order
        self.model = None
        self.fitted_model = None
    def fit(self, series, display_summary=True):
        self.model = SARIMAX(series, order=self.order, seasonal_order=self.seasonal_order,
                            enforce_stationarity=False, enforce_invertibility=False)
        self.fitted_model = self.model.fit(disp=False, maxiter=500)
        if display_summary:
            print(self.fitted_model.summary())
        return self
    def forecast(self, steps=1, alpha=0.05):
        fr = self.fitted_model.get_forecast(steps=steps)
        ci = fr.conf_int(alpha=alpha)
        return pd.DataFrame({"forecast": fr.predicted_mean, "lower": ci.iloc[:,0], "upper": ci.iloc[:,1]})
```

### 3.4 SARIMA参数搜索

```python
def find_optimal_sarima_order(series, max_p=2, max_d=1, max_q=2, max_P=1, max_D=1, max_Q=1, m=12, ic="aicc"):
    results = []
    for p, d, q in itertools.product(range(max_p+1), range(max_d+1), range(max_q+1)):
        for P, D, Q in itertools.product(range(max_P+1), range(max_D+1), range(max_Q+1)):
            try:
                model = SARIMAX(series, order=(p,d,q), seasonal_order=(P,D,Q,m))
                fitted = model.fit(disp=False, maxiter=200)
                results.append({"order": (p,d,q), "sorder": (P,D,Q,m),
                            ic: getattr(fitted, ic.upper()), "aic": fitted.aic})
            except:
                continue
    if not results:
        return None
    df = pd.DataFrame(results).sort_values(by=ic.upper()).reset_index(drop=True)
    return df.iloc[0]
```
---

## 四、状态空间模型与卡尔曼滤波

### 4.1 状态空间模型基础

状态空间模型将时间序列表示为:

**观测方程:** y_t = Z_t alpha_t + d_t + epsilon_t

**状态方程:** alpha_{t+1} = T_t alpha_t + c_t + R_t eta_t

### 4.2 卡尔曼滤波实现

```python
class KalmanFilter:
    def __init__(self, F=None, H=None, Q=None, R=None, x0=None, P0=None):
        self.F = F
        self.H = H
        self.Q = Q
        self.R = R
        self.x = x0
        self.P = P0
    def filter(self, y):
        n = len(y)
        dim = len(self.x) if self.x is not None else 1
        x_filt = np.zeros((n, dim))
        P_filt = np.zeros((n, dim, dim))
        x = self.x.copy() if self.x is not None else np.zeros(dim)
        P = self.P.copy() if self.P is not None else np.eye(dim)
        for t in range(n):
            if self.F is not None:
                x_pred = self.F @ x
                P_pred = self.F @ P @ self.F.T + self.R
            else:
                x_pred, P_pred = x, P
            if self.H is not None:
                y_hat = self.H @ x_pred
                innovation = y[t] - y_hat
                S = self.H @ P_pred @ self.H.T + self.Q
                K = P_pred @ self.H.T / S
                x = x_pred + K * innovation
                P = (np.eye(dim) - np.outer(K, self.H)) @ P_pred
            else:
                x, P = x_pred, P_pred
            x_filt[t] = x
            P_filt[t] = P
        return x_filt, P_filt
```

### 4.3 本地水平模型

```python
def fit_local_level_model(series, level_var=None, obs_var=None):
    y = series.values.astype(float)
    if level_var is None:
        level_var = np.var(np.diff(y)) * 0.1
    if obs_var is None:
        obs_var = np.var(y) * 0.1
    F = np.array([[1.0]])
    H = np.array([[1.0]])
    Q = np.array([[level_var]])
    R = np.array([[obs_var]])
    kf = KalmanFilter(F=F, H=H, Q=Q, R=R, x0=np.array([y[0]]), P0=np.array([[1.0]]))
    x_filt, P_filt = kf.filter(y)
    return pd.DataFrame({"level": x_filt.flatten(), "std": np.sqrt(P_filt.flatten())}, index=series.index)
```

### 4.4 时变AR系数跟踪

```python
def kalman_ar_tracking(series, window=60):
    y = series.values.astype(float)
    n = len(y)
    results = []
    for t in range(window, n):
        y_train = y[t - window:t]
        X = np.column_stack([np.ones(window), y_train[:-1]])
        y_vec = y_train[1:]
        try:
            beta = np.linalg.lstsq(X, y_vec, rcond=None)[0]
            residuals = y_vec - X @ beta
            results.append({
                "date": series.index[t],
                "intercept": beta[0],
                "ar_coef": beta[1],
                "sigma": np.sqrt(np.var(residuals))
            })
        except:
            continue
    return pd.DataFrame(results).set_index("date")
```
---

## 五、时序预测实战

### 5.1 指数平滑方法

```python
def exponential_smoothing_forecast(series, trend=None, seasonal=None, period=12):
    if trend and seasonal:
        model = ExponentialSmoothing(series, trend=trend, seasonal=seasonal, seasonal_periods=period)
    elif trend:
        model = ExponentialSmoothing(series, trend=trend)
    else:
        model = SimpleExpSmoothing(series)
    fitted = model.fit()
    return fitted
```

### 5.2 双指数平滑

```python
def double_exponential_smoothing(series, alpha=0.3, beta=0.1, forecast_periods=10):
    y = series.values.astype(float)
    level = y[0]
    trend = y[1] - y[0]
    for t in range(1, len(y)):
        prev_level = level
        level = alpha * y[t] + (1 - alpha) * (prev_level + trend)
        trend = beta * (level - prev_level) + (1 - beta) * trend
    forecasts = [level + h * trend for h in range(1, forecast_periods + 1)]
    return pd.DataFrame({"horizon": range(1, forecast_periods + 1), "forecast": forecasts})
```

### 5.3 多步预测策略

```python
def multi_step_forecast(series, order=(1,1,1), steps=[1, 5, 10, 20], method="direct"):
    results = {}
    for step in steps:
        predictions = []
        actuals = []
        train_size = len(series) // 2
        for i in range(train_size, len(series) - step):
            train = series.iloc[:i]
            try:
                model = ARIMA(train, order=order).fit(disp=False)
                forecast = model.forecast(steps=step)
                predictions.append(forecast.iloc[-1])
                actuals.append(series.iloc[i + step - 1])
            except:
                continue
        if predictions:
            results[f"step_{step}"] = {
                "rmse": np.sqrt(mean_squared_error(actuals, predictions)),
                "mae": mean_absolute_error(actuals, predictions)
            }
    return results
```

### 5.4 集成预测

```python
class EnsembleForecast:
    def __init__(self, models=None, weights=None):
        self.models = models or []
        self.weights = weights
        self.fitted_models = []
    def fit(self, series, orders=[(1,1,1), (2,1,1), (1,1,2)]):
        for order in orders:
            try:
                model = ARIMA(series, order=order).fit(disp=False)
                self.fitted_models.append({"order": order, "model": model})
            except:
                pass
        if self.weights is None:
            self.weights = [1.0 / len(self.fitted_models)] * len(self.fitted_models)
    def predict(self, steps=10):
        forecasts = []
        for fm in self.fitted_models:
            forecast = fm["model"].forecast(steps=steps)
            forecasts.append(forecast.values)
        ensemble = np.average(forecasts, axis=0, weights=self.weights)
        lower = np.percentile(np.array(forecasts), 5, axis=0)
        upper = np.percentile(np.array(forecasts), 95, axis=0)
        return pd.DataFrame({"forecast": ensemble, "lower_95": lower, "upper_95": upper, "std": np.std(forecasts, axis=0)})
```
---

## 六、异常检测

### 6.1 基于统计的异常检测

```python
def detect_outliers_zscore(series, threshold=3.0):
    mean = series.mean()
    std = series.std()
    z_scores = np.abs((series - mean) / std)
    return pd.DataFrame({"value": series, "z_score": z_scores, "is_outlier": z_scores > threshold})
```

```python
def detect_outliers_iqr(series, multiplier=1.5):
    q1 = series.quantile(0.25)
    q3 = series.quantile(0.75)
    iqr = q3 - q1
    lower = q1 - multiplier * iqr
    upper = q3 + multiplier * iqr
    return pd.DataFrame({"value": series, "lower_bound": lower, "upper_bound": upper,
                        "is_outlier": (series < lower) | (series > upper)})
```

```python
def rolling_outlier_detection(series, window=20, threshold=3.0, min_periods=10):
    rolling_mean = series.rolling(window=window, min_periods=min_periods).mean()
    rolling_std = series.rolling(window=window, min_periods=min_periods).std()
    z_scores = np.abs((series - rolling_mean) / rolling_std)
    return pd.DataFrame({"value": series, "rolling_mean": rolling_mean, "z_score": z_scores,
                        "is_outlier": z_scores > threshold})
```

### 6.2 基于模型的异常检测

```python
def detect_outliers_arima(series, order=(1,1,1), threshold_sigma=3.0):
    model = ARIMA(series, order=order)
    fitted = model.fit(disp=False)
    residuals = fitted.resid
    resid_mean = residuals.mean()
    resid_std = residuals.std()
    z_resid = (residuals - resid_mean) / resid_std
    return pd.DataFrame({"value": series.iloc[-len(residuals):], "fitted": fitted.fittedvalues,
                        "residual": residuals, "z_residual": z_resid,
                        "is_outlier": np.abs(z_resid) > threshold_sigma})
```

```python
def detect_outliers_esd(series, max_outliers=5, alpha=0.05):
    y = series.values.copy()
    outliers = []
    for i in range(max_outliers):
        if len(y) < 3:
            break
        mean = np.mean(y)
        std = np.std(y)
        if std == 0:
            break
        esd_values = np.abs((y - mean) / std)
        max_idx = np.argmax(esd_values)
        from scipy.stats import t
        df = len(y) - 2
        t_critical = t.ppf(1 - alpha / (2 * len(y)), df)
        lambda_critical = ((len(y) - 1) * t_critical) / np.sqrt(len(y) * (1 + t_critical**2))
        if esd_values[max_idx] > lambda_critical:
            outliers.append({"index": max_idx, "value": y[max_idx], "esd": esd_values[max_idx]})
            y = np.delete(y, max_idx)
        else:
            break
    return outliers
```

### 6.3 异常检测可视化

```python
def plot_outliers(series, outliers, title="异常检测结果", figsize=(14, 8)):
    fig, axes = plt.subplots(2, 1, figsize=figsize)
    axes[0].plot(series.index, series.values, "b-", linewidth=0.8, label="正常值")
    if "is_outlier" in outliers.columns:
        outlier_mask = outliers["is_outlier"]
        if outlier_mask.any():
            axes[0].scatter(outliers.index[outlier_mask], outliers.loc[outlier_mask, "value"],
                        c="red", s=50, marker="o", label="异常值", zorder=5)
    axes[0].set_title(title, fontsize=12, fontweight="bold")
    axes[0].legend()
    axes[0].grid(True, alpha=0.3)
    if "residual" in outliers.columns:
        axes[1].scatter(outliers.index, outliers["residual"], alpha=0.6)
        axes[1].axhline(y=0, color="black", linestyle="-", linewidth=0.5)
        axes[1].set_title("残差分布", fontsize=12, fontweight="bold")
        axes[1].grid(True, alpha=0.3)
    plt.tight_layout()
    plt.show()
```
---

## 七、波动率模型

### 7.1 ARCH/GARCH模型

```python
def fit_garch_model(returns, p=1, q=1, mean_model="constant", vol_model="GARCH", dist="normal"):
    try:
        from arch import arch_model
        model = arch_model(returns * 100, mean=mean_model, vol=vol_model, p=p, q=q, dist=dist)
        result = model.fit(disp="off", show_warning=False)
        return {
            "model": result,
            "params": result.params,
            "aic": result.aic,
            "bic": result.bic,
            "conditional_vol": result.conditional_volatility,
            "residuals": result.resid
        }
    except ImportError:
        print("请安装arch库: pip install arch")
        return None
```

### 7.2 滚动GARCH预测

```python
def rolling_garch_forecast(returns, window=252, p=1, q=1):
    from arch import arch_model
    volatility = []
    for i in range(window, len(returns)):
        try:
            train_returns = returns.iloc[i - window:i] * 100
            model = arch_model(train_returns, vol="GARCH", p=p, q=q)
            result = model.fit(disp="off")
            cond_var = result.params["omega"] + result.params["alpha[1]"] * (train_returns.iloc[-1]**2)
            volatility.append({
                "date": returns.index[i],
                "cond_vol": np.sqrt(cond_var) / 100,
                "realized_vol": returns.iloc[i - 1:i].std()
            })
        except:
            continue
    return pd.DataFrame(volatility).set_index("date")
```

### 7.3 波动率可视化

```python
def plot_volatility_forecast(garch_result, title="GARCH波动率预测"):
    fig, axes = plt.subplots(2, 1, figsize=(14, 8))
    vol = garch_result["conditional_vol"] / 100
    axes[0].plot(vol.index, vol.values, "b-", linewidth=0.8)
    axes[0].set_title(f"{title} - 条件波动率", fontsize=12, fontweight="bold")
    axes[0].set_ylabel("波动率")
    axes[0].grid(True, alpha=0.3)
    vol_squared = garch_result["residuals"]**2
    axes[1].plot(vol_squared.index, vol_squared.values, "r-", linewidth=0.5, alpha=0.7)
    axes[1].set_title("收益率平方(波动率聚类)", fontsize=12, fontweight="bold")
    axes[1].set_ylabel("收益率平方")
    axes[1].grid(True, alpha=0.3)
    plt.tight_layout()
    plt.show()
```

---

## 八、多元时间序列

### 8.1 向量自回归(VAR)模型

```python
def fit_var_model(data, maxlags=5, ic="aic"):
    from statsmodels.tsa.api import VAR
    model = VAR(data)
    lag_order = model.select_order(maxlags=maxlags, deterministic="const")
    optimal_lag = getattr(lag_order, ic)
    print(f"最优滞后阶数: {optimal_lag}")
    fitted = model.fit(optimal_lag, trend="c")
    return {"model": fitted, "lag_order": optimal_lag}
```

### 8.2 VAR预测

```python
def var_forecast(var_model, data, steps=10):
    forecast = var_model.forecast(data.values[-var_model.k_ar:], steps=steps)
    return pd.DataFrame(forecast, index=pd.date_range(start=data.index[-1], periods=steps+1, freq="B")[1:],
                       columns=data.columns)
```

### 8.3 Granger因果检验

```python
def granger_causality_test(data, cause_col, effect_col, maxlag=5):
    from statsmodels.tsa.stattools import grangercausalitytests
    test_data = pd.concat([data[effect_col], data[cause_col]], axis=1)
    test_data.columns = [effect_col, cause_col]
    result = grangercausalitytests(test_data, maxlag=maxlag, verbose=False)
    p_values = []
    for lag in range(1, maxlag + 1):
        if lag in result[lag][0].keys():
            p_value = result[lag][0][lag][0]["ssr_ftest"][1]
            p_values.append({"lag": lag, "p_value": p_value})
    return pd.DataFrame(p_values)
```
---

## 九、性能评估与模型选择

### 9.1 预测误差指标

```python
def calculate_metrics(actual, predicted):
    mae = np.mean(np.abs(actual - predicted))
    mse = np.mean((actual - predicted) ** 2)
    rmse = np.sqrt(mse)
    mape = np.mean(np.abs((actual - predicted) / actual)) * 100
    smape = np.mean(2 * np.abs(actual - predicted) / (np.abs(actual) + np.abs(predicted))) * 100
    ss_res = np.sum((actual - predicted) ** 2)
    ss_tot = np.sum((actual - np.mean(actual)) ** 2)
    r2 = 1 - ss_res / ss_tot
    n = len(actual)
    r2_adj = 1 - (1 - r2) * (n - 1) / (n - 2)
    naive_error = np.mean(np.abs(np.diff(actual)))
    mase = mae / naive_error if naive_error > 0 else np.nan
    return {
        "MAE": mae, "MSE": mse, "RMSE": rmse, "MAPE": mape, "SMAPE": smape,
        "R2": r2, "R2_Adjusted": r2_adj, "MASE": mase
    }
```

### 9.2 模型比较

```python
def compare_models(series, models_config, train_size=None, test_size=30):
    if train_size is None:
        train_size = len(series) - test_size
    train = series.iloc[:train_size]
    test = series.iloc[train_size:train_size + test_size]
    results = []
    for config in models_config:
        name = config["name"]
        model_type = config["type"]
        params = config.get("params", {})
        try:
            if model_type == "ARIMA":
                order = params.get("order", (1, 1, 1))
                model = ARIMA(train, order=order).fit(disp=False)
                forecast = model.forecast(steps=test_size)
            elif model_type == "SES":
                model = SimpleExpSmoothing(train).fit()
                forecast = model.forecast(test_size)
            metrics = calculate_metrics(test.values, forecast.values)
            metrics["model"] = name
            results.append(metrics)
        except Exception as e:
            print(f"{name} 拟合失败: {e}")
    return pd.DataFrame(results).set_index("model")
```

### 9.3 模型比较可视化

```python
def plot_model_comparison(results_df, metric="RMSE"):
    fig, axes = plt.subplots(1, 2, figsize=(14, 5))
    results_df[metric].plot(kind="bar", ax=axes[0], color="steelblue")
    axes[0].set_title(f"{metric} 比较", fontsize=12, fontweight="bold")
    axes[0].set_ylabel(metric)
    axes[0].tick_params(axis="x", rotation=45)
    axes[0].grid(True, alpha=0.3, axis="y")
    plt.tight_layout()
    plt.show()
```
---

## 十、实战案例：加密货币价格预测

### 10.1 数据获取与预处理

```python
def load_crypto_data(symbol="BTCUSDT", start_date="2020-01-01", interval="1d"):
    try:
        import ccxt
        exchange = ccxt.binance()
        ohlcv = exchange.fetch_ohlcv(symbol, timeframe=interval,
                             since=exchange.parse8601(start_date), limit=1000)
        df = pd.DataFrame(ohlcv, columns=["timestamp", "open", "high", "low", "close", "volume"])
        df["timestamp"] = pd.to_datetime(df["timestamp"], unit="ms")
        df = df.set_index("timestamp")
        df["returns"] = df["close"].pct_change()
        return df.dropna()
    except ImportError:
        return None
```

### 10.2 生成模拟加密货币数据

```python
def simulate_crypto_data(start_price=10000, start_date="2020-01-01", periods=500, volatility=0.03):
    np.random.seed(42)
    dates = pd.bdate_range(start=start_date, periods=periods)
    drift = 0.0005
    price_changes = np.random.normal(drift, volatility, periods)
    halving_cycle = 0.1 * np.sin(2 * np.pi * np.arange(periods) / (4 * 365))
    price_changes += halving_cycle
    prices = start_price * np.exp(np.cumsum(price_changes))
    data = {
        "date": dates, "close": prices,
        "open": prices * (1 + np.random.uniform(-0.01, 0.01, periods)),
        "high": prices * (1 + np.abs(np.random.uniform(0, 0.02, periods))),
        "low": prices * (1 - np.abs(np.random.uniform(0, 0.02, periods))),
        "volume": np.random.uniform(1000, 10000, periods) * prices
    }
    df = pd.DataFrame(data).set_index("date")
    df["returns"] = df["close"].pct_change()
    return df.dropna()
```

### 10.3 完整预测流程

```python
def crypto_price_forecast_pipeline(data, target_col="close", test_size=30):
    series = data[target_col].copy()
    train = series.iloc[:-test_size]
    test = series.iloc[-test_size:]
    best_order = find_optimal_arima_order(series, max_p=3, max_d=1, max_q=3) or (1,1,1)
    model = ARIMAModel(order=best_order)
    model.fit(train, display_summary=False)
    forecast = model.forecast(steps=test_size)
    metrics = calculate_metrics(test.values, forecast["forecast"].values)
    return {"order": best_order, "forecast": forecast, "test": test, "metrics": metrics}
```
### 10.4 预测结果可视化

```python
def plot_forecast_results(train, test, forecast, order):
    fig, axes = plt.subplots(2, 1, figsize=(14, 10))
    axes[0].plot(train.index, train.values, "b-", linewidth=1, label="训练集")
    axes[0].plot(test.index, test.values, "g-", linewidth=1.5, label="实际值")
    axes[0].plot(forecast.index, forecast["forecast"], "r--", linewidth=1.5, label="预测")
    axes[0].fill_between(forecast.index, forecast["lower"], forecast["upper"],
                        color="red", alpha=0.2, label="95%置信区间")
    axes[0].axvline(x=test.index[0], color="gray", linestyle="--", alpha=0.7)
    axes[0].set_title(f"ARIMA{order} 价格预测", fontsize=14, fontweight="bold")
    axes[0].legend()
    axes[0].grid(True, alpha=0.3)
    residuals = test.values - forecast["forecast"].values
    axes[1].bar(test.index, residuals, color="steelblue", alpha=0.7)
    axes[1].axhline(y=0, color="black", linestyle="-", linewidth=0.5)
    axes[1].set_title("预测残差", fontsize=12, fontweight="bold")
    axes[1].grid(True, alpha=0.3)
    plt.tight_layout()
    plt.show()
```

### 10.5 主程序入口

```python
if __name__ == "__main__":
    crypto_data = simulate_crypto_data(start_price=10000, periods=500, volatility=0.03)
    results = crypto_price_forecast_pipeline(crypto_data, test_size=30)
    train = crypto_data["close"].iloc[:-30]
    test = crypto_data["close"].iloc[-30:]
    plot_forecast_results(train, test, results["forecast"], results["order"])
```

---

## 附录A: 常用函数速查表

| 功能 | 函数 |
|------|------|
| 平稳性检验 | `test_stationarity()` |
| 时间序列分解 | `decompose_timeseries()` |
| ARIMA拟合 | `ARIMAModel.fit()` |
| ARIMA预测 | `ARIMAModel.forecast()` |
| SARIMA拟合 | `SARIMAModel.fit()` |
| 参数搜索 | `find_optimal_arima_order()` |
| 滚动预测 | `rolling_forecast()` |
| 异常检测 | `detect_outliers_arima()` |
| 波动率模型 | `fit_garch_model()` |
| 模型评估 | `calculate_metrics()` |

## 附录B: 常用参数设置

| 场景 | 推荐参数 |
|------|----------|
| 日线股票价格 | ARIMA(1,1,1), SARIMA(1,1,1)(1,0,1,5) |
| 分钟线数据 | ARIMA(2,1,2), m=390 |
| 高波动市场 | GARCH(1,1) |
| 季节性销售数据 | SARIMA(1,1,1)(1,1,1,12) |
| 波动率预测 | GARCH(1,1), EGARCH |

## 附录C: 参考资源

### 书籍
- Time Series Analysis by Hamilton
- Forecasting: Principles and Practice by Hyndman
- Analysis of Financial Time Series by Tsay

### Python库
- statsmodels: 统计模型
- arch: GARCH模型
- Prophet: Facebook时序预测

---

*最后更新: 2026-04-26*

*本指南旨在提供时间序列分析的全面参考，实际应用中请根据数据特点选择合适的模型和参数。*