# Tóm tắt Use Case — Phần mềm Logistics TDI
## Chức năng Quy đổi Ngoại tệ & Quản lý Kỳ Bảng kê

> **Phiên bản:** 1.1 · **Tác giả:** Phạm Thị Hương · **Cập nhật:** 17/03/2026  
> **Tài liệu SRS đầy đủ:** [Tải xuống](SRS-Quy-doi-Ngoai-te-Quan-ly-Ky-Bang-ke-v1.1.docx)

---

## Đối tượng sử dụng

| Tác nhân | Vai trò |
|---|---|
| User (Nhân viên dịch vụ) | Nhập quy đổi ngoại tệ, gửi yêu cầu chuyển kỳ |
| Kế toán / Admin | Phê duyệt chuyển kỳ, thiết lập Danh mục Kỳ, kiểm tra và xuất báo cáo |

---

## UC-QD01 — Quy đổi Ngoại tệ trên Phiếu Dịch vụ

**Module:** Phiếu dịch vụ → Tab Chi phí / Doanh thu  
**Độ ưu tiên:** Must have

### Luồng nghiệp vụ
1. User mở tab Chi phí hoặc Doanh thu trong phiếu dịch vụ
2. Chọn loại tiền tệ từ dropdown (VND / USD / CNY / EUR)
3. Hệ thống tự động lấy tỷ giá từ **Vietcombank** (ưu tiên ngày 01 hàng tháng)
4. Nếu ngày 01 không có dữ liệu (lễ/cuối tuần) → **tự động fallback** sang ngày làm việc tiếp theo
5. User có thể nhấn **"Lấy theo tỷ giá Vietcombank"** hoặc nhập tay (override)
6. Hệ thống tự động tính và hiển thị giá trị quy đổi VNĐ (read-only)
7. Lưu phiếu

### Business Rules

- Loại tiền tệ mặc định khi tạo dòng mới: **VND**
- Khi chọn VND: trường tỷ giá tự động khóa (disabled) và gán giá trị = 1
- Tỷ giá quy đổi: tối đa 4 chữ số thập phân
- Giá trị quy đổi sang VNĐ: luôn làm tròn thành số nguyên
- Các dòng nhập tay được đánh dấu nổi bật (nền vàng hoặc icon cảnh báo)
- Mọi thao tác nhập tay được ghi vào Audit Log: tên user, giá trị cũ, giá trị mới, thời gian
- Tỷ giá được cache trong DB theo ngày → thời gian load < 1 giây cho các user tiếp theo

### Luồng ngoại lệ

| Mã | Điều kiện | Phản hồi hệ thống |
|---|---|---|
| 2.1 | Không chọn loại tiền tệ | Cảnh báo: "Vui lòng chọn loại tiền tệ" |
| 2.2 | Tỷ giá quy đổi ≤ 0 | Cảnh báo: "Tỷ giá phải lớn hơn 0" — không cho lưu |
| 2.3 | Không lấy được tỷ giá (fallback thất bại) | Cảnh báo: "Không lấy được tỷ giá, vui lòng nhập tay" |
| 2.4 | Website Vietcombank timeout (>10 giây) | "Lỗi kết nối nguồn tỷ giá" + ghi log lỗi hệ thống |

### Bảng trường dữ liệu

| Tên trường | Bắt buộc | Kiểu dữ liệu | Mặc định |
|---|---|---|---|
| Loại tiền tệ | ✅ | Dropdown (VND/USD/CNY) | VND |
| Tỷ giá quy đổi | ✅ | Number (4 chữ số thập phân) | Tự động theo rule VCB |
| Nguồn tỷ giá | ✅ | Text | `Auto_VCB` hoặc `Manual_Override` |
| Tổng chi / Tổng thu | — | Number (read-only) | Giá trị nguyên tệ gốc |
| Tổng chi quy đổi / Tổng thu quy đổi | — | Number (read-only) | Tự động tính, làm tròn số nguyên |

### Quy tắc migrate dữ liệu cũ

| Trường hợp | Logic xử lý |
|---|---|
| Phiếu cũ có USD/CNY | Mapping tên cột cũ → loại tiền tệ mới; tỷ giá mặc định: USD=26.300 / CNY=3.700 |
| Phiếu cũ chỉ có VNĐ | Loại tiền = VND, tỷ giá = 1, giá trị nguyên tệ = giá trị quy đổi = giá trị VNĐ cũ |
| Phạm vi áp dụng | Chỉ phiếu ở trạng thái "Đã thực hiện" hoặc "Hoàn thành" |

---

## UC-KY01 — Chuyển Kỳ Bảng kê (có Phê duyệt)

**Module:** Danh sách Phiếu dịch vụ  
**Độ ưu tiên:** Must have  
**Luồng tác nhân:** User (gửi yêu cầu) → Kế toán (phê duyệt)

### Luồng nghiệp vụ
1. User xem danh sách phiếu, đối chiếu cột Kỳ bảng kê hiện tại
2. Nhấn nút **"Chuyển kỳ"**
3. Popup hiện ra: Kỳ hiện tại + dropdown chọn kỳ mới (chỉ hiển thị kỳ liền kề ±1)
4. User chọn kỳ → nhấn **"Gửi yêu cầu"**
5. Trạng thái phiếu chuyển sang **"Chờ phê duyệt"**
6. Kế toán mở danh sách yêu cầu → xem chi tiết → nhấn **"Xác nhận"**
7. Hệ thống tự động cập nhật kỳ mới + tính lại toàn bộ quy đổi ngoại tệ theo tỷ giá kỳ mới
8. Gửi thông báo thành công đến User

### Business Rules quan trọng

- Chỉ được chuyển sang **kỳ liền kề (±1)** — tránh sai sót số liệu trên diện rộng
- Sau khi phê duyệt: toàn bộ báo cáo liên quan (Doanh thu - GV, Theo dõi DT-CP) cập nhật ngay lập tức
- Kế toán từ chối: phải nhập lý do → lưu vào lịch sử phiếu + gửi thông báo cho User
- Mọi thay đổi kỳ được ghi đầy đủ vào Audit Log

### Luồng ngoại lệ

| Mã | Điều kiện | Phản hồi hệ thống |
|---|---|---|
| 2.1 | Kỳ mới = kỳ hiện tại | Không cho gửi yêu cầu |
| 2.2 | Phiếu không có danh mục CP/DT | Cảnh báo: "Không có danh mục để chuyển kỳ" |
| 2.3 | Kế toán từ chối | Lý do từ chối gửi thông báo đến User |
| 2.4 | Chọn kỳ không liền kề | Cảnh báo: "Chỉ được phép chuyển sang kỳ liền kề trước hoặc sau" |

---

## UC-KY02 — Quản lý Danh mục Kỳ

**Module:** Quản lý thành phần → Danh mục Kỳ  
**Độ ưu tiên:** Must have  
**Tác nhân:** Kế toán / Admin

### Luồng nghiệp vụ
1. Truy cập Danh mục Kỳ → xem danh sách tổng hợp theo năm
2. Nhấn **"+ Thêm kỳ mới"** để khởi tạo bộ kỳ cho năm chưa có dữ liệu
3. Nhấn **"Xem chi tiết"** để quản lý từng kỳ trong năm
4. Thực hiện Sửa hoặc Xóa từng kỳ (chỉ áp dụng cho kỳ trong tương lai)

### Business Rules quan trọng

- Mỗi kỳ phải nằm trong phạm vi 01/01 – 31/12 của năm setup
- Không được phép trùng lặp (overlap) về thời gian giữa các kỳ
- Khi thêm năm mới: chỉ chọn được các năm **chưa có dữ liệu** trong hệ thống
- Xóa năm: chỉ cho phép với **năm tương lai**; năm hiện tại và quá khứ bị khóa (disabled)
- Xóa kỳ: chỉ cho phép với **kỳ trong tương lai**
- Quy tắc fallback nếu chưa setup: mặc định theo chu kỳ 26/n – 25/(n+1)

---

## Nhật ký thay đổi — Audit Log (tất cả chức năng)

Hiển thị dạng tab **"Lịch sử thay đổi"** read-only trong màn hình chi tiết phiếu dịch vụ.

| Cột | Kiểu | Mô tả |
|---|---|---|
| Ngày | Date | Ngày phát sinh hành động (dd/mm/yyyy) |
| Thời gian | Time | Thời gian chính xác đến từng giây (hh:mm:ss) |
| Tên tài khoản | Text | Tên User thực hiện hoặc "Hệ thống" nếu là lệnh tự động |
| Hành động | Text | Gửi yêu cầu / Nhập tay / Xác nhận / Từ chối / Tự động cập nhật |
| Nội dung | Text | Giá trị cũ → Giá trị mới (VD: "Tỷ giá: 25.450 → 25.500 (Manual_Override)") |

---

## Link tham chiếu Figma UI

| Màn hình | Link |
|---|---|
| Giao diện quy đổi ngoại tệ | [Xem Figma](https://www.figma.com/design/SLkDuQi0XbT5xio9gvCfY8/STH---Chu%E1%BB%97i-d%E1%BB%8Bch-v%E1%BB%A5-Logistics---WEB?node-id=4793-11584) |
| Màn hình chuyển kỳ (User) | [Xem Figma](https://www.figma.com/design/SLkDuQi0XbT5xio9gvCfY8/STH---Chu%E1%BB%97i-d%E1%BB%8Bch-v%E1%BB%A5-Logistics---WEB?node-id=4796-52610) |
| Màn hình Audit Log | [Xem Figma](https://www.figma.com/design/SLkDuQi0XbT5xio9gvCfY8/STH---Chu%E1%BB%97i-d%E1%BB%8Bch-v%E1%BB%A5-Logistics---WEB?node-id=4827-45506) |
