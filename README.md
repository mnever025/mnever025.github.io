# mnever025.github.io
Useable Penny Stock Predicter
penny-stock-extension/
 ├ manifest.json
 ├ popup.html
 ├ popup.js
 └ styles.css
{
  "manifest_version": 3,
  "name": "Penny Stock Trend Analyzer",
  "version": "1.0",
  "description": "Analyzes penny stocks using market data and news sentiment.",
  "action": {
    "default_popup": "popup.html"
  },
  "permissions": ["storage"],
  "host_permissions": [
    "https://api.marketstack.com/*",
    "https://newsapi.org/*"
  ]
}
<!DOCTYPE html>
<html>
<head>
  <title>Penny Stock Analyzer</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h2>Penny Stock Scanner</h2>
  <button id="scan">Scan Market</button>
  <ul id="results"></ul>

  <script src="popup.js"></script>
</body>
</html>
const MARKETSTACK_KEY = "YOUR_API_KEY";
const NEWS_KEY = "YOUR_API_KEY";

document.getElementById("scan").addEventListener("click", scanStocks);

async function scanStocks() {
  const list = document.getElementById("results");
  list.innerHTML = "Scanning...";

  const stocks = await fetch(
    `http://api.marketstack.com/v1/eod?access_key=${MARKETSTACK_KEY}&limit=50`
  ).then(r => r.json());

  const pennyStocks = stocks.data.filter(s => s.close < 5);

  const scored = await Promise.all(
    pennyStocks.map(async stock => {
      const sentiment = await getNewsScore(stock.symbol);
      const trendScore = stock.close - stock.open;

      return {
        symbol: stock.symbol,
        price: stock.close,
        score: trendScore + sentiment
      };
    })
  );

  scored.sort((a, b) => b.score - a.score);

  list.innerHTML = "";
  scored.slice(0, 10).forEach(s => {
    const li = document.createElement("li");
    li.textContent = `${s.symbol} — $${s.price} (score: ${s.score.toFixed(2)})`;
    list.appendChild(li);
  });
}

async function getNewsScore(symbol) {
  const news = await fetch(
    `https://newsapi.org/v2/everything?q=${symbol}&apiKey=${NEWS_KEY}`
  ).then(r => r.json());

  if (!news.articles) return 0;

  let score = 0;

  news.articles.forEach(article => {
    const text = article.title.toLowerCase();

    if (text.includes("growth")) score += 1;
    if (text.includes("profit")) score += 1;
    if (text.includes("lawsuit")) score -= 1;
    if (text.includes("loss")) score -= 1;
  });

  return score;
}
body {
  font-family: Arial;
  width: 300px;
  padding: 10px;
}

button {
  width: 100%;
  padding: 8px;
}

li {
  margin: 5px 0;
}
chrome://extensions
Chrome Extension (UI)
        ↓
Backend API (Python / Node)
        ↓
AI Scoring Engine
        ↓
Market + News + Macro Data
Momentum score
Volume spike score
Volatility score
Sentiment score
Macro-risk score
AI Score =
0.35 Momentum +
0.25 Sentiment +
0.20 Volume +
0.10 Volatility +
0.10 Macro
pip install yfinance pandas numpy scikit-learn vaderSentiment
import yfinance as yf
import pandas as pd
import numpy as np
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

analyzer = SentimentIntensityAnalyzer()

penny_tickers = ["SNDL", "NAKD", "IDEX", "ZOM", "HCMC"]

def momentum_score(df):
    return (df["Close"].iloc[-1] - df["Close"].iloc[-5]) / df["Close"].iloc[-5]

def volume_score(df):
    avg = df["Volume"].rolling(10).mean().iloc[-1]
    return df["Volume"].iloc[-1] / avg

def volatility_score(df):
    return df["Close"].pct_change().std()

def sentiment_score(news_titles):
    scores = [analyzer.polarity_scores(t)["compound"] for t in news_titles]
    return np.mean(scores) if scores else 0

results = []

for ticker in penny_tickers:
    df = yf.download(ticker, period="1mo", interval="1d")

    if len(df) < 10:
        continue

    m = momentum_score(df)
    v = volume_score(df)
    vol = volatility_score(df)

    fake_news = [
        f"{ticker} announces growth opportunity",
        f"{ticker} expands operations"
    ]

    s = sentiment_score(fake_news)

    ai_score = (
        0.35 * m +
        0.25 * s +
        0.20 * v +
        0.10 * vol
    )

    results.append((ticker, ai_score))

results.sort(key=lambda x: x[1], reverse=True)

print("\nTop Penny Stock Candidates:")
for r in results:
    print(r)
from fastapi import FastAPI
from ai_engine import results

app = FastAPI()

@app.get("/scan")
def scan():
    return {"stocks": results}
uvicorn server:app --reload
async function scanStocks() {
  const res = await fetch("http://127.0.0.1:8000/scan");
  const data = await res.json();

  const list = document.getElementById("results");
  list.innerHTML = "";

  data.stocks.forEach(s => {
    const li = document.createElement("li");
    li.textContent = `${s[0]} (AI score: ${s[1].toFixed(2)})`;
    list.appendChild(li);
  });
}
import streamlit as st

st.title("AI Penny Stock Screener")

if st.button("Scan Market"):
    st.write(results)
https://ai-stock-screener.streamlit.app
