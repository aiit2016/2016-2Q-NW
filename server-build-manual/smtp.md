
## SMTP構築
-----------------------------------------
### Postfixのインストール
```
yum install postfix
rpm -qa | grep postfix
```

### 設定
/etc/postfix/main.cf
```
myhostname = mail.xx-domain
mydomain = xx-domain
myorigin = $myhostname
#inet_interfaces = localhost
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 192.168.0.0/24, 127.0.0.1
home_mailbox = Maildir/

smtpd_recipient_restrictions = permit_mynetworks, reject_unauth_destination
```

### メールの保存
```
mkdir -p /etc/skel/Maildir/{new,cur,tmp}
chmod -R 700 /etc/skel/Maildir/
```

### postfix起動
```
systemctl enable postfix
systemctl start postfix
```

### ポートを開放
firewall-cmd --add-service=smtp --permanent
firewall-cmd --reload
