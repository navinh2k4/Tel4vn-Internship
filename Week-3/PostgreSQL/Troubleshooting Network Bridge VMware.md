
# [Topic]: Troubleshooting Network Bridge VMware
**Ngày:** 2026-01-14
**Nguồn:** [Link tài liệu/Video]
**Tags:** #network #vmware #troubleshooting #firewall

---

## 1. Hiểu nhanh (Feynman Zone)
> *Dừng lại 30s. Viết lại khái niệm này theo cách mình hiểu (như đang giải thích cho người không biết gì).*

- **Khái niệm này là gì?:** ...
- Vấn đề là gì?: Máy ảo đi được ra ngoài Internet dù là ping địa chỉ ip hay DNS đều ổn nhưng không nói chuyện được với máy thật (host).
- **Tại sao?:** Máy thật (Windows) có cơ chế tự vệ (bởi Windows Defender Firewall). Khi thấy một kết nối lạ (Bridge | Và vì là mình thường thay đổi ip server debian vì trong tuần thì thứ 7 -> thứ 3 thì mình ở nhà mà wifi (modern) ở nhà thì có dãy ip là 192.168.1.x vậy nên lúc đó mình đã đổi ip server thành 192.168.1.50, còn lúc lên công ty ở tầng 2 thì wifi có dãy ip là 192.168.2.x vậy nên mình phải đổi lại thành 192.168.2.50) hoặc đang ở chế độ mạng công cộng (Public Network), nó sẽ chặn lệnh Ping (ICMP) và SSH để bảo mật.
- **Nó khác gì cái cũ?:** ...
- Giải pháp: Cần bảo máy thật là "Đây là mạng nhà, an toàn, hãy mở cửa cho các máy khác (VM) kết nối vào".

---

## 2. Thực hành (Hands-on Lab)

### A. Chuẩn bị (Prediction)
*Trước khi gõ lệnh, mình mong đợi điều gì xảy ra?*
- [ ] Dự đoán: Sau khi tắt/mở ip trên Firewall hoặc chuyển mạng sang Private, VM sẽ ping được Host và SSH vào được.

### B. Thực thi (Execution)
*Copy lệnh và kết quả TEXT vào đây. KHÔNG DÙNG ẢNH.*
```
# Dán lệnh vào đây


```
**Bước 1: Kiểm tra Profile mạng trên máy thật (Windows)** _(Thường Windows mặc định mạng mới là Public - Chặn hết)_

1. Trên máy thật: Nhấn phím `Windows` -> Gõ "Ethernet settings" (hoặc WiFi settings nếu dùng WiFi).
    
2. Click vào kết nối mạng đang dùng.
    
3. Chuyển từ **Public network** sang **Private network**.
![](Pasted%20image%2020260114092937.png)

**Bước 2: Mở Firewall cho phép Ping (ICMP) trên máy thật** _(Nếu bước 1 chưa được, hoặc muốn chắc chắn)_

1. Nhấn phím `Windows` -> Gõ "Windows Defender Firewall with Advanced Security".
    
2. Chọn **Inbound Rules**.
    
3. Tìm dòng: **File and Printer Sharing (Echo Request - ICMPv4-In)**. (Chọn dòng áp dụng cho Profile _Private_ hoặc _All_).
    
4. Chuột phải -> **Enable Rule**.
![](Pasted%20image%2020260114093206.png)

```
# Dán kết quả trả về (Log) vào đây

Microsoft Windows [Version 10.0.26100.6899]
(c) Microsoft Corporation. All rights reserved.

C:\Users\Admin>ping 192.168.2.50

Pinging 192.168.2.50 with 32 bytes of data:
Reply from 192.168.2.17: Destination host unreachable.
Reply from 192.168.2.17: Destination host unreachable.
Reply from 192.168.2.17: Destination host unreachable.
Reply from 192.168.2.17: Destination host unreachable.

Ping statistics for 192.168.2.50:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),

C:\Users\Admin>
```

Kết quả ping từ máy ảo ra máy thật:
![](Pasted%20image%2020260114093512.png)

### C. Phân tích (Analysis)

_Kết quả có giống dự đoán không?_

- [ ] Giống.
    
- [x] Khác -> Tại sao? ...
    Vì khi thiết lập xong firewall và chuyển mạng sang private rồi vẫn chưa ping thành công 2 máy. 

---

## 3. Nhật ký lỗi (Troubleshooting)

### Phân tích nhanh hiện tượng (Analysis)

1. **Từ máy thật (Host) ping máy ảo (VM):**
    
    - **Log:** `Reply from 192.168.2.17: Destination host unreachable.`
        
    - **Ý nghĩa:** Máy thật (192.168.2.17) nói rằng: _"Tôi không biết đường nào để đi tới 192.168.2.50 cả"_.
        
    - **Suy luận:** Đây là lỗi Lớp 2 (Data Link Layer). Máy thật không tìm thấy địa chỉ MAC của máy ảo. Nếu là Firewall chặn thì thông báo thường là _"Request timed out"_ (Gửi đi được nhưng không thấy trả lời). Còn _"Unreachable"_ từ chính máy mình trả về nghĩa là **gửi đi không được**.
        
2. **Từ máy ảo (VM) ping máy thật (Host):**
    
    - **Ảnh:** `75% packet loss`, `time=2112 ms`.
        
    - **Ý nghĩa:** Gói tin bị rớt thảm hại, độ trễ cực cao (2 giây mới tới nơi).
        
    - **Suy luận:** Kết nối vật lý ảo (Virtual Bridge) đang cực kỳ thiếu ổn định hoặc bị xung đột.
        

=> **Kết luận:** Vấn đề nằm ở cấu hình **VMware Network Editor** (cách VMware cầu nối với card mạng thật), chứ không đơn thuần là Firewall nữa.

**Lỗi gặp phải:**

1. Host ping VM: `Destination host unreachable`.
    
2. VM ping Host: Packet loss 75%, latency cao (>2000ms).
    

**Nguyên nhân gốc (Root Cause):**

- Lỗi cấu hình **VMnet0 (Bridged)** trong VMware.
    
- Mặc định VMware để chế độ **"Automatic"**, nó có thể tự động bắt cầu nhầm vào một card mạng ảo khác (như VirtualBox Host-Only Adapter) hoặc card Bluetooth thay vì card Wifi/LAN chính, dẫn đến việc không tìm thấy nhau (ARP Fail) hoặc kết nối chập chờn.
    

**Giải pháp (Fix):**

1. Mở **Virtual Network Editor** (Chạy quyền Administrator).
    ![](Pasted%20image%2020260114094637.png)
    
2. Chọn Change Settings --> Chọn "Yes"
	![](Pasted%20image%2020260114094732.png)
	
3. Chọn **VMnet0** (Type: Bridged).
    
4. Tại mục **Bridged to**: Đổi từ _Automatic_ sang **Tên card mạng cụ thể** đang dùng (Ví dụ: _Intel(R) Wi-Fi 6 AX200_ hoặc _Realtek PCIe GBE..._).
    ![](Pasted%20image%2020260114094911.png)
_Để chọn đúng card mạng để kết nối chuẩn thì thao tác như sau:
Nhấn `Windows + R`, gõ `ncpa.cpl` -> Enter.
Nhìn xem biểu tượng nào **đang sáng không có dấu X đỏ** và có tên là **Wi-Fi**.
Nhìn tên thiết bị (Device Name) ở dòng chữ xám bên dưới. (Ví dụ nó sẽ ghi là _Intel(R) Wi-Fi 6E AX211 160MHz_)._
![](Pasted%20image%2020260114095044.png)

![](Pasted%20image%2020260114095058.png)
1. Restart lại card mạng trên máy ảo hoặc reboot máy ảo.

=> Thành quả:

```
  2026-01-14   09:58.49   /home/mobaxterm  ping 192.168.2.50

Pinging 192.168.2.50 with 32 bytes of data:
Reply from 192.168.2.50: bytes=32 time<1ms TTL=64
Reply from 192.168.2.50: bytes=32 time<1ms TTL=64
Reply from 192.168.2.50: bytes=32 time<1ms TTL=64
Reply from 192.168.2.50: bytes=32 time<1ms TTL=64

Ping statistics for 192.168.2.50:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
                                                                                       ✓

  2026-01-14   09:59.23   /home/mobaxterm 

```

![](Pasted%20image%2020260114095910.png)
## 4. Note nhanh / Từ khóa quan trọng

- ncpa.cpl
- Bridge
- Card mạng
- firewall
- private/public network
  

