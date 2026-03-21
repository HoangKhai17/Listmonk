# 🛠️ Listmonk – Hướng Dẫn Setup Môi Trường Dev (Windows)

> 📖 Tài liệu tham khảo chính thức: [https://listmonk.app/docs/developer-setup/](https://listmonk.app/docs/developer-setup/)

---

## 📋 Yêu Cầu Trước Khi Bắt Đầu

| Công cụ | Mục đích | Link tải |
|--------|----------|----------|
| Git | Clone source code | https://git-scm.com |
| Go | Chạy backend | https://go.dev/dl/ |
| Node.js + yarn | Chạy frontend | https://nodejs.org |
| Docker Desktop | Chạy database | https://www.docker.com/products/docker-desktop |

**Kiểm tra đã cài đúng chưa:**
```bash
go version
node -v
yarn -v
docker -v
```

---

## 📁 Cấu Trúc Thư Mục

Tạo 2 folder **tách biệt** để tránh nhầm lẫn:

```
D:\Local Sites\Listmonk\
├── listmonk-docker\        ← Chứa compose.yml và uploads (chạy Database)
│   ├── compose.yml
│   └── uploads\
│
└── listmonk-source\        ← Chứa source code clone từ GitHub
    ├── cmd\
    ├── frontend\
    ├── config.toml.sample
    ├── config.toml
    └── ...
```

> ⚠️ **Lưu ý:** Không clone source code vào cùng folder với Docker. Sẽ gây nhầm lẫn và lỗi khi chạy lệnh.

---

## 🚀 Các Bước Thực Hiện

### Bước 1: Clone Source Code

```bash
cd "D:\Local Sites\Listmonk"
git clone https://github.com/knadh/listmonk.git listmonk-source
```

---

### Bước 2: Cài Go

1. Tải bản Windows `.msi` tại: [https://go.dev/dl/](https://go.dev/dl/)
2. Cài đặt bình thường, Go sẽ tự thêm vào PATH
3. Kiểm tra:
```bash
go version
```

---

### Bước 3: Cài yarn

```bash
npm install -g yarn
```

Kiểm tra:
```bash
yarn -v
```

---

### Bước 4: Cấu Hình file `config.toml`

Vào folder source, copy file mẫu:

```powershell
cd "D:\Local Sites\Listmonk\listmonk-source"
copy config.toml.sample config.toml
```

Mở `config.toml`, tìm phần `[db]` và sửa:

```toml
[db]
host     = "localhost"
port     = 5433
user     = "listmonk"
password = "listmonk_password"
database = "listmonk"
ssl_mode = "disable"
```

> ⚠️ **Lưu ý:** Port trong `config.toml` phải **khớp** với port trong `compose.yml` (xem Bước 5).

---

### Bước 5: Cấu Hình Docker Database

Tạo folder `listmonk-docker` và file `compose.yml`:

```
D:\Local Sites\Listmonk\listmonk-docker\
├── compose.yml
└── uploads\
```

Nội dung file `compose.yml`:

```yaml
services:
  db:
    image: postgres:17-alpine
    container_name: listmonk_db
    restart: unless-stopped
    ports:
      - "5433:5432"   # Dùng 5433 để tránh conflict với PostgreSQL local
    environment:
      POSTGRES_USER: listmonk
      POSTGRES_PASSWORD: listmonk_password
      POSTGRES_DB: listmonk
    volumes:
      - listmonk_data:/var/lib/postgresql/data

volumes:
  listmonk_data:
```

> ⚠️ **Lưu ý:** Dùng port `5433` thay vì `5432` để tránh lỗi conflict nếu máy đã cài PostgreSQL local.

---

### Bước 6: Khởi Chạy Database

```powershell
cd "D:\Local Sites\Listmonk\listmonk-docker"
docker compose up -d db
```

Kiểm tra container đã chạy chưa:
```bash
docker ps
```

---

### Bước 7: Build Frontend

```powershell
cd "D:\Local Sites\Listmonk\listmonk-source\frontend"
yarn install
yarn build
```

> ⚠️ **Lưu ý:** Phải chạy `yarn build` trước khi chạy backend lần đầu. Nếu bỏ qua bước này, backend sẽ báo lỗi `failed reading static files from disk`.

> ℹ️ **Lý do dùng `yarn` thay `npm`:** File `package.json` có script `postinstall` dùng lệnh `cp` (Linux). `npm` trên Windows không hỗ trợ `cp`, nhưng `yarn` xử lý được.

---

### Bước 8: Cài Đặt Database (Lần Đầu)

```powershell
cd "D:\Local Sites\Listmonk\listmonk-source"
go run ./cmd/... --install --config config.toml
```

> ℹ️ Chỉ cần chạy lệnh này **một lần duy nhất** để khởi tạo các bảng trong database.

---

### Bước 9: Khởi Chạy Dự Án

Cần mở **2 terminal riêng biệt**:

**Terminal 1 – Chạy Backend:**
```powershell
cd "D:\Local Sites\Listmonk\listmonk-source"
go run ./cmd/... --config config.toml
```

**Terminal 2 – Chạy Frontend Dev:**
```powershell
cd "D:\Local Sites\Listmonk\listmonk-source\frontend"
yarn dev
```

---

### Bước 10: Mở Trình Duyệt

```
http://localhost:8080/admin
```

---

## 🔁 Quy Trình Chạy Lại (Lần Sau)

Mỗi lần muốn dev, chạy theo thứ tự:

```
1. Khởi động Docker Desktop
2. docker compose up -d db          (trong folder listmonk-docker)
3. go run ./cmd/... --config config.toml    (Terminal 1 - backend)
4. yarn dev                         (Terminal 2 - frontend)
5. Mở http://localhost:8080/admin
```

---

## ⚠️ Các Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `'cp' is not recognized` | npm dùng lệnh Linux trên Windows | Dùng `yarn` thay `npm` |
| `no Go files` | Đang ở sai folder | Kiểm tra lại đường dẫn, phải vào `listmonk-source` |
| `port 5432 not available` | PostgreSQL local đang chiếm port | Đổi sang port `5433` trong `compose.yml` và `config.toml` |
| `failed reading static files` | Frontend chưa được build | Chạy `yarn build` trước |
| `'make' is not recognized` | Windows không có lệnh `make` | Thay `make run` bằng `go run ./cmd/... --config config.toml` |
| `invalid session` | Chưa có tài khoản admin | Chạy lại `--install` hoặc tạo user qua API |

---

## 📝 Ghi Chú

- Các cảnh báo `Deprecation Warning` của Sass là **bình thường**, không ảnh hưởng đến chức năng.
- Lệnh `make run` và `make run-frontend` trong docs gốc **không chạy được trên Windows** vì không có `make`. Dùng lệnh Go và yarn trực tiếp thay thế.
- Sau khi chỉnh sửa logo/màu sắc/brand xong, build production bằng: `yarn build` rồi build Docker image riêng.



===================================================================



# 🐳 Hướng Dẫn Build & Push Docker Image (Windows)

> Tài liệu này hướng dẫn cách build source code thành Docker image và đẩy lên Docker Hub để máy khác có thể sử dụng.

---

## 📋 Yêu Cầu Trước Khi Bắt Đầu

| Công cụ | Mục đích | Link |
|--------|----------|------|
| Docker Desktop | Build & push image | https://www.docker.com/products/docker-desktop |
| Go | Build binary từ source | https://go.dev/dl/ |
| Tài khoản Docker Hub | Lưu trữ image | https://hub.docker.com |

---

## ⚠️ Lưu Ý Quan Trọng

> Vì môi trường dev là **Windows**, khi build Docker image cần chỉ định platform `linux/amd64` để máy chủ Linux có thể chạy được. Nếu không chỉ định platform, image sẽ là Windows format và **không chạy được trên Linux/Mac**.

---

## 🚀 Các Bước Thực Hiện

### Bước 1: Khởi động Docker Desktop

Mở Docker Desktop, chờ icon ở taskbar chuyển sang **trắng/xanh** (không còn loading) trước khi chạy các lệnh bên dưới.

---

### Bước 2: Build Binary Go

Dockerfile yêu cầu file binary `listmonk` phải được build trước. Chạy lệnh sau trong thư mục source:

```powershell
cd "D:\Local Sites\Listmonk\listmonk-source"
go build -o listmonk ./cmd/...
```

> ⚠️ **Lưu ý:** Bước này có thể mất vài phút tùy cấu hình máy.

Sau khi xong sẽ tạo ra file `listmonk` (hoặc `listmonk.exe`) trong thư mục gốc.

---

### Bước 3: Đăng Nhập Docker Hub

```powershell
docker login
```

Nhập **username** và **password** tài khoản Docker Hub của bạn.

---

### Bước 4: Build Image & Push Lên Docker Hub

Dùng lệnh sau để build image đúng platform Linux và push thẳng lên Docker Hub:

```powershell
docker buildx build --platform linux/amd64 -t <dockerhub-username>/listmonk-custom:latest . --push
```

Thay `<dockerhub-username>` bằng username Docker Hub của bạn. Ví dụ:

```powershell
docker buildx build --platform linux/amd64 -t hoangkhai2503/listmonk-custom:latest . --push
```

> ℹ️ Flag `--platform linux/amd64` đảm bảo image chạy được trên máy chủ Linux.  
> ℹ️ Flag `--push` tự động push lên Docker Hub sau khi build xong, không cần chạy lệnh push riêng.

---

### Bước 5: Kiểm Tra Trên Docker Hub

Vào link sau để xác nhận image đã được push thành công:

```
https://hub.docker.com/r/<dockerhub-username>/listmonk-custom
```

---

## 🖥️ Máy Khác Sử Dụng Image

Tạo file `compose.yml` với nội dung sau:

```yaml
services:
  db:
    image: postgres:17-alpine
    container_name: listmonk_db
    restart: unless-stopped
    environment:
      POSTGRES_USER: listmonk
      POSTGRES_PASSWORD: listmonk_password
      POSTGRES_DB: listmonk
    volumes:
      - listmonk_data:/var/lib/postgresql/data

  app:
    image: <dockerhub-username>/listmonk-custom:latest
    container_name: listmonk_app
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "9000:9000"
    environment:
      LISTMONK_app__address: 0.0.0.0:9000
      LISTMONK_db__host: db
      LISTMONK_db__port: 5432
      LISTMONK_db__user: listmonk
      LISTMONK_db__password: listmonk_password
      LISTMONK_db__database: listmonk
      LISTMONK_db__ssl_mode: disable
      TZ: Asia/Ho_Chi_Minh
    volumes:
      - ./uploads:/listmonk/uploads
    command: >
      sh -c "./listmonk --install --idempotent --yes --config '' &&
             ./listmonk --upgrade --yes --config '' &&
             ./listmonk --config ''"

volumes:
  listmonk_data:
```

Sau đó chạy:

```powershell
docker compose up -d
```

Mở trình duyệt vào:
```
http://localhost:9000
```

---

## 🔁 Quy Trình Khi Có Thay Đổi Source Code

Mỗi khi chỉnh sửa source code và muốn cập nhật image:

```
1. Chỉnh sửa source code (logo, màu sắc, brand,...)
2. Build lại frontend:        yarn build
3. Build lại binary Go:       go build -o listmonk ./cmd/...
4. Build & push image mới:    docker buildx build --platform linux/amd64 -t <username>/listmonk-custom:latest . --push
5. Máy khác pull image mới:   docker compose pull && docker compose up -d
```

---

## ⚠️ Các Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `docker buildx build requires 1 argument` | Thiếu dấu `.` ở cuối lệnh | Thêm dấu `.` vào cuối lệnh build |
| `failed to connect to docker API` | Docker Desktop chưa khởi động | Mở Docker Desktop, chờ icon ổn định |
| `"/listmonk": not found` | Chưa build binary Go | Chạy `go build -o listmonk ./cmd/...` trước |
| `push access denied` | Chưa đăng nhập Docker Hub | Chạy `docker login` trước khi push |
| Image không chạy trên Linux | Build trên Windows không chỉ platform | Dùng flag `--platform linux/amd64` khi build |


