
## IPTABLES停止
```
systemctl stop iptables
systemctl disable iptables
systemctl is-enabled iptables
```

## firewalldインストール
```
yum -y install firewalld
systemctl start firewalld
systemctl enable firewalld
systemctl is-enabled firewalld
firewall-cmd --list-all ←設定を確認
```

## ポート開放
```
firewall-cmd --add-service=ftp --permanent ←ftpを開放
firewall-cmd --add-port=4000-4029/tcp --permanent ←ポート4000から4029番を開放
firewall-cmd --add-service=http --zone=public --permanent ←httpを開放
firewall-cmd --add-service=https --permanent ←httpsを開放
firewall-cmd --zone=public --add-service=smtp --permanent ←smtpを開放
firewall-cmd --zone=public --add-port=110/tcp --permanent ←ポート110番を開放
firewall-cmd --zone=public --add-port=143/tcp --permanent ←ポート143番を開放
firewall-cmd --zone=public --add-port=465/tcp --permanent ←ポート465番を開放
firewall-cmd --zone=public --add-service=pop3s --permanent ←pop3sを開放
firewall-cmd --zone=public --add-service=imaps --permanent ←imapsを開放
firewall-cmd --reload ←リロード
```

## firewallコマンドの基本的な使い方
```
firewall-cmd --zone=public --add-service=ssh --permanent ←sshを追加
firewall-cmd --zone=public --add-port=22/tcp --permanent ←ポート22番を追加
firewall-cmd --zone=public --remove-service=ssh --permanent ←sshを削除
firewall-cmd --zone=public --remove-port=22/tcp --permanent ←ポート22番を削除
firewall-cmd --reload ←リロード
firewall-cmd --list-all ←設定を確認
```
