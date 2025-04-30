# Automated Trading System with TradingView & Broker Integration

> ⚠️ *Note: Due to internship confidentiality (Softwired), full source code is not publicly shared.*

## Overview

This project is an end-to-end **automated trading system** that bridges **TradingView** with a custom backend to integrate with a broker’s API. It includes real-time strategy execution via webhooks, a dynamic credential system, and a full-stack web dashboard. The entire infrastructure is **deployed on AWS**, with support for **paper trading** for testing.

---

## Table of Contents

- [Key Features](#key-features)  
- [Architecture](#architecture)  
- [Tech Stack](#tech-stack)  
- [Setup Guide](#setup-guide)  
- [Environment Configuration](#environment-configuration)  
- [How It Works](#how-it-works)  
- [Testing](#testing)  
- [Deployment](#deployment)  
- [License](#license)

---

## Key Features

### ✅ Strategy Integration
- Pine Script strategies trigger live/paper trades using TradingView webhook alerts.
- Webhook handler receives JSON payloads and executes orders via broker APIs.

### ✅ Dynamic Credential Handling
- Secure, runtime API token management for each user account.
- Credential refresh and order validation handled via backend jobs.

### ✅ Web Dashboard
- Manage user accounts, orders, credentials, and TradingView alerts.
- Built-in support for monitoring active strategies and system logs.

### ✅ Paper Trading Support
- Toggle between **real** and **simulated** (paper) trading mode for safe testing.
- Separate handling logic ensures isolation of simulated trades.

### ✅ Deployment
- Fully deployed on **AWS EC2** using systemd and NGINX reverse proxy.
- Redis, Celery, and Django work together to ensure scalable and fault-tolerant execution.

---

## Architecture

```
flowchart TD
    A[TradingView Alert] -->|Webhook| B[Backend API]
    B --> C[Order Execution Logic]
    B --> D[Celery Worker (async tasks)]
    C --> E[Broker API (e.g. SmartAPI)]
    D --> E
    F[React Dashboard] --> B
    B --> G[SQLite DB (Users, Orders, Alerts)]
    H[AWS EC2 Instance] --> B
    H --> D
```

---

## Tech Stack

| Layer          | Technology                     |
|----------------|---------------------------------|
| Front-End      | React (Vite), Axios, React Router |
| Back-End       | Django REST Framework, Celery, Redis |
| Broker API     | SmartAPI (Angel One)             |
| Storage        | SQLite (development), Redis (cache/task queue) |
| Hosting        | AWS EC2 (Linux), NGINX           |
| DevOps         | systemd, `.env` secrets, SSH, Git |

---

## Setup Guide

### 1. Clone the Repository
```bash
git clone <private-repo-url>
cd automated-trading-system
```

### 2. Backend Setup (Django)
```bash
cd backend/
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Frontend Setup (React)
```bash
cd ../frontend/
npm install
```

### 4. Start Services
```bash
# Redis
redis-server

# Django
cd ../backend/
python manage.py migrate
python manage.py runserver

# Celery worker
celery -A backend worker --loglevel=info
celery -A backend beat --loglevel=info

# React
cd ../frontend/
npm run dev
```

---

## Environment Configuration

### `.env` for Django (`backend/.env`)
```dotenv
SECRET_KEY=your-django-secret
TRADING_MODE=paper   # or live
BROKER_API_KEY=...
BROKER_CLIENT_ID=...
REDIS_URL=redis://localhost:6379
```

### `.env` for React (`frontend/.env`)
```dotenv
VITE_BACKEND_URL=http://localhost:8000
```

---

## How It Works

1. **Strategy Trigger**:  
   Pine Script on TradingView emits an alert with a JSON payload via webhook.

2. **Webhook Receiver**:  
   Django captures and authenticates the webhook, then pushes the task to Celery.

3. **Order Processing**:  
   Celery fetches credentials, validates symbol/order format, and places order via SmartAPI.

4. **Paper Mode (Optional)**:  
   Orders are simulated and logged instead of sent to the broker.

5. **Dashboard**:  
   Users can create/edit alerts, view order history, switch accounts, and manage sessions.

---

## Testing

### Backend
```bash
cd backend/
pytest
```

### Frontend
```bash
cd frontend/
npm run test
```

---

## Deployment

- Deploy backend and frontend on **AWS EC2 Linux instance**
- Use **NGINX** to reverse-proxy React and Django
- Start backend services using **systemd** for persistent background execution
- Store credentials securely in environment files or AWS Secrets Manager

---

## License

This project is proprietary and developed under the **Softwired Internship**. Redistribution or disclosure of code is restricted.

---
