# TỔNG HỢP LỆNH CẤU HÌNH TOÀN BỘ 25 THIẾT BỊ MẠNG DOANH NGHIỆP
**Mô hình:** Main Site (HCMC) – Vùng DMZ/Internet – Chi nhánh 1 (DBP) – Chi nhánh 2 (BHTQ)  
**Công cụ:** Cisco Packet Tracer *(Đã lược bỏ hoàn toàn các dấu chấm than `!` và chuẩn hóa từng dòng lệnh)*

---

## PHẦN I: VÙNG MAIN SITE, DMZ & BIÊN INTERNET (15 THIẾT BỊ)

### 1. Tường lửa ASA2 (Vùng DMZ / Internet)
```text
enable
configure terminal
hostname ASA2
interface GigabitEthernet1/1
 nameif outside
 security-level 0
 ip address 203.162.10.2 255.255.255.248
 no shutdown
exit
interface GigabitEthernet1/2
 nameif dmz
 security-level 50
 ip address 10.10.50.2 255.255.255.0
 no shutdown
exit
interface GigabitEthernet1/3
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface GigabitEthernet1/4
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface GigabitEthernet1/5
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface GigabitEthernet1/6
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface GigabitEthernet1/7
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface GigabitEthernet1/8
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface Management1/1
 management-only
 no nameif
 no security-level
 no ip address
 shutdown
exit
object network DMZ_WEB_SERVER
 host 10.10.50.10
 nat (dmz,outside) static 203.162.10.3
exit
object network OBJ-ALL-INTERNAL
 subnet 10.0.0.0 255.0.0.0
 nat (dmz,outside) dynamic interface
exit
route outside 0.0.0.0 0.0.0.0 203.162.10.1 1
route dmz 10.0.0.0 255.0.0.0 10.10.50.1 1
access-list DMZ_TO_OUTSIDE extended permit ip any any
access-list OUTSIDE_TO_DMZ extended permit tcp any host 10.10.50.10 eq www
access-list OUTSIDE_TO_DMZ extended permit icmp any any
access-list OUTSIDE_IN extended permit icmp any 10.10.0.0 255.255.0.0
access-list OUTSIDE_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list OUTSIDE_IN extended permit ip 10.30.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list OUTSIDE_IN extended permit ip 10.255.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list OUTSIDE_IN extended permit icmp any any
access-list OUTSIDE_IN extended permit tcp any eq www any
access-list OUTSIDE_IN extended permit tcp any eq 443 any
access-list OUTSIDE_IN extended permit udp any eq domain any
access-list WAN_TO_INSIDE extended permit icmp any 10.10.0.0 255.255.0.0
access-list WAN_TO_INSIDE extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list WAN_TO_INSIDE extended permit ip 10.30.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list WAN_TO_INSIDE extended permit ip 10.255.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-group DMZ_TO_OUTSIDE in interface dmz
access-group OUTSIDE_IN in interface outside
class-map inspection_default
 match default-inspection-traffic
exit
policy-map type inspect dns preset_dns_map
 parameters
  message-length maximum 512
 exit
exit
policy-map global_policy
 class inspection_default
  inspect dns preset_dns_map
  inspect ftp
  inspect icmp
  inspect tftp
 exit
exit
service-policy global_policy global
telnet timeout 5
ssh timeout 5
end
write memory
```

### 2. Router ISP1-R
```text
enable
configure terminal
hostname ISP1-R
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip cef
no ipv6 cef
no ip domain-lookup
spanning-tree mode pvst
interface GigabitEthernet0/0
 description TO_EDGE_SW_Fa0/1
 ip address 192.0.2.1 255.255.255.0
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/1
 description TO_INTERNET_WEB
 ip address 203.162.10.1 255.255.255.248
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
ip classless
ip route 10.0.0.0 255.0.0.0 203.162.10.2
ip flow-export version 9
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 login
exit
end
write memory
```

### 3. Switch Layer 3 DMZ-SW
```text
enable
configure terminal
hostname DMZ-SW
no profinet
ip routing
spanning-tree mode pvst
interface range FastEthernet0/1 - 24
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface GigabitEthernet0/1
 no shutdown
exit
interface GigabitEthernet0/2
 no shutdown
exit
interface Vlan1
 description GATEWAY_DMZ
 ip address 10.10.50.254 255.255.255.0
 no shutdown
exit
ip classless
ip route 10.0.0.0 255.0.0.0 10.10.50.1
ip route 0.0.0.0 0.0.0.0 10.10.50.2
ip flow-export version 9
line con 0
exit
line aux 0
exit
line vty 0 4
 login
exit
end
write memory
```

### 4. Tường lửa ASA1 (Trung tâm Main Site)
```text
enable
configure terminal
hostname ASA1
interface GigabitEthernet1/1
 nameif backup-wan
 security-level 60
 ip address 10.10.252.1 255.255.255.248
 no shutdown
exit
interface GigabitEthernet1/2
 nameif wan
 security-level 70
 ip address 10.10.251.1 255.255.255.248
 no shutdown
exit
interface GigabitEthernet1/3
 nameif dmz
 security-level 50
 ip address 10.10.50.1 255.255.255.0
 no shutdown
exit
interface GigabitEthernet1/4
 nameif inside
 security-level 100
 ip address 10.10.250.4 255.255.255.248
 no shutdown
exit
interface GigabitEthernet1/5
 nameif inside-backup
 security-level 100
 ip address 10.10.253.1 255.255.255.248
 shutdown
exit
interface GigabitEthernet1/6
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface GigabitEthernet1/7
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface GigabitEthernet1/8
 no nameif
 no security-level
 no ip address
 shutdown
exit
interface Management1/1
 management-only
 no nameif
 no security-level
 no ip address
 shutdown
exit
object network OBJ-ALL-INTERNAL
 subnet 10.0.0.0 255.0.0.0
exit
object network OBJ-LOCAL-NET
 subnet 10.0.0.0 255.0.0.0
exit
route wan 10.20.0.0 255.255.0.0 10.10.251.3 1
route inside-backup 10.10.0.0 255.255.0.0 10.10.250.1 1
route dmz 0.0.0.0 0.0.0.0 10.10.50.2 1
route wan 10.30.0.0 255.255.0.0 10.10.251.3 1
route wan 10.255.0.0 255.255.0.0 10.10.251.3 1
route wan 10.255.0.0 255.255.0.0 10.10.251.4 10
route wan 10.20.0.0 255.255.0.0 10.10.251.4 10
route wan 10.30.0.0 255.255.0.0 10.10.251.4 10
route backup-wan 10.20.0.0 255.255.0.0 10.10.252.4 10
route backup-wan 10.30.0.0 255.255.0.0 10.10.252.4 10
route backup-wan 10.255.0.0 255.255.0.0 10.10.252.4 10
route backup-wan 10.255.0.24 255.255.255.252 10.10.252.4 1
route backup-wan 10.255.0.16 255.255.255.252 10.10.252.4 1
route backup-wan 10.255.0.20 255.255.255.252 10.10.252.4 1
route inside-backup 10.10.0.0 255.255.0.0 10.10.253.2 10
access-list BACKUP_WAN_IN extended permit ip any 10.10.0.0 255.255.0.0
access-list BACKUP_WAN_IN extended permit ip any 10.10.50.0 255.255.255.0
access-list BACKUP_WAN_IN extended permit icmp any 10.10.0.0 255.255.0.0
access-list BACKUP_WAN_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list BACKUP_WAN_IN extended permit ip 10.30.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list BACKUP_WAN_IN extended permit ip 10.255.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list BACKUP_WAN_IN extended permit ip any any
access-list WAN_IN extended permit icmp any 10.10.50.0 255.255.255.0
access-list WAN_IN extended permit ip 10.10.251.0 255.255.255.248 10.10.50.0 255.255.255.0
access-list WAN_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.50.0 255.255.255.0
access-list WAN_IN extended permit ip 10.30.0.0 255.255.0.0 10.10.50.0 255.255.255.0
access-list WAN_IN extended permit ip 10.255.0.0 255.255.0.0 10.10.50.0 255.255.255.0
access-list WAN_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list WAN_IN extended permit ip 10.30.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list WAN_IN extended permit ip 10.255.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list WAN_IN extended permit ip 10.10.251.0 255.255.255.248 10.10.0.0 255.255.0.0
access-list WAN_TO_INSIDE extended permit icmp any 10.10.0.0 255.255.0.0
access-list WAN_TO_INSIDE extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list WAN_TO_INSIDE extended permit ip 10.30.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list WAN_TO_INSIDE extended permit ip 10.255.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list OUTSIDE_TO_INSIDE extended permit icmp any 10.10.0.0 255.255.0.0
access-list OUTSIDE_TO_INSIDE extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list OUTSIDE_TO_INSIDE extended permit ip 10.30.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list OUTSIDE_TO_INSIDE extended permit ip 10.255.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list DMZ_IN extended permit ip 10.10.50.0 255.255.255.0 10.10.0.0 255.255.0.0
access-list DMZ_IN extended permit ip 10.10.50.0 255.255.255.0 10.0.0.0 255.0.0.0
access-list DMZ_IN extended permit tcp any eq www 10.0.0.0 255.0.0.0
access-list DMZ_IN extended permit tcp any eq 443 10.0.0.0 255.0.0.0
access-list DMZ_IN extended permit udp any eq domain 10.0.0.0 255.0.0.0
access-list DMZ_IN extended permit icmp any 10.0.0.0 255.0.0.0
access-group WAN_IN in interface wan
access-group BACKUP_WAN_IN in interface backup-wan
access-group DMZ_IN in interface dmz
class-map inspection_default
 match default-inspection-traffic
exit
policy-map type inspect dns preset_dns_map
 parameters
  message-length maximum 512
 exit
exit
policy-map global_policy
 class inspection_default
  inspect dns preset_dns_map
  inspect ftp
  inspect icmp
  inspect tftp
 exit
exit
service-policy global_policy global
telnet timeout 5
ssh timeout 5
end
write memory
```

### 5. Switch Layer 3 Core-SW1
```text
enable
configure terminal
hostname Core-SW1
no profinet
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip dhcp excluded-address 10.10.20.1 10.10.20.49
ip dhcp pool STAFF-B
 network 10.10.20.0 255.255.254.0
 default-router 10.10.20.1
 dns-server 10.10.40.10
exit
ip dhcp pool VLAN10_DATA
 dns-server 10.10.50.12
exit
ip routing
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,60,70,80,90,250 priority 4096
vlan 10,20,30,40,60,70,80,90,99,250
exit
interface Port-channel1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,60,70,80,90,99,250
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/1
 description TO_Access-A
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,60,80,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,60,80,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
exit
interface FastEthernet0/3
 description TO_Access-B
 switchport trunk native vlan 99
 switchport trunk allowed vlan 20,60,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/4
 description TO_Server-SW
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,40,60,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/5
 description TO_Mgmt-SW
 switchport trunk native vlan 99
 switchport trunk allowed vlan 90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/6
 description TO_Cam-SW
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,60,80,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/7
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,60,70,80,90,99,250
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
exit
interface FastEthernet0/8
 description TO_ASA_1_Eth0/3
 switchport access vlan 250
 switchport mode access
 spanning-tree portfast
exit
interface FastEthernet0/9
 switchport access vlan 250
 switchport mode access
 spanning-tree portfast
exit
interface range FastEthernet0/10 - 24
 shutdown
exit
interface range GigabitEthernet0/1 - 2
 shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan10
 description STAFF-A
 mac-address 00e0.8fc0.3601
 ip address 10.10.10.2 255.255.254.0
 ip helper-address 10.10.40.10
 standby 10 ip 10.10.10.1
 standby 10 priority 200
 standby 10 preempt
 no shutdown
exit
interface Vlan20
 description STAFF-B
 mac-address 00e0.8fc0.3602
 ip address 10.10.20.2 255.255.254.0
 ip helper-address 10.10.40.10
 standby 20 ip 10.10.20.1
 standby 20 priority 200
 standby 20 preempt
 no shutdown
exit
interface Vlan30
 description MEDICAL
 mac-address 00e0.8fc0.3603
 ip address 10.10.30.2 255.255.255.0
 ip helper-address 10.10.40.10
 standby 30 ip 10.10.30.1
 standby 30 priority 200
 standby 30 preempt
 no shutdown
exit
interface Vlan40
 description SERVER-FARM
 mac-address 00e0.8fc0.3604
 ip address 10.10.40.2 255.255.255.0
 standby 40 ip 10.10.40.1
 standby 40 priority 200
 standby 40 preempt
 no shutdown
exit
interface Vlan60
 description GUEST-WIFI
 mac-address 00e0.8fc0.3605
 ip address 10.10.60.2 255.255.252.0
 ip helper-address 10.10.40.10
 ip access-group ACL-GUEST-ISOLATE in
 standby 60 ip 10.10.60.1
 standby 60 priority 200
 standby 60 preempt
 no shutdown
exit
interface Vlan70
 description CAMERA
 mac-address 00e0.8fc0.3606
 ip address 10.10.70.2 255.255.255.0
 ip helper-address 10.10.40.10
 standby 70 ip 10.10.70.1
 standby 70 priority 200
 standby 70 preempt
 no shutdown
exit
interface Vlan80
 description VOICE
 mac-address 00e0.8fc0.3607
 ip address 10.10.80.2 255.255.255.0
 ip helper-address 10.10.40.10
 standby 80 ip 10.10.80.1
 standby 80 priority 200
 standby 80 preempt
 no shutdown
exit
interface Vlan90
 description MANAGEMENT
 mac-address 00e0.8fc0.3608
 ip address 10.10.90.2 255.255.255.0
 ip helper-address 10.10.40.10
 standby 90 ip 10.10.90.1
 standby 90 priority 200
 standby 90 preempt
 no shutdown
exit
interface Vlan250
 description INFRA-TRANSIT
 mac-address 00e0.8fc0.3609
 ip address 10.10.250.2 255.255.255.248
 standby 250 ip 10.10.250.1
 standby 250 priority 200
 standby 250 preempt
 no shutdown
exit
router ospf 1
 router-id 2.2.2.1
 log-adjacency-changes
 network 10.10.0.0 0.0.255.255 area 0
 network 10.255.0.0 0.0.0.255 area 0
 network 10.255.0.0 0.0.255.255 area 0
 network 10.10.250.0 0.0.0.7 area 0
exit
ip classless
ip route 0.0.0.0 0.0.0.0 10.10.250.4
ip flow-export version 9
access-list 10 permit host 10.10.90.30
ip access-list extended ACL-GUEST-ISOLATE
 permit udp any eq bootpc any eq bootps
 permit udp any host 10.10.40.10 eq domain
 deny ip 10.10.60.0 0.0.3.255 10.0.0.0 0.255.255.255
 permit ip any any
exit
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 6. Switch Layer 3 Core-SW2
```text
enable
configure terminal
hostname Core-SW2
no profinet
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip routing
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,60,70,80,90,250 priority 8192
vlan 10,20,30,40,60,70,80,90,99,250,253
exit
interface Port-channel1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,60,70,80,90,99,250
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,60,70,80,90,99,250
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
exit
interface FastEthernet0/2
 description TO_Access-A
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,80,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/3
 description TO_Access-B
 switchport trunk native vlan 99
 switchport trunk allowed vlan 20,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/4
 description TO_Cam-SW
 switchport trunk native vlan 99
 switchport trunk allowed vlan 70,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/5
 description TO_Mgmt-SW
 switchport trunk native vlan 99
 switchport trunk allowed vlan 90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/6
 description TO_Server-SW
 switchport trunk native vlan 99
 switchport trunk allowed vlan 20,40,60,90,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/7
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,60,70,80,90,99,250
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
exit
interface FastEthernet0/8
 switchport access vlan 253
 switchport mode access
 spanning-tree portfast
exit
interface FastEthernet0/9
 switchport access vlan 253
 switchport mode access
 spanning-tree portfast
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan10
 description STAFF-A
 mac-address 00d0.58a1.1201
 ip address 10.10.10.3 255.255.254.0
 ip helper-address 10.10.40.10
 standby 10 ip 10.10.10.1
 standby 10 priority 150
 standby 10 preempt
 no shutdown
exit
interface Vlan20
 description STAFF-B
 mac-address 00d0.58a1.1202
 ip address 10.10.20.3 255.255.254.0
 ip helper-address 10.10.40.10
 standby 20 ip 10.10.20.1
 standby 20 priority 150
 standby 20 preempt
 no shutdown
exit
interface Vlan30
 description MEDICAL
 mac-address 00d0.58a1.1203
 ip address 10.10.30.3 255.255.255.0
 ip helper-address 10.10.40.10
 standby 30 ip 10.10.30.1
 standby 30 priority 150
 standby 30 preempt
 no shutdown
exit
interface Vlan40
 description SERVER-FARM
 mac-address 00d0.58a1.1204
 ip address 10.10.40.3 255.255.255.0
 standby 40 ip 10.10.40.1
 standby 40 priority 150
 standby 40 preempt
 no shutdown
exit
interface Vlan60
 description GUEST-WIFI
 mac-address 00d0.58a1.1205
 ip address 10.10.60.3 255.255.252.0
 ip helper-address 10.10.40.10
 ip access-group ACL-GUEST-ISOLATE in
 standby 60 ip 10.10.60.1
 standby 60 priority 150
 standby 60 preempt
 no shutdown
exit
interface Vlan70
 description CAMERA
 mac-address 00d0.58a1.1206
 ip address 10.10.70.3 255.255.255.0
 ip helper-address 10.10.40.10
 standby 70 ip 10.10.70.1
 standby 70 priority 150
 standby 70 preempt
 no shutdown
exit
interface Vlan80
 description VOICE
 mac-address 00d0.58a1.1207
 ip address 10.10.80.3 255.255.255.0
 ip helper-address 10.10.40.10
 standby 80 ip 10.10.80.1
 standby 80 priority 150
 standby 80 preempt
 no shutdown
exit
interface Vlan90
 description MANAGEMENT
 mac-address 00d0.58a1.1208
 ip address 10.10.90.3 255.255.255.0
 ip helper-address 10.10.40.10
 standby 90 ip 10.10.90.1
 standby 90 priority 150
 standby 90 preempt
 no shutdown
exit
interface Vlan250
 description INFRA-TRANSIT
 mac-address 00d0.58a1.1209
 ip address 10.10.250.3 255.255.255.248
 standby 250 ip 10.10.250.1
 standby 250 preempt
 no shutdown
exit
interface Vlan253
 mac-address 00d0.58a1.120a
 ip address 10.10.253.2 255.255.255.248
 no shutdown
exit
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 network 10.10.250.0 0.0.0.7 area 0
 network 10.10.0.0 0.0.255.255 area 0
exit
ip classless
ip route 0.0.0.0 0.0.0.0 10.10.250.4
ip route 0.0.0.0 0.0.0.0 10.10.253.1 10
ip flow-export version 9
access-list 10 permit host 10.10.90.30
ip access-list extended ACL-GUEST-ISOLATE
 permit udp any eq bootpc any eq bootps
 permit udp any host 10.10.40.10 eq domain
 deny ip 10.10.60.0 0.0.3.255 10.0.0.0 0.255.255.255
 permit ip any any
exit
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 7. Switch Server-SW
```text
enable
configure terminal
hostname Server-SW
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
vlan 10,20,40,60,80,90,99
exit
interface FastEthernet0/1
 description TO_CORE_SW1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,40,60,80,90,99
 switchport mode trunk
exit
interface FastEthernet0/2
 description DNS_DHCP
 switchport access vlan 40
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/3
 description APP
 switchport access vlan 40
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/4
 description DB
 switchport access vlan 40
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/5
 description FTP_FILE
 switchport access vlan 40
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/6
 description CME_R
 switchport access vlan 80
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/7
 description TO_WLC_PT
 switchport access vlan 90
 switchport mode access
 spanning-tree portfast
exit
interface FastEthernet0/8
 description TO_CORE_SW2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,40,60,80,90,99
 switchport mode trunk
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.10.90.13 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 8. Switch Mgmt-SW
```text
enable
configure terminal
hostname Mgmt-SW
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 1
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
vlan 90,99
exit
interface FastEthernet0/1
 description TO_CORE_SW1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 90,99
 switchport mode trunk
exit
interface FastEthernet0/2
 description TO_CORE_SW2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 90,99
 switchport mode trunk
exit
interface FastEthernet0/3
 description SYSLOG_NTP
 switchport access vlan 90
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/4
 description AAA
 switchport access vlan 90
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/5
 description JUMP_HOST
 switchport access vlan 90
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.10.90.14 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 9. Switch Cam-SW
```text
enable
configure terminal
hostname Cam-SW
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
vlan 70,90,99
exit
interface FastEthernet0/1
 description TO_CORE_SW1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 70,90,99
 switchport mode trunk
exit
interface FastEthernet0/2
 description TO_CORE_SW2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 70,90,99
 switchport mode trunk
exit
interface FastEthernet0/3
 description WEBCAM1
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/4
 description WEBCAM2
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/5
 description NVR
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.10.90.15 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 10. Switch Access-A
```text
enable
configure terminal
hostname Access-A
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
vlan 10,60,80,90,99
exit
interface FastEthernet0/1
 description TO_CORE_SW1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,60,80,90,99
 switchport mode trunk
exit
interface FastEthernet0/2
 description TO_CORE_SW2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,60,80,90,99
 switchport mode trunk
exit
interface FastEthernet0/3
 description PC_DATA
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0040.0BA8.7E5E
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/4
 description IP_PHONE
 switchport access vlan 80
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/5
 description PRINTER
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 00D0.D3E5.4B5E
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/6
 description LAP_PT
 switchport access vlan 60
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface range FastEthernet0/7 - 10
 spanning-tree portfast
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.10.90.11 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 11. Switch Access-B
```text
enable
configure terminal
hostname Access-B
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
vlan 20,60,90,99
exit
interface FastEthernet0/1
 description TO_CORE_SW1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 20,60,90,99
 switchport mode trunk
exit
interface FastEthernet0/2
 description TO_CORE_SW2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 20,60,90,99
 switchport mode trunk
exit
interface FastEthernet0/3
 description PC_DATA_WIFI_STAFF
 switchport access vlan 20
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 00E0.8FA8.830D
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/4
 description PRINTER1
 switchport access vlan 20
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 000D.BDDC.2D88
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface FastEthernet0/5
 description LAP_PT1
 switchport access vlan 60
 switchport trunk native vlan 99
 switchport trunk allowed vlan 20,60,90,99
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.10.90.12 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 12. Switch WAN1-SW
```text
enable
configure terminal
hostname WAN1-SW
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
no ip domain-lookup
spanning-tree mode pvst
spanning-tree extend system-id
interface range FastEthernet0/1 - 24
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface range GigabitEthernet0/1 - 2
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface Vlan1
 description MGMT_WAN-SW
 ip address 10.10.251.5 255.255.255.248
 no shutdown
exit
ip default-gateway 10.10.251.1
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 login
exit
line vty 5 15
 login
exit
end
write memory
```

### 13. Switch WAN2-SW
```text
enable
configure terminal
hostname WAN2-SW
no ip domain-lookup
spanning-tree mode pvst
spanning-tree extend system-id
interface range FastEthernet0/1 - 24
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface range GigabitEthernet0/1 - 2
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface Vlan1
 ip address 10.10.252.6 255.255.255.248
 no shutdown
exit
ip default-gateway 10.10.252.1
line con 0
exit
line vty 0 4
 login
exit
line vty 5 15
 login
exit
end
write memory
```

### 14. Router WAN-R2
```text
enable
configure terminal
hostname WAN-R2
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip cef
no ipv6 cef
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
spanning-tree mode pvst
interface GigabitEthernet0/0
 description TO_WAN2_SW
 ip address 10.10.252.4 255.255.255.248
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/1
 description TO_ISP2_R
 ip address 10.255.0.26 255.255.255.252
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
exit
interface Serial0/3/0
 no ip address
 clock rate 2000000
 shutdown
exit
interface Serial0/3/1
 description LEASED_LINE_TO_CARRIER
 ip address 10.255.0.5 255.255.255.252
 ip ospf cost 2000
 clock rate 64000
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
router ospf 1
 router-id 1.1.1.2
 log-adjacency-changes
 redistribute static subnets
 redistribute connected subnets
 network 10.255.0.0 0.0.0.255 area 0
 network 10.255.0.4 0.0.0.3 area 0
 network 10.255.0.24 0.0.0.3 area 0
 network 10.10.252.0 0.0.0.7 area 0
 default-information originate
exit
ip classless
ip route 0.0.0.0 0.0.0.0 10.10.252.1
ip route 10.10.0.0 255.255.0.0 10.10.252.1
ip route 10.10.50.0 255.255.255.0 10.10.252.1
ip route 10.20.0.0 255.255.0.0 10.255.0.25 10
ip route 10.30.0.0 255.255.0.0 10.255.0.25 10
ip route 10.255.0.16 255.255.255.252 10.255.0.25
ip route 10.255.0.20 255.255.255.252 10.255.0.25
ip route 10.10.252.0 255.255.255.248 GigabitEthernet0/0
ip flow-export version 9
access-list 10 permit host 10.10.90.30
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 15. Router WAN-R1
```text
enable
configure terminal
hostname WAN-R1
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip cef
no ipv6 cef
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
spanning-tree mode pvst
interface GigabitEthernet0/0
 description TO_WAN_SW
 ip address 10.10.251.3 255.255.255.248
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 shutdown
exit
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
exit
interface Serial0/3/0
 description LEASED_LINE_TO_CARRIER
 ip address 10.255.0.1 255.255.255.252
 clock rate 64000
 no shutdown
exit
interface Serial0/3/1
 no ip address
 clock rate 2000000
 shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 redistribute bgp 65001 subnets
 redistribute static subnets
 redistribute connected subnets
 network 10.0.0.0 0.255.255.255 area 0
 network 172.16.0.0 0.15.255.255 area 0
 network 10.255.0.0 0.0.0.255 area 0
 network 10.255.0.0 0.0.255.255 area 0
 default-information originate
exit
ip classless
ip route 0.0.0.0 0.0.0.0 10.10.251.1
ip route 10.10.50.0 255.255.255.0 10.10.251.1
ip route 10.10.0.0 255.255.0.0 10.10.251.1
ip route 10.20.0.0 255.255.0.0 10.255.0.2
ip route 10.30.0.0 255.255.0.0 10.255.0.2
ip flow-export version 9
access-list 10 permit host 10.10.90.30
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

---

## PHẦN II: MẠNG TRUYỀN DẪN & HAI CHI NHÁNH DBP, BHTQ (10 THIẾT BỊ)

### 16. Router ISP2-R (Đường truyền dự phòng Broadband)
```text
enable
configure terminal
hostname ISP2-R
no ip domain-lookup
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip cef
no ipv6 cef
access-list 100 deny ip host 10.255.0.25 any
access-list 100 permit ip 10.20.0.0 0.0.255.255 10.10.0.0 0.0.255.255
access-list 100 permit ip 10.30.0.0 0.0.255.255 10.10.0.0 0.0.255.255
interface GigabitEthernet0/0
 description TO_WAN_R2
 ip address 10.255.0.25 255.255.255.252
 ip nat outside
 no shutdown
exit
interface GigabitEthernet0/1
 description BROADBAND_TO_DBP_R2
 ip address 10.255.0.17 255.255.255.252
 ip nat inside
 no shutdown
exit
interface GigabitEthernet0/2
 description BROADBAND_TO_BHTQ_R2
 ip address 10.255.0.21 255.255.255.252
 ip nat inside
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
ip classless
ip route 10.20.0.0 255.255.0.0 10.255.0.18
ip route 10.30.0.0 255.255.0.0 10.255.0.22
ip route 10.10.0.0 255.255.0.0 10.255.0.26
ip route 10.10.50.0 255.255.255.0 10.255.0.26
ip route 10.10.252.0 255.255.255.248 10.255.0.26
ip route 0.0.0.0 0.0.0.0 10.255.0.26
ip nat inside source list 100 interface GigabitEthernet0/0 overload
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 login
exit
end
write memory
```

### 17. Router CARRIER-R (Mạng lõi Leased Line chính)
```text
enable
configure terminal
hostname Carrier-R
no ip domain-lookup
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip cef
no ipv6 cef
interface GigabitEthernet0/0
 shutdown
exit
interface GigabitEthernet0/1
 shutdown
exit
interface GigabitEthernet0/2
 shutdown
exit
interface Serial0/2/0
 description TO_DBP_R1
 ip address 10.255.0.9 255.255.255.252
 ip ospf network point-to-point
 ip ospf priority 1
 clock rate 64000
 no shutdown
exit
interface Serial0/2/1
 description TO_BHTQ_R1
 ip address 10.255.0.13 255.255.255.252
 ip ospf network point-to-point
 clock rate 64000
 no shutdown
exit
interface Serial0/3/0
 description TO_WAN_R2
 ip address 10.255.0.6 255.255.255.252
 ip ospf network point-to-point
 clock rate 2000000
 no shutdown
exit
interface Serial0/3/1
 description TO_WAN_R1
 ip address 10.255.0.2 255.255.255.252
 ip ospf network point-to-point
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
access-list 10 deny 10.20.0.0 0.0.255.255
access-list 10 deny 10.30.0.0 0.0.255.255
access-list 10 permit any
router ospf 1
 router-id 1.1.1.3
 log-adjacency-changes
 passive-interface Serial0/3/0
 network 10.255.0.0 0.0.0.15 area 0
 distance 15
exit
router bgp 65000
 bgp log-neighbor-changes
 no synchronization
 neighbor 10.255.0.1 remote-as 65001
 neighbor 10.255.0.10 remote-as 65002
 neighbor 10.255.0.14 remote-as 65003
exit
ip classless
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 login
exit
end
write memory
```

### 18. Router DBP-R2 (Chi nhánh 1 - Dự phòng)
```text
enable
configure terminal
hostname DBP-R2
service timestamps log datetime msec
no ip domain-lookup
ip domain-name lab.test
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
ip cef
no ipv6 cef
crypto key generate rsa
2048
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
access-list 10 permit host 10.10.90.30
interface GigabitEthernet0/0
 description LAN_TO_SW1_RESERVED
 no ip address
 no shutdown
exit
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.20.10.3 255.255.255.0
 ip helper-address 10.20.40.10
 standby 10 ip 10.20.10.1
 standby 10 priority 105
 standby 10 preempt
exit
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.20.30.3 255.255.255.0
 ip helper-address 10.20.40.10
 standby 30 ip 10.20.30.1
 standby 30 priority 105
 standby 30 preempt
exit
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.20.40.3 255.255.255.0
 standby 40 ip 10.20.40.1
 standby 40 priority 105
 standby 40 preempt
exit
interface GigabitEthernet0/0.70
 encapsulation dot1Q 70
 ip address 10.20.70.3 255.255.255.0
 ip helper-address 10.20.40.10
 standby 70 ip 10.20.70.1
 standby 70 priority 105
 standby 70 preempt
exit
interface GigabitEthernet0/0.90
 encapsulation dot1Q 90
 ip address 10.20.90.3 255.255.255.0
 ip helper-address 10.20.40.10
 standby 90 ip 10.20.90.1
 standby 90 priority 105
 standby 90 preempt
exit
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99 native
 ip address 10.20.99.3 255.255.255.0
 ip helper-address 10.20.40.10
 ip ospf cost 100
 standby 99 ip 10.20.99.1
 standby 99 priority 105
 standby 99 preempt
exit
interface GigabitEthernet0/1
 description BROADBAND_TO_ISP2_R
 ip address 10.255.0.18 255.255.255.252
 ip ospf cost 200
 no shutdown
exit
interface GigabitEthernet0/2
 no ip address
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 passive-interface GigabitEthernet0/0.10
 passive-interface GigabitEthernet0/0.30
 passive-interface GigabitEthernet0/0.40
 passive-interface GigabitEthernet0/0.70
 passive-interface GigabitEthernet0/0.90
 network 10.20.0.0 0.0.255.255 area 0
 network 10.255.0.0 0.0.255.255 area 0
 network 10.20.10.0 0.0.1.255 area 0
 network 10.20.30.0 0.0.0.255 area 0
 network 10.20.40.0 0.0.0.255 area 0
 network 10.20.99.0 0.0.0.255 area 0
 network 10.255.0.16 0.0.0.3 area 0
 default-information originate
exit
ip classless
ip route 0.0.0.0 0.0.0.0 10.255.0.17 150
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
write memory
```

### 19. Router DBP-R1 (Chi nhánh 1 - Chính)
```text
enable
configure terminal
hostname DBP-R1
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
no ip domain-lookup
ip domain-name lab.test
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
ip cef
no ipv6 cef
spanning-tree mode pvst
interface GigabitEthernet0/0
 description TRUNK_TO_SW2
 no ip address
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/0.10
 description STAFF
 encapsulation dot1Q 10
 ip address 10.20.10.2 255.255.255.0
 ip helper-address 10.20.40.10
 standby 10 ip 10.20.10.1
 standby 10 priority 110
 standby 10 preempt
 standby 10 track Serial0/3/0
exit
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.20.30.2 255.255.255.0
 ip helper-address 10.20.40.10
 standby 30 ip 10.20.30.1
 standby 30 priority 110
 standby 30 preempt
 standby 30 track Serial0/3/0
exit
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.20.40.2 255.255.255.0
 standby 40 ip 10.20.40.1
 standby 40 priority 110
 standby 40 preempt
 standby 40 track Serial0/3/0
exit
interface GigabitEthernet0/0.70
 encapsulation dot1Q 70
 ip address 10.20.70.2 255.255.255.0
 ip helper-address 10.20.40.10
 standby 70 ip 10.20.70.1
 standby 70 priority 110
 standby 70 preempt
 standby 70 track Serial0/3/0
exit
interface GigabitEthernet0/0.90
 encapsulation dot1Q 90
 ip address 10.20.90.2 255.255.255.0
 ip helper-address 10.20.40.10
 ip access-group 110 in
 standby 90 ip 10.20.90.1
 standby 90 priority 110
 standby 90 preempt
 standby 90 track Serial0/3/0
exit
interface GigabitEthernet0/0.99
 description NATIVE
 encapsulation dot1Q 99 native
 ip address 10.20.99.2 255.255.255.0
 ip helper-address 10.20.40.10
 ip ospf cost 100
 standby 99 ip 10.20.99.1
 standby 99 priority 110
 standby 99 preempt
 standby 99 track Serial0/3/0
exit
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
exit
interface Serial0/3/0
 description LEASED_LINE_TO_CARRIER
 ip address 10.255.0.10 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
 no shutdown
exit
interface Serial0/3/1
 no ip address
 clock rate 2000000
 shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
router ospf 1
 router-id 2.2.2.1
 log-adjacency-changes
 passive-interface GigabitEthernet0/0.10
 passive-interface GigabitEthernet0/0.30
 passive-interface GigabitEthernet0/0.40
 passive-interface GigabitEthernet0/0.70
 passive-interface GigabitEthernet0/0.90
 network 10.20.0.0 0.0.255.255 area 0
 network 10.255.0.8 0.0.0.3 area 0
 network 10.20.99.0 0.0.0.255 area 0
exit
router bgp 65002
 bgp log-neighbor-changes
 no synchronization
 network 10.20.0.0 mask 255.255.0.0
 network 10.20.10.0 mask 255.255.255.0
 network 10.255.0.8 mask 255.255.255.252
exit
ip classless
ip flow-export version 9
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
copy running-config startup-config
```

### 20. Router BHTQ-R1 (Chi nhánh 2 - Chính)
```text
enable
configure terminal
hostname BHTQ-R1
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
ip dhcp excluded-address 10.30.10.1 10.30.10.49
ip dhcp excluded-address 10.30.10.240 10.30.10.254
ip dhcp pool BHTQ-R1_STAFF
 network 10.30.10.0 255.255.255.0
 default-router 10.30.10.1
 dns-server 10.10.40.10
exit
ip cef
no ipv6 cef
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
spanning-tree mode pvst
interface GigabitEthernet0/0
 description TRUNK_TO_SW2
 no ip address
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.30.10.2 255.255.255.0
 ip helper-address 10.30.40.10
 standby 10 ip 10.30.10.1
 standby 10 priority 110
 standby 10 preempt
 standby 10 track Serial0/3/0
exit
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.30.30.2 255.255.255.0
 ip helper-address 10.30.40.10
 standby 30 ip 10.30.30.1
 standby 30 priority 110
 standby 30 preempt
 standby 30 track Serial0/3/0
exit
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.30.40.2 255.255.255.0
 standby 40 ip 10.30.40.1
 standby 40 priority 110
 standby 40 preempt
 standby 40 track Serial0/3/0
exit
interface GigabitEthernet0/0.70
 description CAMERA
 encapsulation dot1Q 70
 ip address 10.30.70.2 255.255.255.0
 ip helper-address 10.30.40.10
 standby 70 ip 10.30.70.1
 standby 70 priority 110
 standby 70 preempt
 standby 70 track Serial0/3/0
exit
interface GigabitEthernet0/0.90
 description MANAGEMENT
 encapsulation dot1Q 90
 ip address 10.30.90.2 255.255.255.0
 ip helper-address 10.30.40.10
 ip access-group 110 in
 standby 90 ip 10.30.90.1
 standby 90 priority 110
 standby 90 preempt
 standby 90 track Serial0/3/0
exit
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99
 no ip address
 ip helper-address 10.30.40.10
 ip ospf cost 100
 standby 99 ip 10.30.99.1
 standby 99 priority 110
 standby 99 preempt
 standby 99 track Serial0/3/0
exit
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/1.1
 encapsulation dot1Q 1
 no ip address
exit
interface GigabitEthernet0/1.99
 encapsulation dot1Q 99 native
 no ip address
exit
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
exit
interface Serial0/3/0
 description LEASED_LINE_TO_CARRIER
 ip address 10.255.0.14 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 no shutdown
exit
interface Serial0/3/1
 no ip address
 clock rate 2000000
 shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
router ospf 1
 router-id 3.3.3.1
 log-adjacency-changes
 passive-interface GigabitEthernet0/0.10
 passive-interface GigabitEthernet0/0.30
 passive-interface GigabitEthernet0/0.40
 passive-interface GigabitEthernet0/0.70
 passive-interface GigabitEthernet0/0.90
 network 10.30.0.0 0.0.255.255 area 0
 network 10.255.0.12 0.0.0.3 area 0
 network 10.30.99.0 0.0.0.255 area 0
exit
router bgp 65003
 bgp log-neighbor-changes
 no synchronization
 network 10.30.0.0 mask 255.255.0.0
 network 10.30.10.0 mask 255.255.255.0
 network 10.255.0.12 mask 255.255.255.252
exit
ip classless
ip flow-export version 9
access-list 10 permit host 10.10.90.30
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
copy running-config startup-config
```

### 21. Router BHTQ-R2 (Chi nhánh 2 - Dự phòng)
```text
enable
configure terminal
hostname BHTQ-R2
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
username netadmin privilege 15 secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
ip cef
no ipv6 cef
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
spanning-tree mode pvst
interface GigabitEthernet0/0
 description LAN_TO_SW1_RESERVED
 no ip address
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.30.10.3 255.255.255.0
 ip helper-address 10.30.40.10
 standby 10 ip 10.30.10.1
 standby 10 priority 105
 standby 10 preempt
exit
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.30.30.3 255.255.255.0
 ip helper-address 10.30.40.10
 standby 30 ip 10.30.30.1
 standby 30 priority 105
 standby 30 preempt
exit
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.30.40.3 255.255.255.0
 standby 40 ip 10.30.40.1
 standby 40 priority 105
 standby 40 preempt
exit
interface GigabitEthernet0/0.70
 encapsulation dot1Q 70
 ip address 10.30.70.3 255.255.255.0
 ip helper-address 10.30.40.10
 standby 70 ip 10.30.70.1
 standby 70 priority 105
 standby 70 preempt
exit
interface GigabitEthernet0/0.90
 encapsulation dot1Q 90
 ip address 10.30.90.3 255.255.255.0
 ip helper-address 10.30.40.10
 standby 90 ip 10.30.90.1
 standby 90 priority 105
 standby 90 preempt
exit
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99 native
 ip address 10.30.99.3 255.255.255.0
 ip helper-address 10.30.40.10
 ip ospf cost 100
 standby 99 ip 10.30.99.1
 standby 99 priority 105
 standby 99 preempt
exit
interface GigabitEthernet0/1
 description BROADBAND_TO_ISP2_R
 ip address 10.255.0.22 255.255.255.252
 ip ospf cost 200
 duplex auto
 speed auto
 no shutdown
exit
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
router ospf 1
 router-id 3.3.3.2
 log-adjacency-changes
 passive-interface GigabitEthernet0/0.10
 passive-interface GigabitEthernet0/0.30
 passive-interface GigabitEthernet0/0.40
 passive-interface GigabitEthernet0/0.70
 passive-interface GigabitEthernet0/0.90
 network 10.30.0.0 0.0.255.255 area 0
 network 10.255.0.0 0.0.255.255 area 0
 default-information originate
exit
ip classless
ip route 0.0.0.0 0.0.0.0 10.255.0.21 150
ip flow-export version 9
access-list 10 permit host 10.10.90.30
logging 10.10.90.20
line con 0
 password 1
 logging synchronous
 login
exit
line aux 0
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
copy running-config startup-config
```

### 22. Switch DBP-SW1 (Chi nhánh 1)
```text
enable
configure terminal
hostname DBP-SW1
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
interface FastEthernet0/1
 description PC0
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0001.633B.7891
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/2
 description TO_OTHER_SW
 switchport access vlan 10
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,70,90,99
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/3
 description PC1
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 000A.F378.192A
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/4
 description PRINTER0
 switchport access vlan 10
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/24
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/1
 description TO_DBP_R2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/2
 switchport trunk native vlan 99
 switchport mode trunk
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.20.90.11 255.255.255.0
 no shutdown
exit
ip default-gateway 10.20.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
copy running-config startup-config
```

### 23. Switch DBP-SW2 (Chi nhánh 1)
```text
enable
configure terminal
hostname DBP-SW2
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
interface FastEthernet0/1
 description TO_OTHER_SW
 switchport access vlan 40
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,70,90,99
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/2
 switchport access vlan 70
 switchport trunk allowed vlan 10,70,90,99
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/3
 description AP_PT
 switchport access vlan 90
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/4
 description WEBCAM_DBP
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/5
 description PC_DBP3
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0001.64EB.2447
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/6
 switchport access vlan 30
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/24
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/1
 description TO_DBP_R1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,70,90,99
 switchport mode trunk
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.20.90.12 255.255.255.0
 no shutdown
exit
ip default-gateway 10.20.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
copy running-config startup-config
```

### 24. Switch BHTQ-SW2 (Chi nhánh 2)
```text
enable
configure terminal
hostname BHTQ-SW2
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
interface FastEthernet0/1
 switchport access vlan 30
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/2
 description PC_BHTQ3
 switchport access vlan 70
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 000D.BDCB.A8A1
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/3
 description WEBCAM_BHTQ
 switchport access vlan 90
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/4
 description AP_PT_BHTQ
 switchport access vlan 40
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/5
 description TO_OTHER_SW
 switchport access vlan 70
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,70,90,99
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/24
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/1
 description TO_BHTQ_R1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/2
 switchport trunk native vlan 99
 switchport mode trunk
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.30.90.12 255.255.255.0
 no shutdown
exit
ip default-gateway 10.30.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
copy running-config startup-config
```

### 25. Switch BHTQ-SW1 (Chi nhánh 2)
```text
enable
configure terminal
hostname BHTQ-SW1
enable secret 5 $1$mERr$Wmdu8FSDG1wNa1xa4SQGi.
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
no ip domain-lookup
ip domain-name lab.test
username netadmin secret 5 $1$mERr$o2Xuv2uH8EktviSiubpKv.
spanning-tree mode rapid-pvst
spanning-tree extend system-id
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/2
 description Printer3
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0090.0CCE.A098
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/3
 description PC_BHTQ2
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 000B.BEB7.C6D2
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/4
 description PC_BHTQ1
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0030.A338.EB0A
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit
interface FastEthernet0/5
 description TO_OTHER_SW
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,70,90,99
 switchport mode trunk
 no shutdown
exit
interface FastEthernet0/24
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/1
 description TO_BHTQ_R2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,30,40,70,90,99
 switchport mode trunk
 no shutdown
exit
interface GigabitEthernet0/2
 switchport trunk native vlan 99
 switchport mode trunk
 no shutdown
exit
interface Vlan1
 no ip address
 shutdown
exit
interface Vlan90
 description MANAGEMENT
 ip address 10.30.90.11 255.255.255.0
 no shutdown
exit
ip default-gateway 10.30.90.1
logging 10.10.90.20
access-list 10 permit host 10.10.90.30
line con 0
 password 1
 logging synchronous
 login
exit
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
exit
line vty 5 15
 access-class 10 in
 login local
 transport input ssh
exit
end
copy running-config startup-config
```