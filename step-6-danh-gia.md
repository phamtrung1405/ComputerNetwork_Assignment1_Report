# Step 6 – Đánh giá lại thiết kế, vấn đề còn lại và định hướng phát triển

## 1. Đánh giá theo tiêu chí

| Tiêu chí                    | Đã đạt                                                                                                                           | Còn hạn chế                                                                                                                                                   | Đánh giá        |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| **Độ tin cậy (HA)**         | Core đôi + HSRP + LACP (test 4.1: 0% mất gói); chi nhánh 2 router HSRP + `standby track`; WAN chính/phụ; failover BHTQ rơi 1 gói | Firewall không HA (hai ASA độc lập); `DMZ-SW` là điểm hỏng đơn; một ISP phía Internet; Server-SW/Mgmt-SW/Cam-SW chỉ có một switch                        | Khá             |
| **Dễ nâng cấp, mở rộng**    | Subnet dư (`/23`, `/22`); tóm tắt `/16` theo site; cấu trúc module                                                               | Chi nhánh VLAN 10 `/24` không đủ 260 máy (+20%); chưa dùng quản trị tập trung/automation                                                                      | Khá             |
| **Phần mềm hỗ trợ đa dạng** | Server farm VLAN 40 cho HIS, PACS, LIS, CRM, DB; DMZ cho web; thoại IP (VLAN 80)                                                 | Chưa kiểm thử ứng dụng thật; PT chỉ mô phỏng dịch vụ cơ bản (HTTP, DNS, DHCP, email)                                                                          | Trung bình      |
| **An toàn (safety)**        | Camera VLAN 70 riêng; cô lập Medical VLAN 30; nguồn điện và làm mát nêu ở checklist Step 1                                       | Chưa mô phỏng UPS, chống sét, kiểm soát ra vào (thuộc khảo sát vật lý)                                                                                        | Chưa kiểm chứng |
| **Bảo mật mạng**            | Hai tầng firewall, DMZ, NAT/PAT, security level theo zone, IPsec S2S và remote-access VPN, VLAN quản trị                         | **Guest vẫn ping được DMZ (SEC-01 FAIL)**; DMZ cho phép rộng; chưa có IDS/IPS (PT không hỗ trợ); ACL quản trị chưa áp dụng đầy đủ; chưa có AAA/802.1X thực tế | Cần cải thiện   |
| **Hiệu năng**               | Tính băng thông theo Step 3; EtherChannel                                                                                        | Chưa có QoS đã kiểm thử; chưa đo throughput                                                                                                                   | Chưa kiểm chứng |

## 2. Vấn đề còn lại của dự án

### 2.1 Lỗi/điểm yếu đã biết

| #   | Vấn đề                                                                                                   | Hậu quả                                        | Hướng xử lý                                                                                  |
| --- | -------------------------------------------------------------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------------------- |
| 1   | Guest ping được `10.10.50.10` (SEC-01)                                                                   | Vi phạm yêu cầu "khách không truy cập LAN/DMZ" | Áp dụng `ACL-GUEST-ISOLATE` (có `deny 10.0.0.0/8` trước `permit web`) lên `Vlan60`, chạy lại |
| 2   | `DMZ-SW` là điểm hỏng đơn giữa hai firewall                                                         | Mất DMZ và đường Internet ↔ nội bộ             | Thêm switch thứ hai, hoặc dùng EtherChannel; hoặc ghép hai ASA thành cặp HA (thiết bị thật)  |
| 3   | DMZ cho phép lưu lượng rộng                                                                              | DMZ bị khai thác có thể đi sâu vào mạng        | Chỉ cho phép đúng luồng (HTTP/HTTPS tới Web, DNS), chặn mặc định DMZ → inside                |
| 4   | Một vài ACL đã viết nhưng chưa gắn vào interface                                                         | Chính sách không có hiệu lực                   | Gắn `ip access-group` / `access-group`, kiểm tra bộ đếm `show access-lists`                  |
| 5   | Lỗi/khác biệt route trên MAIN-FW-01 (next-hop, `route wan` / `backup-wan`)                                     | Đường dự phòng có thể không như thiết kế       | Kiểm tra `show route`, thử tắt đường chính                                                   |
| 6   | Trộn OSPF và định tuyến tĩnh/mặc định                                                                    | Định tuyến bất đối xứng, firewall mất state    | Dùng AD nhất quán; mọi đường dự phòng là floating static; redistribute có kiểm soát          |
| 7   | Chỉ có một ISP ở phía Internet (ISP1-R)                                                                  | Không đạt "2×DSL có load-balancing" của đề bài | Thêm ISP thứ hai + PBR/ECMP hoặc SLA tracking trên MAIN-FW-02 (thiết kế đích)                 |
| 8   | VLAN 10 chi nhánh `/24` (254 host) so với 260 máy + 20% = 312                                            | Cạn địa chỉ                                    | Đổi sang `/23` hoặc tách thêm VLAN                                                           |
| 9   | Không có load balancer, IPS/IDS, phishing detection trong PT                                             | Không đạt đầy đủ yêu cầu "high security"       | Nêu trong thiết kế đích: Firepower, WAF/LB, email gateway                                    |
| 10  | `inside-backup` (`Gi1/5`) đang shutdown                                                                  | Chưa test được failover MAIN-CORE-SW1 hỏng          | `no shutdown`, kiểm thử Step 5 mục 3.3                                                       |
| 11  | Khác biệt giữa báo cáo và cấu hình (DHCP chi nhánh, decrement track, native VLAN 99, cổng Serial WAN-R2) | Tài liệu và mô phỏng không đồng nhất           | Đối chiếu với `final.pkt` rồi sửa một nơi                                                    |
| 12  | Tên thiết bị đầu cuối và hostname ASA/WAN-R chưa thống nhất                                              | Báo cáo khó đọc                                | Đặt tên theo quy ước `<SITE>-<loại>-<số>`; `hostname MAIN-FW-01`...                           |

### 2.2 Phạm vi chưa làm (talented engineer)

Đề yêu cầu cho chương trình talented engineer: kế hoạch bảo mật, ACL, IDS/IPS, hướng dẫn best practice. Hiện có kế hoạch, ACL và VPN; **chưa có IDS/IPS** và best-practice guide hoàn chỉnh.

## 3. Định hướng phát triển

| Giai đoạn | Hạng mục                                                                                                                                        |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Ngắn hạn  | Sửa SEC-01; gắn các ACL; thống nhất tên; hoàn thiện kiểm thử Step 5; bổ sung bằng chứng                                                         |
| Trung hạn | Firewall HA (Active/Standby); hai nhà mạng Internet; QoS; 802.1X + RADIUS; Syslog/SNMP tập trung                                                |
| Dài hạn   | SD-WAN thay hoặc bổ sung leased line; IPS/IDS (Firepower); WAF + load balancer; NAC; tự động hóa cấu hình (Ansible); 10/40 GbE cho Core và PACS |

## 4. Kết luận ngắn

Thiết kế đạt phần lớn yêu cầu về phân cấp, dự phòng, phân vùng và WAN. Các điểm cần xử lý trước khi nộp: **SEC-01**, kiểm thử còn thiếu (cùng VLAN, Internet → DMZ qua NAT, SSH, camera), tính đồng nhất giữa báo cáo và `.pkt`. Các yêu cầu IPS/IDS, load balancer, HA firewall, hai ISP được trình bày như **thiết kế đích**, không phải kết quả mô phỏng.
