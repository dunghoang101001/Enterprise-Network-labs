# Enterprise Network Architecture Design (High-Availability)

## 1. Network Topology
Đây là sơ đồ kiến trúc mạng doanh nghiệp 3 lớp được thiết kế và giả lập trên Cisco Packet Tracer:

![Enterprise Network Topology](enterprise%20topology.png)

---

## 2. IP Planning Specification

### Phân vùng các phòng ban nội bộ (LAN & WLAN Segments)
| Tầng (Floor) | Phòng ban (Department) | Dải IP Network (Subnet) | Default Gateway (IP ảo HSRP) | Ghi chú (Chức năng) |
| :--- | :--- | :--- | :--- | :--- |
| **First Floor** | **SALE** | `192.168.1.0/25` | `192.168.1.1` | Dành cho PC và AP phòng Kinh doanh |
| **First Floor** | **HR** | `192.168.1.128/25` | `192.168.1.129` | Dành cho PC và AP phòng Nhân sự |
| **Second Floor**| **FINANCE** | `192.168.2.0/25` | `192.168.2.1` | Dành cho PC và AP phòng Kế toán |
| **Second Floor**| **ADMIN** | `192.168.2.128/25` | `192.168.2.129` | Dành cho PC và AP ban Hành chính |
| **Third Floor** | **IT** | `192.168.3.0/25` | `192.168.3.1` | Phân vùng quản trị của phòng IT |

### Phân vùng phòng máy chủ cô lập (Server Room)
| Vị trí | Tên vùng | Dải IP Network (Subnet) | Default Gateway | Ghi chú |
| :--- | :--- | :--- | :--- | :--- |
| **Third Floor** | **SERVER-ROOM** | `192.168.3.128/28` | `192.168.3.129` | Chứa Server0, Server1 và PC1 quản trị hệ thống |

### Phân vùng kết nối hạ tầng lõi (Core & Uplink Interconnections)
| Đường kết nối (Link) | Dải IP Network | IP Thiết bị A | IP Thiết bị B | Ghi chú |
| :--- | :--- | :--- | :--- | :--- |
| **Multilayer1 <-> CORE-R1** | `192.168.3.144/30` | `192.168.3.145` (SW1) | `192.168.3.146` (R1) | Đường Uplink chính của SW1 |
| **Multilayer1 <-> CORE-R2** | `192.168.3.148/30` | `192.168.3.149` (SW1) | `192.168.3.150` (R2) | Đường Uplink dự phòng của SW1 |
| **Multilayer2 <-> CORE-R1** | `192.168.3.152/30` | `192.168.3.153` (SW2) | `192.168.3.154` (R1) | Đường Uplink dự phòng của SW2 |
| **Multilayer2 <-> CORE-R2** | `192.168.3.156/30` | `192.168.3.157` (SW2) | `192.168.3.158` (R2) | Đường Uplink chính của SW2 |

### Phân vùng kết nối Internet Biên (WAN Blocks)
| Giao diện kết nối | Dải IP WAN (Do ISP cấp) | IP Router Biên | IP phía Nhà mạng (Gateway) | Ghi chú |
| :--- | :--- | :--- | :--- | :--- |
| **CORE-R1 <-> ISP1** | `195.136.17.4/30` | `195.136.17.5` | `195.136.17.6` | Hướng đi Internet qua ISP 1 (Băng thông chính) |
| **CORE-R2 <-> ISP2** | `195.136.17.12/30` | `195.136.17.13` | `195.136.17.14` | Hướng đi Internet qua ISP 2 (Băng thông dự phòng) |
| **Đường đồng bộ ISP** | `195.136.17.8/30` | Kết nối liên thông | Giữa hai Router biên | Định tuyến dự phòng chéo |
