---
id: rule.api-raw-sql-fallback
type: rule
title: Use Raw SQL Queries When ORM Models Unavailable
tags: [api, sql, fastapi, orm, sqlalchemy, rapid-development]
summary: Implement functional API endpoints immediately using raw SQL queries with SQLAlchemy text() when ORM models don't exist yet - prevents blocking on model generation.
version: 1
---

# Raw SQL Fallback When ORM Models Unavailable

**Critical Rule: Don't block API development waiting for ORM models - use raw SQL with text() to ship faster.**

---

## 🎯 The Problem

**Common Scenario:**
1. Database schema exists (tables created)
2. Need API endpoints NOW for frontend development
3. ORM models not generated yet (or complex to create)
4. Team blocked waiting for model generation

**Traditional Approach:**
```python
# ❌ Blocking approach
# 1. Generate ORM models (hours of work)
# 2. Create schemas/validators
# 3. Implement endpoints
# 4. Finally: frontend can start
# Timeline: 4-8 hours
```

**Raw SQL Approach:**
```python
# ✅ Non-blocking approach
# 1. Write raw SQL query (5 minutes)
# 2. Create endpoint (10 minutes)
# 3. Test and deploy (5 minutes)
# Timeline: 20 minutes
```

---

## ✅ The Solution

### Pattern 1: Simple SELECT Query

```python
from fastapi import APIRouter, Depends
from sqlalchemy import text
from sqlalchemy.ext.asyncio import AsyncSession
from typing import List, Dict, Any
from app.api.deps import get_db

router = APIRouter()

@router.get("/companies", response_model=List[Dict[str, Any]])
async def get_companies(
    skip: int = 0,
    limit: int = 100,
    db: AsyncSession = Depends(get_db)
):
    """
    Get all companies.
    
    Using raw SQL until ORM models generated.
    TODO: Migrate to ORM models when available.
    """
    query = text("""
        SELECT 
            id,
            name,
            industry,
            created_at,
            updated_at
        FROM companies
        ORDER BY created_at DESC
        LIMIT :limit OFFSET :skip
    """)
    
    result = await db.execute(query, {"limit": limit, "skip": skip})
    
    # Convert rows to dicts for JSON serialization
    return [dict(row._mapping) for row in result]
```

### Pattern 2: GET by ID with 404 Handling

```python
from fastapi import HTTPException, status

@router.get("/companies/{company_id}", response_model=Dict[str, Any])
async def get_company(
    company_id: int,
    db: AsyncSession = Depends(get_db)
):
    """Get company by ID using raw SQL."""
    query = text("""
        SELECT 
            id,
            name,
            industry,
            employee_count,
            created_at
        FROM companies
        WHERE id = :company_id
    """)
    
    result = await db.execute(query, {"company_id": company_id})
    row = result.first()
    
    if not row:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Company {company_id} not found"
        )
    
    return dict(row._mapping)
```

### Pattern 3: POST (Insert) with RETURNING

```python
from pydantic import BaseModel

class CompanyCreate(BaseModel):
    name: str
    industry: str
    employee_count: int | None = None

@router.post("/companies", response_model=Dict[str, Any], status_code=status.HTTP_201_CREATED)
async def create_company(
    company: CompanyCreate,
    db: AsyncSession = Depends(get_db)
):
    """Create new company using raw SQL."""
    query = text("""
        INSERT INTO companies (name, industry, employee_count, created_at)
        VALUES (:name, :industry, :employee_count, NOW())
        RETURNING id, name, industry, employee_count, created_at
    """)
    
    result = await db.execute(query, {
        "name": company.name,
        "industry": company.industry,
        "employee_count": company.employee_count
    })
    await db.commit()
    
    row = result.first()
    return dict(row._mapping)
```

### Pattern 4: PUT (Update)

```python
class CompanyUpdate(BaseModel):
    name: str | None = None
    industry: str | None = None
    employee_count: int | None = None

@router.put("/companies/{company_id}", response_model=Dict[str, Any])
async def update_company(
    company_id: int,
    company: CompanyUpdate,
    db: AsyncSession = Depends(get_db)
):
    """Update company using raw SQL."""
    # Build dynamic UPDATE query based on provided fields
    updates = []
    params = {"company_id": company_id}
    
    if company.name is not None:
        updates.append("name = :name")
        params["name"] = company.name
    
    if company.industry is not None:
        updates.append("industry = :industry")
        params["industry"] = company.industry
    
    if company.employee_count is not None:
        updates.append("employee_count = :employee_count")
        params["employee_count"] = company.employee_count
    
    if not updates:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="No fields to update"
        )
    
    updates.append("updated_at = NOW()")
    
    query = text(f"""
        UPDATE companies
        SET {", ".join(updates)}
        WHERE id = :company_id
        RETURNING id, name, industry, employee_count, updated_at
    """)
    
    result = await db.execute(query, params)
    await db.commit()
    
    row = result.first()
    if not row:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Company {company_id} not found"
        )
    
    return dict(row._mapping)
```

### Pattern 5: DELETE

```python
@router.delete("/companies/{company_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_company(
    company_id: int,
    db: AsyncSession = Depends(get_db)
):
    """Delete company using raw SQL."""
    query = text("""
        DELETE FROM companies
        WHERE id = :company_id
        RETURNING id
    """)
    
    result = await db.execute(query, {"company_id": company_id})
    await db.commit()
    
    row = result.first()
    if not row:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Company {company_id} not found"
        )
    
    return None
```

---

## 📝 Complete CRUD Example

```python
# app/api/routes/companies.py
"""
Companies API endpoints.

Using raw SQL queries until ORM models are generated.
TODO: Migrate to ORM models for better type safety and relationships.
"""
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy import text
from sqlalchemy.ext.asyncio import AsyncSession
from typing import List, Dict, Any
from pydantic import BaseModel

from app.api.deps import get_db

router = APIRouter(prefix="/companies", tags=["companies"])

# Pydantic schemas
class CompanyBase(BaseModel):
    name: str
    industry: str
    employee_count: int | None = None

class CompanyCreate(CompanyBase):
    pass

class CompanyUpdate(BaseModel):
    name: str | None = None
    industry: str | None = None
    employee_count: int | None = None

# Endpoints
@router.get("", response_model=List[Dict[str, Any]])
async def list_companies(
    skip: int = 0,
    limit: int = 100,
    industry: str | None = None,
    db: AsyncSession = Depends(get_db)
):
    """List all companies with optional filtering."""
    if industry:
        query = text("""
            SELECT id, name, industry, employee_count, created_at
            FROM companies
            WHERE industry = :industry
            ORDER BY created_at DESC
            LIMIT :limit OFFSET :skip
        """)
        params = {"industry": industry, "limit": limit, "skip": skip}
    else:
        query = text("""
            SELECT id, name, industry, employee_count, created_at
            FROM companies
            ORDER BY created_at DESC
            LIMIT :limit OFFSET :skip
        """)
        params = {"limit": limit, "skip": skip}
    
    result = await db.execute(query, params)
    return [dict(row._mapping) for row in result]

@router.get("/{company_id}", response_model=Dict[str, Any])
async def get_company(
    company_id: int,
    db: AsyncSession = Depends(get_db)
):
    """Get company by ID."""
    query = text("""
        SELECT id, name, industry, employee_count, created_at, updated_at
        FROM companies
        WHERE id = :company_id
    """)
    
    result = await db.execute(query, {"company_id": company_id})
    row = result.first()
    
    if not row:
        raise HTTPException(status_code=404, detail="Company not found")
    
    return dict(row._mapping)

@router.post("", response_model=Dict[str, Any], status_code=201)
async def create_company(
    company: CompanyCreate,
    db: AsyncSession = Depends(get_db)
):
    """Create new company."""
    query = text("""
        INSERT INTO companies (name, industry, employee_count, created_at)
        VALUES (:name, :industry, :employee_count, NOW())
        RETURNING id, name, industry, employee_count, created_at
    """)
    
    result = await db.execute(query, company.dict())
    await db.commit()
    
    return dict(result.first()._mapping)

@router.put("/{company_id}", response_model=Dict[str, Any])
async def update_company(
    company_id: int,
    company: CompanyUpdate,
    db: AsyncSession = Depends(get_db)
):
    """Update company."""
    updates = []
    params = {"company_id": company_id}
    
    for field, value in company.dict(exclude_unset=True).items():
        updates.append(f"{field} = :{field}")
        params[field] = value
    
    if not updates:
        raise HTTPException(status_code=400, detail="No fields to update")
    
    updates.append("updated_at = NOW()")
    
    query = text(f"""
        UPDATE companies
        SET {", ".join(updates)}
        WHERE id = :company_id
        RETURNING id, name, industry, employee_count, updated_at
    """)
    
    result = await db.execute(query, params)
    await db.commit()
    
    row = result.first()
    if not row:
        raise HTTPException(status_code=404, detail="Company not found")
    
    return dict(row._mapping)

@router.delete("/{company_id}", status_code=204)
async def delete_company(
    company_id: int,
    db: AsyncSession = Depends(get_db)
):
    """Delete company."""
    query = text("DELETE FROM companies WHERE id = :company_id RETURNING id")
    
    result = await db.execute(query, {"company_id": company_id})
    await db.commit()
    
    if not result.first():
        raise HTTPException(status_code=404, detail="Company not found")
```

---

## 🛡️ SQL Injection Prevention

### ✅ Safe: Named Parameters

```python
# ✅ SAFE - Uses parameterized query
query = text("""
    SELECT * FROM users
    WHERE email = :email AND status = :status
""")
result = await db.execute(query, {"email": user_email, "status": "active"})
```

### ❌ Unsafe: String Formatting

```python
# ❌ DANGEROUS - SQL injection vulnerability
query = text(f"""
    SELECT * FROM users
    WHERE email = '{user_email}'
""")
# If user_email = "'; DROP TABLE users; --"
# Query becomes: SELECT * FROM users WHERE email = ''; DROP TABLE users; --'
```

### ✅ Safe Dynamic Queries

```python
# ✅ SAFE - Whitelist column names
ALLOWED_SORT_FIELDS = {"name", "created_at", "industry"}

def get_companies(sort_by: str = "created_at"):
    # Validate against whitelist
    if sort_by not in ALLOWED_SORT_FIELDS:
        raise ValueError(f"Invalid sort field: {sort_by}")
    
    # Safe to use in query since it's validated
    query = text(f"""
        SELECT * FROM companies
        ORDER BY {sort_by} DESC
    """)
    return await db.execute(query)
```

---

## 🧪 Testing Raw SQL Endpoints

### Unit Tests

```python
# tests/test_companies.py
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_create_company():
    """Test company creation with raw SQL."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/api/v1/companies", json={
            "name": "Test Corp",
            "industry": "Technology",
            "employee_count": 50
        })
        
        assert response.status_code == 201
        data = response.json()
        assert data["name"] == "Test Corp"
        assert data["industry"] == "Technology"
        assert "id" in data

@pytest.mark.asyncio
async def test_get_company_not_found():
    """Test 404 handling."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/api/v1/companies/99999")
        assert response.status_code == 404
```

### Manual Testing

```bash
# Test GET list
curl http://localhost:8000/api/v1/companies | jq

# Test GET by ID
curl http://localhost:8000/api/v1/companies/1 | jq

# Test POST
curl -X POST http://localhost:8000/api/v1/companies \
  -H "Content-Type: application/json" \
  -d '{"name":"Acme Corp","industry":"Manufacturing"}' | jq

# Test PUT
curl -X PUT http://localhost:8000/api/v1/companies/1 \
  -H "Content-Type: application/json" \
  -d '{"employee_count":150}' | jq

# Test DELETE
curl -X DELETE http://localhost:8000/api/v1/companies/1 -v
```

---

## 🔄 Migration Path to ORM

### When to Migrate

**Stay with Raw SQL if:**
- ✅ Simple CRUD operations
- ✅ No complex relationships
- ✅ Small number of tables
- ✅ Performance is critical

**Migrate to ORM when:**
- ✅ Complex table relationships needed
- ✅ Need type safety across codebase
- ✅ Want automatic relationship loading
- ✅ Team prefers ORM approach

### Migration Steps

**Step 1: Generate ORM Models**
```python
# app/models/company.py
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.orm import DeclarativeBase
from datetime import datetime

class Base(DeclarativeBase):
    pass

class Company(Base):
    __tablename__ = "companies"
    
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    industry = Column(String, nullable=False)
    employee_count = Column(Integer)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
```

**Step 2: Create Pydantic Schemas**
```python
# app/schemas/company.py
from pydantic import BaseModel
from datetime import datetime

class CompanyBase(BaseModel):
    name: str
    industry: str
    employee_count: int | None = None

class CompanyCreate(CompanyBase):
    pass

class CompanyInDB(CompanyBase):
    id: int
    created_at: datetime
    updated_at: datetime | None = None
    
    class Config:
        from_attributes = True
```

**Step 3: Migrate Endpoints**
```python
# Before (raw SQL)
@router.get("/companies")
async def get_companies(db: AsyncSession = Depends(get_db)):
    query = text("SELECT * FROM companies")
    result = await db.execute(query)
    return [dict(row._mapping) for row in result]

# After (ORM)
from app.models.company import Company
from app.schemas.company import CompanyInDB

@router.get("/companies", response_model=List[CompanyInDB])
async def get_companies(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Company))
    return result.scalars().all()
```

---

## 💡 Pro Tips

### 1. Document Intent
Always add a comment explaining you're using raw SQL temporarily:
```python
"""
Using raw SQL until ORM models generated.
TODO: Migrate to ORM when available for better type safety.
"""
```

### 2. Use Type Hints
Even with raw SQL, use proper type hints:
```python
async def get_companies() -> List[Dict[str, Any]]:
    # Type hints help IDEs and catch bugs
```

### 3. Validate Input
Use Pydantic for validation even with raw SQL:
```python
class CompanyCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    industry: str = Field(..., min_length=1)
```

### 4. Test Immediately
Don't wait - test raw SQL endpoints immediately:
```bash
curl http://localhost:8000/api/v1/companies
```

### 5. Handle Errors Properly
```python
try:
    result = await db.execute(query, params)
except IntegrityError:
    raise HTTPException(status_code=400, detail="Duplicate entry")
except Exception as e:
    logger.error(f"Database error: {e}")
    raise HTTPException(status_code=500, detail="Database error")
```

### 6. Use Transactions
```python
async with db.begin():
    await db.execute(insert_query)
    await db.execute(update_related_query)
    # Automatically commits or rolls back
```

---

## 🔗 Related Arsenal Items

**🔄 Workflow:**
- [Local Test Before Deploy](https://github.com/ChrisTansey007/ai-workflows-arsenal/blob/main/windsurf/deployment/local-test-first.md) - Test SQL endpoints locally

**⚙️ Rule:**
- [Windows Async Event Loop](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/database/windows-async-eventloop.md) - Fix async DB issues on Windows

**💭 Memory:**
- [FastAPI Memory](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/project-types/fastapi-memory.md) - FastAPI patterns

**📝 Prompt:**
- [Implement CRUD with Raw SQL](https://github.com/ChrisTansey007/prompt-arsenal/blob/main/development/api/implement-crud-raw-sql.md) - Complete CRUD template

**🔗 Example:**
- [Database API Development](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/database-api-development) - Full example

---

## ✅ Checklist

Before using raw SQL in production:
- [ ] Input validated with Pydantic
- [ ] Using parameterized queries (`:param` syntax)
- [ ] Error handling implemented
- [ ] 404 responses for missing records
- [ ] Tested manually with curl/Postman
- [ ] Unit tests added
- [ ] TODO comment for ORM migration
- [ ] Documented in endpoint docstring

---

**Result: Ship functional APIs in 20 minutes instead of waiting hours for ORM models!** ⚡
