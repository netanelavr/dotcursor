Fetch all PR comments, classify by severity, and present for user approval before acting.

## Input
`triage pr comments [PR_URL_OR_NUMBER]` (optional)
- Full URL: `https://github.com/your-org/your-repo/pull/123`
- Number only: `123`
- **No input**: Auto-detects PR from current branch

## Flow

### 1. Auto-detect PR if not provided
```bash
PR_NUM=$(gh pr view --json number --jq '.number')
OWNER=$(gh pr view --json headRepositoryOwner --jq '.headRepositoryOwner.login')
REPO_NAME=$(gh pr view --json headRepository --jq '.headRepository.name')
REPO="$OWNER/$REPO_NAME"
```

### 2. Extract ALL 3 comment types
```bash
# A. General PR discussion comments
gh api repos/$REPO/issues/$PR_NUM/comments

# B. Line-specific review comments (most important)
gh api repos/$REPO/pulls/$PR_NUM/comments

# C. Overall review submissions
gh api repos/$REPO/pulls/$PR_NUM/reviews
```

### 3. Filter out resolved/outdated threads
```bash
gh api graphql -f query='query { repository(owner: "'$OWNER'", name: "'$REPO_NAME'") { pullRequest(number: '$PR_NUM') { reviewThreads(first: 100) { nodes { id isResolved isOutdated comments(first: 10) { nodes { id body author { login } }}}}}}}'
```
Skip threads where `isResolved: true` or `isOutdated: true`.

### 4. Classify each comment

For every unresolved comment, assign:

**Severity:**
- **Critical** — Bug, security flaw, data loss risk, broken logic
- **Important** — Type safety, missing error handling, performance issue, code smell
- **Minor** — Naming, style, readability, minor improvement suggestion
- **Noise** — Cosmetic, subjective preference, over-engineering suggestion, bot boilerplate

**Action Recommendation:**
- **Must Fix** — Will cause real issues if ignored
- **Should Fix** — Clear improvement, low effort
- **Optional** — Nice-to-have, debatable value
- **Skip** — Not worth the effort or disagree with suggestion

### 5. Present to user for approval

Display a structured list grouped by severity. For each comment include:

- **#** — Sequential number for easy reference
- **Severity** — Critical / Important / Minor / Noise
- **Recommendation** — Must Fix / Should Fix / Optional / Skip
- **Author** — Who wrote it (human vs bot matters)
- **File:Line** — Where it applies (if line-specific)
- **Comment** — The original comment text (quote it)
- **Proposed Solution** — Concrete description of what you would change to address it
- **Reasoning** — Why you classified it this way

Format example:
```
### Critical (X comments)

**#1** | Must Fix | @reviewer | `src/foo.ts:42`
> Original comment: "This will crash if user is null — need a guard here"

**Proposed Solution:** Add a null check for `user` before accessing `user.id` on line 42. Return early with a default value if null.

**Reasoning:** Unhandled edge case in a production code path — will cause runtime crash.

---

### Important (X comments)
...

### Minor (X comments)
...

### Noise (X comments)
...
```

After the table, ask the user:
> Which comments should I fix? You can reply with:
> - Numbers: `1, 3, 5` — fix specific comments
> - Ranges: `1-5` — fix a range
> - Severity: `all critical`, `all important` — fix by category
> - `all` — fix everything recommended (Must Fix + Should Fix)
> - `none` — skip all

### 6. Implement approved fixes

After user confirms:
- Implement only the approved fixes
- Follow workspace coding standards
- Run lint check after changes

### 7. Resolve addressed threads
```bash
gh api graphql -f query='mutation { resolveReviewThread(input: {threadId: "<ID>"}) { thread { isResolved }}}'
```

### 8. Wait for user to commit and push

Before posting to the PR, **pause and ask the user to confirm** they have committed and pushed the changes:

> Changes are ready. Please commit and push when ready, then confirm here so I can post the summary to the PR.

**Do NOT proceed to step 9 until the user explicitly confirms.**

### 9. Post summary to PR
```bash
cat << 'EOF' | gh pr comment $PR_NUM --body-file -
Addressed X/Y comments from review

**Fixed:**
- <list of fixed items with comment references>

**Skipped (with reasons):**
- <list of skipped items>

EOF
```

**Important:** Use `--body-file -` with heredoc for reliable posting of complex markdown

## Classification Guidelines

**Human comments always rank higher** — a human took time to write it, treat with respect.

**Bot comments context:**
- Bot "Critical/Error" → Treat as Important (verify before upgrading to Critical)
- Bot test suggestions → Minor unless covering a critical path
- Bot style/docs suggestions → Noise

**When in doubt, ask yourself:**
- "Will this cause a bug in production?" → Critical
- "Will this confuse the next developer?" → Important
- "Is this just preference?" → Minor or Noise

## Important Notes
- **Do NOT start fixing before user approval** — this is a triage command
- **Auth fallback**: If gh commands fail, run `gh auth status` to diagnose
- Filter out resolved/outdated threads before classifying

## Output
After approval and fixes: "Fixed X/Y approved comments | Lint clean | PR updated"
