# Enterprise Network Architecture Design (High-Availability)

## 1. Network Topology
Đây là sơ đồ kiến trúc mạng doanh nghiệp 3 lớp được thiết kế và giả lập trên Cisco Packet Tracer:

![Enterprise Network Topology](enterprise%20topology.png)

---

## 2. IP Planning Specification

### Phân vùng các phòng ban nội bộ (LAN & WLAN Segments)
Hệ thống mạng LAN và WLAN được phân chia theo kiến trúc các tầng văn phòng, sử dụng cơ chế dự phòng **HSRP (Hot Standby Router Protocol) trực tiếp trên cặp Multilayer Switch (Layer 3 Switch)** ở lớp Phân phối (Distribution) để làm Default Gateway cho người dùng cuối:

| Tầng (Floor) | Phòng ban (Department) | Dải IP Network (Subnet) | Default Gateway (IP ảo HSRP) | Ghi chú (Chức năng) |
| :--- | :--- | :--- | :--- | :--- |
| **First Floor** | **SALE** | `172.16.1.0/25` | `172.16.1.126` | Dành cho PC và AP phòng Kinh doanh |
| **First Floor** | **HR** | `172.16.1.128/25` | `172.16.1.254` | Dành cho PC và AP phòng Nhân sự |
| **Second Floor**| **FINANCE** | `172.16.2.0/25` | `172.16.2.126` | Dành cho PC và AP phòng Kế toán |
| **Second Floor**| **ADMIN** | `172.16.2.128/25` | `172.16.2.254` | Dành cho PC và AP ban Hành chính |
| **Third Floor**| **IT** | `172.16.3.0/25` | `172.16.3.126` | Dành cho PC và AP phòng IT |

### Phân vùng phòng máy chủ cô lập (Server Room)
Vùng Server được cô lập riêng biệt để tối ưu bảo mật, có thể quản lý tập trung và áp dụng các chính sách ACL:

| Vị trí | Tên vùng | Dải IP Network (Subnet) | Default Gateway (IP ảo HSRP) | Ghi chú |
| :--- | :--- | :--- | :--- | :--- |
| **Third Floor** | **SERVER-ROOM** | `172.16.3.128/28` | `172.16.3.143` | Chứa Server0, Server1 và PC1 quản trị hệ thống |

### Phân vùng kết nối hạ tầng lõi (Core & Uplink Interconnections)
Các dải IP sử dụng subnet `/30` chuyên dụng cho các đường kết nối Point-to-Point chạy định tuyến động giữa lớp Multilayer Switch và Core Router:

| Đường kết nối (Link) | Dải IP Network | IP Thiết bị A | IP Thiết bị B | Ghi chú |
| :--- | :--- | :--- | :--- | :--- |
| **Multilayer1 <-> CORE-R1** | `172.16.3.144/30` | `172.16.3.145` (SW1) | `172.16.3.146` (R1) | Đường Uplink chính của SW1 |
| **Multilayer1 <-> CORE-R2** | `172.16.3.148/30` | `172.16.3.149` (SW1) | `172.16.3.150` (R2) | Đường Uplink dự phòng của SW1 |
| **Multilayer2 <-> CORE-R1** | `172.16.3.152/30` | `172.16.3.153` (SW2) | `172.16.3.154` (R1) | Đường Uplink dự phòng của SW2 |
| **Multilayer2 <-> CORE-R2** | `172.16.3.156/30` | `172.16.3.157` (SW2) | `172.16.3.158` (R2) | Đường Uplink chính của SW2 |

### Phân vùng kết nối Internet Biên (WAN Blocks)
Kết nối từ các Router biên của công ty lên hệ thống của hai nhà mạng độc lập (ISP1 và ISP2):

| Giao diện kết nối | Dải IP WAN (Do ISP cấp) | IP Router Biên | IP phía Nhà mạng (Gateway) | Ghi chú |
| :--- | :--- | :--- | :--- | :--- |
| **CORE-R1 <-> ISP1** | `195.136.17.4/30` | `195.136.17.5` | `195.136.17.6` | Hướng đi Internet qua ISP 1 (Băng thông chính) |
| **CORE-R2 <-> ISP2** | `195.136.17.12/30` | `195.136.17.13` | `195.136.17.14` | Hướng đi Internet qua ISP 2 (Băng thông dự phòng) |
