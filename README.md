# Bài 1: Khôi phục commit đã mất bằng Git Reflog

## Mục tiêu
- Hiểu cách Git ghi lại lịch sử di chuyển của các tham chiếu (HEAD, nhánh) thông qua **Reflog**.
- Dùng `git reflog` để tìm lại mã hash của commit đã "biến mất".
- Khôi phục commit bị mất sau `git reset --hard HEAD~1`, **không viết lại code thủ công**.

## Môi trường
- Windows 10, Git Bash (MINGW64)
- Thư mục thực hành: `/d/rikkei/IT209/Ss5/Ls1`

---

## Các bước thực hiện

### Bước 1 – Khởi tạo dự án thử nghiệm

```bash
mkdir -p /d/rikkei/IT209/Ss5/Ls1 && cd /d/rikkei/IT209/Ss5/Ls1
git init -b main
git config --local user.name "thang-260704"
git config --local user.email "phanhtran0501@gmail.com"
echo "Session 05 - Reflog demo" > app.txt
git add .
git commit -m "Initial commit"
```

Repo cần có một commit nền trước, để lệnh `HEAD~1` ở bước sau có commit cha để lùi về.

### Bước 2 – Tạo commit quan trọng

```bash
echo "Day la tinh nang quan trong" > feature.txt
git add .
git commit -m "Them tinh nang quan trong"
git log --oneline
```

```text
eeaa035 (HEAD -> main) Them tinh nang quan trong
585c467 Initial commit
```

### Bước 3 – Giả lập sự cố: `reset --hard` làm mất commit

```bash
git reset --hard HEAD~1
git log --oneline
ls
```

```text
HEAD is now at 585c467 Initial commit

585c467 (HEAD -> main) Initial commit

app.txt
```

**Phát hiện mất commit:**
- `git log` chỉ còn `Initial commit`, commit `Them tinh nang quan trong` đã biến mất.
- File `feature.txt` cũng bị xoá khỏi thư mục làm việc, vì `--hard` đặt lại cả Staging Area lẫn Working Directory.

![Trước và sau khi reset --hard](images/01-reset-lost.png)

### Bước 4 – Tra cứu Reflog

```bash
git reflog
```

```text
585c467 (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
eeaa035 HEAD@{1}: commit: Them tinh nang quan trong
585c467 HEAD@{2}: commit (initial): Initial commit
```

Dòng `HEAD@{1}: commit: Them tinh nang quan trong` cho biết mã hash của commit bị mất là **`eeaa035`**.

### Bước 5 – Khôi phục commit

```bash
git reset --hard eeaa035
```

```text
HEAD is now at eeaa035 Them tinh nang quan trong
```

Có thể dùng cách viết tương đương `git reset --hard HEAD@{1}`.

### Bước 6 – Kiểm tra kết quả

```bash
git log --oneline
cat feature.txt
git reflog
```

```text
eeaa035 (HEAD -> main) Them tinh nang quan trong
585c467 Initial commit

Day la tinh nang quan trong
```

![Reflog và khôi phục thành công](images/02-reflog-restore.png)

---

## Giải thích cơ chế

**Vì sao commit "mất" nhưng vẫn khôi phục được?**

`git reset --hard HEAD~1` chỉ **di chuyển con trỏ nhánh `main`** lùi về commit cha. Commit `Them tinh nang quan trong` **không bị xoá** khỏi cơ sở dữ liệu của Git (`.git/objects`). Nó chỉ trở thành commit "mồ côi" (unreachable), tức là không còn nhánh nào trỏ tới nên `git log` không hiển thị.

**Reflog là gì?**

Reflog (reference log) là nhật ký **cục bộ** ghi lại mọi lần HEAD hoặc nhánh di chuyển: commit, reset, checkout, merge, rebase… Mỗi mục có dạng `HEAD@{n}`, với `n` là số bước tính ngược từ hiện tại. Nhờ đó, ta tìm lại được hash của commit mồ côi và đưa con trỏ nhánh trỏ về nó.

| | `git log` | `git reflog` |
|---|---|---|
| Hiển thị | Chuỗi commit **đi được tới** từ HEAD | Mọi vị trí HEAD **đã từng đi qua** |
| Commit mồ côi sau reset | ❌ Không thấy | ✅ Vẫn thấy |
| Có được push lên remote không | Có | **Không**, chỉ tồn tại trên máy cục bộ |

**Giới hạn:** các mục reflog mặc định được giữ khoảng **90 ngày** (commit mồ côi khoảng 30 ngày). Sau thời gian đó, `git gc` có thể dọn sạch, và commit sẽ mất hẳn.

**Hai cách khôi phục:**

| Lệnh | Kết quả |
|---|---|
| `git reset --hard <hash>` | Đưa nhánh `main` về lại commit cũ. **Cách được dùng trong bài này.** |
| `git checkout -b recover <hash>` | Tạo nhánh mới trỏ vào commit cũ, không động tới nhánh hiện tại. An toàn hơn khi chưa chắc chắn. |

## Đối chiếu yêu cầu

| Yêu cầu | Kết quả |
|---|---|
| Tạo `feature.txt` với nội dung "Day la tinh nang quan trong" và commit | ✅ |
| Giả lập mất commit bằng `git reset --hard HEAD~1` | ✅ |
| Dùng `git reflog` để tìm hash commit bị mất | ✅ |
| Khôi phục bằng `git reset --hard <hash>`, không viết lại code | ✅ |
| `git log` hiển thị lại đúng commit và nội dung ban đầu | ✅ |
