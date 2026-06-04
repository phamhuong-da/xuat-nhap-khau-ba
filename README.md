# 📋 BA Documents — Phần mềm Logistics TDI
### Chức năng Quy đổi Ngoại tệ & Quản lý Kỳ Bảng kê

**Vai trò:** Business Analyst  
**Công ty:** Sơn Thành Holding  
**Thời gian:** 03/2026 – Hiện tại  
**Trạng thái:** 🟢 Đang triển khai

---

## 🎯 Bối cảnh & Vấn đề

Phần mềm Logistics TDI quản lý toàn bộ hoạt động dịch vụ vận chuyển hàng hóa 
xuất nhập khẩu tại cửa khẩu. Khi mở rộng sang khách hàng Trung Quốc, hệ thống 
gặp 2 vấn đề nghiệp vụ lớn cần giải quyết:

**Vấn đề 1 — Quy đổi ngoại tệ thủ công:**  
Nhân viên phải tra tỷ giá Vietcombank thủ công mỗi ngày rồi nhập vào từng phiếu 
dịch vụ → dễ sai sót, mất thời gian, không có lịch sử kiểm soát.

**Vấn đề 2 — Quản lý kỳ bảng kê thiếu kiểm soát:**  
Không có cơ chế phê duyệt khi cần chuyển kỳ bảng kê → kế toán không kiểm soát 
được thay đổi dữ liệu sau khi đã chốt kỳ → rủi ro sai lệch báo cáo tài chính.

---

## 🔧 Phạm vi dự án BA

Tôi đảm nhận toàn bộ phân tích và đặc tả 3 chức năng:

| Chức năng | Mô tả ngắn | Use Case |
|---|---|---|
| **Quy đổi ngoại tệ** | Tự động lấy tỷ giá Vietcombank, fallback logic, override tay có audit | UC-QD01 |
| **Chuyển kỳ bảng kê** | Quy trình gửi yêu cầu → kế toán phê duyệt → cập nhật tự động | UC-KY01 |
| **Danh mục kỳ** | Admin setup danh sách kỳ theo năm, validate không trùng lặp | UC-KY02 |

---

## 📁 Tài liệu trong repo này

| File | Mô tả |
|---|---|
| [`documents/SRS-Quy-doi-Ngoai-te-Quan-ly-Ky-Bang-ke-v1.1.docx`](documents/SRS-Quy-doi-Ngoai-te-Quan-ly-Ky-Bang-ke-v1.1.docx) | Tài liệu SRS đầy đủ — Use Case, Business Rules, Data Mapping, Mockup spec |
| [`documents/use-cases-summary.md`](documents/use-cases-summary.md) | Tóm tắt 3 Use Case chính, luồng ngoại lệ, bảng trường dữ liệu |

---

## 🔗 Figma UI

| Màn hình | Link |
|---|---|
| Giao diện quy đổi ngoại tệ | [Xem Figma →](https://www.figma.com/design/SLkDuQi0XbT5xio9gvCfY8/STH---Chu%E1%BB%97i-d%E1%BB%8Bch-v%E1%BB%A5-Logistics---WEB?node-id=4793-11584) |
| Màn hình chuyển kỳ — giao diện User | [Xem Figma →](https://www.figma.com/design/SLkDuQi0XbT5xio9gvCfY8/STH---Chu%E1%BB%97i-d%E1%BB%8Bch-v%E1%BB%A5-Logistics---WEB?node-id=4796-52610) |
| Màn hình Audit Log | [Xem Figma →](https://www.figma.com/design/SLkDuQi0XbT5xio9gvCfY8/STH---Chu%E1%BB%97i-d%E1%BB%8Bch-v%E1%BB%A5-Logistics---WEB?node-id=4827-45506) |

---

## 💡 Điểm nổi bật về mặt nghiệp vụ

**Fallback logic tỷ giá** — Không đơn giản là "lấy tỷ giá ngày 01". Tôi phân 
tích và đặc tả cơ chế tự động tìm ngày làm việc gần nhất khi ngày 01 rơi vào 
lễ/cuối tuần, kèm xử lý timeout và cache để tối ưu hiệu năng.

**Ràng buộc chuyển kỳ ±1** — Sau khi phân tích rủi ro, tôi đề xuất giới hạn 
chỉ cho phép chuyển sang kỳ liền kề (trước hoặc sau 1 kỳ) thay vì tự do chọn 
bất kỳ kỳ nào — giảm thiểu nguy cơ sai lệch số liệu tài chính trên diện rộng.

**Data Migration Rules** — Đặc tả rõ quy tắc chuyển đổi dữ liệu lịch sử 
(USD/CNY cũ sang cấu trúc mới) với tỷ giá mặc định và phạm vi áp dụng cụ thể, 
đảm bảo tính kế thừa mà không làm sai lệch báo cáo đã xuất.

**Audit Log toàn diện** — Mọi thay đổi tỷ giá, chuyển kỳ đều được ghi lại 
đầy đủ (user, thời gian, giá trị cũ → mới) giúp kế toán kiểm soát và truy vết.

---

## 🛠 Công cụ sử dụng

`Notion` · `Draw.io` · `Figma` · `Google Meet` · `Word (SRS)`

---

*Dữ liệu và tên công ty trong tài liệu đã được ẩn/thay thế ở những phần nhạy cảm.*
