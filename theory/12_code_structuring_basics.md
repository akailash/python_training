**Structuring a FastAPI Application with PostgreSQL, SQLAlchemy, Alembic, Unit Tests, and Pre-commit Integration**

In this tutorial, we'll set up a FastAPI application with PostgreSQL integration using SQLAlchemy and Alembic, write unit tests, and integrate pre-commit to automatically check code quality at git commit. This setup ensures code quality, reliability, and maintainability in your FastAPI projects.

### 1. Setting up the Environment:

Ensure you have Python, Docker, and Git installed on your system.

### 2. Creating the FastAPI Application:

Create a directory for your FastAPI application:

```bash
mkdir fastapi_app
cd fastapi_app
```

### 3. Setting up Poetry:

Install Poetry if you haven't already:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

Initialize a new Poetry project:

```bash
poetry init
```

### 4. Installing Dependencies:

Install the required dependencies:

```bash
poetry add fastapi uvicorn sqlalchemy psycopg2-binary alembic
poetry add --dev pytest pytest-asyncio requests pre-commit black
```

### 5. Project Structure:

Create the following directory structure for your FastAPI application:

```
fastapi_app/
├── app/
│   ├── __init__.py
│   ├── db.py
│   ├── models.py
│   ├── main.py
│   ├── crud.py
│   ├── schemas.py
│   ├── routes.py
├── migrations/
│   ├── versions/
├── tests/
│   ├── __init__.py
│   ├── test_routes.py
├── .gitignore
├── .pre-commit-config.yaml
├── alembic.ini
├── Dockerfile
├── docker-compose.yml
├── Makefile
├── pyproject.toml
└── README.md
```

### 6. Writing FastAPI Application Code:

**app/main.py:**

```python
from fastapi import FastAPI
from app.routes import router
from app.db import engine, Base

app = FastAPI()

Base.metadata.create_all(bind=engine)

app.include_router(router)
```

**app/db.py:**

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql://user:password@localhost/dbname"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

**app/models.py:**

```python
from sqlalchemy import Column, Integer, String
from app.db import Base

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
```

**app/schemas.py:**

```python
from pydantic import BaseModel

class UserBase(BaseModel):
    name: str

class UserCreate(UserBase):
    pass

class User(UserBase):
    id: int

    class Config:
        orm_mode = True
```

**app/crud.py:**

```python
from sqlalchemy.orm import Session
from app.models import User
from app.schemas import UserCreate

def get_user(db: Session, user_id: int):
    return db.query(User).filter(User.id == user_id).first()

def create_user(db: Session, user: UserCreate):
    db_user = User(name=user.name)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

**app/routes.py:**

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.db import SessionLocal
from app import crud, schemas

router = APIRouter()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@router.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    return crud.create_user(db=db, user=user)
```

### 7. Writing Unit Tests:

Write unit tests for your FastAPI application in the `tests` directory.

**tests/test_routes.py:**

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_user():
    response = client.post("/users/", json={"name": "John Doe"})
    assert response.status_code == 200
    data = response.json()
    assert data["name"] == "John Doe"
    assert "id" in data
```

### 8. Configuring Alembic:

Initialize Alembic for database migrations:

```bash
poetry run alembic init migrations
```

Edit `alembic.ini` to use the correct database URL:

```ini
# line 58
sqlalchemy.url = postgresql://user:password@localhost/dbname
```

Edit `migrations/env.py` to include your models:

```python
from app.db import Base
from app.models import User

target_metadata = Base.metadata
```

Create your first migration:

```bash
poetry run alembic revision --autogenerate -m "Initial migration"
poetry run alembic upgrade head
```

### 9. Configuring Pre-commit:

Create a `.pre-commit-config.yaml` file in the root directory:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-xml
      - id: check-yaml
      - id: check-toml
  - repo: https://github.com/psf/black
    rev: 22.3.0
    hooks:
      - id: black
```

Install pre-commit hooks:

```bash
poetry run pre-commit install
```

### 10. Running the FastAPI Application:

Run the FastAPI application using Uvicorn:

```bash
poetry run uvicorn app.main:app --reload
```

### 11. Running Unit Tests:

Run unit tests using Pytest:

```bash
poetry run pytest
```

### 12. Committing Code with Pre-commit Checks:

When you commit code, pre-commit hooks will automatically run to check for linting issues:

```bash
git add .
git commit -m "Commit message"
```

### 13. Adding Docker Support:

**Dockerfile:**

```Dockerfile
# Use an official Python runtime as a base image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the dependency files into the container
COPY pyproject.toml poetry.lock ./

# Install Poetry and dependencies
RUN pip install poetry && poetry install --no-root --no-dev

# Copy the application code into the container
COPY . .

# Expose port 8000 to allow communication to/from the FastAPI application
EXPOSE 8000

# Command to run the FastAPI application when the container starts
CMD ["poetry", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
  db:
    image: postgres
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: dbname
    ports:
      - "5432:5432"
```

### 14. Writing Makefile:

**Makefile:**

```Makefile
.PHONY: install test lint format run docker-build docker-up

install:
    poetry install

test:
    poetry run pytest --cov=app

lint:
    poetry run flake8

format:
    poetry run black .

run:
    poetry run uvicorn app.main:app --reload

docker-build:
    docker-compose build

docker-up:
    docker-compose up
```

### Conclusion:

In this tutorial, we structured a FastAPI application with PostgreSQL integration using SQLAlchemy and Alembic, wrote unit tests, and integrated pre-commit to automatically check code quality at git commit. We also set up Docker and Docker Compose for containerization and used a Makefile to streamline common tasks. This setup ensures code quality, reliability, and maintainability in your FastAPI projects.