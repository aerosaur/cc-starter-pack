# Git Tasks

Git safety protocols and commit workflows. **Claude MUST follow these rules.**

## Safety Rules (NON-NEGOTIABLE)

**NEVER do these without EXPLICIT user request:**
- `git push --force` (especially to main/master)
- `git reset --hard`
- `git checkout .` or `git restore .`
- `git clean -f`
- `git branch -D` (force delete)
- `git rebase -i` (interactive rebase)
- Skip hooks (`--no-verify`)

**ALWAYS do these:**
- `git status` before committing
- Stage specific files (not `git add .` or `git add -A`)
- Check `git log -1` before amending
- Create NEW commits after hook failures (don't amend)

---

## Commit Workflow

### 1. Review Changes
```bash
git status          # See what's changed
git diff            # See unstaged changes
git diff --staged   # See staged changes
```

### 2. Stage Files
```bash
git add path/to/specific/file.js   # Stage specific files
# AVOID: git add . or git add -A (can include secrets)
```

### 3. Draft Commit Message
- Summarize the "why" not the "what"
- Use imperative mood ("Add feature" not "Added feature")
- Keep first line under 72 characters
- Add Co-Authored-By line

### 4. Commit
```bash
git commit -m "$(cat <<'EOF'
Brief description of what changed and why

More details if needed.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 5. Verify
```bash
git status          # Should show clean working tree
git log -1          # Verify commit looks correct
```

---

## Pre-Commit Hook Failures

**If a pre-commit hook fails:**

1. The commit did NOT happen
2. Fix the issue (lint errors, format, etc.)
3. Re-stage the fixed files
4. Create a NEW commit (do NOT use --amend)

**Why?** `--amend` after hook failure modifies the PREVIOUS commit, potentially destroying work.

---

## Creating Pull Requests

### 1. Review Branch State
```bash
git status
git log origin/main..HEAD    # Commits to be included
git diff origin/main...HEAD  # All changes
```

### 2. Push Branch
```bash
git push -u origin branch-name
```

### 3. Create PR
```bash
gh pr create --title "Title" --body "$(cat <<'EOF'
## Summary
- Bullet points describing changes

## Test plan
- [ ] How to test this PR

Generated with Claude Code
EOF
)"
```

---

## Commit Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting (no code change)
- `refactor`: Code restructure (no behavior change)
- `test`: Adding tests
- `chore`: Maintenance tasks

---

## Sensitive Files - NEVER COMMIT

- `.env` files
- `credentials.json`
- API keys or tokens
- Private keys
- Database dumps
- Anything in `.gitignore`

**If accidentally staged:**
```bash
git reset HEAD -- path/to/sensitive/file
```

---

## Amending (Only When Explicitly Asked)

```bash
# ONLY if user explicitly requests amending
git add changed-file.js
git commit --amend --no-edit
```

**Warning user first:**
"I'll amend the last commit. This will modify commit [hash]. Are you sure?"
