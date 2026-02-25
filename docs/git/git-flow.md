# Git Flow - Workflow chuẩn mực

Git Flow là một branching model được Vincent Driessen giới thiệu năm 2010 và đã trở thành chuẩn de-facto cho nhiều dự án lớn.

## Tổng quan Git Flow

<img src="https://nvie.com/img/git-model@2x.png" alt="Git Flow" width="600">

*Sơ đồ Git Flow model (nguồn: nvie.com)*

Git Flow định nghĩa một cấu trúc branching nghiêm ngặt xung quanh project releases.

### Khi nào nên dùng Git Flow?

✅ **NÊN dùng khi:**
- Dự án lớn, nhiều developers
- Release theo chu kỳ cố định (v1.0, v2.0...)
- Cần maintain nhiều versions cùng lúc
- Có môi trường staging/production riêng biệt
- Team cần quy trình chặt chẽ

❌ **KHÔNG nên dùng khi:**
- Dự án nhỏ, 1-2 developers
- Deploy liên tục (CD)
- Muốn workflow đơn giản
- Startup cần move nhanh

## Các loại Branches

### 1. Main Branches (Branches chính)

#### `main` (hoặc `master`)
- **Mục đích**: Chứa code **production-ready**
- **Luôn stable**: Mọi commit đều có thể deploy
- **Protected**: Không được push trực tiếp
- **Tagged**: Mỗi merge vào main là một version release

#### `develop`
- **Mục đích**: Branch chính cho development
- **Integration branch**: Nơi tích hợp tất cả features
- **Luôn chạy được**: Code phải build thành công
- **Xuất phát**: Từ main ban đầu

```bash
# Khởi tạo Git Flow
git checkout -b develop main
```

### 2. Supporting Branches (Branches hỗ trợ)

#### Feature Branches
- **Tên**: `feature/*` hoặc `feature/<tên-feature>`
- **Xuất phát từ**: `develop`
- **Merge vào**: `develop`
- **Mục đích**: Phát triển tính năng mới
- **Lifetime**: Tồn tại cho đến khi feature hoàn thành

```bash
# Tạo feature branch
git checkout -b feature/user-authentication develop

# ... code feature ...

# Merge vào develop
git checkout develop
git merge --no-ff feature/user-authentication
git branch -d feature/user-authentication
git push origin develop
```

#### Release Branches
- **Tên**: `release/*` hoặc `release/<version>`
- **Xuất phát từ**: `develop`
- **Merge vào**: `main` và `develop`
- **Mục đích**: Chuẩn bị cho production release
- **Chỉ fix bugs**: Không thêm features mới
- **Lifetime**: Cho đến khi release

```bash
# Tạo release branch
git checkout -b release/1.2.0 develop

# Fix bugs, update version numbers
# ... minor fixes ...
git commit -am "chore: bump version to 1.2.0"

# Merge vào main
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"

# Merge lại vào develop
git checkout develop
git merge --no-ff release/1.2.0

# Xóa release branch
git branch -d release/1.2.0
```

#### Hotfix Branches
- **Tên**: `hotfix/*` hoặc `hotfix/<bug-name>`
- **Xuất phát từ**: `main`
- **Merge vào**: `main` và `develop`
- **Mục đích**: Sửa bug khẩn cấp trên production
- **Không thông qua develop**: Sửa trực tiếp từ main

```bash
# Tạo hotfix branch từ main
git checkout -b hotfix/critical-security-fix main

# Fix bug
# ... fix code ...
git commit -am "fix: resolve critical security vulnerability"

# Merge vào main
git checkout main
git merge --no-ff hotfix/critical-security-fix
git tag -a v1.2.1 -m "Hotfix v1.2.1"

# Merge vào develop
git checkout develop
git merge --no-ff hotfix/critical-security-fix

# Xóa hotfix branch
git branch -d hotfix/critical-security-fix
```

## Quy trình làm việc

### Workflow cho Feature mới

```bash
# 1. Developer A bắt đầu feature
git checkout develop
git pull origin develop
git checkout -b feature/shopping-cart

# 2. Code feature
# ... viết code ...
git add .
git commit -m "feat: implement add to cart"

# 3. Push lên remote
git push -u origin feature/shopping-cart

# 4. Update từ develop (nếu có code mới)
git checkout develop
git pull origin develop
git checkout feature/shopping-cart
git merge develop

# 5. Tạo Pull Request trên GitHub/GitLab
# Team review code

# 6. Sau khi approved, merge vào develop
git checkout develop
git merge --no-ff feature/shopping-cart
git push origin develop

# 7. Xóa feature branch
git branch -d feature/shopping-cart
git push origin --delete feature/shopping-cart
```

### Workflow cho Release

```bash
# 1. Tạo release branch khi develop ready
git checkout -b release/2.0.0 develop

# 2. Bump version, fix bugs nhỏ
echo "2.0.0" > VERSION
git commit -am "chore: bump version to 2.0.0"

# 3. Testing QA
# ... QA test và report bugs ...
git commit -am "fix: resolve QA issues"

# 4. Release ready - merge vào main
git checkout main
git merge --no-ff release/2.0.0
git tag -a v2.0.0 -m "Release 2.0.0"
git push origin main --tags

# 5. Merge changes lại develop
git checkout develop
git merge --no-ff release/2.0.0
git push origin develop

# 6. Xóa release branch
git branch -d release/2.0.0
```

### Workflow cho Hotfix

```bash
# 1. Phát hiện bug critical trên production
git checkout main
git checkout -b hotfix/payment-error

# 2. Fix bug nhanh
# ... fix code ...
git commit -am "fix: resolve payment processing error"

# 3. Merge vào main
git checkout main
git merge --no-ff hotfix/payment-error
git tag -a v2.0.1 -m "Hotfix 2.0.1 - Payment error"
git push origin main --tags

# 4. Merge vào develop
git checkout develop
git merge --no-ff hotfix/payment-error
git push origin develop

# 5. Xóa hotfix branch
git branch -d hotfix/payment-error
```

## Git Flow CLI Tool

Có tool hỗ trợ tự động hóa Git Flow:

### Cài đặt

```bash
# macOS
brew install git-flow

# Windows
# Download từ: https://github.com/petervanderdoes/gitflow-avh

# Linux
sudo apt-get install git-flow
```

### Sử dụng

```bash
# Khởi tạo Git Flow
git flow init

# Feature
git flow feature start shopping-cart
# ... code ...
git flow feature finish shopping-cart

# Release
git flow release start 2.0.0
# ... testing, bug fixes ...
git flow release finish 2.0.0

# Hotfix
git flow hotfix start critical-bug
# ... fix ...
git flow hotfix finish critical-bug
```

!!! tip "Lợi ích của git-flow tool"
    - ✅ Tự động tạo branches đúng chuẩn
    - ✅ Tự động merge và tag
    - ✅ Giảm lỗi do thao tác manual
    - ❌ Nhưng **NÊN hiểu rõ Git Flow trước** khi dùng tool

## Branch Protection Rules

Setup trên GitHub/GitLab:

### Cho `main` branch

```yaml
Protection Rules:
  - Require pull request reviews (≥ 2 approvals)
  - Require status checks to pass (CI/CD)
  - Require branches to be up to date
  - Do not allow force pushes
  - Do not allow deletions
  - Require linear history
```

### Cho `develop` branch

```yaml
Protection Rules:
  - Require pull request reviews (≥ 1 approval)
  - Require status checks to pass
  - Allow force pushes (for maintainers only)
```

## Ví dụ thực tế

### Sprint Planning

```bash
# Sprint 1 bắt đầu
# Team lead tạo sprint branch (optional)
git checkout -b sprint/sprint-1 develop

# Developer 1: User Story US-101
git checkout -b feature/US-101-login sprint/sprint-1

# Developer 2: User Story US-102
git checkout -b feature/US-102-register sprint/sprint-1

# ... sau 2 tuần ...

# Merge features vào develop
git checkout develop
git merge --no-ff feature/US-101-login
git merge --no-ff feature/US-102-register

# Chuẩn bị release
git checkout -b release/1.1.0 develop
# ... QA testing ...

# Release
git checkout main
git merge --no-ff release/1.1.0
git tag -a v1.1.0
```

## So sánh với workflows khác

| Tiêu chí | Git Flow | GitHub Flow | GitLab Flow |
|----------|----------|-------------|-------------|
| **Độ phức tạp** | Cao | Thấp | Trung bình |
| **Branches** | Nhiều types | Chỉ feature branches | Environment branches |
| **Release** | Theo versions | Continuous | Environment-based |
| **Phù hợp** | Enterprise | Startups, CI/CD | Microservices |
| **Learning curve** | Khó | Dễ | Trung bình |

## Ưu điểm & Nhược điểm

### ✅ Ưu điểm

- **Cấu trúc rõ ràng**: Mọi người biết branch nào làm gì
- **Parallel development**: Nhiều features cùng lúc
- **Production stability**: Main branch luôn stable
- **Release management**: Dễ quản lý versions
- **Hotfix nhanh**: Có quy trình sửa bug khẩn

### ❌ Nhược điểm

- **Phức tạp**: Nhiều branches, nhiều rules
- **Overhead**: Merge nhiều, conflicts nhiều
- **Không phù hợp CI/CD**: Deploy liên tục khó khăn
- **Learning curve**: Mất thời gian để team làm quen

## Best Practices

### 1. Naming Conventions

```bash
feature/US-123-user-profile
feature/JIRA-456-payment-integration
release/v2.1.0
hotfix/PROD-789-critical-error
```

### 2. Commit Messages

Theo [Conventional Commits](https://www.conventionalcommits.org/):

```bash
feat(auth): add OAuth2 login
fix(payment): resolve null pointer exception
chore(deps): upgrade Spring Boot to 3.2.0
docs(readme): update installation guide
```

### 3. Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Feature
- [ ] Bug fix
- [ ] Hotfix
- [ ] Documentation

## Checklist
- [ ] Code reviewed
- [ ] Tests added
- [ ] Documentation updated
- [ ] No breaking changes
```

### 4. Automated Testing

```yaml
# .github/workflows/develop.yml
name: CI - Develop
on:
  push:
    branches: [ develop ]
  pull_request:
    branches: [ develop ]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: ./gradlew test
```

## Bài tập thực hành

??? example "Project: Todo App với Git Flow"
    Áp dụng Git Flow cho một project Todo App:
    
    **Sprint 1:**
    1. Khởi tạo Git Flow
    2. Tạo `feature/add-todo`
    3. Tạo `feature/list-todos`
    4. Merge cả 2 vào develop
    5. Tạo `release/1.0.0`, merge vào main
    
    **Sprint 2:**
    6. Tạo `feature/delete-todo`
    7. Phát hiện bug trên main, tạo `hotfix/todo-null-check`
    8. Merge hotfix vào main (v1.0.1) và develop
    9. Merge `feature/delete-todo` vào develop
    10. Release v1.1.0
    
    ??? success "Giải đáp"
        ```bash
        # Khởi tạo
        git init
        git checkout -b develop
        
        # Sprint 1
        git checkout -b feature/add-todo develop
        echo "Add todo function" > add-todo.txt
        git add . && git commit -m "feat: add todo"
        
        git checkout develop
        git merge --no-ff feature/add-todo
        
        git checkout -b feature/list-todos develop
        echo "List todos function" > list-todos.txt
        git add . && git commit -m "feat: list todos"
        
        git checkout develop
        git merge --no-ff feature/list-todos
        
        # Release 1.0.0
        git checkout -b release/1.0.0 develop
        echo "1.0.0" > VERSION
        git add . && git commit -m "chore: bump to 1.0.0"
        
        git checkout -b main
        git merge --no-ff release/1.0.0
        git tag -a v1.0.0 -m "Release 1.0.0"
        
        git checkout develop
        git merge --no-ff release/1.0.0
        
        # Hotfix
        git checkout -b hotfix/todo-null-check main
        echo "Null check" > null-check.txt
        git add . && git commit -m "fix: add null check"
        
        git checkout main
        git merge --no-ff hotfix/todo-null-check
        git tag -a v1.0.1 -m "Hotfix 1.0.1"
        
        git checkout develop
        git merge --no-ff hotfix/todo-null-check
        ```

## Checklist

- [ ] Hiểu rõ 5 loại branches trong Git Flow
- [ ] Biết khi nào tạo feature/release/hotfix branch
- [ ] Thực hành ít nhất 1 full cycle: feature → release → hotfix
- [ ] Biết merge với `--no-ff` và tại sao
- [ ] Hiểu flow của hotfix branch
- [ ] Đã setup branch protection rules
- [ ] Có thể giải thích Git Flow cho người khác

## Tiếp theo

- **[GitHub Flow →](github-flow.md)** - Workflow đơn giản hơn
- **[GitLab Flow →](gitlab-flow.md)** - Kết hợp ưu điểm cả hai
- **[Best Practices →](best-practices.md)** - Các practices tốt nhất

## Tài liệu tham khảo

- **[Original Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)** - Bài viết gốc của Vincent Driessen
- **[Git Flow Cheatsheet](https://danielkummer.github.io/git-flow-cheatsheet/)** - Bảng tra cứu nhanh
- **[Atlassian Git Flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)** - Tutorial chi tiết
