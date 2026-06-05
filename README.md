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
| image    | Chỉ định Docker Image được dùng để tạo container (tải từ Docker Hub). | Dữ liệu B |
| Dòng 2    | Dữ liệu C | Dữ liệu D |
