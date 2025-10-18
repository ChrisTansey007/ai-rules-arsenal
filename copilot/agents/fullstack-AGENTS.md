# Full-Stack Application - GitHub Copilot Coding Agent Instructions

**Project Type:** Full-Stack Web Application  
**For:** GitHub Copilot Coding Agent (Autonomous Background Agent)  
**Last Updated:** 2025-01-18

---

## Repository Overview

This is a full-stack web application with separate frontend and backend services. The coding agent should understand the complete architecture and be able to work autonomously on features across the stack.

### Services

- **Frontend:** React/Next.js SPA
- **Backend:** Node.js/Express or Python/FastAPI API
- **Database:** PostgreSQL
- **Cache:** Redis
- **Queue:** Background job processing

---

## Project Structure

```
/
├── apps/
│   ├── web/                  # Frontend (React/Next.js)
│   │   ├── src/
│   │   ├── public/
│   │   ├── tests/
│   │   └── package.json
│   │
│   └── api/                  # Backend (Node/Python)
│       ├── src/
│       ├── tests/
│       └── package.json
│
├── packages/
│   └── shared/               # Shared types/utils
│
├── docker-compose.yml        # Local development
├── .github/
│   └── workflows/            # CI/CD pipelines
└── docs/                     # Documentation
```

---

## Build & Test Commands

### Development Setup

```bash
# Install all dependencies
npm install
# or
pip install -r requirements.txt

# Start development environment
docker-compose up -d         # Start Postgres, Redis

# Start services
npm run dev:web              # Frontend on http://localhost:3000
npm run dev:api              # Backend on http://localhost:5000
```

### Before Committing

**Run these commands and ensure all pass:**

```bash
# 1. Run linters
npm run lint                 # ESLint (frontend)
npm run lint:api             # ESLint/Ruff (backend)

# 2. Format code
npm run format               # Prettier (all)

# 3. Type checking
npm run typecheck            # TypeScript

# 4. Run tests
npm run test                 # Unit tests
npm run test:integration     # Integration tests

# 5. Build verification
npm run build:web            # Verify frontend builds
npm run build:api            # Verify backend builds
```

### CI/CD Pipeline

```bash
# These run automatically in GitHub Actions
npm run ci                   # All checks + tests
npm run test:e2e             # End-to-end tests (Playwright)
npm run test:coverage        # Coverage report
```

---

## Testing Instructions

### Before Marking Work Complete

**You MUST verify:**

1. ✅ **All tests pass**
   ```bash
   npm test
   # Expected: All tests pass, no failures
   ```

2. ✅ **Linters pass with zero errors**
   ```bash
   npm run lint
   # Expected: No errors (warnings acceptable)
   ```

3. ✅ **TypeScript compiles**
   ```bash
   npm run typecheck
   # Expected: No type errors
   ```

4. ✅ **Build succeeds**
   ```bash
   npm run build
   # Expected: Clean build, no errors
   ```

5. ✅ **Code coverage maintained**
   ```bash
   npm run test:coverage
   # Expected: Coverage >= 75% (don't decrease existing coverage)
   ```

### Test Writing Requirements

**For new features, you MUST include:**

- **Unit tests** for all business logic
- **Integration tests** for API endpoints
- **Component tests** for UI components
- **E2E test** for critical user flows (if applicable)

**Test location:**
- Frontend tests: `apps/web/src/**/__tests__/`
- Backend tests: `apps/api/tests/`

---

## Validation Requirements

### Code Quality Standards

#### TypeScript/JavaScript
- ✅ No `any` types (use `unknown` with type guards)
- ✅ ESLint must pass with zero errors
- ✅ Prettier formatting applied
- ✅ No `console.log` in production code
- ✅ All functions have JSDoc comments
- ✅ Async functions properly awaited

#### Python
- ✅ Type hints on all functions
- ✅ Ruff/Flake8 must pass
- ✅ Black formatting applied
- ✅ No debug print statements
- ✅ All functions have docstrings
- ✅ Async functions properly awaited

### Security Requirements

**MUST implement for all code:**

- ✅ **Input Validation:** All user inputs validated (Zod/Pydantic)
- ✅ **Authentication:** Protected routes require valid JWT
- ✅ **Authorization:** Role-based access control where needed
- ✅ **SQL Injection:** Use ORM/parameterized queries only
- ✅ **XSS Prevention:** Sanitize all user-generated content
- ✅ **CSRF Protection:** Implemented for state-changing operations
- ✅ **Rate Limiting:** Applied to public endpoints
- ✅ **Secrets:** Never hardcode, use environment variables

**MUST NOT:**
- ❌ Return passwords or tokens in responses
- ❌ Log sensitive information
- ❌ Trust client-side validation only
- ❌ Use `eval()` or similar dangerous functions
- ❌ Expose internal error details to clients

### Performance Requirements

- ✅ **API Response Time:** < 200ms (p95)
- ✅ **Database Queries:** Indexed on commonly queried fields
- ✅ **N+1 Queries:** Prevented with proper eager loading
- ✅ **Pagination:** Implemented on all list endpoints
- ✅ **Caching:** Applied to frequently accessed data
- ✅ **Bundle Size:** Frontend bundles < 500KB gzipped

---

## Deployment Process

### Pre-Deployment Checklist

Before deploying, verify:

- [ ] All tests passing in CI/CD
- [ ] No TypeScript/linting errors
- [ ] Database migrations created and tested
- [ ] Environment variables documented
- [ ] API documentation updated (OpenAPI/Swagger)
- [ ] Changelog updated with changes
- [ ] Version bumped appropriately (semver)

### Deployment Commands

```bash
# Staging
npm run deploy:staging

# Production
npm run deploy:prod

# Rollback (if needed)
npm run deploy:rollback
```

### Post-Deployment Verification

```bash
# 1. Health checks
curl https://api.example.com/health
# Expected: {"status": "healthy"}

# 2. Smoke tests
npm run test:smoke:prod

# 3. Monitor logs
npm run logs:prod

# 4. Check error rates
# Dashboard: https://monitoring.example.com
```

---

## Special Considerations

### Database Migrations

**IMPORTANT:** Always create migrations for schema changes.

```bash
# Create migration
npm run migration:create -- --name add_users_table

# Run migrations (dev)
npm run migration:run

# Rollback migration (if needed)
npm run migration:rollback
```

**Migration rules:**
- ✅ Make migrations backward compatible when possible
- ✅ Test migrations on staging first
- ✅ Include rollback logic
- ✅ Document breaking changes

### API Changes

**If modifying API endpoints:**

1. Update OpenAPI/Swagger documentation
2. Maintain backward compatibility or version the API
3. Update frontend API client
4. Test with integration tests
5. Document breaking changes in CHANGELOG

### Frontend Changes

**If modifying UI components:**

1. Ensure responsive design (mobile, tablet, desktop)
2. Test accessibility (WCAG 2.1 AA)
3. Update Storybook stories (if applicable)
4. Test in multiple browsers
5. Check bundle size impact

---

## Common Workflows

### Adding a New Feature

**Steps the agent should follow:**

1. **Create branch** from `main`
   ```bash
   git checkout -b feature/user-profile
   ```

2. **Implement feature**
   - Write tests first (TDD)
   - Implement code
   - Update documentation

3. **Run validation**
   ```bash
   npm run validate  # Runs all checks
   ```

4. **Commit changes**
   ```bash
   git add .
   git commit -m "feat: add user profile page"
   ```

5. **Push and create PR**
   ```bash
   git push origin feature/user-profile
   ```

### Fixing a Bug

**Steps the agent should follow:**

1. **Create branch** from `main`
   ```bash
   git checkout -b fix/login-error
   ```

2. **Write failing test** that reproduces the bug

3. **Fix the bug**

4. **Verify test passes**

5. **Run full test suite**

6. **Commit and push**
   ```bash
   git commit -m "fix: resolve login timeout issue"
   git push origin fix/login-error
   ```

### Refactoring Code

**Guidelines:**

- ✅ Maintain existing test coverage
- ✅ Don't change behavior (unless intentional)
- ✅ Update tests if internal structure changes
- ✅ Run performance benchmarks before/after
- ✅ Document reasons for refactoring

---

## Error Handling

### Expected Behavior

**When errors occur during work:**

1. **Log the error** with context
   ```typescript
   console.error('Failed to create user:', {
     error: error.message,
     data: sanitizedData
   });
   ```

2. **Return helpful error messages**
   - Include error code
   - Provide troubleshooting steps
   - Don't expose internals

3. **Clean up partial work**
   - Rollback database transactions
   - Remove temporary files
   - Reset state

4. **Report to monitoring** (in production)
   - Sentry/DataDog/CloudWatch
   - Include stack trace
   - Tag with severity

---

## Resource Limits

### Development

- **Memory:** 4GB per service
- **CPU:** 2 cores per service
- **Disk:** 10GB for local data

### Production

- **Frontend:** Serverless (Vercel/Netlify)
- **Backend:** 2GB RAM, 1 vCPU per instance
- **Database:** Dedicated instance, 4GB RAM
- **Redis:** 1GB cache

---

## Monitoring & Observability

### Health Checks

**Endpoints to monitor:**

```bash
GET /health               # Basic health
GET /health/ready         # Ready to serve traffic
GET /health/live          # Process is alive
```

**Expected responses:**
```json
{
  "status": "healthy",
  "timestamp": "2025-01-18T20:00:00Z",
  "version": "1.2.3",
  "dependencies": {
    "database": "up",
    "redis": "up"
  }
}
```

### Logging

**Log levels:**
- `ERROR` - Errors requiring attention
- `WARN` - Warnings, degraded functionality
- `INFO` - Important events
- `DEBUG` - Detailed debug info (dev only)

**What to log:**
- ✅ Request/response (sanitized)
- ✅ Errors with stack traces
- ✅ Performance metrics
- ✅ Authentication attempts

**Never log:**
- ❌ Passwords or tokens
- ❌ Credit card numbers
- ❌ Personal identifiable information (PII)

---

## Support & Troubleshooting

### Common Issues

**Build failures:**
1. Clear caches: `npm run clean`
2. Reinstall dependencies: `rm -rf node_modules && npm install`
3. Check Node version: `node --version` (should be 20.x)

**Test failures:**
1. Run tests individually to isolate
2. Check test database is running
3. Clear test data: `npm run test:db:reset`

**Deployment failures:**
1. Check environment variables are set
2. Verify database migrations ran
3. Check health endpoint

### Getting Help

**If blocked:**
1. Check existing documentation in `/docs`
2. Review recent commits for similar work
3. Check GitHub Issues for known problems
4. Ask team in PR comments

---

## Quality Gates

### Automated Checks (CI/CD)

**These MUST pass before merge:**

- ✅ All unit tests pass
- ✅ All integration tests pass
- ✅ E2E tests pass
- ✅ Code coverage >= 75%
- ✅ No linting errors
- ✅ No TypeScript errors
- ✅ Build succeeds
- ✅ Bundle size within limits
- ✅ Security scan passes (Snyk/Dependabot)

### Manual Review

**Humans will verify:**

- Code quality and readability
- Architecture decisions
- Security implications
- Performance impact
- Documentation completeness

---

## Success Criteria

### Definition of Done

A feature/fix is considered "done" when:

- [ ] Code implemented and working
- [ ] Tests written and passing (unit + integration)
- [ ] Documentation updated
- [ ] API docs updated (if applicable)
- [ ] CHANGELOG updated
- [ ] No linting/type errors
- [ ] Code reviewed and approved
- [ ] Deployed to staging and verified
- [ ] Performance acceptable
- [ ] Security reviewed

---

## Emergency Procedures

### Critical Bug in Production

1. **Immediate rollback** if recent deployment
   ```bash
   npm run deploy:rollback
   ```

2. **Create hotfix branch** from production tag
   ```bash
   git checkout -b hotfix/critical-bug production-tag
   ```

3. **Fix and test thoroughly**

4. **Fast-track deployment**
   - Skip staging if critical
   - Monitor closely after deploy

5. **Post-mortem**
   - Document what happened
   - Add tests to prevent recurrence
   - Update monitoring/alerts

---

## Contact & Escalation

### For the Agent

**When to escalate to humans:**
- Security vulnerabilities discovered
- Major architectural changes needed
- Breaking changes required
- Data loss risk
- Production incidents
- Unclear requirements

**How to escalate:**
- Comment on PR with `@team` mention
- Create GitHub issue with `priority:high` label
- Update task status to "blocked"

---

## Version History

- **v1.0.0** (2025-01-18) - Initial instructions
- Future versions will be documented here

---

**These instructions enable GitHub Copilot Coding Agent to work autonomously while maintaining high quality and safety standards!**
