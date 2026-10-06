# Hướng Dẫn Cấu Hình Lab pfSense & Mạng Máy Ảo (VMware Workstation)

### 1. Thông tin sinh viên
- **Họ và tên:** Phạm Đỗ Hoàng Luân
- **MSSV:** 1150070026
- **Lớp:** 11_TMĐT
- **Link Video YouTube:** https://www.youtube.com/watch?v=UUzvgiXcS2A

---

Tài liệu này tóm tắt các bước triển khai bài lab cấu hình tường lửa pfSense, thiết lập Virtual Network trên VMware Workstation, cấu hình máy khách Windows 11 và kiểm tra kết nối.

---

##  Danh Sách Cần Chuẩn Bị
1. **VMware Workstation** (phiên bản 17.5 trở lên).
2. **Máy ảo pfSense** (Phiên bản 2.7.2-RELEASE).
3. **Máy ảo khách (Client):** Windows 11 x64.

---

##  Các Bước Thực Hiện

### Bước 1: Thiết Lập Mạng Ảo Trái Đất (Virtual Network Editor)
 Mở **VMware Workstation** chọn **Edit** **Virtual Network Editor**:

1. **VMnet1 (Host-Only):**
   * **Type:** Host-only (kết nối nội bộ giữa máy thật và máy ảo).
   * **Subnet IP:** `10.0.0.0`
   * **Subnet Mask:** `255.0.0.0`
   * **DHCP:** Disabled.
2. **VMnet2 (Host-Only):**
   * **Type:** Host-only (Dành cho vùng DMZ/DZM).
   * **Subnet IP:** `172.16.0.0`
3. **VMnet8 (NAT):**
   * **Type:** NAT (Cung cấp Internet cho mô hình).
   * **Subnet IP:** `192.168.68.0`

---

### Bước 2: Cấu Hình Card Mạng Trên Máy Thật (Host Network Adapter)
1. Trên Windows thật, mở **Control Panel** . **Network Connections**.
2. Chọn card mạng `VMware Network Adapter VMnet1` . chọn **Properties** . **IPv4**:
   * **IP address:** `10.0.0.100`
   * **Subnet mask:** `255.0.0.0`
   * **Default gateway:** *(Để trống)*

---

### Bước 3: Cấu Hình Phần Cứng Máy Ảo pfSense
Mở cài đặt máy ảo pfSense (**Virtual Machine Settings**) và gắn 3 Card mạng:

1. **Network Adapter 1 (WAN):** Chọn `Bridged (Automatic)` hoặc `NAT`.
2. **Network Adapter 2 (LAN):** Chọn `Custom (VMnet1)`.
3. **Network Adapter 3 (DZM/DMZ):** Chọn `Custom (VMnet2)`.

---

### Bước 4: Thiết Lập Ban Đầu Và Truy Cập Web GUI pfSense
1. Khởi động máy ảo pfSense.
2. Từ trình duyệt trên máy thật, truy cập địa chỉ IP mặc định của pfSense: `https://10.0.0.1`
3. **Đăng nhập:**
   * **Username:** `admin`
   * **Password:** `pfsense` *(hoặc mật khẩu khởi tạo của bạn)*
4. Hoàn thành các bước trong **pfSense Setup Wizard** (Step 1 đến Step 9) để kết thúc cấu hình ban đầu.

---

### Bước 5: Cấu Hình Interface & Firewall Rules บน pfSense

#### 5.1. Cấu hình Interface DZM (DMZ)
1. Vào **Interfaces** . Chọn **DZM (em2)**.
2. Tích chọn **Enable interface**.
3. **IPv4 Configuration Type:** Chọn `Static IPv4`.
4. Nhấn **Save** và **Apply Changes**.

#### 5.2. Cấu hình Firewall Rules
1. Vào **Firewall** . **Rules** . **LAN**:
   * Xác nhận có quy tắc **Anti-Lockout Rule** (cho phép truy cập Admin GUI qua port 80/443).
   * Kiểm tra quy tắc `Default allow LAN to any rule`.
2. Vào **Firewall** . **Rules** . **Edit** (thêm/sửa rule WAN):
   * **Action:** `Pass`
   * **Interface:** `WAN`
   * **Address Family:** `IPv4`
   * **Protocol:** `TCP/UDP` (hoặc `TCP`)
   * **Source:** `LAN subnets`
3. **Reset State Table:** Vào **Diagnostics** . **States** . chọn tab **Reset States**  Nhấn **Reset** để áp dụng thay đổi toàn bộ kết nối.

---

### Bước 6: Cấu Hình Máy Khách (Windows 11 Client)
1. Gắn Card mạng của VM Windows 11 vào **VMnet1**.
2. Bật máy ảo Windows 11, vào **Network Connections****Ethernet0 Properties**  **IPv4**:
   * **IP address:** `10.0.0.2`
   * **Subnet mask:** `255.0.0.0`
   * **Default gateway:** `10.0.0.1`
   * **Preferred DNS server:** `8.8.8.8`

---

### Bước 7: Kiểm Tra Kết Nối (Verification)
Mở **Command Prompt (cmd)** trên Windows 11 và chạy các lệnh kiểm tra:

1. **Kiểm tra PING (ICMP Traffic):**
   ```cmd
   ping 8.8.8.8
   ```
   * *Kết quả kỳ vọng:* `Request timed out` (Do tường lửa chặn ICMP/Ping ra ngoài).

2. **Kiểm tra Phân giải Tên miền (DNS):**
   ```cmd
   nslookup example.com 8.8.8.8
   ```
   * *Kết quả kỳ vọng:* Trả về thông tin IP thành công (ví dụ: `172.66.147.243`, `104.20.23.154`).

3. **Kiểm tra Kết nối HTTP/HTTPS:**
   ```cmd
   curl -4 https://example.com
   ```
   * *Kết quả kỳ vọng:* Trả về nội dung mã nguồn HTML của trang `example.com`.

---

##  Ghi Chú Cần Lưu Ý
* Lưu ý bấm **Apply Changes** sau mỗi lần thay đổi cấu hình trên giao diện Web GUI của pfSense.
* Nếu máy khách không truy cập được Internet, hãy thực hiện thao tác **Reset States** trong mục Diagnostics của pfSense.