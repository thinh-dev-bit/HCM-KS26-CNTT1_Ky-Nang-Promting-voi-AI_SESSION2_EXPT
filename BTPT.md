# Bước 1: Đề xuất 2 giải pháp Prompt mẫu

**Bài toán:** Trích xuất thông tin từ tin nhắn báo hỏng của khách hàng thành dữ liệu JSON gồm: Tên, SĐT, Mã đơn, Sản phẩm, Lỗi.

## 🔴 Giải pháp 1: Prompt Đơn giản (Zero-shot, thiếu ràng buộc)
Giải pháp này gõ theo bản năng, văn nói nhanh gọn, không có ví dụ mẫu và không lường trước lỗi.
> **Prompt 1:**
> "Trích xuất thông tin khách hàng từ tin nhắn sau ra định dạng JSON giúp tôi. Các thông tin cần lấy là: Tên, Số điện thoại, Mã đơn hàng, Tên sản phẩm, và Lỗi.
> Tin nhắn: *'Shop ơi chuột Logitech G102 mua tuần trước bị double click rồi, kiểm tra bảo hành giúp mình nha.'*"

## 🟢 Giải pháp 2: Prompt Chi tiết (Khung CREATE + Few-shot + Delimiters)
Giải pháp này áp dụng tư duy kỹ sư: Phân vai, đặt luật lệ chặn lỗi biên, đưa ví dụ mẫu và ép khuôn định dạng đầu ra.
> **Prompt 2:**
> **[Context]** Bạn là Hệ thống Trích xuất Dữ liệu tự động của TechShop.
> **[Request]** Hãy đọc tin nhắn thô trong thẻ `<data>`, trích xuất thông tin ra định dạng JSON.
> **[Explanation - Chặn lỗi biên]** Nếu khách hàng không cung cấp thông tin nào (ví dụ thiếu SĐT hoặc Mã đơn), tuyệt đối không tự bịa ra, phải điền giá trị là `"NULL"`.
> **[Action & Few-shot]** Trích xuất các Key: `Name`, `Phone`, `Order_ID`, `Product`, `Issue`. 
> Học theo ví dụ mẫu sau:
> *Input:* "Mình là Tuấn, sđt 0901234567. Bàn phím cơ bị liệt nút cách."
> *Output:* `{"Name": "Tuấn", "Phone": "0901234567", "Order_ID": "NULL", "Product": "Bàn phím cơ", "Issue": "Liệt nút cách"}`
> **[Tone]** Khách quan, máy móc.
> **[Extras]** KHÔNG chào hỏi, KHÔNG giải thích. Chỉ xuất 1 khối mã JSON duy nhất.
> `<data>`Shop ơi chuột Logitech G102 mua tuần trước bị double click rồi, kiểm tra bảo hành giúp mình nha.`</data>`

---

# Bước 2: Lập bảng so sánh Trade-off

Bảng phân tích sự đánh đổi khi lựa chọn giữa 2 phương pháp ra lệnh:

| Tiêu chí | Giải pháp 1 (Đơn giản, Zero-shot) | Giải pháp 2 (Chi tiết, CREATE + Few-shot) |
| :--- | :--- | :--- |
| **Độ dễ hiểu / Công sức soạn thảo** | **Cao.** Dễ gõ, tốn ít công sức suy nghĩ, ai cũng có thể viết ngay lập tức. | **Trung bình.** Đòi hỏi tư duy hệ thống, tốn thời gian soạn thảo bộ khung ban đầu. |
| **Tốn Token (Chi phí API)** | **Ít.** Do câu lệnh đầu vào (Input Prompt) rất ngắn gọn. | **Nhiều hơn.** Phải tốn thêm token để nạp phần bối cảnh, luật lệ và các ví dụ mẫu (Few-shot). |
| **Chặn lỗi biên (Edge Cases)** | **Rất kém.** Nếu khách quên đưa SĐT, AI có thể tự lấy SĐT ngẫu nhiên để điền vào cho đủ form, gây sai lệch cơ sở dữ liệu. | **Cực tốt.** Nhờ có Rule rõ ràng, AI sẽ gán chữ `"NULL"` để báo hiệu hệ thống cần gọi điện hỏi thêm khách. |
| **Độ sạch định dạng (Automation)**| **Thấp.** AI hay lải nhải *"Dạ đây là kết quả JSON của bạn..."* làm lỗi phần mềm khi quét dữ liệu. | **Tuyệt đối 100%.** Ép ra chuỗi JSON nguyên chất, đẩy thẳng được vào Data Warehouse. |

---

# Bước 3: Lựa chọn, giải thích và Chạy thử

## 1. Quyết định lựa chọn: 
Ban Giám đốc TechShop bắt buộc phải chọn **Giải pháp 2 (Prompt Chi tiết)** để áp dụng cho quy trình tự động hóa.

## 2. Giải thích lý do (Justification):
Trong tự động hóa doanh nghiệp, **tính ổn định định dạng** và **độ chính xác của dữ liệu** quan trọng hơn rất nhiều so với việc tiết kiệm vài đồng chi phí Token. 
*   Nếu dùng *Giải pháp 1*, dữ liệu rác (lời chào của AI) hoặc dữ liệu ảo giác (SĐT AI tự bịa) sẽ làm sập phần mềm quản lý (Crash System) hoặc gửi nhầm hàng, gây thiệt hại uy tín và chi phí xử lý sự cố lớn hơn gấp hàng vạn lần tiền Token tiết kiệm được. 
*   *Giải pháp 2* tuy tốn công thiết lập một lần ban đầu (Làm Template), nhưng sau đó sẽ tự động chạy trơn tru, bảo vệ toàn vẹn dữ liệu cho hàng ngàn tin nhắn.

## 3. Mô phỏng chạy thử kết quả Giải pháp 2:
*(Nạp Giải pháp 2 với Input: "Shop ơi chuột Logitech G102 mua tuần trước bị double click rồi, kiểm tra bảo hành giúp mình nha.")*

**Output thu được từ AI:**
```json
{
  "Name": "NULL",
  "Phone": "NULL",
  "Order_ID": "NULL",
  "Product": "Chuột Logitech G102",
  "Issue": "Bị double click"
}