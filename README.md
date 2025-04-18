# === 📁 AI Trading CEO System ===
# Folder Structure:
# ai_trading_ceo/
# ├── brain.py            # Core AI decision engine
# ├── strategist.py       # Entry/Exit planner
# ├── bot.py              # Order execution via broker
# ├── riskmanager.py      # SL/TP/Trail logic
# ├── dashboard.py        # Local FastAPI UI
# ├── data_handler.py     # Market data fetcher
# ├── config.py           # User/broker settings
# ├── logs/               # Logs & results
# └── main.py             # Runs the system

# === FILE: brain.py ===
# AI logic for detecting trend, AOI, patterns, entries

import pandas as pd
import ta

class TradingBrain:
    def __init__(self, df):
        self.df = df
        self.signals = []

    def compute_indicators(self):
        self.df['ma200'] = ta.trend.sma_indicator(self.df['close'], window=200)
        self.df['atr'] = ta.volatility.average_true_range(self.df['high'], self.df['low'], self.df['close'])

    def detect_trend(self):
        return 'up' if self.df['close'].iloc[-1] > self.df['ma200'].iloc[-1] else 'down'

    def detect_engulfing(self):
        last = self.df.iloc[-1]
        prev = self.df.iloc[-2]
        if last['close'] > last['open'] and prev['close'] < prev['open'] and last['close'] > prev['open']:
            return 'bullish'
        elif last['close'] < last['open'] and prev['close'] > prev['open'] and last['close'] < prev['open']:
            return 'bearish'
        return None

    def generate_signal(self):
        self.compute_indicators()
        trend = self.detect_trend()
        pattern = self.detect_engulfing()

        if trend == 'up' and pattern == 'bullish':
            return {'signal': 'buy'}
        elif trend == 'down' and pattern == 'bearish':
            return {'signal': 'sell'}
        return {'signal': None}

# === FILE: strategist.py ===
# Plans entries, calculates SL/TP, and returns a trade plan

class Strategist:
    def __init__(self, price, atr, rr=2.0):
        self.price = price
        self.atr = atr
        self.rr = rr

    def plan_trade(self, direction):
        if direction == 'buy':
            sl = self.price - self.atr
            tp = self.price + self.atr * self.rr
        elif direction == 'sell':
            sl = self.price + self.atr
            tp = self.price - self.atr * self.rr
        else:
            return None

        return {
            'direction': direction,
            'entry': self.price,
            'sl': sl,
            'tp': tp
        }

# Usage example:
# strat = Strategist(price=1.0845, atr=0.0012)
# plan = strat.plan_trade('buy')
# print(plan)
