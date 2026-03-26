Việc cài đặt OpenVPN Access Server (AS) trên Ubuntu khá đơn giản vì OpenVPN cung cấp một repository chính thức. Khác với bản community bản Access Server có giao diện Web UI giúp bạn quản lý user và cấu hình dễ dàng hơn.

Các bước cài đặt trên Ubuntu 22.04/24.04:

Bước 1: Cập nhật hệ thống và cài các gói cần thiết
```
sudo apt update && sudo apt install -y ca-certificates wget net-tools gnupg
```

Bước 2: Thêm Repository của OpenVPN
```
sudo mkdir -p /etc/apt/keyrings
wget -qO - https://as-repository.openvpn.net/as-repo-public.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/openvpn-as-repo-public.gpg
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/openvpn-as-repo-public.gpg] https://as-repository.openvpn.net/as/debian $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/openvpn-as-repo.list
```

Bước 3: Cài đặt OpenVPN Server
```
sudo apt update
sudo apt install openvpn-as -y
```

Lưu ý: Logs cài đặt sẽ hiển thị thông tin account openvpn cho login giao diện quản trị

```
+++++++++++++++++++++++++++++++++++++++++++++++
Access Server 3.1.0 has been successfully installed in /usr/local/openvpn_as
Configuration log file has been written to /usr/local/openvpn_as/init.log

Access Server Web UIs are available here:
Admin  UI: https://203.162.xxx.xxx:943/admin
Client UI: https://203.162.xxx.xxx:943/
To login please use the "openvpn" account with "xxxxxx" password.
(password can be changed on Admin UI)
+++++++++++++++++++++++++++++++++++++++++++++++
```

Bước 4:  Truy cập Giao diện quản trị 
```
Admin UI: https://<IP_CUA_BAN>:943/admin (Để cấu hình server)
Client UI: https://<IP_CUA_BAN>:943/ (Để user tải profile .ovpn)
```

Các lưu ý quan trọng:
* Mở port Firewall
```
ufw allow 943/tcp    # Web GUI
ufw allow 443/tcp    # HTTPS / TCP VPN
ufw allow 1194/udp   # UDP VPN (Mặc định)
```
* IP Forwarding:
  - Mặc định, Linux sẽ chặn việc chuyển tiếp gói tin giữa các interface (Ví dụ từ interface ảo tun0 của VPN sang interface vật lý eth0 của Internet/LAN. Nếu không bật, Client có thể kết nối thành công sang tới VPN nhuwg không thể truy cập Ineternet hoặc không ping được các máy khác trong nội bộ
  - OpenVPN AS thường tự động bật NAT/Routing. Tuy nhiên, bạn nên kiểm tra file /etc/sysctl.conf xem dòng net.ipv4.ip_forward = 1 đã được bật chưa
  ```
  1. Tạo file cấu hình sysctl riêng cho OpenVPN
  cat >/etc/sysctl.d/openvpn.conf <<EOL
  net.ipv4.ip_forward = 1
  EOL

  2. Áp dụng thay đổi ngay lập tức mà không cần reboot
sysctl -p /etc/sysctl.d/openvpn.conf

3. Kiểm tra lại đã nhận cấu hình chưa
root@monitor:/etc/netplan# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
  ```
 
