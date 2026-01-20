### 1. Quy trình tổng quan (3 giai đoạn)

Để có được workflow SIP từ server về máy tính, luôn có 3 bước:

1. **Capture (Bắt gói tin):** Dùng công cụ thu thập dữ liệu đi qua card mạng server.
    
2. **Transport/Storage (Lưu & Chuyển):** Lưu thành file `.pcap` và chuyển về máy tính cá nhân.
    
3. **Analyze (Phân tích):** Dựng lại kịch bản cuộc gọi (Flow) để xem lỗi ở đâu.

### 2. Các công cụ cần thiết (Toolkit)

Bạn sẽ cần combo 3 công cụ sau phối hợp với nhau:

1. **tcpdump (Trên Server):** Công cụ dòng lệnh kinh điển, có sẵn trên mọi Linux. Chuyên dùng để "dump" (đổ) dữ liệu mạng ra file.
    
2. **sngrep (Trên Server):** Công cụ hiện đại, giao diện trực quan ngay trên terminal. Vừa xem được trực tiếp, vừa xuất được file pcap.
    
3. **Wireshark (Trên Laptop):** Trình đọc file pcap mạnh nhất thế giới. Dùng để "mổ xẻ" file mà bạn đã bắt được từ server.

### 3. Các phương pháp thực hiện (3 Methods)

Dưới đây là 3 cách phổ biến nhất, từ đơn giản đến chuyên sâu:

#### Phương pháp 1: Dùng `sngrep` (Nhanh, Tiện, Trực quan nhất)

Đây là cách các DevOps hiện đại thường dùng nhất vì **vừa xem được ngay, vừa lưu được file**.

- **Cách làm:**
    
    1. Gõ lệnh `sngrep` trên server.
        ![](Pasted%20image%2020260113092701.png)
        
        ![](Pasted%20image%2020260113092710.png)
        
    1. Thực hiện cuộc gọi test.
        ![](Pasted%20image%2020260113092738.png)
        
    2. Thấy cuộc gọi xuất hiện, nhấn `Enter` để xem flow.
        ![](Pasted%20image%2020260113092803.png)
        
        ![](Pasted%20image%2020260113092815.png)
        
        ![](Pasted%20image%2020260113092910.png)
        
    1. Nếu muốn lưu file: Nhấn `F2` (Save), chọn đường dẫn lưu file `.pcap`.
        ![](Pasted%20image%2020260113100508.png)
        
        ![](Pasted%20image%2020260113100519.png)
        
    1. Dùng MobaXterm (SFTP) kéo file đó về máy tính mở bằng Wireshark.
        ![](Pasted%20image%2020260113100537.png)
        
![](Pasted%20image%2020260113100622.png)

![](Pasted%20image%2020260113100645.png)

![](Pasted%20image%2020260113100658.png)

![](Pasted%20image%2020260113100707.png)

![](Pasted%20image%2020260113101010.png)

![](Pasted%20image%2020260113101019.png)

### 1. Xác định vai trò (Who is Who?)

Nhìn vào Port (cổng) để biết ai là Server, ai là Client:

- **192.168.1.50 (Port 5060):** Đây là **SIP Server** (FusionPBX của bạn). Cổng 5060 là cổng mặc định của SIP.
    
- **192.168.1.40 (Port 44515):** Đây là **Client** (Softphone/IP Phone). Nó dùng một cổng ngẫu nhiên (Ephemeral port) để giao tiếp.

### 2. Phân tích Chi tiết luồng đi (The Story of a Packet)

Cuộc gọi này chia làm 3 giai đoạn chính:

#### Giai đoạn 1: Bắt tay và Xác thực (Authentication) - _Đây là chỗ Newbie hay hoang mang nhất_

Bạn thấy có **2 dòng INVITE**? Tại sao mời 2 lần?

1. **INVITE (đầu tiên):** Client (.40) nói: _"Alo server, tôi muốn gọi."_
    
2. **407 Proxy Authentication Required:** Server (.50) chặn lại: _"Khoan, bạn chưa chứng minh danh tính. Cần mật khẩu!"_ (Đây **không phải lỗi**, đây là tính năng bảo mật).
    
3. **ACK:** Client: _"Ok, đã hiểu."_
    
4. **INVITE (thứ hai):** Client gửi lại lời mời, nhưng lần này trong gói tin đã kèm theo **Authorization Header** (chứa mã hóa user/pass).
    

=> **Bài học:** Nếu bạn chỉ thấy dòng INVITE đầu tiên mà không thấy dòng INVITE thứ 2 => **Sai mật khẩu hoặc sai cấu hình Extension.**


#### Phương pháp 2: Dùng `tcpdump` (Truyền thống, Chính xác nhất)

Dùng khi bạn muốn bắt gói tin trong thời gian dài hoặc server không cài được sngrep.

Cài đặt tcpdump
```
apt-get install tcpdump
```
![](Pasted%20image%2020260113094821.png)
- **Cách làm:**
    
    1. Chạy lệnh: `tcpdump -i any -n -s 0 port 5060 -w capture.pcap`
        
        - `-i any`: Bắt trên mọi card mạng.
            
        - `port 5060`: Chỉ bắt gói tin SIP (nếu muốn bắt cả RTP thoại thì bỏ đoạn này đi, nhưng file sẽ rất nặng).
            
        - `-w capture.pcap`: Ghi dữ liệu vào file tên là capture.pcap.
        ![](Pasted%20image%2020260113094849.png)
        
    1. Thực hiện cuộc gọi.
        ![](Pasted%20image%2020260113094914.png)
        
    2. Nhấn `Ctrl + C` để dừng bắt.
        ![](Pasted%20image%2020260113094932.png)
        
    3. Dùng MobaXterm kéo file `capture.pcap` về máy tính.
        ![](Pasted%20image%2020260113094953.png)
        
        ![](Pasted%20image%2020260113095027.png)

![](Pasted%20image%2020260113095100.png)

![](Pasted%20image%2020260113095110.png)

![](Pasted%20image%2020260113095124.png)

![](Pasted%20image%2020260113095135.png)

#### Phương pháp 3: Bắt tại Client (Trên máy tính cá nhân)

Dùng Wireshark cài trực tiếp trên máy tính của bạn.
![](Pasted%20image%2020260113095241.png)

- **Cách làm:** Mở Wireshark -> Chọn card mạng (Wifi/LAN) -> Filter `sip` -> Gọi bằng Softphone trên máy tính.
    ![](Pasted%20image%2020260113095300.png)
    
- **Hạn chế:** Chỉ bắt được những gì đi ra/vào máy tính của bạn, không thấy được những gì server xử lý nội bộ hoặc gửi cho nhà mạng (Carrier).
- ![](Pasted%20image%2020260113095318.png)
- ![](Pasted%20image%2020260113095339.png)
- ![](Pasted%20image%2020260113095353.png)
- ![](Pasted%20image%2020260113095438.png)

### 4. Tối ưu: Chọn phương án nào trong trường hợp nào?

Đây là bảng ma trận quyết định dành cho bạn:

|**Trường hợp (Scenario)**|**Phương án tối ưu**|**Lý do**|
|---|---|---|
|**Debug nhanh hàng ngày**<br><br>  <br><br>(VD: Extension A không gọi được Extension B, báo lỗi Busy)|**`sngrep` (Xem trực tiếp)**|Không cần tải file về máy. Nhìn thấy ngay flow 404, 486, 401 trên màn hình server.|
|**Báo cáo lỗi cho sếp/khách hàng**<br><br>  <br><br>(Cần bằng chứng cụ thể)|**`sngrep` (Xuất file pcap)**|Vừa confirm nhanh được lỗi, vừa có file gửi đi ngay.|
|**Debug lỗi âm thanh (RTP)**<br><br>  <br><br>(Mất tiếng, thoại 1 chiều, tiếng rè)|**`tcpdump` (Full)**|`sngrep` đôi khi không capture đủ RTP. `tcpdump` bắt trọn vẹn mọi thứ để Wireshark phân tích được cả sóng âm (Play stream).|
|**Server bị treo/quá tải**|**`tcpdump`**|`sngrep` có giao diện đồ họa nên tốn tài nguyên hơn. `tcpdump` rất nhẹ, chạy ngầm tốt.|

---

### 5. Cách xem Workflow (Flow Sequence) trên Wireshark

Khi đã có file `.pcap` và mở bằng Wireshark, bạn **đừng đọc từng dòng**, sẽ rất rối. Hãy làm như sau để xem Workflow:

1. Trên thanh menu, chọn **Telephony** -> **VoIP Calls**.
    
2. Một danh sách các cuộc gọi sẽ hiện ra. Chọn cuộc gọi bạn quan tâm.
    
3. Bấm nút **Flow Sequence**.
    

**Kết quả:** Bạn sẽ thấy một **Biểu đồ hình thang (Ladder Diagram)**.

- Trục dọc là thời gian.
    
- Các đường ngang là bản tin SIP (INVITE, TRYING, RINGING, OK, ACK...).
    
- Nhìn vào đây bạn sẽ biết ngay **ai ngắt máy trước**, hoặc **gói tin bị chặn ở đâu**.
    


### 1. Xác định vai trò (Who is Who?)

Nhìn vào Port (cổng) để biết ai là Server, ai là Client:

- **192.168.1.50 (Port 5060):** Đây là **SIP Server** (FusionPBX của bạn). Cổng 5060 là cổng mặc định của SIP.
    
- **192.168.1.40 (Port 44515):** Đây là **Client** (Softphone/IP Phone). Nó dùng một cổng ngẫu nhiên (Ephemeral port) để giao tiếp.
    

---

### 2. Phân tích Chi tiết luồng đi (The Story of a Packet)

Cuộc gọi này chia làm 3 giai đoạn chính:

#### Giai đoạn 1: Bắt tay và Xác thực (Authentication) - _Đây là chỗ Newbie hay hoang mang nhất_

Bạn thấy có **2 dòng INVITE**? Tại sao mời 2 lần?

1. **INVITE (đầu tiên):** Client (.40) nói: _"Alo server, tôi muốn gọi."_
    
2. **407 Proxy Authentication Required:** Server (.50) chặn lại: _"Khoan, bạn chưa chứng minh danh tính. Cần mật khẩu!"_ (Đây **không phải lỗi**, đây là tính năng bảo mật).
    
3. **ACK:** Client: _"Ok, đã hiểu."_
    
4. **INVITE (thứ hai):** Client gửi lại lời mời, nhưng lần này trong gói tin đã kèm theo **Authorization Header** (chứa mã hóa user/pass).
    

=> **Bài học:** Nếu bạn chỉ thấy dòng INVITE đầu tiên mà không thấy dòng INVITE thứ 2 => **Sai mật khẩu hoặc sai cấu hình Extension.**

#### Giai đoạn 2: Thiết lập cuộc gọi (Call Setup)

Sau khi Server chấp nhận INVITE thứ 2:

5. **100 Trying:** Server: _"Đang xử lý nhé, chờ chút."_ (Để Client đỡ sốt ruột gửi lại).
    
6. **183 Session Progress:** Server: _"Đang kết nối luồng thoại/nhạc chờ đây."_
    
    - _Lưu ý:_ Thường bạn sẽ thấy **180 Ringing** (Máy đổ chuông). Nhưng ở đây là **183** (Early Media) - nghĩa là server đã mở luồng âm thanh sớm để phát nhạc chờ hoặc thông báo từ tổng đài trước khi bên kia nhấc máy.
        
7. **200 OK:** Server: _"Bên kia nhấc máy rồi!"_ (Cuộc gọi chính thức bắt đầu tính cước từ giây này).
    
8. **ACK:** Client: _"Xác nhận đã kết nối."_
    

=> **Lúc này (giữa dòng ACK và BYE):** Là lúc hai người nói chuyện. Gói tin **RTP (Real-time Transport Protocol)** chứa giọng nói sẽ chạy qua lại, nhưng biểu đồ này chỉ hiện SIP (tín hiệu) nên bạn không thấy RTP ở đây.

#### Giai đoạn 3: Kết thúc (Teardown)

9. **BYE:** Client (.40) nói: _"Thôi tôi tắt máy đây."_ (Người gọi dập máy trước).
    
10. **200 OK:** Server (.50): _"Ok, đã ngắt kết nối."_
    

---

### 3. Những từ khóa kỹ thuật cần soi (Technical Keywords)

Trong cột "Comment" bên phải, hãy chú ý những thứ này:

- **SDP (Session Description Protocol):** Đi kèm với INVITE và 200 OK.
    
    - Nó chứa thông tin: _"Tôi dùng Codec gì (G711, G729, Opus...)? Tôi nhận âm thanh ở IP nào, Port nào?"_
        
    - **Lỗi thường gặp:** Nếu 2 bên không nghe thấy nhau (One-way audio), lỗi 90% nằm ở việc đàm phán SDP này không khớp (ví dụ: Codec mismatch).
        
- **telephone-event:** Đây là chuẩn để gửi phím bấm (DTMF). Ví dụ khi gọi tổng đài bảo "Bấm phím 1 để gặp kỹ thuật", nó dùng cái này.
    
- **transport=TCP:** Mình thấy trong gói tin INVITE của bạn có dòng này.
    
    - Mặc định SIP dùng **UDP**.
        
    - Hệ thống của bạn đang dùng **TCP** (có thể do cấu hình trên Softphone hoặc Server). Đây là điểm cần lưu ý nếu sau này gặp lỗi chập chờn, hãy thử chuyển về UDP xem sao (UDP nhanh hơn, TCP tin cậy hơn nhưng chậm hơn).
        

---

### 4. Kết luận & Hành động tiếp theo

**Bạn đã làm rất tốt.** File pcap này cho thấy hệ thống Lab của bạn đang hoạt động **hoàn hảo** về mặt tín hiệu (Signaling).

**Task tiếp theo cho bạn (để kiểm tra phần "Máu" - RTP):**

1. Vẫn trên Wireshark này, bạn tắt cửa sổ "Call Flow" đi.
    
2. Trên màn hình chính danh sách gói tin, tìm các gói tin có Protocol là **RTP**.
    
3. Vào menu **Telephony -> RTP -> RTP Player**.
    
4. Thử bấm **Play** xem có nghe thấy giọng nói của chính cuộc gọi đó không?
    
    - _Nếu nghe được:_ Chúc mừng, bạn đã nắm trọn vẹn luồng đi.
        
    - _Nếu không nghe được/không thấy RTP:_ Có thể do bạn chỉ capture port 5060 (chỉ bắt tín hiệu) mà quên bắt dải port RTP (thường là 16384-32768).
        

Bạn hãy thử check phần RTP Player nhé, tính năng đó cực hay!

![](Pasted%20image%2020260113135554.png)

