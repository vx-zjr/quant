# Alpha策略详解

## 一、Alpha策略概述

### 什么是Alpha
- **Alpha定义**: 投资组合相对于基准的超额收益
- **Alpha来源**: 信息优势、模型优势、执行优势
- **Alpha特征**: 与市场涨跌无关的正收益

### Alpha vs Beta
| 类型 | 定义 | 来源 | 风险 |
|------|------|------|------|
| Alpha | 超额收益 | 选股能力 | 非系统性风险 |
| Beta | 市场收益 | 市场波动 | 系统性风险 |
| Sharpe | 风险调整收益 | 综合能力 | 整体风险 |

---

## 二、多因子模型

### 因子分类体系
```
因子分类
├── 基本面因子
│   ├── 价值因子: PE、PB、PCF、PS
│   ├── 质量因子: ROE、ROA、毛利率、资产负债率
│   └── 成长因子: 营收增速、利润增速、订单增长
├── 量价因子
│   ├── 动量因子: 1M/3M/12M收益、收益率反转
│   ├── 波动率因子: 历史波动率、残差波动率
│   └── 换手率因子: 换手率变化、流动性
├── 另类因子
│   ├── 舆情因子: 新闻情绪、社交媒体情绪
│   ├── 卫星因子: 停车场车辆、港口船舶
│   └── 供应链因子: 物流数据、订单数据
└── 机器学习因子
    ├── 深度学习因子: CNN/RNN特征
    ├── 集成因子: XGBoost/LightGBM
    └── NLP因子: 文本Embedding
```

### 核心因子详解

#### 1. 估值因子
```python
def calculate_valuation_factors(stock_data):
    """
    计算估值因子
    
    因子:
    - PE: 市盈率
    - PB: 市净率
    - PCF: 市现率
    - PS: 市销率
    """
    factors = pd.DataFrame()
    
    # PE
    factors['pe_ratio'] = stock_data['market_cap'] / stock_data['net_profit']
    
    # PB
    factors['pb_ratio'] = stock_data['market_cap'] / stock_data['book_value']
    
    # PCF
    factors['pcf_ratio'] = stock_data['market_cap'] / stock_data['cash_flow']
    
    # PS
    factors['ps_ratio'] = stock_data['market_cap'] / stock_data['revenue']
    
    # 行业中性化
    for col in factors.columns:
        industry_median = stock_data.groupby('industry')[col].median()
        factors[f'{col}_neutral'] = stock_data[col] - stock_data['industry'].map(industry_median)
    
    return factors

# 因子IC测试
def factor_ic_test(factor, returns, n_periods=20):
    """
    计算因子IC值
    """
    ic_series = []
    
    for i in range(n_periods, len(factor)):
        factor_window = factor.iloc[i-n_periods:i]
        returns_window = returns.iloc[i]
        
        ic = stats.spearmanr(factor_window.iloc[-1], returns_window)[0]
        ic_series.append(ic)
    
    return {
        'ic_mean': np.mean(ic_series),
        'ic_std': np.std(ic_series),
        'ic_ir': np.mean(ic_series) / np.std(ic_series),
        'ic_positive_rate': np.mean([x > 0 for x in ic_series])
    }
```

#### 2. 动量因子
```python
def calculate_momentum_factors(prices):
    """
    计算动量因子
    """
    factors = pd.DataFrame(index=prices.index)
    
    # 短期动量 (1个月)
    factors['momentum_1m'] = prices.pct_change(20)
    
    # 中期动量 (3个月)
    factors['momentum_3m'] = prices.pct_change(60)
    
    # 长期动量 (12个月)
    factors['momentum_12m'] = prices.pct_change(252)
    
    # 动量加速
    factors['momentum_acceleration'] = factors['momentum_1m'] - factors['momentum_3m']
    
    # 收益率离散度
    monthly_returns = prices.pct_change(20)
    factors['return_dispersion'] = monthly_returns.std(axis=1)
    
    return factors

# 动量止损机制
def momentum_stop_loss(prices, lookback=20, stop_loss_pct=0.1):
    """
    动量止损策略
    """
    returns = prices.pct_change(lookback)
    high_water_mark = prices.cummax()
    drawdown = (prices - high_water_mark) / high_water_mark
    
    # 止损信号
    stop_signal = drawdown < -stop_loss_pct
    
    return {
        'returns': returns,
        'drawdown': drawdown,
        'stop_signal': stop_signal
    }
```

#### 3. 质量因子
```python
def calculate_quality_factors(financial_data):
    """
    计算质量因子
    """
    factors = pd.DataFrame()
    
    # 盈利能力
    factors['roe'] = financial_data['net_profit'] / financial_data['equity']
    factors['roa'] = financial_data['net_profit'] / financial_data['total_assets']
    factors['gross_margin'] = financial_data['gross_profit'] / financial_data['revenue']
    
    # 运营效率
    factors['inventory_turnover'] = financial_data['cost_of_goods_sold'] / financial_data['inventory']
    factors['receivables_turnover'] = financial_data['revenue'] / financial_data['accounts_receivable']
    
    # 财务健康
    factors['debt_ratio'] = financial_data['total_liabilities'] / financial_data['total_assets']
    factors['current_ratio'] = financial_data['current_assets'] / financial_data['current_liabilities']
    
    # 盈利质量
    factors['accrual'] = (financial_data['net_income'] - operating_cash_flow) / total_assets
    
    return factors

# 盈利预期调整
def earnings_expectation_adjustment(consensus_forecast, actual_earnings):
    """
    盈利预期调整因子
    
    超预期 -> 正Alpha
    低于预期 -> 负Alpha
    """
    surprise = actual_earnings - consensus_forecast
    surprise_pct = surprise / abs(consensus_forecast)
    
    # 持续性调整
    adjustment_factor = pd.Series()
    for i in range(len(surprise)):
        if i < 4:  # 最近4个季度
            weight = 1.0
        else:
            weight = 0.5
        adjustment_factor[i] = surprise_pct[i] * weight
    
    return adjustment_factor
```

---

## 三、因子组合方法

### 因子相关性分析
```python
def factor_correlation_analysis(factors, threshold=0.7):
    """
    因子相关性分析
    """
    # 计算相关矩阵
    corr_matrix = factors.corr()
    
    # 找出高相关因子对
    high_corr_pairs = []
    for i in range(len(corr_matrix.columns)):
        for j in range(i+1, len(corr_matrix.columns)):
            if abs(corr_matrix.iloc[i, j]) > threshold:
                high_corr_pairs.append({
                    'factor1': corr_matrix.columns[i],
                    'factor2': corr_matrix.columns[j],
                    'correlation': corr_matrix.iloc[i, j]
                })
    
    return pd.DataFrame(high_corr_pairs)

# 因子正交化
def orthogonalize_factors(factors, method='sequential'):
    """
    因子正交化
    """
    orthogonal_factors = factors.copy()
    
    factor_names = list(factors.columns)
    
    if method == 'sequential':
        # 顺序正交化
        for i in range(1, len(factor_names)):
            current_factor = factor_names[i]
            previous_factors = factor_names[:i]
            
            # 回归
            X = factors[previous_factors].values
            y = factors[current_factor].values
            
            reg = LinearRegression()
            reg.fit(X, y)
            
            # 残差作为正交因子
            orthogonal_factors[current_factor] = y - reg.predict(X)
    
    elif method == 'cholesky':
        # Cholesky正交化
        cov_matrix = factors.cov().values
        L = np.linalg.cholesky(cov_matrix)
        orthogonal_factors = pd.DataFrame(factors.values @ np.linalg.inv(L),
                                          columns=factors.columns)
    
    return orthogonal_factors
```

### 因子权重优化
```python
def optimize_factor_weights(factors, returns, method='mean_variance'):
    """
    因子权重优化
    """
    if method == 'mean_variance':
        # 均值方差优化
        cov_matrix = factors.cov()
        mean_returns = factors.mean()
        
        # 最大化IC/风险比
        def objective(weights):
            ic_portfolio = (weights * mean_returns).sum()
            vol_portfolio = np.sqrt(weights @ cov_matrix.values @ weights)
            return -ic_portfolio / vol_portfolio
        
        # 约束
        constraints = [
            {'type': 'eq', 'fun': lambda w: np.sum(w) - 1},
            {'type': 'ineq', 'fun': lambda w: w}  # 非负约束
        ]
        
        bounds = [(0, 1) for _ in range(len(factors.columns))]
        result = minimize(objective, np.ones(len(factors.columns))/len(factors.columns),
                         method='SLSQP', bounds=bounds, constraints=constraints)
        
        return pd.Series(result.x, index=factors.columns)
    
    elif method == 'risk_parity':
        # 风险平价
        cov_matrix = factors.cov()
        inv_cov = np.linalg.inv(cov_matrix.values)
        ones = np.ones(len(factors.columns))
        
        weights = inv_cov @ ones / (ones @ inv_cov @ ones)
        return pd.Series(weights, index=factors.columns)
    
    elif method == 'ic_weighted':
        # IC加权
        ic_values = factors.apply(lambda x: stats.spearmanr(x, returns)[0])
        weights = ic_values / ic_values.sum()
        return weights
```

### 机器学习因子融合
```python
def ml_factor_ensemble(factor_data, returns, test_size=0.3):
    """
    机器学习因子融合
    """
    from sklearn.ensemble import GradientBoostingRegressor, RandomForestRegressor
    from sklearn.linear_model import Ridge
    from sklearn.model_selection import TimeSeriesSplit
    
    # 准备数据
    X = factor_data.values
    y = returns.values
    
    split_idx = int(len(X) * (1 - test_size))
    X_train, X_test = X[:split_idx], X[split_idx:]
    y_train, y_test = y[:split_idx], y[split_idx:]
    
    # 基础模型
    models = {
        'ridge': Ridge(alpha=1.0),
        'rf': RandomForestRegressor(n_estimators=100, max_depth=5),
        'gbm': GradientBoostingRegressor(n_estimators=100, max_depth=3)
    }
    
    predictions = {}
    for name, model in models.items():
        model.fit(X_train, y_train)
        predictions[name] = model.predict(X_test)
    
    # 简单平均集成
    ensemble_pred = np.mean([predictions[name] for name in models], axis=0)
    
    # 加权集成（基于验证集性能）
    validation_performance = {}
    for name in models:
        val_pred = models[name].predict(X_train[-100:])
        validation_performance[name] = stats.spearmanr(val_pred, y_train[-100:])[0]
    
    weights = np.array([validation_performance[name] for name in models])
    weights = weights / weights.sum()
    
    weighted_pred = np.sum([weights[i] * predictions[name] 
                          for i, name in enumerate(models)], axis=0)
    
    return {
        'ensemble_pred': ensemble_pred,
        'weighted_pred': weighted_pred,
        'weights': dict(zip(models.keys(), weights))
    }
```

---

## 四、Alpha策略实现

### 完整策略框架
```python
class AlphaStrategy:
    """Alpha选股策略"""
    
    def __init__(self, factors, returns, lookback=60):
        self.factors = factors
        self.returns = returns
        self.lookback = lookback
        self.weights = None
        self.factor_signs = None
    
    def compute_factors(self, stock_data):
        """
        计算因子
        """
        factors = pd.DataFrame()
        
        # 估值因子
        factors['pe'] = stock_data['market_cap'] / stock_data['net_profit']
        factors['pb'] = stock_data['market_cap'] / stock_data['book_value']
        
        # 动量因子
        factors['momentum_1m'] = stock_data['close'].pct_change(20)
        factors['momentum_3m'] = stock_data['close'].pct_change(60)
        
        # 质量因子
        factors['roe'] = stock_data['net_profit'] / stock_data['equity']
        factors['gross_margin'] = stock_data['gross_profit'] / stock_data['revenue']
        
        # 量价因子
        factors['turnover'] = stock_data['volume'].pct_change()
        factors['volatility'] = stock_data['close'].rolling(20).std()
        
        return factors
    
    def neutralize_factors(self, factors, stock_data):
        """
        因子中性化
        """
        neutral_factors = factors.copy()
        
        for col in factors.columns:
            # 市值中性化
            log_market_cap = np.log(stock_data['market_cap'])
            reg = LinearRegression().fit(log_market_cap.values.reshape(-1, 1), factors[col].values)
            neutral_factors[col] = factors[col] - reg.predict(log_market_cap.values.reshape(-1, 1))
            
            # 行业中性化
            industry_means = stock_data.groupby('industry')[col].transform('mean')
            neutral_factors[col] = factors[col] - industry_means
        
        return neutral_factors
    
    def rank_factors(self, factors, returns, n_periods=20):
        """
        因子IC排名
        """
        ic_scores = {}
        
        for col in factors.columns:
            ic = stats.spearmanr(factors[col].iloc[-n_periods:], returns.iloc[-n_periods:])[0]
            ic_scores[col] = ic
        
        # 排序
        sorted_factors = sorted(ic_scores.items(), key=lambda x: abs(x[1]), reverse=True)
        
        return sorted_factors
    
    def compute_alpha_score(self, neutral_factors, factor_weights):
        """
        计算综合Alpha得分
        """
        # 标准化
        normalized_factors = (neutral_factors - neutral_factors.mean()) / neutral_factors.std()
        
        # 加权求和
        alpha_score = np.zeros(len(neutral_factors))
        for col, weight in factor_weights.items():
            if col in normalized_factors.columns:
                alpha_score += weight * normalized_factors[col].values
        
        return pd.Series(alpha_score, index=neutral_factors.index)
    
    def select_stocks(self, alpha_score, top_pct=0.2, bottom_pct=0.2):
        """
        选股
        """
        # 按Alpha得分排序
        ranked = alpha_score.sort_values(ascending=False)
        
        n_stocks = len(ranked)
        n_top = int(n_stocks * top_pct)
        n_bottom = int(n_stocks * bottom_pct)
        
        return {
            'long': ranked.head(n_top).index.tolist(),
            'short': ranked.tail(n_bottom).index.tolist()
        }
    
    def backtest(self, selected_stocks, period_returns):
        """
        回测
        """
        long_returns = period_returns[selected_stocks['long']].mean(axis=1)
        short_returns = period_returns[selected_stocks['short']].mean(axis=1)
        
        # 多空组合收益
        portfolio_returns = long_returns - short_returns
        
        return {
            'total_return': (1 + portfolio_returns).prod() - 1,
            'annual_return': (1 + portfolio_returns.mean()) ** 252 - 1,
            'sharpe_ratio': portfolio_returns.mean() / portfolio_returns.std() * np.sqrt(252),
            'max_drawdown': self._calculate_max_drawdown(portfolio_returns)
        }
    
    def _calculate_max_drawdown(self, returns):
        equity = (1 + returns).cumprod()
        peak = equity.cummax()
        drawdown = (equity - peak) / peak
        return drawdown.min()
```

---

## 五、Alpha策略风控

### 风险控制要点
```python
class AlphaRiskControl:
    """Alpha策略风控"""
    
    def __init__(self, max_position_pct=0.05, max_turnover=0.5):
        self.max_position_pct = max_position_pct
        self.max_turnover = max_turnover
    
    def check_position_limits(self, positions, target_weights):
        """
        检查仓位限制
        """
        violations = []
        
        for stock, weight in target_weights.items():
            if weight > self.max_position_pct:
                violations.append({
                    'stock': stock,
                    'weight': weight,
                    'limit': self.max_position_pct,
                    'action': 'reduce'
                })
        
        return violations
    
    def check_turnover_limits(self, current_weights, target_weights):
        """
        检查换手率限制
        """
        turnover = np.abs(target_weights - current_weights).sum() / 2
        
        if turnover > self.max_turnover:
            # 按比例缩放调整
            scale_factor = self.max_turnover / turnover
            adjusted_weights = current_weights + scale_factor * (target_weights - current_weights)
            return adjusted_weights
        
        return target_weights
    
    def check_factor_exposure(self, portfolio_weights, factor_loadings, target_exposure):
        """
        检查因子暴露
        """
        current_exposure = (portfolio_weights * factor_loadings).sum()
        
        exposure_diff = current_exposure - target_exposure
        
        return {
            'current_exposure': current_exposure,
            'target_exposure': target_exposure,
            'difference': exposure_diff,
            'adjustment_needed': exposure_diff > 0.1
        }
    
    def sector_neutral_check(self, portfolio_weights, sector_mapping):
        """
        行业中性检查
        """
        sector_weights = portfolio_weights.groupby(sector_mapping).sum()
        market_cap_weights = portfolio_weights.groupby(sector_mapping).sum()  # 假设市值加权
        
        active_weights = sector_weights - market_cap_weights
        
        return {
            'sector_weights': sector_weights,
            'active_weights': active_weights,
            'max_active': active_weights.abs().max()
        }
```

### 因子风险监控
```python
class FactorRiskMonitor:
    """因子风险监控"""
    
    def __init__(self, benchmark_weights):
        self.benchmark_weights = benchmark_weights
    
    def monitor_exposure(self, portfolio_weights, factor_data):
        """
        监控因子暴露
        """
        portfolio_factor_exposure = {}
        
        for factor_name in factor_data.columns:
            # 计算组合因子暴露
            exposure = (portfolio_weights * factor_data[factor_name]).sum()
            
            # 计算基准暴露
            benchmark_exposure = (self.benchmark_weights * factor_data[factor_name]).sum()
            
            portfolio_factor_exposure[factor_name] = {
                'portfolio': exposure,
                'benchmark': benchmark_exposure,
                'active': exposure - benchmark_exposure
            }
        
        return pd.DataFrame(portfolio_factor_exposure).T
    
    def detect_factor_drift(self, historical_exposure, current_exposure, threshold=0.5):
        """
        检测因子漂移
        """
        drift = current_exposure - historical_exposure.mean()
        
        significant_drifts = drift[abs(drift) > threshold * historical_exposure.std()]
        
        return {
            'drifted_factors': significant_drifts.index.tolist(),
            'drift_amounts': significant_drifts,
            'action_required': len(significant_drifts) > 0
        }
    
    def stress_test_factors(self, portfolio_weights, factor_data, scenarios):
        """
        因子压力测试
        """
        results = {}
        
        for scenario_name, factor_shocks in scenarios.items():
            shocked_factors = factor_data.copy()
            
            for factor, shock in factor_shocks.items():
                shocked_factors[factor] *= (1 + shock)
            
            # 计算冲击后的组合收益
            shocked_returns = (portfolio_weights * shocked_factors.mean(axis=1)).sum()
            
            results[scenario_name] = {
                'expected_return': shocked_returns,
                'loss_from_benchmark': shocked_returns - (portfolio_weights * factor_data.mean(axis=1)).sum()
            }
        
        return results
```

---

## 六、Alpha策略优化

### 过拟合防范
```python
class OverfittingPrevention:
    """防止过拟合"""
    
    def __init__(self, n_splits=5):
        self.n_splits = n_splits
    
    def walk_forward_validation(self, factors, returns, train_window=252, test_window=60):
        """
        步行前进验证
        """
        results = []
        
        for i in range(train_window, len(factors) - test_window, test_window):
            train_factors = factors.iloc[i-train_window:i]
            train_returns = returns.iloc[i-train_window:i]
            
            test_factors = factors.iloc[i:i+test_window]
            test_returns = returns.iloc[i:i+test_window]
            
            # 训练模型
            model = Ridge(alpha=1.0)
            model.fit(train_factors, train_returns)
            
            # 测试
            pred = model.predict(test_factors)
            ic = stats.spearmanr(pred, test_returns)[0]
            
            results.append({
                'period': i,
                'train_ic': stats.spearmanr(model.predict(train_factors), train_returns)[0],
                'test_ic': ic
            })
        
        return pd.DataFrame(results)
    
    def nested_cv(self, factors, returns, outer_folds=5, inner_folds=3):
        """
        嵌套交叉验证
        """
        from sklearn.model_selection import TimeSeriesSplit
        
        outer_cv = TimeSeriesSplit(n_splits=outer_folds)
        outer_results = []
        
        for train_idx, test_idx in outer_cv.split(factors):
            X_train, X_test = factors.iloc[train_idx], factors.iloc[test_idx]
            y_train, y_test = returns.iloc[train_idx], returns.iloc[test_idx]
            
            inner_cv = TimeSeriesSplit(n_splits=inner_folds)
            best_params = None
            best_score = -np.inf
            
            for train_inner, val_inner in inner_cv.split(X_train):
                X_tr, X_val = X_train.iloc[train_inner], X_train.iloc[val_inner]
                y_tr, y_val = y_train.iloc[train_inner], y_train.iloc[val_inner]
                
                for alpha in [0.1, 1.0, 10.0]:
                    model = Ridge(alpha=alpha)
                    model.fit(X_tr, y_tr)
                    score = stats.spearmanr(model.predict(X_val), y_val)[0]
                    
                    if score > best_score:
                        best_score = score
                        best_params = {'alpha': alpha}
            
            # 使用最佳参数在测试集上评估
            model = Ridge(**best_params)
            model.fit(X_train, y_train)
            test_ic = stats.spearmanr(model.predict(X_test), y_test)[0]
            
            outer_results.append({
                'best_params': best_params,
                'test_ic': test_ic,
                'train_ic': best_score
            })
        
        return pd.DataFrame(outer_results)
    
    def purge_kfold(self, factors, returns, n_folds=5, gap_days=5):
        """
        Purged K-Fold交叉验证（防止数据泄露）
        """
        n_samples = len(factors)
        fold_size = n_samples // n_folds
        
        results = []
        
        for fold in range(n_folds):
            train_end = (fold + 1) * fold_size
            test_start = train_end + gap_days
            test_end = test_start + fold_size if fold < n_folds - 1 else n_samples
            
            if test_end > n_samples:
                continue
            
            train_idx = list(range(0, train_end))
            test_idx = list(range(test_start, test_end))
            
            X_train, X_test = factors.iloc[train_idx], factors.iloc[test_idx]
            y_train, y_test = returns.iloc[train_idx], returns.iloc[test_idx]
            
            model = Ridge(alpha=1.0)
            model.fit(X_train, y_train)
            ic = stats.spearmanr(model.predict(X_test), y_test)[0]
            
            results.append({'fold': fold, 'test_ic': ic})
        
        return pd.DataFrame(results)
```

### 参数敏感性分析
```python
def sensitivity_analysis(strategy, param_grid, data):
    """
    参数敏感性分析
    """
    results = []
    
    for params in parameter_grid(param_grid):
        strategy.set_params(**params)
        performance = strategy.backtest(data)
        
        results.append({
            'params': params,
            'performance': performance,
            'sharpe_ratio': performance['sharpe_ratio']
        })
    
    results_df = pd.DataFrame(results)
    
    # 找出稳定区域
    stable_params = []
    for param_name in param_grid:
        param_values = results_df['params'].apply(lambda x: x[param_name])
        performance_variance = results_df.groupby(param_values)['sharpe_ratio'].std()
        stable_region = performance_variance.idxmin()
        stable_params.append((param_name, stable_region))
    
    return {
        'all_results': results_df,
        'stable_params': dict(stable_params),
        'robustness_score': 1 - results_df['sharpe_ratio'].std() / results_df['sharpe_ratio'].mean()
    }
```

---

## 七、Alpha策略评估

### 绩效归因
```python
class PerformanceAttribution:
    """绩效归因"""
    
    def __init__(self, benchmark_weights, factor_data):
        self.benchmark_weights = benchmark_weights
        self.factor_data = factor_data
    
    def brinson_attribution(self, portfolio_weights, benchmark_weights, returns):
        """
        Brinson归因
        """
        # 配置效应
        allocation_effect = (portfolio_weights - benchmark_weights) * returns.mean()
        
        # 选择效应
        selection_effect = benchmark_weights * (returns - returns.mean())
        
        # 交互效应
        interaction_effect = (portfolio_weights - benchmark_weights) * (returns - returns.mean())
        
        return {
            'allocation_effect': allocation_effect.sum(),
            'selection_effect': selection_effect.sum(),
            'interaction_effect': interaction_effect.sum(),
            'total_active_return': (portfolio_weights - benchmark_weights) @ returns
        }
    
    def factor_attribution(self, portfolio_weights, factor_exposure, factor_returns):
        """
        因子归因
        """
        # 计算组合在各因子上的暴露
        portfolio_factor_exposure = (portfolio_weights * factor_exposure).sum()
        
        # 计算因子收益贡献
        factor_contribution = portfolio_factor_exposure * factor_returns
        
        # 特异性收益
        specific_return = returns - (factor_exposure * factor_returns).sum(axis=1)
        
        return {
            'factor_contribution': factor_contribution,
            'specific_contribution': specific_return.mean(),
            'total_return': factor_contribution.sum() + specific_return.mean()
        }
```

---

## 八、实战案例

### 完整Alpha策略示例
```python
def create_alpha_strategy(config):
    """
    创建完整Alpha策略
    """
    # 1. 数据准备
    data = prepare_stock_data(config['tickers'], config['start_date'], config['end_date'])
    
    # 2. 因子计算
    strategy = AlphaStrategy(
        factors=data['factors'],
        returns=data['returns'],
        lookback=config['lookback']
    )
    
    # 3. 计算因子
    factors = strategy.compute_factors(data['stock_data'])
    
    # 4. 因子中性化
    neutral_factors = strategy.neutralize_factors(factors, data['stock_data'])
    
    # 5. 因子排名
    ranked_factors = strategy.rank_factors(neutral_factors, data['returns'])
    
    # 6. 选择有效因子
    effective_factors = [f for f, ic in ranked_factors if abs(ic) > 0.05][:10]
    
    # 7. 因子权重优化
    factor_weights = optimize_factor_weights(
        neutral_factors[effective_factors],
        data['returns'],
        method='ic_weighted'
    )
    
    # 8. 计算Alpha得分
    alpha_score = strategy.compute_alpha_score(neutral_factors[effective_factors], factor_weights)
    
    # 9. 选股
    selected = strategy.select_stocks(alpha_score, top_pct=0.1, bottom_pct=0.1)
    
    # 10. 风控
    risk_control = AlphaRiskControl(max_position_pct=0.05)
    violations = risk_control.check_position_limits(None, selected)
    
    # 11. 回测
    results = strategy.backtest(selected, data['returns'])
    
    return {
        'factors': effective_factors,
        'weights': factor_weights,
        'selected_stocks': selected,
        'backtest_results': results
    }
```

---

*最后更新: 2026-04-26*