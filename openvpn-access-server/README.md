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
Sửa tiếp file uprop.pyc trong thư mục pyovpn/lic
```
cd pyovpn/lic/
```
File pyc là file python đã compiled, tiến hành dịch ngược ra file source code python để chỉnh sửa, cài đặt uncompyle6
```
pip3.8 install uncompyle6
```
Dịch ngược
```
uncompyle6 uprop.pyc > uprop.py
```
Mở file uprop.py, tìm funtion figure
Nội dung function như sau:
```
    def figure(self, licdict):
        proplist = set(('concurrent_connections', ))
        good = set()
        ret = None
        if licdict:
            for key, props in list(licdict.items()):
                if 'quota_properties' not in props:
                    print('License Manager: key %s is missing usage properties' % key)
                else:
                    proplist.update(props['quota_properties'].split(','))
                    good.add(key)
 
        for prop in proplist:
            v_agg = 0
            v_nonagg = 0
            if licdict:
                for key, props in list(licdict.items()):
                    if key in good:
                        if prop in props:
                            try:
                                nonagg = int(props[prop])
                            except:
                                raise Passthru('license property %s (%s)' % (prop, props.get(prop).__repr__()))
 
                            v_nonagg = max(v_nonagg, nonagg)
                            prop_agg = '%s_aggregated' % prop
                            agg = 0
                            if prop_agg in props:
                                try:
                                    agg = int(props[prop_agg])
                                except:
                                    raise Passthru('aggregated license property %s (%s)' % (
                                     prop_agg, props.get(prop_agg).__repr__()))
 
                                v_agg += agg
                        if DEBUG:
                            print('PROP=%s KEY=%s agg=%d(%d) nonagg=%d(%d)' % (
                             prop, key, agg, v_agg, nonagg, v_nonagg))
 
            apc = self._apc()
            v_agg += apc
            if ret == None:
                ret = {}
            ret[prop] = max(v_agg + v_nonagg, bool('v_agg') + bool('v_nonagg'))
            ret['apc'] = bool(apc)
            if DEBUG:
                print("ret['%s'] = v_agg(%d) + v_nonagg(%d)" % (prop, v_agg, v_nonagg))
 
        return ret
```
Để ý dòng cuối dùng return net

Thêm 1 dòng ngay trước nó: ret['concurrent_connections'] = 2048

Tiến hành đóng gói lại thành file pyc như cũ
```
rm -f uprop.pyc
python3.8 -O -m compileall uprop.py && mv __pycache__/uprop.cpython-38.opt-1.pyc uprop.pyc
rm -rf __pycache__ uprop.py
```
Nén lại thành file egg như cũ
```
cd /usr/local/openvpn_as/lib/python/unlock_license
zip -r pyovpn-2.0-py3.8_cracked.zip common EGG-INFO pyovpn
mv pyovpn-2.0-py3.8_cracked.zip pyovpn-2.0-py3.8.egg
mv pyovpn-2.0-py3.8.egg ../ && cd ..
```

Xóa python cache trong /usr/local/openvpn_as/lib/python
```
cd /usr/local/openvpn_as/lib/python
rm -rf __pycache__/*
```
Khởi tạo lại profile openvpn
```
cd /usr/local/openvpn_as/bin
./ovpn-init
```
```
[root@openvpn bin]# ./ovpn-init
Detected an existing OpenVPN-AS configuration.
Continuing will delete this configuration and restart from scratch.
Please enter 'DELETE' to delete existing configuration: DELETE
 
          OpenVPN Access Server
          Initial Configuration Tool
------------------------------------------------------
OpenVPN Access Server End User License Agreement (OpenVPN-AS EULA)
...
Please enter 'yes' to indicate your agreement [no]: yes
 
Once you provide a few initial configuration settings,
OpenVPN Access Server can be configured by accessing
its Admin Web UI using your Web browser.
 
Will this be the primary Access Server node?
(enter 'no' to configure as a backup or standby node)
> Press ENTER for default [yes]:
 
Please specify the network interface and IP address to be
used by the Admin Web UI:
(1) all interfaces: 0.0.0.0
(2) ens33: 192.168.150.132
Please enter the option number from the list above (1-2).
> Press Enter for default [1]:
 
What public/private type/algorithms do you want to use for the OpenVPN CA?
 
Recommended choices:
 
rsa       - maximum compatibility
secp384r1 - elliptic curve, higher security than rsa, allows faster connection setup and smaller user profile files
showall   - shows all options including non-recommended algorithms.
> Press ENTER for default [rsa]:
 
What key size do you want to use for the certificates?
Key size should be between 2048 and 4096
 
> Press ENTER for default [ 2048 ]:
 
What public/private type/algorithms do you want to use for the self-signed web certificate?
 
Recommended choices:
 
rsa       - maximum compatibility
secp384r1 - elliptic curve, higher security than rsa, allows faster connection setup and smaller user profile files
showall   - shows all options including non-recommended algorithms.
> Press ENTER for default [rsa]:
 
What key size do you want to use for the certificates?
Key size should be between 2048 and 4096
 
> Press ENTER for default [ 2048 ]:
 
Please specify the port number for the Admin Web UI.
> Press ENTER for default [943]:
 
Please specify the TCP port number for the OpenVPN Daemon
> Press ENTER for default [443]:
 
Should client traffic be routed by default through the VPN?
> Press ENTER for default [yes]:
 
Should client DNS traffic be routed by default through the VPN?
> Press ENTER for default [yes]:
 
Use local authentication via internal DB?
> Press ENTER for default [yes]:
 
Private subnets detected: ['192.168.150.0/24']
 
Should private subnets be accessible to clients by default?
> Press ENTER for default [yes]:
 
To initially login to the Admin Web UI, you must use a
username and password that successfully authenticates you
with the host UNIX system (you can later modify the settings
so that RADIUS or LDAP is used for authentication instead).
 
You can login to the Admin Web UI as "openvpn" or specify
a different user account to use for this purpose.
 
Do you wish to login to the Admin UI as "openvpn"?
> Press ENTER for default [yes]:
 
> Please specify your Activation key (or leave blank to specify later):
 
 
 
Initializing OpenVPN...
Removing Cluster Admin user login...
userdel "admin_c"
Adding new user login...
useradd -s /sbin/nologin "openvpn"
Writing as configuration file...
Perform sa init...
Wiping any previous userdb...
Creating default profile...
Modifying default profile...
Adding new user to userdb...
Modifying new user as superuser in userdb...
Getting hostname...
Hostname: openvpn
Preparing web certificates...
Getting web user account...
Adding web group account...
Adding web group...
Adjusting license directory ownership...
Initializing confdb...
Generating PAM config...
Enabling service
Starting openvpnas...
 
NOTE: Your system clock must be correct for OpenVPN Access Server
to perform correctly.  Please ensure that your time and date
are correct on this system.
 
Initial Configuration Complete!
 
You can now continue configuring OpenVPN Access Server by
directing your Web browser to this URL:
 
https://192.168.150.132:943/admin
Login as "openvpn" with the same password used to authenticate
to this UNIX host.
 
During normal operation, OpenVPN AS can be accessed via these URLs:
Admin  UI: https://192.168.150.132:943/admin
Client UI: https://192.168.150.132:943/
 
See the Release Notes for this release at:
   https://openvpn.net/vpn-server-resources/release-notes/
```
Đăng nhập lại openvpn sẽ thấy license đã lên 2048 user
<img width="748" alt="image" src="https://github.com/anhln12/openvpn/assets/18412583/bdc06291-5bd7-47e8-8448-f308b0de85cb">

