
# 1. Lỗi không hiển thị thông tin chi tiết CDR

![](_assets/Pasted%20image%2020260115172225.png)

![](_assets/Pasted%20image%2020260115173430.png)

`select * from v_xml_cdr`
![](_assets/Pasted%20image%2020260115173415.png)


# 2. Sau khi gọi test

![](_assets/Pasted%20image%2020260115173604.png)

`select * from v_xml_cdr`
![](_assets/Pasted%20image%2020260115173639.png)


# 3. Check Log Nginx

```
root@debian:~# ls /var/log/
alternatives.log  faillog         lastlog           popularity-contest      README                vmware-network.3.log  vmware-network.log       vmware-vmtoolsd-root.log
apt               fontconfig.log  nginx             popularity-contest.0    runit                 vmware-network.4.log  vmware-vmsvc-root.1.log  wtmp
btmp              freeswitch      php7.2            popularity-contest.gpg  sysstat               vmware-network.5.log  vmware-vmsvc-root.2.log
dpkg.log          installer       php7.2-fpm.log    postgresql              vmware-network.1.log  vmware-network.6.log  vmware-vmsvc-root.3.log
exim4             journal         php7.2-fpm.log.1  private                 vmware-network.2.log  vmware-network.7.log  vmware-vmsvc-root.log
root@debian:~# ls /var/log/nginx/
access.log  access.log.1  error.log  error.log.1
root@debian:~# tail -f /var/log/nginx/access.log
192.168.2.17 - - [15/Jan/2026:17:35:42 +0700] "GET /app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search HTTP/1.1" 200 49857 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:42 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:43 +0700] "GET /app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search HTTP/1.1" 200 49857 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:43 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:43 +0700] "GET /app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search HTTP/1.1" 200 49857 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:43 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:46 +0700] "GET /app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=2026-01-15+00%3A00&start_stamp_end=2026-01-15+00%3A00&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search HTTP/1.1" 200 50557 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=&start_stamp_end=&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:46 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=2026-01-15+00%3A00&start_stamp_end=2026-01-15+00%3A00&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:47 +0700] "GET /app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=2026-01-15+00%3A00&start_stamp_end=2026-01-15+00%3A00&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search HTTP/1.1" 200 50557 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=2026-01-15+00%3A00&start_stamp_end=2026-01-15+00%3A00&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.17 - - [15/Jan/2026:17:35:47 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://navinh.tel4vn.com/app/xml_cdr/xml_cdr.php?direction=&call_result=&caller_id_number=&destination_number=&start_stamp_begin=2026-01-15+00%3A00&start_stamp_end=2026-01-15+00%3A00&caller_id_name=&hangup_cause=&caller_destination=&caller_extension_uuid=&duration=&show=all&submit=Search" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
^C
root@debian:~# tail -f /var/log/nginx/access.log.1
192.168.2.16 - - [08/Jan/2026:17:01:27 +0700] "POST /app/call_flows/call_flow_edit.php?id=951f5c5a-057f-4f55-b962-cf05a8e3dfe2 HTTP/1.1" 302 11 "https://192.168.2.50/app/call_flows/call_flow_edit.php?id=951f5c5a-057f-4f55-b962-cf05a8e3dfe2" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.16 - - [08/Jan/2026:17:01:28 +0700] "GET /app/call_flows/call_flows.php HTTP/1.1" 200 30182 "https://192.168.2.50/app/call_flows/call_flow_edit.php?id=951f5c5a-057f-4f55-b962-cf05a8e3dfe2" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.16 - - [08/Jan/2026:17:01:28 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://192.168.2.50/app/call_flows/call_flows.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.16 - - [08/Jan/2026:17:02:05 +0700] "GET / HTTP/1.1" 302 5 "https://192.168.2.50/app/call_flows/call_flows.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.16 - - [08/Jan/2026:17:02:06 +0700] "GET /core/user_settings/user_dashboard.php HTTP/1.1" 200 43340 "https://192.168.2.50/app/call_flows/call_flows.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.16 - - [08/Jan/2026:17:02:06 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://192.168.2.50/core/user_settings/user_dashboard.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.16 - - [08/Jan/2026:17:02:37 +0700] "GET /app/xml_cdr/xml_cdr.php HTTP/1.1" 200 43823 "https://192.168.2.50/core/user_settings/user_dashboard.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
192.168.2.16 - - [08/Jan/2026:17:02:37 +0700] "GET /themes/default/css.php HTTP/1.1" 200 34428 "https://192.168.2.50/app/xml_cdr/xml_cdr.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36 Edg/143.0.0.0"
127.0.0.1 - b.wnX*pFUd [08/Jan/2026:17:12:09 +0700] "POST /app/xml_cdr/v_xml_cdr_import.php?uuid=a_cd6213c0-e274-4f27-a898-7ad91b118bb3 HTTP/1.1" 200 5 "-" "freeswitch-xml/1.0"
127.0.0.1 - b.wnX*pFUd [08/Jan/2026:17:12:29 +0700] "POST /app/xml_cdr/v_xml_cdr_import.php?uuid=a_804c22fa-7768-40f6-80fc-df60221a5911 HTTP/1.1" 200 5 "-" "freeswitch-xml/1.0"
^C
root@debian:~# tail -f /var/log/nginx/error.log
2026/01/15 17:21:22 [notice] 979#979: getrlimit(RLIMIT_NOFILE): 1024:524288
2026/01/15 17:21:22 [notice] 998#998: start worker processes
2026/01/15 17:21:22 [notice] 998#998: start worker process 1000
2026/01/15 17:21:22 [notice] 998#998: start worker process 1001
2026/01/15 17:21:22 [notice] 998#998: start worker process 1002
2026/01/15 17:21:22 [notice] 998#998: start worker process 1003
2026/01/15 17:21:22 [notice] 998#998: start worker process 1004
2026/01/15 17:21:22 [notice] 998#998: start worker process 1005
2026/01/15 17:21:22 [notice] 998#998: start worker process 1006
2026/01/15 17:21:22 [notice] 998#998: start worker process 1007
^C
```

```
root@debian:~# tail -f /var/log/nginx/error.log.1
2026/01/08 17:23:49 [notice] 22880#22880: worker process 22882 exited with code 0
2026/01/08 17:23:49 [notice] 22880#22880: worker process 22884 exited with code 0
2026/01/08 17:23:49 [notice] 22880#22880: signal 29 (SIGIO) received
2026/01/08 17:23:49 [notice] 22880#22880: signal 17 (SIGCHLD) received from 22884
2026/01/08 17:23:49 [notice] 22880#22880: signal 17 (SIGCHLD) received from 22886
2026/01/08 17:23:49 [notice] 22880#22880: worker process 22885 exited with code 0
2026/01/08 17:23:49 [notice] 22880#22880: worker process 22886 exited with code 0
2026/01/08 17:23:49 [notice] 22880#22880: worker process 22887 exited with code 0
2026/01/08 17:23:49 [notice] 22880#22880: worker process 22888 exited with code 0
2026/01/08 17:23:49 [notice] 22880#22880: exit
^C
root@debian:~#

```

`access.log`
![](Pasted%20image%2020260115174107.png)

`access.log.1`
![](Pasted%20image%2020260115174120.png)

`error.log`
![](Pasted%20image%2020260115174203.png)

`error.log.1`
![](Pasted%20image%2020260115174223.png)

# 4. Check log Postgres

`ls /var/log/postgres/`

```
root@debian:~# ls /var/log/
alternatives.log  faillog         lastlog           popularity-contest      README                vmware-network.3.log  vmware-network.log       vmware-vmtoolsd-root.log
apt               fontconfig.log  nginx             popularity-contest.0    runit                 vmware-network.4.log  vmware-vmsvc-root.1.log  wtmp
btmp              freeswitch      php7.2            popularity-contest.gpg  sysstat               vmware-network.5.log  vmware-vmsvc-root.2.log
dpkg.log          installer       php7.2-fpm.log    postgresql              vmware-network.1.log  vmware-network.6.log  vmware-vmsvc-root.3.log
exim4             journal         php7.2-fpm.log.1  private                 vmware-network.2.log  vmware-network.7.log  vmware-vmsvc-root.log
root@debian:~# ls /var/log/postgresql/
postgresql-12-main.log  postgresql-12-main.log.1
root@debian:~# tail -f /var/log/postgresql/postgresql-12-main.log
                                        and bridge_uuid is null
                                        and sip_hangup_disposition <> 'send_refuse'
                        )
                        or (
                                direction = 'outbound'
                                and answer_stamp is null
                                and bridge_uuid is not null
                        )) THEN 'BUSY' WHEN (answer_stamp is null and bridge_uuid is null and billsec = 0 and sip_hangup_disposition = 'send_refuse') THEN 'FAILED' ELSE 'UNKNOWN' END AS Status
        from v_xml_cdr as c
        left join v_extensions as e on e.extension_uuid = c.extension_uuid left join call_rate cr on c.domain_uuid = cr.domain_uuid and c.xml_cdr_uuid = cr.call_id inner join v_domains as d on d.domain_uuid = c.domain_uuid where start_stamp BETWEEN '2026-01-15 00:00' AND '2026-01-15 00:00' order by start_epoch desc limit 100 offset 0
```
![](Pasted%20image%2020260115174515.png)

# 5. Nguyên nhân



![](Pasted%20image%2020260115174710.png)

![](Pasted%20image%2020260115174651.png)
```
root@debian:~# tail -f /var/log/postgresql/postgresql-12-main.log
                                        and bridge_uuid is null
                                        and sip_hangup_disposition <> 'send_refuse'
                        )
                        or (
                                direction = 'outbound'
                                and answer_stamp is null
                                and bridge_uuid is not null
                        )) THEN 'BUSY' WHEN (answer_stamp is null and bridge_uuid is null and billsec = 0 and sip_hangup_disposition = 'send_refuse') THEN 'FAILED' ELSE 'UNKNOWN' END AS Status
        from v_xml_cdr as c
        left join v_extensions as e on e.extension_uuid = c.extension_uuid left join call_rate cr on c.domain_uuid = cr.domain_uuid and c.xml_cdr_uuid = cr.call_id inner join v_domains as d on d.domain_uuid = c.domain_uuid order by start_epoch desc limit 100 offset 0
2026-01-15 17:45:58.890 +07 [10005] fusionpbx@fusionpbx ERROR:  relation "call_rate" does not exist at character 1332
2026-01-15 17:45:58.890 +07 [10005] fusionpbx@fusionpbx STATEMENT:  select
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
        (c.answer_epoch - c.start_epoch) as tta , c.domain_name
         , CASE
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
        left join v_extensions as e on e.extension_uuid = c.extension_uuid left join call_rate cr on c.domain_uuid = cr.domain_uuid and c.xml_cdr_uuid = cr.call_id inner join v_domains as d on d.domain_uuid = c.domain_uuid order by start_epoch desc limit 100 offset 0
```


# 6. Log lỗi như sau:

```
2026-01-15 17:45:58.890 +07 [10005] fusionpbx@fusionpbx ERROR:  relation "call_rate" does not exist at character 1332
2026-01-15 17:45:58.890 +07 [10005] fusionpbx@fusionpbx STATEMENT:  
select
.....
from v_xml_cdr as c
        left join v_extensions as e on e.extension_uuid = c.extension_uuid left join call_rate cr on c.domain_uuid = cr.domain_uuid and c.xml_cdr_uuid = cr.call_id inner join v_domains as d on d.domain_uuid = c.domain_uuid order by start_epoch desc limit 100 offset 0
```

# 7. Fix 

## Tiến hành tạo bảng đang thiếu là call_rate:

```
su - postgres
```

```
psql -d fusionpbx
```

```
CREATE TABLE call_rate (
    call_rate_uuid uuid NOT NULL,
    domain_uuid uuid,
    call_id uuid,
    keypress text,
    CONSTRAINT call_rate_pkey PRIMARY KEY (call_rate_uuid)
);
```

```
CREATE INDEX idx_call_rate_call_id ON call_rate (call_id);
```

```
\dt call_rate
```
### Kết quả tạo bảng:
```
root@debian:~# su - postgres
postgres@debian:~$ psql
psql (12.22 (Debian 12.22-3.pgdg12+1))
Type "help" for help.

postgres=# \c fusionpbx
You are now connected to database "fusionpbx" as user "postgres".
fusionpbx=# CREATE TABLE call_rate (
    call_rate_uuid uuid NOT NULL,
    domain_uuid uuid,
    call_id uuid,
    keypress text,
    CONSTRAINT call_rate_pkey PRIMARY KEY (call_rate_uuid)
);
CREATE TABLE
fusionpbx=# CREATE INDEX idx_call_rate_call_id ON call_rate (call_id);
CREATE INDEX
fusionpbx=# \dt call_rate
           List of relations
 Schema |   Name    | Type  |  Owner
--------+-----------+-------+----------
 public | call_rate | table | postgres
(1 row)

fusionpbx=#
```

![](Pasted%20image%2020260115175423.png)

![](Pasted%20image%2020260120164703.png)

Bảng call_rate

| Cột          | Loại                  | Chú thích |
| ------------ | --------------------- | --------- |
| customer_num | character varying(30) |           |
| domain_uuid  | uuid                  |           |
| call_id      | uuid                  |           |
| sip_call_id  | text                  |           |
| keypress     | character(30)         |           |
| created_time | timestamp             |           |

![](Pasted%20image%2020260120170255.png)

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



![](Pasted%20image%2020260120170442.png)

# Sau khi tạo bảng xong, vào web kiểm tra

## Refresh lại trang thì đã hiển thị đầy đủ 4 CDR.

![](Pasted%20image%2020260115175534.png)

![](Pasted%20image%2020260115175621.png)

![](Pasted%20image%2020260115175640.png)

![](Pasted%20image%2020260115175700.png)

![](Pasted%20image%2020260115175725.png)

