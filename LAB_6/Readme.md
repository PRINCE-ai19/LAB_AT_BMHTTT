HƯỚNG DẪN BÀI TẬP LAB 6: CẤU HÌNH MẠNG ẢO VÀ FIREWALL PFSENSE

Họ tên: Phạm Đỗ Hoàng Luân

MSSV: 1150070026

Lớp: 11_ĐH_TMĐT

link youtube :

Tài liệu này tóm tắt các bước thực hành cấu hình hệ thống mạng ảo trên VMware Workstation, thiết lập tường lửa phân vùng mạng với pfSense, và cài đặt dịch vụ trên Windows Server 2012.

Chi tiết các bước thực hiện qua hình ảnh

1. Cấu hình Virtual Network Editor (VMware Workstation)

VMnet2 (Host-only): Thiết lập dải IP 192.168.56.0/24. Kết nối card mạng ảo của máy thật vào mạng này để máy chủ host có thể giao tiếp trực tiếp với tường lửa pfSense.

VMnet3 (NAT): Cấu hình dải IP 192.168.250.0/24 kết hợp dịch vụ DHCP cục bộ, cho phép các máy ảo bên trong truy cập Internet thông qua card mạng của máy thật.

2. Cấu hình Card Mạng trên Máy thật (Host)

Cấu hình địa chỉ IPv4 tĩnh (192.168.56.1 / 255.255.255.0) cho card VMware Network Adapter VMnet2 để đảm bảo kết nối ổn định với giao diện quản trị của pfSense.

3. Phân bổ Giao diện Mạng (Interfaces) trên pfSense

WAN (em0): Giao diện kết nối ra mạng bên ngoài/Internet.

LAN (em2): Giao diện mạng nội bộ.

OPT1 / OPT2 (em1 / em3): Cấu hình các giao diện mạng bổ sung dùng cho phân vùng quản trị (MGMT) và vùng đệm (DMZ).

4. Thiết lập Địa chỉ IP cho các Interface qua Console

LAN Interface: Đặt địa chỉ IP 10.6.0.1/24, vô hiệu hóa DHCP Server trên interface này.

OPT2 Interface: Thiết lập địa chỉ IP 172.16.6.1/24 phục vụ cho phân vùng mạng riêng biệt.

5. Cấu hình trên giao diện WebGUI pfSense

Quản lý Interface: Đặt tên định danh (Description) cho các cổng mạng như MGMT và DMZ, chọn kiểu cấu hình Static IPv4 và lưu cấu hình (Apply Changes).

Firewall Rules: Tạo luật tường lửa cho phép tất cả các gói tin IPv4 đi qua giao diện MGMT (Allow All for WebGUI MGMT) nhằm phục vụ công tác quản trị thuận lợi.

6. Cài đặt Dịch vụ trên Windows Server 2012

Sử dụng Server Manager để tiến hành cài đặt thành phần hỗ trợ .NET Framework 3.5 Features (bao gồm .NET 2.0 và 3.0) phục vụ chạy các ứng dụng và dịch vụ trên máy chủ.