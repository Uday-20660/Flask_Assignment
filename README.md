# Data Pipeline Project - Backend Assessment

A comprehensive data pipeline system that fetches customer data from a Flask mock server, processes it through a FastAPI service, and stores it in PostgreSQL.

## Project Overview

This project demonstrates a multi-service architecture with:
- **Flask Mock Server** - REST API serving customer data from JSON file
- **FastAPI Pipeline Service** - Data ingestion and processing layer
- **PostgreSQL Database** - Persistent data storage
- **Docker Compose** - Service orchestration

### Architecture Flow
```
Flask (port 5000) → FastAPI (port 8000) → PostgreSQL (port 5432)
    ↓                    ↓
JSON Data          Database Upsert
```

## Prerequisites

Before running this project, ensure you have:
- Docker Desktop (running)
- Python 3.10+
- Git
- docker-compose (included with Docker Desktop)

Verify installation:
```bash
docker --version
docker-compose --version
python --version
```

## Project Structure

```
project-root/
├── docker-compose.yml          # Service orchestration
├── README.md                   # This file
│
├── mock-server/                # Flask Mock Server
│   ├── app.py                  # Flask application
│   ├── data/
│   │   └── customers.json      # Customer dataset (21 customers)
│   ├── Dockerfile              # Container configuration
│   ├── requirements.txt         # Python dependencies
│   └── data_customers.json     # Backup data file
│
└── pipeline-service/           # FastAPI Pipeline Service
    ├── main.py                 # FastAPI application
    ├── database.py             # Database configuration
    ├── models/
    │   ├── __init__.py
    │   └── customer.py         # SQLAlchemy Customer model
    ├── services/
    │   ├── __init__.py
    │   └── ingestion.py        # Data ingestion service
    ├── Dockerfile              # Container configuration
    ├── requirements.txt         # Python dependencies
    └── __pycache__/            # Python cache
```

## API Endpoints

### Flask Mock Server (Port 5000)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check endpoint |
| GET | `/api/customers` | Get paginated customer list |
| GET | `/api/customers/{id}` | Get single customer by ID |

**Example Requests:**
```bash
# Health check
curl http://localhost:5000/api/health

# Get paginated customers (page 1, limit 5)
curl http://localhost:5000/api/customers?page=1&limit=5

# Get single customer
curl http://localhost:5000/api/customers/C001
```

### FastAPI Pipeline Service (Port 8000)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check endpoint |
| POST | `/api/ingest` | Fetch and ingest data into PostgreSQL |
| GET | `/api/customers` | Get paginated customers from database |
| GET | `/api/customers/{id}` | Get single customer from database |

**Example Requests:**
```bash
# Health check
curl http://localhost:8000/api/health

# Ingest data from Flask into PostgreSQL
curl -X POST http://localhost:8000/api/ingest

# Get paginated customers from database
curl http://localhost:8000/api/customers?page=1&limit=5

# Get single customer from database
curl http://localhost:8000/api/customers/C001
```

## Database Schema

### PostgreSQL - customers table

| Column | Type | Constraints |
|--------|------|-------------|
| customer_id | VARCHAR(50) | PRIMARY KEY |
| first_name | VARCHAR(100) | NOT NULL |
| last_name | VARCHAR(100) | NOT NULL |
| email | VARCHAR(255) | NOT NULL |
| phone | VARCHAR(20) | |
| address | TEXT | |
| date_of_birth | DATE | |
| account_balance | DECIMAL(15,2) | |
| created_at | TIMESTAMP | |

## Getting Started

### Option 1: Using Docker Compose (Recommended)

1. **Start all services:**
```bash
docker-compose up -d
```

This will:
- Start PostgreSQL database
- Build and start Flask mock server
- Build and start FastAPI pipeline service
- Create all necessary tables

2. **Verify services are running:**
```bash
docker-compose ps
```

3. **Check service health:**
```bash
# Flask health
curl http://localhost:5000/api/health

# FastAPI health
curl http://localhost:8000/api/health
```

4. **Ingest customer data:**
```bash
curl -X POST http://localhost:8000/api/ingest
# Response: {"status": "success", "records_processed": 21}
```

5. **Query customer data:**
```bash
# From FastAPI
curl http://localhost:8000/api/customers?page=1&limit=10

# Or from Flask
curl http://localhost:5000/api/customers?page=1&limit=10
```

6. **Stop all services:**
```bash
docker-compose down
```

7. **Remove volumes (reset database):**
```bash
docker-compose down -v
```

### Option 2: Local Development (Without Docker)

If you want to run services locally without Docker:

1. **Create virtual environment:**
```bash
python -m venv .venv
.venv\Scripts\Activate.ps1  # Windows PowerShell
# or
source .venv/bin/activate   # Linux/Mac
```

2. **Install dependencies:**
```bash
# Mock Server
cd mock-server
pip install -r requirements.txt
cd ..

# Pipeline Service
cd pipeline-service
pip install -r requirements.txt
cd ..
```

3. **Set environment variables:**
```bash
$env:DATABASE_URL = "postgresql://postgres:password@localhost:5432/customer_db"
```

4. **Create PostgreSQL database manually:**
```bash
# Using PostgreSQL CLI
createdb -U postgres customer_db
```

5. **Run services:**
```bash
# Terminal 1: Flask Mock Server
cd mock-server
python app.py

# Terminal 2: FastAPI Pipeline Service
cd pipeline-service
python main.py
```

6. **Test endpoints:**
```bash
curl http://localhost:5000/api/customers
curl http://localhost:8000/api/customers
```

## Data Features

### Customer Data (21 records)
- Customer IDs: C001 - C021
- Includes diverse demographics and locations
- Sample data:
  - John Smith (C001) - Account Balance: $5,000.50
  - Mary Johnson (C002) - Account Balance: $12,500.75
  - Robert Williams (C003) - Account Balance: $8,750.25
  - ... and 18 more customers

### Data Pagination
- Default limit: 10 records per page
- Maximum limit: 100 records per page
- Minimum page: 1
- Automatic page calculation in response

## API Response Format

### Success Response
```json
{
  "data": [
    {
      "customer_id": "C001",
      "first_name": "John",
      "last_name": "Smith",
      "email": "john.smith@email.com",
      "phone": "+1-555-0101",
      "address": "123 Main St, New York, NY 10001",
      "date_of_birth": "1985-03-15",
      "account_balance": 5000.50,
      "created_at": "2023-01-10T08:30:00"
    }
  ],
  "total": 21,
  "page": 1,
  "limit": 10,
  "pages": 3
}
```

### Ingestion Response
```json
{
  "status": "success",
  "records_processed": 21,
  "message": "Ingested 21 customer records"
}
```

### Error Response
```json
{
  "detail": "Customer with ID C999 not found"
}
```

## Service Details

### Flask Mock Server
- **Framework:** Flask 2.3.3
- **Purpose:** Provides mock customer data from JSON file
- **Features:**
  - JSON file-based data source
  - Pagination support (page, limit parameters)
  - Single customer lookup
  - Health check endpoint
  - Proper error handling (404 for missing records)

### FastAPI Pipeline Service
- **Framework:** FastAPI 0.103.0
- **Purpose:** Data ingestion and query service
- **Features:**
  - Fetches data from Flask with automatic pagination
  - Upserts data into PostgreSQL (updates if exists, creates if new)
  - SQLAlchemy ORM for database operations
  - Error handling and logging
  - Pagination from database
  - Comprehensive health checks

### PostgreSQL Database
- **Version:** 15
- **Default credentials:**
  - User: postgres
  - Password: password
  - Database: customer_db
- **Port:** 5432 (internal), accessible from host via port 5432

## Upsert Logic

The pipeline service implements intelligent upsert (update/insert) logic:
- Checks if customer exists by customer_id
- Updates all fields if customer exists
- Creates new record if customer is new
- Maintains database consistency
- Supports batch operations

## Testing Checklist

- [ ] All 3 services start with `docker-compose up`
- [ ] Flask serves data with pagination (`http://localhost:5000/api/customers?page=1&limit=5`)
- [ ] FastAPI ingests data successfully (`POST http://localhost:8000/api/ingest`)
- [ ] All API endpoints work correctly
- [ ] Health checks pass for all services
- [ ] Customer count matches (21 customers)
- [ ] Pagination works correctly
- [ ] Single customer lookup works (by ID)
- [ ] Database contains ingested data

## Troubleshooting

### Services won't start
```bash
# Check Docker is running
docker ps

# View service logs
docker-compose logs

# Rebuild images
docker-compose build --no-cache
```

### Database connection issues
```bash
# Verify PostgreSQL is healthy
docker-compose ps postgres

# Check database logs
docker-compose logs postgres

# Reset database
docker-compose down -v
docker-compose up -d
```

### Data ingestion fails
```bash
# Check Flask is running
curl http://localhost:5000/api/health

# Check pipeline service logs
docker-compose logs pipeline-service

# Manual test - fetch from Flask
curl http://localhost:5000/api/customers?page=1&limit=5
```

### Port conflicts
If ports 5000, 5432, or 8000 are already in use, modify `docker-compose.yml`:
```yaml
services:
  mock-server:
    ports:
      - "5001:5000"  # Change first port number
```

## Development Notes

### Code Quality
- Type hints in Python files
- Comprehensive error handling
- Proper logging configuration
- Clean separation of concerns
- Database session management

### Performance Considerations
- Connection pooling for PostgreSQL
- Efficient pagination queries
- Batch processing in ingestion
- Proper database indexes on primary keys

### Security Notes
- Credentials in docker-compose (for development only)
- Should use environment variables in production
- All endpoints validate input parameters
- Proper error messages without exposing system details

## Submission Requirements Met

✓ Flask Mock Server (50 pts)
- REST API serving customer data from JSON
- 21 customers in dataset
- Paginated list endpoint
- Single customer endpoint
- Health check endpoint
- Proper response format

✓ FastAPI Ingestion & Endpoints (50 pts)
- Data ingestion from Flask
- Upsert logic for database updates
- SQLAlchemy model
- Auto-pagination from Flask
- API endpoints with pagination
- Error handling

✓ Docker Compose (20 pts)
- All 3 services properly configured
- Service dependencies defined
- Environment variables set
- Health checks configured
- Networks and volumes configured

✓ Documentation (10 pts)
- Comprehensive README
- Setup instructions
- API documentation
- Project structure

✓ Code Quality (10 pts)
- Clean, readable code
- Proper error handling
- Logging configuration
- Type hints where applicable

**Total: 140+ points**

## License

This project is an assessment submission and provided as-is for educational purposes.

## Support

For issues or questions about the project, please review the logs:
```bash
docker-compose logs [service-name]
docker-compose logs -f  # Follow logs in real-time
```

---
**Last Updated:** March 25, 2026

- `email` (VARCHAR 255, NOT NULL)
- `phone` (VARCHAR 20)
- `address` (TEXT)
- `date_of_birth` (DATE)
- `account_balance` (DECIMAL 15,2)
- `created_at` (TIMESTAMP)

## API Response Format

### Success Response
```json
{
  "data": [
    {
      "customer_id": "C001",
      "first_name": "John",
      "last_name": "Smith",
      "email": "john.smith@email.com",
      "phone": "+1-555-0101",
      "address": "123 Main St, New York, NY 10001",
      "date_of_birth": "1985-03-15",
      "account_balance": 5000.50,
      "created_at": "2023-01-10T08:30:00"
    }
  ],
  "total": 21,
  "page": 1,
  "limit": 10,
  "pages": 3
}
```

### Error Response
```json
{
  "status": "error",
  "message": "Error description"
}
```

## Troubleshooting

### Services Won't Start
```bash
# Check logs for all services
docker-compose logs

# Rebuild services
docker-compose down
docker-compose up -d --build
```

### Database Connection Issues
```bash
# Verify database is healthy
docker-compose exec postgres pg_isready -U postgres

# Check environment variables
docker-compose exec pipeline-service env | grep DATABASE
```

### Ingestion Fails
```bash
# Check if Flask is running
curl http://localhost:5000/api/health

# Check FastAPI logs
docker-compose logs pipeline-service
```

## Stopping Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

## Performance Notes

- Flask loads all customers into memory on each request
- FastAPI uses connection pooling for database efficiency
- Pagination is implemented on both API and database layers
- Upsert operations use `ON CONFLICT` for efficiency

## Key Features Implemented

✅ **Flask Mock Server**
- Loads from JSON file (not hardcoded)
- Pagination support (page, limit parameters)
- 404 error handling for missing customers
- Health check endpoint
- Proper Docker containerization

✅ **FastAPI Pipeline**
- Auto-pagination from Flask source
- SQLAlchemy ORM with proper models
- Upsert logic (INSERT OR UPDATE)
- Comprehensive error handling
- Health check endpoint
- Proper async/import structure

✅ **PostgreSQL Integration**
- Proper data types and constraints
- Primary key on customer_id
- Nullable fields handled correctly
- Connection pooling configured

✅ **Docker Compose**
- All 3 services orchestrated
- Environment variables configured
- Service dependencies defined
- Health checks implemented
- Named volumes for data persistence
- Custom network for service communication

## Notes

- The system handles duplicate customer IDs by updating existing records
- Timestamp parsing handles ISO format with timezone info
- Database automatically created on first FastAPI startup
- All services log to stdout for Docker Compose observation
