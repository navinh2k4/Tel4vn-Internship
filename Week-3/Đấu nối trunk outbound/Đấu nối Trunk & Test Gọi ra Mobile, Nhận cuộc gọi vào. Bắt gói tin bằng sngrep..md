
Bước 1: Chọn tenant
Bước 2: Chọn vào menu Accounts → Gateways
![](Pasted%20image%2020260120114125.png)

Bước 3: Chọn vào icon [+] để tạo thêm Gateways
![](Pasted%20image%2020260120114149.png)

Bước 4: Trong bảng Gateway nhập các thông tin
![](Pasted%20image%2020260120114353.png)

![](Pasted%20image%2020260120114408.png)

### 1. Tại sao `Caller ID In From` = True? 

Đây là lỗi phổ biến nhất khi đấu nối.

- **Bản chất:** Gói tin SIP `INVITE` (gọi đi) có một dòng Header là `From:` (Ai đang gọi?).
    
- **Nếu `False` (Mặc định):** FusionPBX sẽ lấy số Extension để điền vào.
    
    - _Ví dụ:_ `From: "Nguyen Van A" <sip:1000@192.168.1.50>`
        
    - _Hậu quả:_ Nhà mạng Pitel nhận được gói tin, họ kiểm tra hệ thống và thấy "Thằng `1000` là thằng nào? Tao chỉ bán dịch vụ cho tài khoản `5315461852` thôi". -> **Chặn cuộc gọi (Lỗi 403 Forbidden).**
        
- **Nếu `True`:** FusionPBX sẽ lấy **Username** của Trunk (chính là số tài khoản chị đưa) để điền vào, bất kể ai gọi từ bên trong.
    
    - _Ví dụ:_ `From: "5315461852" <sip:5315461852@pitel.com>`
        
    - _Kết quả:_ Nhà mạng thấy "À, khách hàng VIP `5315...` đang gọi". -> **Cho phép (200 OK).**
        

**Kết luận để trả lời:** _"Dạ, phải bật True để server sửa lại Header 'From' thành User của nhà mạng thì mới xác thực được (Authentication), nếu không sẽ bị lỗi 403 Forbidden ạ."_

### 2. Về vấn đề Inbound (Gọi vào) bị chặn do NAT KTX

Chị chấp nhận lý do này. Mạng KTX/Cafe thường chặn các kết nối từ ngoài vào (Inbound Connection) và em không thể mở Port trên Router của họ.

- **Cách trả lời khi Demo:** _"Dạ phần gọi vào (Inbound) hiện tại em chưa demo được do mạng KTX chặn port chiều về. Tuy nhiên, em đã nắm được lý thuyết là cần cấu hình **Inbound Route** với Regex phù hợp (ví dụ `^.*$` để test) và đảm bảo NAT Keep-alive."_

### 3. Tại sao Tài liệu Ánh Dương để `Register = False`? (Thắc mắc cực hay!)

Câu hỏi này chứng tỏ em đã so sánh rất kỹ. Đây là sự khác biệt giữa **SIP Registration** và **SIP Trunking (IP Authentication)**.

- **Mô hình Lab của em (Pitel): Dùng `Register = True`**
    
    - **Giống như:** Em dùng Zoiper/Softphone.
        
    - **Cơ chế:** Server của em (FusionPBX) đóng vai trò là một "Client", định kỳ gửi tin nhắn `REGISTER` (kèm User/Pass) lên Server nhà mạng để báo "Em đang ở đây, có gì gọi em nhé".
        
    - **Tại sao:** Vì em dùng mạng KTX (IP động), em cần đăng ký để nhà mạng biết IP mới nhất của em.
        
- **Mô hình Ánh Dương (Thực tế): Dùng `Register = False`**
    
    - **Giống như:** Hai Server lớn nói chuyện ngang hàng (Peer-to-Peer).
        
    - **Cơ chế:** Hai bên (FusionPBX và Grandstream/CMC) đã biết IP tĩnh (Static IP) của nhau rồi. Tin tưởng nhau tuyệt đối qua IP. Không cần User/Pass, không cần đăng ký. Cứ có cuộc gọi là bắn thẳng sang IP bên kia.
        
    - **Tại sao:** Để giảm tải, tăng tốc độ kết nối và bảo mật theo IP.