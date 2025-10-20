---
id: rule.prompt-quality
type: rule
title: Prompt Quality Standards - 5-Dimensional Evaluation Framework
tags: [prompt-engineering, quality, evaluation, standards, scoring]
summary: Systematic framework for evaluating and improving prompt quality using 5-dimensional scoring (Clarity, Specificity, Constraints/Format, Tool Use, Outcome) and anti-fragility patterns.
version: 1
---

# Prompt Quality Standards

**Evaluate and improve prompts using systematic quality criteria.**

---

## 🎯 The 5-Dimensional Quality Model

Every prompt should score well on these 5 dimensions (scale: 1-5):

### 1. Clarity (C)
*How unambiguous and understandable is the prompt?*

**5** - Crystal clear, no room for misinterpretation  
**4** - Clear with minor ambiguity possible  
**3** - Mostly clear but some terms need definition  
**2** - Multiple interpretations possible  
**1** - Confusing or contradictory  

**High-scoring example (5/5):**
```
"Create a TypeScript interface named 'User' with these exact fields:
- id: number
- email: string (must be valid email format)
- createdAt: Date
- isActive: boolean"
```

**Low-scoring example (1/5):**
```
"Make a user thing"
```

---

### 2. Specificity (S)
*How targeted and precise is the request?*

**5** - Exact requirements, clear scope  
**4** - Well-defined with minor gaps  
**3** - General direction provided  
**2** - Vague requirements  
**1** - No specific direction  

**High-scoring example (5/5):**
```
"Refactor the getUserById function to:
1. Use async/await instead of callbacks
2. Maintain existing error handling for 404 and 500
3. Add JSDoc comments with @param and @returns
4. Keep the same function signature"
```

**Low-scoring example (1/5):**
```
"Improve the code"
```

---

### 3. Constraints & Format (C&F)
*Are output requirements and boundaries specified?*

**5** - Complete format spec, all constraints clear  
**4** - Most format/constraints defined  
**3** - Some guidance on output  
**2** - Minimal format specification  
**1** - No format/constraints mentioned  

**High-scoring example (5/5):**
```
"Output as Markdown table with columns: Name | Type | Description | Required
- Include exactly 5 rows
- Sort alphabetically by Name
- Use code formatting for Type column
- Mark required fields with ✓"
```

**Low-scoring example (1/5):**
```
"List the properties"
```

---

### 4. Tool/Resource Use (T)
*Does the prompt leverage available tools, examples, or references?*

**5** - Explicitly uses tools, cites examples, references docs  
**4** - Mentions relevant resources  
**3** - Implies tool use  
**2** - Ignores available tools  
**1** - Unaware of relevant tools  

**High-scoring example (5/5):**
```
"Using the Button component from @/components/ui (shadcn/ui):
1. Create a primary action button
2. Add loading state support  
3. Reference shadcn/ui button docs for prop types
4. Use Lucide React icons for the icon prop"
```

**Low-scoring example (1/5):**
```
"Create a button"
```

---

### 5. Outcome Quality (O)
*How likely is this prompt to produce high-quality results?*

**5** - Excellent setup for success  
**4** - Good likelihood of quality output  
**3** - Acceptable but could be improved  
**2** - Likely to need revision  
**1** - Unlikely to produce useful output  

*This is a predictive/retrospective score based on combining the other 4 dimensions*

---

## 📊 Scoring Examples

### Excellent Prompt (24/25)
```
"Create a FastAPI endpoint for user registration:

Endpoint: POST /api/v1/users/register

Request Body (Pydantic model):
- email: EmailStr (validated format)
- password: str (min 8 chars, must include number and special char)

Response (200):
{
  "user_id": "uuid",
  "email": "string",
  "created_at": "ISO 8601 datetime"
}

Error Responses:
- 400: Validation failure (return field-specific errors)
- 409: Email already exists

Include:
- Pydantic models for request/response
- Password hashing with bcrypt
- Proper HTTP status codes
- OpenAPI documentation tags"
```

**Scores:** C:5 | S:5 | C&F:5 | T:4 | O:5 = **24/25**

---

### Poor Prompt (6/25)
```
"Make an API endpoint"
```

**Scores:** C:2 | S:1 | C&F:1 | T:1 | O:1 = **6/25**

---

## 🛡️ Anti-Fragility Patterns

Make prompts robust against edge cases, missing data, and ambiguity:

### Pattern 1: Fallback Clauses
```
"If {CONDITION} is unclear, ask me to clarify before proceeding."
```

**Example:**
```
"If the authentication method (OAuth vs API key) is unclear, ask which to use before implementing."
```

### Pattern 2: Assumption Declaration
```
"If no {PARAMETER} is specified, default to {VALUE} and state the assumption."
```

**Example:**
```
"If no database is specified, assume PostgreSQL and note this assumption in comments."
```

### Pattern 3: Alternative Paths
```
"If approach A is not feasible, propose approach B with reasoning."
```

**Example:**
```
"If async/await syntax isn't supported in the target environment, fall back to Promises with explanation."
```

### Pattern 4: Quality Gates
```
"Before finalizing, verify: {CHECKLIST}. If any item fails, revise."
```

**Example:**
```
"Before completing, verify:
- All imports resolve correctly
- No TypeScript errors
- All functions have JSDoc
If any check fails, fix before responding."
```

### Pattern 5: Scope Limiting
```
"Focus only on {SCOPE}. Explicitly note anything out of scope."
```

**Example:**
```
"Focus only on the authentication logic. Note any database or UI concerns as out of scope."
```

---

## ✅ Pre-Send Quality Checklist

**Before sending any prompt, verify:**

- [ ] **Clarity:** Could this be misinterpreted? → Add specificity
- [ ] **Specificity:** Is the scope clear? → Narrow if too broad
- [ ] **Constraints:** Are format/output requirements specified? → Add examples
- [ ] **Tools:** Am I referencing available resources? → Cite relevant docs/examples
- [ ] **Robustness:** What if assumptions are wrong? → Add fallback clauses

---

## 🔄 Upgrade Framework

| Total Score | Quality Level | Action Required |
|-------------|---------------|-----------------|
| < 10/25 | Poor | Major rewrite needed - clarify intent |
| 10-15/25 | Fair | Add constraints and specificity |
| 15-20/25 | Good | Minor refinements, add anti-fragility |
| 20-25/25 | Excellent | Consider adding to prompt library |

---

## 📈 Improvement Workflow

**For any prompt scoring < 20/25:**

### Step 1: Identify Weak Dimensions
Which scores are < 4? Focus there first.

### Step 2: Apply Targeted Fixes

**Low Clarity?**
- Remove ambiguous terms
- Define technical terms
- Break complex requests into steps

**Low Specificity?**
- Add concrete examples
- Specify exact requirements
- Define success criteria

**Low Constraints/Format?**
- Specify output structure
- Provide format examples
- Set boundaries

**Low Tool Use?**
- Reference relevant docs
- Cite examples
- Mention available libraries

### Step 3: Add Anti-Fragility
Choose 1-2 patterns to make prompt more robust.

### Step 4: Re-Score
Rate the improved prompt - aim for 20+/25.

---

## 💡 Quick Wins

### From Fair to Good (+5 points)

**Before:**
```
"Create a login form"
```
C:2 | S:2 | C&F:1 | T:1 | O:2 = 8/25

**After:**
```
"Create a React login form with:
- Email input (validated format)
- Password input (min 8 chars)
- Submit button
- Error message display
Use shadcn/ui components
Output as TypeScript functional component"
```
C:5 | S:4 | C&F:4 | T:4 | O:4 = 21/25

**Changes:** Added specific requirements, format spec, tool reference

---

### From Good to Excellent (+4 points)

**Before:**
```
"Write tests for the getUserById function using Jest"
```
C:4 | S:3 | C&F:3 | T:3 | O:3 = 16/25

**After:**
```
"Write Jest tests for getUserById function covering:

Test cases:
1. Valid ID returns user object
2. Invalid ID throws 404 error
3. Database error throws 500 error

Each test should:
- Use describe/it blocks
- Mock database calls with jest.mock
- Assert on response structure
- Verify error messages

Output: Complete test file with imports and setup"
```
C:5 | S:5 | C&F:5 | T:4 | O:5 = 24/25

**Changes:** Specific test cases, structure requirements, mocking guidance

---

## 🔗 Related Arsenal Items

**💭 Memories:**
- [Prompt Patterns Library](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/prompt-engineering/prompt-patterns-library.md) - Proven patterns

**📝 Prompts:**
- [Prompt Forensics Analyzer](https://github.com/ChrisTansey007/prompt-arsenal/blob/main/meta-prompting/prompt-forensics-analyzer.md) - Analyze conversations

**🔄 Workflows:**
- [Insights Extraction Pipeline](https://github.com/ChrisTansey007/ai-workflows-arsenal/blob/main/windsurf/meta-analysis/insights-extraction-pipeline.md) - Extract patterns

---

**Result: Systematic quality assurance for all AI interactions!** 📊
