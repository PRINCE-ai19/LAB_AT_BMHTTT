# LAB 3 Quét Cổng & Nhận Diện Dịch Vụ Với Nmap

### 1. Thông tin sinh viên
- **Họ và tên:** Phạm Đỗ Hoàng Luân
- **MSSV:** 1150070026
- **Lớp:** 11_TMĐT
- **Link Video YouTube:** https://www.youtube.com/watch?v=o5pZh010pd8

---


Tài liệu hướng dẫn chi tiết quy trình thiết lập môi trường mạng cô lập và thực hiện kỹ thuật rà quét cổng, xác định phiên bản dịch vụ sử dụng Nmap trên VMware Workstation giữa máy tấn công (Kali Linux) và máy mục tiêu (Metasploitable 2).

1. Môi Trường Thực Hành

Nền tảng ảo hóa VMware Workstation

Chế độ mạng Host-only (VMnet1) - Subnet 192.168.14.024

Máy tấn công (Attacker)

OS Kali Linux 2026.x

IP 192.168.14.128

Máy mục tiêu (Target)

OS Metasploitable 2 (Linux)

IP 192.168.14.129

Đăng nhập mặc định msfadmin  msfadmin

2. Quy Trình Triển Khai

Bước 1 Cấu hình dải mạng Host-only trên VMware

Trên thanh menu VMware Workstation, chọn Edit  Virtual Network Editor....

Chọn card mạng VMnet1 (Host-only) và cấu hình

Tích chọn Connect a host virtual adapter to this network.

Tích chọn Use local DHCP service to distribute IP address to VMs.

Subnet IP 192.168.14.0

Subnet mask 255.255.255.0

Nhấn Apply  OK.

Bước 2 Gán card mạng cho máy ảo Kali Linux

Chuột phải vào máy ảo kali-linux  chọn Settings....

Tại mục Hardware  chọn Network Adapter.

Chọn tùy chọn Host-only A private network shared with the host (hoặc Custom VMnet1).

Đảm bảo đã tích chọn Connect at power on  nhấn OK.

Bước 3 Khởi động Kali Linux và xác định địa chỉ IP

Bật nguồn máy ảo Kali Linux và mở Terminal.

Kiểm tra thông tin card mạng

ip -br addr


Kết quả xác nhận Card eth0 nhận địa chỉ IP 192.168.14.12824.

Bước 4 Tải và thiết lập máy ảo Metasploitable 2

Tải gói máy ảo nén Metasploitable 2 từ SourceForge.

Giải nén vào thư mục làm việc (ví dụ Dmetasploitable-linux-2.0.0Metasploitable2-Linux).

Trên VMware, chọn File  Open... và dẫn tới file Metasploitable.vmx.

Đảm bảo cấu hình card mạng của máy ảo này cũng ở chế độ Host-only (VMnet1).

Bước 5 Bật Metasploitable 2 và kiểm tra IP

Bật máy ảo Metasploitable 2 và đăng nhập bằng tài khoản msfadmin  msfadmin.

Kiểm tra IP của máy mục tiêu

ifconfig


Kết quả xác nhận Card eth0 nhận IP 192.168.14.129.

Bước 6 Kiểm tra kết nối mạng (Ping Test)

Từ Terminal trên máy Kali Linux, thực hiện ping kiểm tra tính thông suốt của đường truyền

ping -c 4 192.168.14.129


Kết quả Nhận đủ 4 gói tin phản hồi (0% packet loss), hai máy đã kết nối thành công.

3. Thực Hiện Quét Bằng Nmap

Bước 7 Quét TCP SYN Scan (Stealth Scan)

Thực hiện quét nhanh các cổng TCP đang mở bằng cơ chế SYN handshake nửa chừng

nmap -sS 192.168.14.129


Kết quả Phát hiện nhiều cổng mở phổ biến như

21tcp (ftp), 22tcp (ssh), 23tcp (telnet), 25tcp (smtp)

53tcp (domain), 80tcp (http), 139tcp, 445tcp (smb)

3306tcp (mysql), 5432tcp (postgresql), 5900tcp (vnc),...

Bước 8 Nhận diện phiên bản dịch vụ (Service Version Detection)

Thực hiện thu thập thông tin banner và phiên bản chính xác của từng tiến trình đang chạy

nmap -sV 192.168.14.129


Thông tin quan trọng thu được

FTP vsftpd 2.3.4

SSH OpenSSH 4.7p1 Debian 8ubuntu1

Web Apache httpd 2.2.8 ((Ubuntu) DAV2)

SMB Samba smbd 3.X - 4.X

Database MySQL 5.0.51a, PostgreSQL DB 8.3.0 - 8.3.7

Bước 9 Quét toàn diện và xuất kết quả ra file

Chạy quét dịch vụ kết hợp xuất dữ liệu đồng thời ra 3 định dạng file báo cáo

nmap -sV -oA lab4_nmap 192.168.14.129


Kiểm tra danh sách các file kết quả vừa được tạo

ls -lh lab4_nmap


Các định dạng thu được

lab4_nmap.nmap File text định dạng tiêu chuẩn đọc trực tiếp.

lab4_nmap.xml File XML dùng import vào các công cụ phân tích hoặc Metasploit Framework.

lab4_nmap.gnmap Định dạng Grepable để lọc nhanh kết quả bằng lệnh bashgrep.