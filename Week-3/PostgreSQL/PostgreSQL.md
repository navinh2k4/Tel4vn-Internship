# 1. PostgreSQL là gì và Tại sao FusionPBX cần nó?
Hãy tưởng tượng sự khác biệt giữa **SQLite** (cũ) và **PostgreSQL** (mới) như sau:

- **SQLite:** Giống như một file **Excel** trên máy tính bạn. Nó gọn nhẹ, nhưng chỉ **một người** được ghi vào đó tại một thời điểm. Nếu 100 cuộc gọi ập đến cùng lúc, chúng phải xếp hàng chờ ghi log -> Nghẽn cổ chai (Locking issue).
    ![](Pasted%20image%2020260114081943.png)
    
- **PostgreSQL:** Giống như một **Nhà kho khổng lồ** có hàng nghìn cửa nhập hàng và nhân viên bốc xếp. Nó cho phép hàng nghìn cuộc gọi (Concurrent connections) ghi/đọc dữ liệu cùng lúc mà không ai phải chờ ai.
    ![](Pasted%20image%2020260114082044.png)
    
**=> Đó là lý do hệ thống lớn (Production) bắt buộc phải dùng Postgres.**

# 2. Kiến trúc PostgreSQL trên Debian 12 
Khi bạn cài đặt trên Debian, PostgreSQL chia làm 3 thành phần chính mà bạn phải nhớ vị trí:

## A. File cấu hình 
Nằm tại: `/etc/postgresql/12/main/`

![](Pasted%20image%2020260114082629.png)

=> Có 2 file quan trọng nhất:

### 1.**`postgresql.conf` (Trái tim):
- Quy định DB chạy port nào (mặc định 5432)
- Lắng nghe ai (listen_addresses): Mặc định là localhost. Nếu muốn DB server và VoIP server nằm trên 2 máy khác nhau, bạn phải sửa dòng này thành [*].
- Giới hạn kết nối (max_connections): 

### 2.**`pg_hba.conf` (Bảo vệ):**

- Đây là "người gác cổng" (Firewall của DB).
- Nó quyết định: _User nào_ - _Từ IP nào_ - _Được vào DB nào_ - _Dùng phương thức xác thực gì?_
- **Lỗi kinh điển:** FreeSWITCH kết nối bị từ chối (`Connection refused` hoặc `Ident authentication failed`) 99% là do cấu hình sai file này.

## B. Dữ liệu 

Nằm tại: `/var/lib/postgresql/12/main/` 

- Đây là nơi chứa toàn bộ data cuộc gọi, user, password...
    
- **Lưu ý DevOps:** Không bao giờ xóa hoặc can thiệp trực tiếp vào thư mục này bằng tay. Chỉ dùng lệnh SQL để thao tác.
    

## C. Log hệ thống (Nơi bắt bệnh)

Nằm tại: `/var/log/postgresql/`

- Khi FreeSWITCH không ghi được CDR, hoặc Web không hiện dữ liệu, đây là nơi đầu tiên bạn phải mở ra xem (`tail -f`).
    

# 3. Cheat Sheet: Các lệnh sinh tồn với PostgreSQL

Trong môi trường Linux (CLI), bạn không có giao diện đồ họa. Bạn phải dùng công cụ dòng lệnh tên là `psql`.

## Bước 1: Đăng nhập (Superuser)

Postgres có một user tối cao tên là `postgres`. Để vào DB, bạn phải "nhập vai" user này:

```
Bash

# Cách 1: Vào thẳng giao diện dòng lệnh SQL
sudo -u postgres psql

# Cách 2: Chuyển sang user linux rồi mới vào SQL (dùng khi cần debug lâu)
sudo -i -u postgres
psql
```

## Bước 2: Các lệnh "Meta-Commands" (Bắt đầu bằng dấu gạch chéo `\`)

Khi màn hình hiện `postgres=#`, bạn dùng các lệnh sau:

|**Lệnh**|**Ý nghĩa**|**Tương đương trong lời nói**|
|---|---|---|
|**`\l`**|**L**ist Databases|"Cho tôi xem có bao nhiêu cái kho dữ liệu?"|
|**`\c <tên_db>`**|**C**onnect|"Cho tôi đi vào cái kho tên là `fusionpbx`."|
|**`\dt`**|**D**isplay **T**ables|"Trong kho này có những bảng nào?"|
|**`\d <tên_bảng>`**|**D**escribe|"Cho tôi xem cấu trúc chi tiết của bảng `v_xml_cdr`."|
|**`\du`**|**D**isplay **U**sers|"Liệt kê danh sách user (Role) và quyền hạn."|
|**`\q`**|**Q**uit|"Thoát ra."|

## Bước 3: Các lệnh SQL cơ bản (Truy vấn dữ liệu)

Giống như MySQL hay SQL Server, chuẩn SQL vẫn áp dụng:

- `SELECT * FROM v_xml_cdr LIMIT 5;` (Xem 5 cuộc gọi gần nhất).
    
- `DELETE FROM v_xml_cdr;` (Xóa sạch lịch sử cuộc gọi - **Cẩn thận!**).
    

---

# 4. Áp dụng vào việc Fix lỗi CDR (Tư duy phân tích)

Quay lại bài toán: **"Tại sao log CDR không hiển thị trên UI?"**

Dựa trên kiến thức vừa học, bạn sẽ hình dung luồng dữ liệu (Data Flow) như sau:

1. **Cuộc gọi kết thúc** -> FreeSWITCH (Core) tạo ra thông tin gói tin.
    
2. **FreeSWITCH kết nối DB** -> Nó dùng module `mod_pgsql` hoặc `mod_odbc` để gõ cửa PostgreSQL qua port 5432.
    
3. **Xác thực** -> PostgreSQL check file `pg_hba.conf`. Có cho phép IP `127.0.0.1` user `fusionpbx` vào không?
    
4. **Ghi dữ liệu** -> Nếu vào được, FreeSWITCH chạy lệnh `INSERT INTO v_xml_cdr...`.
    
5. **Hiển thị** -> Web (PHP) chạy lệnh `SELECT * FROM v_xml_cdr` để hiện lên màn hình.
    

**=> Vậy lỗi có thể nằm ở đâu?**

1. **Lỗi bước 2:** FreeSWITCH chưa load module kết nối (Hôm qua bạn đã fix cái này).
    
2. **Lỗi bước 3:** Sai mật khẩu trong file cấu hình XML, hoặc bị chặn bởi `pg_hba.conf`.
    
3. **Lỗi bước 4:** Bảng `v_xml_cdr` bị thiếu cột, hoặc sai kiểu dữ liệu (Schema mismatch).
    
4. **Lỗi bước 5:** DB có dữ liệu, nhưng Web bị lỗi PHP không lấy ra được.
    

---

# Bài tập nhỏ cho bạn (Trước khi fix lỗi thật)

Hãy mở Terminal server của bạn lên và thực hành ngay các lệnh sau để "cảm nhận" hệ thống:

1. Đăng nhập vào DB: `sudo -u postgres psql`
    
2. Kết nối vào DB của FusionPBX: `\c fusionpbx`
    
3. Xem cấu trúc bảng log cuộc gọi: `\d v_xml_cdr`
    
    - _Nhiệm vụ:_ Hãy nhìn xem có bao nhiêu cột? Cột nào là khóa chính (Primary Key)?
        
4. Thử select dữ liệu: `SELECT count(*) FROM v_xml_cdr;`
    
    - _Nhiệm vụ:_ Xem kết quả là 0 hay là số khác?
        

Khi bạn chạy xong các lệnh trên, hãy báo lại kết quả. Chúng ta sẽ biết ngay lỗi CDR nằm ở bước nào trong 5 bước trên.