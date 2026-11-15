# Step 5 – Kiểm thử hệ thống bằng ping, traceroute và các công cụ khác

> Kết quả bên dưới **lấy từ báo cáo `main.pdf`** (các ảnh `graphics/tc_pic`). Mục không có ảnh bằng chứng được đánh dấu **Chưa kiểm thử**; không được ghi PASS khi chưa chạy.

## 1. Sáu kiểm tra bắt buộc của đề bài

| #   | Yêu cầu                      | Lệnh gợi ý                                                             | Kết quả trong báo cáo                                                                                    | Trạng thái                                              |
| --- | ---------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| 1   | PC cùng VLAN                 | `ping` giữa hai PC cùng VLAN                                           | Chưa có ảnh riêng                                                                                        | **Chưa kiểm thử**                                       |
| 2   | PC khác VLAN                 | `ping` từ `PC_DATA` (VLAN 10) tới `10.10.40.10`                        | LAN-01 (`MAIN-PC-STAFFA` tới LAN/DMZ), `PC_DATA` ping server farm liên tục ở thử nghiệm 4.1              | PASS (theo ảnh LAN-01)                                  |
| 3   | Main ↔ chi nhánh             | `ping`, `tracert 10.20.40.10` từ BHTQ                                  | 4/4 tới `10.10.40.10`; tracert 4 hop qua BHTQ-R1 → Carrier-R → DBP-R1 → Server DBP; TC-06: 5/5, RTT 2 ms | PASS                                                    |
| 4   | Tới server DMZ               | Mở `http://10.10.50.11` từ nhân viên                                   | DMZ-01: trang hiển thị                                                                                   | PASS (DMZ `.11`, khác Web Server `.10` trong quy hoạch) |
| 5   | **Khách không vào được LAN** | Ping từ `MAIN-B-GUEST` tới `10.10.20.10`, `10.10.40.10`, `10.10.50.10` | Chặn được staff và server farm, **nhưng ping được DMZ `10.10.50.10`**                                    | **FAIL** (SEC-01)                                       |
| 6   | Internet tới Web server      | Mở `http://192.0.2.10` từ nhân viên                                    | INT-01: trang hiển thị                                                                                   | PASS                                                    |

## 2. Kiểm thử bảo mật và VPN

| Mã         | Nội dung                                                                     | Kết quả quan sát                                                                         | Trạng thái        |
| ---------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ----------------- |
| SEC-01     | Cô lập Guest                                                                 | Chặn `10.10.20.10`, `10.10.40.10`; vẫn ping DMZ `10.10.50.10` 4/4; ping `192.0.2.10` 3/4 | **FAIL**          |
| SEC-02     | Từ `REMOTE` ping `10.10.40.10` (không qua VPN)                               | Timeout 4/4                                                                              | PASS có giới hạn  |
| DMZ-01     | Staff → Web DMZ                                                              | Mở được trang `10.10.50.11`                                                              | PASS              |
| INT-01     | Staff → Web Internet                                                         | Mở được `192.0.2.10`                                                                     | PASS              |
| VPN-S2S-01 | IPsec site-to-site                                                           | IKE SA `QM_IDLE`; IPsec encaps/decaps cho peer `10.255.0.18`                             | PASS có giới hạn  |
| VPN-RA-01  | Remote-access VPN                                                            | Client được cấp `10.10.200.10`                                                           | Xác nhận kết nối  |
| VPN-RA-02  | `REMOTE` ping `10.10.40.10` qua VPN                                          | Mất 1/4 gói lần đầu, 4/4 lần sau; IPsec SA hoạt động                                     | PASS sau hội tụ   |
| (không có) | SSH từ Jump Host; Internet → DMZ qua NAT `203.162.10.3`; camera/NVR; IDS/IPS | Chưa có ảnh                                                                              | **Chưa kiểm thử** |

**Ghi chú SEC-01:** `ACL-GUEST-ISOLATE` đã sửa trong báo cáo nhưng **chưa áp dụng vào mô phỏng và chạy lại**. Lỗi nằm ở chỗ Web DMZ `10.10.50.10` thuộc `10.0.0.0/8`; ACL phải `deny` trước khi `permit` web ra ngoài. Phải áp dụng và chạy lại trước khi ghi PASS.

```text
ip access-list extended ACL-GUEST-ISOLATE
 permit udp any eq bootpc any eq bootps
 permit udp any host 10.10.40.10 eq domain
 permit tcp 10.10.60.0 0.0.3.255 host 10.10.40.10 eq domain
 deny   ip  10.10.60.0 0.0.3.255 10.0.0.0 0.255.255.255
 permit tcp 10.10.60.0 0.0.3.255 any eq www
 permit tcp 10.10.60.0 0.0.3.255 any eq 443
!
interface Vlan60
 ip access-group ACL-GUEST-ISOLATE in
```

## 3. Kịch bản demo (theo báo cáo)

### 3.1 Giai đoạn 1: trạng thái chuẩn và hội tụ

| Bước | Lệnh                                                    | Tiêu chí                                                   | Kết quả           |
| ---- | ------------------------------------------------------- | ---------------------------------------------------------- | ----------------- |
| 1.1  | `show standby brief` (MAIN-CORE-SW1, DBP-R1, BHTQ-R1)        | Mọi nhóm Active, thấy đúng Standby, không có Speak/Unknown | HA-01 (hình 8, 9) |
| 1.2  | `show ip ospf neighbor` (Carrier-R, BHTQ-R2)            | Serial: `FULL/-`; subinterface: `FULL/DR` hoặc `FULL/BDR`  | PASS (listing 2)  |
| 1.3  | `ping 10.10.40.10`, `tracert 10.20.40.10` từ `PC-BHTQ1` | 4/4; đường đi qua BHTQ-R1 → Carrier-R → DBP-R1             | PASS              |

### 3.2 Giai đoạn 3: chuyển đổi dự phòng WAN tại BHTQ

1. `PC-BHTQ3` (`10.30.10.12`) chạy `ping 10.20.40.10 -t`.
2. Trên `BHTQ-R1`: `interface Serial0/3/0` → `shutdown`.
3. Quan sát: HSRP priority của R1 giảm `110 → 50` (decrement 60); `BHTQ-R2` (105) thành Active; floating static `0.0.0.0/0 via 10.255.0.21 AD 150` vào bảng định tuyến.
4. Kết quả ghi nhận: **rơi 1 gói**, RTT tăng `3 ms → 11 ms`; `tracert` đi qua `BHTQ-R2 → ISP2-R → DBP-R2 → Server DBP`.
5. Failback: `no shutdown` → R1 lấy lại Active nhờ `preempt`, OSPF `FULL` lại với Carrier-R.

Ảnh bổ sung WAN-01, WAN-02, WAN-03, HA-02.1, HA-02.2 chỉ hiển thị CLI/ping của DBP-R2, **không tự chứng minh đường dự phòng** hay thời gian gián đoạn.

### 3.3 Giai đoạn 4: dự phòng lõi Main

| Thử nghiệm | Thao tác                                                                                                 | Kết quả báo cáo                                                                  |
| ---------- | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 4.1        | `shutdown` `Fa0/2` trên MAIN-CORE-SW1 (một cáp trong Port-channel 1), ping `10.10.40.10 -t` từ `PC_DATA` | 0% mất gói, LACP dồn sang cáp còn lại                                            |
| 4.2        | Tắt MAIN-CORE-SW1                                                                                        | MAIN-CORE-SW2 thành Active (holdtime 10 s), dùng đường VLAN 253 lên MAIN-FW-01 `Gi1/5` | Mô tả trong báo cáo, **chưa thấy ảnh** |

Khi chạy thử 4.2 cần **bỏ `shutdown` trên `Gi1/5`** của MAIN-FW-01 (cấu hình hiện đang shutdown).

## 4. Bảng nghiệm thu TC-01 → TC-06 (báo cáo)

| Mã    | Hạng mục                                      | Ban đầu           | Kết quả | Nguyên nhân và xử lý                                                                            |
| ----- | --------------------------------------------- | ----------------- | ------- | ----------------------------------------------------------------------------------------------- |
| TC-01 | Định tuyến BHTQ-R2 → Server DBP `10.20.40.10` | Mất 100% (0/5)    | PASS    | Default AD 1 đè OSPF → floating AD 150, bỏ `default-information originate`, `passive-interface` |
| TC-02 | Láng giềng OSPF qua VLAN 99 (BHTQ)            | INIT/DROTHER      | PASS    | Bỏ `native`, dùng `dot1Q 99`, native VLAN switch = 1, xóa tiến trình OSPF                       |
| TC-03 | Bảng MAC trên trunk VLAN 99                   | Học đủ MAC R1, R2 | PASS    | L2 ổn, không vòng STP, không khóa Port-Security                                                 |
| TC-04 | Đồng bộ HSRP nhóm 90 và 99                    | Nhóm 90 Speak     | PASS    | Thêm Gi0/0.90, 0.99 trên cả R1/R2, mở trunk VLAN 1, 10, 30, 40, 70, 90, 99                      |
| TC-05 | Traceroute DBP → Server BHTQ `10.30.40.10`    | Rớt hop 4         | PASS    | Server thiếu gateway → đặt `10.30.40.1`, access VLAN 40                                         |
| TC-06 | Ping trực tiếp qua leased line                | 5/5, RTT 2 ms     | PASS    | Đường chính ổn định                                                                             |

## 5. Việc cần làm để hoàn thiện Step 5

1. **Sửa và chạy lại SEC-01** (áp dụng ACL vào `Vlan60`).
2. Chạy thêm: ping cùng VLAN (mục 1.1), Internet → DMZ qua `203.162.10.3` (HTTP), SSH từ Jump Host, camera → NVR.
3. Chụp ảnh từng ca kiểm thử, đặt mã (ví dụ `LAN-01`) trùng bảng trong báo cáo.
4. Không ghi PASS cho ca chưa có ảnh bằng chứng.
