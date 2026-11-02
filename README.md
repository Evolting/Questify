# 🎮 Questify — Tổng quan dự án

> **Ứng dụng theo dõi thói quen và kỹ năng được game hóa, xây dựng bằng React Native / React và các microservice Java Spring Boot.**

---

## 📌 Tóm tắt dự án

**Questify** là ứng dụng theo dõi thói quen và hiệu suất cá nhân, sử dụng **cơ chế game hóa theo phong cách RPG (Role-Playing Game)**. Thay vì danh sách việc cần làm thông thường, Questify biến quá trình phát triển bản thân, chăm sóc sức khỏe, học tập và làm việc thành một trò chơi tương tác, nơi người dùng là nhân vật chính.

Questify có hai mục tiêu chính:
1. **Giá trị sản phẩm:** Tạo một công cụ quản lý hiệu suất hấp dẫn, có cơ chế tự thưởng và hữu ích cho việc sử dụng hằng ngày của developer.
2. **Rèn luyện kỹ thuật:** Cung cấp một bài toán thực tế để triển khai **kiến trúc microservice, hệ thống hướng sự kiện và quy trình DevOps hiện đại** bằng **Java Spring Boot**, **React Native / React** và **Kubernetes**.

---

## 🎯 Tính năng chính

### 1. ⚔️ Hệ thống Quest (thói quen và công việc)
* **Quest:** Thói quen hoặc công việc với các mức độ khó (Dễ, Trung bình, Khó).
* **Danh mục:** Gắn Quest với từng lĩnh vực (ví dụ: lập trình, thể chất, tài chính).
* **Lặp lại:** Lên lịch hằng ngày, hằng tuần hoặc theo lịch tùy chỉnh.

### 2. 🌳 Cây kỹ năng và lên cấp (hệ thống XP)
* **Điểm kinh nghiệm (XP):** Nhận được khi hoàn thành Quest.
* **Tiến trình lên cấp:** Áp dụng công thức $MaxXP = 100 \times Level^{1.5}$.
* **Cây kỹ năng:** Biểu đồ các nút tiến trình thể hiện sự phát triển trong từng lĩnh vực (ví dụ: trình độ Java, quãng đường chạy).

### 3. 🛍️ Cửa hàng phần thưởng tùy chỉnh (Gold)
* **Kiếm Gold:** Nhận Gold cùng với XP khi hoàn thành Quest.
* **Tự đặt phần thưởng:** Người dùng xác định phần thưởng ngoài đời và mức Gold cần để đổi (ví dụ: 500 Gold = *1 buổi xem phim*).
* **Đổi thưởng:** Dùng tiền tệ ảo để đổi lấy thời gian nghỉ ngơi hoặc phần thưởng ngoài đời thông qua một giao dịch.

### 4. 🛡️ Guild và bảng xếp hạng (tính năng xã hội)
* **Guild:** Tham gia nhóm cùng bạn bè để hỗ trợ nhau duy trì cam kết.
* **Đánh Boss hằng tuần:** Mục tiêu chung của Guild; việc bỏ lỡ thói quen cá nhân gây bất lợi cho cả nhóm.
* **Bảng xếp hạng:** Xếp hạng theo XP theo thời gian thực giữa bạn bè và thành viên Guild.

---

## 🏗️ Kiến trúc và các microservice

```
                         +-------------------+
                         | React / React Native |
                         +---------+---------+
                                   |
                         +---------v---------+
                         |  Spring Cloud     |
                         |  API Gateway      |
                         +----+----+----+----+
                              |    |    |
        +---------------------+    |    +----------------------+
        |                          |                           |
+-------v-------+          +-------v-------+           +-------v-------+
|  User & XP    |          |  Quest & Event|           | Social & Guild|
|  Service      |          |  Engine       |           | Service       |
|  (PostgreSQL) |          |  (PostgreSQL) |           | (Redis/Postgres)|
+-------+-------+          +-------+-------+           +-------+-------+
        |                          |                           |
        +--------------------+     |     +---------------------+
                             |     |     |
                          +--v-----v-----v--+
                          |   Apache Kafka  |
                          +--------+--------+
                                   |
                          +--------v--------+
                          | Notification    |
                          | Service         |
                          +-----------------+
```

### Các microservice

1. **User & Gamification Service**
   * Quản lý hồ sơ người dùng, tính XP, xử lý tiến trình lên cấp và số dư Gold.
   * *Cơ sở dữ liệu:* PostgreSQL

2. **Quest Engine Service**
   * Xử lý việc tạo Quest, quy trình hoàn thành và lịch lặp lại.
   * *Cơ sở dữ liệu:* PostgreSQL

3. **Social & Guild Service**
   * Quản lý quan hệ bạn bè, trò chuyện trong Guild và bảng xếp hạng theo thời gian thực.
   * *Cơ sở dữ liệu:* Redis (lưu đệm bảng xếp hạng) và PostgreSQL

4. **Notification Service**
   * Nhận các sự kiện theo cơ chế bất đồng bộ (ví dụ: hoàn thành Quest, lên cấp) và gửi thông báo đẩy hoặc email.

---

## 🛠️ Mục tiêu DevOps và hạ tầng

* **Đóng gói bằng container:** Dùng Docker cho tất cả service Spring Boot và ứng dụng frontend.
* **Kiến trúc hướng sự kiện:** Dùng **Apache Kafka** hoặc **RabbitMQ** để giảm sự phụ thuộc trực tiếp trong giao tiếp giữa các service (ví dụ: `QuestCompletedEvent`).
* **API Gateway và bảo mật:** Dùng **Spring Cloud Gateway** với mô hình bảo mật **OAuth2 / JWT**.
* **Điều phối:** Triển khai bằng **Kubernetes** (Minikube/k3s) với Horizontal Pod Autoscaling (HPA).
* **Quy trình CI/CD:** Tự động hóa kiểm thử, xây dựng Docker image, quét lỗ hổng bằng Trivy và triển khai qua **GitHub Actions**.
* **Giám sát hệ thống:** Tập trung log bằng **ELK Stack / Loki** và theo dõi chỉ số bằng **Prometheus & Grafana**.

---

## 🚀 Lộ trình triển khai đề xuất

1. **Giai đoạn 1: MVP dạng modular monolith**
   * Xây dựng các domain cốt lõi (Quest, XP, Gold, phần thưởng) trong một ứng dụng Spring Boot.
   * Tạo giao diện React hoặc React Native cơ bản để tương tác với API.

2. **Giai đoạn 2: Tách microservice và đóng gói bằng Docker**
   * Tách ứng dụng thành `User/XP Service` và `Quest Service`.
   * Thiết lập Spring Cloud Gateway và Docker Compose.

3. **Giai đoạn 3: Xử lý hướng sự kiện và bất đồng bộ**
   * Tích hợp Kafka cho các sự kiện bất đồng bộ (`QuestCompletedEvent` $\rightarrow$ `User XP Update` và `Notification`).

4. **Giai đoạn 4: Kubernetes và hạ tầng DevOps**
   * Viết Kubernetes manifest / Helm chart.
   * Thiết lập Prometheus, Grafana và các quy trình CI/CD tự động.
