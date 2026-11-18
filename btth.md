## [BTTH] Bài tập thực hành tổng hợp
### THIẾT KẾ CHUỖI PROMPT (PROMPT CHAINING) KHÉP KÍN

*BƯỚC 1: XỬ LÝ DỮ LIỆU ĐẦU VÀO & SUY LUẬN (CHAIN-OF-THOUGHT)*

- **Prompt 1:**
Bạn là hệ thống phân tích dữ liệu CSKH tự động.
Phân tích khiếu nại sau: 'Máy mới mua hôm qua mà sọc màn hình, yêu cầu đổi mới gấp!'.
1. Áp dụng Chain-of-Thought suy luận từng bước: 
   - Kiểm tra thời gian mua (hôm qua = trong vòng 7 ngày).
   - Xác định lỗi kỹ thuật (sọc màn hình = lỗi phần cứng từ nhà sản xuất, đủ điều kiện Dead on Arrival).
2. Trả về kết quả dưới dạng JSON gồm các trường: status (Approved/Rejected), error_type (Hardware/Software), urgency_level (High/Medium/Low)

*BƯỚC 2: TẠO NỘI DUNG PHẢN HỒI TỰ ĐỘNG & BẮT LỖI DỮ LIỆU*
- **Mục tiêu:** Nhận kết quả từ Bước 1, tự động soạn email xin lỗi lịch sự, chuyên nghiệp, đồng thời áp dụng cơ chế bắt lỗi dị biệt (phòng hờ dữ liệu đầu vào bị thiếu/lỗi).
- **Prompt 2:**
Bạn là Trưởng bộ phận Chăm sóc Khách hàng của TechShop.
Dựa vào kết quả JSON từ bước trước:
{
  "status": "Approved",
  "error_type": "Hardware",
  "urgency_level": "High"
}
1. Soạn thảo một email xin lỗi khách hàng và thông báo quy trình đổi mới 1-đổi-1 miễn phí.
2. Cơ chế bắt lỗi dự phòng: Nếu trường 'status' trống hoặc lỗi định dạng, hệ thống tự động gán nhãn 'Pending Verification' và yêu cầu nhân viên kiểm tra thủ công, tuyệt đối không làm crash hệ thống phản hồi
---
### RÀNG BUỘC KỸ THUẬT & TỐI ƯU TOKEN
- **Code sạch, tối ưu token:** Sử dụng cấu trúc JSON gọn gàng giữa các prompt trung gian để giảm thiểu lượng token dư thừa.
- **Cơ chế bắt lỗi dị biệt (Error Handling):** Có các điều kiện rẽ nhánh (if/else logic bằng ngôn ngữ tự nhiên) trong prompt để xử lý trường hợp thông tin khách hàng không đầy đủ mà không làm gián đoạn toàn bộ hệ thống dây chuyền.
---
### KẾT QUẢ ĐẦU RA TỪ AI (SẠCH LỖI, CHUẨN XÁC)
```json
{
  "status": "Approved",
  "error_type": "Hardware",
  "urgency_level": "High"
}
```
### Nội dung Email phản hồi khách hàng

**Tiêu đề:** [TechShop] Xin lỗi về sự cố sản phẩm & Thông báo hỗ trợ đổi mới 1-đổi-1 miễn phí

Kính gửi Quý khách hàng,

Thay mặt bộ phận Chăm sóc Khách hàng của TechShop, tôi chân thành xin lỗi Quý khách vì trải nghiệm không mong muốn với chiếc máy vừa mua gặp tình trạng sọc màn hình.

Qua kiểm tra thông tin phản hồi, hệ thống xác nhận thiết bị của Quý khách gặp lỗi phần cứng từ nhà sản xuất và hoàn toàn đủ điều kiện áp dụng chính sách **đổi mới 1-đổi-1 hoàn toàn miễn phí** (áp dụng cho sản phẩm lỗi kỹ thuật trong vòng 7 ngày đầu).

Để hỗ trợ Quý khách khắc phục sự cố nhanh nhất có thể, TechShop xin thông báo quy trình đổi mới ưu tiên như sau:

1.  **Xác nhận thông tin:** Quý khách vui lòng phản hồi email này kèm theo **Mã đơn hàng** và **Địa chỉ nhận/giao hàng** (hoặc chi nhánh TechShop gần nhất mà Quý khách muốn đến trực tiếp).
    
2.  **Kiểm tra & Thu hồi:** Nhân viên kỹ thuật của TechShop sẽ liên hệ trực tiếp trong vòng **2 giờ làm việc** để xác nhận tình trạng máy và sắp xếp nhân viên giao nhận đến tận nơi hỗ trợ (nếu Quý khách ở khu vực nội thành).
    
3.  **Bàn giao máy mới:** Quý khách sẽ được nhận ngay một sản phẩm mới 100% nguyên seal cùng loại mà không phát sinh bất kỳ chi phí nào.
    

Một lần nữa, TechShop rất mong Quý khách thông cảm cho sự bất tiện này. Nếu cần hỗ trợ khẩn cấp, Quý khách vui lòng liên hệ trực tiếp qua hotline ưu tiên:  **[Số Hotline CSKH]**.

Trân trọng,

**Trưởng bộ phận Chăm sóc Khách hàng**

Hệ thống bán lẻ TechShop

### Quy tắc xử lý dự phòng (Fallback Mechanism)

-   **Điều kiện kích hoạt:** Trường `status` bị bỏ trống (`null`/`""`), sai kiểu dữ liệu hoặc nằm ngoài giá trị hợp lệ (`Approved` / `Rejected`).
    
-   **Hành động tự động:**
    
    1.  Hệ thống tự động ghi đè giá trị thành:  `"status": "Pending Verification"`.
        
    2.  Tạm ngưng gửi email phản hồi tự động cho khách hàng.
        
    3.  Chuyển tiếp ticket sang hàng đợi **Kiểm tra thủ công (Manual Review)** và gửi cảnh báo cho chuyên viên CSKH trực ca để xử lý trực tiếp, đảm bảo luồng vận hành không bị gián đoạn.