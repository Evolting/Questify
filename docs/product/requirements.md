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
| Q-02 | Quest có thể lặp lại hằng ngày, hằng tuần hoặc theo lịch tùy chỉnh. |
| Q-03 | Người dùng có thể đánh dấu hoàn thành Quest bằng một lần chạm hoặc nhấp. Giao diện phản hồi ngay sau thao tác. |

### XP và Level

| ID | Yêu cầu |
| --- | --- |
| XP-01 | Hoàn thành Quest giúp người dùng nhận XP. |
| XP-02 | Ứng dụng hiển thị XP và tiến trình lên cấp của người dùng. |
| XP-03 | Ngưỡng XP của mỗi Level tuân theo công thức ở mục 5. |

### Gold và phần thưởng

| ID | Yêu cầu |
| --- | --- |
| G-01 | Hoàn thành Quest giúp người dùng nhận Gold cùng với XP. |
| G-02 | Ứng dụng hiển thị số dư Gold của người dùng. |
| R-01 | Người dùng có thể tự tạo phần thưởng ngoài đời và đặt giá bằng Gold. |
| R-02 | Người dùng có thể dùng Gold để đổi phần thưởng đã tạo. |

## 5. Luật nghiệp vụ đã xác định

| ID | Luật |
| --- | --- |
| BR-01 | Ngưỡng XP để lên cấp: $MaxXP = 100 \times Level^{1.5}$. Không thay đổi công thức này khi chưa có yêu cầu rõ ràng. |
| BR-02 | Một lần hoàn thành Quest hợp lệ tạo phần thưởng XP và Gold. Mức thưởng cụ thể chưa được chốt. |
| BR-03 | Giá của mỗi phần thưởng do người dùng đặt bằng Gold. |

## 6. Tình huống cần nghiệm thu

1. Người dùng tạo được Quest với độ khó, danh mục và lịch lặp lại mong muốn.
2. Người dùng đánh dấu hoàn thành Quest trong một thao tác và nhận phản hồi tức thì; XP và Gold được cập nhật theo luật thưởng sau khi luật đó được chốt.
3. Người dùng thấy XP, Level và số dư Gold hiện tại.
4. Người dùng tạo được phần thưởng có giá Gold và thực hiện thao tác đổi thưởng.

## 7. Quyết định còn mở

Các mục dưới đây cần được thống nhất trước khi viết logic tương ứng:

1. **XP và Gold:** Mỗi độ khó thưởng bao nhiêu? Có thưởng khác nhau theo danh mục hoặc chuỗi ngày hoàn thành không?
2. **Lên cấp:** Level ban đầu là bao nhiêu? Làm tròn ngưỡng XP như thế nào? XP dư sau khi lên cấp được xử lý ra sao?
3. **Quest lặp lại:** Lịch tùy chỉnh hỗ trợ những quy tắc nào? Một Quest được hoàn thành tối đa bao nhiêu lần trong một kỳ? Múi giờ nào quyết định ngày mới?
4. **Thao tác hoàn thành:** Có thể hoàn tác không? Nếu có, XP và Gold đã trao được xử lý thế nào?
5. **Đổi thưởng:** Có bắt buộc đủ Gold trước khi đổi không? Khi đổi thành công, có cần lưu lịch sử và cho phép hoàn tác không?

Các câu trả lời được bổ sung vào mục 5 và tình huống nghiệm thu tương ứng khi đã được chốt.
