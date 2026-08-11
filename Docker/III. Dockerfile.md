## 1. Dockerfile là gì?

Dockerfile là một file văn bản chứa tập hợp các **instruction** dùng để Docker tự động build ra một Docker Image.

Cú pháp của một Dockerfile:   `INSTRUCTION arguments`
- **INSTRUCTION** là tên các chỉ thị có trong Dockerfile, mỗi chỉ thị thực hiện một nhiệm vụ nhất định, được Docker quy định. Khi khai báo các chỉ thị này phải được viết bằng chữ IN HOA.
- Một Dockerfile bắt buộc phải bắt đầu bằng chỉ thị **FROM** để khai báo đâu là image sẽ được sử dụng làm nền để xây dựng nên image của bạn.
- `aguments` là phần nội dung của các chỉ thị, quyết định chỉ thị sẽ làm gì.

---
## 2. Các chỉ thị chính trong Dockerfile

Dockerfile Instructions (Best Practice Order)

| Thứ tự |  Instruction  |                                                                                  Ý nghĩa                                                                                  |
| :----: | :-----------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|   1    |    `FROM`     |                                           Chỉ định **base image** để xây dựng image mới. Đây là instruction bắt buộc đầu tiên.                                            |
|   2    |    `LABEL`    |                          Thêm **metadata** cho image như tác giả, phiên bản, mô tả, thông tin dự án... Không ảnh hưởng đến việc chạy container.                           |
|   3    |     `ARG`     |                        Khai báo **biến chỉ tồn tại trong quá trình build**. Thường dùng để truyền phiên bản phần mềm hoặc tham số khi build image.                        |
|   4    |     `ENV`     |                        Khai báo **biến môi trường** tồn tại trong image và container khi chạy. Có thể được sử dụng bởi các lệnh `RUN` và ứng dụng.                        |
|   5    |   `WORKDIR`   |                         Thiết lập **thư mục làm việc mặc định** trong container cho các lệnh `RUN`, `COPY`, `ADD`, `CMD`, `ENTRYPOINT` phía sau.                          |
|   6    |    `COPY`     | Sao chép file hoặc thư mục từ **Build Context** vào image. Thường copy file dependency (`package.json`, `pom.xml`, `requirements.txt`...) trước để tận dụng Docker Cache. |
|   7    |     `RUN`     |                     Thực thi lệnh trong quá trình **build image** (cài package, build source, tạo user...). Mỗi `RUN` thường tạo thêm một layer mới.                      |
|   8    |    `COPY`     |                                     Sao chép toàn bộ mã nguồn của ứng dụng sau khi đã cài dependency nhằm tối ưu Docker Build Cache.                                      |
|   9    |     `ADD`     |                 Tương tự `COPY` nhưng có thêm khả năng **tự giải nén file nén** và tải file từ URL. Chỉ nên dùng khi thật sự cần, còn lại ưu tiên `COPY`.                 |
|   10   |   `EXPOSE`    |                                 Khai báo cổng mà ứng dụng bên trong container sẽ lắng nghe. Chỉ mang tính tài liệu, **không tự mở cổng**.                                 |
|   11   |    `USER`     |                                     Chỉ định user dùng để chạy container. Nên sử dụng user không phải `root` nhằm tăng tính bảo mật.                                      |
|   12   |   `VOLUME`    |                                 Khai báo thư mục dùng để lưu trữ dữ liệu bền vững (persistent data). Docker có thể tự tạo Volume nếu cần.                                 |
|   13   | `HEALTHCHECK` |                                     Định nghĩa cách Docker kiểm tra tình trạng hoạt động của container (`healthy` hoặc `unhealthy`).                                      |
|   14   | `ENTRYPOINT`  |                                        Định nghĩa **chương trình chính** của container. Thường ít bị ghi đè và kết hợp với `CMD`.                                         |
|   15   |     `CMD`     |                             Khai báo **lệnh hoặc tham số mặc định** khi container khởi động. Có thể bị ghi đè bởi lệnh khi chạy `docker run`.                             |

### a. FROM

Lệnh `FROM` định nghĩa image cơ sở mà image của bạn sẽ được xây dựng dựa trên đó. Đây là lựa chọn quan trọng đầu tiên.

- **Chọn image cơ sở tối thiểu:** Thay vì sử dụng các image lớn như `ubuntu` hay `debian` đầy đủ, hãy cân nhắc các phiên bản “slim” hoặc các image siêu nhỏ như `alpine`. `alpine` dựa trên musl libc và BusyBox, rất nhẹ. Ví dụ: `FROM alpine:3.18` thay vì `FROM ubuntu:22.04`. Điều này giúp giảm đáng kể kích thước ảnh cuối cùng. Tuy nhiên, hãy kiểm tra tính tương thích của các thư viện và công cụ bạn cần với image cơ sở tối thiểu.
- **Sử dụng tag cụ thể:** Luôn chỉ định tag phiên bản (ví dụ: `python:3.9-slim`, `node:16-alpine`) thay vì `latest`. Điều này đảm bảo tính lặp lại (reproducibility) cho bản build của bạn.

**Ví dụ:**
```bash
# Nên làm:
FROM python:3.9-slim

# Tránh làm (trừ khi có lý do đặc biệt):
FROM ubuntu
FROM node:latest
```

### b. LABEL

Chỉ thị **LABEL** được dùng để thêm các thông tin meta vào **Docker Image** khi chúng được build. Chúng tồn tại dưới dạng các cặp _key_ - _value_, được lưu trữ dưới dạng chuỗi. Có thể chỉ định nhiều label cho một Docker Image, và tất nhiên mỗi cặp _key_ - _value_ phải là duy nhất. Nếu cùng một _key_ mà được khai báo nhiều giá trị (value) thì giá trị được khai báo gần đây nhất sẽ ghi đè lên giá trị trước đó.

**Cú pháp:**   `LABEL <key>=<value> <key>=<value> <key>=<value> ... <key>=<value>`

**Ví dụ:**
```bash
LABEL author="Hung" \
      version="1.0" \
      description="My Spring Boot API"
```

### c. ENV và ARG

- **`ENV`: Biến môi trường** – Đặt biến môi trường trong image. Các biến này tồn tại khi container chạy.
- **`ARG`: Biến build-time** – Định nghĩa các biến chỉ có sẵn trong quá trình build Docker image. Chúng không tồn tại trong image cuối cùng. `ARG` thường được dùng để truyền thông tin nhạy cảm (tạm thời) hoặc cấu hình build (như phiên bản phần mềm, địa chỉ repository).

**Ví dụ:**
```bash
ARG APP_VERSION=1.0.0
ENV VERSION=${APP_VERSION}
ENV NODE_ENV=production
```

### d. WORKDIR

Lệnh `WORKDIR` thiết lập thư mục làm việc trong container cho các lệnh `RUN`, `CMD`, `ENTRYPOINT`, `COPY` và `ADD` tiếp theo. Sử dụng `WORKDIR` giúp Dockerfile của bạn rõ ràng hơn và tránh lặp lại việc chỉ định đường dẫn đầy đủ.

**Ví dụ:**
```bash
WORKDIR /app
COPY ./package.json ./
RUN npm install
COPY ./src ./src
CMD ["npm", "start"]
```

**Thay vì:**
```bash
COPY ./package.json /app/package.json
RUN cd /app && npm install
COPY ./src /app/src
CMD ["/app/node_modules/.bin/npm", "start"] 
```

### e. RUN

Lệnh `RUN` thực thi một lệnh trong một layer mới của image. Mỗi lệnh `RUN` sẽ tạo ra một layer mới, và số lượng layer ảnh hưởng đến kích thước ảnh và hiệu quả sử dụng cache.

- **Kết hợp các lệnh `RUN`:** Đây là một trong những mẹo quan trọng nhất để giảm số lượng layer và tận dụng cache hiệu quả. Thay vì viết nhiều lệnh `RUN` riêng lẻ để cài đặt các gói hoặc thực hiện các tác vụ liên quan, hãy kết hợp chúng thành một lệnh `RUN` duy nhất bằng cách sử dụng ký tự `&&` và đặt mỗi lệnh con trên một dòng mới bằng ký tự `\` để dễ đọc.

**Ví dụ (Nên làm):**
```bash
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    wget \
    git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```


**Ví dụ (Không nên làm):**
```bash
FROM ubuntu:22.04

RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get install -y git
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*
```


Phiên bản “Nên làm” chỉ tạo ra 1 layer, trong khi phiên bản “Không nên làm” tạo ra 6 layer. Ít layer hơn thường dẫn đến ảnh nhỏ hơn.

- **Dọn dẹp sau khi cài đặt:** Ngay trong cùng một lệnh `RUN` cài đặt gói, hãy dọn dẹp các file cache không cần thiết. Ví dụ với APT, thêm `apt-get clean && rm -rf /var/lib/apt/lists/*` giúp thu nhỏ kích thước layer đó. Tương tự với các trình quản lý gói khác như YUM/DNF 

### f. COPY và ADD

Cả hai lệnh này đều sao chép file/thư mục từ host vào image. Nhưng có sự khác biệt quan trọng.

- **Ưu tiên `COPY`:** Lệnh `COPY` đơn giản, rõ ràng và được khuyến khích sử dụng hơn. Nó chỉ sao chép file hoặc thư mục từ một vị trí cụ thể trên host (trong build context) đến một đích trong image.
- **Sử dụng `ADD` khi cần giải nén:** Lệnh `ADD` có thêm hai tính năng: tự động giải nén file nén (như tar, gzip, bzip2) và hỗ trợ sao chép file từ URL. Tuy nhiên, tính năng tự động giải nén có thể gây khó đoán, và sao chép từ URL không được khuyến khích vì nó tạo ra một layer mới mỗi lần URL thay đổi (hoặc nội dung thay đổi) và khó kiểm soát nguồn gốc file. Chỉ dùng `ADD` khi cần giải nén một file nén cục bộ vào image.

**Ví dụ:**
```bash
# Nên dùng (đơn giản, rõ ràng):
COPY ./app /app

# Nên dùng (khi cần giải nén file nén cục bộ):
ADD ./app.tar.gz /app

# Tránh dùng (khi chỉ cần sao chép):
ADD ./app /app

# Tránh dùng (không kiểm soát được nguồn gốc và cache):
ADD https://example.com/archive.tar.gz /tmp/
```

- **Build Context:** Cần hiểu về build context – thư mục chứa Dockerfile và các file bạn muốn đưa vào image. Chỉ những file/thư mục trong context mới có thể được `COPY` hoặc `ADD`.
- **Sử dụng `.dockerignore`:** Tương tự như `.gitignore`, tệp `.dockerignore` liệt kê các file và thư mục mà Docker client nên bỏ qua khi gửi build context đến Docker daemon. Điều này giúp giảm kích thước context, tăng tốc độ build và tránh đưa các file nhạy cảm (như mật khẩu, file tạm) hoặc không cần thiết (như thư mục `node_modules` nếu bạn cài đặt nó trong Dockerfile, thư mục `.git`, file log…) vào build context.


**Ví dụ tệp .dockerignore:**
```bash
.git
.gitignore
node_modules
npm-debug.log
Dockerfile
README.md
```

### g. EXPOSE

Lệnh `EXPOSE` thông báo rằng container lắng nghe trên các cổng mạng được chỉ định. Lệnh này chỉ mang tính tài liệu, không thực sự publish cổng. Nó giúp người dùng image của bạn biết cần publish cổng nào. Việc publish cổng được thực hiện khi chạy container với tùy chọn `-p` của lệnh `docker run`.

**Ví dụ:**
```bash
EXPOSE 80 443
```

### h. USER

Theo mặc định, các lệnh `RUN` và container chạy với user root. Đây là một rủi ro bảo mật lớn. Luôn tạo một user không phải root và chuyển sang user đó bằng lệnh `USER` trước khi chạy ứng dụng.

**Ví dụ:**
```bash
# Tạo một nhóm và người dùng mới
RUN groupadd --gid 1000 appuser && useradd --uid 1000 --gid appuser --shell /bin/bash --create-home appuser

# Đặt quyền sở hữu cho thư mục ứng dụng
RUN chown -R appuser:appuser /app

# Chuyển sang user mới trước khi chạy ứng dụng
USER appuser

WORKDIR /app
COPY --chown=appuser:appuser ./app /app
CMD ["/app/my-app"]
```

### i. VOLUME

Lệnh `VOLUME` chỉ định một mount point trong image, nó không tạo volume ngay mà chỉ đánh dấu thư mục đó là nơi chứa dữ liệu có thể thay đổi hoặc dữ liệu bền vững. Nó giống như một tài liệu hướng dẫn rằng thư mục này nên được mount từ bên ngoài (thường là Docker Volume hoặc bind mount) khi chạy container. Điều này rất quan trọng cho việc lưu trữ dữ liệu bền vững

**Ví dụ:**
```bash
VOLUME /app/data
```

### j. HEALTHCHECK

**Cho Docker biết ứng dụng bên trong container còn hoạt động bình thường hay không**, thay vì chỉ biết container có đang chạy (`running`) hay không.

**Ví dụ:**
```bash
HEALTHCHECK \
  --interval=30s \
  --timeout=5s \
  --retries=3 \
  CMD <lệnh kiểm tra ứng dụng> || exit 1
```

* Thay `<lệnh kiểm tra ứng dụng>` theo loại dịch vụ (Nginx, Spring Boot, Node.js, Redis, MySQL,...) 

### k. CMD và ENTRYPOINT

Đây là hai lệnh dùng để chỉ định lệnh sẽ được thực thi khi container khởi động từ image. Sự khác biệt giữa chúng rất quan trọng.

- **`CMD`: Lệnh mặc định** – Cung cấp lệnh mặc định cho container đang chạy. Lệnh này có thể bị ghi đè bởi đối số dòng lệnh khi chạy `docker run`. Một Dockerfile chỉ nên có một lệnh `CMD`.
- **`ENTRYPOINT`: Lệnh thực thi** – Cung cấp một lệnh thực thi cho container. Đối số được truyền cho `docker run` sẽ được nối vào cuối lệnh `ENTRYPOINT`. Sử dụng `ENTRYPOINT` khi bạn muốn container chạy như một executable.
- **Kết hợp `ENTRYPOINT` và `CMD`:** Đây là mô hình phổ biến. `ENTRYPOINT` được đặt là executable chính, còn `CMD` cung cấp các tham số mặc định cho executable đó.

**Ví dụ (ENTRYPOINT + CMD):**
```bash
# Container chạy như một executable (ví dụ: `docker run my-image --help`)
ENTRYPOINT ["/app/my-app"]
CMD ["--help"]
```

Với cấu hình này, khi chạy `docker run my-image`, lệnh thực tế là `/app/my-app --help`. Khi chạy `docker run my-image --config /etc/config.yaml`, lệnh thực tế là `/app/my-app --config /etc/config.yaml`.

- **Sử dụng dạng Exec (Exec form):** Đối với cả `CMD` và `ENTRYPOINT`, khuyến khích sử dụng dạng Exec (`["executable", "param1", "param2"]`) thay vì dạng Shell (`command param1 param2`). Dạng Exec không chạy lệnh qua một shell process (`/bin/sh -c`), giúp tránh các tín hiệu bất ngờ (như SIGTERM) bị xử lý sai bởi shell phụ và đảm bảo process chính của ứng dụng là process PID 1 trong container, nhận tín hiệu trực tiếp từ Docker.

---
## 3. Các Kỹ Thuật Nâng Cao Để Viết Dockerfile Hiệu Quả
### a. Luôn cố định phiên bản Base Image (Pin Version)
- ❌ `FROM node:latest`
- ✅ `FROM node:22-alpine`
- Giúp build ổn định, tái lập (reproducible) và tránh thay đổi ngoài ý muốn.

### b. Luôn dọn cache của package manager

Ví dụ Debian/Ubuntu:
```bash
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*
```

- Giảm kích thước image.
- Giảm bề mặt tấn công.

### c. Chỉ COPY những gì cần thiết

Dùng `.dockerignore` để loại bỏ file không cần thiết.

Tách `COPY` file dependency và `COPY` source code để tận dụng Docker Build Cache.

Hạn chế đưa các thư mục lớn hoặc nhạy cảm vào Build Context.

### d. Build Đa Tầng (Multi-stage Builds)

Đây là một trong những kỹ thuật mạnh mẽ nhất để tạo ra các image nhỏ gọn. Multi-stage build cho phép bạn sử dụng nhiều lệnh `FROM` trong một Dockerfile. Mỗi lệnh `FROM` có thể sử dụng một image cơ sở khác nhau và bắt đầu một “stage” build mới. Quan trọng là bạn có thể sao chép các artifacts (file thực thi, thư viện, v.v.) từ một stage trước đó sang stage hiện tại.

Ưu điểm: Tách biệt môi trường build (với tất cả các công cụ, SDK, dependencies cần thiết) và môi trường runtime (chỉ cần những gì để chạy ứng dụng). Kết quả là image cuối cùng chỉ chứa ứng dụng và các dependency cần thiết, không chứa các công cụ build hoặc mã nguồn đầy đủ, giúp giảm đáng kể kích thước và tăng bảo mật.

**Ví dụ Multi-stage Build cho ứng dụng Go:**
```bash
# Stage 1: Build code
FROM golang:1.20 AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/my-app ./cmd/api/main.go

# Stage 2: Run application
FROM alpine:3.18

WORKDIR /app

# Sao chép file thực thi từ stage builder
COPY --from=builder /app/my-app .

# Chạy với user không phải root (tạo user nếu cần, hoặc dùng user có sẵn trong alpine)
RUN adduser -D appuser
USER appuser

CMD ["./my-app"]
```

Trong ví dụ này, image cuối cùng chỉ dựa trên `alpine:3.18` và chỉ chứa file thực thi `my-app`, rất nhỏ gọn so với việc build và chạy trong cùng một image `golang:1.20`.

Kỹ thuật này có thể áp dụng cho hầu hết các ngôn ngữ cần compile hoặc có nhiều dependency chỉ dùng khi build (Node.js với devDependencies, Java với Maven/Gradle, v.v.). Bạn có thể xem xét lại cách [chọn ngôn ngữ lập trình](https://tuyendung.evotek.vn/roadmap-docker-chon-ngon-ngu-lap-trinh-phu-hop-cho-du-an-docker/) và [kiến trúc ứng dụng](https://tuyendung.evotek.vn/roadmap-docker-kien-truc-ung-dung-anh-huong-den-thiet-ke-docker-nhu-the-nao/) để thấy multi-stage build phù hợp như thế nào.

### e. Tận Dụng Bộ Nhớ Cache Khi Build

Docker build xử lý Dockerfile từng lệnh một. Đối với mỗi lệnh, Docker kiểm tra xem có layer nào trong cache khớp với lệnh đó hay không. Nếu có, Docker sử dụng layer cache đó thay vì chạy lại lệnh, giúp quá trình build nhanh hơn.

Quy tắc là: nếu một lệnh khớp với cache, Docker sẽ sử dụng layer cache đó VÀ tất cả các lệnh **tiếp theo** cũng sẽ sử dụng cache (nếu khớp) cho đến khi gặp một lệnh KHÔNG khớp với cache. Từ lệnh không khớp đó trở đi, Docker sẽ phải chạy lại lệnh đó và tất cả các lệnh sau đó.

Để tối ưu hóa cache, hãy đặt các lệnh ít thay đổi ở phía trên Dockerfile (ví dụ: cài đặt các gói hệ thống, sao chép các file cấu hình tĩnh) và các lệnh thay đổi thường xuyên (ví dụ: sao chép mã nguồn ứng dụng) ở phía dưới.

**Ví dụ (Tận dụng Cache):**
```bash
FROM node:16-alpine

WORKDIR /app

# Các file dependency (package.json, package-lock.json) ít thay đổi hơn mã nguồn
# Sao chép và cài đặt dependency trước để tận dụng cache layer này
COPY package.json package-lock.json ./
RUN npm install --production # Cài đặt chỉ các dependency cần thiết cho runtime

# Sao chép toàn bộ mã nguồn ứng dụng (thường xuyên thay đổi) sau
COPY . .

CMD ["npm", "start"]
```

Nếu chỉ thay đổi mã nguồn trong thư mục `./src`, Docker sẽ sử dụng cache cho các lệnh `FROM`, `WORKDIR`, `COPY package.json...` và `RUN npm install...`. Chỉ lệnh `COPY . .` và `CMD` cần chạy lại. Nếu bạn sao chép toàn bộ mã nguồn trước khi cài đặt dependency, mỗi lần thay đổi mã nguồn sẽ làm cache của lệnh `COPY . .` bị invalidate, buộc Docker phải chạy lại cả lệnh `RUN npm install`, tốn nhiều thời gian hơn.

---
## 4. Thực Hành Bảo Mật Tốt Nhất

Bảo mật là yếu tố không thể thiếu khi làm việc với Docker.

- **Chạy với user không phải root:** Như đã đề cập ở trên với lệnh `USER`. Đây là biện pháp cơ bản nhất.
- **Chỉ cài đặt những gì cần thiết:** Mỗi gói bổ sung là một nguy cơ tiềm tàng và làm tăng kích thước ảnh. Sử dụng tùy chọn `--no-install-recommends` với APT hoặc các cờ tương tự với các trình quản lý gói khác để tránh cài đặt các gói không cần thiết.
- **Sử dụng các phiên bản cụ thể:** Chỉ định phiên bản cụ thể cho các gói bạn cài đặt (ví dụ: `apt-get install -y nginx=1.20.1`).
- **Không lưu trữ bí mật (secrets) trong Dockerfile:** Tránh hardcode mật khẩu, khóa API, v.v. Sử dụng biến môi trường được truyền khi chạy container (cẩn thận với log), hoặc tốt hơn là sử dụng các giải pháp quản lý bí mật của orchestration system (Docker Swarm Secrets, Kubernetes Secrets). Tránh truyền bí mật qua `ARG` nếu không cần thiết, vì giá trị của `ARG` có thể hiển thị trong build history.
- **Quét lỗ hổng bảo mật:** Sử dụng các công cụ quét lỗ hổng cho Docker image (ví dụ: Trivy, Clair, Snyk) như một phần của pipeline CI/CD.

---
## 5. Các Mẹo Nhanh Để Nâng Tầm Dockerfile

|                   Mẹo                    |               Lệnh/Kỹ Thuật Liên Quan                |                           Lợi ích chính                           |
| :--------------------------------------: | :--------------------------------------------------: | :---------------------------------------------------------------: |
|        Chọn base image tối thiểu         |                        `FROM`                        |               Kích thước ảnh nhỏ, bảo mật cao hơn.                |
|          Kết hợp các lệnh `RUN`          |                 `RUN` (dùng `&& \`)                  |       Giảm số lượng layer, tận dụng cache, build nhanh hơn.       |
|         Dọn dẹp sau khi cài đặt          |             `RUN` (dùng `&& rm -rf ...`)             |            Giảm kích thước ảnh, giảm bề mặt tấn công.             |
|         Ưu tiên `COPY` hơn `ADD`         |                    `COPY`, `ADD`                     |                Dockerfile rõ ràng, tránh bất ngờ.                 |
|       Sử dụng tệp `.dockerignore`        |             Build context, `COPY`, `ADD`             |        Giảm build context, build nhanh hơn, tăng bảo mật.         |
| Dùng build đa tầng (Multi-stage builds)  |         Nhiều lệnh `FROM`, `COPY --from=...`         | Kích thước ảnh nhỏ, tách biệt môi trường build/run, tăng bảo mật. |
|          Tận dụng bộ nhớ cache           |                   Thứ tự các lệnh                    |                         Build nhanh hơn.                          |
|      Chạy với user không phải root       |               `USER`, `RUN useradd...`               |                           Tăng bảo mật.                           |
| Sử dụng dạng Exec cho `CMD`/`ENTRYPOINT` | `CMD`, `ENTRYPOINT` (dạng `["executable", "param"]`) |             Xử lý tín hiệu tốt hơn, PID 1 chính xác.              |
|            Sử dụng `WORKDIR`             |                      `WORKDIR`                       |      Dockerfile rõ ràng, tránh đường dẫn tuyệt đối lặp lại.       |

