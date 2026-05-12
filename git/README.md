# 🌿 Git Cheatsheet

> Tổng hợp các lệnh Git thường dùng — có ví dụ thực tế và các tình huống hay gặp.

![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

---

## 📂 Mục lục

- [Cài đặt ban đầu](#-cài-đặt-ban-đầu)
- [Khởi tạo repo](#-khởi-tạo-repo)
- [Các lệnh cơ bản hàng ngày](#-các-lệnh-cơ-bản-hàng-ngày)
- [Branch](#-branch)
- [Tình huống thực tế](#-tình-huống-thực-tế)
  - [Thu hồi sau khi add / commit](#1-thu-hồi-sau-khi-add--commit)
  - [Lấy code mới từ đồng nghiệp](#2-lấy-code-mới-từ-đồng-nghiệp)
  - [Xem đồng nghiệp thay đổi gì](#3-xem-đồng-nghiệp-thay-đổi-gì)
  - [Giải quyết conflict](#4-giải-quyết-conflict)
  - [Lỡ commit nhầm branch](#5-lỡ-commit-nhầm-branch)
  - [Xem lại lịch sử commit](#6-xem-lại-lịch-sử-commit)
  - [Quay về commit cũ](#7-quay-về-commit-cũ)
  - [Stash — tạm cất code chưa xong](#8-stash--tạm-cất-code-chưa-xong)
  - [Xóa branch đã merge](#9-xóa-branch-đã-merge)
  - [Tag version](#10-tag-version)
- [Bảng tổng hợp lệnh](#-bảng-tổng-hợp-lệnh)

---

## ⚙️ Cài đặt ban đầu

> Làm 1 lần duy nhất sau khi cài Git.

```bash
git config --global user.name "Tên của bạn"
git config --global user.email "email@gmail.com"

# Kiểm tra lại
git config --list
```

---

## 🚀 Khởi tạo repo

```bash
# Tạo repo mới từ folder local
git init
git remote add origin https://github.com/username/repo-name.git
git push -u origin main

# Clone repo có sẵn về máy
git clone https://github.com/username/repo-name.git

# Clone về và đặt tên folder khác
git clone https://github.com/username/repo-name.git ten-folder-moi
```

---

## 📋 Các lệnh cơ bản hàng ngày

```bash
# Xem trạng thái hiện tại
git status

# Thêm file vào staging
git add .                  # Tất cả file
git add ten-file.txt       # Từng file cụ thể
git add src/               # Cả folder

# Commit
git commit -m "mô tả thay đổi"

# Push lên GitHub
git push origin main
git push origin ten-branch

# Xem lịch sử commit (gọn)
git log --oneline
```

---

## 🌿 Branch

```bash
# Xem danh sách branch
git branch           # local
git branch -r        # remote
git branch -a        # tất cả

# Tạo branch mới
git branch ten-branch

# Tạo và chuyển sang branch mới luôn
git checkout -b ten-branch
# hoặc cách mới hơn
git switch -c ten-branch

# Chuyển branch
git checkout ten-branch
git switch ten-branch      # cách mới

# Merge branch vào main
git checkout main
git merge ten-branch

# Xóa branch local
git branch -d ten-branch   # xóa nếu đã merge
git branch -D ten-branch   # xóa bắt buộc

# Xóa branch remote
git push origin --delete ten-branch
```

---

## 🔥 Tình huống thực tế

---

### 1. Thu hồi sau khi `add` / `commit`

#### ❌ Lỡ `git add .` muốn bỏ staging

```bash
# Bỏ tất cả khỏi staging (giữ nguyên code)
git reset HEAD .

# Bỏ 1 file cụ thể khỏi staging
git reset HEAD ten-file.txt
```

#### ❌ Lỡ `git commit` muốn sửa lại message

```bash
# Sửa message của commit vừa rồi (chưa push)
git commit --amend -m "message mới"
```

#### ❌ Lỡ `git commit` muốn bỏ commit đó (giữ code)

```bash
# Bỏ commit nhưng GIỮ NGUYÊN code (code vẫn còn ở local)
git reset --soft HEAD~1

# Bỏ commit và đưa code về unstaged
git reset HEAD~1

# Bỏ commit và XÓA LUÔN code (cẩn thận!)
git reset --hard HEAD~1
```

> 💡 `HEAD~1` = lùi lại 1 commit. `HEAD~2` = lùi lại 2 commit.

#### ❌ Đã push lên GitHub rồi muốn thu hồi

```bash
# Tạo 1 commit mới để đảo ngược commit cũ (an toàn, không xóa lịch sử)
git revert HEAD
git push origin main

# Nếu muốn revert 1 commit cụ thể
git log --oneline           # lấy commit id
git revert abc1234          # revert commit đó
```

---

### 2. Lấy code mới từ đồng nghiệp

```bash
# Cách 1: fetch + xem trước rồi mới merge (an toàn hơn)
git fetch origin            # tải về nhưng chưa merge
git diff main origin/main   # xem khác nhau gì
git merge origin/main       # merge vào local

# Cách 2: pull thẳng (fetch + merge luôn)
git pull origin main

# Cách 3: pull --rebase (lịch sử gọn hơn)
git pull --rebase origin main
```

> 💡 **Khuyến nghị:** dùng `fetch` trước để xem đồng nghiệp thay đổi gì, rồi mới `merge` — tránh bị conflict bất ngờ.

---

### 3. Xem đồng nghiệp thay đổi gì

```bash
# Xem ai commit gì, lúc nào
git log --oneline
git log --oneline --graph --all   # dạng cây, thấy cả branch

# Xem chi tiết 1 commit cụ thể
git show abc1234

# Xem file nào thay đổi trong commit đó
git show --stat abc1234

# So sánh code giữa 2 branch
git diff main feature/ten-branch

# Xem ai sửa dòng nào trong file
git blame ten-file.txt

# Xem lịch sử thay đổi của 1 file cụ thể
git log --oneline -- ten-file.txt
```

---

### 4. Giải quyết conflict

> Conflict xảy ra khi 2 người sửa cùng 1 chỗ trong file.

```bash
# Sau khi pull/merge bị conflict
git status                  # xem file nào đang conflict
```

Mở file bị conflict, Git sẽ đánh dấu như sau:

```
<<<<<<< HEAD
code của bạn
=======
code của đồng nghiệp
>>>>>>> origin/main
```

**Xử lý:**
1. Xóa các dấu `<<<<<<<`, `=======`, `>>>>>>>>`
2. Giữ lại code đúng (của bạn, của đồng nghiệp, hoặc kết hợp cả hai)
3. Lưu file lại

```bash
# Sau khi sửa xong conflict
git add .
git commit -m "fix: resolve conflict"
git push origin main
```

---

### 5. Lỡ commit nhầm branch

> Tình huống: đang ở `main` mà commit nhầm, lẽ ra phải commit vào `feature/abc`.

```bash
# Bước 1: Lấy commit id vừa commit nhầm
git log --oneline

# Bước 2: Chuyển sang branch đúng
git checkout feature/abc

# Bước 3: Mang commit đó sang branch này
git cherry-pick abc1234

# Bước 4: Quay lại main và xóa commit nhầm
git checkout main
git reset --hard HEAD~1    # xóa commit nhầm khỏi main
```

---

### 6. Xem lại lịch sử commit

```bash
# Xem ngắn gọn
git log --oneline

# Xem dạng cây đẹp
git log --oneline --graph --all --decorate

# Xem chi tiết (tác giả, ngày, nội dung thay đổi)
git log -p

# Xem N commit gần nhất
git log --oneline -5        # 5 commit gần nhất

# Tìm commit theo message
git log --oneline --grep="fix bug"

# Tìm commit theo tác giả
git log --oneline --author="Nguyen Van A"
```

---

### 7. Quay về commit cũ

```bash
# Xem danh sách commit
git log --oneline

# Quay về commit cũ để xem (không sửa gì)
git checkout abc1234

# Quay lại main sau khi xem xong
git checkout main

# Quay về hẳn commit cũ (xóa toàn bộ thay đổi sau đó — CẨN THẬN)
git reset --hard abc1234
```

---

### 8. Stash — tạm cất code chưa xong

> Tình huống: đang code dở, sếp bảo phải chuyển sang branch khác gấp, mà chưa muốn commit.

```bash
# Cất code hiện tại vào stash
git stash

# Cất kèm message để dễ nhớ
git stash save "dang lam tinh nang login"

# Xem danh sách stash
git stash list

# Lấy stash ra dùng lại (stash gần nhất)
git stash pop

# Lấy stash cụ thể
git stash pop stash@{1}

# Xem nội dung stash trước khi lấy ra
git stash show stash@{0}

# Xóa stash không cần nữa
git stash drop stash@{0}

# Xóa toàn bộ stash
git stash clear
```

---

### 9. Xóa branch đã merge

```bash
# Xem branch nào đã merge vào main
git branch --merged

# Xóa branch local đã merge
git branch -d ten-branch

# Xóa hàng loạt branch local đã merge (trừ main và develop)
git branch --merged | grep -v "main\|develop\|master" | xargs git branch -d

# Xóa branch remote
git push origin --delete ten-branch
```

---

### 10. Tag version

```bash
# Tạo tag
git tag v1.0.0

# Tạo tag kèm message
git tag -a v1.0.0 -m "Release version 1.0.0"

# Xem danh sách tag
git tag

# Push tag lên GitHub
git push origin v1.0.0
git push origin --tags     # push tất cả tag

# Xóa tag local
git tag -d v1.0.0

# Xóa tag remote
git push origin --delete v1.0.0
```

---

## 📌 Bảng tổng hợp lệnh

| Lệnh | Mô tả |
|------|-------|
| `git status` | Xem trạng thái hiện tại |
| `git add .` | Thêm tất cả vào staging |
| `git commit -m "msg"` | Commit với message |
| `git push origin main` | Push lên GitHub |
| `git pull origin main` | Lấy code mới về |
| `git fetch origin` | Tải về nhưng chưa merge |
| `git log --oneline` | Xem lịch sử commit |
| `git diff` | Xem thay đổi chưa stage |
| `git branch -b ten` | Tạo và chuyển branch |
| `git merge ten` | Merge branch vào branch hiện tại |
| `git stash` | Cất code tạm thời |
| `git stash pop` | Lấy code đã cất ra |
| `git reset --soft HEAD~1` | Bỏ commit, giữ code |
| `git reset --hard HEAD~1` | Bỏ commit, xóa code |
| `git revert HEAD` | Tạo commit đảo ngược |
| `git cherry-pick id` | Lấy 1 commit từ branch khác |
| `git blame file` | Xem ai sửa dòng nào |

---

> 📌 Tài liệu được cập nhật liên tục theo quá trình học.
