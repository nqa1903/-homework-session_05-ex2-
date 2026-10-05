# Bài 2: Tái cấu trúc lịch sử commit bằng Interactive Rebase

## 1. Mục tiêu

- Dùng `git rebase -i` để chỉnh sửa lịch sử commit cục bộ.
- **squash** 3 commit nhỏ thành 1 commit có ý nghĩa.
- Đổi thông điệp commit sau khi gộp thành `feat: hoan thien module authentication`.
- **drop** commit chứa file rác `temp.txt`.

## 2. Chuẩn bị: tạo nhánh tính năng với 4 commit "lộn xộn"

```bash
mkdir rebase-demo && cd rebase-demo
git init
echo "# Demo" > README.md
git add . && git commit -m "Initial commit"

git checkout -b feature/auth

# Commit 1: tạo auth.js
echo "function login() {}" > auth.js
git add . && git commit -m "feat: khoi tao module auth"

# Commit 2: sửa auth.js (typo)
echo "function logn() {}" > auth.js
git add . && git commit -m "fix typo"

# Commit 3: sửa auth.js (thêm hàm tiện ích)
printf "function login() {}\nfunction hashPassword() {}\n" > auth.js
git add . && git commit -m "adds utility functions"

# Commit 4: tạo file rác
echo "debug" > temp.txt
git add . && git commit -m "add temp file for debug"
```

Lịch sử **trước** khi rebase:

```
6752a49 add temp file for debug
6028277 adds utility functions
7895743 fix typo
7196d4c feat: khoi tao module auth
1bb3225 Initial commit
```

## 3. Thực hiện Interactive Rebase

### Bước 1: Mở giao diện rebase tương tác cho 4 commit gần nhất

```bash
git rebase -i HEAD~4
```

### Bước 2: Chỉnh danh sách lệnh trong trình soạn thảo

Git mở file todo (commit cũ nhất ở trên cùng). Danh sách ban đầu:

```
pick 7196d4c feat: khoi tao module auth
pick 7895743 fix typo
pick 6028277 adds utility functions
pick 6752a49 add temp file for debug
```

Sửa thành:

```
pick   7196d4c feat: khoi tao module auth
squash 7895743 fix typo
squash 6028277 adds utility functions
drop   6752a49 add temp file for debug
```

| Commit | Lệnh | Ý nghĩa |
|---|---|---|
| `feat: khoi tao module auth` | `pick` | Giữ nguyên, làm commit gốc để gộp vào |
| `fix typo` | `squash` | Gộp vào commit phía trên |
| `adds utility functions` | `squash` | Gộp vào commit phía trên |
| `add temp file for debug` | `drop` | Xóa hoàn toàn commit chứa `temp.txt` |

> 📸 **Ảnh chụp giao diện soạn thảo rebase (pick / squash / drop):**
>
> ![Giao diện rebase -i](images/rebase-todo.png)

Lưu và đóng trình soạn thảo.

### Bước 3: Soạn lại thông điệp cho commit đã gộp

Git mở tiếp trình soạn thảo, hiển thị thông điệp của các commit được gộp:

```
feat: khoi tao module auth
fix typo
adds utility functions
```

Xóa toàn bộ nội dung này, thay bằng một dòng:

```
feat: hoan thien module authentication
```

> 📸 **Ảnh chụp màn hình soạn thảo thông điệp commit:**
>
> ![Soạn thông điệp commit](images/rebase-message.png)

Lưu và đóng. Git báo kết quả:

```
[detached HEAD 34f645c] feat: hoan thien module authentication
 1 file changed, 2 insertions(+)
 create mode 100644 auth.js
Successfully rebased and updated refs/heads/feature/auth.
```

## 4. Kiểm tra kết quả

```bash
git log --oneline
```

```
34f645c feat: hoan thien module authentication
1bb3225 Initial commit
```

```bash
ls
cat auth.js
```

```
README.md
auth.js
function login() {}
function hashPassword() {}
```

- Nhánh tính năng chỉ còn **1 commit duy nhất**: `feat: hoan thien module authentication` (commit `Initial commit` bên dưới là commit nền của repo, không thuộc tính năng).
- Không còn các commit `fix typo`, `adds utility functions`, `add temp file for debug`.
- File `temp.txt` không còn tồn tại; `auth.js` giữ nội dung cuối cùng sau khi gộp.

> 📸 **Ảnh chụp `git log --oneline` sau khi hoàn tất:**
>
> ![Git log sau rebase](images/git-log-after.png)

## 5. Giải thích các lệnh trong Interactive Rebase

| Lệnh | Viết tắt | Tác dụng |
|---|---|---|
| `pick` | `p` | Giữ nguyên commit |
| `reword` | `r` | Giữ nội dung, chỉ đổi thông điệp |
| `squash` | `s` | Gộp vào commit trước, cho phép soạn lại thông điệp |
| `drop` | `d` | Xóa bỏ commit (bao gồm thay đổi của nó) |

- Rebase viết lại lịch sử nên các commit mới có **hash khác** commit cũ.
- Chỉ nên rebase các commit **chưa push** lên remote. Nếu đã push, cần `git push --force-with-lease` và có thể ảnh hưởng người khác.
- Nếu rebase gặp sự cố, dùng `git rebase --abort` để quay lại trạng thái trước khi rebase (hoặc dùng `git reflog` như Bài 1 để tìm lại).
