# Thiết kế Database — Questify MVP

> Trạng thái: Bản thiết kế để chốt trước khi viết migration (Liquibase). Nguồn luật nghiệp vụ: [requirements.md](../product/requirements.md) (BR-01..BR-24).

## 1. Quyết định nền tảng

| Hạng mục | Lựa chọn |
| --- | --- |
| CSDL | PostgreSQL |
| Bố trí MVP | Một database, hai schema: `user` và `quest`. Không FK chéo schema; tham chiếu domain khác chỉ bằng UUID. Dễ tách thành hai database khi lên microservice. Gamification (streak, XP, Gold) để `user_streak` ở schema `quest` vì phụ thuộc hoàn thành Quest. |
| Khóa chính | UUID (`gen_random_uuid()` / `uuid_generate_v4()`). |
| Enum | Lưu bằng `varchar` kèm ràng buộc `CHECK`. Không dùng native PG enum (để portable và dễ thêm giá trị). |
| Thời gian | `timestamptz` cho mốc thời gian thật; `date` cho ngày lịch. |
| Migration | Liquibase, changelog định dạng **YAML**. Đặt **theo từng service**: changelog schema `user` ở `services/user-service`, schema `quest` ở `services/quest-service` (mỗi service sở hữu schema của mình, tách sạch khi lên microservice). Viết sau khi schema này được chốt (AGENTS.md §5.1). |

## 2. Schema `user` — User & Gamification Service

### `user.app_user`
Hồ sơ người dùng. Tên `app_user` vì `user` là từ khóa Postgres.

| Cột | Kiểu | Ràng buộc | Ghi chú |
| --- | --- | --- | --- |
| id | uuid | PK | |
| keycloak_id | uuid | NOT NULL, UNIQUE | `sub` của token Keycloak; dùng cho JIT provisioning. Không lưu password (Keycloak giữ). |
| email | varchar(255) | NOT NULL, UNIQUE | Lấy từ claim token |
| display_name | varchar(100) | NOT NULL | Lấy từ claim token lúc tạo |
| timezone | varchar(64) | NOT NULL | IANA, ví dụ `Asia/Ho_Chi_Minh` — BR-14 |
| created_at | timestamptz | NOT NULL, default now() | |
| updated_at | timestamptz | NOT NULL, default now() | |

### `user.user_progress`
XP, Level, số dư Gold của người dùng.

| Cột | Kiểu | Ràng buộc | Ghi chú |
| --- | --- | --- | --- |
| user_id | uuid | PK, FK → app_user(id) | |
| level | int | NOT NULL, default 1, CHECK (level >= 1) | BR-02 |
| current_xp | bigint | NOT NULL, default 0, CHECK (current_xp >= 0) | XP trong level hiện tại — BR-03; MVP không undo nên không âm |
| gold_balance | bigint | NOT NULL, default 0, CHECK (gold_balance >= 0) | BR-06/07; MVP không undo nên không âm |
| updated_at | timestamptz | NOT NULL, default now() | |

### `user.reward`
Phần thưởng do người dùng tự định nghĩa.

| Cột | Kiểu | Ràng buộc | Ghi chú |
| --- | --- | --- | --- |
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → app_user(id) | |
| name | varchar(150) | NOT NULL | |
| cost_gold | bigint | NOT NULL, CHECK (cost_gold > 0) | BR-06 |
| is_active | boolean | NOT NULL, default true | Ẩn/xóa mềm |
| created_at | timestamptz | NOT NULL, default now() | |
| updated_at | timestamptz | NOT NULL, default now() | |

Index: `(user_id, is_active)`.

### `user.reward_redemption`
Lịch sử đổi thưởng — bất biến (BR-08, BR-09). Không UPDATE, không DELETE (chặn ở tầng ứng dụng).

| Cột | Kiểu | Ràng buộc | Ghi chú |
| --- | --- | --- | --- |
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → app_user(id) | |
| reward_id | uuid | NOT NULL, FK → reward(id) | |
| reward_name_snapshot | varchar(150) | NOT NULL | Chụp tên lúc đổi (reward có thể đổi tên sau) |
| cost_gold_snapshot | bigint | NOT NULL | Chụp giá lúc đổi |
| redeemed_at | timestamptz | NOT NULL, default now() | |

Index: `(user_id, redeemed_at desc)`.

## 3. Schema `quest` — Quest Engine Service

Các cột `user_id` ở schema này là **tham chiếu** tới `user.app_user(id)`, không đặt FK chéo schema (để tách DB sau).

### `quest.category`
Danh mục do người dùng tự tạo (BR-05 — chỉ phân loại, không ảnh hưởng thưởng).

| Cột | Kiểu | Ràng buộc | Ghi chú |
| --- | --- | --- | --- |
| id | uuid | PK | |
| user_id | uuid | NOT NULL | Tham chiếu app_user |
| name | varchar(100) | NOT NULL | |
| created_at | timestamptz | NOT NULL, default now() | |

Ràng buộc: UNIQUE `(user_id, name)`.

### `quest.quest_template`
Khuôn Quest + quy tắc lặp (BR-18).

| Cột | Kiểu | Ràng buộc | Ghi chú |
| --- | --- | --- | --- |
| id | uuid | PK | |
| user_id | uuid | NOT NULL | Tham chiếu app_user |
| title | varchar(200) | NOT NULL | |
| difficulty | varchar(10) | NOT NULL, CHECK IN ('EASY','MEDIUM','HARD') | BR-04 |
| category_id | uuid | NULL, FK → category(id) | BR-05 |
| recurrence_type | varchar(10) | NOT NULL, CHECK IN ('ONCE','DAILY','WEEKLY','CUSTOM') | BR-18 |
| recurrence_config | jsonb | NULL | WEEKLY: thứ trong tuần; CUSTOM: mỗi N ngày... |
| start_date | date | NOT NULL | |
| end_date | date | NULL | Bắt buộc khi recurrence_type ≠ ONCE — BR-19 |
| is_active | boolean | NOT NULL, default true | Sửa/xóa mềm — BR-23 |
| created_at | timestamptz | NOT NULL, default now() | |
| updated_at | timestamptz | NOT NULL, default now() | |

Ràng buộc nghiệp vụ (kiểm ở tầng ứng dụng): recurrence_type ≠ ONCE thì end_date NOT NULL và (end_date − start_date) ≤ 1 năm (BR-19).

Index: `(user_id, is_active)`.

### `quest.quest_instance`
Lần xuất hiện theo ngày — đơn vị người dùng hoàn thành (BR-18).

| Cột                  | Kiểu        | Ràng buộc                                                              | Ghi chú                                          |
| -------------------- | ----------- | ---------------------------------------------------------------------- | ------------------------------------------------ |
| id                   | uuid        | PK                                                                     |                                                  |
| template\_id         | uuid        | NOT NULL, FK → quest\_template(id)                                     |                                                  |
| user\_id             | uuid        | NOT NULL                                                               | Denormalize để query theo user + ngày            |
| scheduled\_date      | date        | NOT NULL                                                               | Ngày instance thuộc về — BR-13                   |
| status               | varchar(10) | NOT NULL, default 'PENDING', CHECK IN ('PENDING','COMPLETED','FAILED') | FAILED = tự đóng khi quá hạn — BR-22             |
| difficulty\_snapshot | varchar(10) | NOT NULL, CHECK IN ('EASY','MEDIUM','HARD')                            | Chụp lúc sinh; instance đã xong bất biến — BR-23 |
| xp\_awarded          | int         | NULL                                                                   | Ghi lúc hoàn thành — BR-04                       |
| gold\_awarded        | int         | NULL                                                                   | Ghi lúc hoàn thành — BR-04                       |
| completed\_at        | timestamptz | NULL                                                                   | Chỉ lưu vết — BR-13                              |
| created\_at          | timestamptz | NOT NULL, default now()                                                |                                                  |
| updated\_at          | timestamptz | NOT NULL, default now()                                                |                                                  |

Ràng buộc: UNIQUE `(template_id, scheduled_date)` — một instance mỗi ngày mỗi template (BR-24).

Index:
- `(user_id, scheduled_date)` — "Quest active của ngày D" (BR-10).
- `(status, scheduled_date)` — quét instance quá hạn để auto-fail (BR-22).

### `quest.user_streak`
Trạng thái chuỗi hiện tại (BR-11, BR-12).

| Cột | Kiểu | Ràng buộc | Ghi chú |
| --- | --- | --- | --- |
| user_id | uuid | PK | Tham chiếu app_user |
| current_streak | int | NOT NULL, default 0, CHECK (current_streak >= 0) | BR-11 |
| last_valid_date | date | NULL | Ngày hợp lệ gần nhất, để phát hiện liên tiếp/đứt |
| updated_at | timestamptz | NOT NULL, default now() | |

## 4. Luồng sự kiện (Kafka)

| Sự kiện | Phát bởi | Tiêu thụ bởi | Tác động |
| --- | --- | --- | --- |
| `QuestCompletedEvent(user_id, difficulty, xp, gold)` | quest-service (khi instance → COMPLETED) | user-service | Cộng XP/Gold `user_progress`, xử lý lên cấp (BR-03, BR-04) |
| `StreakMilestoneReachedEvent(user_id, bonus_xp, bonus_gold)` | quest-service (khi chuỗi chạm mốc) | user-service | Cộng thưởng mốc (BR-12) |

Streak tính ở quest-service vì phụ thuộc hoàn thành Quest; lắng nghe hoàn thành và job auto-fail (BR-22).

## 5. Job nền

- **Auto-fail (BR-22):** job quét định kỳ các `quest_instance` có `status = 'PENDING'` và `scheduled_date < today_local(user_timezone)`, đặt `status = 'FAILED'`. Chạy theo mốc chuyển ngày của từng múi giờ (ví dụ quét mỗi giờ). Việc đóng instance làm ngày đó không hợp lệ (BR-10) nên có thể làm đứt chuỗi.

## 6. Ánh xạ luật nghiệp vụ → schema

| BR | Thể hiện ở |
| --- | --- |
| BR-01/02/03 | `user_progress.level`, `current_xp` (logic ở service) |
| BR-04/05 | `quest_instance.xp_awarded`, `gold_awarded`, `difficulty_snapshot`; `category` |
| BR-06/07/08/09 | `reward`, `reward_redemption` |
| BR-10/11/12 | `quest_instance.scheduled_date` + `status`; `user_streak` |
| BR-13/14/15 | `quest_instance.scheduled_date`, `completed_at`; `app_user.timezone` |
| BR-16/17 | Không có cột undo; `reward_redemption` và instance COMPLETED bất biến |
| BR-18..24 | `quest_template`, `quest_instance`, `category` |

## 7. Quyết định và giả định

1. **Đã chốt:** mọi `quest_instance` bắt buộc thuộc một `quest_template` (`template_id` NOT NULL). Quest một lần (ONCE) vẫn tạo một template (recurrence_type = ONCE) và đúng một instance. Template giữ phần định nghĩa; instance giữ trạng thái theo ngày.
2. **Đã chốt:** `recurrence_config` là một cột jsonb linh hoạt cho MVP; hình dạng JSON cụ thể của WEEKLY/CUSTOM sẽ định nghĩa khi hiện thực lịch tùy chỉnh.
3. **Đã chốt:** sinh toàn bộ instance ngay lúc tạo template, tối đa 1 năm (BR-19).
4. **Đã chốt:** `user_streak` và `category` đặt ở schema `quest`.
5. **Đã chốt:** không đồng bộ hóa cột audit giữa các bảng. Mỗi bảng khai báo `created_at` / `updated_at` theo đúng nhu cầu của nó (ví dụ `user_progress` và `user_streak` chỉ có `updated_at`; `reward_redemption` chỉ có `redeemed_at` vì bất biến — BR-09). Phía JPA có thể dùng `@MappedSuperclass` + JPA Auditing cho nhóm bảng có đủ cả hai cột, nhưng đó là chi tiết của `adapters/out/persistence`, không áp đặt lên schema hay domain.
