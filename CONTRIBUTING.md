# 🤝 Contributing to AI Rules Arsenal

**Thank you for wanting to contribute!** This project thrives on community contributions.

---

## 🎯 What We're Looking For

### High Priority
- ✅ **New rule configurations** for popular frameworks/stacks
- ✅ **Real-world examples** from production projects
- ✅ **Improvements** to existing rules
- ✅ **Bug fixes** in documentation or examples
- ✅ **Better explanations** and clarity

### Also Welcome
- Documentation improvements
- Additional best practices
- Tool-specific optimizations
- Use case examples
- Translations (future)

---

## 📋 How to Contribute

### 1. Adding a New Rule Configuration

**Step 1: Choose the right location**

```bash
# For Windsurf
windsurf/by-framework/your-framework.md
windsurf/by-language/your-language.md
windsurf/by-domain/your-domain.md

# For Cursor
cursor/by-stack/your-stack.cursorrules
cursor/by-use-case/your-use-case.cursorrules

# For Copilot
copilot/instructions/your-instructions.md
copilot/agents/your-AGENTS.md
```

**Step 2: Use the template**

Copy from `templates/` folder:
- `windsurf-rule-template.md`
- `cursorrules-template.txt`
- `copilot-instructions-template.md`

**Step 3: Fill in your configuration**

Include:
- Project context (tech stack, versions)
- Code standards and patterns
- Examples (both ✅ good and ❌ bad)
- Common mistakes to avoid
- Testing requirements

**Step 4: Add metadata**

```markdown
---
tool: windsurf
stack: [react, typescript, vite]
use_case: spa
author: YourGitHubUsername
tested: true
last_updated: 2025-01-18
---
```

### 2. Improving Existing Rules

**Found an issue?**
- Open an issue describing the problem
- Or submit a PR with the fix

**Want to enhance?**
- Test your changes with AI first
- Show before/after examples
- Explain why your change is better

### 3. Sharing Real-World Examples

**We love production examples!**

Add to `examples/` folder:
```
examples/
├── your-project-name/
│   ├── README.md              # Project description
│   ├── .cursorrules           # Your actual config
│   ├── screenshots/           # Optional: AI output examples
│   └── results.md             # What improved?
```

---

## 📝 Contribution Guidelines

### File Naming

**Windsurf:**
```bash
lowercase-with-dashes.md
# Examples:
nextjs-app-router.md
fastapi-python.md
security-focused.md
```

**Cursor:**
```bash
lowercase-with-dashes.cursorrules
# Examples:
mern-fullstack.cursorrules
django-rest-api.cursorrules
```

**Copilot:**
```bash
lowercase-with-dashes.md
# Examples:
backend-api.md
fullstack-web.md
microservices-AGENTS.md
```

### Code Quality

**All configurations must:**
- ✅ Be tested with actual AI (Windsurf/Cursor/Copilot)
- ✅ Include real code examples
- ✅ Follow the template structure
- ✅ Have proper Markdown formatting
- ✅ Include both ✅ good and ❌ bad examples
- ✅ Specify versions for dependencies
- ✅ Be production-ready (not toy examples)

**Documentation must:**
- ✅ Be clear and concise
- ✅ Include context (when to use this)
- ✅ Have proper grammar and spelling
- ✅ Use consistent formatting
- ✅ Include links to official docs where relevant

### Testing Your Contribution

**Before submitting, test:**

1. **Generate code** with AI using your rules
2. **Review code** with AI using your rules
3. **Refactor code** with AI using your rules

**Verify:**
- AI follows your patterns
- Examples work correctly
- No conflicts with other rules
- File size is reasonable (<2000 lines Cursor, <12k chars Windsurf)

---

## 🔄 Pull Request Process

### 1. Fork the Repository

```bash
# Click "Fork" on GitHub
# Clone your fork
git clone https://github.com/YOUR-USERNAME/ai-rules-arsenal.git
cd ai-rules-arsenal
```

### 2. Create a Branch

```bash
# Create a descriptive branch
git checkout -b add-vue-typescript-rules
# or
git checkout -b fix-nextjs-example
# or
git checkout -b improve-fastapi-docs
```

### 3. Make Your Changes

- Add your configuration file(s)
- Update any relevant documentation
- Test thoroughly with AI

### 4. Commit Your Changes

```bash
git add .
git commit -m "Add Vue.js + TypeScript configuration for Cursor"
# or
git commit -m "Fix: Correct FastAPI async patterns"
# or
git commit -m "Docs: Improve Next.js example clarity"
```

**Commit message format:**
- `Add: [description]` - New content
- `Fix: [description]` - Bug fixes
- `Docs: [description]` - Documentation only
- `Update: [description]` - Updates to existing content

### 5. Push and Create PR

```bash
git push origin your-branch-name
```

Then create a Pull Request on GitHub with:

**PR Title:**
```
Add: Vue.js TypeScript Cursor rules
Fix: FastAPI async patterns
Update: Next.js App Router examples
```

**PR Description (use this template):**
```markdown
## Description
[What does this PR do?]

## Type of Change
- [ ] New rule configuration
- [ ] Bug fix
- [ ] Documentation improvement
- [ ] Example addition

## Tool & Stack
- Tool: [Windsurf/Cursor/Copilot]
- Stack: [Next.js, React, Python, etc.]
- Versions: [Specify versions]

## Testing
- [ ] Tested with AI tool
- [ ] Generated code successfully
- [ ] Follows template structure
- [ ] Examples work correctly

## Screenshots (optional)
[Before/after AI output if relevant]

## Checklist
- [ ] My code follows the contribution guidelines
- [ ] I have tested this configuration
- [ ] I have updated documentation if needed
- [ ] My commits are properly formatted
```

---

## 🎨 Style Guidelines

### Markdown Formatting

**Headers:**
```markdown
# H1 - Tool/Main Section
## H2 - Major Category
### H3 - Subsection
#### H4 - Details (rarely needed)
```

**Code Blocks:**
````markdown
```typescript
// Always specify language
const example = "like this";
```
````

**Lists:**
```markdown
- Use hyphens for unordered lists
- Keep items parallel in structure
- Use proper punctuation

1. Use numbers for ordered lists
2. When order matters
3. Like step-by-step instructions
```

**Emphasis:**
```markdown
**Bold** for important points
*Italic* for emphasis
`Code` for inline code/commands
```

### Code Examples

**Always include context:**

```markdown
## Data Fetching Pattern

✅ **Good:**
```typescript
// Use TanStack Query for server state
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers
});
```

❌ **Avoid:**
```typescript
// Don't use useEffect for data fetching
useEffect(() => {
  fetch('/api/users').then(setUsers);
}, []);
```
```

---

## 🏆 Recognition

### Contributors

All contributors are listed in:
- Repository README
- Individual rule file credits
- GitHub contributors page

### Featured Contributions

**Outstanding contributions may be:**
- Featured in README
- Highlighted on social media
- Included in video tutorials
- Showcased in blog posts

---

## ❓ Questions?

### Before Contributing

**Check existing content:**
1. Browse the repository
2. Read [BEST-PRACTICES.md](BEST-PRACTICES.md)
3. Review similar configurations

**Still have questions?**
- [Open a Discussion](https://github.com/yourusername/ai-rules-arsenal/discussions)
- [Check existing Issues](https://github.com/yourusername/ai-rules-arsenal/issues)
- Ask in the community

### Getting Help

**For support:**
- **General questions:** GitHub Discussions
- **Bug reports:** GitHub Issues
- **Feature requests:** GitHub Issues
- **Quick questions:** Comments on related PRs

---

## 📜 Code of Conduct

### Our Standards

**We are committed to:**
- 🤝 Being welcoming and inclusive
- 🎯 Focusing on what's best for the community
- 💬 Using respectful and constructive communication
- 🌟 Showing empathy towards others
- 📚 Accepting constructive criticism gracefully

**Unacceptable behavior:**
- ❌ Harassment or discrimination
- ❌ Trolling or insulting comments
- ❌ Public or private harassment
- ❌ Publishing others' private information
- ❌ Unprofessional conduct

### Enforcement

Violations may result in:
1. Warning
2. Temporary ban
3. Permanent ban

Report issues to: [contact information]

---

## 🎉 Thank You!

**Every contribution matters!**

Whether you:
- Add a new configuration
- Fix a typo
- Improve documentation
- Share your experience
- Help others in discussions

**You're making AI coding better for everyone!** 🚀

---

## 📚 Additional Resources

- [Getting Started Guide](GETTING-STARTED.md)
- [Best Practices](BEST-PRACTICES.md)
- [Research Summary](RESEARCH-SUMMARY.md)
- [Main README](README.md)

---

**Ready to contribute?** Fork the repo and start creating! 💪
