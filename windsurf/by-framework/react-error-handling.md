---
id: rule.react-error-rendering
type: rule
title: React Error Object Rendering Safety
tags: [react, nextjs, error-handling, debugging, best-practices]
summary: Comprehensive rules for safely rendering errors in React - prevents "Objects are not valid as a React child" crashes and establishes error handling patterns.
version: 1
---

# React Error Object Rendering Safety

**Critical Rule: Never render Error objects directly in React JSX.**

---

## 🎯 The Problem

One of the most common React crashes:

```
Error: Objects are not valid as a React child (found: [object Error])
```

This happens when you try to render an Error object directly in JSX.

---

## ❌ Don't: Render Error Objects Directly

### Example 1: Direct Error Rendering
```jsx
function Component() {
  const [error, setError] = useState(null);
  
  // ❌ BAD - Will crash
  return <div>{error}</div>;
}
```

### Example 2: String Concatenation
```jsx
function Component() {
  const [error, setError] = useState(null);
  
  // ❌ BAD - Still crashes
  return <div>Error: {error}</div>;
}
```

### Example 3: Template Literals
```jsx
function Component() {
  const [error, setError] = useState(null);
  
  // ❌ BAD - Crashes on render
  return <div>{`Error: ${error}`}</div>;
}
```

### Example 4: Conditional Rendering
```jsx
function Component() {
  const [error, setError] = useState(null);
  
  // ❌ BAD - Conditional doesn't help
  return <div>{error && error}</div>;
}
```

---

## ✅ Do: Convert to String First

### Pattern 1: Use error.message (Recommended)
```jsx
function Component() {
  const [error, setError] = useState(null);
  
  // ✅ GOOD - Safe
  return <div>{error?.message}</div>;
}
```

### Pattern 2: Type Guard with fallback
```jsx
function Component() {
  const [error, setError] = useState(null);
  
  // ✅ GOOD - Handles Error objects and strings
  return (
    <div>
      {error instanceof Error ? error.message : String(error)}
    </div>
  );
}
```

### Pattern 3: Optional Chaining
```jsx
function Component() {
  const [error, setError] = useState(null);
  
  // ✅ GOOD - Null-safe
  return <div>{error?.message || 'An error occurred'}</div>;
}
```

### Pattern 4: Dedicated Error Component
```jsx
function ErrorDisplay({ error }) {
  if (!error) return null;
  
  // ✅ GOOD - Centralized error handling
  const message = error instanceof Error 
    ? error.message 
    : String(error);
  
  return <div className="error">{message}</div>;
}

function Component() {
  const [error, setError] = useState(null);
  return <ErrorDisplay error={error} />;
}
```

---

## 🎨 Complete Examples

### Example 1: API Error Handling

```typescript
import { useState } from 'react';

interface User {
  id: string;
  name: string;
}

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(false);
  
  async function fetchUser() {
    setLoading(true);
    setError(null);
    
    try {
      const res = await fetch(`/api/users/${userId}`);
      if (!res.ok) throw new Error(`Failed to fetch user: ${res.status}`);
      const data = await res.json();
      setUser(data);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  }
  
  if (loading) return <div>Loading...</div>;
  
  // ✅ GOOD - Safe error rendering
  if (error) {
    return (
      <div className="error-container">
        <h3>Error Loading User</h3>
        <p>{error.message}</p>
        <button onClick={fetchUser}>Retry</button>
      </div>
    );
  }
  
  if (!user) return <div>No user found</div>;
  
  return <div>Welcome, {user.name}!</div>;
}
```

### Example 2: Form Validation Errors

```typescript
import { useState } from 'react';

interface FormErrors {
  email?: string;
  password?: string;
}

function LoginForm() {
  const [errors, setErrors] = useState<FormErrors>({});
  const [submitError, setSubmitError] = useState<Error | null>(null);
  
  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setSubmitError(null);
    
    try {
      // Validation
      const newErrors: FormErrors = {};
      if (!email) newErrors.email = 'Email required';
      if (!password) newErrors.password = 'Password required';
      
      if (Object.keys(newErrors).length > 0) {
        setErrors(newErrors);
        return;
      }
      
      // Submit
      const res = await fetch('/api/login', { /* ... */ });
      if (!res.ok) throw new Error('Login failed');
      
    } catch (err) {
      setSubmitError(err instanceof Error ? err : new Error(String(err)));
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      {/* ✅ GOOD - Field errors as strings */}
      {errors.email && (
        <span className="error-text">{errors.email}</span>
      )}
      
      {errors.password && (
        <span className="error-text">{errors.password}</span>
      )}
      
      {/* ✅ GOOD - Submit error safely rendered */}
      {submitError && (
        <div className="error-banner">
          {submitError.message}
        </div>
      )}
      
      <button type="submit">Login</button>
    </form>
  );
}
```

### Example 3: React Query Error Handling

```typescript
import { useQuery } from '@tanstack/react-query';

function DataComponent() {
  const { data, error, isLoading } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData
  });
  
  if (isLoading) return <div>Loading...</div>;
  
  // ✅ GOOD - React Query errors are Error objects
  if (error) {
    return (
      <div className="error">
        <h3>Failed to load data</h3>
        <p>{error.message}</p>
      </div>
    );
  }
  
  return <div>{/* render data */}</div>;
}
```

### Example 4: Next.js Server Actions

```typescript
'use client';

import { useState } from 'react';
import { submitForm } from './actions';

function ServerActionForm() {
  const [error, setError] = useState<Error | null>(null);
  const [success, setSuccess] = useState(false);
  
  async function handleSubmit(formData: FormData) {
    setError(null);
    setSuccess(false);
    
    try {
      await submitForm(formData);
      setSuccess(true);
    } catch (err) {
      // ✅ GOOD - Ensure error is Error object
      setError(err instanceof Error ? err : new Error(String(err)));
    }
  }
  
  return (
    <form action={handleSubmit}>
      {/* ✅ GOOD - Safe error display */}
      {error && (
        <div className="error-alert">
          {error.message}
        </div>
      )}
      
      {success && <div className="success">Submitted!</div>}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## 🛡️ Error Boundaries Integration

Error boundaries catch rendering errors but won't help with direct Error object rendering.

```typescript
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  error: Error | null;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { error: null };
  }
  
  static getDerivedStateFromError(error: Error) {
    return { error };
  }
  
  componentDidCatch(error: Error, errorInfo: any) {
    console.error('ErrorBoundary caught:', error, errorInfo);
  }
  
  render() {
    if (this.state.error) {
      // ✅ GOOD - Error boundary renders error message safely
      return (
        this.props.fallback || (
          <div className="error-boundary">
            <h2>Something went wrong</h2>
            <p>{this.state.error.message}</p>
            <button onClick={() => this.setState({ error: null })}>
              Try again
            </button>
          </div>
        )
      );
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary>
      <YourComponent />
    </ErrorBoundary>
  );
}
```

---

## 📝 TypeScript Patterns

### Type-Safe Error State

```typescript
// Define error type
type ErrorState = Error | null;

function Component() {
  const [error, setError] = useState<ErrorState>(null);
  
  // Type guard helper
  function getErrorMessage(error: unknown): string {
    if (error instanceof Error) return error.message;
    if (typeof error === 'string') return error;
    return 'An unknown error occurred';
  }
  
  // ✅ GOOD - Type-safe rendering
  return (
    <div>
      {error && <span>{getErrorMessage(error)}</span>}
    </div>
  );
}
```

### Custom Error Types

```typescript
class APIError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public endpoint: string
  ) {
    super(message);
    this.name = 'APIError';
  }
}

function Component() {
  const [error, setError] = useState<Error | null>(null);
  
  async function fetchData() {
    try {
      const res = await fetch('/api/data');
      if (!res.ok) {
        throw new APIError(
          'Failed to fetch',
          res.status,
          '/api/data'
        );
      }
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    }
  }
  
  // ✅ GOOD - Can access custom error properties
  return (
    <div>
      {error && (
        <div className="error">
          <p>{error.message}</p>
          {error instanceof APIError && (
            <p>Status: {error.statusCode}</p>
          )}
        </div>
      )}
    </div>
  );
}
```

---

## 🧪 Testing Error States

```typescript
import { render, screen } from '@testing-library/react';
import { Component } from './Component';

describe('Component error handling', () => {
  test('renders error message safely', () => {
    const error = new Error('Test error message');
    
    render(<Component error={error} />);
    
    // ✅ Should render error message, not crash
    expect(screen.getByText('Test error message')).toBeInTheDocument();
  });
  
  test('handles non-Error objects', () => {
    const error = 'String error';
    
    render(<Component error={error} />);
    
    // ✅ Should handle string errors too
    expect(screen.getByText('String error')).toBeInTheDocument();
  });
  
  test('handles null/undefined gracefully', () => {
    render(<Component error={null} />);
    
    // ✅ Should not crash with null
    expect(screen.queryByRole('alert')).not.toBeInTheDocument();
  });
});
```

---

## ⚠️ Common Scenarios

### 1. Async/Await Errors
```typescript
// ❌ BAD
try {
  await fetch('/api/data');
} catch (error) {
  return <div>{error}</div>; // Crashes
}

// ✅ GOOD
try {
  await fetch('/api/data');
} catch (error) {
  const message = error instanceof Error ? error.message : 'Failed to fetch';
  return <div>{message}</div>;
}
```

### 2. Promise Rejection Errors
```typescript
// ❌ BAD
fetchData()
  .catch(error => setError(error)); // Might be Error object

// Then in render:
{error && <div>{error}</div>} // Crashes if Error object

// ✅ GOOD
fetchData()
  .catch(error => {
    const err = error instanceof Error ? error : new Error(String(error));
    setError(err);
  });

// Then in render:
{error && <div>{error.message}</div>} // Safe
```

### 3. Third-Party Library Errors
```typescript
import { someLibrary } from 'some-library';

// ❌ BAD - Library might throw Error objects
try {
  someLibrary.doSomething();
} catch (error) {
  return <div>{error}</div>; // Crashes
}

// ✅ GOOD - Always convert
try {
  someLibrary.doSomething();
} catch (error) {
  const message = error instanceof Error ? error.message : String(error);
  return <div>{message}</div>;
}
```

### 4. Event Handler Errors
```typescript
function Component() {
  const [error, setError] = useState<Error | null>(null);
  
  function handleClick() {
    try {
      // Something that might throw
      riskyOperation();
    } catch (err) {
      // ✅ GOOD - Store as Error object
      setError(err instanceof Error ? err : new Error(String(err)));
    }
  }
  
  // ✅ GOOD - Render message safely
  return (
    <div>
      {error && <div className="error">{error.message}</div>}
      <button onClick={handleClick}>Click</button>
    </div>
  );
}
```

---

## 🎯 Best Practices Checklist

- [ ] **Always** use `error.message` or `String(error)` when rendering
- [ ] **Never** render Error objects directly: `{error}` ❌
- [ ] Use type guards: `error instanceof Error` ✅
- [ ] Provide fallback messages for unexpected error types
- [ ] Use optional chaining: `error?.message` ✅
- [ ] Create reusable error display components
- [ ] Test error states in your components
- [ ] Use TypeScript for type-safe error handling
- [ ] Combine with Error Boundaries for rendering errors
- [ ] Log errors to console/service for debugging

---

## 🔧 ESLint Rule (Optional)

Create custom ESLint rule to catch this pattern:

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    'no-direct-error-render': 'error'
  }
};
```

Or use existing rules:
```javascript
{
  "rules": {
    "react/jsx-no-leaked-render": "error"
  }
}
```

---

## 🔗 Related Arsenal Items

**🔄 Workflow:**
- [Next.js Debug Workflow](https://github.com/ChrisTansey007/ai-workflows-arsenal/blob/main/windsurf/debugging/nextjs-debug-compile.md) - Systematic error debugging

**💭 Memory:**
- [Next.js App Router Memory](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/project-types/nextjs-app-router-memory.md) - Error handling patterns in Next.js

**🤖 Script:**
- [Next.js Dev Starter](https://github.com/ChrisTansey007/ai-scripts-arsenal/tree/main/scripts/development/nextjs-start-dev) - Quick dev server setup

**🔗 Example:**
- [Next.js Debugging Example](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/nextjs-debugging) - Complete debugging setup

---

## 📚 Further Reading

- [React Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [Error Handling in React](https://react.dev/learn/error-boundaries)
- [TypeScript Error Handling](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#instanceof-narrowing)

---

**Remember: Error objects are objects, and React can only render primitives (strings, numbers, etc.). Always convert before rendering!** 🛡️
