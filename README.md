# Bài 5: Khôi phục trạng thái và Đảo ngược commit (Reset vs Revert)

## 1. Mục tiêu

- Thực hành sử dụng `git reset` để di chuyển HEAD về commit trước.
- Giữ lại mã nguồn hiện hành để tiếp tục chỉnh sửa.
- Thực hành sử dụng `git revert` để đảo ngược một commit đã tồn tại.
- Phân biệt `reset` và `revert` khi làm việc cá nhân và làm việc nhóm.

---

## 2. Trường hợp 1: Sử dụng Git Reset

### 2.1. Bối cảnh

Học viên có một commit chứa mã nguồn bị lỗi và muốn quay lại trạng thái trước commit đó.

Tuy nhiên, yêu cầu là:

- Không làm mất mã nguồn hiện tại.
- HEAD lùi lại 1 commit.
- Các thay đổi của commit bị lỗi được giữ lại trong Working Directory.
- Các file thay đổi ở trạng thái `Modified`.

### 2.2. Kiểm tra lịch sử trước khi Reset

Sử dụng:

```bash
git log --oneline -n 5
```

Ví dụ:

```text
6c69e98 (HEAD -> master) Add: chức năng mới
25e6802 Add: chức năng mới
fc4ac26 Initial Commit
```

Trong trường hợp này, commit `6c69e98` là commit cần quay lại.

### 2.3. Thực hiện Reset

Sử dụng:

```bash
git reset HEAD~1
```

Hoặc viết đầy đủ:

```bash
git reset --mixed HEAD~1
```

`--mixed` là chế độ mặc định của `git reset`.

Lệnh này thực hiện:

1. Di chuyển `HEAD` về commit trước đó.
2. Cập nhật lại Staging Area.
3. Giữ nguyên nội dung file trong Working Directory.
4. Các thay đổi của commit vừa reset trở thành `Modified`.

### 2.4. Kiểm tra kết quả

```bash
git status
```

Ví dụ:

```text
Changes not staged for commit:
    modified:   src/main/java/...
```

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Commit bị reset không còn nằm trên đầu nhánh hiện tại.

### 2.5. Kết quả

Sau khi thực hiện:

```bash
git reset HEAD~1
```

Ta có:

```text
HEAD
 |
 v
Commit trước
```

Trong khi nội dung của commit bị reset vẫn còn trong Working Directory:

```text
Working Directory
    |
    +-- modified files
```

Mã nguồn không bị mất và có thể tiếp tục chỉnh sửa.

### 2.6. Không sử dụng Reset Hard

Không sử dụng:

```bash
git reset --hard HEAD~1
```

Vì `--hard` sẽ đưa cả Working Directory về trạng thái của commit trước đó và có thể làm mất các thay đổi hiện tại.

---

# 3. Trường hợp 2: Sử dụng Git Revert

## 3.1. Bối cảnh

Một commit đã được chia sẻ lên remote hoặc đang nằm trên nhánh chung của nhóm.

Không nên thay đổi lịch sử commit đã công khai bằng `reset`.

Thay vào đó, sử dụng:

```bash
git revert
```

Git sẽ tạo ra một commit mới để đảo ngược các thay đổi của commit cũ.

---

## 3.2. Kiểm tra lịch sử

Sử dụng:

```bash
git log --oneline -n 5
```

Ví dụ:

```text
1b78dfb Fix: code
25e6802 Add: chức năng mới
fc4ac26 Initial Commit
```

Giả sử cần đảo ngược commit:

```text
1b78dfd
```

---

## 3.3. Thực hiện Revert

Sử dụng:

```bash
git revert 1b78dfd
```

Git sẽ tạo ra một commit mới có nội dung đảo ngược các thay đổi của commit `1b78dfd`.

Nếu Git mở trình soạn thảo để nhập commit message, có thể giữ thông điệp mặc định.

Có thể sử dụng trực tiếp:

```bash
git revert --no-edit 1b78dfd
```

---

## 3.4. Kiểm tra lịch sử sau Revert

```bash
git log --oneline -n 5
```

Ví dụ kết quả:

```text
1b1587e (HEAD -> master) Revert "Fix: code"
1b78dfb Fix: code
25e6802 Add: chức năng mới
fc4ac26 Initial Commit
```

Commit `Revert "Fix: code"` là commit mới được tạo ra.

Điều này chứng minh Git đã đảo ngược thay đổi của commit cũ mà không xóa commit cũ khỏi lịch sử.

---

## 3.5. Kiểm tra trạng thái

```bash
git status
```

Nếu không còn thay đổi chưa commit:

```text
nothing to commit, working tree clean
```

Có thể đẩy commit revert lên remote:

```bash
git push origin <ten-nhanh>
```

Ví dụ:

```bash
git push origin main
```

---

# 4. So sánh Reset và Revert

| Tiêu chí | `git reset` | `git revert` |
|---|---|---|
| Cách hoạt động | Di chuyển HEAD về commit khác | Tạo commit mới để đảo ngược |
| Tạo commit mới | Không | Có |
| Thay đổi lịch sử | Có | Không |
| Commit cũ còn trong lịch sử nhánh? | Có thể không còn trên nhánh | Có |
| Phù hợp với commit local | Có | Có |
| Phù hợp với nhánh đã push | Không nên | Nên |
| Phù hợp nhánh dùng chung | Không nên | An toàn |
| Có thể làm mất thay đổi? | Có, nếu dùng `--hard` | Thông thường không |
| Mục đích chính | Viết lại lịch sử | Đảo ngược thay đổi |

---

# 5. Reset khi làm việc cá nhân

Khi commit chỉ tồn tại ở local và chưa được chia sẻ cho người khác, có thể sử dụng:

```bash
git reset HEAD~1
```

Điều này cho phép quay lại trạng thái trước commit và tiếp tục chỉnh sửa code.

Ví dụ:

```text
Trước Reset:

A --- B --- C
          ^
         HEAD

Sau Reset:

A --- B
      ^
     HEAD

Code của C vẫn nằm trong Working Directory
```

Đây là trường hợp phù hợp để sử dụng `reset`.

---

# 6. Revert khi làm việc nhóm

Khi commit đã được push lên remote hoặc nằm trên nhánh chung như `main`, không nên tùy tiện sử dụng:

```bash
git reset
```

và sau đó force push để thay đổi lịch sử chung.

Thay vào đó:

```bash
git revert <commit-id>
```

Ví dụ:

```bash
git revert a1b2c3d
```

Kết quả:

```text
A --- B --- C --- D
              lỗi   revert
```

Trong đó:

- `C` là commit chứa code lỗi.
- `D` là commit mới đảo ngược thay đổi của `C`.
- Lịch sử commit vẫn được giữ nguyên.
- Thành viên khác trong nhóm có thể đồng bộ lịch sử mà không bị thay đổi lịch sử bất ngờ.

---

# 7. Quy trình thực hiện bài

## Trường hợp 1 - Reset

```bash
git log --oneline -n 5
git reset HEAD~1
git status
git log --oneline -n 5
```

Kết quả cần đạt:

- HEAD lùi lại 1 commit.
- Mã nguồn vẫn được giữ lại.
- File thay đổi xuất hiện dưới dạng `Modified`.
- Không sử dụng `git reset --hard`.

## Trường hợp 2 - Revert

```bash
git log --oneline -n 5
git revert <commit-id>
git status
git log --oneline -n 5
```

Kết quả cần đạt:

```text
xxxxxxx Revert "..."
xxxxxxx Commit cần đảo ngược
xxxxxxx Commit trước đó
```

Commit bắt đầu bằng `Revert` chứng minh việc đảo ngược đã được thực hiện bằng `git revert`.

---

# 8. Kết luận

`git reset` và `git revert` đều có thể được sử dụng để xử lý commit chứa lỗi nhưng có mục đích khác nhau.

- `git reset` thích hợp khi làm việc cá nhân và muốn viết lại lịch sử local.
- `git reset HEAD~1` hoặc `git reset --mixed HEAD~1` giúp di chuyển HEAD nhưng vẫn giữ lại thay đổi trong Working Directory.
- `git revert` tạo một commit mới để đảo ngược commit cũ.
- Khi commit đã được push lên remote hoặc nằm trên nhánh chung, nên sử dụng `git revert` để tránh thay đổi lịch sử mà các thành viên khác đang sử dụng.

Do đó:

```text
Local / chưa chia sẻ  →  git reset
Đã push / nhánh chung →  git revert
```