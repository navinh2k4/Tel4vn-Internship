
# [Topic]: Troubleshooting SSH Timeout (Debian 12)
**Ngày:** 2026-01-14
**Nguồn:** [Link tài liệu/Video]
**Tags:** #ssh #troubleshooting #debian #firewall #iptables #ufw #debian

---

## 1. Hiểu nhanh (Feynman Zone)
> *Dừng lại 30s. Viết lại khái niệm này theo cách mình hiểu (như đang giải thích cho người không biết gì).*

- **Khái niệm này là gì?:** ...
- Vấn đề là gì?: Đường đi (ping) đã thông, nhưng gõ cửa (SSH Port 22) thì không ai trả lời hoặc bị chặn.
- **Tại sao?:** Có 2 khả năng chính:
	- 1. Dịch vụ SSH Server trên máy ảo chưa chạy (Người nhà đi vắng).
	- 2. Dịch vụ chạy rồi nhưng Firewall trên máy ảo đang chặn cổng 22 (cửa khóa).
- **Nó khác gì cái cũ?:** ...

---

## 2. Thực hành (Hands-on Lab)

### A. Chuẩn bị (Prediction)
*Trước khi gõ lệnh, mình mong đợi điều gì xảy ra?*
- [ ] Dự đoán: SSH Server chưa bật hoặc bị Firewall chặn.

### B. Thực thi (Execution)
**Bước 1: Kiểm tra SSH Server đã chạy chưa trên máy ảo (Debian)** _Nhìn vào ảnh `netstat -tlunp` bạn gửi, tôi thấy cổng 22 chưa hề xuất hiện trong danh sách LISTEN._
```
# Dán lệnh vào đây

# Trên máy ảo Debian, gõ lệnh kiểm tra trạng thái:
systemctl status ssh

# Nếu kết quả là `Active: inactive (dead)` hoặc `Unit ssh.service could not be found`, nghĩa là chưa cài hoặc chưa bật.
```


```
# Dán kết quả trả về (Log) vào đây

```
![](Pasted%20image%2020260114100801.png)

![](Pasted%20image%2020260114100856.png)
### C. Phân tích (Analysis)

_Kết quả có giống dự đoán không?_

- [ ] Giống.
    
- [x] Khác -> Tại sao? ...
     Vì sau khi kiểm tra trạng thái dịch vụ SSH trên Server thì nó vẫn hoạt động bình thường chứ không dead hay chưa cài đặt nhưng khi SSH vào thì vẫn không thành công và báo time out.

## 1. Hiểu nhanh (Feynman Zone)

- **Hiện tượng:** Nhà đã mở cửa (Service Run), đường đi đã thông (Ping OK), nhưng gõ cửa (SSH) thì không ai trả lời.
    
- **Tại sao?:** Có một ông bảo vệ (Firewall) đứng trước cửa nhà chặn lại. Trên Debian, ông bảo vệ này thường tên là `ufw` (Uncomplicated Firewall) hoặc `nftables`.
    
- **Giải pháp:** Ra lệnh cho bảo vệ cho phép khách vào cổng 22.
    

---

## 2. Thực hành (Hands-on Lab)

### A. Chuẩn bị (Prediction)

- [ ] Dự đoán: Sau khi tắt Firewall hoặc mở port 22, kết nối SSH sẽ thành công ngay lập tức.
    

### B. Thực thi (Execution)

**Bước 1: Kiểm tra xem Debian có đang bật UFW không** _(UFW là tường lửa phổ biến nhất trên Debian/Ubuntu)_

Bash

```
# Chạy trên máy ảo Debian
sudo ufw status
```

- **Trường hợp 1:** Nếu báo `Status: active` -> **Đây chính là thủ phạm.**
    
- **Trường hợp 2:** Nếu báo `command not found` hoặc `Status: inactive` -> Chuyển sang Bước 3 (Check iptables/nftables).
    

**Bước 2: Mở port SSH (Nếu UFW đang active)**

Bash

```
# Cho phép SSH
sudo ufw allow ssh
# Hoặc mở cụ thể port 22
sudo ufw allow 22/tcp
# Kiểm tra lại
sudo ufw status
```

**Bước 3: Xử lý mạnh tay (Flush Iptables - Nếu không dùng UFW)** _Nếu UFW không cài, có thể `iptables` hoặc `nftables` đang chặn ngầm. Lệnh sau sẽ xóa sạch các luật chặn tạm thời để test._

Bash

```
# Xóa sạch các luật chặn (Flush)
sudo iptables -F
# Cho phép mọi kết nối (Accept All) - Chỉ dùng để test Lab
sudo iptables -P INPUT ACCEPT
```
![](Pasted%20image%2020260114101532.png)

=> Thử dùng lệnh kiểm tra trạng thái tường lửa nhưng đều báo `command not found`

Vậy nên chuyển hướng sang nguyên nhân khác: 
### Bước 1: Kiểm tra "Xung đột IP" (Nghi vấn số 1)

Hiện tượng "Ping thông nhưng SSH không được" rất hay xảy ra khi có 2 thiết bị trong mạng LAN cùng tranh nhau địa chỉ IP `192.168.2.50`.

- _Ví dụ:_ Máy ảo của bạn là `.50`, nhưng cái Tivi hoặc điện thoại của ai đó trong nhà cũng đang là `.50`. Khi bạn SSH, gói tin chạy nhầm sang cái Tivi -> Tivi không mở port 22 -> Timeout.
    

**Cách test:**

1. **Tắt máy ảo Debian đi** (Power Off).
    
2. Trên máy thật (Windows MobaXterm), chạy lệnh: `ping 192.168.2.50`
    
3. **Phân tích:**
    
    - Nếu vẫn thấy **Reply**: => **Chắc chắn trùng IP**. Có một "kẻ mạo danh" đang dùng IP này. Bạn cần bật máy ảo lên và đổi sang IP khác (ví dụ `.222`).
        
    - Nếu thấy **Request timed out**: => IP sạch, không trùng. Vấn đề nằm ở máy ảo. Chuyển sang Bước 2.

# Trời ơi thì sau khi fix xong các cách trên thì nên khởi động lại cả máy thật nữa nha, sau khi khởi động lại máy thật thì mọi thứ đã ok.

### C. Phân tích (Analysis) 
- [x] SSH thành công (Sau khi reboot máy thật). 

--- 
## 3. Nhật ký lỗi (Troubleshooting) 
**Giải pháp thực tế (Actual Fix):** 
1. Kiểm tra IP Conflict (Không trùng). 
2. Khởi động lại máy thật (Windows Host) để reset VMware Bridge Driver và ARP Table.