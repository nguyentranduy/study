# Git Fundamentals - Git cơ bản

Git là công cụ không thể thiếu trong lập trình hiện đại. (1) Hãy bắt đầu từ những kiến thức nền tảng.
{ .annotate }

1.  :man_raising_hand: **Git** được tạo bởi Linus Torvalds năm 2005 để quản lý Linux kernel source code. Hiện là hệ thống version control phổ biến nhất thế giới!

## Cài đặt Git

### Windows
```bash
# Tải Git for Windows
https://git-scm.com/download/win

# Hoặc dùng Chocolatey
choco install git

# Hoặc dùng winget
winget install Git.Git
```

### macOS
```bash
# Dùng Homebrew
brew install git

# Hoặc dùng Xcode Command Line Tools
xcode-select --install
```

### Linux
```bash
# Ubuntu/Debian
sudo apt-get install git

# Fedora
sudo dnf install git

# Arch Linux
sudo pacman -S git
```

## Cấu hình Git lần đầu

Sau khi cài đặt, cần cấu hình thông tin cá nhân:

```bash
# Cấu hình tên (hiển thị trong commit)
git config --global user.name "Nguyen Van A"

# Cấu hình email
git config --global user.email "nguyenvana@example.com"

# Cấu hình editor mặc định (optional)
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim

# Xem cấu hình hiện tại
git config --list
```

!!! tip "Tại sao cần cấu hình?"
    Mỗi commit sẽ ghi lại **ai** đã thay đổi code và **khi nào**. Git cần biết tên và email của bạn để ghi vào commit history.

## Git Workflow cơ bản

<img src="https://stopbyte.com/uploads/default/original/1X/274acda49c194fc1720e87bce210b661760877f7.png" alt="Git Workflow" width="600">

*Sơ đồ minh họa Git workflow cơ bản*

### 3 khu vực trong Git

1. **Working Directory** - Thư mục làm việc
   - Nơi bạn chỉnh sửa code
   - Files có thể là: untracked, modified, hoặc staged

2. **Staging Area (Index)** - Khu vực tạm
   - Nơi chuẩn bị files cho commit
   - Dùng `git add` để đưa files vào đây

3. **Repository (.git directory)** - Kho lưu trữ
   - Lưu trữ lịch sử commits
   - Dùng `git commit` để lưu vào đây

## Commands cơ bản

### 1. Tạo Repository

```bash
# Tạo repository mới
mkdir my-project
cd my-project
git init

# Hoặc clone repository có sẵn
git clone https://github.com/username/repo-name.git
```

### 2. Kiểm tra trạng thái

```bash
# Xem files đã thay đổi
git status

# Xem chi tiết thay đổi
git diff

# Xem lịch sử commits
git log
git log --oneline --graph --all  # Dạng đẹp hơn
```

### 3. Làm việc với files

```bash
# Thêm file vào staging area
git add filename.txt              # Thêm 1 file
git add .                         # Thêm tất cả files
git add *.java                    # Thêm tất cả .java files

# Xóa file khỏi staging area (nhưng giữ changes)
git reset filename.txt

# Bỏ hết changes (NGUY HIỂM!)
git checkout -- filename.txt      # Bỏ changes của 1 file
git reset --hard                  # Bỏ TẤT CẢ changes
```

### 4. Commit

```bash
# Commit với message
git commit -m "Add login feature"

# Commit với message dài
git commit
# Sẽ mở editor để viết message

# Add và commit cùng lúc (chỉ với tracked files)
git commit -am "Update README"

# Sửa commit cuối cùng
git commit --amend
```

!!! warning "Commit message tốt"
    ❌ SAI:
    ```
    git commit -m "fix"
    git commit -m "update"
    git commit -m "aaa"
    ```
    
    ✅ ĐÚNG:
    ```
    git commit -m "fix: resolve login bug when password is empty"
    git commit -m "feat: add user profile page"
    git commit -m "docs: update installation guide"
    ```

### 5. Làm việc với Remote

```bash
# Xem remote repositories
git remote -v

# Thêm remote repository
git remote add origin https://github.com/username/repo.git

# Push lên remote
git push origin main              # Push branch main
git push -u origin main           # Push và set upstream

# Pull từ remote
git pull origin main              # Fetch + Merge
git fetch origin                  # Chỉ fetch, không merge

# Clone repository
git clone https://github.com/username/repo.git
```

## Hiểu về Commits

Mỗi commit là một **snapshot** (ảnh chụp) của toàn bộ project tại một thời điểm.

```bash
# Cấu trúc một commit
commit abc123def456  # Hash (ID duy nhất)
Author: Nguyen Van A <email@example.com>
Date:   Mon Feb 25 10:30:00 2026 +0700

    feat: add user authentication
    
    - Implement login with email/password
    - Add JWT token generation
    - Create login API endpoint
```

### Commit Hash

```bash
# Xem chi tiết một commit
git show abc123d

# So sánh 2 commits
git diff abc123d def456e

# Quay lại một commit cũ
git checkout abc123d  # Detached HEAD state
```

## .gitignore - Bỏ qua files không cần thiết

Tạo file `.gitignore` trong root folder:

```gitignore
# Dependencies
node_modules/
vendor/

# Build outputs
*.class
*.jar
target/
dist/
build/

# IDE files
.idea/
.vscode/
*.iml

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Environment files
.env
.env.local

# Temporary files
*.tmp
*.swp
```

!!! tip "gitignore.io"
    Tạo `.gitignore` tự động cho project của bạn tại: [gitignore.io](https://www.toptal.com/developers/gitignore)

## Git Aliases - Lệnh tắt

Tạo aliases để gõ lệnh nhanh hơn:

```bash
# Thiết lập aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg 'log --oneline --graph --all --decorate'

# Sử dụng
git st           # thay vì git status
git co main      # thay vì git checkout main
git lg           # xem log đẹp
```

## Ví dụ workflow đầy đủ

```bash
# 1. Tạo project mới
mkdir my-app
cd my-app
git init

# 2. Tạo file đầu tiên
echo "# My App" > README.md
git add README.md
git commit -m "docs: initial commit"

# 3. Kết nối với GitHub
git remote add origin https://github.com/username/my-app.git
git push -u origin main

# 4. Thêm code
# ... viết code ...

# 5. Stage và commit
git add .
git status              # Kiểm tra trước khi commit
git commit -m "feat: add homepage"

# 6. Push lên GitHub
git push

# 7. Lặp lại bước 4-6 mỗi khi code xong một tính năng
```

## Bài tập thực hành

??? example "Bài 1: Tạo repository đầu tiên"
    1. Tạo folder `my-first-repo`
    2. Init Git repository
    3. Tạo file `index.html` với nội dung Hello World
    4. Commit file này
    5. Xem lịch sử commits
    
    ??? success "Giải đáp"
        ```bash
        mkdir my-first-repo
        cd my-first-repo
        git init
        
        echo "<h1>Hello World</h1>" > index.html
        git add index.html
        git commit -m "feat: add index.html"
        
        git log
        ```

??? example "Bài 2: Làm việc với staging area"
    1. Tạo 3 files: `a.txt`, `b.txt`, `c.txt`
    2. Add chỉ `a.txt` và `b.txt` vào staging
    3. Commit 2 files này
    4. Sau đó mới add và commit `c.txt`
    
    ??? success "Giải đáp"
        ```bash
        echo "File A" > a.txt
        echo "File B" > b.txt
        echo "File C" > c.txt
        
        git add a.txt b.txt
        git commit -m "feat: add files A and B"
        
        git add c.txt
        git commit -m "feat: add file C"
        ```

??? example "Bài 3: Push lên GitHub"
    1. Tạo repository mới trên GitHub
    2. Push local repository lên GitHub
    3. Thêm file mới, commit và push
    
    ??? success "Giải đáp"
        ```bash
        # Sau khi tạo repo trên GitHub
        git remote add origin https://github.com/username/repo-name.git
        git push -u origin main
        
        # Thêm file mới
        echo "New content" > new-file.txt
        git add new-file.txt
        git commit -m "feat: add new file"
        git push
        ```

## Checklist kiến thức cơ bản

Đảm bảo bạn nắm vững những điểm này trước khi chuyển sang phần tiếp:

- [ ] Cài đặt và cấu hình Git thành công
- [ ] Hiểu 3 khu vực: Working Directory, Staging Area, Repository
- [ ] Biết dùng: `git init`, `git add`, `git commit`, `git status`
- [ ] Viết được commit message rõ ràng
- [ ] Biết tạo và sử dụng `.gitignore`
- [ ] Đã push code lên GitHub/GitLab ít nhất 1 lần
- [ ] Hiểu sự khác biệt giữa `git pull` và `git fetch`

## Tiếp theo

Sau khi nắm vững Git cơ bản, hãy học về:

1. **[Branching Strategies →](branching.md)** - Làm việc với branches
2. **[Common Scenarios →](common-scenarios.md)** - Xử lý các tình huống thực tế

## Tài liệu tham khảo

- **[Git Documentation](https://git-scm.com/doc)** - Tài liệu chính thức
- **[Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)** - Bảng tra cứu nhanh
- **[Learn Git Branching](https://learngitbranching.js.org/?locale=vi)** - Game học Git

