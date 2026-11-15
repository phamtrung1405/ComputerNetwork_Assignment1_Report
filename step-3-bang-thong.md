# Step 3 – Tính lưu lượng yêu cầu, băng thông ISP dự kiến và cấu hình đề xuất

## 1. Số liệu đầu vào từ đề bài

| Nguồn lưu lượng                       | Tải xuống (MB/ngày) | Tải lên (MB/ngày) | Số lượng                             |
| ------------------------------------- | ------------------- | ----------------- | ------------------------------------ |
| Server (cập nhật, web, cơ sở dữ liệu) | 1000 / server       | 2000 / server     | Main: 10; mỗi chi nhánh: 2           |
| Máy trạm                              | 500 / máy           | 100 / máy         | Main: 600; mỗi chi nhánh: 260        |
| Thiết bị Wi-Fi của khách (tải xuống)  | 500 / thiết bị      | –                 | **Giả định** (đề không nêu số lượng) |

- Giờ cao điểm: **9h–11h và 15h–16h = 3 giờ/ngày**, chiếm **80%** lưu lượng ngày.
- Tăng trưởng **20% trong 5 năm**.

**Giả định số thiết bị Wi-Fi khách:** Main **300**, mỗi chi nhánh **100** (cần xác nhận bằng khảo sát).

## 2. Công thức

```text
Băng thông cao điểm (Mbps) = MB/ngày × 8 × 0,8 ÷ (3 × 3600)
Băng thông thiết kế        = băng thông cao điểm × 1,2
```

Ví dụ: Main tải xuống từ máy trạm và server = (600×500 + 10×1000) = 310 000 MB/ngày → 310 000 × 8 × 0,8 ÷ 10 800 ≈ **183,7 Mbps**.

## 3. Kết quả tính

### 3.1 Theo từng site

| Site | Hướng                         | MB/ngày | Cao điểm (Mbps) | Sau tăng trưởng ×1,2 (Mbps) |
| ---- | ----------------------------- | ------- | --------------- | --------------------------- |
| Main | Tải xuống (máy trạm + server) | 310 000 | 183,7           | **220,4**                   |
| Main | Tải lên                       | 80 000  | 47,4            | **56,9**                    |
| Main | Wi-Fi khách (300 thiết bị)    | 150 000 | 88,9            | **106,7**                   |
| Main | **Tổng**                      | 540 000 | 320,0           | **384,0**                   |
| DBP  | Tải xuống                     | 132 000 | 78,2            | **93,9**                    |
| DBP  | Tải lên                       | 30 000  | 17,8            | **21,3**                    |
| DBP  | Wi-Fi (100 thiết bị)          | 50 000  | 29,6            | **35,6**                    |
| DBP  | **Tổng**                      | 212 000 | 125,6           | **150,8**                   |
| BHTQ | **Tổng** (như DBP)            | 212 000 | 125,6           | **150,8**                   |

### 3.2 Tổng hợp theo liên kết

| Liên kết                       | Cách tính                                                     | Nhu cầu thiết kế | **Băng thông đề xuất**                                                                       |
| ------------------------------ | ------------------------------------------------------------- | ---------------- | -------------------------------------------------------------------------------------------- |
| WAN Main ↔ DBP (leased line)   | Toàn bộ lưu lượng chi nhánh về Main (dịch vụ tập trung)       | ≈ 151 Mbps       | **200 Mbps** (có dư), + broadband dự phòng 100–200 Mbps                                      |
| WAN Main ↔ BHTQ (leased line)  | Như trên                                                      | ≈ 151 Mbps       | **200 Mbps**, + broadband dự phòng                                                           |
| Internet của Main              | Main 384 + DBP 151 + BHTQ 151 (đề bài: mọi Internet qua Main) | ≈ 686 Mbps       | **2 đường × 500 Mbps** (load-balancing, mỗi đường đủ chịu khi đường kia hỏng ở mức giảm cấp) |
| Uplink Access → Core (mỗi tòa) | Tải máy trạm một tòa ≈ 300 máy                                | ≈ 100–150 Mbps   | **2×1 GbE (EtherChannel)**, oversubscription chấp nhận được                                  |
| Server farm                    | Nhiều người dùng truy cập đồng thời                           | < 1 Gbps         | Server nối **1 GbE**, uplink 2×1 GbE; nâng 10 GbE nếu PACS lớn                               |

> **Lưu ý:** cách tính này **cộng toàn bộ lưu lượng** gồm cả lưu lượng nội bộ, nên là cận trên an toàn. Nếu chỉ một phần đi Internet, băng thông Internet có thể thấp hơn. DSL thường không đạt 500 Mbps; mức này giả định cáp quang doanh nghiệp. Nếu dùng DSL thực sự, phải bổ sung nhiều đường hơn.

## 4. Cấu hình đề xuất cho mạng bệnh viện

| Hạng mục | Đề xuất                                                                                                 |
| -------- | ------------------------------------------------------------------------------------------------------- |
| Cấu trúc | Collapsed core (2 Core), HSRP, Rapid-PVST, LACP                                                         |
| WAN      | Leased line/MPLS chính 200 Mbps mỗi chi nhánh + broadband dự phòng; OSPF, floating static               |
| Internet | Hai nhà mạng, load-balancing/failover; mọi lưu lượng qua MAIN-FW-02                                      |
| Firewall | Hai tầng (MAIN-FW-02 biên, MAIN-FW-01 nội bộ), DMZ ở giữa; chọn thiết bị theo throughput khi bật kiểm tra |
| QoS      | Ưu tiên thoại (VLAN 80) và HIS/PACS; giới hạn băng thông Guest                                          |
| VPN      | Site-to-site IPsec giữa router WAN; remote-access cho teleworker (pool `10.10.200.0/24`)                |
| Giám sát | Syslog/NTP tại VLAN 90; SNMP; lưu log tập trung                                                         |

## 5. Giới hạn của phần tính

- Số thiết bị Wi-Fi là giả định; kết quả sẽ thay đổi theo khảo sát.
- Chưa tính lưu lượng ảnh y khoa (PACS/RIS), thoại, camera; nếu có, phải cộng thêm (camera: số camera × bitrate, xem Step 1 mục 3.3).
- Chưa tính overhead IPsec (thường 10–20%).
