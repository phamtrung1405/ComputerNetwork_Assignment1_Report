# Step 7 – Nộp bài lên BKeL

## 1. Danh sách file cần nộp

| File                                                       | Nội dung                                                      | Tình trạng                 |
| ---------------------------------------------------------- | ------------------------------------------------------------- | -------------------------- |
| `final.pkt`                                                | Mô phỏng Packet Tracer 9.0.1 (đặt tên theo yêu cầu nhóm/lớp)  | Có (cần kiểm tra lần cuối) |
| `main.pdf`                                                 | Báo cáo (Nhóm 8, L01, CO3094)                                 | Có                         |
| Ảnh bằng chứng (`graphics/tc_pic`)                         | SEC_01, SEC_02, DMZ_01, INT_01, VPN_01, VPN_01.1, VPN_02, HA… | Có; còn thiếu vài ca       |
| `access_control_matrix.csv`, `acl_implementation_guide.md` | Ma trận kiểm soát truy cập AC-01–AC-24, hướng dẫn ACL         | Có (theo báo cáo)          |

Softcopy nộp BKeL đúng hạn; bản in dùng cho buổi trình bày/demo. Hạn nộp: **4 tuần sau khi đề được giao**; kiểm tra ngày chính xác trên BKeL.

## 2. Checklist trước khi nộp

### 2.1 Mô phỏng

- [ ] Mở `final.pkt` bằng đúng Packet Tracer 9.0.1, không báo lỗi.
- [ ] Tên thiết bị khớp Bảng 1 của báo cáo (`MAIN-FW-01/02`, `MAIN-CORE-SW1/2`, `MAIN-WAN1-SW/WAN2-SW`; hostname trong CLI phải trùng).
- [ ] Tên máy đầu cuối thống nhất.
- [ ] `write memory` trên mọi thiết bị; lưu file.
- [ ] Mọi cổng có nhãn; không còn liên kết đỏ ngoài ý muốn (`inside-backup` đang shutdown cần ghi chú).

### 2.2 Sáu kiểm tra demo bắt buộc

- [ ] Cùng VLAN
- [ ] Khác VLAN
- [ ] Main ↔ DBP/BHTQ
- [ ] Tới server DMZ
- [ ] **Khách không vào được PC trong LAN (SEC-01 phải PASS)**
- [ ] Internet tới Web server

### 2.3 Báo cáo

- [ ] Có đủ Step 1–6 theo đúng thứ tự đề bài (cấu trúc, thiết bị/IP/đi dây, băng thông, mô phỏng, kiểm thử, đánh giá).
- [ ] Thông tin thành viên và mức đánh giá đúng.
- [ ] Mỗi ca kiểm thử có ảnh; không ghi PASS ngoài bằng chứng.
- [ ] Số liệu giả định (Wi-Fi khách, số camera…) được ghi rõ.
- [ ] Phần "vấn đề còn lại" và "định hướng" có mặt (Step 6).
- [ ] Không có mật khẩu thật trong tài liệu.
- [ ] Tài liệu tham khảo đầy đủ.

## 3. Chuẩn bị buổi demo

| Thứ tự | Nội dung                                                  | Thời gian gợi ý |
| ------ | --------------------------------------------------------- | --------------- |
| 1      | Giới thiệu topology (hai firewall, DMZ, WAN chính/phụ)    | 2–3 phút        |
| 2      | `show standby brief`, `show ip ospf neighbor`             | 2 phút          |
| 3      | 6 kiểm tra bắt buộc                                       | 4–5 phút        |
| 4      | Failover WAN tại BHTQ (shutdown `Serial0/3/0`, ping `-t`) | 3 phút          |
| 5      | Failover lõi (`Fa0/2` MAIN-CORE-SW1; tùy chọn tắt MAIN-CORE-SW1)    | 2–3 phút        |
| 6      | VPN site-to-site và remote-access                         | 2 phút          |
| 7      | Hạn chế còn lại và định hướng                             | 2 phút          |

Mẹo: chuẩn bị sẵn cửa sổ CLI các thiết bị; dùng `Alt + D` để đẩy nhanh hội tụ trong PT; sau demo failover nhớ `no shutdown` để khôi phục.

## 4. Phân công (điền)

| Thành viên       | MSSV    | Phần phụ trách |
| ---------------- | ------- | -------------- |
| Phạm Nghĩa Trung | 2413713 |                |
| Nguyễn Minh Bảo  | 2410272 |                |
| Đào Nhật Toàn    | 2413523 |                |
| Mai Hồng Phước   | 2412803 |                |

## 5. Tài liệu đi kèm

| File                               | Dùng cho                      |
| ---------------------------------- | ----------------------------- |
| `step-1-cau-truc-mang.md`          | Step 1                        |
| `step-2-thiet-bi-ip-cabling.md`    | Step 2                        |
| `step-3-bang-thong.md`             | Step 3                        |
| `step-4-mo-phong-packet-tracer.md` | Step 4                        |
| `step-5-kiem-thu.md`               | Step 5                        |
| `step-6-danh-gia.md`               | Step 6                        |
| `doi-chieu-ten-thiet-bi.md`        | Đối chiếu tên với `final.pkt` |
