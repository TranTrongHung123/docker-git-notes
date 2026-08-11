## 1. Docker Compose là gì

**Docker Compose** là công cụ dùng để định nghĩa và quản lý các ứng dụng gồm **nhiều container** bằng một file cấu hình YAML.

Thay vì phải chạy nhiều lệnh `docker run` thủ công:
```bash
docker run -d --name db ...
docker run -d --name redis ...
docker run -d --name backend ...
docker run -d --name nginx ...
```

Ta mô tả toàn bộ hệ thống trong một file:
```bash
compose.yaml
```

Sau đó dùng:
```bash
docker compose up -d
```

Docker Compose sẽ tạo và quản lý các thành phần cần thiết và gom các cấu hình này vào một nơi

```
                    compose.yaml
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Backend        PostgreSQL       Redis
          │              │              │
          └──────────────┼──────────────┘
                         │
                    Docker Network
```

---
## 2. Compose File

File cấu hình hiện đại thường sử dụng:
```
compose.yaml
```

Ngoài ra vẫn có thể gặp:
```
compose.yml
docker-compose.yaml
docker-compose.yml
```

Trong project mới, nên ưu tiên:
```
compose.yaml
```

Ví dụ cấu trúc project:
```bash
my-fullstack-project/           # Thư mục Root
├── .env                        # Biến môi trường dùng chung cho Docker Compose
├── .dockerignore               # Loại bỏ file rác ở mức root
├── compose.yaml                # File điều phối toàn bộ FE, BE, DB
│
├── frontend/                   # Sub-project Frontend (React)
│   ├── src/
│   ├── package.json
│   ├── .dockerignore           # Dockerignore cho FE (node_modules, build...)
│   └── Dockerfile               # Build và chạy Frontend (Multi-stage + Nginx)
│
├── backend/                    # Sub-project Backend (Spring Boot / Node.js / Go)
│   ├── src/
│   ├── pom.xml
│   ├── .dockerignore            # Dockerignore cho BE (target, node_modules...)
│   └── Dockerfile               # Build và chạy Backend (Multi-stage Build)
│
└── db/                         # Chứa file khởi tạo Database
    └── init.sql                # Script SQL chạy tự động khi tạo DB lần đầu
```

---
## 3. Các thành phần trong compose.yaml
### a. services

`services` định nghĩa các service mà application cần.

Ví dụ:
```YAML
services:
  frontend:
    image: nginx:alpine

  backend:
    image: my-backend:1.0

  database:
    image: postgres:16

  redis:
    image: redis:alpine
```

Có thể hình dung:
```bash
services
│
├── frontend
│
├── backend
│
├── database
│
└── redis
```

Mỗi service có thể tạo và quản lý container tương ứng.

### b. image

Dùng image có sẵn để tạo container.
```
services:
  nginx:
    image: nginx:alpine
```

Compose sẽ sử dụng:
```
nginx:alpine
```

Nếu image chưa có local, Docker có thể pull image từ Registry.

Ví dụ:
```
services:
  db:
    image: postgres:16
```

Nên chỉ định tag cụ thể thay vì phụ thuộc để giảm rủi ro image thay đổi ngoài ý muốn.
```YAML
image: postgres   # Thiếu TAG
```

### c. build

`build` cho phép Compose build image từ Dockerfile.

Ví dụ:
```
services:
  backend:
    build: .
```

Docker sẽ sử dụng:

```
.
│
├── Dockerfile
├── package.json
├── src/
└── ...
```

làm build context.

Có thể chỉ định rõ hơn:
```
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
```

Ở đây:
```
context:
    ./backend

dockerfile:
    ./backend/Dockerfile
```

### d. ports

Dùng để publish port của container ra Host.
```YAML
services:
  backend:
    image: my-backend
    ports:
      - "8080:8080"
```

Cấu trúc:
```
HOST:CONTAINER
```

Ví dụ:
```
8080:8080
```

nghĩa là:
```
Host port 8080
       │
       ▼
Container port 8080
```

Nếu ứng dụng chạy ở container port `3000`:
```
ports:
  - "8080:3000"
```

thì:
```
localhost:8080
       ↓
container:3000
```

### e. volumes

Compose có thể mount volume vào container.

Ví dụ Named Volume:
```YAML
services:
  db:
    image: postgres:16
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

Cấu trúc:
```
Docker Volume
     │
     ▼
postgres-data
     │
     ▼
/var/lib/postgresql/data
     │
     ▼
PostgreSQL container
```

Volume tồn tại độc lập với container.

Do đó:
```
docker compose down
```

không mặc định xóa Named Volume.

Nếu muốn xóa cả volume:
```
docker compose down -v
```

**Cẩn thận:** `-v` sẽ xóa các volume được Compose quản lý trong project, vì vậy có thể làm mất dữ liệu persistent nếu dữ liệu nằm trong các volume đó.

### f. Bind Mount

Compose cũng hỗ trợ Bind Mount.
```YAML
services:
  backend:
    volumes:
      - ./backend:/app
```

Ý nghĩa:
```
Host
./backend
    │
    ▼
Container
/app
```

Thường dùng trong development:
```
Source code trên Host
        ↓
      Mount
        ↓
Container
        ↓
Hot reload
```

### g. networks

Compose có thể tạo network cho các service.

Ví dụ:
```YAML
services:
  backend:
    image: my-backend
    networks:
      - app-network

  db:
    image: postgres:16
    networks:
      - app-network

networks:
  app-network:
```

Khi đó:
```
backend
   │
   │ app-network
   ▼
database
```

Các container trong cùng network có thể giao tiếp với nhau.

### h. environment

Dùng để truyền Environment Variables vào container.
```YAML
services:
  backend:
    image: my-backend
    environment:
      DATABASE_HOST: db
      DATABASE_PORT: 5432
      DATABASE_NAME: myapp
```

Trong container:
```bash
DATABASE_HOST=db
DATABASE_PORT=5432
DATABASE_NAME=myapp
```

Application có thể đọc các biến này.

Ngoài ra có thể đưa các giá trị cấu hình ra ngoài `compose.yaml` bằng file .env

`.env`:
```
POSTGRES_USER=admin
POSTGRES_DB=myapp
POSTGRES_PASSWORD=example
```

Compose:
```
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

Điều này giúp tránh hardcode configuration trực tiếp vào YAML.

Tuy nhiên:

> `.env` **không phải một secret manager**.

Không nên coi việc đặt password vào `.env` là giải pháp bảo mật production.

### i. env_file

Có thể truyền environment variables từ một file:
```
services:
  backend:
    env_file:
      - .env
```

Ví dụ `.env`:
```
DATABASE_HOST=db
DATABASE_PORT=5432
DATABASE_NAME=myapp
```

Container sẽ nhận các biến này.

Cần phân biệt:
```
environment:
```

và:
```
env_file:
```

`environment` khai báo trực tiếp trong Compose.

`env_file` đọc biến từ file.

### j. depends_on

Dùng để biểu diễn dependency giữa các service.

Ví dụ:

```
services:
  backend:
    depends_on:
      - db

  db:
    image: postgres:16
```

`depends_on` biểu diễn dependency giữa các service và có thể kiểm soát thứ tự khởi động theo dependency, nhưng mặc định không kiểm tra service đã sẵn sàng phục vụ hay chưa.

Nhưng cần nhớ:

> `depends_on` không đồng nghĩa với việc database đã sẵn sàng nhận request.

Container PostgreSQL có thể đã **started** nhưng database vẫn đang initialization.

### k. healthcheck

Để kiểm tra service có thực sự hoạt động hay không.

Ví dụ:
```YAML
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

Docker sẽ kiểm tra:
```
db
 │
 ├── starting
 │
 ├── healthy
 │
 └── unhealthy
```

Có thể kết hợp với `depends_on`:
```YAML
services:
  backend:
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

Khi đó Compose có thể chờ database đạt trạng thái healthy trước khi khởi động backend theo dependency này.


### l. restart

`restart` policy quy định **Docker Compose sẽ xử lý container như thế nào khi container kết thúc hoặc gặp lỗi**.

Ví dụ:
```
services:
  backend:
    image: my-backend:1.0
    restart: unless-stopped
```

Các policy cơ bản:

|      Policy      |                                        Ý nghĩa                                         |
| :--------------: | :------------------------------------------------------------------------------------: |
|       `no`       |                   Không tự động restart container. Đây là mặc định.                    |
|     `always`     |                       Luôn restart container khi container dừng.                       |
|   `on-failure`   |               Chỉ restart khi container kết thúc với exit code khác `0`.               |
| `unless-stopped` | Restart container nếu container dừng, trừ khi container được người dùng chủ động stop. |

### `no`

```
restart: "no"
```

Không tự động restart.

Đây là hành vi mặc định.

Thường phù hợp với các container chạy task hoặc môi trường development đơn giản.

### `always`

```
restart: always
```

Docker sẽ cố gắng restart container nếu container dừng.

Ví dụ:

```
backend
   │
   ├── crash
   ↓
Docker restart
   ↓
backend chạy lại
```

Phù hợp với các service cần chạy liên tục.

### `on-failure`

```
restart: on-failure
```

Chỉ restart khi process trong container kết thúc với **exit code khác `0`**.

Ví dụ:

```
Application
     │
     ├── exit 1
     ↓
restart
```

Nhưng nếu:

```
Application
     │
     ├── exit 0
     ↓
không restart
```

Có thể giới hạn số lần restart:

```
restart: on-failure:5
```

Nghĩa là Docker sẽ thử restart tối đa 5 lần.

### `unless-stopped`

```
restart: unless-stopped
```

Container sẽ được tự động restart nếu bị dừng ngoài ý muốn.

Nhưng nếu người dùng chủ động stop container:

```
docker compose stop
```

Docker sẽ không tự động chạy lại container đó.

Đây là policy thường phù hợp với các service chạy lâu dài.

### Ví dụ thực tế

```
services:

  backend:
    image: my-backend:1.0
    restart: unless-stopped

  db:
    image: postgres:16
    restart: unless-stopped

  redis:
    image: redis:7
    restart: unless-stopped
```

Có thể hình dung:

```
           Docker
              │
      ┌───────┼────────┐
      ▼       ▼        ▼
   backend    db      redis
      │       │        │
      └───────┴────────┘
              │
        restart policy
```

> **Lưu ý:** `restart` policy chỉ quyết định **có restart container hay không**. Nó không kiểm tra application bên trong container có thực sự hoạt động đúng hay không. Việc kiểm tra trạng thái ứng dụng thường cần `healthcheck`.


---
## 4. Compose CLI

### a. `docker compose up`

Tạo và chạy services.

```
docker compose up
```

Chạy background:

```
docker compose up -d
```

Ép build lại các image có cấu hình trước khi khởi động service:
```
docker compose up --build
```

### b. `docker compose down`

Dừng và xóa các resource do Compose quản lý phù hợp với project.

```
docker compose down
```

Xóa cả volumes:

```
docker compose down -v
```

### c. `docker compose ps`

Xem trạng thái service:

```
docker compose ps
```

### d. `docker compose logs`

Xem log:

```
docker compose logs
```

Một service:

```
docker compose logs backend
```

Theo dõi realtime:

```
docker compose logs -f backend
```

### e. `docker compose exec`

Thực thi command bên trong container đang chạy:

```
docker compose exec backend sh
```

Ví dụ:

```
docker compose exec db psql -U postgres
```

Đây là một trong những lệnh debug quan trọng nhất.

### f. `docker compose build`

Build image:

```
docker compose build
```

Build một service:

```
docker compose build backend
```

---

### g. `docker compose pull`

Pull image:

```
docker compose pull
```

### h. `docker compose restart`

Restart service:

```
docker compose restart backend
```

### i. `docker compose stop`

Dừng container nhưng không xóa chúng:

```
docker compose stop
```

### j. `docker compose start`

Khởi động lại các container đã tồn tại:

```
docker compose start
```

### k. docker compose config

Kiểm tra cú pháp file `compose.yaml` và xem kết quả sau khi Compose đã nạp các biến từ file `.env`

```
docker compose config
```
