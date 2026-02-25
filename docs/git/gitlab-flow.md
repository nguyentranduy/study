# GitLab Flow - Best of Both Worlds

GitLab Flow kết hợp ưu điểm của Git Flow (environment branches) và GitHub Flow (simplicity), phù hợp với modern DevOps practices.

## Tổng quan GitLab Flow

<img src="https://about.gitlab.com/images/git_flow/environment_branches.png" alt="GitLab Flow" width="600">

*GitLab Flow with environment branches*

### Triết lý

!!! quote "GitLab Philosophy"
    "Upstream first" - Changes should flow **downstream**:  
    **feature → main → pre-production → production**

### Đặc điểm chính

✅ **Environment branches**: staging, production branches  
✅ **Issue tracking**: Tight integration với GitLab Issues  
✅ **Merge Requests**: Similar to Pull Requests  
✅ **CI/CD pipelines**: Built-in deployment automation  
✅ **Simpler than Git Flow**: Nhưng mạnh hơn GitHub Flow  

## Khi nào dùng GitLab Flow?

### ✅ Phù hợp với

- **Multiple environments**: Dev, Staging, Production
- **Microservices**: Mỗi service cần deploy riêng
- **GitOps**: Infrastructure as Code
- **Regulated industries**: Cần approval gates
- **Medium to large teams**: 10-100 developers
- **Continuous Delivery** (không phải Continuous Deployment)

### ❌ Không phù hợp với

- **Very simple projects**: GitHub Flow đủ rồi
- **Frequent hotfixes**: Git Flow tốt hơn
- **No DevOps culture**: Cần CI/CD mature

## Ba chiến lược GitLab Flow

### 1. Environment Branches

Mỗi environment một branch:

```
feature branches → main → staging → production
```

```bash
# Feature development
git checkout -b feature/add-payment main
# ... code ...
git push -u origin feature/add-payment

# Create MR to main
# After merge to main → Auto deploy to Dev

# Deploy to staging
git checkout staging
git merge main
git push origin staging
# → Auto deploy to Staging

# Deploy to production
git checkout production
git merge staging
git push origin production
# → Auto deploy to Production
```

**Pros:**
- ✅ Clear separation of environments
- ✅ Can hold back production while staging tests
- ✅ Easy to see what's in each environment

**Cons:**
- ❌ More branches to maintain
- ❌ Need to merge forward after hotfixes

### 2. Release Branches

Tương tự Git Flow nhưng đơn giản hơn:

```
feature branches → main → release/2.0 → production
```

```bash
# When ready to release
git checkout -b release/2.0 main

# Deploy release branch to production
git push origin release/2.0
# → Deploy to Production

# Bug fixes to release branch
git checkout -b fix/critical-bug release/2.0
# ... fix ...
git checkout release/2.0
git merge fix/critical-bug

# Merge back to main
git checkout main
git merge release/2.0
```

**Use when:**
- Need to maintain multiple versions
- Can't deploy main immediately
- Long-running releases

### 3. Main Branch with Tags

Simplest approach (gần giống GitHub Flow):

```
feature branches → main (tagged) → production
```

```bash
# Merge feature to main
git checkout main
git merge feature/payment

# Tag for deployment
git tag -a v1.2.3 -m "Release 1.2.3"
git push origin main --tags

# CI/CD deploy tag v1.2.3
```

**Use when:**
- Fast-paced deployment
- Strong CI/CD
- Can deploy main anytime

## Workflow chi tiết

### Bước 1: Create Issue

Bắt đầu từ GitLab Issue:

```
Issue #123: Add user notification system
Labels: feature, priority::high
Assignee: @john
Milestone: Sprint 5
```

### Bước 2: Create Branch from Issue

```bash
# GitLab tạo branch name tự động từ issue
# Format: <issue-number>-<issue-title>
git checkout -b 123-add-user-notification-system

# Hoặc tạo manual
git checkout -b feature/123-notifications main
```

### Bước 3: Commit với Issue Reference

```bash
# Link commits to issues
git commit -m "feat: add notification service

Implements notification system with email and push.

Related to #123"

git push -u origin 123-add-user-notification-system
```

### Bước 4: Create Merge Request

**Trên GitLab:**
1. Create Merge Request
2. Link to issue: Closes #123
3. Add reviewers
4. Set target branch: main
5. Enable "Delete source branch" option

**MR Description:**
```markdown
## What does this MR do?
Implements user notification system with email and push notifications.

## Related issues
Closes #123

## Screenshots
![Notification UI](screenshot.png)

## How to test?
1. Go to user settings
2. Enable notifications
3. Trigger an event
4. Check email and app notifications

## Checklist
- [x] Tests added
- [x] Documentation updated
- [x] Changelog updated
```

### Bước 5: Review & CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy_staging
  - deploy_production

test:
  stage: test
  script:
    - npm test
  only:
    - merge_requests

build:
  stage: build
  script:
    - npm run build
  only:
    - main
    - staging
    - production

deploy_staging:
  stage: deploy_staging
  script:
    - ./deploy-to-staging.sh
  only:
    - staging
  when: manual

deploy_production:
  stage: deploy_production
  script:
    - ./deploy-to-production.sh
  only:
    - production
  when: manual
```

### Bước 6: Merge to Main

```bash
# Sau khi approved và CI pass
# Merge MR on GitLab
# → Auto deploy to Dev environment
# → Issue #123 automatically closed
```

### Bước 7: Deploy to Environments

```bash
# Deploy to Staging
git checkout staging
git merge main
git push origin staging
# → CI/CD deploys to staging

# Test on staging
# ...

# Deploy to Production
git checkout production
git merge staging
git push origin production
# → CI/CD deploys to production
```

## Issue-driven Development

### Workflow

```
Issue → Branch → Commits → MR → Review → Merge → Close Issue
```

### Issue Board

```
┌─────────────┬─────────────┬─────────────┬─────────────┐
│   Backlog   │  In Progress│   Review    │    Done     │
├─────────────┼─────────────┼─────────────┼─────────────┤
│ #123 Notif  │ #124 Search │ #125 Auth   │ #120 Login  │
│ #126 Export │ #127 Upload │             │ #121 Signup │
└─────────────┴─────────────┴─────────────┴─────────────┘
```

### Quick Actions

Trong MR description hoặc comments:

```markdown
/assign @john
/label ~feature ~priority::high
/milestone Sprint-5
/close #123
/spend 2h
/estimate 8h
```

## GitLab CI/CD Integration

### Pipeline Example

```yaml
# .gitlab-ci.yml
variables:
  DOCKER_IMAGE: registry.gitlab.com/myapp/backend

stages:
  - test
  - build
  - deploy_dev
  - deploy_staging
  - deploy_prod

# Run tests on all MRs
test:
  stage: test
  image: node:18
  script:
    - npm install
    - npm test
    - npm run lint
  coverage: '/Coverage: \d+\.\d+%/'
  only:
    - merge_requests
    - main

# Build Docker image
build:
  stage: build
  image: docker:20
  services:
    - docker:dind
  script:
    - docker build -t $DOCKER_IMAGE:$CI_COMMIT_SHA .
    - docker push $DOCKER_IMAGE:$CI_COMMIT_SHA
  only:
    - main
    - staging
    - production

# Auto deploy to dev
deploy_dev:
  stage: deploy_dev
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE:$CI_COMMIT_SHA
  environment:
    name: development
    url: https://dev.myapp.com
  only:
    - main

# Manual deploy to staging
deploy_staging:
  stage: deploy_staging
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE:$CI_COMMIT_SHA -n staging
  environment:
    name: staging
    url: https://staging.myapp.com
  only:
    - staging
  when: manual

# Manual deploy to production with approval
deploy_production:
  stage: deploy_prod
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE:$CI_COMMIT_SHA -n production
  environment:
    name: production
    url: https://myapp.com
  only:
    - production
  when: manual
  needs:
    - deploy_staging
```

### Environments

```yaml
# Define environments
environment:
  name: production
  url: https://myapp.com
  on_stop: stop_production    # Cleanup job
  auto_stop_in: 1 week         # Auto cleanup
```

### Deployment Strategies

```yaml
# Blue-Green Deployment
deploy_blue:
  script:
    - ./deploy-blue.sh
  environment:
    name: production/blue

deploy_green:
  script:
    - ./deploy-green.sh
  environment:
    name: production/green
  when: manual

# Canary Deployment
deploy_canary:
  script:
    - ./deploy-canary.sh 10%  # 10% traffic
  environment:
    name: production/canary

deploy_full:
  script:
    - ./deploy-full.sh 100%
  when: manual
```

## Merge Request Approvals

### Approval Rules

```yaml
# .gitlab/CODEOWNERS
# Backend code
/backend/** @backend-team

# Frontend code
/frontend/** @frontend-team

# Infrastructure
/infrastructure/** @devops-team @security-team

# Database migrations (need 2 approvals)
/migrations/** @backend-lead @dba-team
```

### Approval Matrix

```
Feature MR:
  - 1 approval from team member

Bug Fix:
  - 1 approval from senior

Hotfix:
  - 2 approvals (1 senior + 1 lead)

Security:
  - 3 approvals (security team + 2 seniors)
```

## GitOps with GitLab Flow

### Infrastructure as Code

```yaml
# infrastructure/kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: myapp
        image: registry.gitlab.com/myapp:v1.2.3
```

### GitOps Workflow

```bash
# 1. Update infrastructure code
git checkout -b infra/add-redis main
# Edit infrastructure files
git commit -m "infra: add Redis cluster"
git push

# 2. Create MR
# 3. Review infrastructure changes
# 4. Merge to main

# 5. GitLab CI/CD applies changes
# → Terraform/Ansible/Kubernetes apply
# → Infrastructure updated automatically
```

## Ví dụ thực tế

### E-commerce Application

```bash
# Feature: Add product reviews
git checkout -b 234-product-reviews main
# ... code ...
git commit -m "feat: add product review system

Closes #234"
git push -u origin 234-product-reviews

# Create MR → main
# CI runs: tests, security scan, build
# Review approved
# Merge to main → Deploy to Dev

# Staging deployment
git checkout staging
git merge main
git push origin staging
# → Deploy to Staging
# QA testing on staging

# Production deployment
git checkout production
git merge staging
git tag -a v2.1.0 -m "Release 2.1.0 - Product Reviews"
git push origin production --tags
# → Deploy to Production (after approval)
```

### Hotfix Scenario

```bash
# Bug found on production
git checkout production
git checkout -b hotfix/payment-error production

# Fix bug
git commit -m "fix: resolve payment processing error

Critical bug causing payment failures.

Fixes #567"

# Test fix
git push -u origin hotfix/payment-error

# Create MR → production
# Fast review and approval
# Merge to production
# → Deploy immediately

# Merge back to main
git checkout main
git merge hotfix/payment-error
git push origin main

# Merge to staging
git checkout staging
git merge main
git push origin staging
```

## Best Practices

### 1. Upstream First

```
✅ ĐÚNG:
feature → main → staging → production

❌ SAI:
feature → production → main (cherry-pick)
```

### 2. Branch Naming

```bash
# Issue-driven
234-add-product-reviews
567-fix-payment-error

# Type-based
feature/product-reviews
fix/payment-error
hotfix/critical-security-patch
```

### 3. Commit Messages

```bash
# Link to issues
git commit -m "feat: add product reviews

Implements review system with ratings and comments.

Closes #234
See also #235"
```

### 4. Keep Branches Updated

```bash
# Rebase on main regularly
git checkout feature/my-feature
git fetch origin
git rebase origin/main
```

### 5. Small MRs

- Mỗi MR < 400 lines changed
- 1 MR = 1 issue/feature
- Easy to review

## So sánh với các workflows khác

| Feature | GitLab Flow | Git Flow | GitHub Flow |
|---------|-------------|----------|-------------|
| **Complexity** | ⭐⭐ Medium | ⭐⭐⭐ High | ⭐ Low |
| **Branches** | main + env/releases | main + develop + features + releases | main + features |
| **Deployment** | Environment-based | Version-based | Continuous |
| **CI/CD** | ⭐⭐⭐ Excellent | ⭐ Basic | ⭐⭐ Good |
| **Issue tracking** | ⭐⭐⭐ Integrated | ⭐ External | ⭐⭐ Good |
| **Best for** | DevOps teams | Enterprise | Startups |

## Checklist

- [ ] Hiểu 3 strategies: Environment, Release, Tags
- [ ] Biết workflow từ Issue → MR → Deploy
- [ ] Setup GitLab CI/CD pipelines
- [ ] Cấu hình environments (dev, staging, prod)
- [ ] Setup approval rules
- [ ] Hiểu "upstream first" principle
- [ ] Đã thực hành ít nhất 1 full cycle

## Tiếp theo

- **[Common Scenarios →](common-scenarios.md)** - Xử lý tình huống thực tế
- **[Best Practices →](best-practices.md)** - Practices tốt nhất
- **[Git Flow →](git-flow.md)** - So sánh với Git Flow

## Tài liệu tham khảo

- **[GitLab Flow Documentation](https://docs.gitlab.com/ee/topics/gitlab_flow.html)** - Tài liệu chính thức
- **[Introduction to GitLab Flow](https://about.gitlab.com/topics/version-control/what-is-gitlab-flow/)** - Blog post chi tiết
- **[GitLab CI/CD](https://docs.gitlab.com/ee/ci/)** - CI/CD documentation
