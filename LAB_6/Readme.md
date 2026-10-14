HƯỚNG DẪN BÀI TẬP LAB 6: XÁC THỰC ACTIVE DIRECTORY VÀ CHÍNH SÁCH TRUY CẬP TRÊN pfSense
Môi trường triển khai: VMware Workstation Pro · pfSense CE 2.7.2 · Windows Server 2012 (AD DS/DNS) · Windows 11 (Client) · Kali Linux (DMZ-WEB)

Họ tên: Phạm Đỗ Hoàng Luân

MSSV: 1150070026

Lớp: 11_ĐH_TMĐT

link youtube :https://www.youtube.com/watch?v=EHlSyIPxp7w

 II. SƠ ĐỒ MẠNG & THIẾT LẬP PHÂN VÙNG (TOPOLOGY)

### 1. Phân bổ Giao diện (Interfaces) trên pfSense
* **Giao diện WAN (`em0`):**
  * Tên Card Virtual: VMnet3 (NAT)
  * Địa chỉ IP / Subnet: `192.168.250.0/24` (Nhận IP qua DHCP)
  * Chức năng: Kết nối mạng Internet bên ngoài.
* **Giao diện MGMT (`em1`):**
  * Tên Card Virtual: VMnet2 (Host-Only)
  * Địa chỉ IP / Subnet: `192.168.56.254/24`
  * Chức năng: Vùng quản trị riêng biệt dành cho việc truy cập giao diện WebGUI pfSense.
* **Giao diện LAN (`em2`):**
  * Tên Card Virtual: LAN Segment (`lab6-lan`)
  * Địa chỉ IP / Subnet: `10.6.0.1/24`
  * Chức năng: Mạng nội bộ kết nối Domain Controller và các máy trạm Client.
* **Giao diện DMZ (`em3`):**
  * Tên Card Virtual: LAN Segment (`lab6-dmz`)
  * Địa chỉ IP / Subnet: `172.16.6.1/24`
  * Chức năng: Phân vùng chứa các máy chủ dịch vụ Web Kali Linux.

### 2. Phân bổ Địa chỉ IP Thiết bị Đầu cuối
* **Máy thật Quản trị:** `192.168.56.1/24` (VMnet2)
* **Domain Controller (DC01):** `10.6.0.10/24` (Default Gateway: `10.6.0.1`, Primary DNS: `10.6.0.10`)
* **Client-A (Windows 11):** `10.6.0.21/24`
* **Client-B:** `10.6.0.22/24`
* **DMZ-WEB (Kali Linux Virtual Hosts):**
  * `172.16.6.10` - Trang web tin tức chung (`GENERAL_SITE`)
  * `172.16.6.20` - Trang thông tin tuyển dụng (`JOB_SITE`)
  * `172.16.6.30` - Trang mạng xã hội (`SOCIAL_SITE`)
  * `172.16.6.40` - Trang web nhạy cảm/cấm truy cập (`RESTRICTED_SITE`)

---

## III. NỘI DUNG TRIỂN KHAI CHÍNH

### 1. Cấu hình Mạng ảo và Quản trị pfSense
* Tạo mạng **Host-only (VMnet2)** phục vụ giao diện MGMT (`192.168.56.0/24`) và **NAT (VMnet3)** phục vụ giao diện WAN (`192.168.250.0/24`).
* Gán 4 card mạng vật lý tương ứng (`em0` - WAN, `em1` - MGMT, `em2` - LAN, `em3` - DMZ) ngay trên Console điều khiển pfSense.
* Bật Rule mở khóa toàn bộ quyền truy cập cho giao diện MGMT (`192.168.56.254`), cô lập hoàn toàn việc quản trị hệ thống ra khỏi phân vùng LAN (Mô hình Out-of-band Management).

### 2. Dựng Active Directory & Tích hợp LDAP
* Thiết lập Windows Server 2012 nâng cấp thành Domain Controller quản lý miền `NHATNGHE.LOCAL`.
* Khởi tạo Organizational Unit (OU) tên `LAB6-Users` chứa các tài khoản:
  * Tài khoản người dùng kiểm thử: `sv_a` và `sv_b`.
  * Tài khoản dịch vụ liên kết: `svc_pfsense_ldap` (Tài khoản thuộc nhóm Domain Users dùng để xác thực LDAP Bind).
* Thêm **Authentication Server** trên pfSense kết nối qua chuẩn **LDAP**, cổng TCP 389, Search Scope `Entire Subtree`.
* Kiểm tra tính đúng đắn của kết nối LDAP thông qua công cụ *Diagnostics > Authentication* trên pfSense đạt kết quả phản hồi thành công.

### 3. Cấu hình Captive Portal Xác thực miền
* Khởi tạo Captive Portal Zone `LAB6_USERS` hoạt động trên giao diện LAN.
* Đặt phương thức xác thực trỏ đến Authentication Server LDAP `LAB6-AD`.
* Yêu cầu tất cả kết nối từ máy trạm LAN khi mở Web phải qua trang đăng nhập bằng tài khoản Active Directory trước khi được cấp quyền ra ngoài.

### 4. Định nghĩa Đối tượng Mạng (Aliases & Schedules)
* **IP Aliases:**
  * `CLIENT_A`: Địa chỉ IP `10.6.0.21`
  * `CLIENT_B`: Địa chỉ IP `10.6.0.22`
  * `POLICY_CLIENTS`: Nhóm địa chỉ chứa cả `CLIENT_A` và `CLIENT_B`.
  * `GENERAL_SITE`: Địa chỉ `172.16.6.10`
  * `JOB_SITE`: Địa chỉ `172.16.6.20`
  * `SOCIAL_SITE`: Địa chỉ `172.16.6.30`
  * `RESTRICTED_SITE`: Địa chỉ `172.16.6.40`
* **Port Aliases:**
  * `WEB_PORTS`: Định nghĩa cổng HTTP (`80`) và HTTPS (`443`).
* **Schedules (Khung giờ áp dụng):**
  * `LUNCH_WINDOW`: Khung giờ nghỉ trưa từ 11:30 đến 13:00.
  * `WORK_HOURS`: Khung giờ làm việc từ 08:00 đến 17:00.
  * `TEST_NOW`: Khung giờ thực tế cài đặt để kiểm thử lập tức kịch bản.

---

## IV. TẬP LUẬT TƯỜNG LƯA (FIREWALL RULES)

### 1. Tập luật trên Tab LAN (Sắp xếp theo thứ tự ưu tiên First-Match)
* **Rule 1 (Block):** Chặn Client-A truy cập trang tuyển dụng.
  * Hành động: Block (Có ghi Log)
  * Nguồn (Source): `CLIENT_A`
  * Đích (Destination): `JOB_SITE`, Cổng `80`
* **Rule 2 (Block):** Chặn toàn bộ máy trạm truy cập trang web nhạy cảm.
  * Hành động: Block (Có ghi Log)
  * Nguồn (Source): `POLICY_CLIENTS`
  * Đích (Destination): `RESTRICTED_SITE`, Cổng `80`
* **Rule 3 (Pass):** Cho phép Client-B lướt mạng xã hội trong giờ nghỉ.
  * Hành động: Pass
  * Nguồn (Source): `CLIENT_B`
  * Đích (Destination): `SOCIAL_SITE`, Cổng `80`
  * Khung giờ: Áp dụng theo `TEST_NOW` / `LUNCH_WINDOW`
* **Rule 4 (Block):** Chặn Client-B lướt mạng xã hội ngoài giờ nghỉ.
  * Hành động: Block (Có ghi Log)
  * Nguồn (Source): `CLIENT_B`
  * Đích (Destination): `SOCIAL_SITE`, Cổng `80`
* **Rule 5 (Pass):** Mở quyền truy cập trang tin tức chung cho các máy trạm.
  * Hành động: Pass
  * Nguồn (Source): `POLICY_CLIENTS`
  * Đích (Destination): `GENERAL_SITE`, Cổng `80`
* **Rule 6 (Pass):** Mở truy cập dịch vụ Web chung trong giờ làm việc.
  * Hành động: Pass
  * Nguồn (Source): `POLICY_CLIENTS`
  * Đích (Destination): Bất kỳ (`*`), Cổng `WEB_PORTS`
  * Khung giờ: Áp dụng theo `WORK_HOURS`
* **Rule 7 (Block):** Luật chặn ngầm định toàn bộ lưu lượng khác (Explicit Deny).
  * Hành động: Block (Có ghi Log)
  * Nguồn (Source): `POLICY_CLIENTS`
  * Đích (Destination): Bất kỳ (`*`), Cổng `*`

### 2. Tập luật trên Tab DMZ
* **Hành động:** Block
* **Nguồn (Source):** `DMZ subnets`
* **Đích (Destination):** `LAN subnets`
* **Mục đích:** Khóa hoàn toàn chiều kết nối khởi tạo từ phân vùng DMZ sang vùng LAN nội bộ, ngăn chặn nguy cơ máy chủ bị tấn công chiếm quyền trong DMZ quay sang xâm nhập LAN.

---

## V. KẾT QUẢ THỰC NGHIỆM CHI TIẾT THEO KỊCH BẢN

* **Kiểm thử Xác thực LDAP:**
  * Yêu cầu: Xác thực tài khoản Active Directory `sv_a` và `sv_b` trên giao diện pfSense.
  * Kết quả thực tế: Hệ thống trả về thông báo *User authenticated successfully* (Màu xanh).
  * Đánh giá: **ĐẠT**

* **Kiểm thử Captive Portal:**
  * Yêu cầu: Bắt buộc mở trang đăng nhập khi thiết bị vùng LAN duyệt Web.
  * Kết quả thực tế: Trình duyệt tự động chuyển hướng về trang Portal (cổng 8002). Thực hiện nhập tài khoản `sv_a` thành công và phiên làm việc được ghi nhận trên danh sách Captive Portal Status.
  * Đánh giá: **ĐẠT**

* **Kịch bản 1 - Chặn Client-A vào Job Site:**
  * Yêu cầu: Client-A bị chặn khi truy cập `JOB_SITE` (`172.16.6.20`), nhưng phải truy cập được `GENERAL_SITE` (`172.16.6.10`).
  * Kết quả thực tế: Truy cập `.10` tải trang bình thường; truy cập `.20` bị dừng và hiển thị lỗi *Connection timed out*.
  * Đánh giá: **ĐẠT**

* **Kịch bản 2 - Cấm các Client vào Restricted Site:**
  * Yêu cầu: Cấm cả Client-A và Client-B truy cập vào `RESTRICTED_SITE` (`172.16.6.40`).
  * Kết quả thực tế: Cả 2 máy trạm đều bị hết thời gian chờ (timeout) khi vào trang `.40`, trong khi các trang web cho phép khác vẫn tải bình thường.
  * Đánh giá: **ĐẠT**

* **Kịch bản 3 - Giới hạn giờ lướt Mạng xã hội của Client-B:**
  * Yêu cầu: Client-B chỉ được truy cập `SOCIAL_SITE` (`172.16.6.30`) trong khung giờ nghỉ.
  * Kết quả thực tế: Khi trong khung giờ `TEST_NOW`, Client-B mở trang `.30` thành công. Khi tắt Rule 3 (ngủ ngoài giờ), Rule 4 lập tức chặn kết nối.
  * Đánh giá: **ĐẠT**

* **Kịch bản 4 - Chỉ mở dịch vụ Web, cấm lệnh Ping:**
  * Yêu cầu: Chỉ cho phép truy cập Web (cổng 80/443), cấm toàn bộ lưu lượng ICMP Ping.
  * Kết quả thực tế: Máy trạm duyệt Web trang `.10` bình thường, tuy nhiên khi thực hiện lệnh `ping 172.16.6.10` bằng CMD thì nhận kết quả *Request timed out* (mất gói 100%).
  * Đánh giá: **ĐẠT**

* **Kịch bản 5 - Bật tính năng cô lập phân vùng DMZ:**
  * Yêu cầu: LAN chủ động truy cập được DMZ, nhưng DMZ không thể chủ động gửi kết nối hoặc Ping về LAN.
  * Kết quả thực tế: Windows 11 (LAN) duyệt trang Web trên Kali Linux (DMZ) bình thường; ngược lại Kali Linux thực hiện lệnh Ping tới Domain Controller (`10.6.0.10`) bị chặn hoàn toàn.
  * Đánh giá: **ĐẠT**

---

## VI. CÂU HỎI PHÂN TÍCH VÀ CƠ CHẾ KỸ THUẬT

1. **Nguyên tắc First-Match:**
   Tường lửa duyệt tập luật từ trên xuống dưới theo thứ tự xuất hiện. Ngay khi gói tin khớp với điều kiện của một luật đầu tiên, hành động tương ứng sẽ được áp dụng ngay lập tức và dừng quá trình kiểm tra các luật phía dưới.

2. **Cơ chế Stateful Firewall và Nút Reset States:**
   pfSense lưu thông tin các kết nối đang hoạt động trong bảng trạng thái (State Table) trên RAM. Khi thay đổi hoặc cập nhật luật tường lửa mới, cần bấm **Reset States** để xóa bộ nhớ đệm, bắt buộc hệ thống phải đánh giá lại các kết nối hiện tại theo danh sách ruleset mới.

3. **So sánh Hành động Block và Reject:**
   * **Block (Drop):** Âm thầm loại bỏ gói tin mà không gửi lại thông báo cho nguồn gửi. Trình duyệt phía Client sẽ phản hồi chậm và báo lỗi *Connection timed out*.
   * **Reject:** Loại bỏ gói tin đồng thời gửi trả lại tín hiệu từ chối (TCP RST hoặc ICMP Destination Unreachable). Trình duyệt phía Client sẽ nhận diện lập tức và hiển thị lỗi bị từ chối ngay.

4. **Sự khác biệt giữa Lọc Layer 3/4 và Web Category Filtering (Layer 7):**
   * Lọc theo IP Alias (Layer 3/4) dựa trên việc chặn các địa chỉ IP cố định, chỉ phù hợp trong quy mô lab hoặc mạng nội bộ nhỏ.
   * Lọc danh mục Web (Layer 7 Web Filter) truy vấn cơ sở dữ liệu đám mây để phân loại tên miền (FQDN), đường dẫn URL và giao thức SNI/DNS, cho phép chặn chính xác các nhóm trang web động trên môi trường Internet thực tế.

---
