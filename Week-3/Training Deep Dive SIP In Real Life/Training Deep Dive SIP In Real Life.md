# PHẦN 1: CHIỀU GỌI RA (OUTBOUND)

## **Kết luận 1: Kiểm tra bản tin INVITE (Header From/To)**

- **Nguyên văn:** "Nếu gọi ra nhà mạng không được, thì phải kiểm tra lại là bản tin invite from to đã đúng chưa."
    
=> Khi gọi ra bị lỗi (ví dụ 403, 404, 503), việc đầu tiên là mở gói tin `INVITE` đầu tiên gửi đi. Kiểm tra xem `From` (số chủ gọi - DID) và `To` (số bị gọi) có đúng định dạng nhà mạng yêu cầu không (ví dụ: có cần thêm 84, có cần bỏ số 0 đầu không).

## **Kết luận 2: Lỗi do phía Nhà mạng (Carrier Block)**

- **Nguyên văn:** "Nếu invite đúng rồi, nhưng nhà mạng không cho gọi ra, thì phải đi kiểm tra với nhà mạng."
    
=> Nếu đã gửi gói tin `INVITE` chuẩn (From/To đúng), nhưng nhà mạng trả về lỗi (như 503 Service Unavailable hoặc 403 Forbidden), thì lỗi nằm ở phía nhà mạng (hết tiền, chưa mở chiều gọi, chặn đầu số...).

## **Kết luận 3: Lỗi cấu hình Tổng đài (Missing SIP Packet)**

- **Nguyên văn:** "Nếu setup bị sai, có khả năng sẽ không có bản tin SIP từ tổng đài đi ra nhà mạng. Nên cần phải kiểm tra Outbound Route và gateway lại."
    
=> Khi bật `sngrep` lên mà **không thấy** bất kỳ dòng `INVITE` nào chạy từ IP Tổng đài sang IP Nhà mạng, nghĩa là tổng đài chưa biết đường đẩy cuộc gọi đi.

 => Kiểm tra lại **Outbound Routes** (Dialplan) và cấu hình **Gateway** (đã Started/Running chưa).

# PHẦN 2: CHIỀU GỌI VÀO (INBOUND)

## **Kết luận 4: Kiểm tra thông tin INVITE chiều về (Header Match)**

- **Nguyên văn:** "Tổng đài đã setup inbound nhưng không thể nhận cuộc gọi được... thì phải kiểm tra cái gì... kiểm tra From/To của bản tin invite từ nhà mạng qua tổng đài."
    
=> Khi nhà mạng đẩy cuộc gọi vào, họ gửi `INVITE`. Thì phải xem họ gửi `To: <số_nào>`. Nếu họ gửi `To: 028...` mà cấu hình Inbound Route (Destination) là `8428...` thì sẽ lệch và không nhận được cuộc gọi.

## **Kết luận 5: Lỗi định tuyến từ Nhà mạng (No Packet Arrived)**

- **Nguyên văn:** "Nếu nhà mạng đã nói là cuộc gọi đã đổ về rồi... Nếu không có bản tin sếp (SIP), thì phần này khả năng cao là lỗi do nhà mạng."
    
=>  Khi đứng ở Server, bật `sngrep`, lấy di động gọi vào số hotline. Nếu nghe tút tút hoặc báo bận mà trên màn hình `sngrep` **không hiện ra dòng nào**, tức là gói tin chưa hề tới cửa. Lỗi do nhà mạng chưa định tuyến (route) đúng IP server.

## **Kết luận 6: Lỗi cấu hình nhận trên Tổng đài (Packet Arrived but Rejected)**

- **Nguyên văn:** "Nếu nhà mạng đã rút đầu số về đúng... nhưng tổng đài trả mã lỗi gì đó thì phía tổng đài phải kiểm tra lại cấu hình."
    
=> ** Nếu thấy gói tin `INVITE` từ nhà mạng bay vào, nhưng Tổng đài tự động trả lời lại là `404 Not Found`, `480 Temporarily Unavailable`... thì lỗi do chưa cấu hình **Destinations (Inbound Route)** hoặc sai **Domain/ACL**.

## **Kết luận 7: Quyền quyết định IP/Port (Carrier Rules)**

- **Nguyên văn:** "Bản tin từ nhà mạng xuống Tống Đài thì nhà mạng sẽ là người quyết định gửi về nội dung... gửi về IP nào, Port nào. Nên nếu bất kỳ những phần nào đã nêu là sai thì phải kêu nhà mạng đổi lại."
    
=> Đối với chiều gọi vào (Inbound), nhà mạng là người gửi tin. Họ quyết định chở gói tin đến cổng 5060 hay 5080 của Tổng Đài. Nếu họ chở nhầm cổng (ví dụ Profile Internal thay vì External), ta không sửa được trên server mà phải gửi Ticket yêu cầu nhà mạng sửa IP/Port đích.


## **Kết luận 8: Bản chất luồng gói tin (Default Behavior)**

- **Nguyên văn:** "Nếu... tổng đài không setup destination, không setup inbound... thì nhà mạng có bản tin ship đổ về tổng đài hay không? Có."

