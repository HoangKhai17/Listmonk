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
