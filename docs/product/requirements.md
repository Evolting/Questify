# Yêu cầu sản phẩm Questify — MVP

> Trạng thái: Bản nháp để chốt luật nghiệp vụ. Nguồn: `README.md`, `AGENTS.md` và phạm vi MVP đã thống nhất.

## 1. Mục tiêu

Questify giúp người dùng quản lý thói quen và công việc cá nhân dưới dạng Quest. Việc hoàn thành Quest tạo phản hồi tức thì thông qua XP và Gold; XP thể hiện tiến trình lên cấp, còn Gold dùng để đổi phần thưởng do người dùng tự đặt.

## 2. Phạm vi MVP

MVP gồm bốn nhóm chức năng:

1. Tạo và hoàn thành Quest.
2. Nhận XP và theo dõi cấp độ.
3. Nhận và theo dõi Gold.
4. Tạo và đổi phần thưởng cá nhân bằng Gold.

Guild, bảng xếp hạng, đánh Boss và các tính năng xã hội thuộc giai đoạn sau.

## 3. Thuật ngữ

| Thuật ngữ | Ý nghĩa |
| --- | --- |
| Quest | Thói quen hoặc công việc mà người dùng muốn hoàn thành. |
| Lần hoàn thành | Một lần người dùng đánh dấu Quest đã hoàn tất. |
| XP | Điểm kinh nghiệm nhận được khi hoàn thành Quest. |
| Level | Cấp độ thể hiện tiến trình tích lũy XP. |
| Gold | Đơn vị thưởng nhận cùng XP và dùng để đổi phần thưởng. |
| Phần thưởng | Hoạt động hoặc lợi ích ngoài đời do người dùng tự định nghĩa, có giá bằng Gold. |

## 4. Yêu cầu chức năng

### Quest

| ID | Yêu cầu |
| --- | --- |
| Q-01 | Người dùng có thể tạo Quest với tên, độ khó Dễ / Trung bình / Khó và danh mục như học tập, thể chất hoặc tài chính. |
| Q-02 | Người dùng có thể tạo Quest một lần hoặc Quest lặp lại (hằng ngày, hằng tuần, hoặc theo lịch tùy chỉnh) với ngày kết thúc bắt buộc. Quest lặp lại sinh nhiều instance theo lịch (BR-18, BR-19). |
| Q-03 | Người dùng có thể đánh dấu hoàn thành Quest bằng một lần chạm hoặc nhấp, sau bước xác nhận (BR-16). Giao diện phản hồi ngay sau thao tác. |

### XP và Level

| ID | Yêu cầu |
| --- | --- |
| XP-01 | Hoàn thành Quest giúp người dùng nhận XP theo độ khó của Quest. |
| XP-02 | Ứng dụng hiển thị XP và tiến trình lên cấp của người dùng. |
| XP-03 | Ngưỡng XP của mỗi Level tuân theo công thức ở mục 5. |
| XP-04 | Giữ chuỗi ngày hoàn thành liên tục sẽ nhận thưởng XP và Gold bổ sung tại các mốc quy định ở mục 5. |

### Gold và phần thưởng

| ID | Yêu cầu |
| --- | --- |
| G-01 | Hoàn thành Quest giúp người dùng nhận Gold cùng với XP. |
| G-02 | Ứng dụng hiển thị số dư Gold của người dùng. |
| R-01 | Người dùng có thể tự tạo phần thưởng ngoài đời và đặt giá bằng Gold. |
| R-02 | Người dùng có thể dùng Gold để đổi phần thưởng đã tạo khi đủ Gold. |
| R-03 | Ứng dụng hiển thị lịch sử các lần đổi phần thưởng. |

## 5. Luật nghiệp vụ đã xác định

| ID | Luật |
| --- | --- |
| BR-01 | Ngưỡng XP để lên cấp: $MaxXP = 100 \times Level^{1.5}$. Không thay đổi công thức này khi chưa có yêu cầu rõ ràng. |
| BR-02 | Người dùng bắt đầu ở Level 1. Ngưỡng XP mỗi level được làm tròn xuống (`floor`); áp dụng cùng một cách làm tròn ở cả client và server. |
| BR-03 | XP tích lũy trong level hiện tại; khi đạt hoặc vượt ngưỡng của level thì lên cấp và **giữ phần XP dư** cho level kế tiếp. Một lần nhận XP có thể làm lên nhiều cấp liên tiếp nếu đủ. Ví dụ: Level 1 cần 100 XP, người dùng đạt 106 XP → lên Level 2 với 6 XP dư. |
| BR-04 | Một lần hoàn thành Quest hợp lệ tạo phần thưởng XP và Gold theo độ khó: Dễ = 10 XP / 2 Gold; Trung bình = 30 XP / 5 Gold; Khó = 100 XP / 15 Gold. |
| BR-05 | Mức thưởng XP và Gold chỉ phụ thuộc độ khó. Danh mục không làm thay đổi mức thưởng; danh mục chỉ dùng để phân loại và lọc Quest. |
| BR-06 | Giá của mỗi phần thưởng do người dùng đặt bằng Gold. |
| BR-07 | Chỉ cho phép đổi phần thưởng khi số dư Gold ≥ giá phần thưởng. Thiếu Gold thì chặn thao tác đổi. |
| BR-08 | Giao diện phải yêu cầu người dùng xác nhận trước khi đổi. Đổi thành công sẽ trừ Gold và ghi một bản ghi lịch sử đổi thưởng. |
| BR-09 | Lịch sử đổi thưởng là bất biến: không cho phép hoàn tác giao dịch đổi và không cho phép xóa bản ghi lịch sử. |

### Luật chuỗi ngày (Streak)

| ID | Luật |
| --- | --- |
| BR-10 | **Quest active của ngày D** = các instance có `scheduled_date == D` và chưa bị đóng. Một ngày D là **ngày hợp lệ** nếu D có ít nhất một Quest active **và** tất cả Quest active của D đã hoàn thành. Ngày không có Quest active nào là ngày **không hợp lệ**. |
| BR-11 | Bộ đếm chuỗi (`current_streak`) tăng 1 cho mỗi ngày hợp lệ liên tiếp. Bất kỳ ngày không hợp lệ nào (không có Quest active, hoặc có nhưng không hoàn thành hết) đều đặt lại bộ đếm về 0 ngay. |
| BR-12 | Thưởng mốc chuỗi: khi `current_streak` đạt 3 → thưởng thêm +50 XP / +20 Gold; khi đạt 7 → thưởng thêm +200 XP / +100 Gold, sau đó đặt lại bộ đếm về 0. Chu kỳ lặp lại từ đầu. Mỗi mốc chỉ thưởng một lần cho mỗi lần đạt. |

### Luật xác định ngày và múi giờ

| ID | Luật |
| --- | --- |
| BR-13 | Mỗi instance có `scheduled_date` (kiểu DATE) là ngày lịch mà instance thuộc về, cố định lúc sinh instance. Ngày của instance cho luật chuỗi (BR-10..12) luôn là `scheduled_date`, không bao giờ tính lại. Khi hoàn thành, lưu thêm `completed_at` (UTC instant) chỉ để lưu vết, không dùng để quyết định ngày. |
| BR-14 | Hệ thống lưu `user_timezone`, chỉ dùng để tính `today_local` (ngày lịch hiện tại theo giờ người dùng). `today_local` dùng cho: (1) cho phép hoàn thành instance khi `scheduled_date == today_local` (BR-21); (2) phát hiện instance quá hạn để tự đóng (BR-22). |
| BR-15 | Đổi `user_timezone` chỉ ảnh hưởng cách tính `today_local` từ lúc đổi trở đi. `scheduled_date` của các instance đã có không thay đổi; lịch sử chuỗi bất biến. |

### Luật hoàn thành Quest

| ID | Luật |
| --- | --- |
| BR-16 | Giao diện phải yêu cầu người dùng xác nhận trước khi đánh dấu hoàn thành Quest, kèm cảnh báo thao tác **không thể hoàn tác**. |
| BR-17 | Lần hoàn thành Quest là bất biến trong MVP: không hỗ trợ hoàn tác (undo). XP, Gold và chuỗi đã trao không bị thu hồi. Hoàn tác thuộc phạm vi phát triển giai đoạn sau. |

### Luật Quest lặp lại và instance

| ID | Luật |
| --- | --- |
| BR-18 | Quest được mô hình hóa thành `quest_template` (định nghĩa + quy tắc lặp) và `quest_instance` (mỗi ngày một bản ghi, có `scheduled_date` và trạng thái). Quest lặp lại sinh sẵn nhiều instance, mỗi `scheduled_date` một instance. Quest một lần là một instance duy nhất với `scheduled_date` riêng. Mỗi instance hoàn thành độc lập, không ảnh hưởng instance khác. |
| BR-19 | Quest lặp lại **bắt buộc** có ngày kết thúc (hoặc số kỳ). Khoảng lặp cho mỗi lần tạo tối đa **1 năm**; vượt quá thì chặn. |
| BR-20 | Chỉ Quest hằng ngày mới có thể duy trì chuỗi. Vì ngày trống làm đứt chuỗi (BR-11), Quest lặp theo tuần / tùy chỉnh và các ngày không có instance sẽ làm đứt chuỗi. Để giữ chuỗi, người dùng cần có ít nhất một Quest hằng ngày và hoàn thành mỗi ngày. |
| BR-21 | Chỉ cho phép hoàn thành instance có `scheduled_date == today_local`. Không cho hoàn thành sớm (instance tương lai) và không cho hoàn thành bù (instance quá hạn). |
| BR-22 | Instance có `scheduled_date < today_local` mà chưa hoàn thành sẽ bị **tự động đánh dấu thất bại và đóng**. Việc tự đóng do một job nền thực hiện tại thời điểm chuyển ngày theo múi giờ người dùng. Instance đã đóng là không hợp lệ (BR-10) nên làm đứt chuỗi. |
| BR-23 | Sửa hoặc xóa `quest_template` chỉ áp dụng cho các instance tương lai **chưa hoàn thành**. Instance đã hoàn thành là bất biến (BR-17). |
| BR-24 | Mỗi instance chỉ hoàn thành một lần. MVP không hỗ trợ hoàn thành nhiều lần trong một ngày (ví dụ "uống nước 3 lần"). |

## 6. Tình huống cần nghiệm thu

1. Người dùng tạo được Quest với độ khó, danh mục và lịch lặp lại mong muốn.
2. Người dùng xác nhận rồi đánh dấu hoàn thành Quest và nhận phản hồi tức thì; XP và Gold được cập nhật đúng mức theo độ khó (BR-04). Thao tác không thể hoàn tác.
3. Người dùng thấy XP, Level và số dư Gold hiện tại.
4. Người dùng tạo được phần thưởng có giá Gold; khi đủ Gold và xác nhận thì đổi thành công, Gold bị trừ và lịch sử ghi lại; khi thiếu Gold thì thao tác đổi bị chặn. Bản ghi lịch sử không thể hoàn tác hay xóa.
5. Người dùng giữ chuỗi ngày hợp lệ liên tục nhận thưởng mốc tại ngày thứ 3 và thứ 7 đúng mức ở BR-12; bỏ lỡ một ngày làm chuỗi về 0.
6. Người dùng đổi múi giờ: `scheduled_date` của các instance đã có không thay đổi và chuỗi quá khứ không thay đổi; chỉ cách tính `today_local` áp dụng múi giờ mới.
7. Người dùng tạo Quest lặp lại hằng ngày trong 10 ngày: hệ thống sinh 10 instance, mỗi ngày một instance; hoàn thành instance nào chỉ xong instance đó.
8. Người dùng không thể hoàn thành instance của ngày tương lai hay ngày đã qua; instance quá hạn chưa hoàn thành bị tự đóng và làm đứt chuỗi.

## 7. Quyết định còn mở

Toàn bộ luật nghiệp vụ cốt lõi của MVP đã được chốt (mục 5). Các mục còn lại thuộc giai đoạn sau: hoàn tác thao tác hoàn thành, hoàn thành nhiều lần trong ngày, và các tính năng xã hội (Guild, bảng xếp hạng, Boss).
