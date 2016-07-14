
## DNS構築
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
