import yfinance as yf
import pandas as pd
from flask import Flask, jsonify
import numpy as np

app = Flask(__name__)

def ai_signal_logic(gold):
    score = 0

    # 1 — قوة المشترين والبائعين
    gold["Body"] = gold["Close"] - gold["Open"]
    gold["VolumePower"] = abs(gold["Body"]) * gold["Volume"]

    buyers = gold[gold["Body"] > 0]["VolumePower"].sum()
    sellers = gold[gold["Body"] < 0]["VolumePower"].sum()

    if buyers > sellers * 1.3:
        score += 2
    elif sellers > buyers * 1.3:
        score -= 2

    # 2 — اتجاه آخر 10 شموع
    last_10 = gold["Close"].tail(10).values
    if last_10[-1] > last_10[0]:
        score += 1
    else:
        score -= 1

    # 3 — السيولة
    liquidity_high = gold["High"].tail(20).mean()
    liquidity_low = gold["Low"].tail(20).mean()

    last_price = gold["Close"].iloc[-1]

    # 4 — قرب السعر من مناطق الشراء أو البيع
    if last_price <= liquidity_low * 1.002:
        score += 2
    if last_price >= liquidity_high * 0.998:
        score -= 2

    # 5 — الزخم (Momentum)
    gold["Momentum"] = gold["Close"].diff()
    momentum = gold["Momentum"].tail(8).sum()

    if momentum > 0:
        score += 1
    else:
        score -= 1

    # 6 — المتوسط الحسابي MA20
    gold["MA20"] = gold["Close"].rolling(20).mean()
    if gold["Close"].iloc[-1] > gold["MA20"].iloc[-1]:
        score += 1
    else:
        score -= 1

    # 7 — شموع انعكاس
    last_candle = gold.tail(1).iloc[0]
    upper_wick = last_candle["High"] - max(last_candle["Close"], last_candle["Open"])
    lower_wick = min(last_candle["Close"], last_candle["Open"]) - last_candle["Low"]

    if lower_wick > abs(last_candle["Body"]) * 1.5:
        score += 1
    if upper_wick > abs(last_candle["Body"]) * 1.5:
        score -= 1

    # النتيجة النهائية
    if score >= 4:
        return "📈 AI SIGNAL: STRONG BUY"
    elif score <= -4:
        return "📉 AI SIGNAL: STRONG SELL"
    else:
        return "⏳ AI SIGNAL: WAIT (NO TRADE)"


def analyze_gold():
    gold = yf.download("GC=F", period="2d", interval="15m")

    if gold.empty:
        return {"error": "No data received"}

    # النتيجة من الذكاء الاصطناعي
    ai_signal = ai_signal_logic(gold)

    last_price = float(gold["Close"].iloc[-1])
    liquidity_high = float(gold["High"].tail(20).mean())
    liquidity_low = float(gold["Low"].tail(20).mean())

    return {
        "last_price": round(last_price, 2),
        "liquidity_high_zone": round(liquidity_high, 2),
        "liquidity_low_zone": round(liquidity_low, 2),
        "AI_signal": ai_signal,
    }


@app.route("/")
def home():
    return jsonify(analyze_gold())

if __name__ == "__main__":
    app.run()
