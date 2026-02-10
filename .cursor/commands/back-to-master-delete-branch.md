# Back to Default Branch & Delete Branch

Cleanup current branch and switch to default branch with latest changes.

## Flow

### 1. Get Current Branch Name
```bash
git rev-parse --abbrev-ref HEAD
```
Store this for deletion later.

### 2. Detect Default Branch
```bash
DEFAULT_BRANCH=$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')
```

### 3. Fetch & Switch to Default Branch
```bash
git checkout $DEFAULT_BRANCH && git pull origin $DEFAULT_BRANCH
```

### 4. Delete the Previous Branch (Local Only)
```bash
git branch -d <branch-name>
```
Use `-D` if branch wasn't merged (force delete).

## Output Format
After completion:
- ✅ Switched to `<default-branch>` (pulled latest)
- 🗑️ Deleted local branch: `<branch-name>`
