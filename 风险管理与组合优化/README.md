# 风险管理与组合优化

## 目录
- [市场风险管理](./市场风险管理.md)
- [信用风险管理](./信用风险管理.md)
- [组合优化方法](./组合优化方法.md)
- [尾部风险管理](./尾部风险管理.md)

---

## 一、市场风险管理

### VaR模型
| 模型 | 公式 | 特点 |
|------|------|------|
| 方差-协方差 | VaR = μ + σ×z_α | 正态分布假设 |
| 历史模拟 | 基于历史数据分位数 | 非参数 |
| Monte Carlo | 随机模拟 | 灵活但计算量大 |

### VaR计算
```python
import numpy as np
import pandas as pd
from scipy import stats

def calculate_var_historical(returns, confidence=0.95):
    """
    历史模拟法VaR
    """
    return np.percentile(returns, (1 - confidence) * 100)

def calculate_var_parametric(returns, confidence=0.95):
    """
    参数法VaR (方差-协方差法)
    """
    mu = returns.mean()
    sigma = returns.std()
    z = stats.norm.ppf(1 - confidence)
    return mu + sigma * z

def calculate_var_monte_carlo(returns, n_simulations=10000, 
                               confidence=0.95):
    """
    Monte Carlo模拟VaR
    """
    mu = returns.mean()
    sigma = returns.std()
    simulated_returns = np.random.normal(mu, sigma, n_simulations)
    return np.percentile(simulated_returns, (1 - confidence) * 100)
```

### CVaR/ES计算
```python
def calculate_cvar(returns, confidence=0.95):
    """
    计算条件VaR (Expected Shortfall)
    """
    var = calculate_var_historical(returns, confidence)
    return returns[returns <= var].mean()
```

---

## 二、信用风险管理

### 信用风险模型
| 模型 | 描述 | 应用 |
|------|------|------|
| CreditMetrics | 信用度量迁移模型 | 信用资产组合 |
| CreditRisk+ | 违约率模型 | 保险精算方法 |
| KMV-Merton | 违约距离模型 | 上市公司信用 |
| Altman Z-Score | Z分数模型 | 财务困境预测 |

### 违约概率计算
```python
def kmv_merton.default_probability(equity, debt, r, sigma_e, T=1):
    """
    KMV-Merton模型计算违约概率
    
    参数:
    equity: 股权价值
    debt: 债务面值
    r: 无风险利率
    sigma_e: 股权波动率
    T: 时间期限
    """
    # 资产价值
    V = equity + debt * np.exp(-r * T)
    
    # 资产波动率
    sigma_v = sigma_e * equity / V
    
    # 违约距离
    dd = (np.log(V / debt) + (r - 0.5 * sigma_v**2) * T) / (sigma_v * np.sqrt(T))
    
    # 违约概率
    return stats.norm.cdf(-dd)
```

---

## 三、组合优化方法

### 现代投资组合理论
```python
def mean_variance_optimization(returns, target_return=None):
    """
    均值-方差优化
    
    参数:
    returns: 收益率矩阵 (n_samples x n_assets)
    target_return: 目标收益率
    """
    cov_matrix = np.cov(returns.T)
    mean_returns = returns.mean()
    n_assets = len(mean_returns)
    
    # 无风险利率
    rf = 0.03
    
    # 有效前沿计算
    if target_return is None:
        # 最大化夏普比率
        cov_inv = np.linalg.inv(cov_matrix)
        ones = np.ones(n_assets)
        numerator = cov_inv @ (mean_returns - rf * ones)
        denominator = ones @ cov_inv @ (mean_returns - rf * ones)
        weights = numerator / denominator
        
        # 计算组合收益和波动率
        portfolio_return = weights @ mean_returns
        portfolio_vol = np.sqrt(weights @ cov_matrix @ weights)
        sharpe = (portfolio_return - rf) / portfolio_vol
        
        return weights, portfolio_return, portfolio_vol, sharpe
    else:
        # 给定目标收益率下的最小方差组合
        # 使用二次规划
        from scipy.optimize import minimize
        
        def portfolio_volatility(weights):
            return np.sqrt(weights @ cov_matrix @ weights)
        
        constraints = [
            {'type': 'eq', 'fun': lambda w: np.sum(w) - 1},
            {'type': 'eq', 'fun': lambda w: w @ mean_returns - target_return}
        ]
        bounds = [(0, 1) for _ in range(n_assets)]
        
        result = minimize(portfolio_volatility, np.ones(n_assets)/n_assets,
                         method='SLSQP', bounds=bounds, constraints=constraints)
        return result.x, result.fun
```

### Black-Litterman模型
```python
def black_litterman(market_cap_weights, cov_matrix, views=None, P=None, Q=None, omega=None, tau=0.05):
    """
    Black-Litterman资产配置模型
    
    参数:
    market_cap_weights: 市场均衡权重
    cov_matrix: 协方差矩阵
    views: 观点矩阵
    """
    # 市场均衡收益
    pi = np.log(market_cap_weights * (1 + 0.03))  # 假设无风险利率3%
    
    if views is not None:
        # 观点收益
        omega = np.diag(np.diag(P @ (tau * cov_matrix) @ P.T))
        
        # 后验收益
        M = np.linalg.inv(np.linalg.inv(tau * cov_matrix) + P.T @ np.linalg.inv(omega) @ P)
        mu = M @ (np.linalg.inv(tau * cov_matrix) @ pi + P.T @ np.linalg.inv(omega) @ Q)
    else:
        mu = pi
    
    return mu
```

---

## 四、尾部风险管理

### 极端事件建模
| 方法 | 描述 |
|------|------|
| GEV分布 | 广义极值分布 |
| GPD分布 | 广义帕累托分布 |
| POT模型 | 超过阈值模型 |

### 系统性风险
| 指标 | 描述 |
|------|------|
| CoVaR | 条件在险价值 |
| SRISK | 系统性风险指标 |
| MES | 边际期望 shortfall |

```python
def calculate_cvar_portfolio(portfolio_returns, market_returns, 
                             confidence=0.95):
    """
    计算CoVaR - 系统性风险度量
    """
    var_market = np.percentile(market_returns, (1 - confidence) * 100)
    
    # 条件VaR: 当市场VaR时的组合损失
    cvar_portfolio = portfolio_returns[market_returns <= var_market].mean()
    return cvar_portfolio
```

### 压力测试
```python
def stress_test(portfolio, scenarios):
    """
    压力测试框架
    
    参数:
    portfolio: 组合持仓
    scenarios: 压力场景
    """
    results = []
    for scenario_name, scenario_returns in scenarios.items():
        # 计算场景下的组合损失
        pnl = portfolio @ scenario_returns
        loss = pnl.min()  # 最大损失
        
        results.append({
            'scenario': scenario_name,
            'loss': loss,
            'loss_rate': loss / portfolio.sum()
        })
    return pd.DataFrame(results)
```

---

## 五、风险管理工具库

### Python风险管理库
| 库 | 功能 |
|------|------|
| QuantLib | 风险计量、定价 |
| riskfolio-lib | 组合风险优化 |
| PyPortfolioOpt | 组合优化 |
| frisk | 金融风险工具 |

### 风控报告模板
```python
def generate_risk_report(portfolio, benchmark, risk_free_rate=0.03):
    """
    生成风险管理报告
    """
    returns = portfolio['returns']
    benchmark_returns = benchmark['returns']
    
    report = {
        # 收益指标
        'total_return': (1 + returns).prod() - 1,
        'annual_return': (1 + returns.mean()) ** 252 - 1,
        'excess_return': returns.mean() - benchmark_returns.mean(),
        
        # 风险指标
        'volatility': returns.std() * np.sqrt(252),
        'var_95': calculate_var_historical(returns, 0.95),
        'cvar_95': calculate_cvar(returns, 0.95),
        'max_drawdown': calculate_max_drawdown(portfolio['equity']),
        
        # 风险调整收益
        'sharpe_ratio': (returns.mean() - risk_free_rate/252) / returns.std() * np.sqrt(252),
        'sortino_ratio': calculate_sortino(returns, risk_free_rate),
        'calmar_ratio': calculate_calmar(returns),
        
        # 相对风险
        'tracking_error': (returns - benchmark_returns).std() * np.sqrt(252),
        'information_ratio': (returns - benchmark_returns).mean() / (returns - benchmark_returns).std() * np.sqrt(252)
    }
    return report
```

---

## 六、风险限额管理

### 风险限额体系
| 限额类型 | 说明 | 典型值 |
|----------|------|--------|
| VaR限额 | VaR上限 | 组合2-5% |
| 止损限额 | 日/周/月亏损 | -3%/-5%/-10% |
| 敞口限额 | 单票/行业上限 | 5%/20% |
| 杠杆限额 | 最大杠杆 | 2-3倍 |

### 动态风控
| 方法 | 描述 |
|------|------|
| 波动率 targeting | 目标波动率管理 |
| 最大回撤控制 | DD止损 |
| 风险平价 | 等风险贡献 |

---

*最后更新: 2026-04-26*
