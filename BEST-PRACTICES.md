# 📚 AI Rules Arsenal - Best Practices Guide

**Comprehensive best practices for writing effective AI coding tool configurations**

---

## Table of Contents

1. [Universal Principles](#universal-principles)
2. [Tool-Specific Guidelines](#tool-specific-guidelines)
3. [Common Patterns](#common-patterns)
4. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
5. [Testing Your Rules](#testing-your-rules)
6. [Maintenance & Evolution](#maintenance--evolution)

---

## Universal Principles

### 1. Start with Context

**Why:** AI needs to understand your project before applying rules.

**Pattern:**
```markdown
# Project: [Name]
Description: [One-line summary]

## Tech Stack
- Language: [Primary language + version]
- Framework: [Main framework + version]
- Database: [DB + version]
- Key Libraries: [Important dependencies]

## Architecture
[High-level structure description]
```

**Example:**
```markdown
# Project: TaskFlow API
Description: Task management API with real-time updates

## Tech Stack
- Language: TypeScript 5.3+
- Framework: Express 4.18+
- Database: PostgreSQL 15 with Prisma ORM
- Key Libraries: Socket.io, Redis, Zod

## Architecture
RESTful API with WebSocket support for real-time updates.
Follows repository pattern with service layer.
```

### 2. Be Specific, Not Generic

**❌ Bad (Generic):**
```markdown
- Write clean code
- Follow best practices
- Make it performant
```

**✅ Good (Specific):**
```markdown
- Limit functions to 50 lines max
- Use async/await instead of promises.then()
- Implement database connection pooling with max 20 connections
- Cache frequently accessed data with 5-minute TTL
```

### 3. Show, Don't Just Tell

**Include code examples for patterns you want:**

```markdown
## Data Fetching Pattern

✅ **Correct:**
```typescript
const { data, isLoading } = useQuery({
  queryKey: ['user', id],
  queryFn: () => fetchUser(id)
});
```

❌ **Avoid:**
```typescript
useEffect(() => {
  fetch(`/api/users/${id}`)
    .then(res => res.json())
    .then(setUser);
}, [id]);
```
```

### 4. Organize by Priority

**Structure matters:**

```markdown
# [Project Name]

## 🔴 Critical (Always Apply)
[Non-negotiable rules]

## 🟡 Important (Usually Apply)
[Strong preferences]

## 🟢 Optional (Nice to Have)
[Suggestions]
```

### 5. Version Everything

**Track what you're targeting:**

```markdown
## Dependencies (As of 2025-01)
- Next.js: 14.0.0+ (App Router)
- React: 18.2.0+
- TypeScript: 5.3+
- TailwindCSS: 3.4+

Last Updated: 2025-01-18
Next Review: 2025-04-18
```

---

## Tool-Specific Guidelines

### Windsurf Rules

#### Use Appropriate Activation Modes

**1. Always On** - Core standards
```markdown
# global_rules.md or .windsurf/rules/core-standards.md
Activation: Always On

## Code Style
- Use TypeScript strict mode
- No console.log in production
- Early returns over nested if
```

**2. Glob Patterns** - Language-specific
```markdown
# .windsurf/rules/python-standards.md
Activation: Glob: **/*.py

## Python Standards
- Use type hints everywhere
- Follow PEP 8
- Black for formatting
```

**3. Model Decision** - Contextual
```markdown
# .windsurf/rules/security-review.md
Activation: Model Decision
Description: "Apply when working on authentication, authorization, or handling user data"

## Security Standards
- Validate all inputs
- Use parameterized queries
- Never log sensitive data
```

**4. Manual** - Specialized tools
```markdown
# .windsurf/rules/migration-helper.md
Activation: Manual (@migration-helper)

## Database Migration Guidelines
[Only apply when explicitly invoked]
```

#### Keep Rules Focused

**❌ One massive file:**
```markdown
# .windsurf/rules/everything.md (12,000 characters)
[Python + React + Docker + Security + Testing + ...]
```

**✅ Multiple focused files:**
```markdown
.windsurf/rules/
├── python-backend.md (2,000 chars)
├── react-frontend.md (1,500 chars)
├── docker-setup.md (800 chars)
├── security-standards.md (3,000 chars)
└── testing-requirements.md (1,200 chars)
```

#### Use XML Tags for Grouping

```markdown
<architecture_patterns>
- Repository pattern for data access
- Service layer for business logic
- Controller layer for HTTP handling
</architecture_patterns>

<security_requirements>
- JWT for authentication
- bcrypt for passwords
- Rate limiting: 100 req/min per IP
</security_requirements>

<testing_strategy>
- Unit tests: 80%+ coverage
- Integration tests: All API endpoints
- E2E tests: Critical user flows
</testing_strategy>
```

### Cursor Rules (.cursorrules)

#### Optimize for Single-File Format

**Structure for readability:**

```
# [Project Name] - Cursor Rules

## 🎯 Project Overview
[1-2 sentences]

## 📦 Tech Stack
[List with versions]

## 📁 Project Structure
[Directory tree]

## 🎨 Code Style
[Language-specific standards]

## 🏗️ Architecture Patterns
[Design patterns used]

## 🔒 Security
[Security requirements]

## 🧪 Testing
[Testing approach]

## 🚀 Deployment
[Deployment specifics]

## 📝 Examples
[Code examples]

## ⚠️ Common Mistakes
[Things to avoid]
```

#### Keep Under 2000 Lines

**When it grows too large:**
1. Remove duplicate info
2. Remove obvious patterns
3. Focus on project-specific rules
4. Link to external docs

**Example reduction:**

❌ **Before (3000 lines):**
```markdown
[Full TypeScript handbook]
[Full React documentation]
[Full Next.js guide]
[Your project rules]
```

✅ **After (1500 lines):**
```markdown
# See: TypeScript Handbook, React Docs, Next.js Docs

## Project-Specific TypeScript Rules
[Only deviations from standard practices]

## Project-Specific React Patterns
[Only custom patterns we use]

## Project-Specific Next.js Setup
[Only our configuration choices]
```

#### Use Sections Effectively

```markdown
## Naming Conventions
- Components: PascalCase (UserProfile.tsx)
- Hooks: camelCase with 'use' prefix (useAuth.ts)
- Utils: camelCase (formatDate.ts)
- Constants: SCREAMING_SNAKE_CASE (API_URL)
- Types: PascalCase with 'Type' suffix (UserType)

## File Organization
src/
├── app/           # Next.js App Router
├── components/    # Reusable components
│   ├── ui/       # shadcn/ui components
│   └── features/ # Feature-specific
├── lib/          # Utilities
│   ├── api/      # API clients
│   └── utils/    # Helper functions
├── hooks/        # Custom hooks
├── types/        # TypeScript types
└── styles/       # Global styles

## Import Order
1. React/Next.js imports
2. External library imports
3. Internal imports (absolute)
4. Relative imports
5. Type imports
6. Style imports

Example:
```typescript
import { useState } from 'react';
import { useRouter } from 'next/navigation';

import { Button } from '@/components/ui/button';
import { api } from '@/lib/api';

import { UserCard } from './UserCard';

import type { User } from '@/types';

import styles from './UserProfile.module.css';
```
```

### GitHub Copilot Instructions

#### Separate Concerns

**Use multiple files:**

```markdown
# copilot-instructions.md (General coding)
[Code style, patterns, testing]

# AGENTS.md (Autonomous agent specific)
[Build commands, validation, deployment]

# .github/prompts/test-writer.md (Reusable prompt)
[Specific task template]
```

#### Focus on Commands

**Agents need executable instructions:**

```markdown
## Build & Test Commands

### Development
```bash
npm install           # Install dependencies
npm run dev          # Start dev server
npm run test:watch   # Run tests in watch mode
```

### Before Committing
```bash
npm run lint         # Run ESLint
npm run format       # Run Prettier
npm run test         # Run all tests
npm run build        # Verify build works
```

### Deployment
```bash
npm run build        # Production build
npm run deploy       # Deploy to Vercel
```

## Validation Checklist
- [ ] All tests pass
- [ ] No TypeScript errors
- [ ] ESLint shows 0 errors
- [ ] Build succeeds
- [ ] No console.log statements
```

#### Include Success Criteria

```markdown
## Definition of Done

### For Features
- [ ] Code implemented and working
- [ ] Unit tests written and passing
- [ ] Integration tests passing
- [ ] Documentation updated
- [ ] PR description complete
- [ ] Code reviewed and approved

### For Bugs
- [ ] Root cause identified
- [ ] Fix implemented
- [ ] Regression test added
- [ ] Verified in multiple scenarios
- [ ] Deployment plan documented
```

---

## Common Patterns

### 1. Security-First Pattern

```markdown
## Security Standards

### Authentication
- JWT tokens with 15-minute expiration
- Refresh tokens with 7-day expiration
- Store tokens in httpOnly cookies
- CSRF protection enabled

### Input Validation
```typescript
// Always validate with Zod
const UserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100)
});

// Use at API boundaries
app.post('/api/users', async (req, res) => {
  const result = UserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error });
  }
  // Process validated data
});
```

### Data Protection
- Never log passwords or tokens
- Encrypt sensitive data at rest
- Use HTTPS only in production
- Set security headers (helmet.js)

### Database Security
- Use parameterized queries only
- Implement row-level security
- Limit database user permissions
- Regular security audits
```

### 2. Performance-First Pattern

```markdown
## Performance Requirements

### Frontend
- Bundle size < 500KB
- First Contentful Paint < 1.5s
- Time to Interactive < 3.5s
- Lighthouse score > 90

### Backend
- API response time < 200ms (p95)
- Database query time < 50ms (p95)
- Memory usage < 512MB per instance
- CPU usage < 70% under normal load

### Implementation
```typescript
// Lazy load heavy components
const HeavyChart = lazy(() => import('./HeavyChart'));

// Memoize expensive calculations
const expensiveValue = useMemo(() => {
  return heavyCalculation(data);
}, [data]);

// Debounce user input
const debouncedSearch = useDebouncedCallback(
  (value) => searchAPI(value),
  500
);

// Implement pagination
const PAGE_SIZE = 20;
const { data } = useQuery({
  queryKey: ['users', page],
  queryFn: () => fetchUsers({ page, limit: PAGE_SIZE })
});
```
```

### 3. Testing-First Pattern

```markdown
## Testing Strategy

### Coverage Requirements
- Unit Tests: 80%+ coverage
- Integration Tests: All API endpoints
- E2E Tests: Critical user journeys
- Performance Tests: All public APIs

### Test Structure (AAA Pattern)
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123!'
      };
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(result).toBeDefined();
      expect(result.email).toBe(userData.email);
      expect(result.password).not.toBe(userData.password); // Hashed
    });

    it('should throw error with invalid email', async () => {
      // Arrange
      const invalidData = {
        email: 'not-an-email',
        password: 'SecurePass123!'
      };
      
      // Act & Assert
      await expect(
        userService.createUser(invalidData)
      ).rejects.toThrow('Invalid email');
    });
  });
});
```

### What to Test
- ✅ Business logic functions
- ✅ API endpoints (request/response)
- ✅ User interactions
- ✅ Error handling
- ✅ Edge cases

### What NOT to Test
- ❌ Framework internals
- ❌ Third-party libraries
- ❌ Trivial getters/setters
- ❌ Configuration files
```

### 4. Documentation Pattern

```markdown
## Documentation Requirements

### Code Documentation
```typescript
/**
 * Fetches user by ID with caching
 * 
 * @param userId - The unique user identifier
 * @returns User object or null if not found
 * @throws {NotFoundError} If user doesn't exist
 * @example
 * const user = await getUser('user-123');
 */
async function getUser(userId: string): Promise<User | null> {
  // Implementation
}
```

### API Documentation
- OpenAPI/Swagger for all endpoints
- Example requests and responses
- Error codes documented
- Rate limits specified

### README Requirements
- Project description
- Setup instructions
- Environment variables
- Common commands
- Deployment guide
- Troubleshooting section

### Inline Comments
```typescript
// Complex algorithm - explain WHY not WHAT
// Using binary search because dataset is sorted
// and can contain 1M+ items (O(log n) required)
const index = binarySearch(sortedArray, target);
```
```

---

## Anti-Patterns to Avoid

### ❌ 1. Too Generic

```markdown
Bad:
- Write good code
- Follow best practices
- Make it fast

Good:
- Functions max 50 lines
- Use async/await for I/O
- Cache API responses for 5 minutes
```

### ❌ 2. Restating Documentation

```markdown
Bad:
[Copies entire React documentation]
[Copies entire Next.js documentation]

Good:
## Our React Patterns
We deviate from React docs in these ways:
- We use Zustand instead of Context for global state
- We prefer server components over client components
- We use TanStack Query for all data fetching
```

### ❌ 3. Contradictory Rules

```markdown
Bad:
- Use classes for components
- Use functional components only
- Prefer TypeScript
- JavaScript is fine

Good:
- Use functional components with hooks (since 2023)
- TypeScript required for all new code
- Existing JavaScript can remain until refactor
```

### ❌ 4. No Examples

```markdown
Bad:
- Use proper error handling

Good:
## Error Handling Pattern
```typescript
try {
  const result = await riskyOperation();
  return { success: true, data: result };
} catch (error) {
  logger.error('Operation failed', { error, context });
  if (error instanceof ValidationError) {
    return { success: false, error: 'Invalid input' };
  }
  throw error; // Re-throw unexpected errors
}
```
```

### ❌ 5. Ignoring Updates

```markdown
Bad:
Last updated: 2020
[References deprecated packages]

Good:
Last updated: 2025-01-18
Next review: 2025-04-18
[Current versions and patterns]
```

---

## Testing Your Rules

### 1. Ask AI to Generate Code

**Method:** Start a new conversation and ask AI to generate a feature.

```
Prompt: "Create a user authentication API endpoint with 
email/password login and JWT tokens"
```

**Check:**
- Does it follow your standards?
- Does it use specified versions?
- Does it match your patterns?

### 2. Review AI Suggestions

**Method:** Have AI review existing code.

```
Prompt: "@existing-code.ts Review this code and suggest improvements"
```

**Check:**
- Does it cite your rules?
- Are suggestions aligned with your standards?
- Does it catch violations?

### 3. Test Edge Cases

```
Prompts to test:
- "Add error handling" → Does it match your error pattern?
- "Write tests" → Does it follow your test structure?
- "Optimize performance" → Does it apply your performance rules?
- "Add security" → Does it implement your security standards?
```

### 4. Measure Impact

**Before rules:**
- Time to generate feature: X minutes
- Quality issues: Y problems
- Revisions needed: Z iterations

**After rules:**
- Time: X - 30%
- Issues: Y - 60%
- Revisions: Z - 50%

---

## Maintenance & Evolution

### Regular Reviews

**Monthly:**
- Check for outdated dependencies
- Update version numbers
- Add new patterns discovered
- Remove obsolete rules

**Quarterly:**
- Major review of all rules
- Align with framework updates
- Incorporate team feedback
- Update examples

### Version Control

```markdown
# Version History

## v2.0.0 (2025-01-18)
- Updated to Next.js 14
- Added App Router patterns
- Removed Pages Router rules

## v1.5.0 (2024-10-15)
- Added TypeScript 5.0 features
- Updated testing patterns
- Added security guidelines

## v1.0.0 (2024-06-01)
- Initial release
```

### Team Collaboration

**Process:**
1. Team member proposes rule change
2. Discussion in PR
3. Trial period (2 weeks)
4. Team vote
5. Merge or reject

**Template for proposals:**
```markdown
## Proposed Rule Change

**Current Rule:**
[What we have now]

**Proposed Rule:**
[What should change]

**Reasoning:**
[Why this improves things]

**Impact:**
[What code needs to change]

**Trial Period Results:**
[After 2 weeks]
```

---

## Summary Checklist

✅ **Good Rules Are:**
- Specific and actionable
- Include code examples
- Have clear reasoning
- Version controlled
- Regularly updated
- Team-agreed
- Tested with AI

✅ **Good Rules Include:**
- Project context
- Tech stack with versions
- File organization
- Code patterns
- Security standards
- Testing requirements
- Examples and anti-patterns

✅ **Good Rules Avoid:**
- Generic advice
- Contradictions
- Copying docs
- Being outdated
- Too much detail
- No examples

---

**Remember:** Rules should empower AI to write code like your best team member would. They're living documents that evolve with your project. 🚀
