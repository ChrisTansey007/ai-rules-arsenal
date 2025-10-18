# Backend API Development - GitHub Copilot Instructions

**Project Type:** RESTful API Backend  
**Last Updated:** 2025-01-18

---

## Project Overview

This project is a production-ready RESTful API backend. Follow these instructions for consistent, high-quality code generation.

---

## Tech Stack

### Core
- **Runtime:** Node.js 20 LTS or Python 3.11+
- **Framework:** Express.js 4.18+ (Node) or FastAPI 0.104+ (Python)
- **Database:** PostgreSQL 15 with connection pooling
- **Authentication:** JWT with refresh tokens
- **Documentation:** OpenAPI/Swagger

### Supporting Libraries
- **Node.js:**
  - Validation: Zod
  - ORM: Prisma
  - Testing: Jest + Supertest
  
- **Python:**
  - Validation: Pydantic
  - ORM: SQLAlchemy 2.0+ (async)
  - Testing: pytest + pytest-asyncio

---

## Architecture

### Layered Architecture

```
src/
├── api/           # HTTP layer (routes, controllers)
├── services/      # Business logic
├── repositories/  # Data access
├── models/        # Database models
├── schemas/       # Validation schemas
├── middleware/    # Express/FastAPI middleware
└── utils/         # Utilities and helpers
```

### Request Flow

```
Request → Middleware → Controller → Service → Repository → Database
                ↓
         Validation
         Authentication
         Authorization
```

---

## Coding Standards

### General Principles

1. **Type Safety:** Use TypeScript (Node) or type hints (Python) everywhere
2. **Validation:** Validate all inputs at API boundaries
3. **Error Handling:** Consistent error responses across all endpoints
4. **Documentation:** JSDoc (Node) or docstrings (Python) for all public functions
5. **Testing:** Minimum 80% coverage, all endpoints tested

### Code Style

**Node.js/TypeScript:**
```typescript
// ✅ Use async/await
async function getUser(userId: string): Promise<User> {
  return await userRepository.findById(userId);
}

// ✅ Proper error handling
try {
  const user = await userService.create(data);
  return res.status(201).json({ success: true, data: user });
} catch (error) {
  next(error); // Let error middleware handle it
}

// ✅ Type all parameters and returns
interface CreateUserData {
  email: string;
  password: string;
  name: string;
}
```

**Python:**
```python
# ✅ Use async/await
async def get_user(user_id: str) -> Optional[User]:
    return await user_repository.find_by_id(user_id)

# ✅ Proper error handling
try:
    user = await user_service.create(data)
    return {"success": True, "data": user}
except UserExistsError as e:
    raise HTTPException(status_code=400, detail=str(e))

# ✅ Type hints everywhere
from typing import Optional, List

async def list_users(skip: int = 0, limit: int = 100) -> List[User]:
    return await user_repository.list(skip=skip, limit=limit)
```

---

## API Design

### RESTful Endpoints

```
POST   /api/v1/users           # Create
GET    /api/v1/users/:id       # Read one
GET    /api/v1/users           # Read many (with pagination)
PATCH  /api/v1/users/:id       # Update
DELETE /api/v1/users/:id       # Delete
```

### Response Format

**Success (200, 201):**
```json
{
  "success": true,
  "data": {
    "id": "123",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

**Success with Pagination (200):**
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "pages": 5
  }
}
```

**Error (400, 404, 500):**
```json
{
  "success": false,
  "error": {
    "message": "User not found",
    "code": "USER_NOT_FOUND",
    "details": {}
  }
}
```

### Status Codes

- `200` - Success (GET, PATCH, DELETE)
- `201` - Created (POST)
- `204` - No Content (DELETE with no response)
- `400` - Bad Request (validation errors)
- `401` - Unauthorized (no/invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `409` - Conflict (duplicate resource)
- `500` - Internal Server Error

---

## Authentication & Authorization

### JWT Pattern

```typescript
// ✅ Generate tokens
function generateTokens(userId: string, role: string) {
  const accessToken = jwt.sign(
    { sub: userId, role },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { sub: userId },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );
  
  return { accessToken, refreshToken };
}

// ✅ Verify and extract
async function authenticate(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({
      success: false,
      error: { message: 'No token provided' }
    });
  }
  
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = payload;
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      error: { message: 'Invalid token' }
    });
  }
}

// ✅ Role-based authorization
function authorize(...roles: string[]) {
  return (req, res, next) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        error: { message: 'Insufficient permissions' }
      });
    }
    next();
  };
}
```

---

## Data Validation

### Input Validation (Node.js with Zod)

```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(200),
});

// In controller
async function createUser(req, res, next) {
  try {
    // Validate
    const data = CreateUserSchema.parse(req.body);
    
    // Process
    const user = await userService.create(data);
    
    res.status(201).json({ success: true, data: user });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        success: false,
        error: {
          message: 'Validation failed',
          details: error.errors
        }
      });
    }
    next(error);
  }
}
```

### Input Validation (Python with Pydantic)

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8, max_length=100)
    name: str = Field(min_length=1, max_length=200)

# In route
@router.post("/", response_model=UserRead)
async def create_user(
    user_in: UserCreate,
    db: AsyncSession = Depends(get_db)
):
    # Pydantic validates automatically
    user = await user_service.create(db, user_in)
    return user
```

---

## Database Patterns

### Connection Management

```typescript
// ✅ Use connection pooling (Prisma handles this)
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient({
  log: ['query', 'error', 'warn'],
});

// Close on shutdown
process.on('SIGINT', async () => {
  await prisma.$disconnect();
  process.exit(0);
});
```

```python
# ✅ Use connection pooling (SQLAlchemy)
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine(
    DATABASE_URL,
    pool_size=20,
    max_overflow=10,
    echo=True  # Dev only
)

AsyncSessionLocal = sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)
```

### Transactions

```typescript
// ✅ Use transactions for multi-step operations
async function transferFunds(fromId: string, toId: string, amount: number) {
  return await prisma.$transaction(async (tx) => {
    // Debit
    await tx.account.update({
      where: { id: fromId },
      data: { balance: { decrement: amount } }
    });
    
    // Credit
    await tx.account.update({
      where: { id: toId },
      data: { balance: { increment: amount } }
    });
    
    // Log
    await tx.transaction.create({
      data: { fromId, toId, amount }
    });
  });
}
```

---

## Error Handling

### Custom Error Classes

```typescript
class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number,
    public code?: string
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404, 'NOT_FOUND');
  }
}

class ValidationError extends AppError {
  constructor(message = 'Validation failed') {
    super(message, 400, 'VALIDATION_ERROR');
  }
}
```

### Error Middleware

```typescript
function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  // Handle known errors
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      success: false,
      error: {
        message: err.message,
        code: err.code
      }
    });
  }
  
  // Log unexpected errors
  console.error('Unexpected error:', err);
  
  // Don't expose internal errors
  res.status(500).json({
    success: false,
    error: {
      message: 'Internal server error',
      code: 'INTERNAL_ERROR'
    }
  });
}
```

---

## Testing

### Integration Tests (Node.js)

```typescript
import request from 'supertest';
import { app } from '../src/app';

describe('POST /api/v1/users', () => {
  it('should create a new user', async () => {
    const userData = {
      email: 'test@example.com',
      password: 'SecurePass123!',
      name: 'Test User'
    };
    
    const response = await request(app)
      .post('/api/v1/users')
      .send(userData)
      .expect(201);
    
    expect(response.body.success).toBe(true);
    expect(response.body.data.email).toBe(userData.email);
    expect(response.body.data.password).toBeUndefined();
  });
  
  it('should return 400 for invalid email', async () => {
    const invalidData = {
      email: 'not-an-email',
      password: 'SecurePass123!',
      name: 'Test User'
    };
    
    const response = await request(app)
      .post('/api/v1/users')
      .send(invalidData)
      .expect(400);
    
    expect(response.body.success).toBe(false);
  });
});
```

---

## Security Best Practices

### Must Implement

- ✅ **Input Validation:** Validate all inputs with Zod/Pydantic
- ✅ **Authentication:** JWT with short expiration (15 min)
- ✅ **Password Hashing:** bcrypt with cost factor 12
- ✅ **Rate Limiting:** 100 requests/minute per IP
- ✅ **CORS:** Whitelist specific origins
- ✅ **HTTPS:** Enforce in production
- ✅ **SQL Injection Prevention:** Use ORM/parameterized queries
- ✅ **Secrets Management:** Environment variables only
- ✅ **Security Headers:** helmet.js or security middleware
- ✅ **Error Messages:** Never expose internals

### Never Do

- ❌ Return passwords in responses
- ❌ Log sensitive data (passwords, tokens)
- ❌ Use `SELECT *` queries
- ❌ Trust client-side validation only
- ❌ Hardcode secrets
- ❌ Use synchronous bcrypt
- ❌ Skip input validation

---

## Performance

### Optimization Checklist

- ✅ Database indexes on frequently queried fields
- ✅ Connection pooling configured
- ✅ Pagination on all list endpoints
- ✅ Caching for frequently accessed data
- ✅ Compression middleware enabled
- ✅ Async/await for all I/O operations
- ✅ N+1 query prevention
- ✅ Response time monitoring

---

## Documentation

### API Documentation

Use OpenAPI/Swagger for all endpoints:

```typescript
/**
 * @swagger
 * /api/v1/users:
 *   post:
 *     summary: Create a new user
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/CreateUser'
 *     responses:
 *       201:
 *         description: User created successfully
 *       400:
 *         description: Validation error
 */
```

---

## Build and Test Commands

```bash
# Development
npm run dev              # Start dev server
python -m uvicorn app.main:app --reload

# Testing
npm test                 # Run tests
pytest                   # Run tests (Python)

# Building
npm run build            # Build for production
docker build -t api .    # Build Docker image

# Linting
npm run lint             # Run ESLint
ruff check .             # Run Ruff (Python)

# Database
npx prisma migrate dev   # Run migrations
alembic upgrade head     # Run migrations (Python)
```

---

## Deployment Checklist

- [ ] Environment variables configured
- [ ] Database connection secured
- [ ] JWT secrets set
- [ ] CORS whitelist configured
- [ ] Rate limiting enabled
- [ ] Error logging setup (Sentry/CloudWatch)
- [ ] Health check endpoint working: `GET /health`
- [ ] All tests passing
- [ ] API documentation generated
- [ ] SSL certificate configured

---

**These instructions ensure consistent, production-ready API code from GitHub Copilot!**
