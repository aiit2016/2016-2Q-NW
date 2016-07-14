
## DNS 設定（ゾーンサーバ）
------------------

### mount ファイルリスト
| mount元	| mount先 |
| ------- | ------- |
| /etc/named | /var/named/chroot/etc/named |
| /etc/named.root.key | /var/named/chroot/etc/named.root.key |
| /etc/named.conf | /var/named/chroot/etc/named.conf |
| /etc/named.rfc1912.zone | /var/named/chroot/etc/named.rfc1912.zones |
| /etc/rndc.key | /var/named/chroot/etc/rndc.key |
| /etc/named.iscdlv.key | /var/named/chroot/etc/named.iscdlv.key |
| /run/named | /var/named/chroot/run/named |
| /var/named | /var/named/chroot/var/named |
| /usr/lib64/bind | /var/named/chroot/usr/lib64/bind |

---------------------------------------------------
### named.conf
#### named.conf概要
```
acl "任意のacl名称" {
        省略
};

options {
        省略
};

logging {
        省略
};

view "任意のview名称" {
        省略
};
```

---------------------------------------------------
### セキュリティー対策
* ゾーンサーバでキャッシュサービスを提供しないように
```
named.conf
options {
    （省略）
        recursion no;
    （省略）
};
```

* バージョンを非表示にする
```
options {
    （省略）
        version "noversion";
    （省略）
};
```


