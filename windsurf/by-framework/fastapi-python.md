---
id: rule.fastapi-python
type: rule
title: FastAPI Python Backend Development Rules
tags: [fastapi, python, backend, api, async]
summary: Development rules and conventions for FastAPI backend projects with async patterns, type hints, and modern Python best practices.
version: 1
---

# FastAPI Python Backend Development Rules

**Activation Mode:** Glob: `app/**/*.py`  
**Last Updated:** 2025-01-18  
**Tested With:** Windsurf Cascade, FastAPI 0.104+, Python 3.10+

---

## Project Context

### Tech Stack
- **Language:** Python 3.11+
- **Framework:** FastAPI 0.104+
- **Database:** PostgreSQL 15 with asyncpg
- **ORM:** SQLAlchemy 2.0+ (async)
- **Validation:** Pydantic 2.0+
- **Testing:** pytest + pytest-asyncio
- **Task Queue:** Celery + Redis
- **Caching:** Redis

### Architecture Philosophy
- Async/await everywhere
- Repository pattern for data access
- Dependency injection
- Type hints on everything
- 80%+ test coverage

---

## Code Standards

### Python Style

<python_style>
```python
# ✅ Always use type hints
from typing import Optional, List
from datetime import datetime

async def get_user(user_id: str) -> Optional[User]:
    """Fetch user by ID."""
    return await db.user.find_unique(where={"id": user_id})

# ✅ Use Pydantic for validation
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8, max_length=100)
    name: str = Field(min_length=1, max_length=200)
    
    model_config = {
        "json_schema_extra": {
            "example": {
                "email": "user@example.com",
                "password": "SecurePass123!",
                "name": "John Doe"
            }
        }
    }

# ✅ Use async/await for I/O operations
async def create_user(data: UserCreate) -> User:
    hashed_password = await hash_password(data.password)
    
    user = await db.user.create({
        "email": data.email,
        "password": hashed_password,
        "name": data.name
    })
    
    return user

# ❌ Don't use sync I/O in async functions
async def bad_function():
    user = db.user.find()  # BAD - blocking call
    return user

# ✅ Use early returns
def validate_age(age: int) -> bool:
    if age < 0:
        return False
    if age > 150:
        return False
    return True

# ❌ Don't nest too deeply
def bad_validation(age: int) -> bool:
    if age >= 0:
        if age <= 150:
            return True
    return False
```
</python_style>

### Formatting & Linting
- **Formatter:** Black (line length: 100)
- **Linter:** Ruff or Flake8
- **Import sorting:** isort
- **Type checking:** mypy (strict mode)

```python
# pyproject.toml
[tool.black]
line-length = 100
target-version = ['py311']

[tool.isort]
profile = "black"
line_length = 100

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
```

---

## Project Structure

```python
app/
├── api/                      # API routes
│   ├── deps.py              # Dependency injection
│   ├── errors.py            # Error handlers
│   └── v1/                  # API version 1
│       ├── endpoints/
│       │   ├── users.py
│       │   ├── auth.py
│       │   └── posts.py
│       └── router.py
│
├── core/                     # Core functionality
│   ├── config.py            # Settings
│   ├── security.py          # Auth utilities
│   └── database.py          # DB connection
│
├── models/                   # SQLAlchemy models
│   ├── base.py
│   ├── user.py
│   └── post.py
│
├── schemas/                  # Pydantic schemas
│   ├── user.py
│   ├── auth.py
│   └── post.py
│
├── services/                 # Business logic
│   ├── user_service.py
│   ├── auth_service.py
│   └── post_service.py
│
├── repositories/             # Data access
│   ├── base.py
│   ├── user_repository.py
│   └── post_repository.py
│
├── tasks/                    # Celery tasks
│   ├── email.py
│   └── notifications.py
│
├── utils/                    # Utilities
│   ├── email.py
│   └── validators.py
│
├── tests/                    # Tests
│   ├── conftest.py
│   ├── test_users.py
│   └── test_auth.py
│
├── main.py                   # Application entry
└── __init__.py
```

---

## FastAPI Patterns

### Application Setup

<app_setup>
```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware
from contextlib import asynccontextmanager

from app.core.config import settings
from app.core.database import engine
from app.api.v1.router import api_router
from app.api.errors import add_exception_handlers

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await engine.connect()
    yield
    # Shutdown
    await engine.dispose()

app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    openapi_url=f"{settings.API_V1_PREFIX}/openapi.json",
    docs_url=f"{settings.API_V1_PREFIX}/docs",
    lifespan=lifespan
)

# Middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
app.add_middleware(GZipMiddleware, minimum_size=1000)

# Exception handlers
add_exception_handlers(app)

# Routes
app.include_router(api_router, prefix=settings.API_V1_PREFIX)

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```
</app_setup>

### Route Patterns

<route_patterns>
```python
# api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.api import deps
from app.schemas.user import UserCreate, UserRead, UserUpdate
from app.services.user_service import UserService

router = APIRouter()

@router.post("/", response_model=UserRead, status_code=status.HTTP_201_CREATED)
async def create_user(
    *,
    user_in: UserCreate,
    db: AsyncSession = Depends(deps.get_db),
    user_service: UserService = Depends(deps.get_user_service)
) -> UserRead:
    """
    Create new user.
    
    Raises:
        HTTPException: 400 if email already exists
    """
    user = await user_service.create(db, user_in)
    return user

@router.get("/{user_id}", response_model=UserRead)
async def get_user(
    user_id: str,
    db: AsyncSession = Depends(deps.get_db),
    user_service: UserService = Depends(deps.get_user_service)
) -> UserRead:
    """Get user by ID."""
    user = await user_service.get(db, user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    return user

@router.get("/", response_model=List[UserRead])
async def list_users(
    skip: int = 0,
    limit: int = 100,
    db: AsyncSession = Depends(deps.get_db),
    user_service: UserService = Depends(deps.get_user_service)
) -> List[UserRead]:
    """List users with pagination."""
    users = await user_service.list(db, skip=skip, limit=limit)
    return users

@router.patch("/{user_id}", response_model=UserRead)
async def update_user(
    user_id: str,
    user_in: UserUpdate,
    db: AsyncSession = Depends(deps.get_db),
    current_user = Depends(deps.get_current_user),
    user_service: UserService = Depends(deps.get_user_service)
) -> UserRead:
    """Update user."""
    # Check permissions
    if current_user.id != user_id and not current_user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough permissions"
        )
    
    user = await user_service.update(db, user_id, user_in)
    return user

@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(
    user_id: str,
    db: AsyncSession = Depends(deps.get_db),
    current_user = Depends(deps.get_current_admin_user),
    user_service: UserService = Depends(deps.get_user_service)
) -> None:
    """Delete user (admin only)."""
    await user_service.delete(db, user_id)
```
</route_patterns>

### Dependency Injection

<dependencies>
```python
# api/deps.py
from typing import AsyncGenerator
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.database import AsyncSessionLocal
from app.core.security import verify_token
from app.models.user import User
from app.repositories.user_repository import UserRepository
from app.services.user_service import UserService

security = HTTPBearer()

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """Database session dependency."""
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: AsyncSession = Depends(get_db)
) -> User:
    """Get current authenticated user."""
    token = credentials.credentials
    
    try:
        payload = verify_token(token)
        user_id = payload.get("sub")
    except Exception:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials"
        )
    
    user_repo = UserRepository(db)
    user = await user_repo.get(user_id)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    
    return user

async def get_current_admin_user(
    current_user: User = Depends(get_current_user)
) -> User:
    """Get current user and verify admin role."""
    if not current_user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough permissions"
        )
    return current_user

def get_user_service(db: AsyncSession = Depends(get_db)) -> UserService:
    """Get user service instance."""
    return UserService(UserRepository(db))
```
</dependencies>

### Service Layer

<service_layer>
```python
# services/user_service.py
from typing import Optional, List
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.security import hash_password
from app.models.user import User
from app.repositories.user_repository import UserRepository
from app.schemas.user import UserCreate, UserUpdate
from app.exceptions import UserAlreadyExistsError

class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
    
    async def create(self, db: AsyncSession, user_in: UserCreate) -> User:
        """Create new user."""
        # Check if user exists
        existing = await self.repository.get_by_email(user_in.email)
        if existing:
            raise UserAlreadyExistsError(f"User with email {user_in.email} already exists")
        
        # Hash password
        hashed_password = await hash_password(user_in.password)
        
        # Create user
        user = User(
            email=user_in.email,
            password=hashed_password,
            name=user_in.name
        )
        
        return await self.repository.create(user)
    
    async def get(self, db: AsyncSession, user_id: str) -> Optional[User]:
        """Get user by ID."""
        return await self.repository.get(user_id)
    
    async def list(
        self, 
        db: AsyncSession, 
        skip: int = 0, 
        limit: int = 100
    ) -> List[User]:
        """List users with pagination."""
        return await self.repository.list(skip=skip, limit=limit)
    
    async def update(
        self, 
        db: AsyncSession, 
        user_id: str, 
        user_in: UserUpdate
    ) -> User:
        """Update user."""
        user = await self.repository.get(user_id)
        if not user:
            raise UserNotFoundError(f"User {user_id} not found")
        
        update_data = user_in.model_dump(exclude_unset=True)
        
        # Hash password if provided
        if "password" in update_data:
            update_data["password"] = await hash_password(update_data["password"])
        
        return await self.repository.update(user, update_data)
    
    async def delete(self, db: AsyncSession, user_id: str) -> None:
        """Delete user."""
        user = await self.repository.get(user_id)
        if not user:
            raise UserNotFoundError(f"User {user_id} not found")
        
        await self.repository.delete(user)
```
</service_layer>

### Repository Pattern

<repository_pattern>
```python
# repositories/base.py
from typing import Generic, TypeVar, Type, Optional, List
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.base import Base

ModelType = TypeVar("ModelType", bound=Base)

class BaseRepository(Generic[ModelType]):
    def __init__(self, model: Type[ModelType], db: AsyncSession):
        self.model = model
        self.db = db
    
    async def create(self, obj: ModelType) -> ModelType:
        """Create new record."""
        self.db.add(obj)
        await self.db.flush()
        await self.db.refresh(obj)
        return obj
    
    async def get(self, id: str) -> Optional[ModelType]:
        """Get record by ID."""
        result = await self.db.execute(
            select(self.model).where(self.model.id == id)
        )
        return result.scalar_one_or_none()
    
    async def list(self, skip: int = 0, limit: int = 100) -> List[ModelType]:
        """List records with pagination."""
        result = await self.db.execute(
            select(self.model).offset(skip).limit(limit)
        )
        return list(result.scalars().all())
    
    async def update(self, obj: ModelType, update_data: dict) -> ModelType:
        """Update record."""
        for field, value in update_data.items():
            setattr(obj, field, value)
        
        await self.db.flush()
        await self.db.refresh(obj)
        return obj
    
    async def delete(self, obj: ModelType) -> None:
        """Delete record."""
        await self.db.delete(obj)
        await self.db.flush()

# repositories/user_repository.py
from typing import Optional
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.user import User
from app.repositories.base import BaseRepository

class UserRepository(BaseRepository[User]):
    def __init__(self, db: AsyncSession):
        super().__init__(User, db)
    
    async def get_by_email(self, email: str) -> Optional[User]:
        """Get user by email."""
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
```
</repository_pattern>

---

## SQLAlchemy Models

<sqlalchemy_models>
```python
# models/base.py
from datetime import datetime
from sqlalchemy import DateTime, String
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy.sql import func
import uuid

class Base(DeclarativeBase):
    pass

class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False
    )

# models/user.py
from sqlalchemy import Boolean, String
from sqlalchemy.orm import Mapped, mapped_column
import uuid

from app.models.base import Base, TimestampMixin

class User(Base, TimestampMixin):
    __tablename__ = "users"
    
    id: Mapped[str] = mapped_column(
        String,
        primary_key=True,
        default=lambda: str(uuid.uuid4())
    )
    email: Mapped[str] = mapped_column(String, unique=True, index=True, nullable=False)
    password: Mapped[str] = mapped_column(String, nullable=False)
    name: Mapped[str] = mapped_column(String, nullable=False)
    is_active: Mapped[bool] = mapped_column(Boolean, default=True, nullable=False)
    is_admin: Mapped[bool] = mapped_column(Boolean, default=False, nullable=False)
    
    def __repr__(self) -> str:
        return f"<User(id={self.id}, email={self.email})>"
```
</sqlalchemy_models>

---

## Pydantic Schemas

<pydantic_schemas>
```python
# schemas/user.py
from typing import Optional
from pydantic import BaseModel, EmailStr, Field, ConfigDict
from datetime import datetime

class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(min_length=1, max_length=200)

class UserCreate(UserBase):
    password: str = Field(min_length=8, max_length=100)

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    name: Optional[str] = Field(None, min_length=1, max_length=200)
    password: Optional[str] = Field(None, min_length=8, max_length=100)

class UserRead(UserBase):
    id: str
    is_active: bool
    is_admin: bool
    created_at: datetime
    updated_at: datetime
    
    model_config = ConfigDict(from_attributes=True)
```
</pydantic_schemas>

---

## Error Handling

<error_handling>
```python
# exceptions.py
class AppException(Exception):
    """Base exception for application errors."""
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)

class NotFoundError(AppException):
    def __init__(self, message: str = "Resource not found"):
        super().__init__(message, status_code=404)

class UserAlreadyExistsError(AppException):
    def __init__(self, message: str = "User already exists"):
        super().__init__(message, status_code=400)

# api/errors.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from sqlalchemy.exc import IntegrityError

from app.exceptions import AppException

def add_exception_handlers(app: FastAPI) -> None:
    @app.exception_handler(AppException)
    async def app_exception_handler(request: Request, exc: AppException):
        return JSONResponse(
            status_code=exc.status_code,
            content={"detail": exc.message}
        )
    
    @app.exception_handler(IntegrityError)
    async def integrity_error_handler(request: Request, exc: IntegrityError):
        return JSONResponse(
            status_code=status.HTTP_400_BAD_REQUEST,
            content={"detail": "Database integrity error"}
        )
    
    @app.exception_handler(Exception)
    async def general_exception_handler(request: Request, exc: Exception):
        # Log the error
        logger.error(f"Unexpected error: {exc}", exc_info=True)
        
        return JSONResponse(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            content={"detail": "Internal server error"}
        )
```
</error_handling>

---

## Testing

<testing>
```python
# tests/conftest.py
import pytest
import pytest_asyncio
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from httpx import AsyncClient

from app.main import app
from app.models.base import Base
from app.api.deps import get_db

TEST_DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/test_db"

@pytest_asyncio.fixture
async def db_session():
    engine = create_async_engine(TEST_DATABASE_URL, echo=True)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    async_session = sessionmaker(
        engine, class_=AsyncSession, expire_on_commit=False
    )
    
    async with async_session() as session:
        yield session
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    
    await engine.dispose()

@pytest_asyncio.fixture
async def client(db_session):
    def override_get_db():
        yield db_session
    
    app.dependency_overrides[get_db] = override_get_db
    
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    
    app.dependency_overrides.clear()

# tests/test_users.py
import pytest

@pytest.mark.asyncio
async def test_create_user(client):
    response = await client.post(
        "/api/v1/users/",
        json={
            "email": "test@example.com",
            "password": "SecurePass123!",
            "name": "Test User"
        }
    )
    
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"
    assert "password" not in data

@pytest.mark.asyncio
async def test_get_user(client, db_session):
    # Create user
    user = User(
        email="test@example.com",
        password="hashed",
        name="Test User"
    )
    db_session.add(user)
    await db_session.commit()
    await db_session.refresh(user)
    
    # Get user
    response = await client.get(f"/api/v1/users/{user.id}")
    
    assert response.status_code == 200
    data = response.json()
    assert data["id"] == user.id
    assert data["email"] == user.email
```
</testing>

---

## Security Best Practices

<security>
- ✅ Use bcrypt for password hashing (cost factor: 12)
- ✅ Implement JWT with short expiration (15 min)
- ✅ Validate all inputs with Pydantic
- ✅ Use parameterized queries (SQLAlchemy does this)
- ✅ Implement rate limiting
- ✅ Enable CORS with whitelist
- ✅ Use environment variables for secrets
- ✅ Never log passwords or tokens
- ✅ Implement HTTPS only in production
- ✅ Use SQL injection prevention (ORM)
</security>

---

## Performance Optimization

<performance>
- ✅ Use async/await for all I/O
- ✅ Implement database connection pooling
- ✅ Add indexes on frequently queried fields
- ✅ Use Redis for caching
- ✅ Implement pagination on list endpoints
- ✅ Use background tasks for heavy operations
- ✅ Enable GZip compression
- ✅ Profile slow endpoints
</performance>

---

## Common Mistakes to Avoid

❌ **Don't use sync code in async functions**
❌ **Don't skip type hints**
❌ **Don't return passwords in responses**
❌ **Don't forget to validate inputs**
❌ **Don't use mutable default arguments**
❌ **Don't nest too deeply (max 3 levels)**
❌ **Don't ignore mypy warnings**
❌ **Don't commit secrets to git**

---

**This rule file ensures Cascade generates clean, production-ready FastAPI code following modern Python best practices!**
