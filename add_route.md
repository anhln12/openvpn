OpenVPN Access Server (OpenVPN AS) quản lý file cấu hình, routing và push route qua Admin Web UI thay vì chỉnh sửa file server.conf thủ công.

Thực hiện các bước sau để push route trong OpenVPN AS:

Bước 1: Đăng nhập Admin Portal

Bước 2: Thêm Route vào Client Setting
- Chọn VPN Settings
- Tìm mục Routing
- Tại phần Should client traffic be routed through the VPN? Chọn No (Nếu chỉ muốn route một số subnet cụ thể) hoặc Yes (Nếu muốn ép toàn bộ default route)
- Tại mục Specify the private subnets to which all clients should be given access, nhập các subnet bạn muốn push xuống client (định dạng CIDR, ví dụ: 192.168.10.0/24), mỗi subnet một dòng không cần cách nhau dấu ,

Bước 3: Save và Update
- Nhận Save Setting ở cuối trang
- Nhận Update Running Server để áp dụng ngay lập tức mà không cần khởi động lại dịch vụ.
- Cách xác định thành công: Kết nối lại OpenVPN Connect client và kiểm tra bảng routing trên client thấy xuất hiện subnet đã cấu hình



Phân biệt Using NAT và Using Routing trong Routing:

<img width="1536" height="75" alt="image" src="https://github.com/user-attachments/assets/7f0a8139-6e81-4444-8986-fe1291c10f3f" />

NAT (Network Address Translation) và Routing khác nhau cốt lõi ở cách gói tin được chuyển đổi và định danh IP:
- Routing (Định tuyến thuần túy):
  + Cơ chế: Gói tin đi từ client vào mạng nội bộ giữ nguyên địa chỉ IP nguồn (IP của client trong dải VPN)
  + Ưu điểm: Các server đích thấy rõ IP gốc của client
  + Nhược điểm/Yêu cầu: Mạng nội bộ (Phía sau VPN Server) bắt buộc phải có routing table (static route) trỏ ngược lại dải IP của VPN Client qua IP của VPN server. Nếu không có route ngược, các server  nội bộ sẽ không biết đường trả lời gói tin.

 - NAT (Chuyển đổi địa chỉ mạng):
  + Cơ chế: Khi gói tin từ client đi qua VPN server vào mạng nội bộ, VPN server sẽ đổi IP nguồn của client thành IP nội bộ của chính VPN server (SNAT/Masquerade)
  + Ưu điểm: Phía mạng nội bộ không cần cấu hình thêm bất kỳ route nào trỏ ngược về VPN client, vì chúng chỉ thấy traffic xuất phát từ VPN Server.
  + Nhược điểm: Các server trong mạng nội bộ mất đi IP gốc của client (tất cả đều hiển thị đến từ IP của VPN server), không phân log hoặc kiểm soát truy cập chi tiết theo từng user.

Case thêm route với 1 IP Cụ thể ví dụ: 14.xxx.80.xxx/32

Do OpenVPN AS có cơ chế lọc: nó chỉ push các private subnet hợp lệ (RFC 1918/ RFC 4193 như các dải 10.x, 172.16.x, 192.168.x). Các IP dạng Public IP cụ thể sẽ bị hệ thống bỏ qua và không push route xuống client

Để ép route IP Public này qua VPN, ta thực hiện các cách sau:

Cách 1: Thêm route thủ công trực tiếp trên máy client của bạn
- Linux/MacOS
  ```
  sudo ip route add 14.xxx.80.xxx/32 dev tun0
  ```
- Windows
  ```
  route add 14.xxx.80.xxx mask 255.255.255.255 <IP_Gateway_VPN>
  ```

Cách 2: Qua cách cấu hình nâng cao

Đề OpenVPN Access Server ép push một Public IP xuống client thay vì bị giới hạn chỉ nhận Private Subnet, chúng ta có thể cấu hình thông qua tính năng cấu hình năng cao (server_config_override hoặc vpn.server.conf_ext) bằng script/cli của access server.

```
# Kiểm tra config hiện tại
sacli ConfigQuery
# Thêm route vào cấu hình push chung của server
sacli --key "vpn.server.conf_ext.1" --val "push \"route 14.xxx.80.xxx 255.255.255.255\"" configPut

(Nếu đã có .1, bạn có thể tăng số thứ tự lên .2, .3... cho các IP tiếp theo).

# Áp dụng
sacli start
```

