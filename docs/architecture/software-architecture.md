# Kiến trúc phần mềm — Questify

> Trạng thái: Bản chốt cho Giai đoạn 1 (modular monolith) và định hướng tách microservice. Liên quan: [requirements.md](../product/requirements.md), [database-schema.md](./database-schema.md).

## 1. Tổng quan

Questify là ứng dụng theo dõi thói quen game hóa, xây dựng theo kiến trúc hướng domain, triển khai dần từ **modular monolith** (MVP) sang **microservices**. Backend là các service Java Spring Boot; frontend là React (web) và React Native (mobile); giao tiếp bất đồng bộ giữa các domain qua Kafka.

Lộ trình (theo README):
1. Giai đoạn 1 — Modular monolith: hai domain `user` và `quest` trong một repo Maven multi-module, DB Postgres hai schema.
2. Giai đoạn 2 — Tách microservice + API Gateway + Docker Compose.
3. Giai đoạn 3 — Event-driven đầy đủ qua Kafka.
4. Giai đoạn 4 — Kubernetes + observability + CI/CD.

## 2. Tech stack

| Tầng | Công nghệ | Phiên bản | Ghi chú |
| --- | --- | --- | --- |
| Ngôn ngữ backend | Java | 21 (LTS) | record, pattern matching, virtual threads |
| Framework | Spring Boot | 3.4.x | |
| Build | Maven (multi-module) | 3.9.x | parent POM + module theo service |
| API Gateway | Spring Cloud Gateway | Spring Cloud 2024.x | định tuyến + xác thực tập trung (từ Giai đoạn 2) |
| Xác thực | Keycloak (OAuth2 / OIDC) | 26.x | self-registration; phát JWT |
| Bảo mật service | Spring Security (resource server, JWT) | theo Spring Boot | verify JWT Keycloak |
| CSDL | PostgreSQL | 16+ | một DB, hai schema (MVP) |
| Migration | Liquibase (YAML) | theo Spring Boot | changelog theo từng service |
| ORM | Spring Data JPA / Hibernate | theo Spring Boot | |
| Mapping DTO | MapStruct | 1.6.x | bắt buộc dùng DTO (AGENTS.md §4.A.1) |
| Messaging | Apache Kafka | — | event inter-domain (`QuestCompletedEvent`...) |
| Cache / leaderboard | Redis | — | giai đoạn Social (sau MVP) |
| Frontend web | React + TypeScript | — | Zustand/Redux Toolkit + TanStack Query |
| Frontend mobile | React Native + TypeScript | — | |
| Container | Docker + Docker Compose | — | |
| Orchestration | Kubernetes (Minikube/k3s) | — | Giai đoạn 4 |
| Observability | Prometheus + Grafana; ELK/Loki | — | |
| CI/CD | GitHub Actions + Trivy | — | |

## 3. Cấu trúc code

### 3.1 Module (Maven multi-module)

```
questify-parent (pom)
├── common               (thư viện dùng chung: DTO sự kiện, tiện ích, cấu hình bảo mật chung)
├── user-service         (domain User & Gamification)
└── quest-service        (domain Quest Engine)
```

- MVP có thể chạy mỗi service như một Spring Boot app độc lập, hoặc gộp chạy chung; ranh giới code tách sẵn để lên microservice không phải refactor lớn.
- `common` chứa các record sự kiện Kafka và mã dùng chung, **không** chứa entity của domain khác.

### 3.2 Kiến trúc trong mỗi service — Hexagonal (Ports & Adapters)

Cả hai service dùng Hexagonal. Cấu trúc package:

```
<service>/src/main/java/com/questify/<domain>/
├── domain/              aggregate, value object, domain service, domain event
│                        → Java thuần, KHÔNG import Spring / JPA / Kafka
├── application/
│   ├── <X>UseCase.java        port in (ca sử dụng)
│   └── port/out/              port out (interface do nghiệp vụ định nghĩa)
│       ├── <X>Repository.java
│       ├── EventPublisher.java
│       └── TimeProvider.java
└── adapters/
    ├── in/web/                REST controller → gọi use case
    ├── out/persistence/       JPA entity + adapter + mapper (MapStruct)
    ├── out/messaging/         Kafka publisher / consumer
    └── out/time/              cung cấp today_local theo múi giờ người dùng
```

Quy tắc bắt buộc:
- **Chiều phụ thuộc hướng vào trong:** `adapters → application → domain`. `domain` và `application` không import framework.
- **Port thuộc về tầng nghiệp vụ:** interface `port/out` nằm trong `application`, không nằm cạnh code hạ tầng. Adapter `implements` các interface này.
- **Hai biểu diễn mỗi aggregate:** domain object (thuần) ở `domain/`, JPA entity ở `adapters/out/persistence/`. Mapping bằng MapStruct. Tuyệt đối **không** dùng JPA entity làm domain object.
- **Không trả JPA entity hay domain object ra controller:** dùng DTO (AGENTS.md §4.A.1).

Các quy tắc này được phản ánh trong AGENTS.md §4.A để mọi agent áp dụng thống nhất.

### 3.3 Mô hình domain — DDD (tactical)

Toàn bộ luật nghiệp vụ BR-01..BR-24 nằm trong tầng `domain/` của hai service, không nằm ở controller hay adapter.

**quest-service**

| Aggregate Root | Value Object bên trong | Luật canh giữ |
| --- | --- | --- |
| `QuestTemplate` | `Difficulty`, `RecurrenceRule`, `DateRange` | BR-18, BR-19 (bắt buộc ngày kết thúc, tối đa 1 năm, sinh instance) |
| `QuestInstance` | `Difficulty`, `InstanceStatus` | BR-21 (chỉ hoàn thành instance của hôm nay), BR-04 (gán XP/Gold) |
| `UserStreak` | — | BR-11, BR-12 (tăng, đứt, mốc thưởng) |
| `Category` | — | — |

Domain service: `InstanceGenerator` (BR-18/19), `StreakPolicy` (BR-10/11/12).
Domain event: `QuestCompleted`, `StreakMilestoneReached`.

**user-service**

| Aggregate Root | Value Object bên trong | Luật canh giữ |
| --- | --- | --- |
| `UserProfile` | — | JIT provisioning từ token Keycloak |
| `UserProgress` | `Level`, `Xp`, `Gold` | BR-01, BR-02, BR-03 (công thức, lên cấp, XP dư) |
| `Reward` | `Gold` | BR-06 (giá > 0) |
| `RewardRedemption` | `Gold` | BR-08, BR-09 (xác nhận, bất biến) |

Domain service: `LevelingPolicy` (BR-03), `RedemptionPolicy` (BR-07/08).

Quy ước: Value Object không có bảng riêng — map thành cột trên bảng của Entity sở hữu (`@Embeddable` hoặc converter). Hai aggregate khác nhau tham chiếu nhau bằng **id**, không ôm object (ví dụ `QuestInstance` giữ `templateId`).

## 4. Ranh giới service và sở hữu dữ liệu

| Service | Sở hữu (schema) | Trách nhiệm |
| --- | --- | --- |
| user-service | `user` | Hồ sơ người dùng, XP/Level, Gold, phần thưởng, lịch sử đổi thưởng |
| quest-service | `quest` | Quest template/instance, danh mục, streak, sinh instance, auto-fail |

Nguyên tắc:
- Mỗi service chỉ truy cập schema của mình. Tham chiếu domain khác bằng UUID, không FK chéo (xem [database-schema.md](./database-schema.md)).
- Không gọi DB của service khác trực tiếp; trao đổi qua REST (đồng bộ) hoặc Kafka (bất đồng bộ).

**Sinh instance (quest-service):** khi tạo template, sinh toàn bộ instance trong khoảng (tối đa 1 năm — BR-19) **đồng bộ, trong cùng transaction** tạo template. Nguyên tử (lỗi thì rollback cả template lẫn instance) và phản hồi ngay cho người dùng. Dùng batch insert (tối đa ~365 row/template hằng ngày) thay vì insert từng dòng.

## 5. Giao tiếp

### 5.1 Đồng bộ — REST qua API Gateway
- Client → Spring Cloud Gateway → service tương ứng.
- Gateway xác thực JWT tập trung, định tuyến theo path (`/api/users/**` → user-service, `/api/quests/**` → quest-service).
- Dùng cho thao tác do người dùng kích hoạt cần phản hồi ngay (tạo Quest, hoàn thành, đổi thưởng, xem hồ sơ).

### 5.2 Bất đồng bộ — Kafka events
Dùng cho giao tiếp giữa domain độc lập, tránh phụ thuộc trực tiếp (AGENTS.md §4.A.2).

| Sự kiện | Phát bởi | Tiêu thụ bởi | Tác động |
| --- | --- | --- | --- |
| `QuestCompletedEvent(user_id, difficulty, xp, gold)` | quest-service | user-service | Cộng XP/Gold, xử lý lên cấp (BR-03, BR-04) |
| `StreakMilestoneReachedEvent(user_id, bonus_xp, bonus_gold)` | quest-service | user-service | Cộng thưởng mốc (BR-12) |

- Hoàn thành Quest (quest-service) và trao thưởng (user-service) tách qua event → hai domain không phụ thuộc đồng bộ.
- (Sau MVP) Notification Service tiêu thụ các event để gửi thông báo.

## 6. Xác thực và phân quyền

- **Keycloak** là Identity Provider: giữ credential, phát JWT (OIDC), hỗ trợ **self-registration**, quên mật khẩu, xác minh email, (tùy chọn) social login.
- Backend là **OAuth2 Resource Server**: verify chữ ký JWT bằng public key Keycloak; không tự quản password.
- **JIT provisioning:** lần đầu một user chạm backend, nếu chưa có `app_user` khớp `keycloak_id` (= claim `sub`) thì tạo mới từ claim token (email, tên), đồng thời khởi tạo `user_progress` (level 1) và `user_streak` (0).
- Liên kết danh tính: Keycloak `sub` ↔ `app_user.keycloak_id`. Dữ liệu nghiệp vụ gắn theo `app_user.id` local.

## 7. Job nền

- **Auto-fail instance (BR-22):** quét `quest_instance` PENDING có `scheduled_date < today_local(user_timezone)`, đặt `FAILED`. Chạy định kỳ (ví dụ mỗi giờ) để phủ các múi giờ. Thuộc quest-service.

## 8. Observability & vận hành

- **Metrics:** Spring Boot Actuator + Micrometer → Prometheus; dashboard Grafana.
- **Log tập trung:** ELK Stack hoặc Loki.
- **Container:** Docker cho mọi service + frontend; Docker Compose cho môi trường dev (Postgres, Kafka, Keycloak, Redis).
- **Orchestration:** Kubernetes (Minikube/k3s) + HPA (Giai đoạn 4).
- **CI/CD:** GitHub Actions — test, build image, quét lỗ hổng Trivy, deploy.
- **Secret:** không commit; qua `.env` / Config Server / K8s Secret (AGENTS.md §5.3).

## 9. Các quyết định kiến trúc (ADR tóm tắt)

| # | Quyết định | Lý do |
| --- | --- | --- |
| ADR-01 | Modular monolith trước, microservice sau | Giảm phức tạp MVP; ranh giới domain tách sẵn để lên microservice |
| ADR-02 | Maven multi-module | Khớp thư mục `services/` sẵn có; tách service không refactor lớn |
| ADR-03 | Java 21 + Spring Boot 3.4 | LTS mới nhất; đáp ứng AGENTS.md Java 17+ |
| ADR-04 | Một DB, hai schema; không FK chéo | Ranh giới dữ liệu rõ; tách DB sau dễ |
| ADR-05 | Kafka cho event inter-domain | Giảm phụ thuộc trực tiếp; đúng AGENTS.md §4.A.2 |
| ADR-06 | Keycloak (OAuth2/OIDC) + JIT provisioning | Chuẩn xác thực thật; self-registration sẵn; password không nằm ở DB app |
| ADR-07 | DTO bắt buộc + MapStruct | Không lộ JPA entity ở controller (AGENTS.md §4.A.1) |
| ADR-08 | Hexagonal (Ports & Adapters) cho cả hai service | Cô lập luật nghiệp vụ khỏi JPA/Kafka/Spring; test domain không cần DB; đổi hạ tầng chỉ thay adapter. Đánh đổi: thêm port/adapter/mapper |
| ADR-09 | DDD tactical (aggregate, value object, domain service, domain event) | Luật BR-01..24 nhiều và dễ bị lách nếu rải ở service; gói vào domain object để luật không vi phạm được. Bounded context đã có sẵn (`user`, `quest`) |

## 10. Còn mở

- Phiên bản cụ thể của Kafka, Redis, PostgreSQL image sẽ chốt khi dựng Docker Compose.
- Cấu trúc realm/client Keycloak (tên realm, client id, roles) chốt khi cấu hình auth.
- Theme trang đăng ký/đăng nhập: dùng trang Keycloak (theme lại) hay form riêng trong app — chốt ở giai đoạn frontend.
