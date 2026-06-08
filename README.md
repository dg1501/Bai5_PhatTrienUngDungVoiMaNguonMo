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

- Tạo Bảng trong MariaDB: Dùng Node MySQL chạy câu lệnh này một lần duy nhất để tạo bảng:

<img width="1920" height="1030" alt="{E50DE4E9-F7DD-479E-AD4F-A75F1AAD9B76}" src="https://github.com/user-attachments/assets/fa4bbf0b-355e-41e5-a460-b25ae5a6755b" /></p>

- Thêm node template vào Flow

<img width="1920" height="1027" alt="{A6AB7496-D598-4A0B-9880-680A70AF051A}" src="https://github.com/user-attachments/assets/b65af825-0d36-4339-901d-573b916f6692" /></p>

-> Sơ đồ cuối cùng 

<img width="1920" height="1032" alt="{820AAA33-30EB-4F73-8C16-97BD41644D81}" src="https://github.com/user-attachments/assets/9ae08e29-3f14-465f-8d8b-20bb646e63bc" /></p>

Luồng A: Tạo bảng MariaDB

Luồng B: Lấy dữ liệu API và Lưu vào các Database

**Bước 6**: Logic Kiểm tra Dữ Liệu và Bắn Alert Telegram

a) Chuẩn bị thông tin Telegram (Lấy Token và Chat ID)

- Lấy Bot Token: * Bạn mở ứng dụng Telegram, tìm kiếm @BotFather (có tích xanh). Gõ lệnh /newbot, đặt tên cho bot và đặt username cho bot (phải kết thúc bằng chữ bot, ví dụ: duong_monitor_bot).

- @BotFather sẽ gửi cho bạn một chuỗi kí tự gọi là HTTP API Token (Dạng như: 123456789:ABCdefGhIJKlmNoPQRsTUV). Hãy lưu lại chuỗi này.

<img width="1285" height="691" alt="image" src="https://github.com/user-attachments/assets/cf3eedf7-60c9-4036-9447-3985860066ec" /></p>

b) Lấy Chat ID nhóm

- Tạo một Group Telegram mới, thêm tài khoản của bạn (và ID 1875746636) cùng con Bot vừa tạo vào nhóm.

<img width="1920" height="1027" alt="image" src="https://github.com/user-attachments/assets/0c3c8930-af10-443f-af87-1e94c6492420" /></p>

<img width="1920" height="1023" alt="{AF2F9E4E-19AF-406C-B033-3F7A5E97701B}" src="https://github.com/user-attachments/assets/b9b52fe6-0586-457e-aeec-6101fbea641f" /></p>

- Tìm kiếm và mời thêm bot @RawDataBot vào nhóm này. Ngay khi vào nhóm, nó sẽ trả về một đoạn code chữ chứa thông tin nhóm. Bạn tìm dòng "id": -100xxxxxxxxxx. Đó chính là Chat ID của nhóm bạn (bắt buộc phải có dấu trừ - ở đầu nhé).

<img width="1917" height="233" alt="{C81ECEA5-8B00-47CB-9508-2844A3EF482B}" src="https://github.com/user-attachments/assets/2178c9c9-0b68-432a-8f3c-16a56ed931f2" /></p>

- Sau khi lấy xong ID, bạn có thể kích @RawDataBot ra khỏi nhóm cho đỡ vướng.

c) Kéo thêm các Node sau từ đầu ra của Node Function.

Bước 1: Kéo và cấu hình Node switch (Bộ lọc điều kiện giá)

Bước 2: Cấu hình node switch

<img width="1920" height="1033" alt="{8D872F53-056B-408A-B46E-F55BDF3DBF92}" src="https://github.com/user-attachments/assets/17c52cbb-a895-406b-8426-7c8b273cc960" /></p>

- Property: msg.currentPrice

- Nhánh 1 (Dấu <): Điền số mức thấp, ví dụ 60000

- Nhánh 2 (Bấm nút + add để thêm dòng, chọn dấu >): Điền số mức cao, ví dụ 75000

Bước 3: Node change số 1 (Nối vào chấm tròn trên của node switch)

<img width="1920" height="1019" alt="{F92DDCA4-F091-417A-B712-FAE160F048FA}" src="https://github.com/user-attachments/assets/876890ff-7a76-4930-9bc2-11b9dfca453d" /></p>

Bước 4: Node change số 2 (Nối vào chấm tròn dưới của node switch)

<img width="1920" height="1022" alt="{3C0C9869-7B8B-4B9C-BEE0-E2C41C7D24ED}" src="https://github.com/user-attachments/assets/3ab3ab09-57b8-4041-afdd-69caf634691e" /></p>

Bước 5: Node Telegram Sender

- Bot: Chọn cấu hình Bot (Tên bất kỳ + Dán chuỗi Token lấy từ @BotFather).

- Chat ID: Dán chính xác mã số nhóm chat

<img width="1920" height="1026" alt="{16A09556-CB5F-4876-B0C0-9BF09C11F7FD}" src="https://github.com/user-attachments/assets/e5355e0b-1771-4bd0-a20d-e6215d62b250" /></p>

---> Sơ đồ tổng thể 

<img width="1920" height="1028" alt="{3AC24D1F-A8F4-4039-A828-0BCC61BB546B}" src="https://github.com/user-attachments/assets/c41b54ba-e682-4ada-9de5-0260de5939b5" /></p>

### 5. Test

<img width="1920" height="1020" alt="{9E8BC13D-AF5A-4670-AAB4-F166F3A9E5D9}" src="https://github.com/user-attachments/assets/efab9d20-199e-4c99-843b-50364bc60ca3" /></p>

<img width="1920" height="1080" alt="{8171C199-50B6-4DBC-9C57-E6919F7A6811}" src="https://github.com/user-attachments/assets/25ebcfb5-05e8-4eb1-8f4e-c4ae882af0bd" /></p>






