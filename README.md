# FastAPI Calculator with PostgreSQL


**A comprehensive web application demonstrating RESTful APIs, database integration, and containerization.**

[Features](#features) • [Installation](#installation) • [Database Setup](#database-setup) • [API Documentation](#api-documentation) • [Testing](#testing)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Running the Application](#running-the-application)
- [Database Setup](#database-setup)
- [SQL Operations](#sql-operations)
- [API Documentation](#api-documentation)
---

## Overview

This FastAPI Calculator application demonstrates modern web development practices including:

- **RESTful API Design** with FastAPI
- **PostgreSQL Database Integration** for data persistence
- **Docker Compose** orchestration of multiple services
- **pgAdmin** for database management
- **Comprehensive Testing** (Unit, Integration, E2E)
- **CI/CD Pipeline** with GitHub Actions

The application performs basic arithmetic operations while storing calculation history and user information in a PostgreSQL database.

---

## Features

<table>
  <tr>
    <td><b>Calculator Operations</b><br/>Add, Subtract, Multiply, Divide</td>
    <td><b>Database Persistence</b><br/>PostgreSQL for storing users and calculations</td>
  </tr>
  <tr>
    <td><b>Web Interface</b><br/>Simple HTML/JavaScript frontend</td>
    <td><b>Database Management</b><br/>pgAdmin 4 for visual database access</td>
  </tr>
  <tr>
    <td><b>Containerization</b><br/>Docker & Docker Compose</td>
    <td><b>Auto Documentation</b><br/>Swagger UI & ReDoc</td>
  </tr>
  <tr>
    <td><b>Comprehensive Testing</b><br/>Unit, Integration, E2E tests</td>
    <td><b>Error Handling</b><br/>Validation and meaningful error messages</td>
  </tr>
</table>

---

## Architecture

```
┌─────────────┐      HTTP       ┌──────────────┐      SQL      ┌──────────────┐
│   Browser   │ ◄──────────────► │   FastAPI    │ ◄───────────► │  PostgreSQL  │
│  (Frontend) │   REST API       │   (Backend)  │   Queries     │  (Database)  │
└─────────────┘                  └──────────────┘               └──────────────┘
                                        │                               ▲
                                        │                               │
                                        ▼                               │
                                 ┌──────────────┐                      │
                                 │  Operations  │                      │
                                 │  Functions   │                      │
                                 └──────────────┘                      │
                                                                        │
                                                                 ┌──────┴──────┐
                                                                 │   pgAdmin   │
                                                                 │ (Port 5050) │
                                                                 └─────────────┘
```

### Services:

- **FastAPI Web Server** (Port 8000) - REST API endpoints
- **PostgreSQL Database** (Port 5432) - Data persistence
- **pgAdmin** (Port 5050) - Database administration interface

---

### Verify Installations

```bash
docker --version        # Should show Docker version 20.10+
docker-compose --version  # Should show version 2.0+
git --version          # Should show Git version
```

---

## Installation

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd <repository-directory>
```

### Step 2: Review Docker Compose Configuration

The `docker-compose.yml` file defines three services:

```yaml
services:
  web:      # FastAPI application
  db:       # PostgreSQL database
  pgadmin:  # Database management tool
```

### Step 3: Build and Start Services

```bash
docker-compose up --build
```

This command will:
- Build the FastAPI Docker image
- Pull PostgreSQL and pgAdmin images
- Create a network for the services
- Start all containers

<details>
<summary><b>What happens during startup?</b></summary>

1. PostgreSQL database initializes
2. FastAPI application connects to the database
3. pgAdmin becomes available for database management
4. Health checks ensure all services are ready

</details>

---

## Running the Application

### Access Points

Once all services are running, you can access:

| Service | URL | Credentials |
|---------|-----|-------------|
| **FastAPI Calculator** | http://localhost:8000 | - |
| **Swagger API Docs** | http://localhost:8000/docs | - |
| **ReDoc API Docs** | http://localhost:8000/redoc | - |
| **pgAdmin** | http://localhost:5050 | Email: `admin@example.com`<br/>Password: `admin` |

### Stopping the Application

```bash
# Stop containers (keeps data)
docker-compose down

# Stop and remove volumes (deletes data)
docker-compose down -v
```

---

## Database Setup

### Accessing pgAdmin

1. **Open pgAdmin** at http://localhost:5050
2. **Login** with credentials:
   - Email: `admin@example.com`
   - Password: `admin`

### Connecting to PostgreSQL

<details>
<summary><b>Step-by-Step Connection Guide</b></summary>

1. In pgAdmin, right-click **Servers** → **Register** → **Server**
2. **General Tab**:
   - Name: `FastAPI Database`
3. **Connection Tab**:
   - Host name/address: `db`
   - Port: `5432`
   - Maintenance database: `fastapi_db`
   - Username: `postgres`
   - Password: `postgres`
4. Click **Save**

</details>

### Opening Query Tool

1. Expand **Servers** → **FastAPI Database** → **Databases** → **fastapi_db**
2. Right-click on **fastapi_db** → **Query Tool**

---

## SQL Operations

### A. Create Tables

Create the database schema for users and calculations:

```sql
-- Users table to store user information
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Calculations table to store calculation history
CREATE TABLE calculations (
    id SERIAL PRIMARY KEY,
    operation VARCHAR(20) NOT NULL,
    operand_a FLOAT NOT NULL,
    operand_b FLOAT NOT NULL,
    result FLOAT NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    user_id INTEGER NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

<details>
<summary><b>Understanding the Schema</b></summary>

**Users Table:**
- `id`: Auto-incrementing primary key
- `username`: Unique username (max 50 characters)
- `email`: Unique email address (max 100 characters)
- `created_at`: Automatic timestamp of account creation

**Calculations Table:**
- `id`: Auto-incrementing primary key
- `operation`: Type of calculation (add, subtract, multiply, divide)
- `operand_a`, `operand_b`: The two numbers used in calculation
- `result`: The calculated result
- `timestamp`: When the calculation was performed
- `user_id`: Foreign key linking to the users table

**Relationship:**
- Each calculation belongs to one user
- `ON DELETE CASCADE`: When a user is deleted, their calculations are automatically deleted

</details>

### B. Insert Sample Data

```sql
-- Add sample users
INSERT INTO users (username, email) 
VALUES 
    ('alice', 'alice@example.com'), 
    ('bob', 'bob@example.com');

-- Add sample calculations
INSERT INTO calculations (operation, operand_a, operand_b, result, user_id)
VALUES
    ('add', 2, 3, 5, 1),
    ('divide', 10, 2, 5, 1),
    ('multiply', 4, 5, 20, 2);
```

### C. Query Data

```sql
-- Retrieve all users
SELECT * FROM users;

-- Retrieve all calculations
SELECT * FROM calculations;

-- Join users and calculations to see who performed which calculation
SELECT 
    u.username, 
    c.operation, 
    c.operand_a, 
    c.operand_b, 
    c.result,
    c.timestamp
FROM calculations c
JOIN users u ON c.user_id = u.id
ORDER BY c.timestamp DESC;

-- Get calculation statistics per user
SELECT 
    u.username,
    COUNT(c.id) as total_calculations,
    AVG(c.result) as average_result
FROM users u
LEFT JOIN calculations c ON u.id = c.user_id
GROUP BY u.username;
```

### D. Update Records

```sql
-- Update a calculation result
UPDATE calculations
SET result = 6
WHERE id = 1;

-- Update user email
UPDATE users
SET email = 'newalice@example.com'
WHERE username = 'alice';
```

### E. Delete Records

```sql
-- Delete a specific calculation
DELETE FROM calculations
WHERE id = 2;

-- Delete all calculations for a specific user
DELETE FROM calculations
WHERE user_id = 1;

-- Delete a user (will cascade and delete their calculations)
DELETE FROM users
WHERE username = 'bob';
```

---

## API Documentation

### Available Endpoints

| Method | Endpoint | Description | Request Body | Response |
|--------|----------|-------------|--------------|----------|
| GET | `/` | Home page | - | HTML page |
| POST | `/add` | Add two numbers | `{"a": 10, "b": 5}` | `{"result": 15}` |
| POST | `/subtract` | Subtract b from a | `{"a": 10, "b": 5}` | `{"result": 5}` |
| POST | `/multiply` | Multiply two numbers | `{"a": 10, "b": 5}` | `{"result": 50}` |
| POST | `/divide` | Divide a by b | `{"a": 10, "b": 2}` | `{"result": 5.0}` |

### Testing with cURL

```bash
# Addition
curl -X POST "http://localhost:8000/add" \
  -H "Content-Type: application/json" \
  -d '{"a": 10, "b": 5}'

# Division
curl -X POST "http://localhost:8000/divide" \
  -H "Content-Type: application/json" \
  -d '{"a": 10, "b": 2}'

# Error handling (division by zero)
curl -X POST "http://localhost:8000/divide" \
  -H "Content-Type: application/json" \
  -d '{"a": 10, "b": 0}'
```

### Interactive Documentation

- **Swagger UI**: http://localhost:8000/docs
  - Interactive API testing interface
  - Try out endpoints directly in your browser
  - View request/response schemas

- **ReDoc**: http://localhost:8000/redoc
  - Clean, readable API documentation
  - Better for sharing documentation with others

---

## Testing

### Running Tests

The project includes comprehensive testing:

```bash
# Run all tests
pytest

# Run with coverage report
pytest --cov=app --cov-report=html

# Run specific test types
pytest tests/unit/           # Unit tests only
pytest tests/integration/    # Integration tests only
pytest tests/e2e/           # End-to-end tests only
```

### Test Types

<details>
<summary><b>Unit Tests</b></summary>

Test individual functions in isolation:

```bash
pytest tests/unit/test_calculator.py -v
```

**Coverage:**
- Addition, subtraction, multiplication, division
- Edge cases (negative numbers, floats, zeros)
- Error handling (division by zero)

</details>

<details>
<summary><b>Integration Tests</b></summary>

Test API endpoints:

```bash
pytest tests/integration/test_fastapi_calculator.py -v
```

**Coverage:**
- HTTP request/response handling
- JSON serialization/deserialization
- Status codes (200, 400, 500)
- Error responses

</details>

<details>
<summary><b>End-to-End Tests</b></summary>

Test complete user workflows with Playwright:

```bash
pytest tests/e2e/test_e2e.py -v
```

**Coverage:**
- Browser interaction
- Frontend JavaScript execution
- Complete request-response cycle
- User interface behavior

</details>

---

## Project Structure

```
fastapi-calculator/
├── app/
│   └── operations/
│       └── __init__.py          # Calculator operation functions
├── templates/
│   └── index.html               # Frontend interface
├── tests/
│   ├── unit/
│   │   └── test_calculator.py   # Unit tests
│   ├── integration/
│   │   └── test_fastapi_calculator.py  # API tests
│   ├── e2e/
│   │   └── test_e2e.py          # Browser tests
│   └── conftest.py              # Test configuration
├── main.py                      # FastAPI application
├── Dockerfile                   # Container image definition
├── docker-compose.yml           # Multi-container orchestration
├── requirements.txt             # Python dependencies
├── pytest.ini                   # Test configuration
└── README.md                    # This file
```

---

## Troubleshooting

<details>
<summary><b>Cannot connect to pgAdmin</b></summary>

**Problem**: pgAdmin not accessible at http://localhost:5050

**Solutions**:
```bash
# Check if containers are running
docker-compose ps

# View pgAdmin logs
docker-compose logs pgadmin

# Restart services
docker-compose restart pgadmin

# Check if port 5050 is available
lsof -i :5050  # Mac/Linux
netstat -ano | findstr :5050  # Windows
```
</details>

<details>
<summary><b>PostgreSQL connection refused</b></summary>

**Problem**: Cannot connect to database from pgAdmin

**Solutions**:
1. Verify PostgreSQL is running:
   ```bash
   docker-compose ps db
   ```

2. Check PostgreSQL logs:
   ```bash
   docker-compose logs db
   ```

3. Ensure you're using `db` as the host (not `localhost`)

4. Wait for database health check:
   ```bash
   docker-compose exec db pg_isready -U postgres
   ```
</details>

<details>
<summary><b>Docker build fails</b></summary>

**Solutions**:
```bash
# Clean up Docker resources
docker system prune -a

# Remove volumes
docker-compose down -v

# Rebuild from scratch
docker-compose build --no-cache
docker-compose up
```
</details>

<details>
<summary><b>Port already in use</b></summary>

**Problem**: Port 8000, 5432, or 5050 already in use

**Solutions**:
```bash
# Find process using the port
lsof -i :8000  # Mac/Linux
netstat -ano | findstr :8000  # Windows

# Kill the process or change ports in docker-compose.yml
```
</details>

<details>
<summary><b>Tests failing</b></summary>

**Solutions**:
1. Install test dependencies:
   ```bash
   pip install -r requirements.txt
   playwright install
   ```

2. Ensure services are running:
   ```bash
   docker-compose up -d
   ```

3. Check Python version:
   ```bash
   python --version  # Should be 3.10+
   ```
</details>

---

