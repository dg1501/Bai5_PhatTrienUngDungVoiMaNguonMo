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

---

## B. THỰC HÀNH ÁP DỤNG

Chúng ta sẽ tạo một thư mục dự án tên là **app-monitor**. Cấu trúc thư mục chuẩn cho bài này như sau:

```text
app-monitor/
├── docker-compose.yml
├── api/
│   ├── app.py
│   └── requirements.txt
├── web/
│   └── index.html
└── nginx/
    └── nginx.conf
```

### 1. Tạo thư mục dự án app-monitor

- Sử dụng lệnh `sudo mkdir app-monitor` để tạo thư mục dự án.</p>
- Lệnh `cd app-monitor` để vào thư mục dự án vừa tạo.

<img width="1350" height="735" alt="{44FC1FB4-92A8-42DA-8214-CA61174E0A42}" src="https://github.com/user-attachments/assets/766a549b-5917-4407-adf3-6c1e82a7e188" /></p>

---

### 2. Tạo File cấu hình hệ Thống docker-compose.yml

- Chạy lệnh `sudo nano docker-compose.yml`

- Sau đó gõ 1 ký tự bất kì vào File -> Mục đích để có thể khiến File tồn tại.

<img width="1346" height="734" alt="{FDA5830B-2745-4DC5-A693-9DB08D3F23EA}" src="https://github.com/user-attachments/assets/ed236369-7365-4f08-9940-bd548e7a81a2" /></p>

- Tiếp theo tiến hành thêm nội dung cho file

<img width="1349" height="741" alt="{DAB85491-416C-4909-B950-A0817EB67374}" src="https://github.com/user-attachments/assets/ff4781e6-2814-48d1-b9d8-c699075d8614" /></p>

- Chi tiết về từng Sevice trong File cấu hình: </p>

| Dịch vụ (Service) | Công nghệ / Hình ảnh | Tác dụng / Nhiệm vụ chính | Nội dung cấu hình kỹ thuật |
| :--- | :---: | :--- | :--- |
| **`mariadb`** | MariaDB 10.6 | Lưu trữ cấu hình, trạng thái hiện tại (on/off, online/offline) của thiết bị cần truy xuất nhanh. | Cổng `3306` • DB: `monitor_db` • Khởi động cùng hệ thống • Lưu dữ liệu bền vững qua volume `mariadb_data`. |
| **`influxdb`** | InfluxDB 1.8 | Lưu dữ liệu chuỗi thời gian (Time-series), gom các thông số đo lường liên tục (nhiệt độ, độ ẩm...) để vẽ lịch sử. | Cổng `8086` • DB: `history_db` • Khởi động cùng hệ thống • Lưu dữ liệu qua volume `influxdb_data`. |
| **`nodered`** | Node-RED | Trạm trung chuyển dữ liệu (thu thập từ IoT), ghi vào DB và xử lý logic để gửi cảnh báo qua Telegram. | Cổng `1880` • Lưu flows qua `nodered_data` • Chỉ chạy sau khi `mariadb` và `influxdb` đã sẵn sàng. |
| **`grafana`** | Grafana | Lấy dữ liệu từ 2 DB để trực quan hóa thành các biểu đồ, đồ thị, đồng hồ đo trên Dashboard. | Cổng `3000` • Cấu hình lưu qua `grafana_data` • Cho phép nhúng iframe và cho phép xem ẩn danh không cần login. |
| **`flask-api`** | Python 3.9-slim | Đọc dữ liệu từ MariaDB, xử lý logic Backend rồi trả về định dạng JSON sạch cho Frontend sử dụng. | Cổng `5000` • Đồng bộ code từ thư mục `./api` • Tự động cài `requirements.txt` và chạy `app.py` • Phụ thuộc vào `mariadb`. |
| **`nginx`** | Nginx Alpine | "Mặt tiền" của hệ thống, làm máy chủ chứa giao diện người dùng và điều hướng request (Reverse Proxy) đến Flask API. | Cổng `80` • Đồng bộ giao diện từ `./web` và file cấu hình `./nginx/default.conf` • Chỉ chạy sau khi `flask-api` đã sẵn sàng. |

---

### 3. Viết Flask API

- Tạo thư mục ***api*** bên trong ***app-monitor***

<img width="1352" height="732" alt="{DF1322FC-9461-4E78-A94B-B1F1FB50E6ED}" src="https://github.com/user-attachments/assets/14d5075c-f1fd-476c-bc76-01b4a8081737" /></p>

- Bên trong ***api** tạo 2 file ***app.py*** và ***requirements.txt***

<img width="1082" height="591" alt="{B3EA851A-0B72-48A9-92DB-98FE705FAEF1}" src="https://github.com/user-attachments/assets/9a54cafd-2422-4839-b42b-15585d6750c0" /></p>

File requirements.txt

- **Flask (Bộ khung API)**: Dùng để tạo Web Server ở cổng 5000 và định nghĩa các đường dẫn (như /api/realtime) để trả về dữ liệu dạng JSON cho giao diện.

- **Flask-CORS (Cửa gác bảo mật)**: Cho phép giao diện (Frontend chạy ở cổng 80 của Nginx) có quyền gọi và lấy dữ liệu từ Backend (chạy ở cổng 5000) mà không bị trình duyệt chặn lỗi bảo mật.

- **mysql-connector-python (Cầu nối Database)**: Là driver giúp code Python "nói chuyện" với MariaDB, gửi câu lệnh SQL (SELECT...) vào database để lấy dữ liệu về cho Flask xử lý.

<img width="1355" height="745" alt="{53849D8B-7730-4857-B6D3-E5CB722121DC}" src="https://github.com/user-attachments/assets/32d71f54-daa0-4eae-9e01-3ba0d618c956" /></p>

File app.py

- **CORS(app)**: Kích hoạt tính năng thông tuyến bảo mật, cho phép giao diện Frontend (Nginx cổng 80) thoải mái lấy dữ liệu từ Backend mà không bị trình duyệt chặn lỗi.

- **get_db_connection()**: Khởi tạo đường truyền tới MariaDB. Nhờ mạng nội bộ của Docker, nó có thể gọi thẳng tên host là "mariadb" thay vì dùng địa chỉ IP.

- **@app.route('/api/realtime')**: Tạo một đường dẫn API (endpoint). Khi có ai truy cập vào, nó sẽ chạy câu lệnh SQL để móc dữ liệu giá vàng mới nhất (LIMIT 1) trong database ra.

- **jsonify(result)**: Chuyển đổi dữ liệu thô lấy từ MariaDB thành định dạng chuỗi JSON tiêu chuẩn và gửi trả về cho Frontend hiển thị lên màn hình.

<img width="1351" height="743" alt="{83041228-2030-4BA8-A920-FC8C110BD999}" src="https://github.com/user-attachments/assets/721eeebe-937d-4f24-a32f-c76701b88cf6" /></p>

---

### 4. Cấu Hình Nginx và Front-end HTML/JS

**Bước 1:** Tạo thư mục nginx và file nginx.conf để cấu hình Nginx tránh lỗi bảo mật khi gọi Iframe hoặc API:

<img width="1353" height="740" alt="{CCED9159-45BD-488C-927E-4111FCDEB513}" src="https://github.com/user-attachments/assets/e766d432-77da-43b7-a8f1-2d62d6a966bc" /></p>

Thêm nội dung

<img width="1345" height="726" alt="{DE06CDA8-3719-483C-A729-08FA8A688816}" src="https://github.com/user-attachments/assets/79ea044f-b2b7-4879-beb3-4ef7f9d15779" /></p>

- listen 80: Mở cổng mạng 80 (cổng HTTP mặc định) để người dùng có thể truy cập vào giao diện trang web từ trình duyệt.

- server_name localhost: Định danh tên miền cho máy chủ là localhost (truy cập trực tiếp bằng IP máy hoặc qua cổng local).

- location /: Tiếp nhận tất cả các yêu cầu truy cập thông thường đổ vào trang chủ (đường dẫn gốc /).

- root và index: Chỉ định thư mục chứa giao diện là /usr/share/nginx/html và tự động tải file index.html lên màn hình khi người dùng truy cập.

**Bước 2:** Tạo thư mục web và file web/index.html:

<img width="1095" height="629" alt="{4789A514-E45B-4790-B42E-3728E6F4E593}" src="https://github.com/user-attachments/assets/06fd27a3-9089-4221-9555-e075b9b7a60a" /></p>

Thêm nội dung:

<img width="1354" height="734" alt="{70734F18-6F4F-4C3D-B599-63B703BF859C}" src="https://github.com/user-attachments/assets/3c2a8629-963a-45fe-a057-ed68a638034a" /></p>

**Bước 3**: Bước 4: Logic và Cấu hình Node-RED (Lấy dữ liệu, Lưu DB, Bắn Telegram)

- Sau khi chạy lệnh khởi động `docker compose up -d`
  
<img width="1361" height="720" alt="{07F03A40-E6C8-4A82-8A01-E58D9B2FA54A}" src="https://github.com/user-attachments/assets/5ef5bb38-d9cf-45f9-a922-5553bb68a693" /></p>

<img width="1348" height="709" alt="{BDC92DD6-5E50-4953-B7BD-F4104E05F4AC}" src="https://github.com/user-attachments/assets/1fef0cbf-3a46-4b11-a71e-7ec011398da1" /></p>

- Vào trình duyệt gõ `http://<IP_UBUNTU>:1882` để cấu hình Node-RED bằng giao diện kéo thả.

<img width="1920" height="1026" alt="{AADF4540-2496-4733-8EB3-78865F29F987}" src="https://github.com/user-attachments/assets/d21dc690-913c-4a6f-8688-428f917606e5" /></p>

**Bước 4**: Cài đặt thư viện phụ trợ trong Node-RED:

Vào góc phải trên cùng -> Manage palette -> Thẻ Install -> Tìm và cài các node:

- node-red-node-mysql (Kết nối MariaDB)
  
<img width="1277" height="691" alt="{6BAE5BAB-FDEE-4A28-8E5D-E33140B48FC7}" src="https://github.com/user-attachments/assets/f35a7a5f-7d72-4933-aafc-d95d9815018e" /></p>

- node-red-contrib-influxdb (Kết nối InfluxDB)

<img width="1593" height="738" alt="{04DDCF93-07AF-4A86-8816-9D7B0960E854}" src="https://github.com/user-attachments/assets/b6a544c1-3c79-4cae-897f-5e733455abb9" /></p>

- node-red-contrib-telegrambot (Gửi tin nhắn Telegram)

<img width="1277" height="681" alt="{55196EB6-6A84-446D-ACE7-8FD90531679E}" src="https://github.com/user-attachments/assets/67b07010-50d2-4b2e-a00c-9402e75b846c" /></p>

**Bước 5**: Luồng xử lý dữ liệu (Kéo thả Node)

- Node Inject (Mốc thời gian): Cấu hình lặp lại sau mỗi 5 giây (Interval: every 5 seconds).

<img width="1920" height="1022" alt="{A21E833F-51B6-40E2-9196-91756C2DCA84}" src="https://github.com/user-attachments/assets/44596768-8fd4-4799-9492-f759f513e203" /></p>

- Node HTTP Request: Gọi API lấy giá vàng/chứng khoán thực tế. Bạn có thể dùng API miễn phí như: https://api.coindesk.com/v1/bpi/currentprice.json (Lấy giá Bitcoin).

<img width="1920" height="1027" alt="{A45CD169-FA98-4FC2-A01B-0F35B52DB69A}" src="https://github.com/user-attachments/assets/57711da9-0f43-41e2-ac61-65180bf3fa7c" /></p>

- Node Function (Phân tách dữ liệu): Viết code JS nhỏ để lấy giá trị số:

<img width="1920" height="1026" alt="{3A8664C0-ACC2-4159-959B-21F951C37A93}" src="https://github.com/user-attachments/assets/1b240de5-4ac3-4fe8-9511-e8b0b8e8e08d" /></p>
