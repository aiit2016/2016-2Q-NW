
## DNS構築（外部）
-----------------------------------------
### インストール
```
yum -y install bind bind-chroot
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
```

### 設定
別途参照

### ファイアウォールのDNSポート解放
```
firewall-cmd --list-all
cat /usr/lib/firewalld/services/dns.xml
firewall-cmd --add-service=dns --permanent
firewall-cmd --reload
```

### BIND起動
```
systemctl start named-chroot
systemctl enable named-chroot ← 自動起動設定
```

### 名前解決確認
dig www.xxx.x
dig -x nnn.nnn.nnn.nnn


-----------------------------------------
## DNS構築（内部）
-----------------------------------------
### インストール
```
yum -y install unbound
```

### 設定
/etc/unbound/local.d/local.conf
```
#
# Interfaces Settings
#
interface: 127.0.0.1
#interface: 172.16.10.251
#interface: 172.16.10.252
interface: 172.16.cc5.dd5
 
#
# local Network Settings
#
access-control: 127.0.0.1 allow
access-control: 172.16.0.0/20 allow

#
# DNS name Change
#
# ローカルのサーバの名前とIPアドレスの対応を登録
local-data: "net03.xxx.local. IN A 172.16.0.dd6"
local-data-ptr: "172.16.0.dd6 net03.xxx.local."

# Enable IPv6, "yes" or "no".
do-ip6: no
```

### 鍵ファイルの作成
```
systemctl start unbound-keygen.service
```

### 設定ファイルの確認
```
unbound-checkconf
```

### ファイアウォールのDNSポート解放
```
firewall-cmd --list-all
cat /usr/lib/firewalld/services/dns.xml
firewall-cmd --add-service=dns --permanent
firewall-cmd --reload
```

### BIND起動
```
systemctl start unbound
systemctl is-active unbound
systemctl enable unbound ← 自動起動設定
```

### 名前解決確認
dig www.xxx.x
dig -x nnn.nnn.nnn.nnn
