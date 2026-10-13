# IT209 - Session 09 - Bài tập 4: Lưu trữ dữ liệu đơn giản với Docker Volume

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Học phần:** IT209
- **Đường dẫn nộp bài:** `homework/session_09/ex4/`
- **Trạng thái:** Hoàn thành (100%)

---

## 1. Mục tiêu bài tập
- Thực hành khởi tạo và quản lý **Named Volume** trong Docker (`easy-volume`).
- Kiểm chứng tính bền vững của dữ liệu (Data Persistence): Dữ liệu lưu trong Volume hoàn toàn không bị mất đi khi container kết thúc và bị hủy bỏ (`--rm`).
- Thực hiện cơ chế chia sẻ và lưu trữ dữ liệu giữa các container Alpine độc lập.

---

## 2. Toàn bộ các lệnh đã thực hiện & Kết quả Terminal

### Bước 1: Khởi tạo Docker Named Volume
```bash
docker volume create easy-volume
```

#### Kết quả đầu ra:
```plaintext
easy-volume
```

#### Kiểm tra danh sách Volume:
```bash
docker volume ls
```
```plaintext
DRIVER    VOLUME NAME
local     easy-volume
```

#### Kiểm tra chi tiết cấu hình Volume (Inspect):
```bash
docker volume inspect easy-volume
```
```json
[
    {
        "CreatedAt": "2026-10-13T09:09:15+07:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/easy-volume/_data",
        "Name": "easy-volume",
        "Options": null,
        "Scope": "local"
    }
]
```

---

### Bước 2: Sử dụng Container 1 để ghi dữ liệu vào Volume

Container Alpine thứ nhất được khởi chạy, gắn kết volume `easy-volume` vào thư mục `/data`, ghi nội dung chuỗi vào tệp `/data/test.txt` và tự động bị hủy sau khi hoàn tất (`--rm`):

```bash
docker run --rm -v easy-volume:/data alpine sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"
```

#### Kết quả đầu ra:
```plaintext
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
43c4264eed91: Pull complete 
Digest: sha256:beefdbd8a1da6d2915566fde36db9db0b524eb7377ec9c2f6c5e66c2aa78383f
Status: Downloaded newer image for alpine:latest
```
> **Đánh giá:** Container 1 đã hoàn thành lệnh ghi, kết thúc tiến trình và lập tức bị xóa bỏ khỏi hệ thống do cờ `--rm`. Toàn bộ dữ liệu nằm an toàn tại `Mountpoint` của `easy-volume`.

---

### Bước 3: Sử dụng Container 2 để đọc lại dữ liệu từ Volume (Kiểm tra)

Khởi chạy container Alpine thứ hai hoàn toàn mới, gắn kết cùng volume `easy-volume` vào thư mục `/data` và đọc nội dung tệp `test.txt`:

```bash
docker run --rm -v easy-volume:/data alpine cat /data/test.txt
```

#### Kết quả đầu ra:
```plaintext
Luu tru du lieu Docker
```

> **Kết quả mong đợi:** Terminal in ra chính xác dòng chữ: **`Luu tru du lieu Docker`**.

---

### Bước 4: Dọn dẹp Volume (Sau khi hoàn tất bài tập)

```bash
# Xóa volume đã tạo
docker volume rm easy-volume
```

#### Kết quả đầu ra:
```plaintext
easy-volume
```

#### Xác nhận volume đã được xóa:
```bash
docker volume ls
```
```plaintext
DRIVER    VOLUME NAME
```

---

## 3. Bảng đối chiếu kết quả kiểm thử

| Tiêu chí | Lệnh yêu cầu | Kết quả thực tế | Trạng thái |
| :--- | :--- | :--- | :---: |
| **Tạo Volume** | `docker volume create easy-volume` | Volume `easy-volume` driver `local` | **PASS** |
| **Kiểm tra danh sách** | `docker volume ls` | Hiển thị `easy-volume` trong danh sách | **PASS** |
| **Ghi dữ liệu (C1)** | `docker run --rm -v easy-volume:/data alpine sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"` | Ghi thành công, container 1 tự xóa | **PASS** |
| **Đọc dữ liệu (C2)** | `docker run --rm -v easy-volume:/data alpine cat /data/test.txt` | In đúng: *"Luu tru du lieu Docker"* | **PASS** |
| **Tính bền vững** | Dữ liệu tồn tại dù container đã bị xóa | Tệp `test.txt` nguyên vẹn giữa 2 phiên container | **PASS** |
