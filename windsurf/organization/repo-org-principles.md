---
id: rule.repo-org-principles
type: rule
title: Repository Organization Principles
tags: [organization, best-practices, structure, maintainability, standards]
summary: Core principles for creating maintainable repository structures - organize by purpose, use numbered prefixes, keep 2-3 levels max, evidence-driven decisions.
version: 1
---

# Repository Organization Principles

**7 proven principles for creating maintainable, discoverable repository structures.**

---

## 🎯 Core Principle

> **Organize by PURPOSE, not by file type.**

Good repositories group files by their function in the workflow, not by their extension.

---

## 📋 The 7 Principles

### 1. Organize by PURPOSE, not type

**❌ Don't organize by file type:**
```
repo/
├── markdown/
│   ├── setup.md
│   ├── api.md
│   └── deploy.md
├── python/
│   ├── start.py
│   ├── test.py
│   └── deploy.py
└── javascript/
    ├── build.js
    └── test.js
```

**✅ Do organize by function/purpose:**
```
repo/
├── docs/
│   ├── 01-GUIDES/setup.md
│   ├── 02-TECHNICAL/api.md
│   └── 03-RUNBOOKS/deployment.md
└── scripts/
    ├── 00-STARTUP/start.py
    ├── 03-TESTING/test.py
    └── 05-DEPLOYMENT/deploy.sh
```

**Why:** Files are found by **what they do**, not what they are.

---

### 2. Numbered prefixes for logical ordering

Use `00-`, `01-`, `02-` prefixes to create natural ordering.

**Three types of ordering:**

**A) Workflow/Temporal Order** (when things happen):
```
scripts/
├── 00-STARTUP/      # First: initialize
├── 01-DEVELOPMENT/  # Then: develop
├── 02-DATABASE/     # Then: set up data
├── 03-TESTING/      # Then: test
├── 04-BUILD/        # Then: build
├── 05-DEPLOYMENT/   # Finally: deploy
└── 06-OPERATIONS/   # Ongoing: maintain
```

**B) Priority/Importance Order**:
```
docs/
├── 00-CURRENT/      # Most important: active work
├── 01-GUIDES/       # High priority: how-to
├── 02-TECHNICAL/    # Medium: reference
├── 03-RUNBOOKS/     # Medium: procedures
└── 05-ARCHIVE/      # Low priority: historical
```

**C) Frequency Order**:
```
scripts/
├── 00-DAILY/        # Run every day
├── 01-WEEKLY/       # Run weekly
├── 02-MONTHLY/      # Run monthly
└── 03-ADHOC/        # Run as needed
```

**Why:** Directories appear in meaningful order without mental mapping.

---

### 3. Keep hierarchy 2-3 levels maximum

**❌ Too deep (hard to navigate):**
```
scripts/backend/api/testing/unit/services/user/test-auth.js
```

**✅ Good (easy to find):**
```
scripts/03-TESTING/api/test-user-auth.js
```

**Rule of thumb:**
- **1 level:** Only for tiny projects (< 20 files)
- **2 levels:** Sweet spot for most projects
- **3 levels:** Maximum for large projects
- **4+ levels:** Almost always too deep

**Why:** Deep nesting increases cognitive load and path length.

---

### 4. Self-explanatory names

Names should be immediately clear without context.

**❌ Vague or cryptic:**
```
docs/
├── stuff/
├── misc/
├── old/
├── temp/
└── new/

scripts/
├── utils/
├── helpers/
└── tools/
```

**✅ Clear and specific:**
```
docs/
├── 00-CURRENT/         # Currently active docs
├── 01-GUIDES/          # How-to guides
├── 05-ARCHIVE/         # Historical reference
└── 06-REFERENCE/       # External resources

scripts/
├── 00-STARTUP/         # Initialization
├── 03-TESTING/         # Test runners
└── 07-UTILITIES/       # General purpose tools
```

**Guidelines:**
- Use full words, not abbreviations (`DEPLOYMENT` not `DEPLOY` or `DEP`)
- Be specific about purpose (`ARCHIVE` not `OLD`)
- Use consistent terminology across repo
- Avoid generic terms (utils, misc, stuff, things)

**Why:** Reduces guessing and questions from team members.

---

### 5. Evidence-driven decisions

Document **why** files go where they do.

**In migration scripts:**
```powershell
# ✅ Good - documents reasoning
Move-File "start.ps1" "scripts/00-STARTUP/start.ps1" `
    "Main entry point - runs first in workflow"

Move-File "user-auth-test.js" "scripts/03-TESTING/api/user-auth-test.js" `
    "Tests user authentication API endpoint"
```

**In README or CONTRIBUTING.md:**
```markdown
## Where to Add Files

### Scripts

**00-STARTUP/** - Scripts that initialize the application
- Entry points (start.py, init.sh)
- Environment setup
- Dependency checks

**03-TESTING/** - Test runners and quality checks
- Unit tests
- Integration tests
- E2E tests
```

**Why:** New team members understand the system. Decisions are reviewable and can be challenged if wrong.

---

### 6. Plan for growth

Leave room for future categories.

**❌ No room to grow:**
```
docs/
├── 00-CURRENT/
├── 01-GUIDES/
├── 02-TECHNICAL/
├── 03-RUNBOOKS/
├── 04-PLANNING/
├── 05-ARCHIVE/
└── 06-REFERENCE/
# What if we need another category between GUIDES and TECHNICAL?
```

**✅ Gaps allow insertion:**
```
docs/
├── 00-CURRENT/      # Gap allows 01 if needed
├── 02-GUIDES/       # Gap allows 03 if needed
├── 04-TECHNICAL/    # Gap allows 05 if needed
├── 06-RUNBOOKS/
├── 08-PLANNING/
├── 10-ARCHIVE/
└── 12-REFERENCE/
```

**Or use increments:**
```
scripts/
├── 00-STARTUP/
├── 10-DEVELOPMENT/
├── 20-DATABASE/
├── 30-TESTING/
├── 40-BUILD/
└── 50-DEPLOYMENT/
```

**Why:** Adding categories doesn't require renumbering everything.

---

### 7. Safety first

Migrations should be safe, reversible, and verifiable.

**Required safety features:**

✅ **Dry-run by default**
```powershell
param([switch]$DryRun = $true)  # Must opt-in to real execution
```

✅ **Automated backup**
```powershell
if (-not $DryRun) {
    $timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
    Copy-Item -Path . -Destination "../backup-$timestamp" -Recurse
}
```

✅ **Existence checks**
```powershell
if (Test-Path $srcPath) {
    Move-Item ...
} else {
    Write-Warning "File not found: $srcPath"
}
```

✅ **Progress tracking and logging**
```powershell
Write-Host "✓ Moved: $filename" -ForegroundColor Green
$movedCount++
Write-Host "`nTotal files moved: $movedCount"
```

✅ **Rollback documentation**
```markdown
## Rollback Procedure

1. Stop application
2. Restore from backup: `cp -r ../backup-20251019/* .`
3. Or use git: `git checkout HEAD -- .`
4. Restart application
```

**Why:** Mistakes happen. Make them easy to undo.

---

## ❌ Anti-Patterns to Avoid

### 1. Organizing by File Type
```
❌ /markdown, /python, /javascript
✅ /docs, /scripts, /config
```

### 2. Deep Nesting (4+ levels)
```
❌ scripts/backend/api/testing/unit/services/user/auth/
✅ scripts/03-TESTING/api/test-user-auth.js
```

### 3. Vague or Generic Names
```
❌ /stuff, /misc, /old, /new, /utils, /temp
✅ /ARCHIVE, /CURRENT, /UTILITIES (with clear purpose)
```

### 4. No Documentation
```
❌ Undocumented structure, team guesses where files go
✅ CONTRIBUTING.md explains category purposes
```

### 5. No Room for Growth
```
❌ 00, 01, 02, 03... (consecutive numbering)
✅ 00, 10, 20, 30... or 00, 02, 04, 06... (gaps)
```

### 6. Migrating Without Testing
```
❌ Run script directly in production
✅ Dry-run → Review → Backup → Execute → Verify
```

### 7. Inconsistent Naming
```
❌ /docs/current, /scripts/STARTUP, /config/Active
✅ /docs/00-CURRENT, /scripts/00-STARTUP, /config/00-ACTIVE
```

---

## 🎯 When to Apply These Principles

**Good candidates for reorganization:**
- 50+ files in root directory
- Team members spend 5+ minutes finding files
- "Where does this go?" questions are frequent
- Onboarding takes 2+ hours
- Import paths are confusing
- Multiple directories for same purpose

**Warning signs you need this:**
- Files named `new-`, `old-`, `final-`, `final-v2-`
- Directories named `stuff/`, `misc/`, `tmp/`
- Team Slack has repeated "where is X file?" questions
- New developers create their own organization system

---

## 📚 Examples from the Wild

### Well-Organized Open Source Projects

**Next.js:**
```
nextjs/
├── packages/          # Monorepo packages by feature
├── examples/          # Usage examples
├── docs/              # Documentation
└── test/              # Test suites
```

**React:**
```
react/
├── packages/
│   ├── react/         # Core package
│   ├── react-dom/     # DOM renderer
│   └── scheduler/     # Scheduler
└── scripts/           # Build and test scripts
```

**Principles applied:**
- Organize by purpose (packages, examples, docs)
- Clear category names
- 2-3 level hierarchy
- Documented structure

---

## 🔄 Evolution Over Time

**Good structure evolves with project:**

**Phase 1 (< 50 files):**
```
repo/
├── src/
├── docs/
└── scripts/
```

**Phase 2 (50-200 files):**
```
repo/
├── src/
├── docs/
│   ├── guides/
│   ├── api/
│   └── runbooks/
└── scripts/
    ├── dev/
    ├── test/
    └── deploy/
```

**Phase 3 (200+ files):**
```
repo/
├── src/
├── docs/
│   ├── 00-CURRENT/
│   ├── 01-GUIDES/
│   ├── 02-TECHNICAL/
│   └── 03-RUNBOOKS/
└── scripts/
    ├── 00-STARTUP/
    ├── 01-DEVELOPMENT/
    ├── 03-TESTING/
    └── 05-DEPLOYMENT/
```

**When to reorganize:** Annually or when finding files becomes painful.

---

## 🎓 Teaching the System

### For New Team Members

**In CONTRIBUTING.md:**
```markdown
## Repository Organization

Our repo is organized by **purpose/function**, not file type.

### Structure

**docs/** - All documentation
- `00-CURRENT/` - Active working documents
- `01-GUIDES/` - How-to guides
- `02-TECHNICAL/` - Technical specifications

**scripts/** - All automation scripts
- `00-STARTUP/` - Initialization scripts
- `03-TESTING/` - Test runners
- `05-DEPLOYMENT/` - Deployment scripts

### Where to Add New Files

1. Identify the file's **primary purpose**
2. Find the matching category
3. If unsure, look for similar existing files
4. Document your choice in the commit message

Examples:
- New API endpoint test → `scripts/03-TESTING/`
- New setup guide → `docs/01-GUIDES/`
- New deployment script → `scripts/05-DEPLOYMENT/`
```

---

## ✅ Success Criteria

You've succeeded when:

1. **New team members** can find any file in < 2 minutes
2. **"Where does this go?"** questions drop by 80%
3. **Search time** reduced from 5-10 min to 30 sec - 2 min
4. **Structure is obvious** - categories explain themselves
5. **Easy to maintain** - team follows system without enforcement
6. **Room to grow** - adding files doesn't break organization
7. **Onboarding time** cut in half

---

## 📖 Further Reading

**Concepts:**
- Domain-Driven Design (organize by domain)
- Clean Architecture (organize by feature)
- Monorepo patterns (workspace organization)

**Tools:**
- `tree` - Visualize directory structure
- `fd` or `find` - Locate files quickly
- `rg` or `grep` - Search content efficiently

---

## 🔗 Related Arsenal Items

**🔄 Workflow:**
- [Repository File Organization](https://github.com/ChrisTansey007/ai-workflows-arsenal/blob/main/windsurf/project-organization/repo-organize-files.md) - 3-phase workflow that applies these principles

**🤖 Script:**
- [Safe File Migration Script](https://github.com/ChrisTansey007/ai-scripts-arsenal/tree/main/scripts/repository-management/migrate-files) - Automated migration following these principles

**🔗 Complete Example:**
- [Repository Organization Example](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/repo-organization) - Real-world implementation

### Quick Setup

```bash
# Install Arsenal repos
curl -sSL https://raw.githubusercontent.com/ChrisTansey007/arsenal-integration-hub/main/scripts/install-all.sh | bash

# Copy this rule + workflow + script
cp ~/arsenals/ai-rules-arsenal/windsurf/organization/repo-org-principles.md .windsurf/rules/
cp ~/arsenals/ai-workflows-arsenal/windsurf/project-organization/repo-organize-files.md .windsurf/workflows/
cp ~/arsenals/ai-scripts-arsenal/scripts/repository-management/migrate-files/migrate-files-safe.ps1 ./scripts/
```

---

**Remember: Good organization is invisible. If team members have to think about where files go, the system needs improvement.** 🎯
