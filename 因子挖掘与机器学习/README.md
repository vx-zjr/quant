# 因子挖掘与机器学习量化

> **更新**: 2026-04-26 | **收集范围**: 因子挖掘/特征工程/ML模型/深度学习预测

---

## 一、因子体系总览

### 1.1 因子分类框架

```
┌──────────────────────────────────────────────────────────────────┐
│                        量化因子体系                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   风格因子   │  │   行业因子   │  │  情绪因子   │              │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤              │
│  │ 市值        │  │ 银行        │  │ 资金流向    │              │
│  │ 估值        │  │ 医药        │  │ 波动率      │              │
│  │ 动量        │  │ 消费        │  │ 舆情指数    │              │
│  │ 质量        │  │ 科技        │  │ 分析师情绪  │              │
│  │ 波动        │  │ ...        │  │ 持仓集中度  │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   另类因子   │  │   宏观因子   │  │  技术因子   │              │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤              │
│  │ 卫星图像    │  │ 利率        │  │ MACD        │              │
│  │ 供应链      │  │ 汇率        │  │ RSI         │              │
│  │ ESG评分     │  │ 通胀        │  │ 布林带      │              │
│  │ 专利        │  │ 增长        │  │ 成交量      │              │
│  │ 电商        │  │ 政策        │  │ 趋势线      │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 1.2 因子层级结构

| 层级 | 因子类型 | 示例 | 信息来源 |
|------|----------|------|----------|
| 市场层 | Beta/波动率 | Beta, Realized Vol | 日内数据 |
| 行业层 | 行业轮动 | 行业相对强弱 | 日频数据 |
| 风格层 | 风格因子 | Size, Value, Momentum | 日频数据 |
| 个股层 | Alpha因子 | 估值/质量/情绪 | 日频数据 |
| 高频层 | 微观结构 | 订单流/簿不平衡 | Tick数据 |

---

## 二、因子挖掘方法论

### 2.1 系统化因子挖掘流程

```python
"""
系统化因子挖掘框架
"""

import pandas as pd
import numpy as np
from typing import List, Tuple, Dict
from dataclasses import dataclass
from abc import ABC, abstractmethod

@dataclass
class FactorConfig:
    """因子配置"""
    name: str
    category: str  # value/momentum/quality/sentiment
    lookback_period: int
    decay_half_life: int  # 半衰期
    neutralization: List[str]  # 中性化变量
    winsorize_pct: float = 0.01  # 去极值百分比

class FactorMiningPipeline:
    """
    因子挖掘流水线
    
    步骤:
    1. 数据清洗
    2. 因子计算
    3. 去极值与标准化
    4. 中性化处理
    5. IC分析
    6. 多因素组合
    """
    
    def __init__(self):
        self.factors = {}
        self.factor_returns = {}
        self.ic_matrix = pd.DataFrame()
        
    def calculate_raw_factor(self, data: pd.DataFrame, config: FactorConfig) -> pd.Series:
        """
        计算原始因子
        
        不同类型因子的计算方法
        """
        if config.category == 'value':
            return self._value_factor(data)
        elif config.category == 'momentum':
            return self._momentum_factor(data, config.lookback_period)
        elif config.category == 'quality':
            return self._quality_factor(data)
        elif config.category == 'sentiment':
            return self._sentiment_factor(data)
            
    def _value_factor(self, data: pd.DataFrame) -> pd.Series:
        """
        价值因子
        
        常用指标: PE, PB, PCF, PS, EV/EBITDA
        """
        # 市盈率倒数(EP)
        ep = 1 / data['pe'].replace(0, np.nan)
        
        # 市净率倒数(BP)
        bp = 1 / data['pb'].replace(0, np.nan)
        
        # 现金流率(CP)
        cp = data['operating_cf'] / data['market_cap']
        
        # 综合价值因子(等权平均)
        value_score = (ep.rank() + bp.rank() + cp.rank()) / 3
        
        return value_score
    
    def _momentum_factor(self, data: pd.DataFrame, lookback: int) -> pd.Series:
        """
        动量因子
        
        多周期动量叠加
        """
        returns = data['close'].pct_change()
        
        # 短期动量 (20日)
        mom_20d = returns.rolling(20).sum()
        
        # 中期动量 (60日)
        mom_60d = returns.rolling(60).sum()
        
        # 长期动量 (240日)
        mom_240d = returns.rolling(240).sum()
        
        # 动量加速
        mom_accel = mom_20d - mom_60d.shift(20)
        
        # 综合动量因子
        # W_{short}=0.4, W_{medium}=0.3, W_{long}=0.3
        momentum = 0.4 * mom_20d.rank() + 0.3 * mom_60d.rank() + 0.3 * mom_accel.rank()
        
        return momentum
    
    def _quality_factor(self, data: pd.DataFrame) -> pd.Series:
        """
        质量因子
        
        衡量公司盈利能力和运营效率
        """
        # 盈利能力
        roa = data['net_income'] / data['total_assets']
        roe = data['net_income'] / data['equity']
        
        # 运营效率
        asset_turnover = data['revenue'] / data['total_assets']
        
        # 财务健康
        debt_to_equity = data['total_debt'] / data['equity']
        current_ratio = data['current_assets'] / data['current_liabilities']
        
        # 盈利质量
        accrual = (net_income - operating_cf) / total_assets
        
        # 综合质量因子
        quality_score = roa.rank() + roe.rank() + asset_turnover.rank() - debt_to_equity.rank()
        
        return quality_score
    
    def _sentiment_factor(self, data: pd.DataFrame) -> pd.Series:
        """
        情绪因子
        
        资金流向、机构持仓、分析师评级
        """
        # 资金流向
        money_flow = data['close'] * data['volume']
        inflow = money_flow.rolling(5).mean()
        
        # 分析师评级
        analyst_rating = data['buy'] - data['sell']
        
        # 持仓变化
        institution_holding_chg = data['inst_holding'].pct_change()
        
        # 综合情绪
        sentiment = inflow.rank() + analyst_rating.rank() + institution_holding_chg.rank()
        
        return sentiment
    
    def neutralize(self, factor: pd.Series, controls: pd.DataFrame) -> pd.Series:
        """
        因子中性化
        
        去除不需要的风格暴露
        """
        from sklearn.linear_model import LinearRegression
        
        # 构建回归模型
        X = controls[['size', 'beta', 'sector_dummy']].values
        y = factor.values
        
        model = LinearRegression()
        model.fit(X, y)
        
        # 残差作为中性化因子
        residual = y - model.predict(X)
        
        return pd.Series(residual, index=factor.index)
    
    def winsorize(self, factor: pd.Series, pct: float = 0.01) -> pd.Series:
        """
        去极值处理
        
        使用分位数截断
        """
        lower = factor.quantile(pct)
        upper = factor.quantile(1 - pct)
        
        return factor.clip(lower, upper)
    
    def standardize(self, factor: pd.Series) -> pd.Series:
        """
        标准化处理
        
        Z-score标准化
        """
        mean = factor.mean()
        std = factor.std()
        
        return (factor - mean) / std
```

### 2.2 IC分析框架

```python
"""
因子IC分析模块
"""

import pandas as pd
import numpy as np
from typing import Dict, List

class ICAnalysis:
    """
    信息系数(IC)分析
    
    衡量因子预测能力
    """
    
    def __init__(self):
        self.ic_results = {}
        
    def calculate_ic(self, factor: pd.Series, returns: pd.Series) -> Dict:
        """
        计算IC统计量
        
        IC = Correlation(factor, forward_returns)
        """
        # Pearson IC
        pearson_ic = factor.corr(returns)
        
        # Spearman IC (Rank IC)
        spearman_ic = factor.rank().corr(returns.rank())
        
        # 分组回测IC
        quantiles = factor.quantile([0.2, 0.4, 0.6, 0.8])
        
        return {
            'pearson_ic': pearson_ic,
            'spearman_ic': spearman_ic,
            'quantiles': quantiles,
            'factor_quantiles': pd.qcut(factor, 10, labels=False)
        }
    
    def rolling_ic(self, factor: pd.DataFrame, returns: pd.DataFrame, 
                   window: int = 20) -> pd.Series:
        """
        滚动IC时序
        
        分析因子稳定性
        """
        ic_series = []
        
        for i in range(window, len(factor)):
            window_factor = factor.iloc[i-window:i]
            window_returns = returns.iloc[i-window:i]
            
            ic = window_factor.corr(window_returns)
            ic_series.append(ic)
            
        return pd.Series(ic_series, index=factor.index[window:])
    
    def ic_ir_ratio(self, ic_series: pd.Series) -> Dict:
        """
        计算IC IR比率
        
        IR = Mean(IC) / Std(IC)
        """
        mean_ic = ic_series.mean()
        std_ic = ic_series.std()
        ir = mean_ic / std_ic if std_ic > 0 else 0
        
        # IC胜率
        win_rate = (ic_series > 0).sum() / len(ic_series)
        
        return {
            'mean_ic': mean_ic,
            'std_ic': std_ic,
            'ir': ir,
            'win_rate': win_rate,
            'positive_ic_ratio': win_rate
        }
    
    def decay_analysis(self, factor: pd.DataFrame, returns: pd.DataFrame,
                       max_holding: int = 20) -> pd.DataFrame:
        """
        因子衰减分析
        
        因子预测能力随持仓时间的衰减
        """
        results = []
        
        for horizon in range(1, max_holding + 1):
            forward_returns = returns.shift(-horizon)
            
            ic = factor.corr(forward_returns)
            
            results.append({
                'horizon': horizon,
                'ic_mean': ic.mean(),
                'ic_std': ic.std(),
                'ir': ic.mean() / ic.std() if ic.std() > 0 else 0
            })
            
        return pd.DataFrame(results)
    
    def turnover_analysis(self, factor: pd.DataFrame, 
                         quantile: int = 10) -> pd.Series:
        """
        因子换手率分析
        
        衡量因子稳定性
        """
        # 计算每日因子排名
        factor_rank = factor.rank(ascending=False)
        
        # 计算排名变化
        rank_change = factor_rank.diff().abs()
        
        # 换手率 = 排名变化的绝对值均值
        turnover = rank_change.mean(axis=1) / (quantile - 1)
        
        return turnover
```

---

## 三、机器学习量化模型

### 3.1 特征工程

```python
"""
特征工程模块
"""

import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler

class FeatureEngineering:
    """
    量化特征工程
    
    从原始数据构建机器学习特征
    """
    
    def __init__(self):
        self.scaler = StandardScaler()
        self.feature_names = []
        
    def calculate_returns_features(self, prices: pd.DataFrame) -> pd.DataFrame:
        """计算收益率特征"""
        returns = prices.pct_change()
        
        features = pd.DataFrame(index=prices.index)
        
        # 收益率统计
        for window in [1, 5, 10, 20, 60]:
            features[f'return_{window}d'] = returns.rolling(window).mean()
            features[f'volatility_{window}d'] = returns.rolling(window).std()
            features[f'skewness_{window}d'] = returns.rolling(window).skew()
            features[f'kurtosis_{window}d'] = returns.rolling(window).kurt()
            
        # 动量特征
        for window in [5, 10, 20, 60]:
            features[f'momentum_{window}d'] = returns.rolling(window).sum()
            
        # 波动率特征
        for window in [5, 10, 20]:
            rolling_vol = returns.rolling(window).std()
            features[f'vol_regime_{window}d'] = rolling_vol > rolling_vol.rolling(60).mean()
            
        return features
    
    def calculate_volume_features(self, volume: pd.DataFrame) -> pd.DataFrame:
        """计算成交量特征"""
        features = pd.DataFrame(index=volume.index)
        
        # 量价相关性
        for window in [10, 20, 60]:
            features[f'vp_count_{window}d'] = (volume > volume.rolling(window).mean()).rolling(window).sum()
            
        # 资金流向
        features['money_flow'] = volume * prices.pct_change()
        features['obv'] = (np.sign(prices.pct_change()) * volume).cumsum()  # 能量潮
        
        return features
    
    def calculate_orderbook_features(self, orderbook: dict) -> pd.DataFrame:
        """计算订单簿特征"""
        features = {}
        
        # 买卖不平衡
        features['bid_ask_imbalance'] = (orderbook['bid_vol'] - orderbook['ask_vol']) / \
                                        (orderbook['bid_vol'] + orderbook['ask_vol'])
        
        # 深度不对称
        features['depth_asymmetry'] = (orderbook['bid_vol'][:5].sum() - 
                                       orderbook['ask_vol'][:5].sum()) / \
                                      (orderbook['bid_vol'][:5].sum() + 
                                       orderbook['ask_vol'][:5].sum())
        
        # 订单流冲击
        features['order_flow_imbalance'] = orderbook['order_flow'].sum()
        
        return pd.DataFrame(features)
    
    def calculate_macro_features(self, macro_data: pd.DataFrame) -> pd.DataFrame:
        """计算宏观因子特征"""
        features = pd.DataFrame(index=macro_data.index)
        
        # 利率敏感度
        features['rate_beta'] = returns.rolling(60).cov(macro_data['risk_free_rate'])
        
        # 信用利差
        features['credit_spread'] = macro_data['corporate_bond_yield'] - \
                                   macro_data['treasury_yield']
        
        # 恐慌指数
        features['vix_level'] = macro_data['vix'] / macro_data['vix'].rolling(252).mean()
        
        return features
    
    def create_lagged_features(self, data: pd.DataFrame, 
                               lags: List[int] = [1, 2, 3, 5, 10]) -> pd.DataFrame:
        """创建滞后特征"""
        lagged = pd.DataFrame(index=data.index)
        
        for col in data.columns:
            for lag in lags:
                lagged[f'{col}_lag_{lag}'] = data[col].shift(lag)
                
        return lagged
    
    def create_interaction_features(self, features: pd.DataFrame) -> pd.DataFrame:
        """创建交互特征"""
        interactions = pd.DataFrame(index=features.index)
        
        # 常见交互
        interactions['momentum_x_value'] = features['momentum_20d'] * features['value_score']
        interactions['quality_x_growth'] = features['quality_score'] * features['earnings_growth']
        
        return interactions
```

### 3.2 传统ML模型

```python
"""
传统机器学习模型
"""

import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForest, GradientBoosting
from sklearn.linear_model import ElasticNet, LogisticRegression
from sklearn.model_selection import TimeSeriesSplit
from sklearn.metrics import accuracy, roc_auc_score

class MLQuantModel:
    """
    机器学习量化模型基类
    """
    
    def __init__(self, model_type='regression'):
        self.model_type = model_type
        self.models = {}
        self.feature_importance = {}
        
    def train(self, X: pd.DataFrame, y: pd.Series, model_name='model'):
        """训练模型"""
        if self.model_type == 'regression':
            model = self._train_regression(X, y)
        else:
            model = self._train_classification(X, y)
            
        self.models[model_name] = model
        return model
    
    def predict(self, X: pd.DataFrame, model_name='model') -> np.ndarray:
        """预测"""
        if model_name not in self.models:
            raise ValueError(f"Model {model_name} not found")
            
        return self.models[model_name].predict(X)
    
    def get_feature_importance(self, model_name='model') -> pd.Series:
        """获取特征重要性"""
        model = self.models[model_name]
        
        if hasattr(model, 'feature_importances_'):
            return pd.Series(model.feature_importances_, index=X.columns)
        elif hasattr(model, 'coef_'):
            return pd.Series(np.abs(model.coef_), index=X.columns)
            
        return pd.Series()

class RandomForestModel(MLQuantModel):
    """随机森林量化模型"""
    
    def __init__(self):
        super().__init__('regression')
        self.params = {
            'n_estimators': 100,
            'max_depth': 10,
            'min_samples_leaf': 50,
            'n_jobs': -1
        }
        
    def _train_regression(self, X, y):
        """回归任务"""
        from sklearn.ensemble import RandomForestRegressor
        
        model = RandomForestRegressor(**self.params)
        model.fit(X, y)
        
        self.feature_importance['random_forest'] = pd.Series(
            model.feature_importances_, index=X.columns
        )
        
        return model
    
    def cross_validate(self, X, y, n_splits=5):
        """时间序列交叉验证"""
        tscv = TimeSeriesSplit(n_splits=n_splits)
        
        cv_scores = []
        for train_idx, test_idx in tscv.split(X):
            X_train, X_test = X.iloc[train_idx], X.iloc[test_idx]
            y_train, y_test = y.iloc[train_idx], y.iloc[test_idx]
            
            model = RandomForestRegressor(**self.params)
            model.fit(X_train, y_train)
            
            pred = model.predict(X_test)
            
            # IC-like metric
            ic = np.corrcoef(pred, y_test)[0, 1]
            cv_scores.append(ic)
            
        return cv_scores

class GradientBoostingModel(MLQuantModel):
    """梯度提升模型"""
    
    def __init__(self):
        super().__init__('regression')
        self.params = {
            'n_estimators': 200,
            'learning_rate': 0.05,
            'max_depth': 5,
            'subsample': 0.8,
            'reg_alpha': 0.1,
            'reg_lambda': 0.1
        }
        
    def _train_regression(self, X, y):
        """回归任务"""
        from sklearn.ensemble import GradientBoostingRegressor
        
        model = GradientBoostingRegressor(**self.params)
        model.fit(X, y)
        
        return model

class XGBoostModel:
    """XGBoost模型"""
    
    def __init__(self):
        try:
            import xgboost
            self.xgb = xgboost
        except ImportError:
            print("XGBoost not installed, install with: pip install xgboost")
            
        self.params = {
            'objective': 'reg:squarederror',
            'max_depth': 6,
            'learning_rate': 0.05,
            'n_estimators': 200,
            'subsample': 0.8,
            'colsample_bytree': 0.8,
            'reg_alpha': 0.1,
            'reg_lambda': 0.1,
            'tree_method': 'hist'  # 快速训练
        }
        
    def train(self, X, y):
        """训练XGBoost"""
        model = self.xgb.XGBRegressor(**self.params)
        model.fit(X, y)
        
        return model
    
    def feature_importance(self, model):
        """特征重要性"""
        importance = model.feature_importances_
        return pd.Series(importance, index=X.columns)
    
    def optimize(self, X, y, param_grid):
        """超参数优化"""
        from sklearn.model_selection import GridSearchCV
        
        base_model = self.xgb.XGBRegressor(**self.params)
        
        cv = GridSearchCV(
            base_model, 
            param_grid,
            cv=TimeSeriesSplit(n_splits=3),
            scoring='neg_mean_squared_error'
        )
        
        cv.fit(X, y)
        
        return cv.best_params_
```

### 3.3 深度学习模型

```python
"""
深度学习量化模型
"""

import torch
import torch.nn as nn
from typing import Tuple

class QuantTransformer(nn.Module):
    """
    Transformer量化预测模型
    
    用于捕捉市场数据的时序依赖和交叉效应
    """
    
    def __init__(self, input_dim, d_model=128, n_heads=8, n_layers=4, dropout=0.1):
        super().__init__()
        
        # 特征嵌入
        self.embedding = nn.Linear(input_dim, d_model)
        
        # 位置编码
        self.pos_encoder = PositionalEncoding(d_model, dropout)
        
        # Transformer编码器
        encoder_layer = nn.TransformerEncoderLayer(
            d_model=d_model,
            nhead=n_heads,
            dim_feedforward=d_model * 4,
            dropout=dropout,
            batch_first=True
        )
        self.transformer = nn.TransformerEncoder(encoder_layer, num_layers=n_layers)
        
        # 输出层
        self.fc = nn.Sequential(
            nn.Linear(d_model, d_model // 2),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(d_model // 2, 1)
        )
        
    def forward(self, x):
        # x: (batch, seq_len, features)
        x = self.embedding(x)
        x = self.pos_encoder(x)
        x = self.transformer(x)
        
        # 取最后一个时间步
        x = x[:, -1, :]
        
        return self.fc(x)

class PositionalEncoding(nn.Module):
    """位置编码"""
    
    def __init__(self, d_model, max_len=5000, dropout=0.1):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)
        
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-np.log(10000.0) / d_model))
        
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)  # (1, max_len, d_model)
        
        self.register_buffer('pe', pe)
        
    def forward(self, x):
        x = x + self.pe[:, :x.size(1), :]
        return self.dropout(x)

class LSTMQuantModel(nn.Module):
    """
    LSTM量化模型
    
    用于时序预测
    """
    
    def __init__(self, input_dim, hidden_dim=128, num_layers=2, dropout=0.2):
        super().__init__()
        
        self.lstm = nn.LSTM(
            input_size=input_dim,
            hidden_size=hidden_dim,
            num_layers=num_layers,
            batch_first=True,
            dropout=dropout if num_layers > 1 else 0
        )
        
        self.attention = AttentionLayer(hidden_dim)
        
        self.fc = nn.Sequential(
            nn.Linear(hidden_dim, hidden_dim // 2),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(hidden_dim // 2, 1)
        )
        
    def forward(self, x):
        # x: (batch, seq_len, features)
        lstm_out, _ = self.lstm(x)
        
        # 注意力机制
        attended = self.attention(lstm_out)
        
        return self.fc(attended)

class AttentionLayer(nn.Module):
    """注意力层"""
    
    def __init__(self, hidden_dim):
        super().__init__()
        self.attention = nn.Linear(hidden_dim, 1)
        
    def forward(self, lstm_output):
        # lstm_output: (batch, seq_len, hidden_dim)
        attention_scores = self.attention(lstm_output)
        attention_weights = torch.softmax(attention_scores, dim=1)
        
        # 加权求和
        context = torch.sum(attention_weights * lstm_output, dim=1)
        
        return context

def train_quant_model(model, train_loader, val_loader, epochs=100, lr=0.001):
    """训练量化模型"""
    optimizer = torch.optim.Adam(model.parameters(), lr=lr)
    scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
        optimizer, mode='min', patience=5, factor=0.5
    )
    criterion = nn.MSELoss()
    
    best_val_loss = float('inf')
    
    for epoch in range(epochs):
        # 训练
        model.train()
        train_loss = 0
        for X, y in train_loader:
            optimizer.zero_grad()
            pred = model(X)
            loss = criterion(pred, y)
            loss.backward()
            optimizer.step()
            train_loss += loss.item()
            
        # 验证
        model.eval()
        val_loss = 0
        with torch.no_grad():
            for X, y in val_loader:
                pred = model(X)
                loss = criterion(pred, y)
                val_loss += loss.item()
                
        scheduler.step(val_loss)
        
        if val_loss < best_val_loss:
            best_val_loss = val_loss
            # 保存最佳模型
            torch.save(model.state_dict(), 'best_model.pt')
            
        print(f"Epoch {epoch}: Train Loss={train_loss:.4f}, Val Loss={val_loss:.4f}")
```

---

## 四、因子组合与优化

### 4.1 多因子组合

```python
"""
多因子组合优化
"""

import numpy as np
import pandas as pd
from scipy.optimize import minimize

class FactorPortfolio:
    """
    多因子组合
    
    组合多个因子构建Alpha策略
    """
    
    def __init__(self, factors: pd.DataFrame, returns: pd.Series):
        self.factors = factors
        self.returns = returns
        
    def equal_weight(self) -> np.ndarray:
        """等权组合"""
        n_factors = len(self.factors.columns)
        return np.ones(n_factors) / n_factors
    
    def ic_weight(self) -> np.ndarray:
        """IC加权"""
        ic_values = []
        for col in self.factors.columns:
            ic = self.factors[col].corr(self.returns)
            ic_values.append(ic)
            
        ic_array = np.array(ic_values)
        # 归一化权重
        weights = ic_array / ic_array.sum()
        
        return weights
    
    def risk_parity(self) -> np.ndarray:
        """
        风险平价组合
        
        每个因子对组合风险的贡献相等
        """
        # 计算因子收益
        factor_returns = self.factors.apply(lambda x: self.returns.corr(x))
        
        # 计算因子波动率
        factor_vol = self.factors.std()
        
        # 风险平价权重
        inv_vol = 1 / factor_vol
        weights = inv_vol / inv_vol.sum()
        
        return weights.values
    
    def mean_variance_optimize(self, risk_aversion=0.5) -> np.ndarray:
        """
        均值方差优化
        
        最大化: μ'w - (λ/2) * w'Σw
        """
        n = len(self.factors.columns)
        
        # 预期收益向量
        mu = self.factors.apply(lambda x: self.returns.corr(x)).values
        
        # 协方差矩阵
        cov = self.factors.cov().values
        
        # 优化
        def objective(w):
            port_return = np.dot(mu, w)
            port_risk = np.dot(w, np.dot(cov, w))
            return -(port_return - risk_aversion * port_risk / 2)
        
        # 约束: 权重和为1
        constraints = {'type': 'eq', 'fun': lambda w: np.sum(w) - 1}
        
        # 边界
        bounds = [(0, 1) for _ in range(n)]
        
        # 初始值
        x0 = np.ones(n) / n
        
        result = minimize(objective, x0, method='SLSQP',
                         bounds=bounds, constraints=constraints)
        
        return result.x
    
    def constrained_optimize(self, max_weight=0.3, min_weight=0.05) -> np.ndarray:
        """
        带约束的优化
        
        - 最大权重限制
        - 最小权重限制
        - 风格中性
        """
        n = len(self.factors.columns)
        
        mu = self.factors.apply(lambda x: self.returns.corr(x)).values
        cov = self.factors.cov().values
        
        def objective(w):
            port_return = np.dot(mu, w)
            port_risk = np.dot(w, np.dot(cov, w))
            return -(port_return - 0.5 * port_risk / 2)
        
        constraints = [
            {'type': 'eq', 'fun': lambda w: np.sum(w) - 1}
        ]
        
        bounds = [(min_weight, max_weight) for _ in range(n)]
        
        x0 = np.ones(n) / n
        
        result = minimize(objective, x0, method='SLSQP',
                         bounds=bounds, constraints=constraints)
        
        return result.x
```

### 4.2 因子正交化

```python
"""
因子正交化处理
"""

from sklearn.decomposition import PCA
from scipy.stats import spearmanr

class FactorOrthogonalization:
    """
    因子正交化
    
    去除因子间冗余信息
    """
    
    def __init__(self):
        self.pca = None
        self.orthogonalized_factors = None
        
    def gram_schmidt(self, factors: pd.DataFrame) -> pd.DataFrame:
        """
        Gram-Schmidt正交化
        
        逐步正交化因子
        """
        ortho_factors = pd.DataFrame(index=factors.index)
        factor_names = factors.columns.tolist()
        
        # 第一个因子保持不变
        ortho_factors[factor_names[0]] = factors[factor_names[0]]
        
        # 后续因子依次正交化
        for i in range(1, len(factor_names)):
            new_factor = factors[factor_names[i]].copy()
            
            # 对每个已正交因子回归
            for prev_factor in ortho_factors.columns:
                # 回归系数
                coef = new_factor.cov(prev_factor) / prev_factor.var()
                # 减去投影
                new_factor = new_factor - coef * prev_factor
                
            ortho_factors[factor_names[i]] = new_factor
            
        self.orthogonalized_factors = ortho_factors
        return ortho_factors
    
    def pca_orthogonalize(self, factors: pd.DataFrame, n_components=None) -> pd.DataFrame:
        """
        PCA正交化
        
        使用主成分分析
        """
        if n_components is None:
            n_components = min(factors.shape[1], factors.shape[0])
            
        self.pca = PCA(n_components=n_components)
        
        # PCA变换
        pca_factors = self.pca.fit_transform(factors)
        
        # 转换为DataFrame
        result = pd.DataFrame(
            pca_factors,
            index=factors.index,
            columns=[f'PC{i+1}' for i in range(n_components)]
        )
        
        return result
    
    def residualize(self, factor: pd.Series, controls: pd.DataFrame) -> pd.Series:
        """
        残差化处理
        
        去除对控制变量的依赖
        """
        from sklearn.linear_model import LinearRegression
        
        X = controls.values
        y = factor.values
        
        model = LinearRegression()
        model.fit(X, y)
        
        residual = y - model.predict(X)
        
        return pd.Series(residual, index=factor.index)
    
    def neutralize_by_style(self, factor: pd.DataFrame, style_factors: list) -> pd.DataFrame:
        """
        风格中性化
        
        去除风格因子的暴露
        """
        neutral = factor.copy()
        
        style_data = factor[style_factors]
        
        for col in factor.columns:
            if col not in style_factors:
                neutral[col] = self.residualize(factor[col], style_data)
                
        return neutral
```

---

## 五、特征重要性与解释

### 5.1 SHAP分析

```python
"""
SHAP值分析
"""

import shap

class FactorExplainability:
    """
    因子可解释性分析
    
    使用SHAP值解释模型
    """
    
    def __init__(self):
        self.explainer = None
        self.shap_values = None
        
    def fit(self, model, X_train: pd.DataFrame):
        """训练SHAP解释器"""
        self.explainer = shap.TreeExplainer(model)
        return self.explainer
        
    def compute_shap(self, X: pd.DataFrame) -> np.ndarray:
        """计算SHAP值"""
        self.shap_values = self.explainer.shap_values(X)
        return self.shap_values
    
    def plot_importance(self):
        """绘制特征重要性"""
        shap.summary_plot(self.shap_values, X)
        
    def plot_dependence(self, feature_name: str):
        """绘制特征依赖"""
        shap.dependence_plot(feature_name, self.shap_values, X)
        
    def top_features(self, n: int = 20) -> pd.Series:
        """获取Top N重要特征"""
        mean_abs_shap = np.abs(self.shap_values).mean(axis=0)
        importance = pd.Series(mean_abs_shap, index=X.columns)
        return importance.nlargest(n)
```

### 5.2 因子衰减分析

```python
"""
因子衰减与生命周期分析
"""

class FactorDecay:
    """
    因子生命周期分析
    
    分析因子的时效性
    """
    
    def __init__(self):
        self.half_lives = {}
        
    def compute_decay_rate(self, factor: pd.DataFrame, 
                          returns: pd.DataFrame) -> pd.DataFrame:
        """
        计算因子衰减率
        
        预测能力如何随时间变化
        """
        horizons = list(range(1, 21))  # 1-20天
        
        results = []
        for h in horizons:
            # h天后收益
            forward_returns = returns.shift(-h)
            
            ic = factor.corrwith(forward_returns, axis=1).mean()
            
            results.append({
                'horizon': h,
                'ic': ic
            })
            
        decay_df = pd.DataFrame(results)
        
        # 计算半衰期
        initial_ic = decay_df['ic'].iloc[0]
        self.half_lives = {}
        
        for col in factor.columns:
            ic_series = decay_df.set_index('horizon')[col]
            half_life = self._find_half_life(ic_series, initial_ic)
            self.half_lives[col] = half_life
            
        return decay_df
    
    def _find_half_life(self, ic_series, initial_ic):
        """找到IC减半的时间点"""
        half_ic = initial_ic / 2
        
        for h in ic_series.index:
            if abs(ic_series[h]) < abs(half_ic):
                return h
                
        return len(ic_series)
    
    def adaptive_weighting(self, factor: pd.DataFrame, half_lives: dict) -> pd.DataFrame:
        """
        自适应权重调整
        
        根据衰减速度调整权重
        """
        weights = pd.DataFrame(index=factor.index, columns=factor.columns)
        
        for col in factor.columns:
            hl = half_lives.get(col, 20)
            # 指数衰减权重
            age = (factor.index - factor.index[0]).days
            decay = np.exp(-age / hl)
            weights[col] = decay
            
        return weights
```

---

## 六、实战代码示例

### 6.1 完整因子挖掘流程

```python
"""
完整因子挖掘示例
"""

import pandas as pd
import numpy as np

# 1. 数据准备
def prepare_data():
    """准备股票数据"""
    # 假设从tushare获取数据
    import tushare as ts
    
    pro = ts.pro_api('YOUR_TOKEN')
    
    # 获取日线数据
    df = pro.daily(ts_code='000001.SZ', start_date='20180101')
    
    return df

# 2. 因子计算
def calculate_factors(data):
    """计算多因子"""
    factors = pd.DataFrame(index=data.index)
    
    # 价值因子
    factors['ep'] = 1 / data['pe']
    factors['bp'] = 1 / data['pb']
    factors['sp'] = data['revenue'] / data['market_cap']
    
    # 动量因子
    factors['mom_20d'] = data['close'].pct_change(20)
    factors['mom_60d'] = data['close'].pct_change(60)
    
    # 质量因子
    factors['roe'] = data['net_profit'] / data['equity']
    factors['roa'] = data['net_profit'] / data['total_assets']
    
    return factors

# 3. IC分析
def analyze_ic(factors, returns):
    """IC分析"""
    ic_analyzer = ICAnalysis()
    
    results = {}
    for col in factors.columns:
        ic_result = ic_analyzer.calculate_ic(factors[col], returns)
        results[col] = ic_result
        
    return results

# 4. 模型训练
def train_model(factors, returns):
    """训练预测模型"""
    from sklearn.ensemble import GradientBoostingRegressor
    
    # 训练集/测试集
    train_size = int(len(factors) * 0.8)
    X_train, X_test = factors[:train_size], factors[train_size:]
    y_train, y_test = returns[:train_size], returns[train_size:]
    
    # 模型
    model = GradientBoostingRegressor(n_estimators=200, max_depth=5)
    model.fit(X_train, y_train)
    
    # 预测
    pred = model.predict(X_test)
    
    return model, pred
```

---

## 七、关键资源

### 7.1 因子数据库

| 数据库 | 说明 | 网址 |
|--------|------|------|
| wind因子库 | 万德量化因子 | wind.com |
| tushare因子 | 免费A股因子 | tushare.pro |
| Bari Fama | FF因子 | mba.tuck.dartmouth.edu |
| AQR因子 | AQR风格因子 | aqrfunds.com |

### 7.2 特征工程库

```bash
# 安装
pip install featuretools      # 自动特征工程
pip install tsfresh           # 时间序列特征
pip install shap             # 模型解释
pip install eli5              # 模型解释
```

### 7.3 机器学习框架

| 框架 | 用途 | GitHub |
|------|------|--------|
| XGBoost | GBDT | github.com/dmlc/xgboost |
| LightGBM | GBDT | github.com/microsoft/LightGBM |
| CatBoost | GBDT | github.com/catboost/catboost |
| PyTorch | 深度学习 | github.com/pytorch/pytorch |
| TensorFlow | 深度学习 | github.com/tensorflow/tensorflow |

---

## 八、最佳实践

### 8.1 因子开发流程

1. **想法生成**: 基于金融理论或观察
2. **数据验证**: 使用历史数据检验
3. **IC分析**: 评估预测能力
4. **组合测试**: 多因子组合效果
5. **实盘模拟**: 小资金验证
6. **全量上线**: 风控监控

### 8.2 常见陷阱

| 陷阱 | 说明 | 解决方案 |
|------|------|----------|
| 前视偏差 | 使用未来信息 | 严格时间序列切分 |
| 过拟合 | 模型过于复杂 | 正则化/交叉验证 |
|幸存者偏差| 只用现存股票 | 包含已退市股票 |
| 因子拥挤 | 过多相同因子 | 相关性分析 |
| 交易成本 | 忽略摩擦成本 | 加入成本模拟 |

### 8.3 因子监控指标

```python
MONITORING_METRICS = {
    'IC': {'threshold': 0.02, 'alert': 'below'},
    'IR': {'threshold': 0.5, 'alert': 'below'},
    'Turnover': {'threshold': 0.5, 'alert': 'above'},
    'Long_short_return': {'threshold': 0.0005, 'alert': 'below'}
}
```

---

*文档持续更新中*