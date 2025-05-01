```markdown
# Automated Trading System with TradingView & Angel One Integration

> ⚠️ **Confidential:** Developed under the Softwired Internship. Full source code is proprietary and not publicly shared.

---

## Table of Contents

1. [Project Overview](#project-overview)  
2. [Key Concepts & Workflow](#key-concepts--workflow)  
3. [Detailed Architecture](#detailed-architecture)  
4. [Features & Capabilities](#features--capabilities)  
5. [Tech Stack](#tech-stack)  
6. [Getting Started](#getting-started)  
   - [Prerequisites](#prerequisites)  
   - [Clone & Install](#clone--install)  
   - [Environment Configuration](#environment-configuration)  
   - [Database Setup & Migrations](#database-setup--migrations)  
   - [Running Services Locally](#running-services-locally)  
7. [Usage Guide](#usage-guide)  
   - [1. Strategy Development & Alerts on TradingView](#1-strategy-development--alerts-on-tradingview)  
   - [2. Webhook Configuration](#2-webhook-configuration)  
   - [3. Broker Account Onboarding](#3-broker-account-onboarding)  
   - [4. Creating & Managing Strategies](#4-creating--managing-strategies)  
   - [5. Paper Trading vs. Live Trading](#5-paper-trading-vs-live-trading)  
   - [6. Real-Time Price Monitoring & Order Execution](#6-real-time-price-monitoring--order-execution)  
   - [7. Bot Control (Start / Stop)](#7-bot-control-start--stop)  
   - [8. Viewing & Managing Trade History](#8-viewing--managing-trade-history)  
8. [Order Lifecycle & Edge-Case Handling](#order-lifecycle--edge-case-handling)  
9. [Deployment on AWS](#deployment-on-aws)  
10. [Security & Best Practices](#security--best-practices)  
11. [Testing](#testing)  
12. [Contributing](#contributing)  
13. [License](#license)  

---

## Project Overview

An **end-to-end automated trading system** that:

- Allows you to **design your own Pine Script strategies** on TradingView.
- Emits **alert webhooks** from TradingView when buy/sell conditions are met.
- Processes alerts on a **Django + Celery** backend, applying user-configured stop-loss, target, and trailing-stop logic.
- Executes orders automatically via **Angel One’s SmartAPI** or simulates them in **paper-trading mode**.
- Provides a **React/Vite** dashboard to manage users, broker accounts, strategies, webhooks, and trade history.
- Streams live market data via **WebSocket + Redis** for real-time monitoring.
- Deployed on **AWS EC2** with **NGINX**, **systemd**, **Redis**, and **Celery** for scalability and reliability.

---

## Key Concepts & Workflow

1. **Strategy Definition**  
   - Develop & backtest your trading logic in Pine Script on TradingView (e.g., 2-min, 5-min charts).  
   - Embed alert conditions (`BUY`, `SELL`, `PUT`, `CALL`) in the script.

2. **Alert → Webhook**  
   - Configure TradingView alerts to send JSON payloads to a custom webhook URL.  
   - Format payload with placeholders (e.g. `{{ticker}}`, `{{strategy}}`, `{{price}}`).

3. **Webhook Reception**  
   - Django receives the POST, authenticates signature (optional), enqueues a Celery task.

4. **Order Processing**  
   - Task loads user’s Angel One API credentials (Client ID, API Key, Password, TOTP).  
   - Validates symbol, side (PUT/CALL), and applies configured stop-loss, targets, trailing stops.  
   - If **live mode**, sends order via SmartAPI. If **paper mode**, simulates and logs.

5. **Live Market Data**  
   - Backend subscribes to real-time price feeds over WebSocket.  
   - Price updates pushed into Redis; Celery workers consume updates.

6. **Sell Conditions & Exit Logic**  
   - Continuously evaluate stop-loss, target, and trailing-stop for open positions.  
   - When condition is met, backend triggers the corresponding sell order and unsubscribes.

7. **Dashboard Interaction**  
   - Users manage accounts, define strategies, create webhook links, view order logs, download history, and control the “bot” (start/stop).

---

## Detailed Architecture

```mermaid
flowchart TB
  subgraph TradingView
    A[Chart w/ Pine Script] -->|Alert(JSON)| B[Webhook URL]
  end

  subgraph Backend [Django + Celery]
    B --> C[Webhook Receiver View]
    C --> D[Celery Task Queue]
    D --> E[Order Processor Task]
    E --> F{Paper vs Live}
    F -->|Live| G[SmartAPI (Angel One)]
    F -->|Paper| H[Simulated Orders Logger]
    E --> I[Redis Pub/Sub: Price Feed]
    E --> J[Database: Users, Accounts, Orders]
  end

  subgraph Dashboard [React + Vite]
    K[User UI] --> L[REST API (Django DRF)]
    L --> J
    L --> M[Account & Webhook Management]
  end

  subgraph AWS EC2
    Backend & Dashboard hosted behind NGINX + systemd
  end
```

---

## Features & Capabilities

- **Multi-Strategy Support**  
  - Create, enable/disable, and manage multiple Pine Script strategies per account.

- **Custom Webhooks**  
  - Generate unique webhook endpoints tied to an (Account × Strategy) pair.  
  - Configure for each: Stop-loss %, Target %, Trailing-stop %, Paper-trade toggle.

- **Broker Integration**  
  - Onboard Angel One accounts via SmartAPI: Client ID, API Key, Password, TOTP.  
  - Automatic token refresh & secure credential storage.

- **Real-Time Monitoring**  
  - WebSocket subscription to live price feeds; Redis for pub/sub distribution.  
  - Instant evaluation of exit conditions.

- **Dashboard Controls**  
  - Start/Stop Bot: Pause/resume processing of new alerts.  
  - View/download/delete historical trade logs (CSV export).  
  - Role-based access (admin vs. trader).

- **Resilient Async Processing**  
  - Celery workers & Beat scheduler for webhook handling, order placement, credential refresh, and symbol sync.  
  - Redis as broker & result backend.

- **AWS Deployment**  
  - EC2 instances running NGINX, Django, Celery, Redis under systemd for auto-restart & health checks.

---

## Tech Stack

| Layer             | Technology                      |
|-------------------|----------------------------------|
| Frontend          | React, Vite, Axios, React Router |
| Backend           | Python 3.10+, Django, DRF, Celery |
| Task Broker       | Redis (broker & pub/sub)         |
| Broker API        | Angel One SmartAPI               |
| WebSockets        | Django Channels / WebSocket lib  |
| Database          | SQLite (dev)                     |
| Hosting           | AWS EC2 (Linux), NGINX, systemd  |
| CI / Deployment   | Git, SSH, AWS CLI                |

---

## Getting Started

### Prerequisites

- Python 3.10+ & pip  
- Node.js 16+ & npm/yarn  
- Redis server  
- AWS EC2 (for production)  
- Git

### Clone & Install

```bash
git clone <private-repo-url> automated-trading-system
cd automated-trading-system
```

#### Backend

```bash
cd backend/
python -m venv venv
source venv/bin/activate    # Linux/macOS
venv\Scripts\activate       # Windows
pip install -r requirements.txt
```

#### Frontend

```bash
cd ../frontend/
npm install    # or yarn
```

### Environment Configuration

#### `backend/.env`

```dotenv
SECRET_KEY=your_django_secret
DEBUG=True
REDIS_URL=redis://localhost:6379/0
TRADING_MODE=paper          # "live" or "paper"
SMARTAPI_CLIENT_ID=...
SMARTAPI_API_KEY=...
SMARTAPI_PASSWORD=...
SMARTAPI_TOTP_SECRET=...
```

#### `frontend/.env`

```dotenv
VITE_BACKEND_URL=http://localhost:8000
```

### Database Setup & Migrations

```bash
cd backend/
python manage.py migrate
# (Optional) create superuser
python manage.py createsuperuser
```

### Running Services Locally

1. **Start Redis**  
   ```bash
   redis-server
   ```

2. **Run Django & Celery**  
   ```bash
   # Django API
   python manage.py runserver

   # Celery worker + beat
   celery -A backend worker --loglevel=info
   celery -A backend beat  --loglevel=info
   ```

3. **Run React Frontend**  
   ```bash
   cd frontend/
   npm run dev    # or yarn dev
   ```

Open your browser at [http://localhost:5173](http://localhost:5173).

---

## Usage Guide

### 1. Strategy Development & Alerts on TradingView

1. Write your strategy in Pine Script (e.g., MACD crossover, RSI thresholds).  
2. Plot buy/sell signals on your chosen timeframe (2 min, 5 min, etc.).  
3. In the TradingView alert dialog:  
   - Set **Condition** to your strategy’s `alert()` calls.  
   - Choose **Webhook URL** → paste the URL generated in Dashboard (see below).  
   - Customize **Message** payload (JSON) with placeholders:
     ```json
     {
       "symbol": "{{ticker}}",
       "strategy": "MyStrategy",
       "side": "{{strategy.order.action}}",
       "price": "{{close}}"
     }
     ```

### 2. Webhook Configuration

In the Dashboard → **Webhooks** → **Create New**:

| Field                  | Description                                                    |
|------------------------|----------------------------------------------------------------|
| **Name**               | Friendly name for this endpoint                                |
| **Account**            | Select your Angel One account                                  |
| **Strategy**           | Select the PricingView strategy to listen for                  |
| **Stop-loss %**        | Percentage from entry to trigger stop-loss                     |
| **Target %**           | Profit target percentage                                       |
| **Trailing Stop %**    | Starting trailing-stop percentage                              |
| **Paper Trade**        | ☐ Enable to simulate without live orders                       |

Copy the generated **Webhook URL** and paste into TradingView.

### 3. Broker Account Onboarding

In Dashboard → **Accounts** → **Add New**:

1. Enter **Username**, **Password**, **First Name**, **Last Name** (for your platform login).
2. Enter **SmartAPI Client ID**, **API Key**, **Password**, **TOTP Secret**.
3. Save and wait for automatic token validation & refresh.

### 4. Creating & Managing Strategies

- **Strategies** page lists all Pine Script strategies you’ve defined.  
- You can **edit** alert message format or **disable** a strategy without deleting it.

### 5. Paper Trading vs. Live Trading

- **Paper Mode**:  
  - No orders sent to Angel One; simulated trades recorded in DB.  
  - Ideal for testing new strategies or for beginners.

- **Live Mode**:  
  - Real orders placed via SmartAPI.  
  - Monitored in real time and visible in your Angel One account.

Toggle per-webhook or globally via `TRADING_MODE` env variable.

### 6. Real-Time Price Monitoring & Order Execution

- When an alert arrives:  
  1. Celery enqueues an **OrderProcessor** task.  
  2. Task subscribes to the symbol’s WebSocket price feed via Redis pub/sub.  
  3. Continuously evaluates exit logic: stop-loss, target, trailing-stop.  
  4. On exit condition, sends **SELL** order and unsubscribes.

### 7. Bot Control (Start / Stop)

- **Stop Bot**:  
  - Completes any in-flight orders, then ignores new alerts until restarted.

- **Start Bot**:  
  - Resumes processing of incoming webhooks and order evaluations.

### 8. Viewing & Managing Trade History

- **Orders** page shows all executed & simulated trades.  
- Download CSV of order history or **Delete** individual records.

---

## Order Lifecycle & Edge-Case Handling

| Stage              | Action                                                         |
|--------------------|----------------------------------------------------------------|
| **Alert Received** | Validate payload JSON, identify strategy & account.           |
| **Order Placement**| Send market/limit order via SmartAPI (live) or simulate (paper). |
| **Subscription**   | Subscribe to WebSocket feed for live price updates.           |
| **Exit Logic**     | Continuously test stop-loss, target, trailing-stop.           |
| **Order Exit**     | On condition match, place SELL order & unsubscribe.           |
| **Failure Handling**| Retry on network/API failures; alert admin on repeated failures. |

---

## Deployment on AWS

1. **EC2 Setup**  
   - Ubuntu server, open ports 80/443 (NGINX), 8000 (if needed).  
   - Install Docker or manage services directly.

2. **Service Configuration**  
   - Create `systemd` units for:
     - `gunicorn` (Django API)
     - `celery-worker` & `celery-beat`
     - `redis-server`
   - Configure **NGINX** as reverse proxy to Gunicorn and static files.

3. **SSL/TLS**  
   - Use Let’s Encrypt certbot for HTTPS.

4. **Scaling**  
   - Attach an AWS Load Balancer and auto-scale EC2 instances as needed.  
   - Use AWS Secrets Manager for production-grade credential storage.

---

## Security & Best Practices

- Never commit API keys or secrets to Git.  
- Restrict webhook endpoints via HMAC validation (TradingView signature).  
- Enforce HTTPS for all endpoints.  
- Monitor Celery queue lengths and Redis memory usage.  
- Regularly rotate API credentials and TOTP secrets.

---

## Testing

- **Backend** (Django + Celery):  
  ```bash
  cd backend/
  pytest
  ```

- **Frontend** (React):  
  ```bash
  cd frontend/
  npm run test
  ```

- **Integration**:  
  - Use Postman or curl to simulate webhook payloads.  
  - Validate end-to-end live/paper trading flows.

---

## Contributing

1. Fork the repo & create a branch (`feature/XYZ`).  
2. Ensure all tests pass & add new tests for new features.  
3. Submit a Pull Request with detailed description & screenshots if UI changes.  
4. Follow existing code style & update this README if needed.

---

## License

Proprietary — developed under the **Softwired Internship**. Redistribution or disclosure of source code is prohibited without express permission.

---
```
