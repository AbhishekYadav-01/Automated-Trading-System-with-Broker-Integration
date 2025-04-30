# Automated-Trading-System-with-Broker-Integration

Not permission to upload the complete code because its done under the Softwired Internship.


Ran tool

Ran tool

Ran tool

Read file: options-algotrader-be/README.md

Read file: options-algotrader-be/backend/manage.py

Ran tool

Ran tool

Ran tool

Ran tool

Ran tool

Read file: options-algotrader-fe/options-algotrader-fe/README.md

Read file: options-algotrader-be/backend/backend/settings.py

Ran tool

Read file: options-algotrader-fe/options-algotrader-fe/frontend/package.json

Ran tool

Read file: options-algotrader-be/backend/backend/urls.py

Ran tool

Read file: options-algotrader-be/backend/users/serializers.py

Ran tool

Read file: options-algotrader-fe/options-algotrader-fe/frontend/README.md

Ran tool

Read file: options-algotrader-be/backend/angelhq/models.py

Ran tool

Read file: options-algotrader-fe/options-algotrader-fe/frontend/src/constants.js

Ran tool

Read file: options-algotrader-fe/options-algotrader-fe/frontend/src/pages/Home.jsx




Ran tool
Below is a suggested top-level `README.md` to place at the root of your project. It covers both the Django back-end and the React/Vite front-end in detail. Feel free to adjust any paths, URLs or commands to match your exact setup.

```markdown
# Options AlgoTrader

A full-stack web application to manage, automate and alert on options-trading strategies.  
It consists of:
- A Django REST back-end (with Celery, Redis & SQLite)  
- A React/Vite front-end (with JWT auth, TradingView webhook management, account & order UIs)

---

## Table of Contents

- [Features](#features)  
- [Architecture](#architecture)  
- [Prerequisites](#prerequisites)  
- [Installation](#installation)  
- [Configuration](#configuration)  
- [Running the Application](#running-the-application)  
- [API Endpoints](#api-endpoints)  
- [Project Structure](#project-structure)  
- [Testing](#testing)  
- [Contributing](#contributing)  
- [License](#license)  

---

## Features

### Back-end (Django)
- JWT-based authentication (login / refresh tokens)  
- User management (CRUD)  
- Angel Broking SmartAPI account integration  
- Symbol list fetcher & caching  
- Scheduled tasks / alerts powered by Celery + Beat + Redis  
- REST API (Django REST Framework) with built-in browsable API & admin UI

### Front-end (React / Vite)
- JWT login flow (stores tokens in localStorage)  
- Protected routes & navigation  
- Manage users, accounts, orders & alerts via forms & tables  
- Configure TradingView webhooks on the fly  
- Toast notifications & modals for a fluid UX  
- ESLint + Pre-configured Vite setup

---

## Architecture

```
┌───────────────────┐                          ┌───────────────────┐
│   TradingView     │ webhook/event →         │    Front-end      │
│   / Chartink /    │                         │    React + Vite   │
│   Custom Alerts   │                         │                   │
└─────────┬─────────┘                         └─────────┬─────────┘
          │                                            │
          │ HTTP (JSON + JWT)                          │ HTTP (JSON + JWT)
          ▼                                            ▼
┌────────────────────────────────────────────────────────────────────┐
│                             Back-end                              │
│     Django REST + Celery + Redis + SQLite + AngelHQ + Strategy    │
└────────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- **Python** ≥ 3.10  
- **Node.js** ≥ 16 (with npm or yarn)  
- **Redis** (for Celery broker / result backend)  
- Git

---

## Installation

1. **Clone the repo**  
   ```bash
   git clone https://github.com/your-org/options-algotrader.git
   cd options-algotrader
   ```

2. **Back-end setup**  
   ```bash
   cd options-algotrader-be/backend
   python -m venv .venv
   source .venv/bin/activate      # Linux/macOS
   .venv\Scripts\activate         # Windows PowerShell

   pip install --upgrade pip
   pip install -r requirements.txt
   ```

3. **Front-end setup**  
   ```bash
   cd ../../options-algotrader-fe/options-algotrader-fe/frontend
   npm install          # or yarn
   ```

---

## Configuration

1. **Back-end `.env`**  
   At `options-algotrader-be/backend`, create a `.env`:
   ```dotenv
   SECRET_KEY=<your-django-secret-key>
   BACKEND_BASE_URL=http://localhost:8000
   ```
   - `SECRET_KEY`: Django secret  
   - `BACKEND_BASE_URL`: Base URL for front-end to hit  

2. **Front-end `.env`**  
   In `options-algotrader-fe/options-algotrader-fe/frontend`, create `.env`:
   ```dotenv
   VITE_BACKEND_URL=http://localhost:8000
   ```
   Vite will expose this as `import.meta.env.VITE_BACKEND_URL`.

---

## Running the Application

### 1. Start Redis
```bash
redis-server
```

### 2. Migrate & Seed Database
```bash
# from options-algotrader-be/backend
python manage.py migrate
# (Optional) create superuser
python manage.py createsuperuser
```

### 3. Start Celery Worker & Beat
```bash
# from options-algotrader-be/backend
celery -A backend worker --loglevel=info
celery -A backend beat  --loglevel=info
```

### 4. Run Django Server
```bash
python manage.py runserver
```

### 5. Run React Front-end
```bash
# from options-algotrader-fe/options-algotrader-fe/frontend
npm run dev    # or yarn dev
```

Open your browser at [http://localhost:5173](http://localhost:5173) (or the port Vite reports).

---

## API Endpoints

| Path                         | Method | Description                                  |
|------------------------------|--------|----------------------------------------------|
| `/user/token/`               | POST   | Obtain JWT access & refresh tokens           |
| `/user/token/refresh/`       | POST   | Refresh access token                         |
| `/users/`                    | GET/POST | List users / create user                   |
| `/users/<id>/`               | GET/PUT/DELETE | CRUD on a single user                 |
| `/angelhq/api/...`           | *varies* | AngelHQ account & symbol APIs             |
| `/alert/...`                 | *varies* | Strategy alert endpoints                   |

_For full list, check each app’s `urls.py` or visit the browsable API at `/`._

---

## Project Structure

```
/
├─ options-algotrader-be/
│   ├─ backend/                   # Django project
│   │   ├─ settings.py
│   │   ├─ urls.py
│   │   ├─ celery.py
│   │   └─ ...
│   ├─ users/                     # User auth app
│   ├─ angelhq/                   # Angel Broking API integration
│   ├─ strategyalert/             # Strategy alert & tasks
│   └─ manage.py
│
└─ options-algotrader-fe/
    └─ options-algotrader-fe/
        ├─ frontend/              # React + Vite SPA
        │   ├─ src/
        │   │   ├─ components/
        │   │   ├─ pages/
        │   │   └─ api.js
        │   ├─ index.html
        │   ├─ package.json
        │   └─ vite.config.js
        └─ README.md
```

---

## Testing

- **Back-end**  
  ```bash
  cd options-algotrader-be/backend
  pytest   # or python manage.py test
  ```

- **Front-end**  
  Add your preferred JS test runner (e.g. Jest, Vitest) as needed.

---

## Contributing

1. Fork this repository  
2. Create a new branch (`git checkout -b feature/XYZ`)  
3. Commit your changes  
4. Submit a Pull Request  

Please follow existing code style, add tests for new features, and update this README when necessary.

---

## License

This project is released under the [MIT License](LICENSE).  
Feel free to use, modify and distribute.
```
