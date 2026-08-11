## 1. Tại Sao Cần Lưu Trữ Bền Vững cho Container

Như chúng ta đã biết, container được thiết kế để có tính tạm thời (ephemeral). Khi một container bị dừng hoặc xóa, mọi thay đổi được ghi vào lớp filesystem của container sẽ bị mất. Điều này hoàn toàn phù hợp cho các thành phần ứng dụng không trạng thái (stateless), nhưng lại là vấn đề lớn đối với các ứng dụng cần lưu trữ dữ liệu lâu dài như cơ sở dữ liệu, logs, hoặc dữ liệu người dùng. Để giải quyết bài toán này, Docker cung cấp các cơ chế để container có thể truy cập và lưu trữ dữ liệu bên ngoài vòng đời của chính nó. Hai cơ chế chính là Volume Mounts và Bind Mounts.

Việc hiểu rõ sự khác biệt và trường hợp sử dụng của từng loại mount là cực kỳ quan trọng. Nó không chỉ ảnh hưởng đến hiệu suất và tính di động của ứng dụng mà còn liên quan đến bảo mật và quản lý dữ liệu. Trong bài viết này, chúng ta sẽ “mổ xẻ” từng loại, so sánh chúng và đưa ra những lời khuyên thực tế dựa trên kinh nghiệm.

---
## 2. Volume Mounts: Giải Pháp Lưu Trữ Được Quản Lý Bởi Docker
### a. Volume Mounts là gì

Volume Mounts là cách được khuyến nghị để lưu trữ dữ liệu bền vững cho các ứng dụng Docker. Volumes là các khu vực lưu trữ được tạo và quản lý hoàn toàn bởi Docker. Khi bạn tạo một volume, Docker sẽ tạo một thư mục trên hệ thống host và quản lý quyền truy cập cũng như vòng đời của thư mục đó.

### b. Cách Thức Hoạt Động của Volume Mounts

Khi bạn sử dụng volume mount cho một container, Docker sẽ “gắn” (mount) volume đó vào một đường dẫn cụ thể bên trong filesystem của container. Dữ liệu được ghi vào đường dẫn đó trong container thực chất sẽ được lưu trữ trong thư mục tương ứng của volume trên host.

Điểm khác biệt cốt lõi là Docker hoàn toàn chịu trách nhiệm quản lý vị trí vật lý của volume trên host. Bạn không cần quan tâm nó được lưu ở đâu (thường nằm trong thư mục `/var/lib/docker/volumes/` trên Linux, nhưng vị trí này có thể thay đổi và không nên truy cập trực tiếp từ bên ngoài Docker). Docker đảm bảo volume được tạo, gắn và xóa đúng cách khi bạn quản lý các container sử dụng nó.

### c. Khi Nào Nên Sử Dụng Volume Mounts

Volume Mounts là lựa chọn mặc định cho hầu hết các trường hợp cần lưu trữ dữ liệu bền vững, bao gồm:

1. **Lưu trữ dữ liệu cơ sở dữ liệu:** Các CSDL như PostgreSQL, MySQL, MongoDB… cần lưu trữ dữ liệu một cách an toàn và hiệu quả. Volumes cung cấp hiệu suất tốt và đảm bảo dữ liệu không bị mất khi container CSDL được cập nhật hoặc khởi động lại.
2. **Chia sẻ dữ liệu giữa các container:** Nhiều container có thể mount cùng một volume để chia sẻ dữ liệu một cách dễ dàng. Ví dụ, một container web server có thể mount volume chứa các file tĩnh được tạo bởi một container khác.
3. **Sao lưu, phục hồi và di chuyển dữ liệu:** Vì volumes có vòng đời độc lập với container, việc sao lưu, phục hồi hoặc di chuyển dữ liệu giữa các môi trường trở nên đơn giản hơn nhiều bằng cách quản lý volume.
4. **Lưu trữ dữ liệu cho các ứng dụng trạng thái (stateful applications):** Bất kỳ ứng dụng nào cần lưu trữ trạng thái hoặc dữ liệu lâu dài đều nên sử dụng volume.

### d. Cách Sử Dụng Volume Mounts

Bạn có thể sử dụng volume mounts với cú pháp `-v` hoặc `–mount` trong lệnh `docker run`. Cú pháp `–mount` được khuyến khích hơn vì nó rõ ràng và chi tiết hơn.

**Sử dụng cú pháp `-v`:**

```bash
docker run -d --name my-db -v my-database-volume:/var/lib/mysql mysql:latest
```

Trong ví dụ này:

- `my-database-volume`: Tên của volume trên host (nếu chưa tồn tại, Docker sẽ tự tạo).
- `/var/lib/mysql`: Đường dẫn bên trong container mà volume sẽ được mount vào.

**Sử dụng cú pháp `–mount`:**

```bash
docker run -d --name my-db --mount type=volume,source=my-database-volume,target=/var/lib/mysql mysql:latest
```

Trong cú pháp `–mount`:

- `type=volume`: Chỉ định loại mount là volume.
- `source=my-database-volume`: Tên của volume (hoặc ID volume nếu bạn đã tạo trước).
- `target=/var/lib/mysql`: Đường dẫn bên trong container.

Bạn cũng có thể tạo volume trước bằng lệnh `docker volume create`:

```bash
docker volume create my-database-volume
docker run -d --name my-db --mount type=volume,source=my-database-volume,target=/var/lib/mysql mysql:latest
```

### e. Ưu Điểm của Volume Mounts

- **Dễ quản lý:** Docker quản lý vòng đời của volume, bao gồm tạo, gắn, và xóa.
- **Hiệu suất tốt:** Volumes được lưu trữ ở một vị trí tối ưu hóa cho Docker, có thể cung cấp hiệu suất tốt hơn trên một số hệ thống file hoặc với các trình điều khiển lưu trữ (storage drivers) chuyên biệt.
- **An toàn hơn:** Vị trí của volume trên host được Docker quản lý, giảm thiểu rủi ro container truy cập hoặc sửa đổi các file hệ thống quan trọng của host. Docker cũng quản lý quyền sở hữu và quyền hạn (permissions) một cách hiệu quả. 
- **Tính di động:** Volumes có thể được sử dụng với các trình điều khiển lưu trữ khác nhau (như NFS, cloud storage drivers) để di chuyển dữ liệu dễ dàng giữa các host hoặc sử dụng trong môi trường swarm/Kubernetes.
- **Khởi tạo dữ liệu:** Khi mount một volume trống vào một thư mục trong container đã có dữ liệu (ví dụ: thư mục dữ liệu mặc định của CSDL trong image), Docker sẽ tự động copy dữ liệu từ image vào volume. Tính năng này rất hữu ích khi khởi tạo volume cho lần đầu tiên.

### f. Nhược Điểm của Volume Mounts

- **Khó truy cập trực tiếp từ host:** Vì Docker quản lý vị trí, việc truy cập và chỉnh sửa nội dung volume trực tiếp từ hệ thống host không đơn giản và không được khuyến khích.

---
## 3. Bind Mounts: Kết Nối Trực Tiếp Thư Mục Host
### a. Bind Mounts là gì

Bind Mounts là một cơ chế lưu trữ dữ liệu trong Docker đã tồn tại từ rất lâu (từ những ngày đầu của Docker). Thay vì tạo và quản lý một khu vực lưu trữ riêng như volume, bind mount kết nối trực tiếp một thư mục (hoặc file) trên hệ thống host vào một đường dẫn bên trong container.

### b. Cách Thức Hoạt Động của Bind Mounts

Với bind mount, Docker chỉ đơn giản là ánh xạ (map) một đường dẫn tuyệt đối trên host (ví dụ: `/home/user/my-app/data`) tới một đường dẫn bên trong container (ví dụ: `/app/data`). Mọi thao tác đọc/ghi dữ liệu trong thư mục `/app/data` của container sẽ trực tiếp ảnh hưởng đến nội dung của thư mục `/home/user/my-app/data` trên host.

Docker không quản lý vòng đời của thư mục trên host. Thư mục này phải tồn tại trước khi container được chạy hoặc Docker sẽ tạo nó (tùy thuộc vào phiên bản và cấu hình Docker daemon). Quyền truy cập và quản lý thư mục này hoàn toàn thuộc về hệ điều hành host và người dùng chạy Docker

### c. Khi Nào Nên Sử Dụng Bind Mounts

Bind Mounts thường được sử dụng trong các trường hợp sau:

1. **Phát triển ứng dụng:** Đây là trường hợp sử dụng phổ biến nhất. Bạn có thể bind mount mã nguồn ứng dụng từ host vào container. Khi bạn chỉnh sửa code trên host, các thay đổi sẽ ngay lập tức phản ánh trong container (đặc biệt hữu ích với các công cụ hot-reloading), giúp tăng tốc quá trình phát triển.
2. **Chia sẻ file cấu hình:** Mount file cấu hình từ host vào container để dễ dàng quản lý tập trung và cập nhật cấu hình mà không cần build lại image.
3. **Truy cập các tiện ích host:** Mount các file hoặc thư mục hệ thống từ host vào container (cần thận trọng về bảo mật). Ví dụ: mount `/etc/resolv.conf` để container sử dụng cấu hình DNS của host.
4. **Xem logs hoặc dữ liệu sinh ra trên host:** Mặc dù không phải là cách tốt nhất để quản lý logs trong môi trường production, bạn có thể bind mount một thư mục logs từ container ra host để dễ dàng xem bằng các công cụ trên host.

### d. Cách Sử Dụng Bind Mounts

Tương tự volume mounts, bind mounts cũng sử dụng cú pháp `-v` hoặc `–mount`.

**Sử dụng cú pháp `-v`:**

```bash
docker run -d --name my-app -p 8080:80 -v /path/to/your/app/code:/app/code my-app-image
```

Trong ví dụ này:

- `/path/to/your/app/code`: Đường dẫn tuyệt đối trên host.
- `/app/code`: Đường dẫn bên trong container.

**Sử dụng cú pháp `–mount`:**

```bash
docker run -d --name my-app -p 8080:80 --mount type=bind,source=/path/to/your/app/code,target=/app/code my-app-image
```

Trong cú pháp `–mount`:

- `type=bind`: Chỉ định loại mount là bind.
- `source=/path/to/your/app/code`: Đường dẫn tuyệt đối trên host.
- `target=/app/code`: Đường dẫn bên trong container.

Lưu ý: Với bind mounts, đường dẫn `source` trên host phải là đường dẫn tuyệt đối. Nếu bạn sử dụng đường dẫn tương đối với `-v`, Docker sẽ hiểu đó là tên volume.

### e. Ưu Điểm của Bind Mounts

- **Dễ dàng truy cập từ host:** Bạn có thể trực tiếp truy cập và chỉnh sửa dữ liệu trên host bằng bất kỳ công cụ nào.
- **Lý tưởng cho phát triển:** Thay đổi code trên host được cập nhật ngay lập tức trong container, rất tiện lợi cho vòng lặp phát triển nhanh.
- **Không cần Docker quản lý:** Sử dụng các thư mục host hiện có mà không cần Docker tạo và quản lý.

### f. Nhược Điểm của Bind Mounts

- **Phụ thuộc vào cấu trúc thư mục host:** Container sẽ không hoạt động nếu đường dẫn nguồn trên host không tồn tại hoặc có cấu trúc khác. Điều này làm giảm tính di động của ứng dụng.
- **Rủi ro bảo mật:** Container có thể truy cập và ghi vào các file/thư mục quan trọng trên host nếu bind mount được cấu hình không cẩn thận.
- **Hiệu suất có thể kém hơn:** Hiệu suất có thể bị ảnh hưởng nếu bind mount được sử dụng trên các hệ thống file phân tán (distributed filesystems) hoặc trong các môi trường phức tạp.
- **Tính di động kém:** Bind mounts phụ thuộc vào hệ điều hành và cấu trúc file của host, khiến việc di chuyển ứng dụng sang các host khác khó khăn hơn.

---
## 4. Mounting Volumes sử dụng Dockerfile

Mặc dù bạn có thể định nghĩa volumes trong Dockerfile bằng lệnh `VOLUME`, điều này chỉ tạo ra một mount point với volume ẩn danh (anonymous volume) hoặc volume có tên. Việc gắn các volume hoặc bind mounts cụ thể thường được thực hiện khi chạy container (`docker run` hoặc trong file Docker Compose) chứ không phải trong Dockerfile. Lệnh `VOLUME` trong Dockerfile chủ yếu dùng để chỉ ra rằng thư mục đó sẽ chứa dữ liệu biến đổi và nên được mount từ bên ngoài.

```bash
# Dockerfile
FROM ubuntu
RUN mkdir /data
RUN echo "Hello from image" > /data/greeting.txt
VOLUME /data
CMD ["cat", "/data/greeting.txt"]
```

Khi chạy container từ image này mà không chỉ định mount gì vào `/data`:

```bash
docker run my-image
# Output: Hello from image
# Docker tự động tạo một anonymous volume và mount vào /data.
# Nội dung "Hello from image" từ image sẽ được copy vào volume.
# Volume này sẽ tồn tại sau khi container dừng.
```

Khi chạy container và mount một volume có tên:

```bash
docker run --name my-container --mount type=volume,source=my-named-volume,target=/data my-image
# Docker sẽ mount volume "my-named-volume" vào /data.
# Nếu my-named-volume trống, nội dung từ /data trong image ("Hello from image") sẽ được copy vào đó.
```

Khi chạy container và bind mount một thư mục host:

```bash
mkdir host_data
echo "Hello from host" > host_data/greeting.txt
docker run --name my-container --mount type=bind,source=$(pwd)/host_data,target=/data my-image
# Output: Hello from host
# Nội dung của thư mục bind mount từ host sẽ che khuất nội dung gốc trong image.
```

Sử dụng `VOLUME` trong Dockerfile là một cách tốt để báo hiệu ý định của image, nhưng việc cấu hình lưu trữ cụ thể (volume có tên hay bind mount) thường là trách nhiệm của người vận hành khi chạy container.

---
### 5. Tmpfs Mounts: Lưu Trữ Trên RAM

Ngoài volume mounts và bind mounts, Docker còn hỗ trợ `tmpfs` mounts. `tmpfs` mounts cho phép container ghi dữ liệu vào bộ nhớ RAM của host. Dữ liệu này chỉ tồn tại trong bộ nhớ và sẽ bị mất khi container dừng. `tmpfs` mounts rất hữu ích cho các trường hợp cần lưu trữ dữ liệu tạm thời nhạy cảm hoặc cần tốc độ truy cập cực cao mà không quan tâm đến tính bền vững, ví dụ như các file nhạy cảm không muốn ghi xuống đĩa cứng, hoặc các file cache tạm thời.

```bash
docker run -d --name my-temp-app --mount type=tmpfs,target=/app/cache ubuntu:latest
```

Trong ví dụ này, thư mục `/app/cache` bên trong container sẽ được mount trên RAM của host.

