# Automated Trading System with TradingView & Angel One Integration

> **Note:** Full source code is proprietary under the Softwired Internship and is not publicly shared.

---
Our project is an end-to-end automated trading platform designed to take a custom technical strategy from initial idea all the way through to live execution on a broker’s exchange, with an optional paper-trading mode for safe experimentation. We begin by formulating a strategy in Pine Script on TradingView—defining entry and exit rules based on indicators such as SuperTrend or MACD, and backtesting it across different chart intervals (for example, 2-minute or 5-minute timeframes). Within the same script, we embed alert conditions for buy and sell signals (e.g., “BUY_CALL” or “SELL_PUT”), and configure TradingView’s webhook feature to send the alert payload—formatted as JSON containing symbol, strategy name, action, timestamp, and other metadata—to our backend URL whenever a signal fires.

Once an alert arrives at our backend, it is authenticated and immediately handed off to a Celery worker for asynchronous processing. The worker retrieves the appropriate user’s Angel One SmartAPI credentials (Client ID, API Key, password, and TOTP token), validates the message against the strategy configured for that webhook link, and places either a real order on the broker or simulates it entirely in paper-trading mode. Simultaneously, our system establishes a live market-data subscription over WebSocket, broadcasting price updates via Redis Pub/Sub so that every tick can be evaluated against stop-loss, profit-target, and trailing-stop rules. When any of those conditions are met, the worker issues the corresponding sell order and tears down the subscription, ensuring that each position’s lifecycle is managed cleanly.

Beyond simple execution, the platform addresses numerous edge cases—partial fills, API downtime, flash crashes, and market-close behavior. Every day at 3:29 PM IST, a scheduled job inspects all open positions: profitable trades are held overnight, while losing positions are closed to limit risk. Users interact with a React-based dashboard where they can create and manage accounts, configure webhook links tied to specific strategies, toggle paper trading on or off, and start or stop the automated “bot,” which drains pending orders but ignores any new alerts until reactivated. Finally, the entire application stack—Django REST API, Celery workers, Redis, React frontend—is deployed on AWS EC2 behind NGINX with systemd service managers, providing a scalable, fault-tolerant environment for real-time, algorithmic trading.
Automated Trading System
├─ 1. Strategy Development
│   ├─ Define technical indicators & rules
│   ├─ Backtest on multiple timeframes (2m, 5m, …)
│   └─ Implement in Pine Script
│       ├─ alertcondition() for BUY_CALL / SELL_PUT
│       └─ JSON payload fields:
│           ├─ strategy
│           ├─ symbol
│           ├─ action
│           └─ timestamp
│
├─ 2. TradingView Setup
│   ├─ Create alerts in TradingView
│   ├─ Paste “Webhook URL” from dashboard
│   └─ Ensure JSON schema matches backend expectations
│
├─ 3. Web Dashboard Configuration
│   ├─ User & Account Management
│   │   ├─ Register user (username, password, first/last name)
│   │   └─ Link Angel One account (Client ID, API Key, Password, TOTP)
│   ├─ Webhook Link Definition
│   │   ├─ Name
│   │   ├─ Select Account
│   │   ├─ Strategy (must match payload)
│   │   ├─ Stop-Loss %, Target %, Trailing-Stop %
│   │   └─ Paper-Trading toggle
│   └─ Dashboard Features
│       ├─ View live & historical orders
│       ├─ Download / delete history
│       ├─ Start / Stop bot control
│       └─ System logs & status
│
├─ 4. Alert Reception & Processing
│   ├─ Webhook endpoint receives JSON POST
│   ├─ Authenticate & validate strategy
│   ├─ Enqueue task to Celery worker
│   └─ Respond <200 ms to TradingView
│
├─ 5. Order Execution
│   ├─ Credential Retrieval
│   │   └─ Fetch & refresh SmartAPI tokens if needed
│   ├─ Live vs. Paper Mode
│   │   ├─ Live → place real order via SmartAPI
│   │   └─ Paper → simulate & log only
│   ├─ On BUY execution:
│   │   ├─ Record entry price & quantity
│   │   └─ Subscribe to live price feed
│   └─ On SELL execution:
│       ├─ Place market sell (or simulate)
│       └─ Unsubscribe from price feed
│
├─ 6. Real-Time Monitoring
│   ├─ WebSocket client → exchange feed
│   ├─ Redis Pub/Sub → broadcast ticks
│   └─ Evaluate each tick for:
│       ├─ Stop-Loss condition
│       ├─ Target condition
│       └─ Trailing-Stop adjustment
│
├─ 7. Edge-Case & Error Handling
│   ├─ Partial fills → track remaining qty
│   ├─ API downtime → retry with backoff
│   ├─ Flash crashes & price gaps → fail-safe logic
│   └─ Duplicate alerts → idempotent processing
│
├─ 8. End-of-Day Routine (3:29 PM IST)
│   ├─ Celery Beat schedules EOD job
│   ├─ Compile open positions JSON:
│   │   [{"symbol","expiry","strike"}, …]
│   ├─ For each position:
│   │   ├─ If profit → hold overnight
│   │   └─ If loss   → force-sell immediately
│   └─ Save EOD report/log
│
├─ 9. Bot Control
│   ├─ Stop Bot → finish pending, ignore new alerts
│   └─ Start Bot → resume processing new alerts
│
├─10. Data Persistence & Reporting
│   ├─ Database stores:
│   │   ├─ Users, Accounts
│   │   ├─ Webhook Links
│   │   ├─ Orders (live & paper)
│   │   └─ System Logs
│   └─ Dashboard CSV export & delete functions
│
├─11. Tech Stack
│   ├─ Front-End: React, Vite, Axios, React Router
│   ├─ Back-End: Django REST, Celery, Redis
│   ├─ Broker API: Angel One SmartAPI
│   ├─ Data Stores: SQLite (dev), Redis (cache & Pub/Sub)
│   └─ Deployment: AWS EC2 (Ubuntu), NGINX, Gunicorn, systemd
│
└─12. Deployment & Scaling
    ├─ Provision EC2 instance(s)
    ├─ Install & configure NGINX reverse proxy
    ├─ Set up systemd services:
    │   ├─ Gunicorn (Django)
    │   ├─ Celery worker & beat
    │   └─ Redis (optional)
    ├─ Manage secrets via .env or AWS Secrets Manager
    └─ Scale Celery horizontally; monitor logs & metrics

## Table of Contents

- [Overview](#overview)  
- [Workflow & Strategy Development](#workflow--strategy-development)  
- [TradingView Integration](#tradingview-integration)  
- [Backend Architecture](#backend-architecture)  
  - [Broker (Angel One) Integration](#broker-angel-one-integration)  
  - [User & Account Management](#user--account-management)  
  - [Webhook Link Configuration](#webhook-link-configuration)  
  - [Paper Trading Mode](#paper-trading-mode)  
- [Live Monitoring & Order Execution](#live-monitoring--order-execution)  
  - [WebSocket & Redis Price Feed](#websocket--redis-price-feed)  
  - [Subscription Lifecycle](#subscription-lifecycle)  
  - [Sell Conditions & Edge Cases](#sell-conditions--edge-cases)  
  - [Bot Control (Start/Stop)](#bot-control-startstop)  
- [Scheduling & End-of-Day Logic](#scheduling--end-of-day-logic)  
- [Web Dashboard Features](#web-dashboard-features)  
- [Tech Stack](#tech-stack)  
- [Setup & Installation](#setup--installation)  
- [Configuration & Environment Variables](#configuration--environment-variables)  
- [Running Locally](#running-locally)  
- [Testing](#testing)  
- [Deployment on AWS](#deployment-on-aws)  
- [Project Structure](#project-structure)  
- [License](#license)  

---

## Overview

This project implements a fully automated trading system that:

1. **Develops and backtests** custom stock-market strategies in Pine Script on TradingView.  
2. **Generates webhook alerts** for buy/sell signals (Put/Call).  
3. **Receives alerts** via a secure backend, routes them to Celery for async processing.  
4. **Executes orders** live on Angel One (via SmartAPI) or in “paper trading” mode for testing.  
5. **Monitors positions** in real time using WebSocket and Redis.  
6. **Handles sell logic** (stop-loss, target, trailing stop, end-of-day exit) with robust edge-case coverage.  
7. **Provides a React dashboard** for management, history, and control (start/stop bot).  
8. **Deploys** the entire stack on AWS EC2 with NGINX, systemd, and environment-based secrets.

---

## Workflow & Strategy Development

1. **Strategy Creation & Analysis**  
   - Define technical indicators, entry/exit rules in Pine Script.  
   - Backtest across multiple timeframes (2 min, 5 min, etc.) to evaluate P&L, drawdowns.  

2. **Pine Script Alerts**  
   - Embed `alertcondition()` calls for Put/Call buy signals.  
   - Customize alert message format (JSON payload) including:  
     ```json
     {
       "strategy": "MySuperTrend",
       "symbol": "RELIANCE",
       "action": "BUY_CALL",
       "timestamp": "{{timenow}}"
     }
     ```
   - Enable TradingView’s webhook URL feature to POST alerts to your backend.

---

## TradingView Integration

- **Webhook Configuration**  
  - Paste your backend’s “Webhook Link” URL into TradingView alert dialog.  
  - Ensure payload matches your backend’s expected JSON schema.  

- **Strategy Filtering**  
  - Multiple strategies may fire alerts simultaneously.  
  - Each “Webhook Link” is tied to a specific strategy name.  
  - Backend ignores alerts whose `strategy` field does not match the configured link.

---

## Backend Architecture

```mermaid
flowchart LR
    TV[TradingView Alert] -->|Webhook POST| API[Backend API]
    API -->|Enqueue| Celery[Celery Worker]
    Celery --> Broker[Angel One SmartAPI]
    Celery --> Redis[Redis Pub/Sub]
    Redis -->|Price Updates| Monitor[Live Monitor Service]
    Monitor --> Celery
    Monitor -->|Sell Order| Broker
```

### Broker (Angel One) Integration

- **SmartAPI Credentials**  
  - **Client ID**  
  - **API Key**  
  - **Password**  
  - **TOTP Token**  
- Securely stored and rotated via Celery tasks.

### User & Account Management

- **User Fields:**  
  | Field       | Description             |
  |-------------|-------------------------|
  | Username    | Unique login ID         |
  | Password    | Hashed via Django       |
  | First Name  | User’s first name       |
  | Last Name   | User’s last name        |
- **Account Linking:**  
  - One user → One or more Angel One accounts.  
  - Each account holds its own SmartAPI credentials.

### Webhook Link Configuration

| Field                   | Description                                                        |
|-------------------------|--------------------------------------------------------------------|
| **Name**                | Friendly identifier (e.g., “ST Trend Bot”)                         |
| **Account**             | Select from linked Angel One accounts                              |
| **Strategy**            | Strategy name (matches `strategy` in Pine Script alert payload)    |
| **Stop-Loss (%)**       | Maximum allowable loss per trade                                   |
| **Target (%)**          | Profit target per trade                                            |
| **Trailing Stop (%)**   | Dynamic exit buffer                                               |
| **Paper Trading**       | ☑️ Enable simulated orders (no real broker calls)                  |

### Paper Trading Mode

- **Simulation Only:**  
  - Orders are **calculated** and **logged**, but **not** sent to Angel One.  
  - Allows beginners to test strategies without capital risk.

---

## Live Monitoring & Order Execution

### WebSocket & Redis Price Feed

- **WebSocket Client** subscribes to live market data (exchange feed).  
- **Redis Pub/Sub** broadcasts price updates to monitoring services.

### Subscription Lifecycle

1. **On Buy Execution:**  
   - Subscribe to symbol’s price channel.  
2. **Continuous Monitoring:**  
   - Evaluate stop-loss/target/trailing conditions on each tick.  
3. **On Sell Execution or Bot Stop:**  
   - Unsubscribe from symbol to free resources.

### Sell Conditions & Edge Cases

- **Stop-Loss:** Immediate market sell when price ≤ entry × (1 – SL%).  
- **Target:** Market sell when price ≥ entry × (1 + Target%).  
- **Trailing Stop:** Adjust stop-loss upward as price moves in favor.  
- **Edge Cases:**  
  - **Flash crashes** (price gap)  
  - **Re-quotes** or **order rejections**  
  - **Partial fills** (track remaining quantity)  
  - **API downtime** (retry logic)

### Bot Control (Start/Stop)

- **Stop Bot:**  
  - Completes all **pending** orders.  
  - **Ignores** new incoming alerts until restarted.  
- **Start Bot:**  
  - Resumes processing of fresh alerts.

---

## Scheduling & End-of-Day Logic

- **Order List JSON Generation**  
  - Daily Cron (via Celery Beat) compiles JSON of all open positions:  
    ```json
    [
      {"symbol": "RELIANCE", "expiry": "2025-05-08", "strike": 2450},
      …
    ]
    ```
- **3:29 PM Close Routine**  
  - Market closes at 3:30 PM IST.  
  - At 3:29 PM, check all open positions:  
    - If **in profit**, keep overnight.  
    - If **not in profit**, force-sell to cut losses.

---

## Web Dashboard Features

- **Dashboard Views:**  
  - **Active Orders** (live & paper)  
  - **Order History** (with Download & Delete options)  
  - **Account & Credential Management**  
  - **Webhook Links** (CRUD)  
  - **Bot Status** (Start/Stop toggle)  
  - **System Logs** (task outcomes, errors)

---

## Tech Stack

| Layer        | Technology                                  |
|--------------|----------------------------------------------|
| Front-End    | React, Vite, Axios, React Router             |
| Back-End     | Django REST Framework, Celery, Redis         |
| Broker API   | Angel One SmartAPI                           |
| Data Store   | SQLite (dev), Redis (cache & pub/sub)        |
| Hosting      | AWS EC2 (Ubuntu), NGINX, systemd             |
| CI/CD        | Git (private repo), GitHub Actions (optional)|

---

## Setup & Installation

1. **Clone Repo**  
   ```bash
   git clone <private-repo-url>
   cd automated-trading-system
   ```

2. **Backend**  
   ```bash
   cd backend/
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. **Frontend**  
   ```bash
   cd ../frontend/
   npm install
   ```

---

## Configuration & Environment Variables

### Backend (`backend/.env`)

```dotenv
SECRET_KEY=…
TRADING_MODE=paper       # or live
BROKER_CLIENT_ID=…
BROKER_API_KEY=…
BROKER_PASSWORD=…
BROKER_TOTP_SECRET=…
REDIS_URL=redis://localhost:6379/0
```

### Frontend (`frontend/.env`)

```dotenv
VITE_BACKEND_URL=http://localhost:8000
```

---

## Running Locally

```bash
# Start Redis
redis-server

# Backend
cd backend/
python manage.py migrate
python manage.py runserver
celery -A backend worker --loglevel=info &
celery -A backend beat --loglevel=info &

# Frontend
cd ../frontend/
npm run dev
```

---

## Testing

- **Backend Tests**  
  ```bash
  cd backend/
  pytest
  ```
- **Frontend Tests**  
  ```bash
  cd frontend/
  npm run test
  ```

---

## Deployment on AWS

1. **Provision EC2 (Ubuntu)**  
2. **Install Dependencies** (Python, Node.js, Redis, NGINX)  
3. **Clone & Build** the repo  
4. **Configure NGINX** reverse proxy to serve React and Django  
5. **Create systemd services** for:  
   - `gunicorn` (Django)  
   - `celery-worker` & `celery-beat`  
   - `react-app` (if built)  
6. **Environment Variables** via `/etc/profile.d/` or AWS Secrets Manager  
7. **Enable & Start** services  
8. **Open Ports** (80, 443 for HTTPS)

---

## Project Structure

```
/
├── backend/
│   ├── backend/          # Django project
│   ├── users/            # User & auth
│   ├── broker/           # Angel One SmartAPI integration
│   ├── strategies/       # Webhook & tasks
│   ├── celery.py
│   ├── manage.py
│   └── requirements.txt
└── frontend/
    ├── public/
    ├── src/              # React app
    ├── vite.config.js
    └── package.json
```

---

## License

Proprietary—developed under **Softwired Internship**. Redistribution or disclosure of code is restricted.
