## 1. Nhóm Quản Lý Image
### a. `docker build` – Đóng gói Image từ Dockerfile

**Cú pháp:** `docker build [OPTIONS] PATH`

Các OPTIONS quan trọng:

- **`-t` (`--tag`):** Đặt tên và phiên bản cho Image.
	-  _Ví dụ:_  `docker build -t my-backend:v1.0 .`

- **`-f` (`--file`):** Chỉ định file Dockerfile khác tên mặc định.
	-  _Ví dụ:_  `docker build -t my-backend:dev -f Dockerfile.dev .`

- **`--build-arg`:** Truyền giá trị vào biến `ARG` khai báo trong Dockerfile.
	-  _Ví dụ:_  `docker build -t my-app --build-arg APP_PORT=8080 .`

- **`--no-cache`:** Ép Docker build lại từ đầu, không dùng lại cache cũ.
	- _Ví dụ:_  `docker build --no-cache -t my-app:latest .`

- **`--target`:** Chỉ build đến 1 Stage cụ thể trong file Multi-stage Dockerfile.
	- _Ví dụ:_  `docker build --target builder -t my-app:builder-only .`

### b. `docker pull` – Tải Image từ Docker Hub / Registry về máy

**Cú pháp:** `docker pull NAME[:TAG|@DIGEST]`

- Nếu không chỉ định TAG, Docker sẽ sử dụng tag mặc định là `latest`
	- _Ví dụ:_  `docker pull redis`

- Tải phiên bản cụ thể bằng TAG
	- _Ví dụ:_  `docker pull mysql:8.0`

### c. `docker push` – Đẩy Image từ máy lên Docker Hub / Registry

- Đăng nhập vào tài khoản Docker Hub trên Terminal:   `docker login`
- Đổi tag cho image với tiền tố là tên acc trên DockerHub `hungtt/` :  `docker tag backend:v1.0 hungtt/backend:v1.0`
- Đẩy Image đã tag lên Docker Hub:   `docker push hungtt/backend:v1.0`

### d. `docker images` – Xem danh sách Image hiện có trên máy

### e. `docker rmi` – Xóa Image khỏi máy

**Cú pháp:** `docker rmi [OPTIONS] IMAGE [IMAGE...]`

Các OPTIONS quan trọng:

- Xóa 1 Image bằng tên hoặc ID khi không có container nào dùng
	- _Ví dụ:_  `docker rmi mysql:8.0`

- **`-f` (`--force`):** Ép xóa Image kể cả khi đang có container trỏ tới nó
	- _Ví dụ:_  `docker rmi -f redis:alpine`

### f. `docker history` –  Xem lịch sử tạo layers trong image

Hiển thị lịch sử các layer được tạo và các lệnh Dockerfile đã tạo nên Image

**Cú pháp:** `docker history mysql:8.0`

---
## 2. Nhóm Quản Lý Container
### a. `docker run` – Khởi tạo và chạy Container mới

**Cú pháp:** `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

Các OPTIONS quan trọng:

- **`-d (--detach)`:** Chạy container ở chế độ nền (Background / Detached mode)
	- _Ví dụ:_  `docker run -d nginx:alpine`

- **`-p <host_port>:<container_port>`:** Map cổng từ máy thật (Host) vào cổng bên trong Container
	- _Ví dụ:_  `docker run -p 8080:80 nginx:alpine` (Mở cổng 8080 máy thật sẽ trỏ vào cổng 80 của container)

- **`--name <string>`:** Đặt tên tùy chỉnh cho Container (nếu không truyền, Docker sẽ tự đặt tên ngẫu nhiên).
	-  _Ví dụ:_  `docker run --name my-web-server nginx:alpine`

* **Volume (`-v volume_name:container_path`)**: dùng để lưu dữ liệu lâu dài (database, Redis), an toàn và không mất khi container bị tắt hoặc xóa.
	* _Ví dụ:_  `docker run -d -v mysql-data:/var/lib/mysql mysql:8.0` (Docker sẽ tự quản lý thư mục lưu trữ của Volume (mặc định trên Linux là `/var/lib/docker/volumes/`).)

* **Bind Mount (`-v host_path:container_path`)**: dùng khi đang phát triển ứng dụng, muốn chỉnh sửa file trên máy và container cập nhật ngay.
	* _Ví dụ:_  `docker run -d -v $(pwd)/src:/app/src node:20` (thư mục trên Host được đồng bộ trực tiếp với thư mục trong Container do người dùng chỉ định).

* **`-e (--env)`:** Truyền biến môi trường vào Container
	* _Ví dụ:_  `docker run -d -e MYSQL_ROOT_PASSWORD=123456 mysql:8.0`

- **`--env-file`:** Truyền danh sách biến môi trường từ 1 file `.env`
	* _Ví dụ:_  `docker run -d --env-file .env my-spring-app:latest`

- **`--network`:** Chỉ định Docker Network mà container sẽ tham gia.
	- _Ví dụ:_  `docker run -d --network my-app-net my-java-app`

- **`-it`:** Kết hợp `-i` (nhập dữ liệu) và `-t` (cấp Terminal ảo) để tương tác trực tiếp.
	- _Ví dụ:_  `docker run -it ubuntu:24.04 /bin/bash`


> [!NOTE]
>  **Khi chạy `docker run`, nếu image chưa tồn tại trên máy local thì Docker sẽ tự động pull image từ Docker Hub (hoặc registry được cấu hình) trước, rồi mới tạo và chạy container.**

### b. `docker ps` – Liệt kê Container

**Cú pháp:** `docker ps [OPTIONS]`

Các OPTIONS quan trọng:

- _(Không tham số):_ Liệt kê các container **đang chạy**.
	- _Ví dụ:_  `docker ps`

- **`-a (--all)`:** Liệt kê **tất cả** container (bao gồm cả các container đã `Exited`/dừng)
	- _Ví dụ:_  `docker ps -a`

- **`-q (--quiet)`:** Chỉ hiển thị duy nhất danh sách ID của Container.
	- _Ví dụ:_  `docker ps -q`

* **`-f (--filter)`:** Lọc danh sách theo trạng thái (`status`), tên (`name`),...
	- _Ví dụ:_  `docker ps -a -f status=exited`

### c. `docker start` – Khởi động lại Container đã bị dừng

**Cú pháp:** `docker start [OPTIONS] CONTAINER [CONTAINER...]`

### d. `docker stop` – Dừng Container đang chạy một cách an toàn

**Cú pháp:** `docker stop [OPTIONS] CONTAINER [CONTAINER...]`

### e. `docker restart` – Khởi động lại Container

**Cú pháp:** `docker restart [OPTIONS] CONTAINER [CONTAINER...]`

### f. `docker rm` – Xóa một hoặc nhiều Container

**Cú pháp:** `docker rm [OPTIONS] CONTAINER [CONTAINER...]`

Các OPTIONS quan trọng:

- Xóa 1 container đã dừng
	- _Ví dụ:_  `docker rm my-app-container`

- `-f` (`--force`): Ép xóa container đang chạy mà không cần stop trước
	- _Ví dụ:_  `docker rm -f my-app-container`

- `-v` (`--volumes`): Xóa container kèm theo các Anonymous Volume gắn với nó
	- _Ví dụ:_  `docker rm -fv my-app-container`

- Lệnh chạy kết hợp để dọn dẹp nhanh các container bị tắt
	- _Ví dụ:_  `docker rm $(docker ps -a -q -f status=exited)`

### g. `docker logs` – Xem log hoạt động của Container

**Cú pháp:** `docker logs [OPTIONS] CONTAINER`

Các OPTIONS quan trọng:

- **`-f` (`--follow`):** Theo dõi log liên tục theo thời gian thực (Real-time).
	- _Ví dụ:_  `docker logs -f my-java-app`

- **`--tail`:** Chỉ hiển thị số dòng log mới nhất tính từ cuối lên.
	- _Ví dụ:_  `docker logs --tail 50 my-java-app`

- **`-t` (`--timestamps`):** Hiển thị thêm mốc thời gian thực hiện log.
	- _Ví dụ:_  `docker logs -t my-java-app`

### h. `docker exec` – Chạy câu lệnh bên trong Container đang hoạt động

**Cú pháp:** `docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`

Các OPTIONS quan trọng:

- **`-it`:** Mở giao diện dòng lệnh (Shell/Bash) tương tác bên trong Container.
	- _Ví dụ:_  `docker exec -it sachxanh-db bash`(Nếu OS trong container không có bash, thay bằng sh)

### i. **`docker stats`:** Xem CPU, RAM, Network I/O của các Container

**Cú pháp:** `docker stats`

### j. `docker cp`: Dùng để copy file/folder giữa Host và Container

**Cú pháp:**  `docker cp [OPTIONS] SRC_PATH DEST_PATH`

* Copy từ Host → Container:  `docker cp <host_path> <container_name>:<container_path>`
	-  _Ví dụ:_ `docker cp app.jar my-java-app:/app/`

- Copy từ Container → Host:  `docker cp <container_name>:<container_path> <host_path>`
	-  _Ví dụ:_ `docker cp my-java-app:/app/logs/app.log .`

---
## 3. Nhóm Quản Lý Volume
### a. Tạo Volume

**Cú pháp:** `docker volume create VOLUME`

### b. Xem danh sách Volume

**Cú pháp:** `docker volume ls`

### c. Xóa 1 Volume cụ thể

**Cú pháp:** `docker volume rm VOLUME`

---
## 4. Nhóm Quản Lý Network

### a. Tạo một Docker Network mới

**Cú pháp:**  `docker network create sachxanh-net`

### b. Xem danh sách các Network hiện có

**Cú pháp:**  `docker network ls`

### c. Xem chi tiết cấu hình và danh sách Container trong Network

**Cú pháp:**  `docker network inspect sach-xanh`

### d. Xóa một hoặc nhiều Network

**Cú pháp:**  `docker network rm NETWORK`

---
## 5. `docker inspect` – Xem thông tin chi tiết của Docker Object

Dùng để xem toàn bộ thông tin cấu hình và trạng thái của **Image, Container, Volume hoặc Network** dưới dạng JSON.

**Cú pháp:** `docker inspect OBJECT_NAME_OR_ID`

Ví dụ: 
- docker image inspect mysql:8.0
- docker container inspect my-container
- docker volume inspect mysql-data
- docker network inspect app-network

Thường dùng để xem: 
- Địa chỉ IP của Container 
- Ports đang publish 
- Environment Variables (ENV)
- Volume đang mount 
- Network đang kết nối 
- Trạng thái Container 
- Mountpoint của Volume
- Cấu hình của Image

---
## 6. Dọn dẹp tài nguyên hệ thống

Sau một thời gian sử dụng, Docker sẽ tích tụ các container đã dừng, image không còn sử dụng, build cache và network không dùng gây tốn dung lượng.

- Xem mức độ chiếm dụng ổ cứng của Docker:   `docker system df`

- Lệnh dọn dẹp:   `docker system prune`

