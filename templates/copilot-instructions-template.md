---
id: rule.template-copilot
type: rule
title: GitHub Copilot Instructions Template
tags: [template, copilot, documentation, howto]
summary: Template for creating new GitHub Copilot instructions with standardized structure and front-matter.
version: 1
---

# [Project Name] - GitHub Copilot Instructions

**Project Type:** [Web App / API / Mobile App / Library / etc.]  
**Last Updated:** YYYY-MM-DD

---

## Project Overview

[Brief description of what this project does and its purpose]

---

## Tech Stack

### Core
- **[Runtime/Language]:** [Name + Version]
- **[Framework]:** [Name + Version]
- **[Database]:** [Name + Version]
- **[Authentication]:** [Method/Library]

### Supporting Libraries
- **[Category]:** [Libraries used]
- **[Category]:** [Libraries used]
- **[Testing]:** [Test framework + tools]

---

## Architecture

### [Architecture Pattern]

```
[project-root]/
├── [folder1]/           # [Description]
│   ├── [subfolder]/     # [Description]
│   └── [files]
├── [folder2]/           # [Description]
└── [config-files]
```

### [Request/Data Flow]

```
[Diagram or description of how data/requests flow through the system]
```

---

## Coding Standards

### General Principles

1. **[Principle 1]:** [Description]
2. **[Principle 2]:** [Description]
3. **[Principle 3]:** [Description]

### Code Style

**[Language] Style:**
```[language]
// ✅ Use [pattern name]
[Good code example]

// ✅ Proper [aspect] handling
[Another good example]

// ❌ Don't use [anti-pattern]
[Bad code example to avoid]
```

---

## [Key Feature Area] (e.g., API Design, Data Fetching)

### [Pattern/Standard Name]

```[language]
// ✅ Recommended approach
[Code example]

// Explanation:
// [Why this approach is better]
```

### [Response/Data Format]

**[Type] format:**
```json
{
  "example": "format"
}
```

---

## Authentication & Authorization

### [Auth Method]

```[language]
// ✅ Implementation pattern
[Code example]

// Requirements:
// - [Requirement 1]
// - [Requirement 2]
```

---

## Data Validation

### Input Validation

```[language]
// ✅ Validation pattern
[Code example with validation library]
```

---

## Error Handling

### Error Response Format

```[language]
// ✅ Consistent error handling
[Code example]

// Error format:
// [Show error structure]
```

---

## Testing

### Test Structure

```[language]
// ✅ Test pattern
[Test example]

// Coverage requirements:
// - [Requirement 1]
// - [Requirement 2]
```

---

## Security Best Practices

### Must Implement

- ✅ **[Security aspect]:** [Description]
- ✅ **[Security aspect]:** [Description]
- ✅ **[Security aspect]:** [Description]

### Never Do

- ❌ [Dangerous practice]
- ❌ [Dangerous practice]
- ❌ [Dangerous practice]

---

## Performance

### Optimization Guidelines

- ✅ [Optimization 1]
- ✅ [Optimization 2]
- ✅ [Optimization 3]

---

## Documentation

### Code Documentation

```[language]
/**
 * [Documentation format]
 * 
 * @param [param] - [Description]
 * @returns [Description]
 * 
 * @example
 * [Usage example]
 */
```

### [Other Documentation Type]

- ✅ [Requirement]
- ✅ [Requirement]

---

## Build and Test Commands

```bash
# Development
[dev command]              # [Description]

# Testing
[test command]             # [Description]
[coverage command]         # [Description]

# Building
[build command]            # [Description]

# Linting
[lint command]             # [Description]
```

---

## Deployment Checklist

- [ ] [Deployment requirement]
- [ ] [Deployment requirement]
- [ ] [Deployment requirement]
- [ ] Environment variables configured
- [ ] All tests passing
- [ ] Build succeeds

---

**These instructions ensure consistent, production-ready code from GitHub Copilot!**
