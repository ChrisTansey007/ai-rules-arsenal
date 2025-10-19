---
id: rule.email-docs-standards
type: rule
title: Email Setup Documentation Standards
tags: [documentation, email, user-experience, technical-writing, accessibility]
summary: Comprehensive documentation standards for email setup guides - ensure clear, user-friendly instructions that address passkey authentication, app passwords, and common user confusion.
version: 1
---

# Email Setup Documentation Standards

**Write email setup documentation that users actually understand - especially those using modern authentication (passkeys).**

---

## 🎯 Core Principles

### 1. Front-Load Context (The "Why")

**Rule:** Start every guide with 1-2 sentences explaining when/why this is needed.

**❌ Bad:**
```markdown
# Gmail Setup

Step 1: Go to https://myaccount.google.com/apppasswords
```

**✅ Good:**
```markdown
# Gmail Setup

**Use this guide when connecting your Gmail account to the email app.**  
Gmail requires an app-specific password for email apps due to security requirements.

Step 1: Go to https://myaccount.google.com/apppasswords
```

**Why it matters:** Users need to know if this guide applies to them before investing time reading it.

---

### 2. Number All Steps

**Rule:** Use `1)` `2)` `3)` format - never

 unnumbered bullet lists for procedures.

**❌ Bad:**
```markdown
- Go to security settings
- Click app passwords
- Generate password
- Copy and paste
```

**✅ Good:**
```markdown
1. Go to security settings
2. Click "App Passwords"
3. Click "Generate"
4. Copy the 16-character password
5. Paste it in the password field
```

**Why it matters:** Users need to know progress ("I'm on step 3 of 5") and reference specific steps when asking for help.

---

### 3. Distinguish Password Types

**Rule:** Always label password fields with their specific type.

**❌ Bad:**
```markdown
Enter your password
```

**✅ Good:**
```markdown
Enter your **app-specific password** (not your regular Yahoo password)
```

**Even Better:**
```markdown
**App-Specific Password** (16 characters, like: abcd efgh ijkl mnop)
_Not your regular password or the one you use on Yahoo's website_
```

**Why it matters:** Users with passkeys don't have "regular passwords" - specificity prevents confusion.

---

### 4. Address Passkeys Explicitly

**Rule:** If provider commonly uses passkeys, mention it upfront.

**❌ Bad:**
```markdown
Enter your password to continue
```

**✅ Good:**
```markdown
**Don't have a password?**  
If you sign in to Yahoo with your fingerprint or face (passkey), you'll need to generate an app-specific password for this email app. See instructions below.
```

**Why it matters:** 40%+ of users on modern providers use passkeys. They'll be confused by "enter password" instructions.

---

### 5. Make Links Clickable and Explicit

**Rule:** Use `[descriptive text](url)` with `target="_blank"` where applicable.

**❌ Bad:**
```markdown
Go to your security settings
```

**✅ Good:**
```markdown
Go to [Yahoo Account Security](https://account.yahoo.com/security) (opens in new tab)
```

**❌ Bad:**
```markdown
Visit: https://account.yahoo.com/security
```

**✅ Good:**
```markdown
Visit: [account.yahoo.com/security](https://account.yahoo.com/security)
```

**Why it matters:** Users should be able to click directly - reduces copy/paste errors and friction.

---

### 6. Add Troubleshooting Tables

**Rule:** Include "Symptom → Cause → Fix" tables for common issues.

**Format:**
```markdown
## Troubleshooting

| Symptom | Possible Cause | Solution |
|---------|---------------|----------|
| "Authentication failed" | Using regular password instead of app password | Generate and use app-specific password from security settings |
| "App passwords" option missing | 2FA not enabled | Enable Two-Factor Authentication first |
| Password not working | Spaces in password | Remove spaces or paste exactly as shown |
```

**Why it matters:** Users need quick solutions, not full debugging guides. Tables are scannable.

---

### 7. Include Time Estimates

**Rule:** Tell users how long things take.

**❌ Bad:**
```markdown
Test your connection
```

**✅ Good:**
```markdown
Test your connection (takes 10-15 seconds)
```

**❌ Bad:**
```markdown
Complete Gmail setup
```

**✅ Good:**
```markdown
Complete Gmail setup (5 minutes total)
```

**Why it matters:** Sets expectations and reduces abandonment ("This is taking forever!").

---

### 8. Provide Concrete Examples

**Rule:** Show real examples, not placeholders.

**❌ Bad:**
```markdown
Enter your email: [your-email@example.com]
```

**✅ Good:**
```markdown
Enter your email: john.doe@yahoo.com
```

**❌ Bad:**
```markdown
The password will look like: ****************
```

**✅ Good:**
```markdown
The password will look like: abcd efgh ijkl mnop (16 characters with spaces)
```

**Why it matters:** Users pattern-match. Real examples reduce confusion about format.

---

### 9. Use Progressive Disclosure

**Rule:** Provide quick start, then detailed guide. Don't overwhelm.

**Structure:**
```markdown
## Quick Start (2 minutes)
[Essential steps only - no explanation]

## Detailed Guide (5 minutes)
[Full walkthrough with screenshots]

## Advanced Options
[For power users only]
```

**Why it matters:** Power users want speed. Beginners want detail. Serve both.

---

### 10. End with Success Criteria

**Rule:** Tell users how to know they succeeded.

**❌ Bad:**
```markdown
Setup complete!
```

**✅ Good:**
```markdown
## You'll Know It Worked When...

✅ You see "Syncing emails..." in the app  
✅ Your recent emails appear within 30 seconds  
✅ No error messages displayed

**Still seeing errors?** See the Troubleshooting section above.
```

**Why it matters:** Reduces support tickets from users unsure if they succeeded.

---

## 📝 Complete Template

```markdown
# [Provider] Email Setup

**Use this guide when:** [One sentence - when user needs this]  
**Time required:** [X minutes]

---

## Quick Start

**Prerequisites:**
- [ ] [Any requirements]
- [ ] [2FA enabled, etc.]

**Steps:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

---

## Detailed Guide

### Step 1: [Action]

[Detailed explanation]

[Screenshot or example if helpful]

**Tip:** [Pro tip or common mistake]

### Step 2: Generate App Password

**Don't have a password?**  
If you sign in with a passkey (fingerprint/face), you'll need an app-specific password for email apps.

1. Go to [Provider Security Page](https://url.com)
2. Sign in with your passkey
3. Find "Generate app password" or similar
4. Name it: "Your App Name"
5. Copy the password (format: xxxx xxxx xxxx xxxx)

**Important:** This is NOT your regular password. It's a special password just for email apps.

### Step 3: Enter Credentials

1. Email: your.email@provider.com
2. Password: **App-specific password** (the one you just generated)
3. Click "Connect"

---

## Troubleshooting

| Symptom | Possible Cause | Solution |
|---------|---------------|----------|
| [Error message] | [Why it happens] | [How to fix] |
| [Issue 2] | [Cause 2] | [Fix 2] |

### Still Having Issues?

- Check [Provider's Help Center](https://url.com)
- Contact support: support@app.com

---

## You'll Know It Worked When...

✅ [Success indicator 1]  
✅ [Success indicator 2]  
✅ [Success indicator 3]

**Next steps:** [What to do after setup]

---

## Advanced Options

[Optional advanced configuration]

---

_Last updated: [Date] | Questions? [Contact info]_
```

---

## 🎨 Before/After Examples

### Example 1: Password Field Instructions

**❌ Before:**
```markdown
### Enter Password

Enter your password below.

[Password field]
```

**✅ After:**
```markdown
### Enter App-Specific Password

**Using a passkey?** If you sign in with your fingerprint or face instead of a password, you'll need to generate an app-specific password. [See instructions](#generating-app-password)

**Format:** 16 characters (like: abcd efgh ijkl mnop)

[App-Specific Password field] [Show/Hide button]

⚠️ This is NOT your regular Yahoo password - it's a special password for email apps only.
```

### Example 2: Setup Steps

**❌ Before:**
```markdown
Go to your account settings and enable app passwords. Generate one and use it.
```

**✅ After:**
```markdown
1. Go to [Yahoo Account Security](https://account.yahoo.com/security)
2. Sign in with your passkey (or password if you still use one)
3. Scroll to "Generate app password"
4. Click "Generate app password"
5. Select "Other App"
6. Enter name: "My Email App"
7. Click "Generate"
8. Copy the 16-character password
9. Paste it in the password field above

**Time required:** 2-3 minutes
```

### Example 3: Error Messages

**❌ Before:**
```markdown
If it doesn't work, check your credentials.
```

**✅ After:**
```markdown
## Troubleshooting

### "Authentication failed" error

**Possible causes:**
1. **Using wrong password type**  
   Solution: Use the app-specific password (16 chars), not your regular password

2. **Password has spaces**  
   Solution: Remove spaces when pasting (or paste exactly as shown with spaces)

3. **App password expired** (Yahoo, after 6 months inactive)  
   Solution: Generate a new app password

### Can't find "App Passwords" option

**Cause:** Two-Factor Authentication (2FA) not enabled  
**Solution:** Enable 2FA first in your account security settings

### Still not working?

Contact support with:
- Email provider (Yahoo, Gmail, etc.)
- Error message (screenshot helpful)
- Account type (personal vs work)
```

---

## ♿ Accessibility Guidelines

### Screen Readers

**Required:**
- [ ] All images have alt text
- [ ] Links have descriptive text (not "click here")
- [ ] Form labels associated with inputs
- [ ] Error messages use `role="alert"`

**Example:**
```html
<label for="app-password">App-Specific Password</label>
<input 
  id="app-password" 
  type="password" 
  aria-describedby="password-help"
/>
<span id="password-help" class="help-text">
  16-character password from your provider's security settings
</span>
```

### Visual Clarity

**Required:**
- [ ] Text contrast ratio >= 4.5:1
- [ ] Font size >= 14px for body text
- [ ] Links underlined or clearly distinguished
- [ ] Error messages in color + icon + text

---

## 🌍 Internationalization Considerations

### Language-Neutral Examples

**❌ Bad:**
```markdown
Your email will be: john.doe@yahoo.com
```

**✅ Good:**
```markdown
Your email format: yourname@yahoo.com
```

### Universal Concepts

**Avoid region-specific references:**
- ❌ "Like your SSN, but for email"
- ✅ "A unique identifier for your account"

### Translatable Structure

**Keep translation in mind:**
```markdown
<!-- Good: Short, self-contained sentences -->
Generate an app password. It takes 2 minutes.

<!-- Bad: Complex sentence structure -->
You'll need to, if you haven't already done so previously, generate what's called an app-specific password by visiting your security settings page.
```

---

## 📊 Quality Gates Checklist

**Before publishing, verify:**

### Content
- [ ] Numbered steps (no unnumbered lists for procedures)
- [ ] Max 10 main steps (break into sub-steps if needed)
- [ ] Passkeys mentioned if relevant provider
- [ ] App vs regular password distinction clear
- [ ] Time estimate provided

### Links & Resources
- [ ] All links clickable (not plain text URLs)
- [ ] Links open in new tab where appropriate
- [ ] Provider security URLs direct (not homepage)
- [ ] Screenshots show current UI (not outdated)

### User Experience
- [ ] Success criteria clearly stated
- [ ] At least 3 troubleshooting entries
- [ ] Concrete examples (not placeholders)
- [ ] Progressive disclosure (quick start + detailed)
- [ ] "Don't have a password?" addressed

### Accessibility
- [ ] Images have alt text
- [ ] Links have descriptive text
- [ ] Form labels present
- [ ] Color not sole indicator

### Technical Accuracy
- [ ] IMAP/SMTP settings correct
- [ ] Port numbers accurate
- [ ] Security requirements current
- [ ] Tested on real accounts

---

## 🚫 Common Mistakes to Avoid

### Mistake 1: Assuming Users Have Passwords

**❌ Don't assume:**
```markdown
Enter your password
```

**✅ Acknowledge reality:**
```markdown
Enter your app-specific password (or generate one if you use passkeys)
```

### Mistake 2: Using Technical Jargon

**❌ Technical:**
```markdown
Configure your MUA to authenticate via SMTP AUTH using STARTTLS on port 587
```

**✅ Plain language:**
```markdown
Set up your email app to send mail securely through Yahoo's servers
```

### Mistake 3: Vague Error Messages

**❌ Vague:**
```markdown
Something went wrong. Try again.
```

**✅ Specific:**
```markdown
Connection failed: Check your password
- Using app-specific password? (not regular password)
- Remove any spaces when pasting
- [Generate new password →]
```

### Mistake 4: No Visual Aids

**❌ Text only:**
```markdown
Click the gear icon, then settings, then security, then app passwords
```

**✅ With visual:**
```markdown
1. Click the ⚙️ gear icon (top right)
2. Select "Settings"
3. Go to "Security" tab
4. Find "App Passwords" section

[Screenshot showing the security page with App Passwords highlighted]
```

### Mistake 5: Forgetting Mobile Users

**❌ Desktop-only:**
```markdown
Right-click and select "Copy"
```

**✅ Cross-platform:**
```markdown
Copy the password (long-press on mobile, right-click on desktop)
```

---

## 📱 Mobile-Specific Considerations

### Touch Targets

**Minimum size:** 48x48px for interactive elements

```css
/* Ensure buttons are tappable */
.copy-button {
  min-width: 48px;
  min-height: 48px;
  padding: 12px 24px;
}
```

### Keyboard Behavior

```html
<!-- Auto-focus on mobile -->
<input 
  type="email" 
  inputmode="email"
  autocomplete="email"
/>

<!-- Numeric for verification codes -->
<input 
  type="text" 
  inputmode="numeric" 
  pattern="[0-9]*"
/>
```

### Responsive Images

```markdown
![App password screen](image.png)
_Tap image to enlarge_
```

---

## 🔗 Related Arsenal Items

**💭 Memory:**
- [Email Passkey Authentication](https://github.com/ChrisTansey007/windsurf-memories-arsenal/blob/main/development-tools/email-passkey-auth.md)

**📝 Prompts:**
- [Email Provider Preset Generator](https://github.com/ChrisTansey007/prompt-arsenal/blob/main/development/email/generate-provider-preset.md)

**🔄 Workflows:**
- [OAuth Email Debug](https://github.com/ChrisTansey007/ai-workflows-arsenal/blob/main/windsurf/debugging/oauth-email-debug.md)

**🔗 Example:**
- [Email Integration Example](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/email-integration)

---

## 📚 Resources

**Writing Style Guides:**
- [Google Developer Documentation Style Guide](https://developers.google.com/style)
- [Microsoft Writing Style Guide](https://docs.microsoft.com/en-us/style-guide/welcome/)
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)

**Accessibility:**
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [WebAIM Checklist](https://webaim.org/standards/wcag/checklist)

**Email Standards:**
- [RFC 3501: IMAP](https://tools.ietf.org/html/rfc3501)
- [RFC 5321: SMTP](https://tools.ietf.org/html/rfc5321)

---

**Result: Email setup docs that users actually understand and successfully complete!** 📖
