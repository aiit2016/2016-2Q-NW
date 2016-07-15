
## DNS構築（外向け）
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
```
dig www.xxx.x
dig -x nnn.nnn.nnn.nnn
```


-----------------------------------------
## DNS構築（内向け）
-----------------------------------------
### インストール
```
yum -y install unbound
```

### 設定
別途参照

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

### unbound起動
```
systemctl start unbound
systemctl is-active unbound
systemctl enable unbound ← 自動起動設定
```

### 名前解決確認
```
dig www.xxx.x
dig -x nnn.nnn.nnn.nnn
```
