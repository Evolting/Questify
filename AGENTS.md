# 🤖 AGENTS.md — Guidelines & Protocols for AI Coding Agents

File này chứa toàn bộ các chỉ dẫn, quy tắc và nguyên tắc bắt buộc dành cho **AI Agents** (như Cursor, GitHub Copilot, Claude, ChatGPT, v.v.) khi đọc, sửa đổi hoặc mở rộng codebase của dự án **Questify**.

---

## 📌 1. Mục tiêu Dự án (Project Goals)

* **Về mặt Sản phẩm:** Xây dựng ứng dụng theo dõi thói quen & hiệu suất cá nhân (Habit & Skill Tracker) được game hóa theo phong cách RPG (Level, XP, Gold, Skill Trees, Guilds).
* **Về mặt Kỹ thuật:** Đóng vai trò là hệ thống mẫu chuẩn mực về **Microservices (Java Spring Boot)** và **DevOps Pipelines** (Docker, Kubernetes, Kafka, CI/CD, Observability) đi kèm với giao diện người dùng **React / React Native**.

---

## 🎯 2. Đối tượng Người dùng (Target Audience)

1. **Chính bản thân Developer (End-user chính):** Dùng để tự quản lý việc học tập, luyện tập và làm việc hàng ngày.
2. **Những người yêu thích Gamification & RPG:** Cần động lực tức thì (Instant Gratification) để duy trì thói quen tốt.
3. **Các Nhà tuyển dụng / Reviewer (Technical Audience):** Đánh giá tư duy kiến trúc Microservices, Clean Code và tư duy DevOps của tác giả.

---

## 🎨 3. Phong cách Giao diện (UI/UX Style Guide)

* **Chủ đề Gamified / Modern RPG:** Giao diện đậm chất hiện đại, hơi hướng Pixel-Art tinh tế hoặc Dark-Mode Cyber/Fantasy (tương tự các ứng dụng Gameify cao cấp).
* **Tone màu chủ đạo:**
  * Background: Dark / Semi-dark (`#121212`, `#1E1E2E`).
  * Accent (Kinh nghiệm/XP): Cyan / Purple (`#8A2BE2`, `#00FFFF`).
  * Gold/Thưởng: Amber / Gold (`#FFD700`).
  * Trạng thái thành công/Thất bại: Emerald Green / Crimson Red.
* **UX Principles:** 
  * Tương tác phản hồi tức thì (Instant feedback): Có hiệu ứng animation nhỏ (confetti, thanh XP nảy lên) khi complete Quest.
  * Tránh rườm rà: Thao tác đánh dấu hoàn thành Quest không quá 1 tap/click.

---

## 📜 4. Quy tắc Code (Coding Standards & Guidelines)

### A. Backend (Java Spring Boot)
1. **Kiến trúc Clean Architecture / Layered Architecture:**
   * Chia rõ: `controller` $\rightarrow$ `service` $\rightarrow$ `repository` $\rightarrow$ `entity/domain` $\rightarrow$ `dto`.
   * **Bắt buộc dùng DTO:** Tuyệt đối không trả về JPA Entity trực tiếp tại Controller Layer. Dùng MapStruct hoặc Record DTOs.
2. **Xử lý Bất đồng bộ & Microservices:**
   * Dùng **Kafka Events** cho giao tiếp inter-service giữa các domain độc lập (VD: `QuestCompletedEvent`).
   * Sử dụng `@Transactional` chính xác tại Service layer. Xử lý Exception tập trung tại `@RestControllerAdvice`.
3. **Code Style:**
   * Java 17+ (ưu tiên dùng `record`, `var` đúng chỗ, Pattern Matching).
   * Tuân thủ Naming convention: CamelCase cho biến/hàm, PascalCase cho Class, SNAKE_CASE cho Constants.

### B. Frontend (React / React Native)
1. **Component Structure:**
   * Viết bằng **TypeScript 100%**. Nói KHÔNG với `any`.
   * Sử dụng Functional Components + Hooks. Không dùng Class Components.
2. **State Management:**
   * Dùng Zustand hoặc Redux Toolkit cho Global State.
   * Dùng TanStack Query (React Query) cho Server State & Caching.

---

## 🚫 5. Những điều KHÔNG ĐƯỢC TỰ Ý THAY ĐỔI (Strict Boundaries)

1. **Kiến trúc Domain Cốt lõi:**
   * Không tự ý gộp lại các Microservices thành Monolith trừ khi có chỉ thị rõ ràng.
   * Cấu trúc cơ sở dữ liệu (Database Schema) đã chốt phải qua xác nhận trước khi viết Migration Script (Liquibase/Flyway).
2. **Luật Đếm/Tính Điểm (Core Business Formulas):**
   * Công thức tính XP Lên cấp: $MaxXP = 100 \times Level^{1.5}$. **Không sửa công thức này** nếu không có yêu cầu.
3. **Bảo mật & Biến môi trường:**
   * Không commit trực tiếp Secret Keys, Passwords, API Keys, JWT Secrets vào Git. Tất cả phải qua `.env` hoặc Config Server.

---

## 📋 6. Quy trình Báo cáo sau mỗi lần sửa Code (Reporting Protocol)

Sau mỗi lần thực hiện Task, refactor hoặc sửa lỗi, AI Agent **cần trình bày kết quả công việc** ngắn gọn theo mẫu cấu trúc sau:

```markdown
### 📝 AI Change Report

#### 1. Tóm tắt thay đổi (Summary of Changes)
* [File/Component được thay đổi]: Mô tả ngắn gọn việc đã làm.

#### 2. Quyết định kỹ thuật (Technical Decisions)
* Lý do chọn giải pháp này (nếu có).

#### 3. Tác động tới Hệ thống (Impact & Dependencies)
* [ ] Sửa đổi Database Schema? (Có/Không)
* [ ] Có làm thay đổi API Contract (Request/Response) không?
* [ ] Có ảnh hưởng đến Service khác không?

#### 4. Hướng dẫn Kiểm thử (Verification / How to Test)
* Các bước để Developer test lại thay đổi vừa rồi.
```