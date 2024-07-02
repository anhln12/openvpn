Signup to Openvpn and download OpenVPN access server
https://myaccount.openvpn.com/signup/as



# Hướng dẫn unlock OpenVPN Access server

Tắt hết openvpn service
```
systemctl stop openvpnas
ps -ef | grep openvpn
root       2451   1779  0 11:31 pts/0    00:00:00 grep --color=auto openvpn
```

Thư mục /usr/local/openvpn_as/lib/python chứa toàn bộ thư viện openvpn-as sử dụng bao gồm cả phần license. Phần license do thư viện pyovpn-2.0-py3.8.egg xử lý.

Chúng ta tiến hành edit lại file này để unlock, backup file gốc, tạo thư mục mới để làm việc.
```
cd /usr/local/openvpn_as/lib/python
mkdir unlock_license
mv pyovpn-2.0-py3.8.egg pyovpn-2.0-py3.8.egg_bak
cp -rp pyovpn-2.0-py3.8.egg_bak unlock_license/pyovpn-2.0-py3.8.zip
cd unlock_license
```
Bản chất egg là file zip, nên chỉ rename, rồi giải nén là có thể edit được nội dung
```
yum install -y zip unzip
unzip pyovpn-2.0-py3.8.zip
```


