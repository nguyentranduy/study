# Git & Git Workflow

Git là hệ thống quản lý phiên bản phân tán (Distributed Version Control System) phổ biến nhất hiện nay. Việc thành thạo Git và các Git Workflow là kỹ năng bắt buộc đối với mọi lập trình viên.

## Tại sao cần học Git?

!!! tip "Git là kỹ năng bắt buộc"
    - ✅ **Làm việc nhóm**: Nhiều người cùng code một dự án không conflict
    - ✅ **Theo dõi thay đổi**: Biết ai đã sửa gì, khi nào, tại sao
    - ✅ **Backup tự động**: Code được lưu trữ an toàn trên GitHub/GitLab
    - ✅ **Rollback dễ dàng**: Quay lại phiên bản cũ khi code bị lỗi
    - ✅ **Tuyển dụng**: Hầu hết công ty yêu cầu biết Git

## Nội dung học tập

### 📚 Git cơ bản (BẮT BUỘC)
1. **[Git Fundamentals](fundamentals.md)** ⭐ BẮT ĐẦU TẠI ĐÂY
   - Cài đặt và cấu hình Git
   - Repository, Commit, Branch
   - Git commands cơ bản: add, commit, push, pull
   - Working with remote (GitHub/GitLab)

2. **[Branching Strategies](branching.md)** ⭐ QUAN TRỌNG
   - Branch là gì và tại sao cần dùng
   - Tạo, chuyển, merge branches
   - Giải quyết conflicts
   - Branch naming conventions

### 🔀 Git Workflow (NÂNG CAO)
3. **[Git Flow](git-flow.md)** - Workflow phổ biến nhất
   - Main branches: master, develop
   - Supporting branches: feature, release, hotfix
   - Quy trình làm việc chuẩn

4. **[GitHub Flow](github-flow.md)** - Workflow đơn giản cho CI/CD
   - Main branch + feature branches
   - Pull Request workflow
   - Deploy liên tục

5. **[GitLab Flow](gitlab-flow.md)** - Kết hợp ưu điểm của cả hai
   - Environment branches
   - Issue tracking integration
   - Phù hợp với microservices

### 🛠️ Thực hành & Tips
6. **[Common Scenarios](common-scenarios.md)** - Các tình huống thực tế
   - Undo commit
   - Sửa commit message
   - Cherry-pick commits
   - Rebase vs Merge

7. **[Best Practices](best-practices.md)** - Làm sao để code sạch
   - Commit message conventions
   - Code review process
   - .gitignore best practices

## Lộ trình học Git

### Tuần 1: Cơ bản (BẮT BUỘC)
- **Ngày 1-2**: Cài đặt Git, tạo repository đầu tiên
- **Ngày 3-4**: Thực hành add, commit, push, pull
- **Ngày 5-7**: Làm việc với branches, giải quyết conflicts

### Tuần 2: Git Workflow (NÊN HỌC)
- **Ngày 8-10**: Tìm hiểu Git Flow
- **Ngày 11-12**: Thực hành GitHub Flow với Pull Requests
- **Ngày 13-14**: Project nhỏ áp dụng workflow

### Tùy chọn: Nâng cao
- **GitLab CI/CD**: Tự động hóa deployment
- **Git Hooks**: Automation với Git events
- **Advanced Git**: Rebase, Cherry-pick, Bisect

## Công cụ hỗ trợ

### GUI Clients
- **GitHub Desktop** - Đơn giản, dễ dùng cho người mới
- **SourceTree** - Miễn phí, đầy đủ tính năng
- **GitKraken** - Đẹp, trực quan nhưng có phí
- **VS Code Git Extension** - Tích hợp sẵn trong editor

### Command Line Tools
- **Git CLI** - Công cụ gốc, mạnh mẽ nhất
- **tig** - Text-mode interface for git
- **lazygit** - Terminal UI đơn giản, nhanh

## Câu hỏi thường gặp

??? question "Git khác GitHub như thế nào?"
    - **Git**: Là phần mềm quản lý version control (chạy local)
    - **GitHub**: Là dịch vụ hosting Git repositories online (giống Google Drive cho code)
    - **GitLab, Bitbucket**: Tương tự GitHub, cũng là hosting services

??? question "Nên học Git Flow hay GitHub Flow?"
    - **Dự án nhỏ, startup**: Dùng **GitHub Flow** (đơn giản hơn)
    - **Dự án lớn, nhiều môi trường**: Dùng **Git Flow** (chuẩn chỉnh hơn)
    - **Microservices**: Dùng **GitLab Flow** (tích hợp CI/CD tốt)

??? question "Commit bao nhiêu lần là đủ?"
    Commit mỗi khi hoàn thành một **đơn vị công việc nhỏ**:
    
    - ✅ ĐÚNG: Mỗi feature nhỏ 1 commit
    - ✅ ĐÚNG: Mỗi bugfix 1 commit
    - ❌ SAI: 1 tuần code xong mới commit 1 lần
    - ❌ SAI: Mỗi dòng code commit 1 lần

??? question "Viết commit message thế nào cho đúng?"
    Theo chuẩn **Conventional Commits**:
    
    ```
    <type>(<scope>): <subject>
    
    <body>
    
    <footer>
    ```
    
    **Ví dụ:**
    ```
    feat(auth): add login with Google
    
    - Implement OAuth2 flow
    - Add Google sign-in button
    - Store user session
    
    Closes #123
    ```

??? question "Làm sao biết mình đã giỏi Git?"
    Khi bạn tự tin:
    
    - ✅ Tạo branch, merge, rebase không cần Google
    - ✅ Giải quyết conflicts một cách hiệu quả
    - ✅ Undo được commit, rollback được code
    - ✅ Làm việc nhóm qua Pull Requests
    - ✅ Hiểu và áp dụng được Git Workflow

## Tài liệu tham khảo

!!! info "Resources hay"
    - **[Pro Git Book](https://git-scm.com/book/vi/v2)** - Sách Git chính thống (có tiếng Việt)
    - **[Learn Git Branching](https://learngitbranching.js.org/?locale=vi)** - Game học Git (có tiếng Việt)
    - **[GitHub Guides](https://guides.github.com/)** - Hướng dẫn từ GitHub
    - **[Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials)** - Tutorial chi tiết
    - **[Oh My Git!](https://ohmygit.org/)** - Game học Git concepts

---

**Bắt đầu với:** [Git Fundamentals →](fundamentals.md)
