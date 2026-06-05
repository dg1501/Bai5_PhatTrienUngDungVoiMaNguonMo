## PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (BÀI 05: 
### HỌ TÊN: NGUYỄN ĐỨC DƯƠNG
### LỚP: K58KTP
### MSSV: K225480106093

---

## A. LÝ THUYẾT

### 1. Docker là gì? 

Docker là một nền tảng mã nguồn mở cho phép bạn đóng gói ứng dụng và tất cả các thành phần phụ thuộc của nó (thư viện, môi trường, cấu hình...) thành một đơn vị duy nhất gọi là Container.</p>

Hiểu đơn giản: Nếu máy ảo (Virtual Machine trên Hyper-V) là một ngôi nhà hoàn chỉnh (nặng, cần hệ điều hành riêng), thì Docker Container giống như một căn hộ chung cư mini được lắp ghép sẵn. Nó dùng chung nền móng (Kernel hệ điều hành của máy host) nên cực kỳ nhẹ, khởi động trong vài giây và chạy đâu cũng giống nhau.</p>

### 2. Các Keyword trong Docker-compose.yml

Docker-compose là công cụ giúp bạn quản lý và chạy hệ thống gồm nhiều container (như bài tập này) thông qua một file cấu hình duy nhất.</p>

- **Thành phần chính**

***version***: Định nghĩa phiên bản của file docker-compose.

***services***: Định nghĩa các container (dịch vụ) sẽ chạy.

***networks***: Định nghĩa mạng nội bộ để các container giao tiếp với nhau.

***volumes***: Định nghĩa nơi lưu trữ dữ liệu bền vững (không bị mất khi container bị xóa).

- **Các keyword chi tiết mô tả Service:**

| Từ khóa   | Ý nghĩa   | Ví dụ minh họa |
| --------- | --------- | --------- |
| image    | Chỉ định Docker Image được dùng để tạo container (tải từ Docker Hub). | image: mariadb:10.6 |
| container_nam    | Đặt tên cố định cho container thay vì để Docker tự sinh tên ngẫu nhiên. | container_name: my_mariadb |
| ports    | Ánh xạ cổng từ Máy thật (Host) vào bên trong Container theo cấu hình Host:Container. | ports: - "8080:80" (Vào web qua cổng 8080 của máy thật). |
| environment    | Khai báo các biến môi trường (cấu hình, password, tài khoản...). | environment: - MYSQL_ROOT_PASSWORD=secret |
| volumes    | Gắn một thư mục máy thật vào container để lưu dữ liệu hoặc đồng bộ code. | volumes: - ./html:/usr/share/nginx/html |
| networks    | Chỉ định container này thuộc mạng nội bộ nào. | networks: - monitor-net |
| depends_on    | Quy định thứ tự khởi động (Container A phải chạy trước container B). | depends_on: - mariadb (Flask đợi MariaDB chạy xong mới chạy). |
| restart    | Tự động khởi động lại container nếu bị lỗi hoặc máy chủ reboot. | restart: always |

### 3. Ưu điểm khi triển khai ứng dụng bằng Docker

- **Tính nhất quán (Consistency)**: Giải quyết triệt để câu nói "Nhưng trên máy code của em vẫn chạy được mà!". Chạy trên laptop cá nhân hay máy chủ thật đều y hệt nhau.

- **Tiết kiệm tài nguyên**: Container chia sẻ tài nguyên phần cứng trực tiếp với máy host, không tốn tài nguyên cho hệ điều hành ảo hóa như Hyper-V.

- **Triển khai nhanh chóng (Speed)**: Tạo mới, hủy bỏ, nâng cấp ứng dụng chỉ mất vài giây bằng câu lệnh.

- **Cách ly an toàn (Isolation)**: Mỗi container là một môi trường độc lập. Lỗi của container này không làm sập container khác.

### 4. Quy trình triển khai App lên Máy chủ thật KHÔNG CÓ INTERNET (Môi trường Air-gapped)

Khi máy chủ thật không có internet, bạn không thể dùng lệnh docker pull để tải image từ mạng. Bạn cần làm như sau trên Laptop cá nhân (có internet) trước:</p>

- Bước 1: Tải các image cần thiết về laptop: docker pull nginx:latest

- Bước 2: Đóng gói image thành file nén .tar: `docker save -o nginx_image.tar nginx:latest`

- Bước 3: Dùng USB, ổ cứng di động sao chép các file .tar và toàn bộ thư mục chứa mã nguồn (docker-compose.yml, code Flask, code HTML...) sang máy chủ thật.

- Bước 4: Tại máy chủ thật, giải nén và nạp image vào Docker: `docker load -i nginx_image.tar`

- Bước 5: Chạy ứng dụng bằng lệnh: `docker compose up -d`
