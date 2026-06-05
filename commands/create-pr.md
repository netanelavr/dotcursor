Commit changes and create a PR with proper formatting.

## Flow

### 1. Get Ticket Number

Extract the ticket number from the branch name:
```bash
git rev-parse --abbrev-ref HEAD
```
If the ticket number isn't in the branch name, ask the user for it.

### 2. Commit Changes

**A. Check git status:**
```bash
git status
```

**B. Stage changes:**
```bash
git add -A
```
Or let the user pick specific files to stage.

**C. Commit with ticket number in message:**
```bash
git commit -m "[TICKET] Fix: <problem users experience>"
```

**Commit message format:**
- Bug fix: `[TICKET] Fix: <problem users experience>`
- Feature: `[TICKET] Feature: <what users can now do>`
- Improvement: `[TICKET] Improve: <what's better for users>`
- Refactor: `[TICKET] Refactor: <description>`

**Important:** 
- Describe the PROBLEM, not your solution
- Ask: "What did users experience that was wrong?"

### 3. Create Pull Request

**A. Push the branch:**
```bash
git push -u origin HEAD
```

**B. Create PR using gh CLI:**
```bash
gh pr create \
  --title "[TICKET] Fix: <problem users experience>" \
  --body "## Problem
<What users experienced - why this matters to them>

## Solution
<Brief explanation of how it's fixed>"
```

## Best Practices

### Commit Message
- **Format:** `[TICKET] Type: Description`
- **Types:** Fix, Feature, Improve, Refactor
- **Description:** Describe the problem being fixed, not the code change
- **Length:** One line, < 72 chars if possible
- **Tense:** Imperative mood (Fix, Add, Update - not Fixed, Added, Updated)
- Bad: "Add null check to user handler" (what you did)
- Good: "Users see crash when opening empty profile" (what users experienced)

### PR Description
- **Problem:** What the user experienced (lead with this, make it clear why it matters)
- **Solution:** Brief explanation of how it's fixed (keep technical details minimal)
- Skip "Root Cause" and "Changes" sections - focus on user impact

### Branch Naming
- Use descriptive lowercase names with hyphens
- Examples: `fix-login-redirect`, `feature-chat-export`, `improve-loading-speed`

## Troubleshooting

### Finding Current Branch PR
```bash
gh pr view --json number,url,title
```

## Example Complete Flow

```
1. Branch: fix-new-chat-reload → Ticket: PROJ-5674

2. Commit:
   git commit -m "[PROJ-5674] Fix: New Chat button returns to previous chat after reload"
   
3. Push:
   git push -u origin fix-new-chat-reload

4. Create PR:
   gh pr create \
     --title "[PROJ-5674] Fix: New Chat button returns to previous chat after reload" \
     --body "## Problem
   Users click New Chat expecting a fresh conversation, but after page reload they see the previous chat instead.
   
   ## Solution
   Clear chat state properly when starting a new conversation."
   
   → Result: PR #1780
```

## Output Format
After completion, provide user with (DO NOT wrap in code block - URLs must be clickable):

✅ **Commit**: `[TICKET] <Type>: <Description>`

✅ **PR**: [#NUMBER - <Title>](PR_URL)
