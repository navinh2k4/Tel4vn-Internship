# [Week 3] Troubleshooting: Fix lỗi giao diện CDR không hiển thị dữ liệu

## 1. Mô tả vấn đề 

**Hiện tượng:**

- Mặc dù hệ thống đã ghi nhận cuộc gọi (kiểm tra trong database `v_xml_cdr` có dữ liệu), nhưng giao diện **Apps -> Call Detail Records** vẫn hiển thị trắng trang hoặc không có dữ liệu.

<div align="center">
  <img src="_assets/Pasted%20image%2020260115172225.png" width="100%">
  <p><i>Hình 1: Giao diện CDR bị lỗi trắng trang ban đầu</i></p>
</div>

<br>

- Đã thử gọi test nhiều cuộc nhưng UI vẫn không cập nhật.

**Minh chứng lỗi:** _Giao diện CDR trống trơn:_

<div align="center">
  <img src="_assets/Pasted%20image%2020260115173604.png" width="100%">
  <p><i>Hình 2: Sau khi gọi điện test thì giao diện CDR vẫn bị lỗi trắng như ban đầu</i></p>
</div>

<br>

_Trong khi Database vẫn có dữ liệu:_ `select * from v_xml_cdr;`

<div align="center">
  <img src="_assets/Pasted%20image%2020260115173415.png" width="100%">
  <p><i>Hình 3: Trước khi gọi điện test thì trong bảng v_xml_cdr có 3 bản ghi</i></p>
</div>

<br>

<div align="center">
  <img src="_assets/Pasted%20image%2020260115173639.png" width="100%">
  <p><i>Hình 4: Sau khi gọi điện test thì trong bảng v_xml_cdr có 4 bản ghi</i></p>
</div>

<br>

## 2. Quy trình phân tích 

### Bước 1: Kiểm tra Web Server (Nginx Logs)

Kiểm tra xem request tới file `xml_cdr.php` có được xử lý không. File log: `/var/log/nginx/access.log`

```
tail -f /var/log/nginx/access.log
```

**Kết quả:**

<div align="center">
  <img src="_assets/Pasted%20image%2020260120215336.png" width="100%">
  <p><i>Hình 5: Log Nginx hiện thị theo thời gian thực</i></p>
</div>

<br>

<div align="center">
  <img src="_assets/Pasted%20image%2020260120215257.png" width="100%">
  <p><i>Hình 6: Chi tiết rõ hơn log Nginx cho thấy dịch vụ hoạt động ổn định</i></p>
</div>

<br>

> **Nhận định:** HTTP Status trả về **200 OK**. Chứng tỏ Web Server hoạt động bình thường, Code PHP đã được thực thi. Vấn đề nằm ở tầng ứng dụng hoặc Database.

### Bước 2: Kiểm tra Database Logs (Postgres Logs)

Đây là bước quan trọng để xem query bị lỗi gì. File log: `/var/log/postgresql/postgresql-12-main.log`

```
tail -f /var/log/postgresql/postgresql-12-main.log
```

**Kết quả (Phát hiện lỗi):**

```
2026-01-20 21:55:08.199 +07 [5617] fusionpbx@fusionpbx ERROR:  relation "call_rate" does not exist at character 1314
2026-01-20 21:55:08.199 +07 [5617] fusionpbx@fusionpbx STATEMENT:  select
        c.domain_uuid,
        CASE WHEN cr.keypress IS NULL THEN '0' ELSE cr.keypress END AS callrate,
        e.extension,
        c.start_stamp,
        c.end_stamp,
        c.start_epoch,
        c.hangup_cause,
        c.duration,
        c.billmsec,
        c.record_path,
        c.record_name,
        c.xml_cdr_uuid,
        c.bridge_uuid,
        c.direction,
        c.billsec,
        c.caller_id_name,
        c.caller_id_number,
        c.caller_destination,
        c.source_number,
        c.destination_number,
        c.leg,
        (c.xml IS NOT NULL OR c.json IS NOT NULL) AS raw_data_exists,
        c.accountcode,
        c.answer_stamp,
        c.sip_hangup_disposition,
        c.pdd_ms,
        c.rtp_audio_in_mos,
        (c.answer_epoch - c.start_epoch) as tta , CASE
                                                                WHEN (answer_stamp is not null and bridge_uuid is not null) THEN 'ANSWERED'
                                                                WHEN (answer_stamp is not null and bridge_uuid is null) THEN 'NO ANSWER'

                        WHEN
                        ((
                                (direction = 'inbound' or direction = 'local')
                                        and answer_stamp is null
                                        and bridge_uuid is null
                                        and sip_hangup_disposition <> 'send_refuse'
                        )
                        or (
                                direction = 'outbound'
                                and answer_stamp is null
                                and bridge_uuid is not null
                        )) THEN 'BUSY' WHEN (answer_stamp is null and bridge_uuid is null and billsec = 0 and sip_hangup_disposition = 'send_refuse') THEN 'FAILED' ELSE 'UNKNOWN' END AS Status
        from v_xml_cdr as c
        left join v_extensions as e on e.extension_uuid = c.extension_uuid left join call_rate cr on c.domain_uuid = cr.domain_uuid and c.xml_cdr_uuid = cr.call_id inner join v_domains as d on d.domain_uuid = c.domain_uuid where c.domain_uuid = '05bb58df-da3a-42cf-bb1f-b69922c3dbd3'
         and start_stamp BETWEEN '2026-01-20 ' AND '2026-01-20 23:59' order by start_epoch desc limit 100 offset 0

```

<div align="center">
  <img src="_assets/Pasted%20image%2020260120215714.png" width="100%">
  <p><i>Hình 7: Log Postgres xuất hiện lỗi theo thời gian thực (Khi thao tác truy cập App --> Call Detail Records trên web)</i></p>
</div>

<br>

## 3. Nguyên nhân gốc rễ 

- Mã nguồn PHP của trang CDR thực hiện câu lệnh SQL có `LEFT JOIN` đến bảng **`call_rate`**.
    
- Tuy nhiên, trong Database hiện tại **chưa tồn tại bảng `call_rate`**.
    
- Do thiếu bảng này, câu truy vấn SQL bị lỗi (Fatal Error), dẫn đến việc không lấy được dữ liệu hiển thị ra UI.

## 4. Giải pháp 

Tiến hành tạo bảng `call_rate` còn thiếu. Cập nhật Schema chuẩn theo hệ thống thực tế 

**Thực hiện:**

```
su - postgres
psql -d fusionpbx
```

**Câu lệnh SQL:**

```
# Create table
CREATE TABLE call_rate (
    customer_num character varying(30),
    domain_uuid uuid,
    call_id uuid,
    sip_call_id text,
    keypress character(30),
    created_time timestamp DEFAULT now()
);

# Create index
CREATE INDEX idx_call_rate_call_id ON call_rate (call_id);
```

<div align="center">
  <img src="_assets/Pasted%20image%2020260120220133.png" width="100%">
  <p><i>Hình 8: Tạo bảng call_rate và index trong bảng</i></p>
</div>

<br>

**Kiểm tra lại cấu trúc bảng:**
```
\dt call_rate
```

_Kết quả:_

<div align="center">
  <img src="_assets/Pasted%20image%2020260120220957.png" width="100%">
  <p><i>Hình 9: Kết quả tạo bảng call_rate thành công</i></p>
</div>

<br>

## 5. Kết quả 

Sau khi tạo bảng xong, Refresh lại trang CDR. Hệ thống đã hiển thị đầy đủ lịch sử cuộc gọi.

**Giao diện hiển thị thành công:**

<div align="center">
  <img src="_assets/Pasted%20image%2020260120220254.png" width="100%">
  <p><i>Hình 10: Kết quả UI đã hiển thị thông tin lịch sử cuộc gọi</i></p>
</div>

<br>

_Chi tiết cuộc gọi:_

<div align="center">
  <img src="_assets/Pasted%20image%2020260120220352.png" width="100%">
  <p><i>Hình 11: Thông tin chi tiết lịch sử cuộc gọi</i></p>
</div>

<br>
