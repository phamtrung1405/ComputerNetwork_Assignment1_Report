# Step 2 – Danh sách thiết bị tối thiểu, quy hoạch IP và sơ đồ đi dây

> Tên thiết bị theo **ảnh chụp `final.pkt`** (nguồn chuẩn), bổ sung từ Bảng 1 của `main.pdf` khi ảnh không đọc rõ. Chi tiết đối chiếu: `doi-chieu-ten-thiet-bi.md`.
> Mức tin cậy: **[CFG]** = lấy từ `show running-config` đã gửi; **[BC]** = lấy từ báo cáo `main.pdf`; **[ĐX]** = đề xuất, chưa có cấu hình xác nhận.

---

## 1. Danh sách thiết bị đề xuất và thông số điển hình

### 1.1 25 thiết bị mạng trong topology mô phỏng [BC]

| #   | Tên thiết bị   | Dòng thiết bị      | Vị trí / vùng         | Vai trò chính                                                   |
| --- | -------------- | ------------------ | --------------------- | --------------------------------------------------------------- |
| 1   | MAIN-CORE-SW1  | Catalyst 3560-24PS | Main – Core           | HSRP Active (prio 200), STP root (4096), OSPF RID 2.2.2.1, LACP |
| 2   | MAIN-CORE-SW2  | Catalyst 3560-24PS | Main – Core           | HSRP Standby (prio 150), STP secondary (8192), OSPF RID 2.2.2.2 |
| 3   | Server-SW | Catalyst 2960-24TT | Main – Data center    | Switch L2 cho server farm VLAN 40                               |
| 4   | Mgmt-SW   | Catalyst 2960-24TT | Main – phòng IT       | Jump Host, Syslog/NTP (VLAN 90)                                 |
| 5   | Cam-SW    | Catalyst 2960-24TT | Main – an ninh        | Camera IP, NVR (VLAN 70)                                        |
| 6   | MAIN-Access-A  | Catalyst 2960-24TT | Main – tòa A          | Máy trạm nhân viên, AP khách (VLAN 10, 60, 80)                  |
| 7   | MAIN-Access-B  | Catalyst 2960-24TT | Main – tòa B          | Khối hành chính, bệnh án, Wi-Fi (VLAN 20, 60)                   |
| 8   | MAIN-FW-01      | Cisco ASA 5506-X   | Main – firewall trong | Phân tách inside / inside-backup / wan / backup-wan / dmz       |
| 9   | MAIN-FW-02      | Cisco ASA 5506-X   | Main – firewall biên  | Static NAT cho Web DMZ, PAT cho nội bộ ra Internet              |
| 10  | DMZ-SW    | Catalyst 3560-24PS | Main – DMZ            | Trung chuyển MAIN-FW-01 ↔ MAIN-FW-02 ↔ Web Server                           |
| 11  | MAIN-WAN1-SW   | Catalyst 2960-24TT | Main – biên WAN       | Tuyến chính: MAIN-FW-01 `Gi1/2` ↔ MAIN-WAN-R1                         |
| 12  | MAIN-WAN2-SW   | Catalyst 2960-24TT | Main – biên WAN       | Tuyến phụ: MAIN-FW-01 `Gi1/1` ↔ MAIN-WAN-R2                           |
| 13  | MAIN-WAN-R1    | Cisco 2911         | Main – biên WAN       | Leased line: OSPF Area 0, cost 10, Serial0/3/0 tới Carrier-R    |
| 14  | MAIN-WAN-R2    | Cisco 2911         | Main – biên WAN       | Broadband dự phòng: định tuyến tĩnh, tới ISP2-R                 |
| 15  | ISP1-R         | Cisco 2911         | Internet              | Router nhà cung cấp Internet, dải công cộng 203.162.10.0/29     |
| 16  | Carrier-R      | Cisco 2911         | Backbone              | Mạng leased line/MPLS giả lập, OSPF Area 0                      |
| 17  | ISP2-R         | Cisco 2911         | Backbone phụ          | Mạng broadband giả lập cho đường dự phòng                       |
| 18  | DBP-R1         | Cisco 2911         | DBP                   | HSRP Active (110), router-on-a-stick, OSPF qua Carrier-R        |
| 19  | DBP-R2         | Cisco 2911         | DBP                   | HSRP Standby (105), floating static AD 150 qua ISP2-R           |
| 20  | DBP-SW1        | Catalyst 2960-24TT | DBP                   | Trunk 802.1Q tới DBP-R1/R2, VLAN 10/30/40/70/90                 |
| 21  | DBP-SW2        | Catalyst 2960-24TT | DBP                   | Dự phòng vòng lặp STP, máy trạm và phòng xét nghiệm             |
| 22  | BHTQ-R1        | Cisco 2911         | BHTQ                  | Như DBP-R1                                                      |
| 23  | BHTQ-R2        | Cisco 2911         | BHTQ                  | Như DBP-R2                                                      |
| 24  | BHTQ-SW1       | Catalyst 2960-24TT | BHTQ                  | Như DBP-SW1                                                     |
| 25  | BHTQ-SW2       | Catalyst 2960-24TT | BHTQ                  | Như DBP-SW2                                                     |

### 1.1b Thiết bị có trong ảnh topology nhưng không có trong Bảng 1 của báo cáo

Tên lấy từ ảnh chụp `final.pkt` (nguồn chuẩn). Vai trò chưa có cấu hình xác nhận nên **cần bạn bổ sung**.

| Tên trong ảnh | Dòng thiết bị  | Vị trí                          | Ghi chú                                                                 |
| ------------- | -------------- | ------------------------------- | ----------------------------------------------------------------------- |
| MAIN-VPN      | Cisco 2911     | Vùng Internet Edge              | Router VPN (site-to-site/remote-access); nối ISP1-SW và vùng firewall   |
| ISP1-SW       | Catalyst 2960-24TT | Phía Internet               | Nối ISP1-R, MAIN-VPN và máy `REMOTE`                                    |
| REMOTE        | Laptop-PT      | Phía Internet                   | Máy teleworker thử VPN                                                  |
| INTERNET WEB  | Server-PT      | Phía Internet                   | Web server Internet (`192.0.2.10` trong báo cáo)                        |

Quy ước tên theo ảnh: firewall **MAIN-FW-01** (trong, nối Core/WAN/DMZ) và **MAIN-FW-02** (biên Internet); switch core **MAIN-CORE-SW1/2**; switch WAN **MAIN-WAN1-SW/MAIN-WAN2-SW**; `DMZ-SW`, `Server-SW`, `Mgmt-SW`, `Cam-SW` không có tiền tố `MAIN-` trong ảnh. Tên switch tòa A/B (Access) chưa đọc rõ trong ảnh nên vẫn ghi `MAIN-Access-A/B`.

### 1.2 Thiết bị đầu cuối và dịch vụ (Packet Tracer)

| Thiết bị                                          | Số lượng / ghi chú                                                                                                       | Mức tin cậy |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------- |
| Server-PT: DNS/DHCP `10.10.40.10`                 | Server farm VLAN 40; còn App, DB, File server ở `.11–.13`                                                                | [BC]/[ĐX]   |
| Server-PT: Web Server `10.10.50.10`               | DMZ; Static NAT `203.162.10.3`; có thêm máy Web `10.10.50.11` dùng trong test                                            | [CFG]/[BC]  |
| Jump Host `10.10.90.30`, Syslog/NTP `10.10.90.20` | VLAN 90                                                                                                                  | [BC]        |
| Server camera/NVR, webcam                         | VLAN 70 ở mỗi site                                                                                                       | [ĐX]        |
| Máy trạm, máy in, laptop, smartphone              | Tên thật cần lấy từ `.pkt` (báo cáo ghi: `MAIN-PC-STAFFA`, `PC_DATA`, `DBP-PC1`, `PC-BHTQ1/3`, `MAIN-B-GUEST`, `REMOTE`) | [BC]        |
| AP/WLC (LAP-PT, WLC-PT)                           | Wi-Fi nhân viên và khách                                                                                                 | [ĐX]        |

### 1.3 Thiết bị thực tế tương ứng (khi triển khai thật) [ĐX]

| Vai trò    | Gợi ý thiết bị thật                        | Thông số điển hình cần có                               |
| ---------- | ------------------------------------------ | ------------------------------------------------------- |
| Core (L3)  | Catalyst 9300/9500                         | 24–48 cổng 10G, stacking/StackWise, PoE+ không bắt buộc |
| Access     | Catalyst 9200/2960-X, PoE+                 | 24/48 cổng 1G PoE+, uplink 10G (SFP+)                   |
| Firewall   | Cisco Firepower 1120/2110 hoặc tương đương | Throughput khi bật IPS ≥ 2 Gbps, chạy HA Active/Standby |
| Router WAN | Cisco ISR 4331/4431                        | Hỗ trợ IPsec, QoS, 2 WAN                                |
| Wireless   | Catalyst 9120 + WLC 9800                   | Wi-Fi 6, 802.1X                                         |
| Camera/NVR | Camera IP PoE 4 MP, NVR 64 kênh            | Lưu trữ theo số ngày yêu cầu                            |

> Các dòng "thiết bị thật" là gợi ý minh họa, cần chốt theo báo giá. Packet Tracer không có IPS/IDS và failover firewall ASA nên các tính năng này chỉ nằm trong thiết kế đích.

---

## 2. Quy hoạch IP

### 2.1 Dải địa chỉ [BC]

| Site / mạng       | Dải             | Ghi chú                                   |
| ----------------- | --------------- | ----------------------------------------- |
| Main Site         | 10.10.0.0/16    | Tóm tắt route về Main                     |
| DBP               | 10.20.0.0/16    |                                           |
| BHTQ              | 10.30.0.0/16    |                                           |
| WAN điểm–điểm     | 10.255.0.0/24   | Các liên kết /30                          |
| Internet mô phỏng | 203.162.10.0/29 | ISP1-R `.1`, MAIN-FW-02 `.2`, Web NAT `.3` |
| Remote-access VPN | 10.10.200.0/24  | Client nhận `10.10.200.10` (VPN-RA-01)    |

### 2.2 VLAN và subnet tại Main Site [BC]

| VLAN | Tên             | Subnet         | VIP HSRP             | Ghi chú                                    |
| ---- | --------------- | -------------- | -------------------- | ------------------------------------------ |
| 10   | Staff-A         | 10.10.10.0/23  | 10.10.10.1           | DHCP relay tới 10.10.40.10                 |
| 20   | Staff-B         | 10.10.20.0/23  | 10.10.20.1           | Khối hành chính, tiếp đón                  |
| 30   | Medical Devices | 10.10.30.0/24  | 10.10.30.1           |                                            |
| 40   | Server Farm     | 10.10.40.0/24  | 10.10.40.1           | HIS, PACS, LIS, DNS/DHCP                   |
| 50   | DMZ             | 10.10.50.0/24  | MAIN-FW-01 `.1`, MAIN-FW-02 `.2` | Web Server `.10`                           |
| 60   | Guest Wi-Fi     | 10.10.60.0/22  | 10.10.60.1           | ACL cô lập                                 |
| 70   | Camera          | 10.10.70.0/24  | 10.10.70.1           |                                            |
| 80   | Voice           | 10.10.80.0/24  | 10.10.80.1           | QoS                                        |
| 90   | Management      | 10.10.90.0/24  | 10.10.90.1           | Jump Host, Syslog/NTP                      |
| 250  | Transit Inside  | 10.10.250.0/29 | 10.10.250.1          | Core ↔ MAIN-FW-01 `Gi1/4` (`10.10.250.4`)        |
| 253  | Inside Backup   | 10.10.253.0/29 | –                    | MAIN-CORE-SW2 ↔ MAIN-FW-01 `Gi1/5` (`10.10.253.1`)    |
| 99   | Native (L2)     | –              | –                    | Chỉ là native VLAN trên trunk, không có IP |

### 2.3 Chi nhánh [BC]/[CFG]

| VLAN | DBP           | BHTQ          | VIP HSRP (.1) |
| ---- | ------------- | ------------- | ------------- |
| 10   | 10.20.10.0/24 | 10.30.10.0/24 | `.1`          |
| 30   | 10.20.30.0/24 | 10.30.30.0/24 | `.1`          |
| 40   | 10.20.40.0/24 | 10.30.40.0/24 | `.1`          |
| 70   | 10.20.70.0/24 | 10.30.70.0/24 | `.1`          |
| 90   | 10.20.90.0/24 | 10.30.90.0/24 | `.1`          |
| 99   | 10.20.99.0/24 | 10.30.99.0/24 | `.1`          |

Router R1 dùng `.2`, R2 dùng `.3` trên mỗi subinterface. **Lưu ý dung lượng:** 260 máy trạm không vừa `/24` ở VLAN 10; nếu giữ đúng yêu cầu đề bài, đổi VLAN 10 chi nhánh sang `/23` (xem Step 6).

### 2.4 Mạng WAN điểm–điểm /30 [BC]/[CFG]

| Liên kết                | Subnet         | Đầu A             | Đầu B            |
| ----------------------- | -------------- | ----------------- | ---------------- |
| MAIN-WAN-R1 – Carrier-R | 10.255.0.0/30  | `.1` (WAN-R1)     | `.2` (Carrier-R) |
| MAIN-WAN-R2 – Carrier-R | 10.255.0.4/30  | `.5` (WAN-R2)     | `.6` (Carrier-R) |
| DBP-R1 – Carrier-R      | 10.255.0.8/30  | `.9` (Carrier-R)  | `.10` (DBP-R1)   |
| BHTQ-R1 – Carrier-R     | 10.255.0.12/30 | `.13` (Carrier-R) | `.14` (BHTQ-R1)  |
| DBP-R2 – ISP2-R         | 10.255.0.16/30 | `.17` (ISP2-R)    | `.18` (DBP-R2)   |
| BHTQ-R2 – ISP2-R        | 10.255.0.20/30 | `.21` (ISP2-R)    | `.22` (BHTQ-R2)  |
| MAIN-WAN-R2 – ISP2-R    | 10.255.0.24/30 | `.25` (ISP2-R)    | `.26` (WAN-R2)   |

### 2.5 Mạng vùng biên Main [CFG]

| Mạng                     | Subnet          | Thiết bị                                                        |
| ------------------------ | --------------- | --------------------------------------------------------------- |
| Internet                 | 203.162.10.0/29 | ISP1-R `.1`, MAIN-FW-02 `outside` `.2`, NAT Web `.3`             |
| DMZ                      | 10.10.50.0/24   | MAIN-FW-01 `dmz` `.1`, MAIN-FW-02 `dmz` `.2`, Web Server `.10`              |
| WAN chính                | 10.10.251.0/29  | MAIN-FW-01 `wan` `.1`, MAIN-WAN-R1 `.3`                               |
| WAN dự phòng             | 10.10.252.0/29  | MAIN-FW-01 `backup-wan` `.1`, MAIN-WAN-R2 `.4`                        |
| Inside transit           | 10.10.250.0/29  | VIP Core `.1`, MAIN-CORE-SW1 `.2`, MAIN-CORE-SW2 `.3`, MAIN-FW-01 `inside` `.4` |
| Inside backup (shutdown) | 10.10.253.0/29  | MAIN-FW-01 `inside-backup` `.1`, MAIN-CORE-SW2 `.2`                        |

Bảng địa chỉ đầy đủ từng thiết bị: `bang-dia-chi-thiet-bi.md`. (Bảng đó vẫn dùng tên cũ `WAN-SW`, `MAIN-CORE-SW1`…; đối chiếu bằng `doi-chieu-ten-thiet-bi.md`.)

---

## 3. Sơ đồ đi dây (cabling)

### 3.1 Đi dây vật lý Main Site

- Trung tâm dữ liệu/IT/MDF đặt trong phòng riêng, cách tòa A và B **50 m**. Do vượt giới hạn 90 m của cáp đồng khi cộng đường đi trong tòa, dùng **cáp quang đa mode** (OM3/OM4) nối MDF với IDF của từng tòa; trong tòa dùng Cat6.
- Mỗi tầng một IDF (tòa A và B: 5 tầng); gộp về switch tòa (`MAIN-Access-A`, `MAIN-Access-B`) rồi uplink về hai Core.
- Mọi uplink Access → Core dùng **EtherChannel LACP**; hai Core nối nhau bằng **Port-channel 1** (hai cáp).

### 3.2 Bảng kết nối chính (logic) [CFG]/[BC]

| Từ              | Cổng          | Đến                                         | Cổng / ghi chú                                   |
| --------------- | ------------- | ------------------------------------------- | ------------------------------------------------ |
| MAIN-CORE-SW1   | Fa0/2, Fa0/7  | MAIN-CORE-SW2                               | Port-channel 1 (LACP), trunk                     |
| MAIN-CORE-SW1   | (uplink)      | MAIN-FW-01                                   | `Gi1/4` (inside), VLAN 250                       |
| MAIN-CORE-SW2   | (uplink)      | MAIN-FW-01                                   | `Gi1/5` (inside-backup), VLAN 253, đang shutdown |
| MAIN-CORE-SW1/2 | (uplink)      | MAIN-Access-A/B, Server-SW, Mgmt-SW, Cam-SW | Trunk, EtherChannel                              |
| MAIN-FW-01       | Gi1/3         | DMZ-SW                                 | `dmz`, `10.10.50.1`                              |
| MAIN-FW-02       | Gi1/2         | DMZ-SW                                 | `dmz`, `10.10.50.2`                              |
| MAIN-FW-02       | Gi1/1         | ISP1-R                                      | `outside` `203.162.10.2/29`                      |
| MAIN-FW-01       | Gi1/2         | MAIN-WAN1-SW                                | `wan`                                            |
| MAIN-WAN1-SW    | –             | MAIN-WAN-R1                                 | Gi0/0                                            |
| MAIN-FW-01       | Gi1/1         | MAIN-WAN2-SW                                | `backup-wan`                                     |
| MAIN-WAN2-SW    | –             | MAIN-WAN-R2                                 | Gi0/0                                            |
| MAIN-WAN-R1     | Serial0/3/0   | Carrier-R                                   | `10.255.0.0/30`, OSPF cost 10                    |
| MAIN-WAN-R2     | Gi0/1         | ISP2-R                                      | `10.255.0.24/30`                                 |
| Carrier-R       | Serial        | DBP-R1, BHTQ-R1                             | `10.255.0.8/30`, `10.255.0.12/30`                |
| ISP2-R          | Gi0/1, Gi0/2  | DBP-R2, BHTQ-R2                             | `10.255.0.16/30`, `10.255.0.20/30`               |
| DBP-R1/R2       | Gi0/0 (trunk) | DBP-SW1/SW2                                 | Subinterface `Gi0/0.10/30/40/70/90/99`           |
| BHTQ-R1/R2      | Gi0/0 (trunk) | BHTQ-SW1/SW2                                | Như DBP                                          |

Số cổng cụ thể trên các switch truy cập và thứ tự cổng Serial của Carrier-R **cần đối chiếu với `final.pkt`** (báo cáo và cấu hình dán chưa khớp hoàn toàn, ví dụ WAN-R2 `Serial0/3/1`).

### 3.3 Hai đường WAN giữa Main và chi nhánh

| Đường                    | Đường đi                                                                        | Cơ chế                                            |
| ------------------------ | ------------------------------------------------------------------------------- | ------------------------------------------------- |
| **Chính (leased line)**  | MAIN-FW-01 `wan` → MAIN-WAN1-SW → MAIN-WAN-R1 → Carrier-R → DBP-R1 / BHTQ-R1     | OSPF Area 0, cost 10                              |
| **Dự phòng (broadband)** | MAIN-FW-01 `backup-wan` → MAIN-WAN2-SW → MAIN-WAN-R2 → ISP2-R → DBP-R2 / BHTQ-R2 | Floating static, AD 150 (chi nhánh), AD 10 (MAIN-FW-01) |

### 3.4 Công nghệ WAN: các phương án và chi phí (so sánh)

Đề bài yêu cầu liệt kê các phương án, đề xuất chi phí, phân tích ưu nhược điểm. **Chi phí dưới đây là ước lượng minh họa**, thay bằng báo giá nhà mạng thực.

| Phương án                         | Mô tả                                                         | Chi phí tương đối | Ưu điểm                                             | Nhược điểm                                                   |
| --------------------------------- | ------------------------------------------------------------- | ----------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| **Leased line (kênh thuê riêng)** | Đường truyền riêng điểm–điểm Main ↔ từng chi nhánh            | Cao               | Băng thông cam kết, độ trễ thấp, riêng biệt         | Đắt; mở rộng thêm site tốn kém                               |
| **MPLS L3VPN**                    | Nhà mạng cấp VPN lớp 3, QoS theo lớp dịch vụ                  | Cao – trung bình  | SLA, QoS, dễ mở rộng site                           | Phí tháng cao; phụ thuộc nhà mạng                            |
| **SD-WAN trên Internet**          | Overlay IPsec trên nhiều đường Internet, điều khiển tập trung | Trung bình – thấp | Rẻ, linh hoạt, dùng nhiều đường, quản trị tập trung | Chất lượng phụ thuộc Internet; cần bản quyền/thiết bị SD-WAN |
| **IPsec VPN trên broadband**      | Đường hầm site-to-site qua Internet thường                    | Thấp              | Rẻ, triển khai nhanh                                | Không cam kết chất lượng                                     |
| **Hybrid (chọn)**                 | Leased line/MPLS làm chính + broadband IPsec làm dự phòng     | Trung bình        | Cân bằng chi phí và độ tin cậy                      | Cấu hình định tuyến phức tạp hơn                             |

**Phương án chọn:** hybrid. Mô phỏng: Carrier-R giả lập leased line/MPLS (OSPF), ISP2-R giả lập broadband (floating static) và có IPsec site-to-site giữa các router WAN (`VPN-S2S-01`).

---

## 4. Việc cần đối chiếu với `final.pkt`

1. Tên thiết bị đầu cuối và số cổng cụ thể.
2. Cổng Serial của Carrier-R và WAN-R2 nối tới ai.
3. Subnet VLAN 10 chi nhánh (`/24` hay `/23`) và DHCP chi nhánh (relay tới `10.10.40.10` theo báo cáo, hay server riêng `10.20.40.10` theo cấu hình đã dán).
