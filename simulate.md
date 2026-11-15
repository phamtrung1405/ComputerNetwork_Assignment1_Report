# TÀI LIỆU MÔ PHỎNG & KỊCH BẢN DEMO HỆ THỐNG MẠNG DOANH NGHIỆP ĐA CHI NHÁNH

**Dự án:** Thiết kế, Triển khai và Kiểm thử Hệ thống Mạng Doanh nghiệp Đa chi nhánh (Main Site – DMZ/Internet – Chi nhánh DBP – Chi nhánh BHTQ)  
**Nền tảng mô phỏng:** Cisco Packet Tracer (Phiên bản 8.2 trở lên)

---

## 1. HƯỚNG DẪN MỞ FILE & THIẾT LẬP MÔI TRƯỜNG MÔ PHỎNG

### 1.1. Quy trình mở file & Khởi động hệ thống
1. **Khởi chạy phần mềm:** Mở ứng dụng **Cisco Packet Tracer** (khuyến nghị bản 8.2.x trở lên để tương thích đầy đủ với dòng tường lửa ASA 5506-X OS 9.6 và Router 2911 IOS 15.1).
2. **Mở tệp dự án:** Chọn **File $\rightarrow$ Open** (hoặc tổ hợp phím `Ctrl + O`), trỏ tới tệp mô phỏng phiên bản nghiệm thu mới nhất (ví dụ: `Enterprise_MultiSite_v2.0_Final_Verified.pkt`).
3. **Tua nhanh thời gian hội tụ (Fast Forward Time):**
   * Ngay khi vừa mở file, các giao thức Spanning Tree (Rapid-PVST+), EtherChannel (LACP), HSRP và OSPF cần khoảng 30–50 giây thực tế để đàm phán trạng thái.
   * Bấm tổ hợp phím **`Alt + D`** (hoặc bấm nút *Fast Forward Time* ở góc dưới bên trái) **2–3 lần** cho đến khi toàn bộ các biểu tượng liên kết chuyển từ màu cam sang **tam giác xanh lá**.
4. **Thông tin đăng nhập thiết bị:**
   * **Console / Enable Password:** `[REDACTED]`; thay bằng mật khẩu riêng, đủ mạnh khi chạy mô phỏng.
   * **Tài khoản quản trị SSH:** Username `[ADMIN_USER]` | SSH Version `2` | Chỉ cho phép truy cập từ máy trạm quản trị `JUMP_HOST` (`10.10.90.30` thông qua `access-list 10`). Không ghi mật khẩu vào tài liệu.

### 1.2. Quy tắc quản lý phiên bản & An toàn dữ liệu
* Trước khi thực hiện bất kỳ thao tác ngắt mạch (`shutdown`) hoặc thay đổi định tuyến nào trong lúc Demo, luôn sao lưu tệp sang phiên bản mới bằng **File $\rightarrow$ Save As...** (ví dụ: `v2.0_Final_Verified.pkt` $\rightarrow$ `v2.1_Demo_Backup.pkt`).
* Sau khi kết thúc các bài kiểm thử sự cố, bắt buộc khôi phục trạng thái cổng về `no shutdown` và lưu cấu hình vào NVRAM bằng lệnh `write memory` (hoặc `copy running-config startup-config`).

---

## 2. KIẾN TRÚC TOPOLOGY & CÁC GIẢ ĐỊNH THIẾT KẾ

### 2.1. Sơ đồ phân vùng & Quy hoạch địa chỉ IP (25 thiết bị mạng)

| Phân vùng / Site | Thiết bị chính | Quy hoạch Subnet & VLAN | Vai trò & Giao thức cốt lõi |
| :--- | :--- | :--- | :--- |
| **Main Site (HCMC)** | `Core-SW1` (Active), `Core-SW2` (Standby), `Server-SW`, `Mgmt-SW`, `Cam-SW`, `Access-A`, `Access-B` | `10.10.0.0/16`<br>• VLAN 10: Staff-A (`10.10.10.0/23`)<br>• VLAN 20: Staff-B (`10.10.20.0/23`)<br>• VLAN 30: Medical (`10.10.30.0/24`)<br>• VLAN 40: Server Farm (`10.10.40.0/24`)<br>• VLAN 60: Guest WiFi (`10.10.60.0/22`)<br>• VLAN 70: Camera (`10.10.70.0/24`)<br>• VLAN 80: Voice (`10.10.80.0/24`)<br>• VLAN 90: Management (`10.10.90.0/24`)<br>• VLAN 99: Native Trunk | • **Core-SW1:** HSRP Active (Priority 200), STP Root Bridge (Priority 4096), OSPF RID `2.2.2.1`.<br>• **Core-SW2:** HSRP Standby (Priority 150), OSPF RID `2.2.2.2`.<br>• **Port-channel 1:** LACP Active nối `Core-SW1` $\leftrightarrow$ `Core-SW2`.<br>• **ACL-GUEST-ISOLATE:** Cô lập VLAN 60 không truy cập nội bộ `10.0.0.0/8`. |
| **Biên & DMZ / Internet** | `ASA1`, `ASA2`, `DMZ-SW`, `WAN-R1`, `WAN-R2`, `WAN1-SW`, `WAN2-SW`, `ISP1-R` | • Transit Inside: VLAN 250 (`10.10.250.0/29`)<br>• Inside Backup: VLAN 253 (`10.10.253.0/29`)<br>• DMZ: `10.10.50.0/24`<br>• WAN1: `10.10.251.0/29` | Backup WAN: `10.10.252.0/29`<br>• Public ISP1: `203.162.10.0/29` | • **ASA1:** Phân tách 5 vùng bảo mật (`inside`, `inside-backup`, `dmz`, `wan`, `backup-wan`).<br>• **ASA2:** Thực hiện Static NAT cho Web Server (`10.10.50.10` $\leftrightarrow$ `203.162.10.3`) và Dynamic PAT ra Internet. |
| **Mạng truyền dẫn WAN** | `Carrier-R` (Chính), `ISP2-R` (Dự phòng) | • Leased Line: `10.255.0.0/30` (WAN-R1), `10.255.0.4/30` (WAN-R2), `10.255.0.8/30` (DBP-R1), `10.255.0.12/30` (BHTQ-R1)<br>• Broadband: `10.255.0.16/30` (DBP-R2), `10.255.0.20/30` (BHTQ-R2), `10.255.0.24/30` (WAN-R2) | • **Carrier-R:** Chạy OSPF Area 0 (`distance 15`, `point-to-point`) + BGP AS 65000.<br>• **ISP2-R:** Mạng dự phòng Broadband chạy định tuyến tĩnh & NAT Overload. |
| **Chi nhánh 1 (DBP)** | `DBP-R1` (Chính), `DBP-R2` (Dự phòng), `DBP-SW1`, `DBP-SW2` | `10.20.0.0/16`<br>• VLAN 10, 30, 40, 70, 90, 99 (`10.20.x.0/24`)<br>• Trunk: native VLAN 1; VLAN 99 tagged | • **DBP-R1:** HSRP Active (Priority 110, `track Serial0/3/0`), OSPF Cost 10 qua `Carrier-R`.<br>• **DBP-R2:** HSRP Standby (Priority 105), Floating Static Route (`AD 150`) qua `ISP2-R`. |
| **Chi nhánh 2 (BHTQ)** | `BHTQ-R1` (Chính), `BHTQ-R2` (Dự phòng), `BHTQ-SW1`, `BHTQ-SW2` | `10.30.0.0/16`<br>• VLAN 10, 30, 40, 70, 90, 99 (`10.30.x.0/24`)<br>• Trunk: native VLAN 1; VLAN 99 tagged | • **BHTQ-R1:** HSRP Active (Priority 110, `track Serial0/3/0`), OSPF Cost 10 qua `Carrier-R`.<br>• **BHTQ-R2:** HSRP Standby (Priority 105), Floating Static Route (`AD 150`) qua `ISP2-R`. |

### 2.2. Các giả định thiết kế (Design Assumptions)
1. **Giả định hạ tầng truyền dẫn nhà mạng:**
   * Mạng `Carrier-R` giả lập kênh thuê riêng tốc độ cao (Leased Line / MPLS L3VPN) với độ trễ thấp (OSPF Cost = 10) đóng vai trò đường truyền chính giữa Main Site và hai chi nhánh.
   * Mạng `ISP2-R` giả lập đường truyền Internet băng thông rộng (FTTH/Broadband) đóng vai trò kênh dự phòng (Backup WAN) thông qua Floating Static Route có Administrative Distance (`AD = 150`) cao hơn OSPF (`AD = 110`) để tránh xung đột định tuyến bất đối xứng.
2. **Giả định dịch vụ tập trung:**
   * Hệ thống máy chủ DNS/DHCP tập trung của Main Site đặt tại `10.10.40.10` (VLAN 40), các chi nhánh sử dụng `ip helper-address` (`10.20.40.10` tại DBP và `10.30.40.10` tại BHTQ) hoặc DHCP Pool dự phòng cục bộ trên Router.
   * Máy chủ thu thập nhật ký tập trung (Syslog/NTP) đặt tại `10.10.90.20` và trạm nhảy quản trị duy nhất (Jump Host) được cấp quyền SSH toàn mạng có IP cố định `10.10.90.30`.
3. **Giả định chuẩn hóa Trunking & Subinterface:**
   * Trên các liên kết nội bộ chi nhánh chạy đồng thời HSRP và OSPF (`Gi0/0.99`), sử dụng chuẩn gắn thẻ 802.1Q tường minh (`encapsulation dot1Q 99`, bỏ từ khóa `native` và đồng bộ `native vlan 1` trên Switch) để đảm bảo gói tin multicast hội tụ hai chiều ổn định.

---

## 3. CÁC TÍNH NĂNG CHỈ MÔ PHỎNG MỘT PHẦN HOẶC CHƯA KIỂM CHỨNG

Do giới hạn kiến trúc của công cụ mô phỏng sự kiện **Cisco Packet Tracer** so với thiết bị phần cứng thực tế (Cisco IOS/ASA OS thật), một số tính năng chỉ được triển khai ở mức thay thế tương đương hoặc chưa thể kiểm chứng đầy đủ:

| STT | Tính năng / Giao thức | Mức độ đáp ứng trên Packet Tracer | Giải thích giới hạn kỹ thuật |
| :--- | :--- | :--- | :--- |
| **1** | **IP SLA Tracking kết hợp Static Route trên Cisco ASA** | **Mô phỏng một phần** *(Dùng Metric tuyến tĩnh thay thế)* | Hệ điều hành ASA 9.6 trên Packet Tracer không hỗ trợ đầy đủ bộ lệnh giám sát chủ động (`sla monitor`, `track ... rtr`) để tự động gỡ tuyến khi mất kết nối đầu xa. Do đó, hệ thống sử dụng cơ chế định tuyến tĩnh dự phòng theo trọng số Metric (`Metric 1` qua `wan` và `Metric 10` qua `backup-wan`). |
| **2** | **Stateful HA Failover (Active/Standby) giữa hai tường lửa ASA** | **Chưa kiểm chứng / Không hỗ trợ** | Packet Tracer không hỗ trợ giao thức đồng bộ bảng trạng thái phiên kết nối (`xlate`/`conn` state) qua cổng Failover Link giữa hai thiết bị ASA. Kiến trúc phải thiết kế `ASA1` (tường lửa biên nội bộ/WAN) và `ASA2` (tường lửa biên Internet/DMZ) hoạt động độc lập theo phân lớp vùng. |
| **3** | **Tùy chỉnh kiểu mạng OSPF trên Subinterface (`point-to-point`)** | **Mô phỏng một phần** *(Chạy chế độ Broadcast mặc định)* | IOS 15.1 trên Packet Tracer chỉ hỗ trợ lệnh `ip ospf network point-to-point` trên cổng vật lý (như `Serial0/3/0`), báo lỗi cú pháp khi nhập trên subinterface 802.1Q (`Gi0/0.99`). Các subinterface phải vận hành ở chế độ mặc định `BROADCAST` (có bầu chọn DR/BDR). |
| **4** | **Chạy đồng thời HSRP và OSPF trên Native VLAN Subinterface** | **Giới hạn giả lập** *(Đã xử lý bằng Tagged VLAN)* | Khi cấu hình `encapsulation dot1Q 99 native` kết hợp đồng thời HSRP (`224.0.0.2`) và OSPF (`224.0.0.5`), bộ mô phỏng dễ gặp lỗi rớt khung tin không gắn thẻ một chiều (treo `INIT/DROTHER`). Hệ thống đã khắc phục bằng cách chuyển VLAN 99 sang gắn thẻ chuẩn (`encapsulation dot1Q 99`). |
| **5** | **Định tuyến theo chính sách (PBR) & BFD (Bidirectional Forwarding Detection)** | **Chưa kiểm chứng / Không hỗ trợ** | Packet Tracer không hỗ trợ `route-map` phức tạp cho PBR hay giao thức phát hiện đứt tuyến siêu tốc BFD (< 1 giây). Thời gian hội tụ dự phòng phụ thuộc vào bộ định thời của HSRP (Hello 3s / Hold 10s) và OSPF (Hello 10s / Dead 40s). |
| **6** | **Kiểm tra gói tin nâng cao NGFW (IPS/Firepower) & AAA TACACS+ chi tiết** | **Mô phỏng một phần** | Tường lửa chỉ dừng ở mức Stateful Packet Inspection cơ bản (`inspect dns/ftp/icmp/tftp`) và Extended ACL; chưa mô phỏng được tính năng lọc nội dung sâu (Deep Packet Inspection / IPS) và phân quyền từng câu lệnh (Command Authorization) qua TACACS+. |

---

## 4. KỊCH BẢN DEMO HỆ THỐNG (SYSTEM DEMO SCRIPT)

Kịch bản dưới đây được thiết kế theo trình tự 5 phần (thời lượng dự kiến 15–20 phút) nhằm chứng minh toàn diện khả năng vận hành, bảo mật và tự động phục hồi sau sự cố của hệ thống.

### Giai đoạn 1: Kiểm tra trạng thái vận hành chuẩn (Normal Operation)
* **Mục tiêu:** Kiểm tra trạng thái HSRP/OSPF trên các thiết bị và tuyến được nêu trong từng bước; không suy rộng các kết quả này thành kiểm thử đầy đủ cả 25 thiết bị.
* **Thao tác thực hiện:**
  1. Trên **`Core-SW1`**, **`DBP-R1`**, **`BHTQ-R1`**: Chạy `show standby brief` $\rightarrow$ Xác nhận các thiết bị chính đều ở trạng thái **`Active`**, các thiết bị dự phòng (`Core-SW2`, `DBP-R2`, `BHTQ-R2`) ở trạng thái **`Standby`**.
  2. Trên **`BHTQ-R2`** và **`DBP-R2`**: Chạy `show ip ospf neighbor` $\rightarrow$ Xác nhận trạng thái láng giềng đạt **`FULL`**.
  3. Mở Command Prompt trên máy trạm **`PC-BHTQ1`** (`10.30.10.x` - Chi nhánh 2), thực hiện ping liên vùng:
     * `ping 10.10.40.10` (Tới Server Farm Main Site) $\rightarrow$ **Thành công 100%**.
     * `ping 10.20.40.10` (Tới Server Chi nhánh 1 DBP) $\rightarrow$ **Thành công 100%**.
     * `tracert 10.20.40.10` $\rightarrow$ Xác nhận gói tin đi qua tuyến chính **`10.30.10.2` (BHTQ-R1) $\rightarrow$ `10.255.0.13` (Carrier-R) $\rightarrow$ `10.255.0.10` (DBP-R1) $\rightarrow$ `10.20.40.10`**.

### Giai đoạn 2: Demo Bảo mật Phân vùng, NAT & Kiểm soát Truy cập (Security & Access Control)
* **Mục tiêu:** Thực hiện các phép thử riêng cho Guest, Web DMZ và SSH quản trị. Các kết quả này không thay thế toàn bộ test plan SEC.
* **Thao tác thực hiện:**
  1. **Cô lập mạng khách (Guest WiFi Isolation):** Báo cáo mô phỏng ghi nhận ping từ VLAN 60 (`LAP_PT`) tới `10.10.40.10` bị chặn. ACL đã được chỉnh để chỉ cho DNS tới máy chủ này và TCP/80, TCP/443 ra Internet; đích `10.10.50.10` thuộc `10.0.0.0/8` nên cũng phải bị chặn. **Chạy lại Guest-to-LAN, Guest-to-DMZ và Guest-to-Internet sau khi áp dụng ACL; chưa đánh dấu PASS cho cấu hình đã chỉnh nếu chưa có output mới.**
  2. **Truy cập dịch vụ DMZ từ Internet (Static NAT):** Báo cáo mô phỏng ghi nhận từ máy khách ngoài Internet (qua `ISP1-R`) truy cập **`http://203.162.10.3`** thành công; `ASA2` ánh xạ tới **`10.10.50.10`** qua HTTP/80. Bổ sung ảnh chụp hoặc log NAT/ACL để lưu bằng chứng.
  3. **Bảo mật quản trị (SSH & Jump Host):** Thử SSH từ một máy nhân viên thông thường (`10.10.10.x`) vào `Core-SW1` $\rightarrow$ **Bị từ chối**; báo cáo mô phỏng ghi nhận SSH từ **`JUMP_HOST` (`10.10.90.30`)** với tài khoản `[ADMIN_USER]` đăng nhập thành công. Không ghi mật khẩu; cần lưu ảnh chụp/output SSH làm bằng chứng.

### Giai đoạn 3: Demo Chuyển đổi dự phòng WAN tại Chi nhánh (Branch WAN Failover)
* **Mục tiêu:** Chứng minh khi đứt cáp Leased Line chính, chi nhánh tự động chuyển sang đường Broadband `ISP2-R` mà không cần can thiệp thủ công.
* **Thao tác thực hiện:**
  1. Trên máy **`PC-BHTQ3`** (Chi nhánh 2), mở Command Prompt chạy lệnh ping liên tục: `ping 10.20.40.10 -t`.
  2. Mở CLI của **`BHTQ-R1`**, giả lập sự cố đứt cáp WAN chính:
     ```
     configure terminal
     interface Serial0/3/0
      shutdown
     end
     ```
  3. Bấm phím tắt **`Alt + D`** một lần để tua nhanh thời gian hội tụ.
  4. **Quan sát kết quả:**
     * Trên `BHTQ-R1`: HSRP Tracking tự động trừ 60 điểm Priority (từ `110` xuống `50`).
     * Trên `BHTQ-R2`: Chạy `show standby brief` $\rightarrow$ Tự động chiếm quyền **`Active`** (Priority `105`).
     * Trên `PC-BHTQ3`: Màn hình ping chỉ rớt 1–2 gói tin trong lúc hội tụ, sau đó tự động **`Reply from 10.20.40.10`** trở lại qua đường dự phòng **`ISP2-R` (`10.255.0.21`)**.
  5. **Khôi phục (Failback):** Vào `BHTQ-R1` gõ `no shutdown` trên `Serial0/3/0`, bấm `Alt + D` $\rightarrow$ `BHTQ-R1` tự động giành lại quyền `Active` nhờ `preempt` và đưa lưu lượng về đường chính `Carrier-R`.

### Giai đoạn 4: Demo Dự phòng Switch Lõi & Bó cáp EtherChannel tại Main Site (Core Redundancy)
* **Mục tiêu:** Chứng minh khả năng chịu lỗi của tầng Core tại trụ sở chính.
* **Thao tác thực hiện:**
  1. Từ máy trạm `PC_DATA` (`Access-A` - VLAN 10), chạy `ping 10.10.40.10 -t`.
  2. **Thử nghiệm 1 (Đứt 1 cáp trong Port-channel 1):** Xóa hoặc tắt cổng `FastEthernet0/2` trên `Core-SW1` $\rightarrow$ Lưu lượng chuyển tức thì sang cổng `FastEthernet0/7` trong bó LACP **không rớt bất kỳ gói tin nào (0% packet loss)**.
  3. **Thử nghiệm 2 (Sự cố Core-SW1):** Tắt các cổng downlink hoặc nguồn của `Core-SW1` $\rightarrow$ `Core-SW2` (Priority `150`) lập tức tiếp quản vai trò `Active` cho toàn bộ 9 VLAN và định tuyến ra `ASA1` qua tuyến dự phòng VLAN 253 (`10.10.253.1`).
  4. Khôi phục lại `Core-SW1` sau khi kết thúc bài test.

### Giai đoạn 5: Tổng kết & Lưu trữ trạng thái nghiệm thu
* Kiểm tra lại toàn bộ bảng định tuyến (`show ip route`), trạng thái HSRP (`show standby brief`) trên các thiết bị chính đảm bảo đã quay về trạng thái chuẩn ban đầu.
* Thực hiện `write memory` trên các thiết bị và lưu tệp `.pkt` nghiệm thu cuối cùng.