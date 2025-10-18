# 🚀 Getting Started with AI Rules Arsenal

**Your 5-minute guide to configuring AI coding tools**

---

## What You'll Learn

1. Choose the right tool for your needs
2. Copy and customize a configuration
3. Test your setup
4. Fine-tune for your project

**Time Required:** 5-10 minutes

---

## Step 1: Choose Your Tool (2 minutes)

### Quick Comparison

**👉 Use Windsurf if:**
- You want the most advanced features
- You need multiple rule files
- You want context-aware activation (glob patterns)
- You're building complex projects

**👉 Use Cursor if:**
- You want the largest community (60k+ members)
- You prefer simplicity (single file)
- You want battle-tested configurations
- You're an individual developer or small team

**👉 Use GitHub Copilot if:**
- You're in an enterprise environment
- You need GitHub integration
- You want official Microsoft support
- You have existing GitHub workflows

### Still Not Sure?

**Start with Cursor** - it's the easiest and most popular. You can always switch later!

---

## Step 2: Find Your Configuration (1 minute)

### By Framework/Stack

Navigate to the folder for your tool and stack:

```bash
# For Cursor users building with Next.js
cursor/by-stack/nextjs-typescript-tailwind.cursorrules

# For Windsurf users building APIs with FastAPI
windsurf/by-framework/fastapi-python.md

# For Copilot users building MERN stack
copilot/instructions/mern-fullstack.md
```

### By Use Case

Not sure which stack? Start with your use case:

```bash
# Building an MVP quickly
cursor/by-use-case/startup-mvp.cursorrules

# Enterprise production app
cursor/by-use-case/enterprise.cursorrules

# Open source library
cursor/by-use-case/open-source.cursorrules
```

### Can't Find Your Stack?

1. Check `cross-tool/` for universal patterns
2. Use a template from `templates/` folder
3. Ask in GitHub Discussions

---

## Step 3: Copy the Configuration (1 minute)

### For Windsurf

**Option A: Single Rule File (Simpler)**

```bash
# Copy to your project
cp windsurf/by-framework/nextjs-app-router.md .windsurf/rules/nextjs.md

# Edit activation mode if needed
# Open .windsurf/rules/nextjs.md and set:
# Activation: Always On
# or
# Activation: Glob: app/**/*.{ts,tsx}
```

**Option B: Multiple Rule Files (Organized)**

```bash
# Create rules directory
mkdir -p .windsurf/rules

# Copy multiple rules
cp windsurf/by-framework/nextjs-app-router.md .windsurf/rules/
cp windsurf/by-domain/security-focused.md .windsurf/rules/
cp windsurf/by-domain/testing-focused.md .windsurf/rules/
```

### For Cursor

```bash
# Copy to your project root
cp cursor/by-stack/nextjs-typescript-tailwind.cursorrules .cursorrules

# That's it! Single file, always active
```

### For GitHub Copilot

```bash
# Copy to .github folder (or root)
mkdir -p .github
cp copilot/instructions/fullstack-web.md .github/copilot-instructions.md

# Optional: Add AGENTS.md for coding agent
cp copilot/agents/microservices-AGENTS.md .github/AGENTS.md
```

---

## Step 4: Customize for Your Project (2-5 minutes)

### Update Project Information

Open your copied file and update these sections:

```markdown
# 1. Project Name
Change: # Generic Project
To:     # YourProjectName API

# 2. Tech Stack Versions
Change: - Next.js: 14.0.0+
To:     - Next.js: 14.2.0  (your actual version)

# 3. Database
Change: - Database: PostgreSQL 15
To:     - Database: MySQL 8.0  (if different)

# 4. Authentication
Change: - Auth: NextAuth.js v5
To:     - Auth: Clerk  (if you use Clerk)
```

### Add Project-Specific Rules

Add your team's specific patterns:

```markdown
## Team-Specific Patterns

### Our API Client
// We use a custom API client wrapper
import { api } from '@/lib/api-client';

// ✅ Always use this
const data = await api.get('/users');

// ❌ Don't use fetch directly
const data = await fetch('/api/users');

### Our Error Handling
// We use a custom error boundary
import { ErrorBoundary } from '@/components/ErrorBoundary';

// Wrap all async components
<ErrorBoundary>
  <AsyncComponent />
</ErrorBoundary>
```

### Remove What You Don't Need

```markdown
# If you're NOT using TypeScript, remove TypeScript rules
# If you're NOT using TailwindCSS, remove styling rules
# Keep it focused on what you actually use!
```

---

## Step 5: Test Your Configuration (2 minutes)

### Method 1: Ask AI to Generate Code

**Test Prompt:**
```
Create a new user registration API endpoint with email validation,
password hashing, and error handling.
```

**Check:**
- ✅ Does it use your specified versions?
- ✅ Does it follow your patterns?
- ✅ Does it match your file structure?

### Method 2: Ask AI to Review Code

**Test Prompt:**
```
Review this code and suggest improvements:
[paste some existing code]
```

**Check:**
- ✅ Does it reference your rules?
- ✅ Are suggestions aligned with your standards?
- ✅ Does it catch violations?

### Method 3: Check Rule Recognition

**For Windsurf:**
```
Open the Customizations panel (top right)
Check that your rules appear
Verify activation mode is correct
```

**For Cursor:**
```
Open command palette (Cmd/Ctrl + Shift + P)
Type "Cursor: Show .cursorrules"
Verify your file is being read
```

**For Copilot:**
```
Open any file
Start typing a function
Check if suggestions match your patterns
```

---

## Step 6: Fine-Tune (Ongoing)

### Week 1: Watch for Issues

**Common problems:**

**Issue:** AI ignores your rules
**Fix:** Make rules more specific with examples

**Issue:** AI suggests outdated patterns
**Fix:** Update version numbers and add "don't do this" examples

**Issue:** Rules conflict with each other
**Fix:** Consolidate into one clear rule

### Week 2: Add What's Missing

**Add rules when you notice patterns:**

```markdown
# You keep correcting AI to use your custom hook?
Add it to the rules:

## Custom Hooks
Always use our useApi hook for data fetching:
import { useApi } from '@/hooks/useApi';
```

### Week 3: Share with Team

```bash
# Commit your rules to git
git add .cursorrules  # or .windsurf/rules/
git commit -m "Add AI coding rules"
git push

# Now your whole team uses the same standards!
```

---

## Common Workflows

### Starting a New Feature

```
Prompt: "Create a new feature for [feature name] following our rules"

AI will:
✅ Use your file structure
✅ Follow your patterns
✅ Match your code style
✅ Include your required tests
```

### Refactoring Existing Code

```
Prompt: "Refactor this code to follow our standards"

AI will:
✅ Apply your patterns
✅ Fix style issues
✅ Add missing documentation
✅ Improve error handling
```

### Code Review

```
Prompt: "Review this PR for our standards"

AI will:
✅ Check against your rules
✅ Suggest improvements
✅ Flag violations
✅ Recommend best practices
```

---

## Troubleshooting

### My Rules Aren't Being Applied

**Windsurf:**
1. Check file is in `.windsurf/rules/` folder
2. Verify activation mode is set correctly
3. Check file size (max 12,000 characters)
4. Restart Windsurf

**Cursor:**
1. Verify file is named exactly `.cursorrules`
2. Check it's in project root
3. Restart Cursor
4. File size should be under 2000 lines

**Copilot:**
1. Verify file path (`.github/copilot-instructions.md` or `copilot-instructions.md`)
2. Check GitHub Copilot settings are enabled
3. Restart VS Code
4. Try explicitly mentioning the file in prompts

### AI Still Suggests Wrong Patterns

**Make rules more explicit:**

```markdown
# ❌ Vague rule (ignored)
Use modern React patterns

# ✅ Specific rule (followed)
## Data Fetching
Always use TanStack Query for server state:

```typescript
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers
});
```

Never use useEffect for data fetching.
```

### Rules Are Too Long

**Prioritize:**

1. **Keep:** Project-specific patterns
2. **Keep:** Common mistakes your team makes
3. **Keep:** Tool/library versions
4. **Remove:** General best practices (AI knows these)
5. **Remove:** Detailed framework docs (AI knows these)
6. **Remove:** Obvious patterns

---

## Next Steps

### Learn More

- 📚 [Best Practices Guide](BEST-PRACTICES.md) - Deep dive into effective rules
- 🔬 [Research Summary](RESEARCH-SUMMARY.md) - How we built this
- 📖 [Full Documentation](README.md) - Complete feature guide

### Join the Community

- 💬 [GitHub Discussions](https://github.com/yourusername/ai-rules-arsenal/discussions)
- 🐛 [Report Issues](https://github.com/yourusername/ai-rules-arsenal/issues)
- ⭐ [Star the Repo](https://github.com/yourusername/ai-rules-arsenal)

### Contribute

Found a great configuration? [Share it with the community!](CONTRIBUTING.md)

---

## Success Checklist

After following this guide, you should have:

- [x] Chosen the right tool
- [x] Copied a configuration
- [x] Customized for your project
- [x] Tested with AI
- [x] Committed to git

**Time to build! Your AI now knows your standards.** 🚀

---

## Quick Reference Card

### File Locations

```bash
# Windsurf
.windsurf/rules/*.md           # Project rules
~/global_rules.md              # Global rules (all projects)

# Cursor
.cursorrules                   # Project root (single file)

# GitHub Copilot
.github/copilot-instructions.md   # General instructions
.github/AGENTS.md                 # Coding agent specific
.github/prompts/*.md              # Reusable prompts
copilot-instructions.md           # Alternative root location
```

### Activation Modes

```markdown
# Windsurf
Activation: Always On              # Applied everywhere
Activation: Manual                 # @mention to use
Activation: Glob: **/*.ts          # TypeScript files only
Activation: Model Decision         # AI decides when to apply

# Cursor
Always active (no configuration)

# Copilot
Always active (no configuration)
```

### Testing Commands

```bash
# Generate something new
"Create a [feature] following our standards"

# Review existing code
"Review this code against our rules"

# Refactor
"Refactor this to match our patterns"

# Explain
"Explain why this violates our standards"
```

---

**Questions?** [Open a discussion!](https://github.com/yourusername/ai-rules-arsenal/discussions)

**Found this helpful?** ⭐ Star the repo!
