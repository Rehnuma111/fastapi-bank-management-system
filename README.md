# Bank Management System

A beginner-friendly **REST API** built with FastAPI, PostgreSQL, MongoDB, Redis, and JWT Auth.

> **Database split:** Users, Accounts, Loans → PostgreSQL | Audit Logs, Support Tickets → MongoDB | Caching → Redis

---

## Project Structure

```
bank-management/
├── .env                          # Environment variables (never commit this)
├── .env.example                  # Template for environment variables
├── .dockerignore                 # Files excluded from Docker image
├── Dockerfile                    # Builds the FastAPI app container
├── docker-compose.yml            # Runs all 4 containers together
├── requirements.txt              # Python dependencies
├── scripts/
│   └── seed_data.py              # Populate DB with dummy users
└── app/
    ├── main.py                   # FastAPI app entry point + router registration
    ├── core/
    │   ├── config.py             # Load all settings from .env
    │   ├── security.py           # JWT creation/verification + bcrypt helpers
    │   └── redis_client.py       # Redis connection + cache helpers
    ├── db/
    │   ├── database.py           # SQLAlchemy engine & session (PostgreSQL)
    │   └── mongo.py              # MongoDB connection (audit logs + tickets)
    ├── models/
    │   └── models.py             # ORM table definitions (PostgreSQL)
    ├── schemas/
    │   └── schemas.py            # Pydantic request/response models
    ├── services/
    │   ├── auth_service.py       # Register, login, token logic
    │   ├── account_service.py    # Deposit, withdraw, history
    │   ├── loan_service.py       # Apply, approve, reject loans
    │   ├── audit_service.py      # Write and read audit logs (MongoDB)
    │   └── ticket_service.py     # Support ticket CRUD (MongoDB)
    └── api/
        └── routes/
            ├── auth.py           # /auth/*
            ├── users.py          # /users/*
            ├── accounts.py       # /accounts/*
            ├── loans.py          # /loans/*
            ├── audit.py          # /audit/*
            └── tickets.py        # /tickets/*
```

---

## Setup Guide

### Option A — Docker (Recommended)

No need to install Python, PostgreSQL, MongoDB, or Redis manually. One command starts everything.

**Prerequisites:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.

```bash
# Step 1: Enter the project folder
cd bank-management

# Step 2: Build and start all 4 containers
docker-compose up --build -d

# Step 3: Check all containers are running
docker ps

# Step 4: Seed the database with test users
docker exec -it bank-app python scripts/seed_data.py

# Step 5: Open Swagger UI
# Visit http://localhost:8000/docs
```

---

### Option B — Manual (Without Docker)

**Prerequisites:** Python 3.10+, PostgreSQL, MongoDB, and Redis installed and running.

```bash
# Step 1: Enter the project folder
cd bank-management

# Step 2: Create and activate a virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (Mac/Linux)
source venv/bin/activate

# Step 3: Install dependencies
pip install -r requirements.txt

# Step 4: Create the PostgreSQL database
psql -U postgres
CREATE DATABASE "bankDb";
\q

# Step 5: Configure environment variables
# Copy .env.example to .env and fill in your values
cp .env.example .env
```

Edit `.env` with your values:

```env
# PostgreSQL
DATABASE_URL=postgresql://postgres:YOUR_PASSWORD@localhost:5432/bankDb

# MongoDB (Audit Logs + Support Tickets)
MONGO_URL=mongodb://localhost:27017

# Redis (Caching)
REDIS_URL=redis://localhost:6379/0

# JWT
SECRET_KEY=change-this-to-a-long-random-string
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7

# App
APP_NAME=Bank Management System
DEBUG=True
```

```bash
# Step 6: Seed the database with test users
python scripts/seed_data.py

# Step 7: Start the server
uvicorn app.main:app --reload

# Step 8: Open Swagger UI
# Visit http://127.0.0.1:8000/docs
```

---

## Test Credentials (after seeding)

| Role  | Email             | Password  |
|-------|-------------------|-----------|
| Admin | admin@bank.com    | admin123  |
| User  | alice@example.com | alice123  |
| User  | bob@example.com   | bob123    |

---

## How to Authenticate in Swagger

1. Call `POST /auth/login` with email and password
2. Copy the `access_token` from the response
3. Click **Authorize** (top-right in Swagger)
4. Enter: `Bearer <paste_token_here>`
5. All protected routes are now available

---

## API Endpoints Reference

### Auth

| Method | Endpoint       | Auth | Description              |
|--------|----------------|------|--------------------------|
| POST   | /auth/register | No   | Register a new user      |
| POST   | /auth/login    | No   | Login and get JWT tokens |
| POST   | /auth/refresh  | No   | Refresh access token     |

### Users

| Method | Endpoint    | Auth       | Description       |
|--------|-------------|------------|-------------------|
| GET    | /users/me   | User       | Get my profile    |
| GET    | /users/     | Admin only | List all users    |
| GET    | /users/{id} | Admin only | Get user by ID    |

### Accounts

| Method | Endpoint                    | Auth       | Description           |
|--------|-----------------------------|------------|-----------------------|
| POST   | /accounts/                  | User       | Create a bank account |
| GET    | /accounts/my                | User       | View my accounts      |
| POST   | /accounts/{id}/deposit      | User       | Deposit money         |
| POST   | /accounts/{id}/withdraw     | User       | Withdraw money        |
| GET    | /accounts/{id}/transactions | User       | Transaction history   |
| GET    | /accounts/                  | Admin only | List all accounts     |

### Loans

| Method | Endpoint          | Auth       | Description               |
|--------|-------------------|------------|---------------------------|
| POST   | /loans/           | User       | Apply for a loan          |
| GET    | /loans/my         | User       | View my loan applications |
| GET    | /loans/           | Admin only | List all loans            |
| GET    | /loans/pending    | Admin only | List pending loans        |
| PATCH  | /loans/{id}/status| Admin only | Approve or reject a loan  |

### Audit Logs (MongoDB)

| Method | Endpoint              | Auth       | Description                         |
|--------|-----------------------|------------|-------------------------------------|
| GET    | /audit/my             | User       | My activity history (last 50 logs)  |
| GET    | /audit/               | Admin only | All logs (filter by action, user)   |
| GET    | /audit/action/{action}| Admin only | Logs filtered by action type        |

Available action types: `REGISTER`, `LOGIN`, `TOKEN_REFRESH`, `ACCOUNT_CREATED`, `DEPOSIT`, `WITHDRAWAL`, `LOAN_APPLIED`, `LOAN_STATUS_UPDATED`, `TICKET_CREATED`, `USER_DELETED`

### Support Tickets (MongoDB)

| Method | Endpoint              | Auth       | Description                       |
|--------|-----------------------|------------|-----------------------------------|
| POST   | /tickets/             | User       | Open a new support ticket         |
| GET    | /tickets/my           | User       | View my tickets                   |
| GET    | /tickets/{id}         | User/Admin | View full ticket thread           |
| POST   | /tickets/{id}/reply   | User/Admin | Reply to a ticket                 |
| GET    | /tickets/             | Admin only | List all tickets (filter by status)|
| PATCH  | /tickets/{id}/status  | Admin only | Update status (open/in_progress/closed) |
| DELETE | /tickets/{id}         | Admin only | Delete a ticket permanently       |

---

## Database Schema

### PostgreSQL Tables

```
users
  id, name, email, password (bcrypt hash), role (admin|user), created_at

accounts
  id, user_id → users.id, balance, created_at

transactions
  id, account_id → accounts.id, amount, type (deposit|withdrawal), note, created_at

loans
  id, user_id → users.id, amount, reason, status (pending|approved|rejected), created_at, updated_at
```

### MongoDB Collections

```
audit_logs
  _id, user_id, user_email, action, resource, detail, ip_address, created_at
  Indexes: user_id, action, created_at (descending)

support_tickets
  _id, user_id, user_email, subject, messages[], status (open|in_progress|closed), created_at, updated_at
  Indexes: user_id, status, created_at (descending)
```

---

## Key Concepts

| Concept | Where | What it does |
|---|---|---|
| **bcrypt** | `core/security.py` | Hashes passwords so they are never stored as plain text |
| **JWT** | `core/security.py` | Creates signed tokens that prove who you are |
| **Depends()** | All routes | FastAPI's way of injecting shared logic (auth, DB session) |
| **Pydantic v2** | `schemas/schemas.py` | Validates all incoming request data and shapes responses |
| **SQLAlchemy ORM** | `models/models.py` | Lets you work with PostgreSQL rows as Python objects |
| **RBAC** | `auth_service.py` | `require_admin` dependency blocks non-admin access |
| **MongoDB** | `db/mongo.py` | Stores unstructured data — audit logs and support tickets |
| **Redis** | `core/redis_client.py` | Caches frequent DB queries; gracefully disabled if unavailable |
| **Audit Logging** | `services/audit_service.py` | Every important action is logged with user, IP, and timestamp |

---

## Docker Reference

### Containers

| Container | Image | Port | Purpose |
|---|---|---|---|
| `bank-app` | Built from Dockerfile | 8000 | FastAPI application |
| `bank-db` | postgres:15-alpine | 5432 | Primary database |
| `bank-mongo` | mongo:7-jammy | 27017 | Audit logs + support tickets |
| `bank-redis` | redis:alpine | 6379 | Cache |

### Common Docker Commands

```bash
# Start everything
docker-compose up --build -d

# View running containers
docker ps

# View app logs (live)
docker-compose logs -f app

# Run seed script inside container
docker exec -it bank-app python scripts/seed_data.py

# Open a shell inside the app container
docker exec -it bank-app bash

# Connect to PostgreSQL directly
docker exec -it bank-db psql -U postgres -d bankDb

# Stop all containers (data is preserved)
docker-compose stop

# Stop and remove containers (data is preserved in volumes)
docker-compose down

# Stop and remove everything including database data
docker-compose down -v
```

---

## Running in Production

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

> In production: set a strong `SECRET_KEY`, set `DEBUG=False`, restrict CORS origins, and move all secrets to environment variables — never hardcode them.

---

## Health Check

```
GET /
```

Returns:
```json
{
  "status": "running",
  "app": "Bank Management System",
  "version": "1.0.0",
  "docs": "/docs"
}
```

---

## Additional Docs

- Swagger UI: http://localhost:8000/docs
- ReDoc UI: http://localhost:8000/redoc
