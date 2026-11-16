# Python Implementation Guide

Production-ready patterns for Python 3.12+ projects following modern best practices.

## Table of Contents
1. [Project Structure](#project-structure)
2. [Configuration Management](#configuration-management)
3. [Type Safety](#type-safety)
4. [Database Patterns](#database-patterns)
5. [Testing](#testing)
6. [Error Handling](#error-handling)
7. [Logging](#logging)
8. [Dependencies](#dependencies)

---

## Project Structure

### Standard Layout
```
my-project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── config.py          # Configuration
│       ├── models/             # Data models
│       │   ├── __init__.py
│       │   ├── domain.py       # Pydantic models
│       │   └── database.py     # SQLAlchemy models
│       ├── services/           # Business logic
│       │   ├── __init__.py
│       │   └── processor.py
│       ├── utils/              # Utilities
│       │   ├── __init__.py
│       │   └── helpers.py
│       └── api/                # API layer (if needed)
│           ├── __init__.py
│           └── main.py
├── tests/
│   ├── conftest.py            # Shared fixtures
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── docs/
│   └── decisions/             # Architecture decisions
├── pyproject.toml             # Dependencies & config
├── .env.example               # Config template
├── .gitignore
└── README.md
```

### Key Principles
- Use `src/` layout to avoid import issues
- Keep business logic in `services/`
- Separate domain models from database models
- One module per major feature

---

## Configuration Management

### Using Pydantic Settings

**config.py**
```python
from pydantic_settings import BaseSettings
from pydantic import Field, validator
from functools import lru_cache
import os

class Settings(BaseSettings):
    """Application settings with environment variable support."""

    # Required settings
    database_url: str = Field(..., env="DATABASE_URL")
    api_key: str = Field(..., env="API_KEY")

    # Optional with defaults
    environment: str = Field(default="development", env="ENVIRONMENT")
    log_level: str = Field(default="INFO", env="LOG_LEVEL")
    max_connections: int = Field(default=10, env="MAX_CONNECTIONS")

    # Feature flags
    enable_caching: bool = Field(default=True, env="ENABLE_CACHING")

    @validator("log_level")
    def validate_log_level(cls, v):
        allowed = ["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]
        if v.upper() not in allowed:
            raise ValueError(f"log_level must be one of {allowed}")
        return v.upper()

    @validator("database_url")
    def validate_database_url(cls, v):
        if not v.startswith(("postgresql://", "sqlite://")):
            raise ValueError("database_url must be postgresql:// or sqlite://")
        return v

    class Config:
        env_file = ".env"
        case_sensitive = False
        env_file_encoding = "utf-8"

@lru_cache()
def get_settings() -> Settings:
    """Get cached settings instance."""
    return Settings()

# Usage
settings = get_settings()
```

**.env.example**
```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb

# API Keys (never commit actual keys!)
API_KEY=your-api-key-here

# Application
ENVIRONMENT=development
LOG_LEVEL=INFO
MAX_CONNECTIONS=10

# Features
ENABLE_CACHING=true
```

### Best Practices
- Always use `.env` files for local development
- Validate configuration at startup
- Cache settings with `@lru_cache()`
- Provide `.env.example` without secrets
- Use type hints for all settings
- Fail fast if required config is missing

---

## Type Safety

### Type Hints Everywhere

```python
from typing import List, Dict, Optional, Union
from pathlib import Path

def process_file(
    file_path: Path,
    max_lines: int = 100,
    encoding: str = "utf-8"
) -> Dict[str, int]:
    """
    Process a file and return statistics.

    Args:
        file_path: Path to the file to process
        max_lines: Maximum lines to read
        encoding: File encoding

    Returns:
        Dictionary with 'lines' and 'words' counts
    """
    with file_path.open(encoding=encoding) as f:
        lines = f.readlines()[:max_lines]

    return {
        "lines": len(lines),
        "words": sum(len(line.split()) for line in lines)
    }
```

### Pydantic Models for Validation

```python
from pydantic import BaseModel, Field, validator
from datetime import datetime
from typing import List, Optional

class User(BaseModel):
    """User data model with validation."""

    id: str
    email: str
    name: str = Field(..., min_length=1, max_length=100)
    age: Optional[int] = Field(None, ge=0, le=150)
    tags: List[str] = Field(default_factory=list)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    @validator("email")
    def validate_email(cls, v):
        if "@" not in v:
            raise ValueError("Invalid email address")
        return v.lower()

    @validator("tags")
    def validate_tags(cls, v):
        return [tag.strip().lower() for tag in v]

# Usage
user = User(
    id="123",
    email="user@example.com",
    name="John Doe",
    age=30,
    tags=["python", "developer"]
)

# Validation happens automatically
try:
    invalid_user = User(id="456", email="invalid", name="")
except ValidationError as e:
    print(e.json())
```

### Type Checking with mypy

**pyproject.toml**
```toml
[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
```

Run: `uv run mypy src/`

---

## Database Patterns

### SQLAlchemy with Connection Pooling

**models/database.py**
```python
from sqlalchemy import create_engine, Column, String, Integer, DateTime, Index
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import QueuePool
from datetime import datetime
from contextlib import contextmanager
from config import get_settings

settings = get_settings()

# Engine with connection pooling
engine = create_engine(
    settings.database_url,
    poolclass=QueuePool,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,  # Verify connections before use
    echo=settings.log_level == "DEBUG"
)

SessionLocal = sessionmaker(bind=engine, autoflush=False, autocommit=False)
Base = declarative_base()

@contextmanager
def get_db():
    """Get database session with automatic cleanup."""
    db = SessionLocal()
    try:
        yield db
        db.commit()
    except Exception:
        db.rollback()
        raise
    finally:
        db.close()

# Models
class Contract(Base):
    __tablename__ = "contracts"

    id = Column(String, primary_key=True)
    content = Column(String, nullable=False)
    status = Column(String, default="pending", index=True)
    created_at = Column(DateTime, default=datetime.utcnow, index=True)

    # Composite index for common queries
    __table_args__ = (
        Index("idx_status_created", "status", "created_at"),
    )

# Usage
def get_contract(contract_id: str) -> Optional[Contract]:
    """Get contract by ID."""
    with get_db() as db:
        return db.query(Contract).filter(Contract.id == contract_id).first()

def create_contract(id: str, content: str) -> Contract:
    """Create new contract."""
    with get_db() as db:
        contract = Contract(id=id, content=content)
        db.add(contract)
        db.flush()  # Get ID without committing
        return contract
```

### pgvector Integration

```python
from pgvector.sqlalchemy import Vector
from sqlalchemy import select

class Document(Base):
    __tablename__ = "documents"

    id = Column(String, primary_key=True)
    content = Column(String, nullable=False)
    embedding = Column(Vector(1536))  # OpenAI embedding size

def find_similar_documents(
    embedding: List[float],
    limit: int = 5
) -> List[Document]:
    """Find similar documents using vector search."""
    with get_db() as db:
        return (
            db.query(Document)
            .order_by(Document.embedding.cosine_distance(embedding))
            .limit(limit)
            .all()
        )
```

---

## Testing

### pytest Configuration

**pyproject.toml**
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--strict-markers",
    "--cov=src",
    "--cov-report=term-missing"
]
```

### Fixtures in conftest.py

**tests/conftest.py**
```python
import pytest
from pathlib import Path
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from models.database import Base

@pytest.fixture
def sample_data_dir() -> Path:
    """Path to sample data directory."""
    return Path(__file__).parent / "fixtures"

@pytest.fixture
def sample_contract(sample_data_dir) -> str:
    """Load sample contract text."""
    return (sample_data_dir / "sample_contract.txt").read_text()

@pytest.fixture
def db_session():
    """In-memory database session for testing."""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    session.close()

@pytest.fixture
def mock_api_key(monkeypatch):
    """Mock API key for testing."""
    monkeypatch.setenv("API_KEY", "test-api-key")
```

### Test Organization

**tests/unit/services/test_processor.py**
```python
import pytest
from services.processor import process_contract

class TestProcessContract:
    """Happy path tests."""

    def test_processes_valid_contract(self, sample_contract):
        result = process_contract(sample_contract)
        assert result["status"] == "success"
        assert len(result["sections"]) > 0

    def test_returns_expected_structure(self, sample_contract):
        result = process_contract(sample_contract)
        assert "status" in result
        assert "sections" in result
        assert "metadata" in result

class TestProcessContractEdgeCases:
    """Edge case and error tests."""

    def test_handles_empty_content(self):
        result = process_contract("")
        assert result["status"] == "error"

    def test_handles_unicode(self):
        result = process_contract("Contract with émojis 🎉")
        assert result["status"] == "success"

    def test_handles_very_long_content(self):
        content = "a" * 1_000_000
        result = process_contract(content)
        assert result["status"] == "success"

class TestProcessContractIntegration:
    """Integration tests with database."""

    def test_saves_to_database(self, db_session, sample_contract):
        result = process_contract(sample_contract, save_to_db=True)

        saved = db_session.query(Contract).first()
        assert saved is not None
        assert saved.content == sample_contract
```

### Parametrized Tests

```python
@pytest.mark.parametrize("input,expected", [
    ("hello world", 2),
    ("", 0),
    ("one", 1),
    ("multiple words here", 3),
])
def test_word_count(input, expected):
    assert count_words(input) == expected
```

### Running Tests

```bash
# All tests
uv run pytest

# Specific test file
uv run pytest tests/unit/test_processor.py

# With coverage
uv run pytest --cov=src --cov-report=html

# Fast fail
uv run pytest -x

# Verbose
uv run pytest -vv
```

---

## Error Handling

### Custom Exceptions

```python
class ContractError(Exception):
    """Base exception for contract-related errors."""
    pass

class ValidationError(ContractError):
    """Contract failed validation."""
    pass

class ProcessingError(ContractError):
    """Error during contract processing."""
    pass

class DatabaseError(ContractError):
    """Database operation failed."""
    pass
```

### Error Handling Pattern

```python
from typing import Union
import logging

logger = logging.getLogger(__name__)

def process_contract(content: str) -> Dict[str, Union[str, List]]:
    """
    Process contract with comprehensive error handling.

    Returns result dict with 'status' field.
    """
    try:
        # Validation
        if not content.strip():
            raise ValidationError("Contract content cannot be empty")

        # Processing
        sections = split_into_sections(content)

        if not sections:
            raise ProcessingError("Failed to extract sections")

        return {
            "status": "success",
            "sections": sections
        }

    except ValidationError as e:
        logger.warning(f"Validation failed: {e}")
        return {
            "status": "validation_error",
            "error": str(e)
        }

    except ProcessingError as e:
        logger.error(f"Processing failed: {e}")
        return {
            "status": "processing_error",
            "error": str(e)
        }

    except Exception as e:
        logger.exception(f"Unexpected error: {e}")
        return {
            "status": "error",
            "error": "Internal error occurred"
        }
```

---

## Logging

### Structured Logging Setup

**logging_config.py**
```python
import logging
import sys
from pythonjsonlogger import jsonlogger
from config import get_settings

def setup_logging():
    """Configure structured logging."""
    settings = get_settings()

    # Create logger
    logger = logging.getLogger()
    logger.setLevel(settings.log_level)

    # JSON formatter for production
    if settings.environment == "production":
        handler = logging.StreamHandler(sys.stdout)
        formatter = jsonlogger.JsonFormatter(
            "%(asctime)s %(name)s %(levelname)s %(message)s"
        )
        handler.setFormatter(formatter)
    else:
        # Human-readable for development
        handler = logging.StreamHandler(sys.stdout)
        formatter = logging.Formatter(
            "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        )
        handler.setFormatter(formatter)

    logger.addHandler(handler)
    return logger

# Usage in modules
logger = logging.getLogger(__name__)

def process_data(data: dict):
    logger.info("Processing data", extra={
        "data_size": len(data),
        "data_type": type(data).__name__
    })

    try:
        result = do_work(data)
        logger.info("Processing complete", extra={
            "result_count": len(result)
        })
        return result
    except Exception as e:
        logger.exception("Processing failed", extra={
            "error": str(e),
            "data_keys": list(data.keys())
        })
        raise
```

---

## Dependencies

### Using uv (Modern)

**pyproject.toml**
```toml
[project]
name = "my-project"
version = "0.1.0"
description = "Project description"
requires-python = ">=3.12"
dependencies = [
    "pydantic>=2.0.0",
    "pydantic-settings>=2.0.0",
    "sqlalchemy>=2.0.0",
    "psycopg2-binary>=2.9.0",
    "pgvector>=0.2.0",
    "httpx>=0.25.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "mypy>=1.5.0",
    "ruff>=0.1.0",
]

[project.scripts]
my-cli = "my_project.cli:main"

[tool.ruff]
line-length = 88
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]
ignore = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

### Commands

```bash
# Initialize project
uv init

# Add dependency
uv add pydantic

# Add dev dependency
uv add --dev pytest

# Install all dependencies
uv sync

# Run command
uv run python -m my_project

# Run tests
uv run pytest

# Format code
uv run ruff format .

# Lint code
uv run ruff check .

# Type check
uv run mypy src/
```

---

## Complete Example

**src/my_project/main.py**
```python
"""
Main application entry point.
"""
from pathlib import Path
from typing import Dict, List
import logging
from config import get_settings
from models.database import get_db, Contract
from services.processor import process_contract

logger = logging.getLogger(__name__)
settings = get_settings()

def main() -> None:
    """Run the application."""
    logger.info("Starting application", extra={
        "environment": settings.environment
    })

    try:
        # Load contract
        contract_path = Path("data/contract.txt")
        content = contract_path.read_text()

        # Process
        result = process_contract(content)

        if result["status"] == "success":
            # Save to database
            with get_db() as db:
                contract = Contract(
                    id="contract-001",
                    content=content,
                    status="processed"
                )
                db.add(contract)

            logger.info("Contract processed successfully", extra={
                "contract_id": "contract-001",
                "sections": len(result["sections"])
            })
        else:
            logger.error("Processing failed", extra={
                "error": result.get("error")
            })

    except Exception as e:
        logger.exception("Application error", extra={
            "error": str(e)
        })
        raise

if __name__ == "__main__":
    from logging_config import setup_logging
    setup_logging()
    main()
```

---

## Quick Reference

### Common Patterns

```python
# Configuration
from config import get_settings
settings = get_settings()

# Database session
from models.database import get_db
with get_db() as db:
    result = db.query(Model).all()

# Logging
import logging
logger = logging.getLogger(__name__)
logger.info("message", extra={"key": "value"})

# Validation
from pydantic import BaseModel
class Data(BaseModel):
    field: str

# Error handling
try:
    result = risky_operation()
except CustomError as e:
    logger.error(f"Failed: {e}")
    return {"status": "error", "error": str(e)}
```

### Best Practices Checklist

- [ ] Type hints on all functions
- [ ] Pydantic models for data validation
- [ ] Configuration via environment variables
- [ ] Structured logging with context
- [ ] Custom exceptions for domain errors
- [ ] Database connection pooling
- [ ] Comprehensive test coverage
- [ ] Error handling at boundaries
- [ ] Documentation in docstrings
- [ ] Conventional commit messages

---

## Related Resources

- [Pydantic Documentation](https://docs.pydantic.dev/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [pytest Documentation](https://docs.pytest.org/)
- [Python Type Hints (PEP 484)](https://www.python.org/dev/peps/pep-0484/)
