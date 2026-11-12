# Workflow làm việc với Git/GitHub

## 1. Bắt đầu feature mới: cập nhật `main`

```bash
git switch main
git pull origin main
```

Mục đích: đảm bảo `main` local đang ở version mới nhất trên GitHub.

---

## 2. Tạo/chuyển sang feature branch

Nếu branch chưa có:

```bash
git switch -c feature
```

Nếu branch đã có:

```bash
git switch feature
```

---

## 3. Code + commit

```bash
git add .
git commit -m "message"
```

Có thể commit nhiều lần trong quá trình code.

---

## 4. Cập nhật feature theo `main` mới nhất trước khi Push / Pull Request

Đầu tiên:

```bash
git fetch origin
```

`git fetch origin` lấy thông tin/commit mới nhất từ GitHub về và cập nhật `origin/main`, nhưng chưa thay đổi code của branch hiện tại.

### Trường hợp A: Feature branch chỉ một người làm → dùng `rebase`

```bash
git rebase origin/main
```

Nếu không có conflict thì rebase hoàn tất, chuyển sang bước Push.

Nếu có conflict:

1. Sửa các file bị conflict.
2. Đánh dấu đã xử lý:

```bash
git add .
```

3. Tiếp tục rebase:

```bash
git rebase --continue
```

Nếu còn conflict thì tiếp tục lặp lại:

```text
sửa conflict
→ git add .
→ git rebase --continue
```

Muốn hủy toàn bộ quá trình rebase:

```bash
git rebase --abort
```

---

### Trường hợp B: Feature branch có nhiều người cùng làm → dùng `merge`

Không nên rebase branch dùng chung vì rebase thay đổi lịch sử commit.

Đang đứng ở feature:

```bash
git fetch origin
git merge origin/main
```

Nếu không có conflict thì merge hoàn tất.

Nếu có conflict:

1. Sửa các file bị conflict.
2. Đánh dấu đã xử lý:

```bash
git add .
```

3. Hoàn tất merge:

```bash
git commit
```

Hoặc:

```bash
git commit -m "merge main into feature"
```

Muốn hủy merge:

```bash
git merge --abort
```

Sau đó push bình thường:

```bash
git push
```

---

## 5. Push feature lên GitHub

### Branch chưa từng push lên remote

```bash
git push -u origin feature
```

### Branch đã push trước đó và vừa `rebase`

```bash
git push --force-with-lease
```

Do rebase đã thay đổi lịch sử commit.

### Branch đã push và không rebase

Ví dụ vừa code thêm hoặc vừa merge:

```bash
git push
```

---

## 6. Tạo Pull Request

Trên GitHub:

```text
feature
   ↓
Pull Request
   ↓
main
```

Sau đó:

```text
Review
↓
CI / Test
↓
Kiểm tra conflict
↓
Merge
```

Có thể dùng:

```text
Squash and Merge
```

để lịch sử `main` gọn hơn.

---

## 7. Sau khi Pull Request đã merge

Quay lại `main`:

```bash
git switch main
git pull origin main
```

Nếu không cần feature branch local nữa:

```bash
git branch -d feature
```

---

## Quy tắc cần nhớ

```text
Feature branch chỉ 1 người làm
→ fetch
→ rebase origin/main

Feature branch nhiều người cùng push
→ fetch
→ merge origin/main
```

Tức là:

```bash
# Branch cá nhân
git fetch origin
git rebase origin/main
```

Còn:

```bash
# Branch dùng chung
git fetch origin
git merge origin/main
```

### Workflow tổng quát

```text
main mới nhất
    ↓
tạo feature branch
    ↓
code + commit
    ↓
git fetch origin
    ↓
┌────────────────────────────┐
│ Branch cá nhân → rebase    │
│ Branch dùng chung → merge  │
└────────────────────────────┘
    ↓
xử lý conflict nếu có
    ↓
push feature
    ↓
Pull Request
    ↓
Review / CI
    ↓
Merge vào main
    ↓
switch main + pull
```
