# Branching Strategies - Chiến lược phân nhánh

Branches (nhánh) là một trong những tính năng mạnh mẽ nhất của Git. Hiểu và sử dụng tốt branches giúp team làm việc hiệu quả mà không conflict.

## Branch là gì?

**Branch** là một con trỏ di động trỏ đến một commit. Mỗi branch đại diện cho một luồng phát triển độc lập.

<img src="https://wac-cdn.atlassian.com/dam/jcr:a905ddfd-973a-452a-a4ae-f1dd65430027/01%20Git%20branch.svg?cdnVersion=1483" alt="Git Branching" width="600">

*Sơ đồ minh họa Git branches*

### Tại sao cần dùng branch?

✅ **Làm việc song song**: Nhiều người code nhiều tính năng cùng lúc  
✅ **Bảo vệ code chính**: Main branch luôn ổn định, có thể deploy  
✅ **Dễ review code**: Mỗi feature một branch, dễ review  
✅ **Dễ rollback**: Có vấn đề chỉ cần xóa branch đó  
✅ **Testing an toàn**: Test feature mới không ảnh hưởng code chính  

## Các lệnh cơ bản với Branch

### 1. Xem branches

```bash
# Xem tất cả branches (local)
git branch

# Xem cả remote branches
git branch -a

# Xem branch hiện tại
git branch --show-current
```

### 2. Tạo branch mới

```bash
# Tạo branch mới (nhưng không chuyển sang)
git branch feature/login

# Tạo và chuyển sang branch mới
git checkout -b feature/login

# Hoặc dùng lệnh mới hơn (Git 2.23+)
git switch -c feature/login

# Tạo branch từ một commit cụ thể
git branch feature/payment abc123d
```

### 3. Chuyển branch

```bash
# Chuyển sang branch khác
git checkout feature/login

# Hoặc dùng lệnh mới
git switch feature/login

# Chuyển về branch trước đó
git checkout -
```

### 4. Xóa branch

```bash
# Xóa branch đã merge (an toàn)
git branch -d feature/login

# Xóa branch chưa merge (force)
git branch -D feature/login

# Xóa remote branch
git push origin --delete feature/login
```

### 5. Đổi tên branch

```bash
# Đổi tên branch hiện tại
git branch -m new-name

# Đổi tên branch khác
git branch -m old-name new-name
```

## Merge Branches

### Fast-Forward Merge

Khi không có commit mới trên branch đích, Git chỉ cần di chuyển con trỏ:

```bash
# Merge feature vào main
git checkout main
git merge feature/login
```

<img src="https://wac-cdn.atlassian.com/dam/jcr:d90f2536-7951-4e5e-ab79-f45a502fb4c8/03-04%20Fast%20forward%20merge.svg?cdnVersion=1483" alt="Fast Forward Merge" width="500">

### 3-Way Merge

Khi có commits mới ở cả hai branches, Git tạo merge commit:

```bash
git checkout main
git merge feature/login
# Git tự động tạo merge commit
```

<img src="https://wac-cdn.atlassian.com/dam/jcr:91aece4a-8fa0-4fc3-bae9-69d51932f104/05-06%20Fast%20forward%20merge.svg?cdnVersion=1483" alt="3-Way Merge" width="500">

### Merge với --no-ff

Luôn tạo merge commit (dù có thể fast-forward):

```bash
git merge --no-ff feature/login
```

!!! tip "Khi nào dùng --no-ff?"
    - ✅ Khi muốn giữ lịch sử rõ ràng: "feature này được merge lúc nào"
    - ✅ Khi làm việc theo Git Flow
    - ❌ Không cần thiết với commits nhỏ, đơn giản

## Rebase - Cách merge "sạch" hơn

Rebase ghi lại commits của bạn lên trên commits mới nhất của branch khác:

```bash
# Thay vì merge
git checkout feature/login
git rebase main

# Sau đó merge vào main (sẽ là fast-forward)
git checkout main
git merge feature/login
```

<img src="https://wac-cdn.atlassian.com/dam/jcr:4e576671-1b7f-43db-afb5-cf8db8df8e4a/01%20What%20is%20git%20rebase.svg?cdnVersion=1483" alt="Git Rebase" width="600">

### Merge vs Rebase

| Tiêu chí | Merge | Rebase |
|----------|-------|--------|
| **Lịch sử** | Giữ nguyên lịch sử thực | Viết lại lịch sử tuyến tính |
| **Merge commits** | Có merge commits | Không có merge commits |
| **Conflicts** | Giải quyết 1 lần | Có thể giải quyết nhiều lần |
| **Dùng khi** | Public branches | Private feature branches |
| **An toàn** | An toàn hơn | Nguy hiểm nếu dùng sai |

!!! warning "Golden Rule of Rebase"
    **KHÔNG BAO GIỜ rebase một branch đã được push và người khác đang dùng!**
    
    ✅ OK: Rebase feature branch riêng của bạn  
    ❌ KHÔNG: Rebase main/develop branch  

## Giải quyết Conflicts

Conflict xảy ra khi cùng một đoạn code bị sửa ở 2 branches khác nhau.

### Ví dụ conflict

```java
// File: User.java
<<<<<<< HEAD (Current Change)
public String getFullName() {
    return firstName + " " + lastName;
}
=======
public String getFullName() {
    return this.firstName + " " + this.lastName;  
}
>>>>>>> feature/update-user (Incoming Change)
```

### Các bước giải quyết

```bash
# 1. Merge và gặp conflict
git merge feature/login
# CONFLICT (content): Merge conflict in User.java

# 2. Xem files có conflict
git status

# 3. Mở file và sửa (xóa các dấu <<<<, ====, >>>>)
# Chọn code nào giữ lại hoặc kết hợp cả hai

# 4. Mark resolved
git add User.java

# 5. Commit merge
git commit
# Hoặc với rebase
git rebase --continue

# Nếu muốn hủy merge/rebase
git merge --abort
git rebase --abort
```

### Tips giải quyết conflicts

!!! tip "Công cụ hỗ trợ"
    - **VS Code**: Built-in merge tool rất trực quan
    - **GitKraken**: UI đẹp, dễ hiểu
    - **Meld**: Free, cross-platform
    - **P4Merge**: Perforce merge tool
    
    Cấu hình merge tool:
    ```bash
    git config --global merge.tool vscode
    git config --global mergetool.vscode.cmd 'code --wait $MERGED'
    ```

## Branch Naming Conventions

Đặt tên branch theo convention giúp dễ quản lý:

### Chuẩn phổ biến

```bash
# Feature branches
feature/user-authentication
feature/payment-integration
feat/search-functionality

# Bug fix branches
fix/login-button-not-working
bugfix/memory-leak-issue
hotfix/critical-security-patch

# Release branches
release/v1.2.0
release/2024-02-25

# Experimental branches
experiment/new-algorithm
poc/blockchain-integration
```

### Pattern thường dùng

```
<type>/<short-description>

Type:
- feature/ hoặc feat/   : Tính năng mới
- fix/ hoặc bugfix/      : Sửa bug
- hotfix/                : Sửa bug khẩn cấp trên production
- release/               : Chuẩn bị release
- docs/                  : Cập nhật documentation
- style/                 : Format code (không ảnh hưởng logic)
- refactor/              : Refactor code
- test/                  : Thêm tests
- chore/                 : Công việc maintenance
```

!!! example "Ví dụ thực tế"
    ```bash
    feature/US-123-user-profile
    fix/BUG-456-null-pointer
    hotfix/PROD-789-payment-error
    release/v2.0.0
    ```

## Working với Remote Branches

```bash
# Push branch lên remote
git push origin feature/login

# Push và set upstream (lần đầu)
git push -u origin feature/login

# Pull branch từ remote
git pull origin feature/login

# Fetch tất cả branches
git fetch --all

# Checkout remote branch
git checkout -b feature/login origin/feature/login

# Hoặc ngắn gọn hơn
git checkout feature/login  # Git tự động tìm remote branch

# Xem tracking relationship
git branch -vv
```

## Branch Protection Rules

Trên GitHub/GitLab, nên bảo vệ các branches quan trọng:

### Quy tắc thường áp dụng cho main/develop

✅ **Require pull request reviews**: Phải có review mới được merge  
✅ **Require status checks**: CI/CD phải pass  
✅ **Require branches to be up to date**: Phải update từ main trước khi merge  
✅ **Include administrators**: Cả admin cũng phải follow rules  
✅ **Restrict who can push**: Chỉ một số người được push trực tiếp  

## Ví dụ Workflow thực tế

```bash
# 1. Bắt đầu feature mới
git checkout main
git pull origin main                # Đảm bảo có code mới nhất
git checkout -b feature/user-profile

# 2. Code feature
# ... viết code ...
git add .
git commit -m "feat: add user profile page"

# 3. Push lên remote
git push -u origin feature/user-profile

# 4. Update từ main (nếu main có code mới)
git checkout main
git pull origin main
git checkout feature/user-profile
git merge main                       # Hoặc: git rebase main

# 5. Giải quyết conflicts (nếu có)
# ... fix conflicts ...
git add .
git commit -m "merge: resolve conflicts with main"

# 6. Push updates
git push

# 7. Tạo Pull Request trên GitHub
# ... review code ...

# 8. Merge vào main
git checkout main
git pull origin main                 # Lấy code sau khi PR merged
git branch -d feature/user-profile   # Xóa local branch
```

## Bài tập thực hành

??? example "Bài 1: Tạo và merge branches"
    1. Tạo branch `feature/header` từ main
    2. Thêm file `header.html`, commit
    3. Chuyển về main, tạo branch `feature/footer`
    4. Thêm file `footer.html`, commit
    5. Merge cả 2 branches vào main
    
    ??? success "Giải đáp"
        ```bash
        # Feature header
        git checkout -b feature/header
        echo "<header>Header</header>" > header.html
        git add header.html
        git commit -m "feat: add header"
        
        # Feature footer
        git checkout main
        git checkout -b feature/footer
        echo "<footer>Footer</footer>" > footer.html
        git add footer.html
        git commit -m "feat: add footer"
        
        # Merge
        git checkout main
        git merge feature/header
        git merge feature/footer
        ```

??? example "Bài 2: Xử lý conflict"
    1. Tạo file `config.txt` trên main với nội dung "mode=dev"
    2. Tạo branch `feature/prod`, sửa thành "mode=prod", commit
    3. Về main, sửa thành "mode=test", commit
    4. Merge `feature/prod` vào main và giải quyết conflict
    
    ??? success "Giải đáp"
        ```bash
        # Setup
        echo "mode=dev" > config.txt
        git add config.txt
        git commit -m "feat: add config"
        
        # Branch prod
        git checkout -b feature/prod
        echo "mode=prod" > config.txt
        git commit -am "feat: set prod mode"
        
        # Main với test
        git checkout main
        echo "mode=test" > config.txt
        git commit -am "feat: set test mode"
        
        # Merge (sẽ có conflict)
        git merge feature/prod
        # Sửa config.txt, chọn mode nào
        git add config.txt
        git commit -m "merge: resolve config conflict"
        ```

## Checklist

Kiểm tra xem bạn đã nắm vững:

- [ ] Tạo, chuyển, xóa branches thành thạo
- [ ] Hiểu sự khác biệt giữa merge và rebase
- [ ] Biết cách giải quyết conflicts
- [ ] Đặt tên branch theo convention
- [ ] Biết khi nào nên tạo branch mới
- [ ] Hiểu branch protection rules
- [ ] Đã thực hành ít nhất 5 lần merge branches

## Tiếp theo

Sau khi thành thạo branching, học về các Git Workflows:

1. **[Git Flow →](git-flow.md)** - Workflow chuẩn cho dự án lớn
2. **[GitHub Flow →](github-flow.md)** - Workflow đơn giản cho CI/CD
3. **[Common Scenarios →](common-scenarios.md)** - Xử lý tình huống thực tế
