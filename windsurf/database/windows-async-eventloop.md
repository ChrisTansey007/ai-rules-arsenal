---
id: rule.db-windows-async
type: rule
title: Windows Async Event Loop Configuration for Database Operations
tags: [database, windows, async, psycopg, asyncpg, sqlalchemy, python]
summary: Configure Windows event loop policy correctly to prevent ProactorEventLoop errors with async database operations using psycopg, asyncpg, and other async DB libraries.
version: 1
---

# Windows Async Event Loop Configuration

**Critical Rule: Always set WindowsSelectorEventLoopPolicy on Windows before async database operations.**

---

## 🎯 The Problem

Starting with Python 3.8, Windows uses `ProactorEventLoop` as the default event loop, which is **incompatible** with many async database libraries including:
- `psycopg` (PostgreSQL)
- `asyncpg` (PostgreSQL)
- `motor` (MongoDB)
- Some SQLAlchemy async operations

**Result:** Runtime errors, hanging connections, or complete failure.

---

## ⚠️ Symptoms

### Error 1: Event Loop Already Running
```python
RuntimeError: This event loop is already running
```

### Error 2: Not Implemented
```python
NotImplementedError: Datagram protocols are not supported on Windows
```

### Error 3: Event Loop Closed
```python
RuntimeError: Event loop is closed
```

### Error 4: Connection Hangs
```python
# No error, but script hangs indefinitely
# Database connection never completes
```

### Error 5: SSL/TLS Issues
```python
ssl.SSLError: [SSL: WRONG_VERSION_NUMBER] wrong version number
```

---

## ✅ The Solution

### Pattern 1: At Script Startup (Recommended)

**Place at the very top of your main script:**

```python
import sys
import asyncio

# CRITICAL: Set on Windows before any async operations
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

# Now import database libraries
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
import psycopg
```

### Pattern 2: In Module Initialization

**For library/package code:**

```python
# app/db/session.py
import sys
import asyncio

# Set Windows event loop policy before any database imports
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

# Rest of your database setup
```

### Pattern 3: In Database Seed Scripts

```python
# scripts/seed_database.py
import sys
import asyncio

# Windows compatibility - MUST be at top
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from app.db.session import SessionLocal
from app.models import User, Company

async def seed_data():
    async with SessionLocal() as session:
        # Your seeding logic
        pass

if __name__ == "__main__":
    asyncio.run(seed_data())
```

---

## 📝 Complete Examples

### Example 1: FastAPI Application

```python
# main.py
import sys
import asyncio

# CRITICAL: Windows async compatibility
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from fastapi import FastAPI, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.db.session import get_db
from app.api import companies, customers

app = FastAPI()

@app.get("/health")
async def health_check():
    return {"status": "ok"}

@app.get("/api/v1/companies")
async def get_companies(db: AsyncSession = Depends(get_db)):
    result = await db.execute(text("SELECT * FROM companies"))
    return [dict(row._mapping) for row in result]

# Include routers
app.include_router(companies.router, prefix="/api/v1")
app.include_router(customers.router, prefix="/api/v1")
```

### Example 2: Database Migration Script

```python
# alembic/env.py
import sys
import asyncio

# Windows compatibility for async migrations
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from logging.config import fileConfig
from sqlalchemy import engine_from_config
from sqlalchemy import pool
from alembic import context

# Your migration config
config = context.config

def run_migrations_online():
    """Run migrations in 'online' mode."""
    # Migration logic here
    pass

if context.is_offline_mode():
    run_migrations_offline()
else:
    asyncio.run(run_migrations_online())
```

### Example 3: Standalone Database Script

```python
# scripts/create_test_data.py
"""Generate test data for development database."""
import sys
import asyncio

# Windows async fix - MUST be before SQLAlchemy imports
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from sqlalchemy import text
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql+psycopg://user:pass@localhost/testdb"

async def create_test_data():
    engine = create_async_engine(DATABASE_URL)
    async_session = sessionmaker(
        engine, class_=AsyncSession, expire_on_commit=False
    )
    
    async with async_session() as session:
        # Insert test companies
        await session.execute(text("""
            INSERT INTO companies (name, industry)
            VALUES ('Test Corp', 'Technology')
        """))
        await session.commit()
        print("✓ Test data created")

if __name__ == "__main__":
    asyncio.run(create_test_data())
```

### Example 4: pytest Fixtures

```python
# tests/conftest.py
import sys
import asyncio
import pytest

# Windows compatibility for async tests
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

@pytest.fixture
async def db_session():
    """Create async database session for tests."""
    engine = create_async_engine("postgresql+psycopg://...")
    async_session = sessionmaker(engine, class_=AsyncSession)
    
    async with async_session() as session:
        yield session
    
    await engine.dispose()

@pytest.mark.asyncio
async def test_create_company(db_session):
    result = await db_session.execute(text("SELECT 1"))
    assert result.scalar() == 1
```

---

## 🔧 Why This Happens

### Technical Background

**ProactorEventLoop (Windows Default):**
- Based on I/O Completion Ports (IOCP)
- Optimized for network operations
- **Does NOT support** datagram protocols
- **Does NOT support** some socket operations

**SelectorEventLoop:**
- Based on select/poll system calls
- Universal compatibility
- Supports all socket types
- Used by Unix/Linux/macOS

**Database Libraries:**
- Most async DB drivers use low-level socket operations
- These operations require SelectorEventLoop
- Windows ProactorEventLoop cannot handle them
- Result: RuntimeError or NotImplementedError

---

## 🎯 Best Practices

### ✅ Do This

**1. Set at Module Top**
```python
# At the very top of your entry point
import sys
import asyncio

if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

**2. Check Platform First**
```python
# Always check platform - doesn't harm Unix/Linux
if sys.platform == 'win32':
    # Only set on Windows
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

**3. Document Why**
```python
# Windows compatibility: psycopg requires SelectorEventLoop
# ProactorEventLoop (Windows default) doesn't support needed socket ops
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

**4. Set Before Imports**
```python
# ✅ GOOD - Set before importing DB libraries
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from sqlalchemy.ext.asyncio import create_async_engine
```

**5. Test on Windows**
- Always test scripts on Windows if you support it
- CI/CD should include Windows tests
- Document Windows-specific requirements

### ❌ Don't Do This

**1. Setting After Imports**
```python
# ❌ BAD - Too late
from sqlalchemy.ext.asyncio import create_async_engine

if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

**2. Setting in Function**
```python
# ❌ BAD - Should be at module level
async def connect_db():
    if sys.platform == 'win32':
        asyncio.set_event_loop_policy(...)
    # Connection code
```

**3. Hardcoding Without Check**
```python
# ❌ BAD - Breaks Unix/Linux
asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

**4. Ignoring the Error**
```python
# ❌ BAD - Masking the real problem
try:
    await db.execute(query)
except RuntimeError:
    pass  # Ignoring doesn't fix it
```

---

## 🧪 Testing

### Verify It Works

```python
# test_windows_async.py
import sys
import asyncio

if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

from sqlalchemy.ext.asyncio import create_async_engine
from sqlalchemy import text

async def test_connection():
    """Test that async database operations work on Windows."""
    engine = create_async_engine("postgresql+psycopg://...")
    
    async with engine.begin() as conn:
        result = await conn.execute(text("SELECT 1 as test"))
        value = result.scalar()
        assert value == 1
        print("✓ Async database operations working on Windows")
    
    await engine.dispose()

if __name__ == "__main__":
    asyncio.run(test_connection())
```

### Expected Output

**Without Fix:**
```
RuntimeError: This event loop is already running
```

**With Fix:**
```
✓ Async database operations working on Windows
```

---

## 🔄 Alternative Solutions

### Option 1: Use Docker (Recommended for Production)

**Pros:**
- No Windows-specific issues
- Consistent across all platforms
- Production-ready environment

**Cons:**
- Requires Docker setup
- Slightly slower local development

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0"]
```

### Option 2: Use WSL2 (Windows Subsystem for Linux)

**Pros:**
- Native Linux environment on Windows
- No event loop issues
- Better performance

**Cons:**
- Requires WSL2 setup
- Different environment from native Windows

```bash
# In WSL2 terminal
python main.py  # Works without event loop fix
```

### Option 3: Use Synchronous Drivers

**Pros:**
- No async issues
- Simpler code

**Cons:**
- Less performant
- Blocks event loop

```python
# Use psycopg2 instead of psycopg (async)
import psycopg2

# Synchronous - no event loop needed
conn = psycopg2.connect("postgresql://...")
```

### Option 4: Use Different Async Library

**Some libraries handle Windows better:**
```python
# aiomysql handles Windows better than asyncpg
import aiomysql

# May not need event loop policy
async with aiomysql.create_pool(...) as pool:
    async with pool.acquire() as conn:
        await conn.execute("SELECT 1")
```

---

## 🚨 Troubleshooting

### Issue: Still Getting Errors After Setting Policy

**Cause:** Set too late or in wrong place

**Solution:**
```python
# Ensure it's the FIRST thing in your script
import sys
import asyncio

if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

# Everything else comes after
```

### Issue: Works in Script, Fails in Tests

**Cause:** Test framework creates event loop first

**Solution:**
```python
# In conftest.py at the very top
import sys
import asyncio

if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

# Before importing pytest
import pytest
```

### Issue: Performance Degradation

**Cause:** SelectorEventLoop may be slower than ProactorEventLoop

**Solution:**
- For production, use Docker/Linux
- For development, acceptable tradeoff
- Benchmark if critical

### Issue: SSL Errors After Setting Policy

**Cause:** Some SSL operations differ between event loops

**Solution:**
```python
# Add SSL context configuration
import ssl

ssl_context = ssl.create_default_context()
ssl_context.check_hostname = False
ssl_context.verify_mode = ssl.CERT_NONE

# Pass to database connection
engine = create_async_engine(
    DATABASE_URL,
    connect_args={"ssl": ssl_context}
)
```

---

## 📚 Affected Libraries

### Requires This Fix

- ✅ **psycopg** (psycopg3 async)
- ✅ **asyncpg** (PostgreSQL async driver)
- ✅ **motor** (MongoDB async driver)
- ✅ **aiopg** (PostgreSQL async)
- ✅ **asyncmy** (MySQL async)

### Works Without Fix

- ✅ **aiomysql** (better Windows support)
- ✅ **psycopg2** (synchronous, no event loop)
- ✅ **pymongo** (synchronous, no event loop)
- ✅ **sqlite3** (synchronous, no event loop)

---

## 🔗 Related Arsenal Items

**🔄 Workflow:**
- [Local Test Before Deploy](https://github.com/ChrisTansey007/ai-workflows-arsenal/blob/main/windsurf/deployment/local-test-first.md) - Test database scripts locally

**⚙️ Rule:**
- [Raw SQL Fallback](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/database/raw-sql-fallback.md) - API development with raw SQL

**💭 Memory:**
- [FastAPI Memory](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/project-types/fastapi-memory.md) - FastAPI patterns and setup

**🔗 Example:**
- [Database API Development](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/database-api-development) - Complete setup example

---

## ✅ Checklist

Before running async database code on Windows:
- [ ] Event loop policy set at module top
- [ ] Platform check (`sys.platform == 'win32'`) included
- [ ] Set before database library imports
- [ ] Documented in comment why needed
- [ ] Tested on actual Windows machine
- [ ] Included in test configuration
- [ ] CI/CD includes Windows tests (if supporting Windows)

---

## 📖 Further Reading

- [Python asyncio Event Loop](https://docs.python.org/3/library/asyncio-eventloop.html)
- [SQLAlchemy Async](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)
- [psycopg Documentation](https://www.psycopg.org/psycopg3/docs/)
- [Windows Event Loop Issue](https://github.com/python/cpython/issues/87419)

---

**Remember: Set WindowsSelectorEventLoopPolicy on Windows BEFORE any async database operations!** 🪟
