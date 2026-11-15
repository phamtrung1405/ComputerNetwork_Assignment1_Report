# DANH SÁCH ẢNH CẦN CHỤP LẠI / BỔ SUNG THEO SƠ ĐỒ CHUẨN `h.png`

Căn cứ: `h.png` là sơ đồ chuẩn. Các ảnh test trong `graphics/tc_pic/` được chụp từ phiên bản mô phỏng cũ (tiêu đề cửa sổ ghi `demo_v2.pkt`) nên một số ảnh còn thể hiện tên/máy chủ khác `h.png`.

## 1. Khác biệt giữa phiên bản cũ (thấy trong ảnh test) và `h.png`

| Hạng mục           | Ảnh test cũ                                               | `h.png` (chuẩn)                                                                                 |
| ------------------ | --------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Server Farm        | `MAIN-DHCP`, `MAIN-APP`, `MAIN-DB`, `MAIN-FTP/File`       | `MAIN-DHCP`, `MAIN-SYSLOG`, `MAIN-EMAIL`, `MAIN-FTP`                                            |
| Management         | `MAIN-Syslog/NTP` (một máy), `AAA`, `MAIN-SSH`            | `MAIN-NTP`, `AAA`, `MAIN-SSH` (Syslog chuyển sang Server Farm)                                  |
| Switch Office A    | nhãn `ASW-MAIN-A`                                         | `SW-MAIN-A`                                                                                     |
| Hostname trong CLI | `Core-SW1#`, `Core-SW2#`, `WAN-R2#`, `VPN-GW#`, `DBP-R2#` | tên trên sơ đồ: `MAIN-CORE-SW1/2`, `MAIN-WAN-R2`, `MAIN-VPN`... (chỉ chụp lại nếu đổi hostname) |
| Địa chỉ Syslog     | log hiển thị `Logging to host 10.10.90.20`                | `MAIN-SYSLOG` = `10.10.40.11` (cần đổi `logging host` trên thiết bị)                            |

## 2. Ảnh có sẵn cần chụp lại

| Ảnh                                                                                | Ưu tiên           | Lý do                                                                                                                                                                        | Cần chụp lại như thế nào                                                                                                          |
| ---------------------------------------------------------------------------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `HA_01.1.png`                                                                      | Cao               | Nền topology còn nhãn cũ (`MAIN-APP`, `MAIN-DB`, `MAIN-Syslog/NTP`, `ASW-MAIN-A`); các cổng Core đang đỏ. Caption trong `cisco.tex` đang mô tả sai nội dung ảnh (xem mục 4). | Chụp lại khi mọi liên kết đã xanh, `ping -t` từ `MAIN-PC-STAFFA` tới đích DMZ; ghi rõ máy đích của `10.10.50.12` là thiết bị nào. |
| `HA_02.1.png`, `HA_02.2.png`                                                       | Trung bình        | CLI `DBP-R2` hiển thị `Logging to host 10.10.90.20`; nền topology cũ.                                                                                                        | Chụp lại sau khi đổi `logging host`; giữ lệnh `show standby brief` và ping `10.10.40.10`.                                         |
| `HA_01.png`                                                                        | Thấp              | Nền topology cũ; prompt `Core-SW1#`/`Core-SW2#`.                                                                                                                             | Chụp lại nếu đổi hostname sang `MAIN-CORE-SW1/2`.                                                                                 |
| `DMZ_01.png`                                                                       | Thấp              | Nền còn nhãn `MAIN-DB`, `MAIN-FT...`. Ảnh chứng minh `DMZ-SERVER` = `10.10.50.11`.                                                                                           | Chụp lại cùng bố cục sơ đồ mới; thêm 1 ảnh truy cập `DMZ-WEB` (`10.10.50.10`) nếu cần.                                            |
| `INT_01.png`                                                                       | Thấp              | Nền còn nhãn `MAIN-DB`.                                                                                                                                                      | Chụp lại cùng bố cục mới.                                                                                                         |
| `VPN_01.png`                                                                       | Thấp              | Nền Server Farm cũ (`MAIN-APP`, `MAIN-DB`, `MAIN-Syslog/NTP`).                                                                                                               | Chụp lại cùng bố cục mới.                                                                                                         |
| `VPN_01.1.png`                                                                     | Thấp              | Prompt `VPN-GW#`, tiêu đề cửa sổ `MAIN-VPN`.                                                                                                                                 | Giữ nguyên hoặc đổi hostname thành `MAIN-VPN`.                                                                                    |
| `VPN_02.png`                                                                       | Thấp              | Prompt `WAN-R2#`.                                                                                                                                                            | Chụp lại nếu đổi hostname sang `MAIN-WAN-R2`.                                                                                     |
| `SEC_01.png`                                                                       | Cao (khi sửa ACL) | Ca cô lập Guest đang FAIL vì Guest ping được `10.10.50.10`.                                                                                                                  | Sau khi chặn Guest khỏi DMZ, chụp lại: ping tới `10.10.20.10`, `10.10.40.10`, `10.10.50.10` đều thất bại.                         |
| `LAN_01.png`, `LAN_02.png`, `SEC_02.png`, `WAN_01.png`, `WAN_02.png`, `WAN_03.png` | Không cần         | Không hiển thị tên thiết bị cũ.                                                                                                                                              | Giữ nguyên.                                                                                                                       |

## 3. Ảnh cần chụp mới (chưa có bằng chứng)

1. Tuyến dự phòng `MAIN-CORE-SW2` -> `MAIN-FW-01` (VLAN 253, cổng `Gi1/5`): `show interfaces status`/`show ip route` trên cả hai đầu, và ảnh sơ đồ với liên kết đã xanh. Trên `h.png` đầu này đang có tam giác đỏ.
2. SSH từ `MAIN-SSH` (Jump Host, `10.10.90.30`) vào một thiết bị mạng.
3. Internet vào DMZ qua Static NAT: từ `INTERNET WEB` hoặc `REMOTE` truy cập `203.162.10.3`.
4. `MAIN-SYSLOG` (`10.10.40.11`) nhận log từ thiết bị; `MAIN-NTP` (`10.10.90.20`) đồng bộ giờ.
5. DHCP chi nhánh: `DBP-PC*`/`BHTQ-PC*` nhận địa chỉ từ `DBP-DHCP`/`BHTQ-DHCP` (`ipconfig`).
6. `show standby brief` trên `BHTQ-R1`/`BHTQ-R2` và `DBP-R1` (hiện chỉ có `DBP-R2` và hai switch lõi).
7. Ảnh toàn bộ topology hiện tại (`h.png`) để đưa vào báo cáo.

## 4. Caption trong `cisco.tex` (đã sửa theo nội dung ảnh)

Các caption `HA-01.1`, `WAN-01`, `WAN-02`, `WAN-03`, `HA-02.1` và đoạn mô tả "Ảnh bổ sung" đã được sửa cho khớp nội dung thật của ảnh, nên không cần chụp lại chỉ vì caption. Nếu chụp lại ảnh theo mục 2 thì phải cập nhật lại caption tương ứng.


## 5. Cấu hình cần đối chiếu trước khi chụp lại

- `config.md` còn `logging 10.10.90.20`; `MAIN-SYSLOG` mới là `10.10.40.11`.
- `config.md` còn `ip helper-address 10.10.40.10` trên subinterface router chi nhánh; sơ đồ có `DBP-DHCP`/`BHTQ-DHCP` tại chỗ.
- `config.md` dùng hostname `ASA1`, `ASA2`, `Core-SW1/2`, `Access-A/B`, `WAN1-SW`, `WAN2-SW`, `WAN-R1/R2` khác tên trên sơ đồ.
- Chưa có cấu hình cho `MAIN-VPN`, `ISP1-SW`, `MAIN-R`, `MAIN-WLC` trong `config.md`.
