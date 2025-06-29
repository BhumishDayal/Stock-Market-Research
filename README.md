# 🧠 n8n Stock Analysis Assistant

## 📘 Project Overview

The **Stock Analysis Assistant** is an automated market insight engine built with **n8n**, designed to help everyday users understand stock behavior — no trading experience required.

### 📈 What It Does

Every weekday during U.S. stock market hours, this workflow:

1. **Monitors major S&P 500 stocks** like TSLA, AAPL, GOOGL, etc.
2. **Fetches historical market data** from Alpaca’s API.
3. **Calculates key technical indicators**:  
   - **RSI (Relative Strength Index)** to gauge momentum.  
   - **MACD (Moving Average Convergence Divergence)** to detect trend changes.
4. **Determines a recommendation status** — `Buy`, `Hold`, or `Sell` — for each stock.
5. **Summarizes the findings** in easy-to-read, plain English using an OpenAI Assistant.
6. **Delivers updates** to Discord and Telegram channels in Markdown, categorized by:
   - 🟢 **Buy Watchlist** – Stocks showing signs of recovery or momentum.
   - ⚪ **Neutral Hold** – Stable or unclear movement.
   - 🔴 **Caution / Sell** – Potential pullbacks or overbought signals.

### 🤖 Why It’s Different

This isn’t just a data pipeline — it’s a **conversational analyst**. The assistant uses familiar language like _“cooling off,” “gaining steam,”_ and _“may attract bargain hunters”_ rather than dense financial jargon. The output is ideal for hobby traders, newsletter curators, Discord communities, or anyone curious about how popular stocks are performing.

### 📅 When It Runs

The workflow is scheduled to run **every 30 minutes from 6:30 AM to 2:30 PM PST**, Monday through Friday, ensuring timely updates during active market hours.

### 📦 Use Cases

- Stock market Telegram/Discord bot
- Weekly investor reports
- Educational financial dashboards
- Market watchlists for small trading groups

---

## 🔧 Setup Instructions

### 1. Schedule Trigger

- **Node Name:** `Schedule Trigger`
- **Purpose:** Runs the analysis on a defined schedule.
- **Cron Expression:** `0 30 6-14 * * 1-5`
- **Schedule:** Every 30 minutes from 6:30 AM to 2:30 PM PST (Monday–Friday).

---

### 2. Market Status Check

#### A. Check Market Status
- **Node Name:** `Check Market Status`
- **Endpoint:** `https://paper-api.alpaca.markets/v2/clock`
- **Authentication:** Set up a Generic HTTP Authentication using Alpaca API credentials.
- **Purpose:** Verifies whether the market is currently open.

#### B. Check if Market is open
- **Node Name:** `Check if Market is open`
- **Logic:** Checks if `is_open` is `true`.
- **True:** Proceeds with analysis.
- **False:** Routes to `Market is Closed`.

#### C. Market is Closed
- **Node Name:** `Market is Closed`
- **Type:** NoOp node that gracefully exits when the market is closed.

---

### 3. Ticker Setup

- **Node Name:** `Ticker List`
- **Type:** `Set`
- **Mode:** `Raw JSON`
- **Data Example:**
```json
{
  "symbols": "AAPL,MSFT,NVDA,TSLA,AMZN,GOOGL,META,JPM,XOM,UNH,GME"
}
```
- **Purpose:** Specifies the list of stock tickers to analyze.

---

### 4. Fetch Stock Data

- **Node Name:** `Fetch Stock Data`
- **URL:** `https://data.alpaca.markets/v2/stocks/bars`
- **Query Parameters:**
  - `symbols`: From `Ticker List`
  - `timeframe`: `1Day`
  - `limit`: `1000`
  - `feed`: `iex`
  - `start`: 100 days ago
  - `end`: today
- **Authentication:** Same Alpaca credentials
- **Purpose:** Retrieves historical daily bar data for each stock.

---

### 5. Interpret Data

- **Node Name:** `Interpret Data`
- **Type:** Python Code
- **Calculations:**
  - RSI(14)
  - MACD(12,26,9)
  - Status: Buy, Hold, or Sell
- **Outputs:**
  - `stocks`: Array of objects with ticker, RSI, MACD, signal, and status.
  - `summary`: A compact JSON string for GPT prompt use.

---

### 6. AI Assistant Summary

- **Node Name:** `Stock Analysis Assistant`
- **Type:** OpenAI Assistant Node
- **Prompt Includes:**
  - `summary` JSON
  - `timestamp` via `{{ $now }}`
- **Expected Output:**
  - Categorized stock commentary:
    - 🟢 Buy Watchlist
    - ⚪ Neutral Hold
    - 🔴 Caution / Sell
  - Plain English insights using markdown
  - Example output:
```markdown
📊 Market Summary

🟢 Buy Watchlist  
• TSLA – Recovering after a dip; gaining steam. This type of rebound often attracts early buyers.

⚪ Neutral Hold  
• AAPL – Holding steady. This often means the market is waiting on new developments.

🔴 Caution / Sell  
• GOOGL – Appears overbought; may pull back. A good time for caution.
```

---

### 7. Post to Discord and Telegram

#### A. Send a message (Discord)
- **Node Name:** `Send a message`
- **Content:** `{{ $json.output }}`
- **Target:** Configure with appropriate Guild and Channel ID.

#### B. Send a text message (Telegram)
- **Node Name:** `Send a text message`
- **Text:** `{{ $json.output }}`
- **Target:** Configure with the appropriate Telegram chat/channel.

---

### 8. End of Flow
- **Node Name:** `End of Flow`
- **Type:** NoOp
- **Purpose:** Terminates the workflow after messages are sent.

---

## 📌 Notes

- Update the **Ticker List** node to include your desired stock symbols.
- Make sure **Alpaca API keys** are set for both `/v2/clock` and `/v2/stocks/bars` endpoints.
- OpenAI Assistant ID must be configured in `Stock Analysis Assistant` node.
- Discord and Telegram nodes must be authorized and correctly configured.
