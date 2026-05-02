## 概述

事件驱动策略是量化交易中的核心策略类型之一。事件驱动策略通过识别和利用市场中的特定事件来获取 alpha。
本文档涵盖六种主要事件驱动策略的完整实现。

---

## 一、财报事件策略

### 1.1 盈利预告策略

策略说明：
- 基于分析师预期和历史盈利修正
- 在财报发布前识别可能超预期的股票
- 利用期权市场隐含波动率计算预期移动幅度

核心指标：
- 评级变化：分析师评级上调/下调
- 盈利修正：分析师盈利预测的调整幅度
- 成交量放大：股价异动的早期信号


```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

class EarningsPreviewStrategy:
    """盈利预告策略 - 基于分析师预期和历史盈利修正"""
    
    def __init__(self, lookback_days=60, surprise_threshold=0.05):
        self.lookback_days = lookback_days
        self.surprise_threshold = surprise_threshold
    
    def calculate_expected_move(self, ticker, historical_data):
        """
        计算基于期权的隐含波动率预期移动幅度
        """
        iv = historical_data['implied_volatility'].iloc[-1]
        days_to_expiry = historical_data['days_to_earnings'].iloc[-1]
        
        # 期权定价模型预期的价格移动
        expected_move = iv * np.sqrt(days_to_expiry / 365)
        return expected_move
    
    def identify_earnings_surprise_candidates(self, df):
        """
        识别可能超预期的股票
        - 分析师上调评级
        - 盈利修正向上
        - 近期成交量放大
        """
        df['rating_change'] = df['current_rating'] - df['previous_rating']
        df['estimate_revision'] = (df['current_estimate'] - df['previous_estimate']) / df['previous_estimate']
        
        # 超预期信号
        candidates = df[
            (df['rating_change'] > 0) &
            (df['estimate_revision'] > self.surprise_threshold) &
            (df['volume_ratio'] > 1.5)
        ]
        return candidates
    
    def calculate_position_size(self, expected_move, risk_per_trade=0.02):
        """
        根据预期移动幅度计算仓位
        """
        base_position = risk_per_trade / expected_move
        return min(base_position, 0.1)  # 最大10%仓位限制

    def backtest_earnings_strategy(self, historical_data, entry_days_before=5, exit_days_after=2):
        """
        回测盈利预告策略
        - 在财报发布前N天买入
        - 在财报发布后N天卖出
        """
        results = []
        
        for ticker in historical_data['ticker'].unique():
            ticker_data = historical_data[historical_data['ticker'] == ticker].copy()
            
            earnings_dates = ticker_data[ticker_data['is_earnings']]['date']
            
            for earnings_date in earnings_dates:
                # 入场点：财报前N天
                entry_idx = ticker_data[ticker_data['date'] == earnings_date - timedelta(days=entry_days_before)].index
                if len(entry_idx) == 0:
                    continue
                entry_price = ticker_data.loc[entry_idx[0], 'close']
                
                # 出场点：财报后N天
                exit_idx = ticker_data[ticker_data['date'] == earnings_date + timedelta(days=exit_days_after)].index
                if len(exit_idx) == 0:
                    continue
                exit_price = ticker_data.loc[exit_idx[0], 'close']
                
                # 计算收益
                return_pct = (exit_price - entry_price) / entry_price
                
                # 获取实际盈利数据
                actual_eps = ticker_data.loc[entry_idx[0], 'actual_eps']
                expected_eps = ticker_data.loc[entry_idx[0], 'expected_eps']
                surprise = (actual_eps - expected_eps) / expected_eps if expected_eps != 0 else 0
                
                results.append({
                    'ticker': ticker,
                    'earnings_date': earnings_date,
                    'return_pct': return_pct,
                    'surprise': surprise,
                    'actual_vs_expected': 'beat' if surprise > 0.05 else ('miss' if surprise < -0.05 else 'in-line')
                })
        
        return pd.DataFrame(results)

    def analyze_surprise_patterns(self, backtest_results):
        """
        分析盈利超预期/不及预期的收益模式
        """
        summary = backtest_results.groupby('actual_vs_expected').agg({
            'return_pct': ['mean', 'std', 'count']
        }).round(4)
        
        # 计算夏普比率
        for group in ['beat', 'in-line', 'miss']:
            group_data = backtest_results[backtest_results['actual_vs_expected'] == group]
            if len(group_data) > 1:
                sharpe = group_data['return_pct'].mean() / group_data['return_pct'].std() * np.sqrt(252)
                print(f"{group.upper()}: Mean={group_data['return_pct'].mean():.4f}, Sharpe={sharpe:.2f}, Count={len(group_data)}")
        
        return summary
```

### 1.2 业绩发布策略

```python
class EarningsAnnouncementStrategy:
    """财报发布时的事件驱动策略"""
    
    def __init__(self, hold_period=5, use_iv_crunch=True):
        self.hold_period = hold_period
        self.use_iv_crunch = use_iv_crunch  # 是否考虑期权隐含波动率挤压效应
    
    def calculate_iv_crunch(self, earnings_date, option_chain):
        """
        计算期权隐含波动率挤压效应
        IV在财报前上升，财报后下降 - 这是盈利策略的关键
        """
        pre_earnings_iv = option_chain['iv_before_earnings'].iloc[-1]
        post_earnings_iv = option_chain['iv_after_earnings'].iloc[-1]
        
        iv_crunch_pct = (post_earnings_iv - pre_earnings_iv) / pre_earnings_iv
        return iv_crunch_pct  # 负值表示IV下降
    
    def execute_earnings_trade(self, ticker, earnings_date, option_chain, stock_data):
        """
        执行财报交易
        策略：买入跨式期权或宽跨式期权组合
        """
        current_price = stock_data['close'].iloc[-1]
        implied_vol = option_chain['implied_volatility'].iloc[-1]
        
        # 计算期权价值
        days_to_earnings = (earnings_date - datetime.now()).days
        vega = 0.01 * current_price * implied_vol * np.sqrt(days_to_earnings / 365)
        
        # 预期波动率变化
        if self.use_iv_crunch:
            iv_crunch = self.calculate_iv_crunch(earnings_date, option_chain)
            expected_vol_change = iv_crunch * vega
        else:
            expected_vol_change = 0
        
        return {
            'ticker': ticker,
            'current_price': current_price,
            'implied_vol': implied_vol,
            'expected_move_pct': implied_vol * np.sqrt(days_to_earnings / 365),
            'iv_crunch_pct': iv_crunch if self.use_iv_crunch else None
        }
```

### 1.3 超预期策略

```python
class BeatEstimateStrategy:
    """盈利超预期策略 - 基于实际vs预期盈利比较"""
    
    def analyze_earnings_surprise(self, ticker, actual_eps, expected_eps, revenue_actual, revenue_expected):
        """
        分析盈利超预期程度
        """
        eps_surprise = (actual_eps - expected_eps) / expected_eps if expected_eps != 0 else 0
        rev_surprise = (revenue_actual - revenue_expected) / revenue_expected if revenue_expected != 0 else 0
        
        # 分类超预期程度
        if eps_surprise > 0.05 and rev_surprise > 0.02:
            return 'strong_beat'
        elif eps_surprise > 0.02:
            return 'moderate_beat'
        elif eps_surprise < -0.05:
            return 'strong_miss'
        elif eps_surprise < -0.02:
            return 'moderate_miss'
        else:
            return 'in_line'
    
    def calculate_post_earnings_momentum(self, ticker, earnings_data, window_days=20):
        """
        计算财报后动量 - 超预期股票的持续性
        """
        earnings_data = earnings_data.sort_values('date')
        
        results = []
        for idx, row in earnings_data.iterrows():
            earnings_date = row['date']
            surprise_type = self.analyze_earnings_surprise(
                row['ticker'], row['actual_eps'], row['expected_eps'],
                row['revenue_actual'], row['revenue_expected']
            )
            
            # 计算窗口期收益
            start_price = earnings_data.loc[earnings_date + timedelta(days=1), 'close']
            end_date = earnings_date + timedelta(days=window_days)
            end_price = earnings_data.loc[end_date, 'close']
            
            results.append({
                'ticker': ticker,
                'earnings_date': earnings_date,
                'surprise_type': surprise_type,
                'post_earnings_return': (end_price - start_price) / start_price
            })
        
        return pd.DataFrame(results)
```

---

## 二、宏观事件策略

### 2.1 非农就业策略

```python
import requests
from bs4 import BeautifulSoup
from datetime import datetime

class NonFarmPayrollStrategy:
    """非农就业数据事件驱动策略"""
    
    def __init__(self):
        self.expected_nfp = None
        self.actual_nfp = None
    
    def fetch_consensus(self, date):
        """
        获取市场共识预期
        """
        # 通常需要通过Bloomberg/Reuters API获取
        # 这里演示结构
        return {
            'nfp_expected': 180000,  # 千为单位
            'unemployment_expected': 3.8,
            'wage_growth_expected': 0.3
        }
    
    def calculate_surprise(self, actual, expected):
        """
        计算数据意外程度
        使用标准差标准化
        """
        std_dev = expected * 0.1  # 假设10%的标准差
        return (actual - expected) / std_dev  # Z-score
    
    def trade_nonfarm_release(self, release_time, data):
        """
        非农数据发布后的交易逻辑
        """
        actual_nfp = data['nfp_actual']
        expected_nfp = data['nfp_expected']
        
        surprise = self.calculate_surprise(actual_nfp, expected_nfp)
        
        # 交易逻辑
        if surprise > 1.0:  # 强于预期
            return {
                'action': 'buy_usd',
                'target': 'EURUSD',
                'stop_loss': -0.005,
                'take_profit': 0.01
            }
        elif surprise < -1.0:  # 弱于预期
            return {
                'action': 'sell_usd',
                'target': 'EURUSD',
                'stop_loss': 0.005,
                'take_profit': -0.01
            }
        else:
            return None  # 符合预期，减少交易
```

### 2.2 CPI通胀策略

```python
class CPIInflationStrategy:
    """CPI通胀数据事件驱动策略"""
    
    def calculate_cpi_surprise(self, actual_cpi, expected_cpi, core_cpi=None, food_energy=False):
        """
        计算CPI意外程度
        """
        cpi_surprise = actual_cpi - expected_cpi
        
        if core_cpi is not None:
            # 核心CPI更重要
            surprise_score = cpi_surprise * 0.6 + (core_cpi - expected_cpi) * 0.4
        else:
            surprise_score = cpi_surprise
        
        return surprise_score
    
    def predict_fed_reaction(self, cpi_surprise):
        """
        预测美联储对CPI意外的反应
        """
        if cpi_surprise > 0.3:  # 远超预期
            return {
                'fed_action': 'hike_probability_increase',
                'duration': 'longer_than_expected',
                'risk': 'inflationary'
            }
        elif cpi_surprise < -0.3:  # 低于预期
            return {
                'fed_action': 'pivot_probability_increase',
                'duration': 'rate_cut_possible',
                'risk': 'disinflationary'
            }
        else:
            return {
                'fed_action': 'no_change',
                'duration': 'hold',
                'risk': 'neutral'
            }
```

### 2.3 利率决议策略

```python
class InterestRateStrategy:
    """央行利率决议事件驱动策略"""
    
    def __init__(self):
        self.dovish_keywords = ['accommodative', 'patient', 'supportive', 'measured']
        self.hawkish_keywords = ['vigilant', 'preemptive', 'inflation_focus', 'tightening']
    
    def analyze_statement_tone(self, statement_text):
        """
        分析央行声明的鸽派/鹰派倾向
        """
        dovish_count = sum(1 for word in self.dovish_keywords if word.lower() in statement_text.lower())
        hawkish_count = sum(1 for word in self.hawkish_keywords if word.lower() in statement_text.lower())
        
        tone_score = (hawkish_count - dovish_count) / max(hawkish_count + dovish_count, 1)
        
        if tone_score > 0.3:
            return 'hawkish'
        elif tone_score < -0.3:
            return 'dovish'
        else:
            return 'neutral'
    
    def calculate_rate_expectations(self, fed_funds_rate, dot_plot, forward_rates):
        """
        根据点阵图计算市场利率预期
        """
        median_dot = dot_plot.median()
        
        # 计算当前利率到目标利率的距离
        rate_distance = median_dot - fed_funds_rate
        
        return {
            'current_rate': fed_funds_rate,
            'expected_rate': median_dot,
            'rate_distance': rate_distance,
            'direction': 'hike' if rate_distance > 0 else ('cut' if rate_distance < 0 else 'hold')
        }
```

---

## 三、政策事件策略

### 3.1 产业政策策略

```python
class IndustrialPolicyStrategy:
    """产业政策事件驱动策略"""
    
    def __init__(self):
        self.policy_keywords = {
            'positive': ['支持', '鼓励', '补贴', '扶持', '减税'],
            'negative': ['限制', '监管', '禁止', '打击', '规范']
        }
    
    def extract_policy_signals(self, policy_text):
        """
        从政策文本中提取信号
        """
        positive_signals = sum(1 for word in self.policy_keywords['positive'] if word in policy_text)
        negative_signals = sum(1 for word in self.policy_keywords['negative'] if word in policy_text)
        
        policy_score = positive_signals - negative_signals
        
        return {
            'policy_score': policy_score,
            'positive_count': positive_signals,
            'negative_count': negative_signals,
            'sentiment': 'positive' if policy_score > 0 else ('negative' if policy_score < 0 else 'neutral')
        }
    
    def identify_affected_sectors(self, policy_text, sector_mapping):
        """
        识别受影响的行业板块
        """
        affected = []
        for sector, keywords in sector_mapping.items():
            for keyword in keywords:
                if keyword in policy_text:
                    affected.append(sector)
                    break
        return affected
```

### 3.2 监管变化策略

```python
class RegulatoryChangeStrategy:
    """监管变化事件驱动策略"""
    
    def analyze_regulatory_impact(self, regulation_text, company_exposure):
        """
        分析监管变化对公司的影响
        """
        impact_categories = {
            'compliance_cost': ['合规成本', '罚款', '处罚'],
            'revenue_impact': ['收入影响', '市场准入', '许可'],
            'competitive_advantage': ['竞争优势', '进入壁垒', '垄断']
        }
        
        results = {}
        for category, keywords in impact_categories.items():
            count = sum(1 for kw in keywords if kw in regulation_text)
            results[category] = count
        
        return results
    
    def predict_regulatory_drift(self, historical_events, current_policy):
        """
        预测监管趋势变化方向
        """
        # 基于历史数据的时间序列分析
        import statsmodels.api as sm
        
        X = historical_events['time'].values.reshape(-1, 1)
        y = historical_events['regulatory_index'].values
        
        model = sm.OLS(y, sm.add_constant(X)).fit()
        prediction = model.predict([[current_policy]])
        
        return prediction[0]
```

---

## 四、舆情事件策略

### 4.1 新闻情绪策略

```python
from textblob import TextBlob
import re

class NewsSentimentStrategy:
    """新闻情绪事件驱动策略"""
    
    def __init__(self):
        self.sentiment_threshold = 0.3  # 情绪阈值
        self.volume_threshold = 1.5  # 成交量放大倍数
    
    def clean_text(self, text):
        """清洗新闻文本"""
        text = re.sub(r'<.*?>', ', text)  # 移除HTML标签
        text = re.sub(r'\s+', ' ', text)  # 合并空格
        return text.strip()
    
    def analyze_sentiment(self, news_text):
        """分析新闻情绪"""
        cleaned = self.clean_text(news_text)
        blob = TextBlob(cleaned)
        
        polarity = blob.sentiment.polarity  # -1 to 1
        subjectivity = blob.sentiment.subjectivity  # 0 to 1
        
        return {
            'polarity': polarity,
            'subjectivity': subjectivity,
            'sentiment': 'positive' if polarity > self.sentiment_threshold else
                          ('negative' if polarity < -self.sentiment_threshold else 'neutral')
        }
    
    def calculate_news_score(self, sentiment_data, price_data):
        """计算综合新闻得分"""
        weighted_sentiment = sentiment_data['polarity'] * (1 - sentiment_data['subjectivity'])
        volume_ratio = price_data['volume'] / price_data['avg_volume']
        
        if volume_ratio > self.volume_threshold and abs(weighted_sentiment) > self.sentiment_threshold:
            return weighted_sentiment * volume_ratio
        return 0
```

### 4.2 社交媒体信号策略

```python
import twitter as tw
import snscrape.modules.twitter as sntwitter

class SocialMediaSignalStrategy:
    """社交媒体信号事件驱动策略"""
    
    def __init__(self):
        self.mention_threshold = 100  # 提及次数阈值
        self.viral_velocity = 5  # 病毒式传播速度
    
    def fetch_twitter_sentiment(self, ticker, hours=24):
        """获取推特情绪数据"""
        query = f'${ticker} lang:en until:{datetime.now().strftime('%Y-%m-%d')}'+
              f' since:{(datetime.now() - timedelta(hours=hours)).strftime('%Y-%m-%d')}' 
        
        tweets = []
        for tweet in sntwitter.TwitterSearchScraper(query).get_items():
            if len(tweets) >= 1000:  # 限制数量
                break
            tweets.append({
                'text': tweet.rawContent,
                'date': tweet.date,
                'likes': tweet.likeCount,
                'retweets': tweet.retweetCount
            })
        
        return self.calculate_aggregate_sentiment(tweets)
    
    def calculate_aggregate_sentiment(self, tweets):
        """计算聚合情绪指标"""
        sentiment_scores = [TextBlob(t['text']).sentiment.polarity for t in tweets]
        
        return {
            'mean_sentiment': np.mean(sentiment_scores),
            'median_sentiment': np.median(sentiment_scores),
            'total_mentions': len(tweets),
            'total_engagement': sum(t['likes'] + t['retweets'] for t in tweets)
        }
    
    def detect_sentiment_momentum(self, ticker, time_buckets=12):
        """检测情绪动量变化"""
        # 将时间分成多个桶
        tweets = self.fetch_twitter_sentiment(ticker, hours=time_buckets)
        
        sentiment_trend = tweets['mean_sentiment'].rolling(time_buckets).mean()
        momentum = sentiment_trend.diff()
        
        return 'accelerating' if momentum > 0 else 'decelerating'
```

---

## 五、技术事件策略

### 5.1 突破策略

```python
class BreakoutStrategy:
    """价格突破事件驱动策略"""
    
    def __init__(self, lookback_period=20, volume_multiplier=1.5):
        self.lookback_period = lookback_period
        self.volume_multiplier = volume_multiplier
    
    def identify_support_resistance(self, high_prices, low_prices, window=20):
        """识别支撑和阻力位"""
        resistance_levels = high_prices.rolling(window=window).max()
        support_levels = low_prices.rolling(window=window).min()
        
        return {
            'resistance': resistance_levels,
            'support': support_levels
        }
    
    def detect_breakout(self, current_price, resistance, support, volume, avg_volume):
        """检测突破信号"""
        volume_ratio = volume / avg_volume
        
        # 上突破
        if current_price > resistance and volume_ratio > self.volume_multiplier:
            return {
                'direction': 'upside_breakout',
                'strength': (current_price - resistance) / resistance,
                'volume_confirm': volume_ratio
            }
        
        # 下突破
        if current_price < support and volume_ratio > self.volume_multiplier:
            return {
                'direction': 'downside_breakout',
                'strength': (support - current_price) / support,
                'volume_confirm': volume_ratio
            }
        
        return None
    
    def calculate_breakout_target(self, breakout_price, stop_loss, atr):
        """计算突破交易的目标位"""
        risk = abs(breakout_price - stop_loss)
        reward_to_risk = 2  # 2:1盈亏比
        
        return {
            'target': breakout_price + risk * reward_to_risk,
            'stop_loss': stop_loss,
            'risk_reward_ratio': reward_to_risk,
            'position_size': 0.02 / risk  # 2%风险
        }
```

### 5.2 背离策略

```python
class DivergenceStrategy:
    """价格与指标背离事件驱动策略"""
    
    def calculate_rsi(self, prices, period=14):
        """计算RSI指标"""
        delta = prices.diff()
        gain = (delta.where(delta > 0, 0)).rolling(window=period).mean()
        loss = (-delta.where(delta < 0, 0)).rolling(window=period).mean()
        
        rs = gain / loss
        rsi = 100 - (100 / (1 + rs))
        return rsi
    
    def detect_regular_divergence(self, prices, indicator, lookback=5):
        """检测常规背离"""
        # 价格创新低但指标没有创新低 - 潜在反转买入信号
        price_lows = prices.rolling(lookback).min()
        indicator_lows = indicator.rolling(lookback).min()
        
        price_new_low = prices == price_lows
        indicator_no_new_low = indicator > indicator_lows
        
        bullish_divergence = price_new_low & indicator_no_new_low
        
        # 价格创新高但指标没有创新高 - 潜在反转卖出信号
        price_highs = prices.rolling(lookback).max()
        indicator_highs = indicator.rolling(lookback).max()
        
        price_new_high = prices == price_highs
        indicator_no_new_high = indicator < indicator_highs
        
        bearish_divergence = price_new_high & indicator_no_new_high
        
        return {
            'bullish_divergence': bullish_divergence,
            'bearish_divergence': bearish_divergence
        }
    
    def detect_hidden_divergence(self, prices, indicator, lookback=5):
        """检测隐藏背离 - 趋势延续信号"""
        # 下降趋势中：价格创新低但指标没有 - 趋势延续
        price_higher_swing = prices.diff() > 0  # 上升摆动
        indicator_lower_swing = indicator.diff() < 0  # 指标下降摆动
        
        hidden_bullish = price_higher_swing & indicator_lower_swing
        
        # 上升趋势中：价格创新高但指标没有 - 趋势延续
        price_lower_swing = prices.diff() < 0
        indicator_higher_swing = indicator.diff() > 0
        
        hidden_bearish = price_lower_swing & indicator_higher_swing
        
        return {
            'hidden_bullish': hidden_bullish,
            'hidden_bearish': hidden_bearish
        }
```

### 5.3 形态识别策略

```python
class PatternRecognitionStrategy:
    """技术形态识别事件驱动策略"""
    
    def detect_double_top(self, prices, tolerance=0.02):
        """检测双顶形态"""
        peak_threshold = prices.max() * (1 - tolerance)
        peaks = prices[prices >= peak_threshold]
        
        if len(peaks) >= 2:
            # 检查两个峰值是否相近
            price_diff = abs(peaks.iloc[0] - peaks.iloc[-1]) / peaks.iloc[0]
            if price_diff < tolerance:
                return True
        return False
    
    def detect_head_shoulders(self, prices, lookback=50):
        """检测头肩顶形态"""
        recent = prices[-lookback:]
        
        # 简化检测：寻找三个峰值，中间最高
        peaks = self.find_peaks(recent, n=3)
        
        if len(peaks) == 3:
            left, head, right = peaks
            if head > left and head > right:
                # 检查颈线位置
                neckline = min(recent[peaks[0]:peaks[1]].min(),
                             recent[peaks[1]:peaks[2]].min())
                return {
                    'pattern': 'head_shoulders',
                    'neckline': neckline,
                    'target': neckline - (head - neckline)
                }
        return None
    
    def detect_triangle_pattern(self, prices, lookback=30):
        """检测三角形整理形态"""
        recent = prices[-lookback:]
        
        # 上轨：连接高点
        upper_slope = (recent.max() - recent.iloc[0]) / len(recent)
        
        # 下轨：连接低点
        lower_slope = (recent.min() - recent.iloc[0]) / len(recent)
        
        # 判断三角形类型
        if upper_slope < 0 and lower_slope > 0:
            return 'symmetrical_triangle'
        elif upper_slope < 0:
            return 'descending_triangle'
        elif lower_slope > 0:
            return 'ascending_triangle'
        else:
            return None
```

---

## 六、事件日历

### 6.1 财报日历构建

```python
class EarningsCalendar:
    """财报日历构建"""
    
    def __init__(self):
        self.us_earnings_dates = {}  # 美股财报日期
        self.cn_earnings_dates = {}  # A股财报日期
    
    def scrape_us_earnings(self, start_date, end_date):
        """抓取美股财报日历"""
        # 使用Yahoo Finance API获取财报日期
        from yahoo_earnings_calendar import YahooEarningsCalendar
        
        yec = YahooEarningsCalendar()
        earnings_data = yec.earnings_between(start_date, end_date)
        
        return pd.DataFrame([{
            'ticker': e['ticker'],
            'earnings_date': e['startdate'],
            'type': e.get('earningstype', 'Unknown')
        } for e in earnings_data])
    
    def build_cn_earnings_calendar(self, year, quarter):
        """构建A股财报日历"""
        # 财报预约披露时间表
        schedule = {
            'Q1': ['04-01', '04-30'],  # 一季报
            'Q2': ['07-01', '08-31'],  # 中报
            'Q3': ['10-01', '10-31'],  # 三季报
            'Q4': ['01-01', '04-30']   # 年报 (次年)
        }
        
        return schedule.get(quarter, [])
    
    def calculate_earnings_impact(self, ticker, days_to_event, implied_vol, historical_moves):
        """计算财报潜在影响"""
        # 历史平均移动幅度
        avg_move = historical_moves['post_earnings_move'].abs().mean()
        
        # IV末日论效应
        iv_decay = 0.7  # 假设IV下降30%
        
        return {
            'days_to_event': days_to_event,
            'expected_move': avg_move,
            'iv_decay_factor': iv_decay,
            'trade_recommendation': self.get_trade_recommendation(days_to_event, avg_move)
        }
    
    def get_trade_recommendation(self, days_to_event, expected_move):
        """获取交易建议"""
        if days_to_event <= 3:
            return 'straddle_or_strangle_only'
        elif days_to_event <= 14:
            return 'volatility_play_with_iv_crush'
        elif days_to_event <= 30:
            return 'directional_with_options_hedge'
        else:
            return 'stock_position_before_volatility_expansion'
```

### 6.2 宏观日历构建

```python
class MacroCalendar:
    """宏观事件日历构建"""
    
    def __init__(self):
        self.high_impact_events = [
            'NFP', 'CPI', 'FOMC', 'GDP', 'PMI', 'Retail Sales'
        ]
        self.medium_impact_events = [
            'Initial Claims', 'Housing Starts', 'Industrial Production',
            'Consumer Confidence', 'ISM Manufacturing'
        ]
    
    def fetch_economic_calendar(self, start_date, end_date, timezone='US/Eastern'):
        """获取经济数据日历"""
        import investing.com as ic
        
        calendar = ic.get_calendar(start_date, end_date)
        
        # 分类重要性
        for event in calendar:
            if event['name'] in self.high_impact_events:
                event['impact'] = 'high'
            elif event['name'] in self.medium_impact_events:
                event['impact'] = 'medium'
            else:
                event['impact'] = 'low'
        
        return pd.DataFrame(calendar)
    
    def calculate_historical_volatility_spike(self, event_name, lookback_months=12):
        """计算特定事件后的历史波动率峰值"""
        historical_moves = self.get_historical_event_impact(event_name, lookback_months)
        
        return {
            'avg_vol_spike': historical_moves['vol_spike'].mean(),
            'max_vol_spike': historical_moves['vol_spike'].max(),
            'occurence_count': len(historical_moves)
        }
    
    def build_trading_schedule(self, calendar_df):
        """构建交易时间表"""
        # 只关注高影响事件
        high_impact = calendar_df[calendar_df['impact'] == 'high']
        
        schedule = []
        for _, row in high_impact.iterrows():
            schedule.append({
                'event_date': row['date'],
                'event_name': row['name'],
                'consensus': row.get('consensus'),
                'previous': row.get('previous'),
                'pre_market_prep': self.prepare_pre_market_trade(row),
                'post_release_action': self.get_post_release_action(row)
            })
        
        return pd.DataFrame(schedule)
    
    def prepare_pre_market_trade(self, event_data):
        """盘前准备交易"""
        consensus = event_data.get('consensus')
        previous = event_data.get('previous')
        
        if consensus and previous:
            diff = consensus - previous
            return 'buy_usd' if diff > 0 else 'sell_usd'
        return 'wait_for_release'
    
    def get_post_release_action(self, event_data):
        """发布后交易动作"""
        actual = event_data.get('actual')
        consensus = event_data.get('consensus')
        
        if actual and consensus:
            surprise = actual - consensus
            return 'strong_buy' if surprise > consensus * 0.5 else 
                   ('strong_sell' if surprise < -consensus * 0.5 else 'neutral')
        return 'await_confirmation'
```

---

## 七、事件驱动回测

### 7.1 事件窗口分析

```python
class EventWindowAnalysis:
    """事件窗口分析 - 测量事件前后收益"""
    
    def __init__(self, pre_window=20, post_window=20):
        self.pre_window = pre_window  # 事件前天数
        self.post_window = post_window  # 事件后天数
    
    def calculate_abnormal_return(self, stock_returns, market_returns, event_date_idx):
        """计算异常收益 (CAR - Cumulative Abnormal Return)"""
        # 估计窗口前期的市场模型参数
        pre_event_returns = stock_returns[event_date_idx - self.pre_window:event_date_idx]
        pre_market_returns = market_returns[event_date_idx - self.pre_window:event_date_idx]
        
        # OLS回归估计alpha和beta
        import statsmodels.api as sm
        X = sm.add_constant(pre_market_returns)
        model = sm.OLS(pre_event_returns, X).fit()
        alpha, beta = model.params
        
        # 计算事件窗口的预期收益
        post_event_market = market_returns[event_date_idx:event_date_idx + self.post_window]
        expected_returns = alpha + beta * post_event_market
        
        # 实际收益
        actual_returns = stock_returns[event_date_idx:event_date_idx + self.post_window]
        
        # 异常收益
        abnormal_returns = actual_returns - expected_returns
        
        # 累计异常收益
        cumulative_ar = abnormal_returns.cumsum()
        
        return {
            'abnormal_returns': abnormal_returns,
            'cumulative_ar': cumulative_ar,
            'total_car': cumulative_ar.iloc[-1],
            'alpha': alpha,
            'beta': beta
        }
    
    def run_event_study(self, events_df, stock_data, market_data):
        """运行完整事件研究"""
        results = []
        
        for _, event in events_df.iterrows():
            ticker = event['ticker']
            event_date = event['event_date']
            
            # 找到事件在数据中的索引
            ticker_data = stock_data[stock_data['ticker'] == ticker]
            event_idx = ticker_data[ticker_data['date'] == event_date].index
            
            if len(event_idx) == 0:
                continue
            
            event_result = self.calculate_abnormal_return(
                ticker_data['returns'].values,
                market_data.loc[ticker_data.index, 'returns'].values,
                event_idx[0]
            )
            
            results.append({
                'ticker': ticker,
                'event_date': event_date,
                'event_type': event['type'],
                'car': event_result['total_car']
            })
        
        return pd.DataFrame(results)
```

### 7.2 异常收益计算

```python
class AbnormalReturnCalculator:
    """异常收益计算器 - 支持多种模型"""
    
    def market_model(self, stock_returns, market_returns, estimation_window):
        """市场模型 - 最常用的异常收益计算方法"""
        import statsmodels.api as sm
        
        X = sm.add_constant(market_returns[:estimation_window])
        y = stock_returns[:estimation_window]
        model = sm.OLS(y, X).fit()
        
        predicted = model.predict(sm.add_constant(market_returns[estimation_window:]))
        actual = stock_returns[estimation_window:]
        
        return actual - predicted
    
    def mean_adjusted_model(self, stock_returns, estimation_window):
        """均值调整模型 - 简单但有效"""
        expected_return = stock_returns[:estimation_window].mean()
        actual_returns = stock_returns[estimation_window:]
        
        return actual_returns - expected_return
    
    def capm_model(self, stock_returns, market_returns, risk_free_rate, estimation_window):
        """CAPM模型 - 考虑无风险利率"""
        excess_stock = stock_returns[:estimation_window] - risk_free_rate
        excess_market = market_returns[:estimation_window] - risk_free_rate
        
        # 计算市场溢价
        market_premium = excess_market.mean()
        stock_beta = excess_stock.cov(excess_market) / excess_market.var()
        
        # 预期超额收益
        expected_excess = stock_beta * market_premium
        
        # 异常收益
        actual_excess = stock_returns[estimation_window:] - risk_free_rate
        return actual_excess - expected_excess
    
    def three_factor_model(self, stock_returns, market_returns, smb, hml, estimation_window):
        """三因子模型 (Fama-French)"""
        import statsmodels.api as sm
        
        factors = pd.DataFrame({
            'MKT': market_returns[:estimation_window],
            'SMB': smb[:estimation_window],
            'HML': hml[:estimation_window]
        })
        
        X = sm.add_constant(factors)
        y = stock_returns[:estimation_window]
        model = sm.OLS(y, X).fit()
        
        # 预测事件窗口收益
        test_factors = pd.DataFrame({
            'MKT': market_returns[estimation_window:],
            'SMB': smb[estimation_window:],
            'HML': hml[estimation_window:]
        })
        
        predicted = model.predict(sm.add_constant(test_factors))
        actual = stock_returns[estimation_window:]
        
        return actual - predicted
    
    def calculate_t_statistic(self, abnormal_returns):
        """计算异常收益的t统计量"""
        ar = abnormal_returns.dropna()
        n = len(ar)
        
        if n < 2:
            return None
        
        mean_ar = ar.mean()
        std_ar = ar.std()
        
        t_stat = mean_ar / (std_ar / np.sqrt(n))
        
        return t_stat
```

### 7.3 事件驱动回测框架

```python
class EventDrivenBacktester:
    """事件驱动策略回测框架"""
    
    def __init__(self, initial_capital=100000, commission=0.001):
        self.initial_capital = initial_capital
        self.commission = commission
        self.current_capital = initial_capital
        self.positions = {}
        self.trades = []
    
    def run_backtest(self, events_df, price_data, event_strategies):
        """运行事件驱动回测"""
        results = []
        
        for date in sorted(price_data['date'].unique()):
            day_data = price_data[price_data['date'] == date]
            day_events = events_df[events_df['event_date'] == date]
            
            for event in day_events.itertuples():
                ticker = event.ticker
                event_type = event.type
                if event_type in event_strategies:
                    signal = event_strategies[event_type].generate_signal(event, day_data)
                    self.execute_signal(signal, ticker, day_data)
            
            self.update_positions(day_data)
            
            results.append({
                'date': date,
                'capital': self.current_capital,
                'positions_count': len(self.positions)
            })
        
        return pd.DataFrame(results)

    
    def execute_signal(self, signal, ticker, day_data):
        """执行交易信号"""
        if signal['action'] == 'buy':
            current_price = day_data[day_data['ticker'] == ticker]['close'].iloc[0]
            position_value = self.current_capital * signal.get('size', 0.1)
            shares = position_value / current_price
            cost = shares * current_price * (1 + self.commission)
            if cost <= self.current_capital:
                self.positions[ticker] = {
                    'shares': shares,
                    'entry_price': current_price,
                    'entry_date': day_data['date'].iloc[0]
                }
                self.current_capital -= cost
        elif signal['action'] == 'sell' and ticker in self.positions:
            shares = self.positions[ticker]['shares']
            current_price = day_data[day_data['ticker'] == ticker]['close'].iloc[0]
            proceeds = shares * current_price * (1 - self.commission)
            self.current_capital += proceeds
            del self.positions[ticker]

    
    def update_positions(self, day_data):
        """更新持仓市值"""
        for ticker, position in self.positions.items():
            current_price = day_data[day_data['ticker'] == ticker]['close'].iloc[0]
            position['current_price'] = current_price
            position['market_value'] = position['shares'] * current_price
    
    def calculate_performance(self, results_df):
        """计算回测绩效"""
        results_df['returns'] = results_df['capital'].pct_change()
        total_return = (results_df['capital'].iloc[-1] - self.initial_capital) / self.initial_capital
        sharpe_ratio = results_df['returns'].mean() / results_df['returns'].std() * np.sqrt(252)
        max_drawdown = (results_df['capital'] / results_df['capital'].cummax() - 1).min()
        return {
            'total_return': total_return,
            'sharpe_ratio': sharpe_ratio,
            'max_drawdown': max_drawdown,
            'total_trades': len(self.trades)
        }
```

### 7.4 回测注意事项与最佳实践

```python
# 1. 前视偏差 (Look-ahead Bias) 避免
def avoid_look_ahead_bias(events_df, price_data):
    """确保使用历史数据时没有前视偏差"""
    # 使用事件公告日期而不是实际发生日期
    # 确保数据在事件发生时已经可用
    pass

# 2. 市场影响力考虑
def apply_market_impact(event_size, avg_volume, liquidity_factor=0.1):
    """估算市场影响力成本"""
    participation_rate = event_size / avg_volume
    market_impact = liquidity_factor * participation_rate ** 0.6
    return market_impact

# 3. 交易滑点模拟
def apply_slippage(execution_price, slippage_bps=10):
    """应用交易滑点"""
    return execution_price * (1 + slippage_bps / 10000)
```

---

## 总结与扩展

### 使用示例

```python
# 完整使用示例

# 1. 初始化策略
earnings_strategy = EarningsPreviewStrategy(lookback_days=60, surprise_threshold=0.05)
breakout_strategy = BreakoutStrategy(lookback_period=20, volume_multiplier=1.5)
sentiment_strategy = NewsSentimentStrategy()

# 2. 构建事件日历
earnings_calendar = EarningsCalendar()
macro_calendar = MacroCalendar()

# 3. 回测框架
backtester = EventDrivenBacktester(initial_capital=100000, commission=0.001)
event_strategies = {
    'earnings': earnings_strategy,
    'breakout': breakout_strategy,
    'sentiment': sentiment_strategy
}

# 4. 运行回测
results = backtester.run_backtest(events_df, price_data, event_strategies)
performance = backtester.calculate_performance(results)
print(performance)
```

### 策略组合建议

1. **财报季策略组合**：盈利预告 + IV Crunch + 超预期
2. **宏观事件组合**：非农 + CPI + 利率决议协调交易
3. **技术+情绪组合**：突破 + 新闻情绪 + 社交媒体信号
4. **政策驱动组合**：产业政策 + 监管变化 + 超预期识别

### 风险管理要点

- 每笔交易不超过总资金的2%风险敞口
- 财报事件需考虑期权隐含波动率的IV Crush效应
- 宏观事件交易需设置合理的止损
- 舆情策略需要注意假新闻风险
- 技术突破需要成交量确认

---

*文档版本: 1.0*
*更新时间: 2024*
