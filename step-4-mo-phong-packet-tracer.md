# Step 4 – Thiết kế bản đồ mạng trên Cisco Packet Tracer

Phần mềm: **Cisco Packet Tracer 9.0.1**, file mô phỏng `final.pkt`. Tên thiết bị theo Bảng 1 của `main.pdf`.

## 1. Tổng quan topology

Gồm 25 thiết bị mạng trên 4 vùng:

| Vùng                        | Thiết bị                                                                    |
| --------------------------- | --------------------------------------------------------------------------- |
| Main Site – lõi và truy cập | MAIN-CORE-SW1/2, Server-SW, Mgmt-SW, Cam-SW, MAIN-Access-A/B |
| Main Site – biên, DMZ, WAN  | MAIN-FW-01, MAIN-FW-02, DMZ-SW, MAIN-WAN1-SW/2, MAIN-WAN-R1/R2, ISP1-R   |
| Backbone WAN                | Carrier-R (leased line/MPLS giả lập), ISP2-R (broadband giả lập)            |
| Chi nhánh                   | DBP-R1/R2, DBP-SW1/SW2; BHTQ-R1/R2, BHTQ-SW1/SW2                            |

Hai firewall nối tiếp (screened subnet): `Internet – ISP1-R – MAIN-FW-02 – DMZ-SW – MAIN-FW-01 – Core/WAN`.

## 2. Các tính năng đã mô phỏng

| Nhóm          | Tính năng                                                                                                | Thiết bị chính                      |
| ------------- | -------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| Lớp 2         | VLAN, trunk 802.1Q, Rapid-PVST, LACP Port-channel 1 giữa hai Core                                        | Core, Access                        |
| Dự phòng cổng | HSRP (Core: prio 200/150; chi nhánh: 110/105, `standby track Serial0/3/0`)                               | MAIN-CORE-SW1/2, DBP/BHTQ-R1/R2          |
| Định tuyến    | OSPF Area 0 (leased line, cost 10); static + default; floating static AD 150 (chi nhánh) và AD 10 (MAIN-FW-01) | WAN-R1, Carrier-R, branch routers   |
| Firewall      | Zone và security level: inside 100, inside-backup 100, wan 70, backup-wan 60, dmz 50, outside 0          | MAIN-FW-01, MAIN-FW-02                |
| NAT           | Static NAT `10.10.50.10` ↔ `203.162.10.3` (TCP/80); PAT cho `10.0.0.0/8` ra Internet                     | MAIN-FW-02                           |
| ACL           | `ACL-GUEST-ISOLATE` cho VLAN 60 (cần áp dụng lại, xem Step 5/6)                                          | Core                                |
| DHCP          | DHCP relay (`ip helper-address 10.10.40.10`) từ SVI/subinterface                                         | Core, branch routers                |
| VPN           | IPsec site-to-site giữa router WAN; remote-access VPN (client nhận `10.10.200.10`)                       | WAN routers, router/ASA VPN, REMOTE |
| Không dây     | LAP-PT, WLC-PT, SSID staff và guest                                                                      | Access, Server farm                 |

## 3. Cấu hình quan trọng (trích)

### 3.1 HSRP và tracking ở chi nhánh

```text
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.20.10.2 255.255.255.0
 ip helper-address 10.10.40.10
 standby version 2
 standby 10 ip 10.20.10.1
 standby 10 priority 110
 standby 10 preempt
 standby 10 track Serial0/3/0 60
```

R2: `standby 10 priority 105`, không cần `track`.

### 3.2 OSPF trên liên kết leased line

```text
router ospf 1
 router-id 3.3.3.1
 network 10.255.0.8 0.0.0.3 area 0
 network 10.20.0.0 0.0.255.255 area 0
!
interface Serial0/3/0
 ip ospf cost 10
```

### 3.3 Đường dự phòng (floating static)

```text
! Trên DBP-R2
ip route 0.0.0.0 0.0.0.0 10.255.0.17 150
! Trên MAIN-FW-01
route backup-wan 10.20.0.0 255.255.0.0 10.10.252.4 10
route wan        10.20.0.0 255.255.0.0 10.10.251.3 1
```

### 3.4 NAT trên MAIN-FW-02

```text
object network WEB-SERVER
 host 10.10.50.10
 nat (dmz,outside) static 203.162.10.3
object network INTERNAL-NET
 subnet 10.0.0.0 255.0.0.0
 nat (dmz,outside) dynamic interface
route outside 0.0.0.0 0.0.0.0 203.162.10.1
route dmz 10.0.0.0 255.0.0.0 10.10.50.1
```

Cấu hình đầy đủ nằm trong `running-config` các thiết bị và file `cau-hinh-theo-dia-chi-moi.txt` (bản cũ, tên thiết bị chưa đổi sang `MAIN-`).

## 4. Hạn chế của Packet Tracer ảnh hưởng thiết kế

| Yêu cầu thiết kế đích  | Tình trạng trong PT | Cách xử lý                                         |
| ---------------------- | ------------------- | -------------------------------------------------- |
| Failover firewall (HA) | Không có            | Hai ASA độc lập, ghi vào "vấn đề còn lại"          |
| IPS/IDS                | Không có            | Ghi trong thiết kế đích, không mô phỏng            |
| Load balancer          | Không có            | Đa Server-PT hoặc DNS round-robin; ghi thuyết minh |
| SD-WAN / MPLS thật     | Không có            | Mô phỏng bằng leased line + floating static        |
| Subinterface trên ASA  | Không dùng          | Mỗi vùng một cổng vật lý (5506-X có 8 cổng)        |

## 5. Mapping tên thiết bị đầu cuối

Cần danh sách tên thật trong `final.pkt`. Báo cáo dùng: `MAIN-PC-STAFFA`, `PC_DATA`, `MAIN-B-GUEST`, `REMOTE`, `DBP-PC1`, `PC-BHTQ1`, `PC-BHTQ3`, Server DBP `10.20.40.10`, Server BHTQ `10.30.40.10`. Các tên này **không thống nhất** về định dạng; nên thống nhất lại trước khi nộp.

## 6. Các lỗi mô phỏng đã xử lý (từ báo cáo)

| Lỗi                                 | Nguyên nhân                                            | Cách sửa                                                                                               |
| ----------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| BHTQ-R2 không thấy `10.20.40.0/24`  | Default static AD 1 đè OSPF → bất đối xứng             | Đổi thành floating AD 150, bỏ `default-information originate`, `passive-interface` trên cổng broadband |
| OSPF kẹt INIT/DROTHER qua VLAN 99   | `dot1Q native` kết hợp HSRP/OSPF trên subinterface     | Bỏ `native`, dùng `dot1Q 99`, native VLAN của switch về 1                                              |
| HSRP nhóm 90 báo Speak              | Thiếu subinterface Gi0/0.90 và allowed VLAN trên trunk | Khai báo đồng bộ trên cả R1/R2, mở trunk VLAN 1, 10, 30, 40, 70, 90, 99                                |
| Không tới Server BHTQ `10.30.40.10` | Server thiếu default gateway                           | Đặt gateway `10.30.40.1`, gán access VLAN 40                                                           |

## 7. Việc cần làm

1. Chụp hình topology cuối (thiết bị và nhãn cổng) để chèn vào báo cáo.
2. Đồng bộ tên thiết bị đầu cuối.
3. Đối chiếu hostname trong `running-config` (`MAIN-FW-01`, `MAIN-FW-02`, `WAN-R1`, `WAN-R2`) với tên `MAIN-*`.
