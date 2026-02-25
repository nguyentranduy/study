# GitHub Flow - Simple & Effective

GitHub Flow là một workflow đơn giản, nhẹ nhàng được thiết kế cho Continuous Deployment. Được GitHub phát triển và sử dụng nội bộ.

## Tổng quan GitHub Flow

<img src="https://lucamezzalira.files.wordpress.com/2014/03/screen-shot-2014-03-08-at-23-07-361.png" alt="GitHub Flow" width="600">

*GitHub Flow workflow*

### Nguyên tắc cốt lõi

!!! tip "Quy tắc vàng"
    - ✅ **Main branch luôn deployable** - Có thể deploy bất cứ lúc nào
    - ✅ **Feature branches** - Mỗi task một branch riêng
    - ✅ **Pull Requests** - Review code trước khi merge
    - ✅ **Deploy ngay** - Sau khi merge là deploy luôn
    - ✅ **Đơn giản** - Ít branches, ít rules

## Khi nào nên dùng GitHub Flow?

### ✅ Phù hợp với

- **Continuous Deployment**: Deploy nhiều lần/ngày
- **Web applications**: SaaS products, web services
- **Small teams**: 2-10 developers
- **Startups**: Cần move nhanh
- **Microservices**: Mỗi service một repo
- **Open source**: Dễ contribute

### ❌ Không phù hợp với

- **Multiple versions**: Cần maintain nhiều versions (dùng Git Flow)
- **Scheduled releases**: Release theo lịch cố định
- **Mobile apps**: Không thể deploy tùy ý
- **Enterprise**: Cần approval process phức tạp

## Workflow chi tiết

### Bước 1: Create a Branch

Tạo branch từ `main` cho mỗi task:

```bash
# Pull code mới nhất
git checkout main
git pull origin main

# Tạo feature branch
git checkout -b feature/add-comment-system

# Hoặc bugfix
git checkout -b fix/user-profile-crash
```

**Branch naming:**
```
feature/descriptive-name
fix/bug-description
hotfix/critical-issue
docs/update-readme
refactor/improve-performance
```

### Bước 2: Add Commits

Commit thường xuyên với messages rõ ràng:

```bash
# Làm việc và commit
git add .
git commit -m "feat: add comment model and API"

# Continue working
git add .
git commit -m "feat: add comment UI component"

# Push lên remote
git push -u origin feature/add-comment-system
```

!!! tip "Commit Often"
    - Commit mỗi khi hoàn thành một đơn vị công việc nhỏ
    - Push thường xuyên để backup code
    - Không sợ commit nhiều - có thể squash sau

### Bước 3: Open a Pull Request

Tạo Pull Request ngay khi sẵn sàng (hoặc sớm để discuss):

**Trên GitHub:**
1. Vào repository → Pull Requests → New Pull Request
2. Choose base: `main` ← compare: `feature/add-comment-system`
3. Fill in title và description
4. Add reviewers
5. Create Pull Request

**PR Template tốt:**
```markdown
## What does this PR do?
Adds comment system allowing users to comment on posts.

## Type of change
- [x] New feature
- [ ] Bug fix
- [ ] Breaking change
- [ ] Documentation update

## How to test?
1. Go to any post page
2. Scroll to bottom
3. Try adding a comment
4. Comment should appear in the list

## Screenshots (if applicable)
![Comment UI](https://example.com/screenshot.png)

## Checklist
- [x] Code compiles
- [x] Tests pass
- [x] Documentation updated
- [x] No breaking changes
```

### Bước 4: Discuss and Review

Team members review code và discuss:

```bash
# Reviewer comments trên GitHub
# Developer fix theo feedback

# Make changes
git add .
git commit -m "fix: address review comments"
git push
```

**Review checklist:**
- [ ] Code logic đúng
- [ ] Tests đầy đủ
- [ ] Coding standards
- [ ] No security issues
- [ ] Performance OK
- [ ] Documentation updated

### Bước 5: Deploy và Test

Deploy branch lên staging/preview environment:

```bash
# Nhiều teams deploy PR branch lên staging tự động
# Hoặc manual deploy:
git checkout feature/add-comment-system
./deploy-to-staging.sh
```

**Testing:**
- Functional testing
- Integration testing
- User acceptance testing (nếu cần)

### Bước 6: Merge

Sau khi approved và tests pass:

```bash
# Trên GitHub: Click "Merge pull request"
# Hoặc command line:
git checkout main
git merge --no-ff feature/add-comment-system
git push origin main

# Xóa feature branch
git branch -d feature/add-comment-system
git push origin --delete feature/add-comment-system
```

**Merge options:**
- **Merge commit**: Giữ toàn bộ commits
- **Squash and merge**: Gộp thành 1 commit
- **Rebase and merge**: Rebase lên main

### Bước 7: Deploy to Production

Deploy `main` lên production ngay:

```bash
# Auto deployment (recommended)
# CI/CD tự động deploy khi merge vào main

# Manual deployment
git checkout main
git pull origin main
./deploy-to-production.sh
```

## Ví dụ thực tế

### Workflow hoàn chỉnh

```bash
# Day 1: Bắt đầu feature mới
git checkout main
git pull origin main
git checkout -b feature/user-notifications

# Code feature
echo "notification service" > notifications.js
git add .
git commit -m "feat: add notification service"
git push -u origin feature/user-notifications

# Tạo PR trên GitHub
# Title: "Add user notification system"
# Request reviews từ @alice và @bob

# Day 2: Nhận feedback
# Alice comment: "Should add error handling"

# Fix theo feedback
echo "with error handling" >> notifications.js
git add .
git commit -m "fix: add error handling to notification service"
git push

# Bob approve PR
# CI/CD runs tests → ✅ Pass

# Merge PR
# → Main branch updated
# → Auto deploy to production
# → Notification sent to team

# Cleanup
git checkout main
git pull origin main
git branch -d feature/user-notifications
```

### Xử lý Hotfix

```bash
# Phát hiện bug critical trên production
git checkout main
git pull origin main
git checkout -b hotfix/fix-payment-error

# Fix bug nhanh
# ... fix code ...
git add .
git commit -m "fix: resolve payment processing error"
git push -u origin hotfix/fix-payment-error

# Tạo PR với label [HOTFIX]
# Request urgent review
# Merge ASAP
# Deploy immediately
```

## GitHub Features hỗ trợ

### 1. Draft Pull Requests

Tạo PR sớm để discuss:

```bash
# Tạo Draft PR khi chưa ready
# Team có thể comment early
# Convert to "Ready for review" khi xong
```

### 2. Code Review Tools

- **Line comments**: Comment trực tiếp trên dòng code
- **Suggestions**: Suggest changes trực tiếp
- **Review status**: Approve, Request changes, Comment
- **Review teams**: Tag cả team để review

### 3. GitHub Actions

Auto-deploy khi merge:

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Deploy
        run: |
          echo "Deploying to production..."
          ./deploy.sh
```

### 4. Branch Protection Rules

Setup trên GitHub Settings:

```yaml
Protection Rules for 'main':
  ✅ Require pull request before merging
  ✅ Require approvals (1-2)
  ✅ Dismiss stale reviews when new commits pushed
  ✅ Require status checks (CI/CD)
  ✅ Require branches to be up to date
  ✅ Include administrators
  ❌ Allow force pushes (never!)
  ❌ Allow deletions
```

### 5. Auto-merge

```bash
# Enable auto-merge khi có approval
# PR sẽ tự động merge khi:
# - Có đủ approvals
# - CI/CD pass
# - No conflicts
```

## Best Practices

### 1. Keep PRs Small

❌ **BAD**: PR với 50 files, 2000+ lines changed
```
Title: "Refactor entire codebase"
```

✅ **GOOD**: PR với 3-5 files, <300 lines
```
Title: "Refactor user authentication module"
```

### 2. Clear PR Descriptions

```markdown
## Problem
Users cannot reset password via email.

## Solution
- Add password reset email template
- Implement reset token generation
- Add reset password API endpoint

## Testing
Tested on staging with 10 users successfully.

Closes #123
```

### 3. Continuous Integration

```bash
# Run tests on every push
# Build on every PR
# Auto-deploy on merge to main
```

### 4. Feature Flags

Deploy code nhưng chưa enable:

```javascript
// Use feature flags for risky features
if (featureFlags.isEnabled('new-comment-system')) {
  return <NewCommentSystem />;
} else {
  return <OldCommentSystem />;
}
```

### 5. Rollback Strategy

```bash
# Nếu deploy lỗi, rollback nhanh:
git revert <commit-hash>
git push origin main
# Auto-deploy previous version
```

## So sánh với Git Flow

| Feature | GitHub Flow | Git Flow |
|---------|-------------|----------|
| **Branches** | main + features | main + develop + features + releases + hotfixes |
| **Complexity** | ⭐ Simple | ⭐⭐⭐ Complex |
| **Deploy** | Continuous | Scheduled releases |
| **Hotfix** | Create branch → PR → merge | Separate hotfix branch process |
| **Best for** | Web apps, SaaS | Enterprise, versioned products |
| **Learning curve** | ⭐ Easy | ⭐⭐⭐ Hard |

## Common Issues & Solutions

### Issue 1: Merge Conflicts

```bash
# Update branch với main
git checkout feature/my-feature
git fetch origin
git merge origin/main

# Resolve conflicts
# ... fix conflicts ...
git add .
git commit -m "merge: resolve conflicts with main"
git push
```

### Issue 2: PR Quá lớn

```bash
# Tách thành nhiều PRs nhỏ
git checkout feature/big-feature
git checkout -b feature/part-1

# Cherry-pick commits liên quan
git cherry-pick <commit1> <commit2>
git push -u origin feature/part-1

# Tạo PR cho part-1
# Sau đó làm part-2, part-3...
```

### Issue 3: CI/CD Fails

```bash
# Fix locally
git checkout feature/my-feature
npm test  # Reproduce error
# ... fix ...
git add .
git commit -m "fix: resolve CI test failures"
git push
```

## Automated Deployment

### Simple Deploy Script

```bash
#!/bin/bash
# deploy.sh

set -e

echo "Pulling latest code..."
git pull origin main

echo "Installing dependencies..."
npm install

echo "Running tests..."
npm test

echo "Building..."
npm run build

echo "Deploying..."
rsync -avz --delete dist/ user@server:/var/www/app/

echo "✅ Deployment successful!"
```

### With Docker

```yaml
# docker-compose.yml
version: '3'
services:
  app:
    image: myapp:${TAG:-latest}
    ports:
      - "80:3000"
```

```bash
# deploy.sh
docker build -t myapp:latest .
docker-compose up -d
```

## Checklist

Đảm bảo bạn hiểu:

- [ ] Main branch luôn ready to deploy
- [ ] Mỗi task/feature một branch riêng
- [ ] Tạo PR sớm để discuss
- [ ] Review kỹ trước khi merge
- [ ] Deploy ngay sau khi merge
- [ ] Có rollback plan
- [ ] Setup branch protection
- [ ] CI/CD working properly

## Tiếp theo

- **[GitLab Flow →](gitlab-flow.md)** - Kết hợp GitHub Flow + environments
- **[Common Scenarios →](common-scenarios.md)** - Xử lý các tình huống thực tế
- **[Best Practices →](best-practices.md)** - Practices tốt nhất

## Tài liệu tham khảo

- **[GitHub Flow Guide](https://guides.github.com/introduction/flow/)** - Hướng dẫn chính thức
- **[Understanding GitHub Flow](https://githubflow.github.io/)** - Interactive guide
- **[Atlassian: GitHub Flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)** - So sánh workflows
