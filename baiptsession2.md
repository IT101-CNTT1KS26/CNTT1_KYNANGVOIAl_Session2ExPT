# PHÂN TÍCH TRADE-OFF: SO SÁNH GIẢI PHÁP THIẾT KẾ PROMPT TIÊU CHUẨN & ĐỊNH DẠNG


## BƯỚC 1: ĐỀ XUẤT 2 PROMPT MẪU

Để giải quyết bài toán tự động hóa trích xuất dữ liệu phản hồi khách hàng (áp dụng các kỹ thuật CREATE, Few-shot, Ép định dạng JSON/Table), dưới đây là 2 phương án thiết kế Prompt với mức độ chi tiết khác nhau:

###  Giải pháp 1: Prompt Đơn giản & Ngắn gọn

*Phương án này tập trung vào câu lệnh trực diện, lược bỏ các phần khung lý thuyết dài dòng để tiết kiệm từ vựng.*

```text
Bạn là trợ lý AI. Hãy đọc phản hồi khách hàng sau đây, trích xuất thông tin thành mã JSON và một Bảng Markdown gồm các cột: [Mã đơn, Vấn đề, Cảm xúc].
Ví dụ:
- Input: "Đơn #123 bị hỏng, chán quá."
- Output JSON: {"order_id": "#123", "issue": "Hỏng", "sentiment": "Tiêu cực"}
- Output Bảng: | #123 | Hỏng | Tiêu cực |

Input cần xử lý: "Giao hàng chậm trễ quá, không nhớ rõ mã đơn lắm nhưng bực mình ghê."

```

###  Giải pháp 2: Prompt Chi tiết (Đầy đủ ràng buộc khung CREATE & Xử lý lỗi)

*Phương án này áp dụng đầy đủ khung CREATE, Few-shot chuẩn hóa, ép định dạng nghiêm ngặt và có câu lệnh điều kiện xử lý dữ liệu trống/chống ảo giác.*

```text
[Context / Character]
Bạn là Trợ lý AI chuyên tự động hóa quy trình phân tích và chuẩn hóa dữ liệu vận hành cho hệ thống CSKH tại TechShop.

[Request]
Hãy tiếp nhận phản hồi thô từ khách hàng, bóc tách thông tin cấu trúc, xử lý các trường hợp dữ liệu bị trống và xuất kết quả theo định dạng yêu cầu.

[Examples / Few-shot]
Ví dụ 1:
- Input: "Shop ơi đơn #TS-999 giao nhanh lắm, cảm ơn shop."
- Output JSON: {"order_id": "TS-999", "issue": "Không có", "sentiment": "Tích cực", "missing_fields": []}

[Action / Adjustments & Error Handling]
Quy tắc xử lý logic:
1. Trích xuất `order_id`. Nếu không tìm thấy hoặc khách hàng không cung cấp, bắt buộc gán giá trị `"Chưa xác thực"` và ghi nhận vào mảng `missing_fields`.
2. Xác định vấn đề (`issue`) và cảm xúc (`sentiment`).

[Type of Output & Format]
Kết quả trả về PHẢI bao gồm đúng 2 phần:
1. Khối mã JSON thuần túy (đặt trong cặp ```json ... ```).
2. Một Bảng Markdown tổng hợp gồm các cột: [Mã đơn hàng | Vấn đề chính | Cảm xúc].

[Exclusions / Constraints]
- Ràng buộc chống ảo giác: Tuyệt đối KHÔNG tự bịa đặt mã đơn hàng nếu đầu vào không có.

---
[Input Data để xử lý ngay]:
"Giao hàng chậm trễ quá, không nhớ rõ mã đơn lắm nhưng bực mình ghê."

```

---

## BƯỚC 2: LẬP BẢNG SO SÁNH TRADE-OFF (ĐÁNH GIÁ ĐỔI LẤY)

| Tiêu chí đánh giá | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
| --- | --- | --- |
| **Độ dễ hiểu / Viết nhanh** | **Cao** (Dễ viết, nhanh chóng, không tốn thời gian cấu trúc). | **Trung bình** (Đòi hỏi tư duy thiết kế khung và quy tắc ràng buộc logic rõ ràng). |
| **Tốn Token (Chi phí API)** | **Ít** (Số lượng token đầu vào thấp, phản hồi nhanh). | **Nhiều hơn** (Tốn token hơn cho các khối ngữ cảnh và ví dụ Few-shot). |
| **Chặn lỗi biên (Error Handling)** | **Trung bình** (Dễ gặp hiện tượng AI tự bịa số liệu khi gặp dữ liệu trống). | **Cực tốt** (Có câu lệnh điều kiện xử lý lỗi, giá trị null và cơ chế chống ảo giác triệt để). |
| **Độ ổn định định dạng Output** | **Thấp - Trung bình** (Thỉnh thoảng AI quên cấu trúc bảng hoặc trả thiếu trường JSON). | **Rất cao** (Ép buộc cấu trúc nghiêm ngặt qua Type of Output và các quy tắc cụ thể). |

---

## BƯỚC 3: LỰA CHỌN PHƯƠNG ÁN TỐI ƯU VÀ KẾT QUẢ CHẠY THỬ

### 1. Lựa chọn phương án tối ưu

* **Giải pháp 2** là phương án tối ưu nhất và được lựa chọn cho môi trường doanh nghiệp (như quy trình tự động hóa tại TechShop).
* **Lý do lựa chọn:** Trong nghiệp vụ xử lý dữ liệu thực tế, rủi ro lớn nhất là **AI Hallucination (ảo giác)** và lệch định dạng JSON khiến hệ thống tự động hóa bị sập (crash). Đầu tư thêm một lượng token nhỏ cho Giải pháp 2 giúp đổi lại sự an toàn tuyệt đối, dữ liệu chuẩn hóa và khả năng xử lý mượt mà các trường hợp khách hàng nhập liệu thiếu sót.

### 2. Mô phỏng kết quả chạy thử (Thực thi Giải pháp 2 với Input lỗi)

Khi chạy **Giải pháp 2** với đoạn đầu vào *"Giao hàng chậm trễ quá, không nhớ rõ mã đơn lắm nhưng bực mình ghê."*, AI sẽ trả về kết quả chính xác và an toàn như sau:

```json
{
  "order_id": "Chưa xác thực",
  "issue": "Giao hàng chậm trễ",
  "sentiment": "Tiêu cực",
  "missing_fields": ["order_id"]
}

```

| Mã đơn hàng | Vấn đề chính | Cảm xúc |
| --- | --- | --- |
| *Chưa xác thực* | Giao hàng chậm trễ | Tiêu cực |