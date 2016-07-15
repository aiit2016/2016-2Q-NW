
## DHCPのインストールと設定
-----------------------------------------------------

### インストール
```
yum -y install dhcp
```

### 設定
* /etc/dhcp/dhcpd.conf
```
# 以下の内容で新規作成
# ドメイン名指定
option domain-name     "srv.world";
# ネームサーバーのホスト名, またはIPアドレス指定
option domain-name-servers     dlp.srv.world;
# デフォルト貸出期間
default-lease-time 600;
# 最大貸出期間
max-lease-time 7200;
# 正当な DHCP サーバーであることの宣言
authoritative;
# ネットワークアドレスとサブネットマスク指定
subnet 10.0.0.0 netmask 255.255.255.0 {
    # 貸し出すIPアドレスの範囲指定
    range dynamic-bootp 10.0.0.200 10.0.0.254;
    # ブロードキャストアドレス指定
    option broadcast-address 10.0.0.255;
    # ゲートウェイアドレス指定
    option routers 10.0.0.1;
}
```

### 起動
```
systemctl start dhcpd 
systemctl enable dhcpd 
```

### Firewalld を有効にしている場合は DHCP サービスの許可が必要です。なお、DHCPサーバー は 67/UDP を使用します。
```
firewall-cmd --add-service=dhcp --permanent 
[root@dlp ~]# firewall-cmd --reload 
```
