# Step 1 – Tìm cấu trúc mạng phù hợp cho các tòa nhà

Phạm vi: Main Site (TP.HCM) và hai Auxiliary Site (DBP Street, BHTQ Street).

> **Cách đọc các con số:** số lượng người dùng, thiết bị, camera, băng thông… là **giả định để tính toán**, suy ra từ topology và quy hoạch địa chỉ (ví dụ `/23`, `/22`). Chúng phải được xác nhận bằng checklist khảo sát ở mục 2. Mình ghi rõ chỗ nào là giả định.

---

## 1. Phân tích yêu cầu hệ thống mạng

### 1.1 Vai trò từng site

| Site                   | Vai trò                                                                                 | Dải địa chỉ   |
| ---------------------- | --------------------------------------------------------------------------------------- | ------------- |
| **Main Site – TP.HCM** | Trung tâm mạng, quản lý và cung cấp dịch vụ tập trung (server, DMZ, firewall, Internet) | 10.10.0.0/16  |
| **DBP Street Site**    | Chi nhánh 1: người dùng, máy in, camera, Wi-Fi; dùng dịch vụ từ Main Site               | 10.20.0.0/16  |
| **BHTQ Street Site**   | Chi nhánh 2: tương tự DBP                                                               | 10.30.0.0/16  |
| **WAN**                | Liên kết liên site (leased line chính, broadband dự phòng)                              | 10.255.0.0/24 |

### 1.2 Yêu cầu chức năng

| Nhóm                 | Yêu cầu                                                  | Biểu hiện trong thiết kế          |
| -------------------- | -------------------------------------------------------- | --------------------------------- |
| Truy cập người dùng  | Có dây và không dây cho nhân viên; Wi-Fi riêng cho khách | VLAN STAFF-A, STAFF-B, GUEST-WIFI |
| Dịch vụ tập trung    | DNS, DHCP, ứng dụng, cơ sở dữ liệu, file, thoại IP       | Server farm VLAN 40, CME          |
| Dịch vụ công khai    | Web, DNS, Email cho Internet                             | DMZ VLAN 50                       |
| Thiết bị chuyên dụng | Thiết bị y tế, camera giám sát, đầu ghi NVR              | VLAN MEDICAL, CAMERA              |
| Liên lạc liên site   | Chi nhánh dùng dịch vụ của Main Site                     | WAN leased line + broadband       |
| Quản trị             | Giám sát tập trung, log, xác thực quản trị               | Syslog/NTP, AAA, Jump Host        |

### 1.3 Yêu cầu phi chức năng

| Nhóm      | Yêu cầu                                             | Chỉ tiêu đề xuất (cần xác nhận)                |
| --------- | --------------------------------------------------- | ---------------------------------------------- |
| Sẵn sàng  | Không điểm hỏng đơn ở lõi và gateway                | Core đôi + HSRP; hai ISP; chi nhánh hai router |
| Bảo mật   | Phân vùng, kiểm soát luồng, cách ly khách và camera | Firewall nhiều zone, VLAN, ACL                 |
| Hiệu năng | Không nghẽn ở server farm, WAN, firewall            | Mục 3                                          |
| Mở rộng   | Dự phòng **tăng 20% thiết bị trong 5 năm**          | Quy hoạch subnet có dư (`/23`, `/22`)          |
| Vận hành  | Dễ quản trị, dễ đi dây, thẩm mỹ                     | Mục 4                                          |

### 1.4 Quy mô địa chỉ cho phép (dùng làm mục tiêu thiết kế)

| VLAN        | Subnet | Host tối đa | Mục tiêu hiện tại tối đa (sau trừ 20% tăng trưởng) |
| ----------- | ------ | ----------- | -------------------------------------------------- |
| STAFF-A     | /23    | 510         | ≈ 425                                              |
| STAFF-B     | /23    | 510         | ≈ 425                                              |
| MEDICAL     | /24    | 254         | ≈ 210                                              |
| SERVER-FARM | /24    | 254         | ≈ 210                                              |
| DMZ         | /24    | 254         | ≈ 210                                              |
| GUEST-WIFI  | /22    | 1022        | ≈ 850                                              |
| CAMERA      | /24    | 254         | ≈ 210                                              |

Công thức: số host dùng được = 2^(32 − prefix) − 2; mục tiêu hiện tại = số host tối đa ÷ 1,2.

---

## 2. Checklist khảo sát tại vị trí lắp đặt

Tick từng mục khi khảo sát từng site (Main Site, DBP, BHTQ).

### 2.1 Tòa nhà và không gian

- [ ] Số tầng, diện tích từng tầng, sơ đồ mặt bằng (bản vẽ kiến trúc)
- [ ] Vị trí phòng kỹ thuật chính (MDF) và phòng/tủ từng tầng (IDF); khoảng cách giữa chúng
- [ ] Đường kỹ thuật dọc (riser), máng cáp, trần giả, sàn nâng; khả năng đi cáp mới
- [ ] Vật liệu tường, kính, kim loại ảnh hưởng sóng Wi-Fi
- [ ] Hạn chế thẩm mỹ: khu vực không được lộ cáp, thiết bị gắn trần

### 2.2 Điện, làm mát, an toàn

- [ ] Nguồn điện phòng máy, ổ cắm, công suất; có UPS và máy phát không
- [ ] Làm mát và thông gió phòng máy; nhiệt độ hiện tại
- [ ] Tiếp địa, chống sét; chống cháy, lối thoát
- [ ] Kiểm soát ra vào phòng máy (khóa, thẻ, camera)

### 2.3 Hệ thống cáp

- [ ] Loại cáp hiện có (Cat5e, Cat6, quang); chiều dài (đồng ≤ 90 m mỗi đoạn)
- [ ] Số điểm mạng hiện có và cần thêm cho từng khu (văn phòng, máy in, camera, AP)
- [ ] Cáp quang giữa các tòa nhà/tầng; loại (đơn mode, đa mode)
- [ ] Nhãn cáp, sơ đồ patch panel hiện có

### 2.4 Người dùng và thiết bị

- [ ] Số nhân viên theo phòng/khu; số tăng dự kiến sau 5 năm
- [ ] Số máy tính, máy in, IP phone, máy chủ hiện có
- [ ] Số camera và độ phân giải, thời gian lưu trữ yêu cầu
- [ ] Danh sách **thiết bị y tế**: nhà sản xuất, giao thức, có cố định IP, có cần PoE không
- [ ] Ứng dụng quan trọng và yêu cầu về độ trễ, thời gian chết cho phép

### 2.5 Wi-Fi

- [ ] Khảo sát phủ sóng (site survey): vị trí AP đề xuất, công suất, kênh
- [ ] Nhiễu sẵn có (Wi-Fi lân cận, thiết bị khác băng 2,4 GHz)
- [ ] Mật độ người dùng tại phòng họp, sảnh, khu chờ
- [ ] Yêu cầu Wi-Fi khách (số lượng, thời gian truy cập, cổng đăng nhập)

### 2.6 WAN và Internet

- [ ] Nhà mạng sẵn có tại từng địa điểm; loại kết nối (leased line, cáp quang, broadband)
- [ ] Băng thông, SLA, thời gian lắp, chi phí; khả năng dùng nhà mạng thứ hai
- [ ] Địa chỉ IP công cộng được cấp; điểm đấu nối vào tòa nhà

### 2.7 An ninh và tuân thủ

- [ ] Chính sách bảo mật hiện có của đơn vị
- [ ] Yêu cầu tuân thủ liên quan (bảo vệ dữ liệu bệnh nhân/khách hàng nếu có)
- [ ] Yêu cầu lưu log và sao lưu

---

## 3. Khu vực tải cao và cấu hình thiết bị (kể cả load balancer)

### 3.1 Khu vực tải cao

| Khu vực                      | Lý do tải cao                                      | Cấu hình thiết bị đề xuất                                                                                            |
| ---------------------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Core ↔ Access**            | Gom lưu lượng toàn bộ người dùng                   | Uplink gộp bằng **EtherChannel (LACP)**, Core đôi, ưu tiên cổng Gigabit trở lên                                      |
| **Server farm**              | Mọi người dùng và chi nhánh truy cập DB, App, file | Server nối hai Core; tách tầng App và DB; NIC kép nếu có; cần **load balancer** (xem 3.2)                            |
| **Firewall (Internet edge)** | NAT/PAT, kiểm tra gói của toàn bộ ra/vào           | Chọn firewall theo **throughput khi bật kiểm tra**, không theo thông số danh nghĩa; kiến trúc **hai firewall nối tiếp** (MAIN-FW-02 biên Internet, MAIN-FW-01 nội bộ); hiện chưa dùng cặp Active/Standby          |
| **WAN (leased line)**        | Chi nhánh dùng dịch vụ tập trung ở Main Site       | Băng thông leased line đủ cho đỉnh; **QoS** ưu tiên thoại, giữ chỗ băng thông cho ứng dụng; đường dự phòng broadband |
| **Camera / NVR**             | Luồng video liên tục                               | VLAN riêng, PoE đủ công suất, NVR có đường uplink riêng; không đi qua WAN nếu không cần                              |
| **Wi-Fi**                    | Nhiều thiết bị di động, Wi-Fi khách                | Mật độ AP phù hợp, giới hạn băng thông khách, WLC quản lý tập trung                                                  |
| **DMZ web**                  | Truy cập công khai từ Internet                     | Reverse proxy/WAF, **load balancer** trước nhiều web server                                                          |

### 3.2 Nơi đặt load balancer

| Vị trí                          | Mục đích                                                     | Triển khai                                                       |
| ------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------------- |
| **DMZ, trước Web Server**       | Chia tải và dự phòng cho web công khai, che giấu server thật | Cặp load balancer/reverse proxy đặt trong DMZ, sau firewall      |
| **Server farm, trước tầng App** | Chia tải nhiều App server, kiểm tra sức khỏe dịch vụ         | Load balancer nội bộ (VIP cho dịch vụ)                           |
| **Internet edge**               | Chia tải hai nhà mạng                                        | Chính sách định tuyến (PBR/ECMP), failover tự động               |
| **WAN HQ**                      | Dùng hai đường về chi nhánh                                  | OSPF chọn đường theo cost; leased line chính, đường phụ dự phòng |
| **Core**                        | Chia tải liên kết giữa hai Core                              | EtherChannel LACP (chia tải theo băm)                            |

> **Với Packet Tracer:** không có thiết bị load balancer. Cách mô phỏng: nhiều Server-PT cùng dịch vụ + DNS trả về nhiều địa chỉ (round-robin), hoặc dùng Web Proxy làm "đại diện". Load balancer thật ghi trong thuyết minh là thiết bị thiết kế đích.

### 3.3 Ví dụ tính băng thông (số liệu minh họa, thay bằng số khảo sát)

| Mục                            | Công thức                                      | Ví dụ                                                                    |
| ------------------------------ | ---------------------------------------------- | ------------------------------------------------------------------------ |
| Camera                         | số camera × bitrate mỗi camera                 | 32 camera × 4 Mbps = **128 Mbps** vào NVR                                |
| PoE camera                     | số camera × công suất mỗi camera               | 32 × 7 W ≈ 224 W (cộng dự phòng 20%)                                     |
| WAN chi nhánh → HQ             | số người dùng đồng thời × lưu lượng trung bình | 100 người × 1 Mbps ≈ **100 Mbps**                                        |
| Oversubscription access → core | (số cổng × tốc độ) ÷ tốc độ uplink             | 24 × 1 Gbps ÷ 2 Gbps (EtherChannel) = 12:1; chấp nhận được cho văn phòng |

---

## 4. Chọn cấu trúc mạng phù hợp kiến trúc tòa nhà

### 4.1 Tiêu chí chọn

| Tiêu chí         | Câu hỏi từ khảo sát                      |
| ---------------- | ---------------------------------------- |
| Quy mô           | Bao nhiêu tầng, bao nhiêu access switch? |
| Khoảng cách      | Đồng ≤ 90 m hay cần quang?               |
| Phát triển       | Có mở rộng thêm tòa nhà/tầng không?      |
| Chi phí vận hành | Có đủ nhân lực quản trị nhiều lớp không? |

### 4.2 Quyết định cấu trúc

| Site          | Cấu trúc đề xuất                                                                              | Lý do                                                                                                                                                                                      |
| ------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Main Site** | **Phân cấp 2 tầng (collapsed core)**: Core đôi gộp core và distribution, access theo khu/tầng | Số access switch vừa phải; ít thiết bị hơn 3 tầng; vẫn có dự phòng. **Nếu khảo sát thấy nhiều tòa nhà hoặc nhiều hơn khoảng 10–15 access switch, nâng lên 3 tầng** (thêm lớp distribution) |
| **DBP, BHTQ** | **Router + switch**, hai router (leased line và broadband) dùng HSRP                          | Quy mô nhỏ; ưu tiên dự phòng WAN hơn là nhiều lớp                                                                                                                                          |
| **Liên site** | **Hub-and-spoke**, HQ là hub                                                                  | Dịch vụ tập trung ở Main Site; chi nhánh chỉ cần đường về hub                                                                                                                              |

### 4.3 Tiện lợi và thẩm mỹ

| Hạng mục | Đề xuất                                                                                                  |
| -------- | -------------------------------------------------------------------------------------------------------- |
| Đi cáp   | Cáp có cấu trúc (structured cabling): MDF ở phòng chính, IDF ở từng tầng; cáp đi máng/trần giả, không lộ |
| Tủ rack  | Bố trí theo lớp: cáp quang và core ở trên, access ở dưới; có thanh quản lý cáp, patch panel              |
| Nhãn     | Dán nhãn cáp và cổng theo quy ước thống nhất (tầng-tủ-cổng)                                              |
| Màu cáp  | Phân màu patch cord theo chức năng (người dùng, server, quản trị, camera)                                |
| AP Wi-Fi | Gắn trần, đặt hợp mỹ quan, phủ đều; hạn chế AP đặt trong phòng riêng                                     |
| Dự phòng | Chừa 20–30% cổng trống và không gian rack cho mở rộng                                                    |

---

## 5. Thiết kế Wi-Fi, chuẩn bảo mật và phân vùng

### 5.1 Thiết kế Wi-Fi

| Hạng mục                         | Thiết kế                                                                                                                           |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Kiến trúc                        | **Lightweight AP (LAP) + WLC** đặt ở server farm; AP quản lý trong VLAN 90                                                         |
| SSID nhân viên                   | **WPA2/WPA3-Enterprise (802.1X)**, xác thực bằng RADIUS (AAA Server), gán **VLAN 20 STAFF-B**                                      |
| SSID khách                       | **VLAN 60 GUEST-WIFI**, cô lập thiết bị (client isolation), giới hạn băng thông, trang đăng nhập (captive portal), chỉ ra Internet |
| Thiết bị y tế không dây (nếu có) | SSID/VLAN riêng theo yêu cầu nhà sản xuất                                                                                          |
| Quy hoạch kênh                   | 2,4 GHz dùng kênh 1/6/11; 5 GHz ưu tiên cho nhân viên; công suất vừa phải để tránh chồng lấn                                       |
| Phủ sóng                         | Đặt AP theo kết quả site survey; mục tiêu tín hiệu khoảng −67 dBm tại vùng sử dụng                                                 |
| Chi nhánh                        | AP độc lập hoặc LAP về WLC tùy băng thông WAN; SSID nhân viên ở VLAN staff của chi nhánh                                           |

### 5.2 Chuẩn bảo mật áp dụng

| Lớp               | Chuẩn / biện pháp                                                                                                       |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Wi-Fi             | IEEE 802.11i (WPA2/WPA3), IEEE 802.1X với RADIUS; phát hiện AP lạ (rogue AP)                                            |
| Truy cập thiết bị | SSH v2, xác thực tập trung (AAA), chỉ Jump Host được quản trị                                                           |
| Cổng switch       | Port-security, BPDU Guard, tắt cổng không dùng                                                                          |
| Phân vùng         | VLAN + firewall zone + ACL (nguyên tắc ít quyền nhất)                                                                   |
| Giám sát          | Syslog tập trung, đồng bộ giờ NTP                                                                                       |
| Khung quản trị    | Có thể tham chiếu ISO/IEC 27001 và các quy định bảo vệ dữ liệu liên quan; **chọn chuẩn cụ thể theo yêu cầu của đơn vị** |

### 5.3 Phân vùng (partition) và chính sách giữa các vùng

| Vùng               | VLAN / zone | Security level (ASA) | Ghi chú                                    |
| ------------------ | ----------- | -------------------- | ------------------------------------------ |
| Internet (outside) | Outside     | 0                    | Không tin cậy                              |
| DMZ                | VLAN 50     | 50                   | Web Proxy, Web Server, DNS/Email công khai |
| Server farm        | VLAN 40     | Inside               | DNS/DHCP, App, DB, File, CME, WLC          |
| Mạng người dùng    | VLAN 10, 20 | Inside               | Nhân viên                                  |
| Khách              | VLAN 60     | Inside (hạn chế)     | Chỉ ra Internet                            |
| Camera             | VLAN 70     | Inside (hạn chế)     | Chỉ nói chuyện với NVR                     |
| Thiết bị y tế      | VLAN 30     | Inside (hạn chế)     | Theo yêu cầu nhà sản xuất                  |
| Quản trị           | VLAN 90     | Inside               | Syslog, AAA, Jump Host, AP                 |
| WAN                | Zone WAN    | 60                   | Chi nhánh về HQ                            |

**Ma trận cho phép (tóm tắt):**

| Từ ↓ \ Đến →   | Internet          | DMZ                           | Server farm           | Người dùng                    | Quản trị        |
| -------------- | ----------------- | ----------------------------- | --------------------- | ----------------------------- | --------------- |
| **Internet**   | –                 | Chỉ HTTP/HTTPS tới web server | Chặn                  | Chặn                          | Chặn            |
| **DMZ**        | Cho phép (có NAT) | –                             | Chặn                  | Chặn                          | Chỉ gửi syslog  |
| **Người dùng** | Cho phép (NAT)    | Cho phép (HTTP)               | Chỉ dịch vụ cần thiết | Giữa các nhóm theo chính sách | Chặn            |
| **Khách**      | Cho phép (NAT)    | Chặn                          | Chặn                  | Chặn                          | Chặn            |
| **Camera**     | Chặn              | Chặn                          | Chặn                  | Chặn                          | Chỉ syslog      |
| **Quản trị**   | Cho phép          | Cho phép                      | Cho phép              | Cho phép                      | –               |
| **Chi nhánh**  | Qua HQ            | Cho phép (HTTP)               | Dịch vụ cần thiết     | –                             | Theo chính sách |

---

### 5.4 Cấu trúc vùng biên đã chọn (topology cuối)

Dùng mô hình **screened subnet (DMZ giữa hai firewall)**:

```text
Internet ── ISP1-R ── MAIN-FW-02 ── DMZ-SW ── MAIN-FW-01 ──┬─ inside (Core, VLAN 250/253)
                       (biên)        (VLAN 50)     (nội bộ)    ├─ wan        → MAIN-WAN1-SW → MAIN-WAN-R1 → Carrier-R (leased line)
                                                               └─ backup-wan → MAIN-WAN2-SW → MAIN-WAN-R2 → ISP2-R (broadband)
```

- Hai firewall **không nối trực tiếp**; mọi lưu lượng giữa chúng đi qua `DMZ-SW`.
- Lưu lượng Internet ↔ nội bộ phải qua cả hai firewall; lưu lượng WAN đi qua `MAIN-FW-01`.
- Hai firewall hiện là **hai firewall độc lập**, chưa phải cặp HA (Packet Tracer không hỗ trợ failover).

---

## 6. Liên kết với thiết kế đã có

| Phần trong Step 1       | Tài liệu / hình đã làm                                    |
| ----------------------- | --------------------------------------------------------- |
| Cấu trúc mạng           | Topology cuối cùng trong `final.pkt` và `main.pdf`        |
| Địa chỉ, VLAN           | `Network Assignment 2 (1).md`, `bang-dia-chi-thiet-bi.md` |
| Thiết bị từng vùng      | `step-2-thiet-bi-ip-cabling.md`                           |
| Phân vùng và chính sách | Mục 5.3 ở trên                                            |

## 7. Việc còn lại cần bạn cung cấp

1. Kết quả khảo sát thật (checklist mục 2): số tầng, số người, số camera, băng thông WAN.
2. Xác nhận Main Site là một tòa nhà hay nhiều tòa nhà (quyết định 2 tầng hay 3 tầng).
3. Chuẩn tuân thủ cụ thể bạn muốn trích dẫn trong báo cáo.
