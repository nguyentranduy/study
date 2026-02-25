# Best Practices - Thực hành tốt nhất

Tổng hợp các best practices khi làm việc với Git để code clean, dễ maintain, và team collaboration hiệu quả.

## 1. Commit Messages

### Conventional Commits

Theo chuẩn [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types phổ biến

```
feat:     Tính năng mới
fix:      Sửa bug
docs:     Thay đổi documentation
style:    Format code (không ảnh hưởng logic)
refactor: Refactor code
test:     Thêm/sửa tests
chore:    Maintenance tasks (build, dependencies...)
perf:     Cải thiện performance
ci:       Thay đổi CI/CD
revert:   Revert commit trước
```

### Ví dụ Good Commits

```bash
# Feature
git commit -m "feat(auth): add OAuth2 Google login

- Implement OAuth2 flow
- Add Google sign-in button
- Store JWT token in localStorage

Closes #123"

# Bug fix
git commit -m "fix(payment): resolve null pointer in checkout

When user has no saved payment method, checkout crashes.
Added null check before accessing payment object.

Fixes #456"

# Breaking change
git commit -m "feat(api)!: change user endpoint response format

BREAKING CHANGE: /api/users now returns {users: [...]} instead of [...]

This provides consistency with other endpoints.

Related to #789"
```

### ❌ Bad Commits vs ✅ Good Commits

```bash
# ❌ BAD
git commit -m "fix"
git commit -m "update"
git commit -m "changes"
git commit -m "asdfasdf"
git commit -m "Fixed the thing"

# ✅ GOOD
git commit -m "fix(login): resolve token expiration handling"
git commit -m "docs(readme): update installation steps"
git commit -m "refactor(user): extract validation logic"
git commit -m "test(auth): add login integration tests"
```

### Commit Message Template

Tạo template:

```bash
# ~/.gitmessage
# <type>(<scope>): <subject>

# <body>

# <footer>

# Types: feat, fix, docs, style, refactor, test, chore
# Scope: module/component name
# Subject: imperative mood, no period, < 50 chars
# Body: explain what and why (not how), wrap at 72 chars
# Footer: issues, breaking changes
```

Config:
```bash
git config --global commit.template ~/.gitmessage
```

## 2. Branching Strategy

### Branch Naming Convention

```bash
# Features
feature/user-authentication
feature/US-123-shopping-cart
feat/payment-integration

# Bug fixes
fix/login-button-not-working
bugfix/JIRA-456-memory-leak
fix/null-pointer-in-checkout

# Hotfixes
hotfix/critical-security-patch
hotfix/PROD-789-payment-error

# Releases
release/v1.2.0
release/2024-02-25

# Documentation
docs/update-api-guide
docs/add-deployment-steps

# Refactoring
refactor/optimize-queries
refactor/clean-up-user-service

# Experiments
experiment/new-ui-design
poc/blockchain-integration
spike/performance-testing
```

### Pattern

```
<type>/<ticket-id>-<short-description>
<type>/<short-description>

Examples:
feature/JIRA-123-user-profile
fix/github-456-login-error
hotfix/production-payment-crash
```

### Branch Lifecycle

```bash
# 1. Create from latest main
git checkout main
git pull origin main
git checkout -b feature/new-feature

# 2. Regular commits
git add .
git commit -m "feat: implement core logic"

# 3. Keep updated with main
git fetch origin
git rebase origin/main  # or merge

# 4. Push regularly
git push -u origin feature/new-feature

# 5. Create PR/MR when ready

# 6. After merge, cleanup
git checkout main
git pull origin main
git branch -d feature/new-feature
```

## 3. Small, Focused Commits

### Commit Often, Commit Small

❌ **BAD**: 1 giant commit
```bash
git commit -m "feat: add entire user management system"
# 50 files changed, 5000+ lines
```

✅ **GOOD**: Multiple small commits
```bash
git commit -m "feat(user): add User model"
git commit -m "feat(user): add UserRepository"
git commit -m "feat(user): add UserService with CRUD"
git commit -m "feat(user): add UserController endpoints"
git commit -m "test(user): add UserService tests"
```

### Atomic Commits

Mỗi commit = 1 đơn vị logic hoàn chỉnh:

```bash
# ✅ Each commit compiles and tests pass
git commit -m "feat: add User model"        # Compiles ✓
git commit -m "feat: add User validation"   # Compiles ✓
git commit -m "test: add User tests"        # Tests pass ✓

# ❌ Broken commits
git commit -m "feat: add half of User model"     # Won't compile ✗
git commit -m "feat: finish User model"          # Finally compiles ✓
```

### Partial Staging

```bash
# Stage specific parts of files
git add -p filename.java

# Git asks for each change:
# y - stage this hunk
# n - don't stage
# s - split into smaller hunks
# e - manually edit
```

## 4. Pull Request / Merge Request Best Practices

### PR Size

```
Small PR:    < 200 lines     ✅ Easy to review
Medium PR:   200-400 lines   ⚠️  Acceptable
Large PR:    > 400 lines     ❌ Split into smaller PRs
```

### PR Template

```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] 🚀 New feature (non-breaking change which adds functionality)
- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)
- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] 📝 Documentation update
- [ ] 🎨 Style/refactor (no functional changes)

## Related Issues
Closes #123
Related to #456

## How Has This Been Tested?
- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing on dev environment
- [ ] Tested on staging

## Screenshots (if applicable)
Before:
![before](url)

After:
![after](url)

## Checklist
- [ ] My code follows the style guidelines
- [ ] I have performed a self-review
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
- [ ] Any dependent changes have been merged and published

## Deployment Notes
Any special deployment requirements or database migrations?
```

### PR Description Best Practices

```markdown
# ❌ BAD
"Fixed bug"

# ✅ GOOD
## Problem
Users cannot reset password when email contains uppercase letters.

## Root Cause
Email comparison is case-sensitive in `UserService.findByEmail()`.

## Solution
Convert email to lowercase before database query.

## Testing
- Added unit tests for case-insensitive email lookup
- Tested manually with emails: Test@Example.com, TEST@example.com
- All existing tests pass

Fixes #456
```

## 5. Code Review Guidelines

### As Author

```bash
# Before creating PR:
# 1. Self-review your code
git diff main...feature/my-feature

# 2. Run tests locally
npm test

# 3. Run linter
npm run lint

# 4. Check for console.logs, debuggers
grep -r "console.log" src/
grep -r "debugger" src/

# 5. Update documentation
```

### As Reviewer

**Checklist:**
- [ ] **Functionality**: Code does what it says
- [ ] **Tests**: Adequate test coverage
- [ ] **Design**: Well-structured, follows patterns
- [ ] **Naming**: Clear variable/function names
- [ ] **Comments**: Complex logic is commented
- [ ] **Performance**: No obvious bottlenecks
- [ ] **Security**: No vulnerabilities
- [ ] **Error handling**: Proper exception handling
- [ ] **Documentation**: README/docs updated

**Review Comments:**
```markdown
# ❌ BAD
"This is wrong"
"Change this"

# ✅ GOOD
"Consider using a Set instead of Array here for O(1) lookup instead of O(n)"

"This might cause a race condition if two requests come simultaneously. 
Suggest adding a lock or using a transaction."

"Great approach! Minor suggestion: we could extract this into a separate 
function for reusability. What do you think?"
```

## 6. .gitignore Best Practices

### Comprehensive .gitignore

```gitignore
# ===== Language-specific =====

# Java
*.class
*.jar
*.war
target/
.mvn/
*.iml

# Node.js
node_modules/
npm-debug.log*
yarn-error.log*
.pnpm-debug.log*

# Python
__pycache__/
*.py[cod]
venv/
.Python

# ===== IDEs =====

# IntelliJ IDEA
.idea/
*.iml
*.iws

# VS Code
.vscode/
*.code-workspace

# Eclipse
.classpath
.project
.settings/

# ===== OS =====

# macOS
.DS_Store
.AppleDouble
.LSOverride

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini

# Linux
*~
.directory

# ===== Environment & Config =====

# Environment variables
.env
.env.local
.env.*.local

# Secrets
secrets/
*.pem
*.key
credentials.json

# ===== Build outputs =====

# General
dist/
build/
out/
*.log

# ===== Databases =====

# SQLite
*.db
*.sqlite
*.sqlite3

# ===== Temporary files =====

*.tmp
*.swp
*.swo
*~
.#*
```

### .gitignore Tips

```bash
# Check if file is ignored
git check-ignore -v filename.txt

# Force add ignored file (if really needed)
git add -f ignored-but-needed.txt

# Remove already tracked file
git rm --cached accidental-commit.txt
echo "accidental-commit.txt" >> .gitignore
git commit -m "chore: add to .gitignore"

# Global gitignore for all repos
git config --global core.excludesfile ~/.gitignore_global
```

## 7. Merge vs Rebase

### When to Merge

```bash
# ✅ Use merge for:
# - Bringing changes FROM public branches (main, develop)
# - Preserving complete history
# - Safety first approach

git checkout feature/my-feature
git merge main
```

### When to Rebase

```bash
# ✅ Use rebase for:
# - Updating private feature branches
# - Cleaning up commits before PR
# - Linear history

git checkout feature/my-feature
git rebase main

# Interactive rebase to clean up
git rebase -i main
```

### Golden Rule

!!! danger "Important"
    **NEVER rebase commits that have been pushed to public/shared branches!**
    
    ✅ OK: `git rebase` on your feature branch  
    ❌ NEVER: `git rebase` on main/develop/shared branches

## 8. Git Hooks

### Pre-commit Hook

```bash
# .git/hooks/pre-commit
#!/bin/sh

echo "Running pre-commit checks..."

# Run linter
npm run lint
if [ $? -ne 0 ]; then
  echo "❌ Linting failed. Please fix errors."
  exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
  echo "❌ Tests failed. Please fix tests."
  exit 1
fi

echo "✅ Pre-commit checks passed!"
```

### Commit-msg Hook

```bash
# .git/hooks/commit-msg
#!/bin/sh

commit_msg_file=$1
commit_msg=$(cat "$commit_msg_file")

# Check commit message format
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"; then
  echo "❌ Invalid commit message format!"
  echo "Format: <type>(<scope>): <subject>"
  echo "Example: feat(auth): add login button"
  exit 1
fi

echo "✅ Commit message format is valid"
```

### Husky (easier hook management)

```bash
# Install husky
npm install --save-dev husky

# Initialize
npx husky install

# Add hooks
npx husky add .husky/pre-commit "npm test"
npx husky add .husky/commit-msg "npx commitlint --edit $1"
```

## 9. Repository Structure

### README.md Best Practices

```markdown
# Project Name

Brief description of what the project does.

## Features
- Feature 1
- Feature 2
- Feature 3

## Prerequisites
- Node.js 18+
- PostgreSQL 14+
- Redis (optional)

## Installation

\```bash
git clone https://github.com/user/project.git
cd project
npm install
cp .env.example .env  # Configure environment
\```

## Usage

\```bash
# Development
npm run dev

# Production
npm run build
npm start
\```

## Testing

\```bash
npm test              # Run all tests
npm run test:watch    # Watch mode
npm run test:coverage # Coverage report
\```

## Project Structure

\```
src/
├── controllers/   # Route controllers
├── models/        # Database models
├── services/      # Business logic
├── utils/         # Helper functions
└── config/        # Configuration files
\```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## License

MIT
```

### CONTRIBUTING.md

```markdown
# Contributing

## How to Contribute

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Commit Message Guidelines

Follow [Conventional Commits](https://www.conventionalcommits.org/)

## Code Style

- Run `npm run lint` before committing
- Follow existing code style
- Add tests for new features

## Pull Request Process

1. Update README.md with any new dependencies or env variables
2. Ensure all tests pass
3. Request review from maintainers
4. Squash commits before merge (if requested)
```

## 10. Security Best Practices

### Never Commit Secrets

```bash
# ❌ NEVER commit:
- API keys
- Passwords
- Private keys (.pem, .key)
- Database credentials
- OAuth secrets
- JWT secrets

# ✅ Use instead:
- .env files (in .gitignore)
- Environment variables
- Secret management services (AWS Secrets Manager, Vault)
```

### Check for Secrets

```bash
# Use git-secrets
git secrets --scan

# Use trufflehog
trufflehog git file://path/to/repo

# Manual check
git log -p | grep -i "password\|api_key\|secret"
```

### Remove Committed Secrets

```bash
# If you accidentally committed a secret:

# 1. Remove from latest commit
git rm --cached secrets.txt
git commit --amend

# 2. Remove from history (use BFG)
java -jar bfg.jar --delete-files secrets.txt
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 3. IMPORTANT: Rotate the compromised secret!
#    Even after removing from Git, assume it's compromised
```

## 11. Collaboration Tips

### Communication

```bash
# Good commit messages ARE communication
# Others should understand changes without asking

# Link to issues/tickets
git commit -m "feat: add payment gateway

Implements Stripe integration for credit card payments.

Closes #123
See also #124"

# Explain WHY, not just WHAT
git commit -m "perf: cache user queries

Users table is queried frequently but rarely updated.
Adding 5-minute cache reduces database load by 60%.

Related to #456"
```

### Code Ownership

```
# CODEOWNERS file
# Auto-request reviews from owners

# Backend
/backend/**               @backend-team
/backend/auth/**          @auth-team

# Frontend
/frontend/**              @frontend-team
/frontend/components/**   @ui-team

# Infrastructure
/infrastructure/**        @devops-team @security-team
*.yml                     @devops-team

# Database
/migrations/**            @dba-team
```

## 12. Performance & Maintenance

### Keep Repository Clean

```bash
# Remove old branches
git remote prune origin
git branch --merged | grep -v main | xargs git branch -d

# Garbage collection
git gc --aggressive --prune=now

# Check repository size
du -sh .git

# Find large files
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  sort -k3 -n -r | head -20
```

### Git LFS for Large Files

```bash
# Install Git LFS
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "*.zip"

git add .gitattributes
git commit -m "chore: setup Git LFS"
```

## Checklist

- [ ] Tuân theo Conventional Commits
- [ ] Commit messages rõ ràng, có body khi cần
- [ ] Branches có naming convention
- [ ] PRs nhỏ, focused (< 400 lines)
- [ ] Self-review trước khi tạo PR
- [ ] .gitignore comprehensive
- [ ] Không commit secrets
- [ ] Setup pre-commit hooks
- [ ] README.md đầy đủ
- [ ] Link commits với issues/tickets

## Tài liệu tham khảo

- **[Conventional Commits](https://www.conventionalcommits.org/)** - Commit message standard
- **[Git Best Practices](https://sethrobertson.github.io/GitBestPractices/)** - Comprehensive guide
- **[Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials/learn-git-with-bitbucket-cloud)** - Best practices từ Atlassian
- **[GitHub Flow Guide](https://docs.github.com/en/get-started/quickstart/github-flow)** - GitHub's workflow
