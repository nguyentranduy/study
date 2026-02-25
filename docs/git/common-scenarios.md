# Common Scenarios - Tình huống thực tế

Tổng hợp các tình huống thường gặp khi làm việc với Git và cách xử lý.

## 1. Undo Changes

### Undo Working Directory Changes

```bash
# Bỏ changes của 1 file (chưa add)
git checkout -- filename.txt

# Hoặc dùng lệnh mới hơn
git restore filename.txt

# Bỏ tất cả changes (NGUY HIỂM!)
git checkout -- .
git restore .
```

### Undo Staged Changes

```bash
# Unstage file (nhưng giữ changes)
git reset HEAD filename.txt

# Hoặc dùng lệnh mới
git restore --staged filename.txt

# Unstage tất cả
git reset HEAD
git restore --staged .
```

### Undo Last Commit

```bash
# Undo commit, giữ changes trong working directory
git reset --soft HEAD~1

# Undo commit, giữ changes staged
git reset --mixed HEAD~1  # hoặc git reset HEAD~1

# Undo commit, bỏ luôn changes (NGUY HIỂM!)
git reset --hard HEAD~1
```

### Undo Multiple Commits

```bash
# Undo 3 commits gần nhất
git reset --soft HEAD~3

# Undo đến một commit cụ thể
git reset --soft abc123d
```

!!! warning "Reset vs Revert"
    - **`git reset`**: Xóa commits khỏi history (dùng cho local)
    - **`git revert`**: Tạo commit mới để undo (dùng cho commits đã push)

### Revert Pushed Commits

```bash
# Revert 1 commit (tạo commit mới)
git revert abc123d

# Revert nhiều commits
git revert abc123d..def456e

# Revert nhưng không commit ngay
git revert --no-commit abc123d
# ... review changes ...
git commit -m "revert: undo payment feature"
```

## 2. Sửa Commits

### Sửa Commit Message

```bash
# Sửa commit cuối cùng (chưa push)
git commit --amend

# Sửa message mà không thay đổi code
git commit --amend -m "fix: correct typo in commit message"

# Sửa commits đã push (cần force push - nguy hiểm!)
git commit --amend
git push --force-with-lease origin branch-name
```

### Thêm Files vào Commit Cuối

```bash
# Quên add file
git add forgotten-file.txt
git commit --amend --no-edit

# Hoặc với message mới
git commit --amend -m "feat: add user profile (complete)"
```

### Sửa Commit Cũ hơn

```bash
# Interactive rebase
git rebase -i HEAD~3  # Sửa 3 commits gần nhất

# Editor mở ra:
pick abc123d feat: add login
pick def456e feat: add logout
pick ghi789f feat: add profile

# Đổi 'pick' thành 'edit' cho commit muốn sửa:
edit def456e feat: add logout

# Git sẽ stop ở commit đó
# ... make changes ...
git add .
git commit --amend
git rebase --continue
```

## 3. Stash - Cất Changes tạm thời

### Basic Stash

```bash
# Lưu changes tạm
git stash

# Hoặc với message
git stash save "WIP: working on login feature"

# Xem danh sách stashes
git stash list

# Apply stash gần nhất (giữ stash)
git stash apply

# Apply và xóa stash
git stash pop

# Apply stash cụ thể
git stash apply stash@{2}
```

### Advanced Stash

```bash
# Stash cả untracked files
git stash -u

# Stash tất cả (kể cả ignored files)
git stash -a

# Stash một phần (interactive)
git stash -p

# Tạo branch từ stash
git stash branch feature/from-stash

# Xóa stash
git stash drop stash@{0}

# Xóa tất cả stashes
git stash clear
```

### Use Case: Chuyển nhánh nhanh

```bash
# Đang code feature A
# Đột nhiên cần fix bug khẩn

git stash save "WIP: feature A halfway"
git checkout main
git checkout -b hotfix/critical-bug

# ... fix bug ...
git commit -am "fix: critical bug"

# Quay lại feature A
git checkout feature/feature-A
git stash pop
```

## 4. Cherry-pick - Copy Commit

### Basic Cherry-pick

```bash
# Copy 1 commit từ branch khác
git cherry-pick abc123d

# Copy nhiều commits
git cherry-pick abc123d def456e ghi789f

# Copy range of commits
git cherry-pick abc123d..ghi789f
```

### Cherry-pick với Conflicts

```bash
# Cherry-pick gặp conflict
git cherry-pick abc123d
# CONFLICT!

# Giải quyết conflicts
# ... fix conflicts ...
git add .
git cherry-pick --continue

# Hoặc abort
git cherry-pick --abort
```

### Use Case: Hotfix

```bash
# Fix bug trên develop
git checkout develop
git checkout -b fix/payment-bug
# ... fix ...
git commit -am "fix: payment calculation error"

# Cần fix tương tự trên production
git checkout main
git cherry-pick <commit-hash>
git push origin main
```

## 5. Rebase - Viết lại History

### Interactive Rebase

```bash
# Cleanup commits trước khi merge PR
git rebase -i HEAD~5

# Editor mở:
pick abc123d feat: add login form
pick def456e fix: typo in login
pick ghi789f feat: add validation
pick jkl012m fix: another typo
pick mno345p feat: add submit button

# Có thể:
# - squash: gộp vào commit trước
# - fixup: gộp và bỏ message
# - reword: đổi message
# - edit: sửa commit
# - drop: xóa commit
```

### Squash Commits

```bash
# Gộp 5 commits thành 1
git rebase -i HEAD~5

# Đổi thành:
pick abc123d feat: add login form
squash def456e fix: typo in login
squash ghi789f feat: add validation
squash jkl012m fix: another typo
squash mno345p feat: add submit button

# Save và đóng editor
# Git prompt cho combined message
```

### Rebase onto Main

```bash
# Update feature branch với main
git checkout feature/my-feature
git rebase main

# Nếu có conflicts
# ... resolve conflicts ...
git add .
git rebase --continue

# Hoặc abort
git rebase --abort

# Push sau rebase (cần force)
git push --force-with-lease origin feature/my-feature
```

!!! danger "Rebase Golden Rule"
    **KHÔNG rebase commits đã push lên public branch!**
    
    ✅ OK: Rebase local feature branch  
    ❌ NGUY HIỂM: Rebase main/develop branch

## 6. Merge Conflicts

### Identify Conflicts

```bash
# Sau khi merge/rebase conflict
git status

# Xem conflicts
git diff

# Xem changes từ các branches
git diff --ours    # Changes from current branch
git diff --theirs  # Changes from incoming branch
```

### Resolve Conflicts

```java
// File có conflict
<<<<<<< HEAD (Current Change)
public void calculateTotal() {
    return price * quantity;
}
=======
public Money calculateTotal() {
    return Money.of(price).multiply(quantity);
}
>>>>>>> feature/use-money-type (Incoming Change)
```

**Strategies:**
```bash
# 1. Manual: Sửa file và chọn code nào giữ

# 2. Accept ours (keep current)
git checkout --ours filename.java
git add filename.java

# 3. Accept theirs (take incoming)
git checkout --theirs filename.java
git add filename.java

# 4. Use merge tool
git mergetool
```

### Prevent Conflicts

```bash
# Update thường xuyên
git fetch origin
git rebase origin/main  # hoặc git merge origin/main

# Communicate với team
# - Ai làm file nào
# - Ai merge vào main trước
```

## 7. Reflog - Git's Safety Net

### View Reflog

```bash
# Xem lịch sử tất cả operations
git reflog

# Xem reflog của branch cụ thể
git reflog show feature/my-feature

# Output:
# abc123d HEAD@{0}: commit: feat: add login
# def456e HEAD@{1}: checkout: moving from main to feature
# ghi789f HEAD@{2}: merge: fast-forward
```

### Recover Lost Commits

```bash
# Trường hợp: Đã reset --hard nhầm
# Commits "mất" nhưng vẫn trong reflog

git reflog
# Tìm commit muốn recover: abc123d

git checkout abc123d
# Hoặc tạo branch
git checkout -b recovery/lost-commits abc123d

# Hoặc reset về đó
git reset --hard abc123d
```

### Recover Deleted Branch

```bash
# Xóa nhầm branch
git branch -D feature/important

# Tìm lại trong reflog
git reflog
# Tìm commit cuối của branch đó: def456e

# Tạo lại branch
git checkout -b feature/important def456e
```

## 8. Bisect - Tìm Bug

### Use Case: Bug xuất hiện không biết ở commit nào

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good abc123d

# Git checkout middle commit
# Test...

# If still has bug
git bisect bad

# If no bug
git bisect good

# Repeat until find the culprit commit

# Reset
git bisect reset
```

### Automated Bisect

```bash
# Với automated test
git bisect start HEAD abc123d
git bisect run npm test

# Git tự động test và tìm bad commit
```

## 9. Clean Up

### Remove Untracked Files

```bash
# Xem files sẽ bị xóa (dry run)
git clean -n

# Xóa untracked files
git clean -f

# Xóa cả directories
git clean -fd

# Xóa cả ignored files
git clean -fdx
```

### Delete Merged Branches

```bash
# Xem branches đã merge
git branch --merged main

# Xóa local branches đã merge
git branch --merged main | grep -v "main" | xargs git branch -d

# Xóa remote branches đã merge
git branch -r --merged main | grep -v main | sed 's/origin\///' | xargs -I {} git push origin --delete {}
```

### Prune Remote Branches

```bash
# Xóa references đến remote branches đã không còn
git fetch --prune

# Hoặc
git remote prune origin
```

## 10. Large Files & History

### Remove Large File from History

```bash
# Find large files
git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -10 | awk '{print$1}')"

# Remove file from history (BFG Cleaner - easier)
java -jar bfg.jar --delete-files large-file.zip .git
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Or use git-filter-repo (newer, better)
git filter-repo --path large-file.zip --invert-paths
```

### Git LFS for Large Files

```bash
# Install Git LFS
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.mp4"

# Commit .gitattributes
git add .gitattributes
git commit -m "chore: add LFS tracking"

# Add large files as usual
git add large-video.mp4
git commit -m "docs: add demo video"
```

## 11. Submodules

### Add Submodule

```bash
# Add external repo as submodule
git submodule add https://github.com/user/library.git libs/library

# Commit
git add .gitmodules libs/library
git commit -m "chore: add library submodule"
```

### Clone with Submodules

```bash
# Clone và init submodules
git clone --recursive https://github.com/user/project.git

# Hoặc sau khi clone
git submodule init
git submodule update
```

### Update Submodules

```bash
# Update submodule to latest
cd libs/library
git pull origin main
cd ../..
git add libs/library
git commit -m "chore: update library to latest"
```

## 12. Aliases cho Scenarios thường dùng

```bash
# Undo last commit
git config --global alias.undo 'reset HEAD~1 --mixed'

# Amend without editing message
git config --global alias.amend 'commit --amend --no-edit'

# Show what will be pushed
git config --global alias.ready 'rebase -i @{u}'

# Clean branches
git config --global alias.cleanup '!git branch --merged main | grep -v main | xargs git branch -d'

# Pretty log
git config --global alias.lg 'log --graph --oneline --decorate --all'

# Show files in last commit
git config --global alias.last 'show --name-only'

# Stash with message
git config --global alias.save '!git stash save'

# List stashes
git config --global alias.stashes 'stash list'
```

## Checklist

- [ ] Biết undo changes ở 3 stages: working, staged, committed
- [ ] Thành thạo git stash
- [ ] Biết khi nào dùng cherry-pick
- [ ] Hiểu rebase interactive và cách squash commits
- [ ] Tự tin giải quyết merge conflicts
- [ ] Biết dùng reflog để recover
- [ ] Đã thực hành ít nhất 5 scenarios

## Tiếp theo

- **[Best Practices →](best-practices.md)** - Practices tốt nhất
- **[Git Flow →](git-flow.md)** - Workflow chuẩn
- **[GitHub Flow →](github-flow.md)** - Workflow đơn giản

## Tài liệu tham khảo

- **[Git Flight Rules](https://github.com/k88hudson/git-flight-rules)** - Giải pháp cho mọi tình huống
- **[Oh Shit, Git!?!](https://ohshitgit.com/)** - Quick fixes cho lỗi thường gặp
- **[Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)** - Tutorial chi tiết
