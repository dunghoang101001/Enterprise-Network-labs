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

## 3. Key Configuration

### Cấu hình dự phòng HSRP trên Multilayer Switch
```cisco
interface Vlan10
 mac-address 00d0.5803.9601
 ip address 172.16.1.1 255.255.255.128
 ip helper-address 172.16.3.132
 standby 1 ip 172.16.1.126
 standby 1 priority 15
 standby 1 preempt
!
interface Vlan20
 mac-address 00d0.5803.9603
 ip address 172.16.1.129 255.255.255.128
 standby 2 ip 172.16.1.254
 standby 2 priority 15
 standby 2 preempt
!
interface Vlan30
 mac-address 00d0.5803.9604
 ip address 172.16.2.1 255.255.255.128
 standby 3 ip 172.16.2.126
 standby 3 priority 15
 standby 3 preempt
!
interface Vlan40
 mac-address 00d0.5803.9605
 ip address 172.16.2.129 255.255.255.128
 standby 4 ip 172.16.2.254
 standby 4 priority 15
 standby 4 preempt
!
interface Vlan50
 mac-address 00d0.5803.9606
 ip address 172.16.3.1 255.255.255.128
 ip helper-address 172.16.3.132
 standby 5 ip 172.16.3.126
 standby 5 priority 15
 standby 5 preempt
!
interface Vlan60
 mac-address 00d0.5803.9607
 ip address 172.16.3.129 255.255.255.240
 standby 6 ip 172.16.3.143
 standby 6 priority 15
 standby 6 preempt
```

### Cấu hình định tuyến OSPF trên Core-R1
```cisco
router ospf 1
 log-adjacency-changes
 redistribute bgp 1 subnets 
 network 172.16.3.144 0.0.0.3 area 0
 network 172.16.3.152 0.0.0.3 area 0
```

---

## 4. Failover Verification Scenarios (Kịch bản kiểm tra và xác thực sự cố)

Để đảm bảo hệ thống có tính sẵn sàng cao (High-Availability) và hoạt động đúng thiết kế, các kịch bản kiểm tra lỗi (Failover Test) đã được thực hiện trực tiếp trên giả lập Cisco Packet Tracer với các bước và kết quả như sau:

### Kịch bản 1: Kiểm tra tính năng dự phòng Gateway (HSRP Failover)
* **Mục đích:** Đảm bảo khi một Switch Phân phối (Distribution Switch) gặp sự cố, máy trạm trong mạng LAN vẫn kết nối ra ngoài thông suốt thông qua Switch còn lại mà không cần thay đổi cấu hình Default Gateway.
* **Các bước thực hiện:**
  1. Chọn một máy trạm bất kỳ trong mạng nội bộ (ví dụ: PC thuộc phòng **SALE - VLAN 10**).
  2. Mở cửa sổ **Command Prompt** trên PC và chạy lệnh ping liên tục đến địa chỉ Server Room hoặc một IP bên ngoài: 
     ```bash
     ping -t 172.16.3.132 
     ```
  3. Trong lúc lệnh ping đang chạy, truy cập vào giao diện CLI của **Multilayer Switch 1** (thiết bị đang đóng vai trò `Active` cho VLAN 10) và chủ động tắt giao diện:
     ```cisco
     MLSwitch1(config)# interface Vlan10
     MLSwitch1(config-if)# shutdown
     ```
* **Kết quả xác thực (Expected Result):** * Tại màn hình ping của PC phòng SALE, tiến trình ping chỉ bị khựng lại (hiển thị `Request timed out`) từ **1 đến 2 gói tin**.
  * **Multilayer Switch 2** (đang ở trạng thái `Standby`) lập tức phát hiện mất tín hiệu từ thiết bị Active, tự động chuyển trạng thái từ `Standby` sang `Active` để gánh toàn bộ lưu lượng của VLAN 10.
  * Tiến trình ping trên PC tự động khôi phục (`Reply from...`), mạng nội bộ tiếp tục hoạt động bình thường.
  * **Multilayer Switch 1** sẽ chiếm lại quyền `Active` sau khi nó hoạt động trở lại vì được cấu hình preemt.

---

### Kịch bản 2: Kiểm tra dự phòng đường truyền Internet biên (WAN Failover)
* **Mục đích:** Xác thực khả năng định tuyến và dự phòng kênh truyền Internet biên. Khi đường truyền của nhà mạng chính (ISP1) bị đứt, toàn bộ lưu lượng đi ra Internet của doanh nghiệp phải tự động chuyển hướng qua đường truyền dự phòng (ISP2) một cách thông suốt.
* **Các bước thực hiện:**
  1. Từ một máy trạm nội bộ, thực hiện ping liên tục đến một địa chỉ IP giả lập Internet công cộng nằm sau các nhà mạng (ví dụ: tạo một IP Loopback `8.8.8.8` trên Router phía trên của ISP hoặc ping một vùng mạng chung mà cả 2 ISP đều tới được).
     ```bash
     ping -t 8.8.8.8 
     ```
  2. Truy cập vào Router biên chính **CORE-R1**, tiến hành tắt cổng kết nối vật lý hướng lên ISP1 (đường truyền chính):
     ```cisco
     CORE-R1(config)# interface Serial0/0/1  *(Cổng vật lý nối sang ISP1)*
     CORE-R1(config-if)# shutdown
     ```
* **Kết quả xác thực (Expected Result):**
  * Ngay khi cổng nối ISP1 bị shutdown, tuyến đường ưu tiên qua ISP1 trong bảng định tuyến sẽ bị xóa bỏ.
  * Nhờ giao thức định tuyến động, hệ thống tự động điều hướng lưu lượng đi vòng qua đường đồng bộ giữa 2 router biên sang **CORE-R2**, rồi đẩy qua **ISP2** để đi ra Internet.
  * Kiểm tra tiến trình ping trên máy trạm: Lệnh ping tới địa chỉ Internet (`8.8.8.8`) chỉ bị mất từ **1 đến 3 gói tin** trong quá trình chuyển tuyến (Convergence time), sau đó tự động có mạng lại (`Reply từ 8.8.8.8...`), chứng minh đường truyền đã backup thành công.
