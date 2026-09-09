**English** | [Русский](README.ru.md)

# Finance Helper

**Finance Helper** is a microservices-based application for tracking and analyzing personal finances through a Telegram bot and Mini App.

Developed as a graduation thesis project, it includes several FastAPI services, PostgreSQL, migrations, an internal API, analytics, shared budgets, bank statement imports, data exports, and automated tests.

## Skills demonstrated

- designing a backend system composed of multiple services;
- building REST APIs with FastAPI;
- working with PostgreSQL through SQLAlchemy and Alembic;
- developing a Telegram bot with aiogram;
- routing requests through an API Gateway;
- authenticating internal service requests with an API key;
- managing personal and shared budgets;
- generating analytical reports and data exports;
- providing a Mini App as an additional user interface;
- configuring services through environment variables;
- creating automated and manual test scenarios.

---

## Features

Users can:

- track income and expenses;
- add transactions through text messages, commands, and menu buttons;
- specify transaction dates, including past dates;
- view, edit, and delete transactions;
- create custom income and expense categories;
- assign keywords for automatic category selection;
- set spending limits and budgets;
- receive alerts when limits are reached;
- generate reports for a selected period;
- receive a daily financial summary;
- analyze spending patterns;
- use shared budgets and workspaces;
- export data to CSV and XLSX;
- view advanced analytics in the Mini App;
- import bank statements.

---

## Interface

### Quick transaction entry

Finance Helper understands transactions entered as ordinary messages, including the amount, category, and relative date.

![Finance Helper — quick transaction entry](docs/assets/finance-helper-bot-quick-entry.png)

### Step-by-step expense entry

Transactions can also be added through an interactive flow with category, comment, and date selection.

![Finance Helper — expense entry](docs/assets/finance-helper-bot-expense-flow.png)

### Financial reports

The bot generates monthly reports showing expenses, income, balance, and category breakdowns.

![Finance Helper — financial report](docs/assets/finance-helper-bot-monthly-report.png)

### Mini App

The Mini App provides an extended dashboard with balance, expenses, income, a forecast, and a spending breakdown by category.

![Finance Helper Mini App — dashboard](docs/assets/finance-helper-miniapp-dashboard.png)

### AI spending analysis

The analytics module identifies key changes, the largest categories and transactions, and generates recommendations based on financial data.

![Finance Helper Mini App — AI analysis](docs/assets/finance-helper-miniapp-ai-analysis.png)

### Transaction history

The Mini App provides income and expense history with categories, dates, participants, and comments.

![Finance Helper Mini App — transactions](docs/assets/finance-helper-miniapp-operations.png)

---

## Architecture

```text
Telegram Bot                 Mini App
     │                           │
     └────────────┬──────────────┘
                  ▼
             API Gateway
              /       \
             /         \
            ▼           ▼
   Finance Service   Analytics Service
          ▲   │             │
          │   │             │ HTTP
          │   ▼             │
          │ PostgreSQL      │
          └─────────────────┘
```

`finance-service` is the source of financial data and uses PostgreSQL. `analytics-service` requests transactions and limits from `finance-service` through an internal HTTP API rather than accessing its database directly.

### `finance-service`

The core data service. Responsible for:

- users;
- financial transactions;
- categories;
- limits;
- workspaces;
- shared budget participants;
- bank statement imports.

### `analytics-service`

The analytics and reporting service:

- requests financial data from `finance-service`;
- generates daily summaries;
- builds reports for a selected period;
- analyzes expenses;
- generates CSV/XLSX exports;
- serves data for the Mini App.

### `api-gateway`

A single entry point for user and internal requests. Routes requests between services and serves the Mini App.

### `bot-service`

The Telegram interface built with aiogram. The bot provides access to Finance Helper's core user workflows.

### Mini App

A web interface for extended financial data and analytics views. In the server configuration, the Mini App is available at the public HTTPS address specified in `MINIAPP_PUBLIC_URL`.

Services communicate over HTTP. Internal requests are protected by a separate `INTERNAL_API_KEY`.

---

## Technology stack

| Layer | Technologies |
| --- | --- |
| Backend | Python 3.11+, FastAPI, Pydantic |
| Telegram | aiogram, Telegram Bot API |
| Database | PostgreSQL, SQLAlchemy |
| Migrations | Alembic |
| API | REST, internal service-to-service HTTP |
| Mini App | HTML, CSS, JavaScript, Telegram Mini App |
| Testing | pytest, manual test scenarios |
| Deployment | cloud server, HTTPS domain, separate Python processes |

The current server configuration does not require Docker or `ngrok`.

---

## Project structure

```text
finance-helper/
├── README.md
└── finance_helper/
    ├── run_and_configuration_guide/
    │   └── Finance_Helper_Guide.pdf
    └── source_files/
        ├── .env.example
        ├── Makefile
        ├── pytest.ini
        ├── requirements.txt
        ├── docs/
        │   └── test_scenarios.md
        ├── scripts/
        │   └── seed_demo.py
        ├── services/
        │   ├── analytics-service/
        │   ├── api-gateway/
        │   ├── bot-service/
        │   └── finance-service/
        └── tests/
```

Working directory:

```text
finance_helper/source_files
```

---

## Environment setup

All `cd finance_helper/source_files/...` paths below are relative to **the repository root**.
Start each section from the root; run services in separate terminals
with the environment activated. For a local demo, follow
“One-command local demo” after installing dependencies and preparing the database.

Change to the working directory and create `.env` from the safe template:

```bash
cd finance_helper/source_files
cp .env.example .env
```

Key variables:

| Variable | Purpose |
| --- | --- |
| `BOT_TOKEN` | Telegram bot token |
| `INTERNAL_API_KEY` | Key for internal requests between services |
| `MINIAPP_SIGNING_SECRET` | Secret for signing Mini App token/data |
| `MINIAPP_PUBLIC_URL` | Public HTTPS address of the Mini App |
| `FINANCE_URL` | finance-service address |
| `ANALYTICS_URL` | analytics-service address |
| `GATEWAY_URL` | API Gateway address |
| PostgreSQL variables | Database connection settings |

> Never commit the real `.env` to GitHub. The repository contains only `.env.example`, with no active secrets.

---

## Installation

```bash
cd finance_helper/source_files
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

On Windows, activate the environment with:

```powershell
.\venv\Scripts\Activate.ps1
```

## Database migrations

After creating the PostgreSQL database, run:

```bash
cd finance_helper/source_files/services/finance-service
alembic upgrade head
```

---

## Running the services

### One-command local demo

After installing dependencies and creating **a separate local PostgreSQL database**,
set its connection parameters in `finance_helper/source_files/.env`.
From `finance_helper/source_files`, with the venv activated, run:

```bash
python scripts/run_demo.py
```

The command applies migrations, starts three services on `127.0.0.1:8100–8102`,
creates a separate demo user, and prints a ready-to-use Mini App link.
The Telegram bot is not required for this demo. Keep the terminal open;
`Ctrl+C` stops the services. The next run generates a new link.
Do not publish the token-bearing link. Data is stored in the selected database;
transactions are added only if the demo user has none yet.

If the ports are occupied: `python scripts/run_demo.py --port 8200`.
To verify and stop automatically: `python scripts/run_demo.py --smoke-test`.
The launcher does not create the PostgreSQL database/role or install dependencies:
complete the setup and installation sections above before the first run.

### Finance Service

```bash
cd finance_helper/source_files/services/finance-service
uvicorn app.main:app --host 0.0.0.0 --port 8001
```

### Analytics Service

```bash
cd finance_helper/source_files/services/analytics-service
uvicorn app.main:app --host 0.0.0.0 --port 8002
```

### API Gateway

```bash
cd finance_helper/source_files/services/api-gateway
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### Telegram Bot

```bash
cd finance_helper/source_files/services/bot-service
python -m app.main
```

For persistent server operation, processes can be configured as `systemd` services or managed with another process manager.

---

## Mini App

Set the public HTTPS URL in `.env`:

```env
MINIAPP_PUBLIC_URL=https://your-domain.example/miniapp/app
```

The `/miniapp/app` and `/miniapp/public/...` routes must be forwarded to `api-gateway`.

---

## Demo seed

To populate the system with demo data:

```bash
cd finance_helper/source_files
python scripts/seed_demo.py
```

The script uses `GATEWAY_URL`, `INTERNAL_API_KEY`, `DEMO_TELEGRAM_ID`, and `DEMO_TELEGRAM_USERNAME`.

---

## Testing

Automated tests are located in:

```text
finance_helper/source_files/tests
```

Start:

```bash
cd finance_helper/source_files
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest -q
```

PowerShell:

```powershell
$env:PYTEST_DISABLE_PLUGIN_AUTOLOAD='1'
python -m pytest -q
```

Some smoke tests check a running `api-gateway`. If the gateway is not running, the corresponding integration checks may be skipped.

Manual scenarios are located in:

```text
finance_helper/source_files/docs/test_scenarios.md
```

---

## Documentation

Current local setup guide: [`docs/local-run.md`](docs/local-run.md).
The PDF describes the server configuration; localhost-specific additions are in this
guide and the README.

- `README.md` — project overview and quick start;
- `finance_helper/run_and_configuration_guide/Finance_Helper_Guide.pdf` — detailed configuration guide;
- `finance_helper/source_files/docs/test_scenarios.md` — manual test scenarios.

---

## Status

The project is complete as the release version of a graduation thesis and prepared for deployment to a cloud server.

The main technical focus areas are **Python backend development, FastAPI, microservices architecture, PostgreSQL, Telegram integrations, and testing**.

## Author

[Nicole Zhurbenko](https://github.com/nikamurkaa)
