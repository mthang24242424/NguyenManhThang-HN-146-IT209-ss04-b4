# Báo cáo Bài tập 4: Quản lý tệp tin bỏ qua (.gitignore) và Sửa lịch sử (Amend)

## 1. Mục tiêu

- Cấu hình tệp tin `.gitignore` để tự động bỏ qua các file nhạy cảm hoặc file rác hệ thống.
- Sử dụng lệnh `git rm --cached` để gỡ bỏ file đã commit ra khỏi cache theo dõi của Git một cách an toàn, không xóa file vật lý.
- Chỉnh sửa thông điệp và nội dung của commit gần nhất thông qua tùy chọn `--amend`.

---

## 2. Bối cảnh

Học viên vô tình commit nhầm file `credentials.txt` chứa thông tin bảo mật (mật khẩu, API key, AWS secret key) lên Git. Cần thực hiện 3 việc:

1. **Gỡ bỏ** `credentials.txt` khỏi sự theo dõi của Git mà **không xóa file** trên đĩa cứng.
2. **Cấu hình** `.gitignore` để Git bỏ qua file này trong tương lai.
3. **Sửa lại** thông điệp commit gần nhất cho sạch sẽ và rõ nghĩa hơn.

**Ràng buộc:** Không sử dụng lệnh xóa vật lý file trên hệ điều hành, phải giữ lại tệp tin ở thư mục làm việc cục bộ.

---

## 3. Các bước thực hiện

### Bước 1: Tạo file `credentials.txt` và commit nhầm vào Git

Tạo file `credentials.txt` chứa thông tin bảo mật (mô phỏng tình huống vô tình commit):

```text
DB_HOST=localhost
DB_USER=admin
DB_PASSWORD=SuperSecret123!
API_KEY=sk-abc123xyz789
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

Kiểm tra trạng thái trước khi commit:

```bash
git status
```

```text
On branch main
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        credentials.txt
```

Thêm vào Staging Area và commit (mô phỏng thao tác nhầm):

```bash
git add credentials.txt
git commit -m "add project config"
```

```text
[main 7e0680a] add project config
 1 file changed, 6 insertions(+)
 create mode 100644 credentials.txt
```

Kiểm tra lịch sử commit:

```bash
git log --oneline -3
```

```text
7e0680a (HEAD -> main) add project config
bbde8e1 (origin/main) bai1
8ef4cc0 btvn
```

> **Vấn đề:** Commit `7e0680a` hiện đang theo dõi `credentials.txt` chứa thông tin nhạy cảm.

---

### Bước 2: Gỡ `credentials.txt` khỏi cache theo dõi của Git

Dùng `git rm --cached` để xóa file khỏi **index (Staging Area)** mà **không xóa file** trên đĩa cứng:

```bash
git rm --cached credentials.txt
```

```text
rm 'credentials.txt'
```

Kiểm tra trạng thái sau lệnh:

```bash
git status
```

```text
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    credentials.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        credentials.txt
```

Xác nhận file vẫn còn trên đĩa cứng (không bị xóa vật lý):

```bash
# Kiểm tra file còn tồn tại
Test-Path credentials.txt
```

```text
True
```

| Trạng thái | Ý nghĩa |
|---|---|
| `deleted: credentials.txt` (Staged) | Git đã lên kế hoạch "không theo dõi" file này trong commit tiếp theo |
| `credentials.txt` (Untracked) | File vẫn còn trên đĩa cứng, chỉ là Git không quản lý nữa |

> **Sự khác biệt giữa `git rm` và `git rm --cached`:**
> - `git rm credentials.txt` — xóa file khỏi index **VÀ** xóa file vật lý trên đĩa.
> - `git rm --cached credentials.txt` — chỉ xóa khỏi index, **giữ nguyên** file vật lý.

---

### Bước 3: Tạo tệp `.gitignore` để bỏ qua file trong tương lai

Tạo file `.gitignore` tại thư mục gốc của dự án:

```bash
# Nội dung file .gitignore
```

```text
# Tệp tin chứa thông tin bảo mật
credentials.txt
*.env
.env*

# Tệp tin hệ thống Windows
Thumbs.db
desktop.ini

# Tệp tin hệ thống macOS
.DS_Store

# Thư mục build / dependency
node_modules/
dist/
*.log
```

Kiểm tra `.gitignore` đã có hiệu lực — `credentials.txt` không còn xuất hiện trong danh sách **Untracked files**:

```bash
git status
```

```text
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    credentials.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
```

> `credentials.txt` đã biến mất khỏi mục `Untracked files` — `.gitignore` đang hoạt động đúng.

Thêm `.gitignore` vào Staging Area:

```bash
git add .gitignore
```

---

### Bước 4: Sửa commit gần nhất bằng `--amend`

Tại thời điểm này Staging Area có:
- `deleted: credentials.txt` — gỡ file bảo mật khỏi tracking
- `.gitignore` (new file) — cấu hình bỏ qua

Dùng `--amend` để **thay thế** commit `7e0680a` ("add project config") bằng một commit mới sạch hơn, có cả `.gitignore` và không còn `credentials.txt`:

```bash
git commit --amend -m "chore: add .gitignore and remove credentials.txt from tracking"
```

```text
[main 57d7a70] chore: add .gitignore and remove credentials.txt from tracking
 Date: Mon Oct 5 01:42:16 2026 +0700
 1 file changed, 16 insertions(+)
 create mode 100644 .gitignore
```

> **Cơ chế `--amend`:**  
> Git không sửa commit cũ mà tạo một commit **hoàn toàn mới** với hash khác (`57d7a70` thay cho `7e0680a`), kế thừa tất cả thay đổi của commit cũ cộng với các thay đổi mới trong Staging Area. Commit cũ bị thay thế hoàn toàn trong lịch sử.

---

## 4. Lệnh kiểm tra và kết quả mong đợi

### Kiểm tra trạng thái file

```bash
git status
```

```text
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what git status will be committed)
        modified:   Bai1.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        Bai2.md
```

**Kết quả đạt:** `credentials.txt` không xuất hiện trong bất kỳ mục nào (`Staged`, `Modified`, hay `Untracked`) — Git đã hoàn toàn bỏ qua file này.

### Kiểm tra lịch sử commit gần nhất

```bash
git log -n 1
```

```text
commit 57d7a7083f85d205e603061a5e441675dc504031 (HEAD -> main)
Author: Nguyen Manh Thang <wunkevan@gmail.com>
Date:   Mon Oct 5 01:42:16 2026 +0700

    chore: add .gitignore and remove credentials.txt from tracking
```

**Kết quả đạt:** Thông điệp commit đã được sửa thành công, rõ nghĩa và phản ánh đúng nội dung thay đổi.

### Bảng tổng hợp lệnh kiểm tra

| Lệnh | Mục đích | Kết quả mong đợi |
|---|---|---|
| `git status` | Kiểm tra trạng thái file | `credentials.txt` không ở dạng Staged hay Modified hay Untracked |
| `git log -n 1` | Xem commit gần nhất | Thông điệp đã sửa hiển thị đúng |
| `Test-Path credentials.txt` | Xác nhận file vật lý | `True` — file vẫn còn trên đĩa cứng |
| `git show HEAD --name-status` | Xem nội dung commit mới | Chỉ có `.gitignore`, không có `credentials.txt` |

---

## 5. Ghi chú

- `git rm --cached <file>` chỉ gỡ file khỏi **index** (vùng theo dõi) của Git, không đụng đến file vật lý — đây là lệnh an toàn để xử lý tình huống commit nhầm file bảo mật.
- `git commit --amend` tạo commit mới thay thế commit cũ với hash khác hoàn toàn. **Không nên dùng `--amend` trên các commit đã push lên remote** vì sẽ làm diverge lịch sử với người dùng khác.
- `.gitignore` chỉ có hiệu lực với các file **chưa từng được Git theo dõi**. Nếu file đã được commit trước đó, phải dùng `git rm --cached` để gỡ khỏi index trước, rồi `.gitignore` mới phát huy tác dụng.
- Để kiểm tra một file có đang bị ignore hay không, dùng: `git check-ignore -v credentials.txt`.

---
