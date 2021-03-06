
## DNS 設定（外向け）
------------------------------------------------------

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
#### named.conf設定
```
options {
        #listen-on port 53 { 127.0.0.1; };          # <= コメントアウト
        #listen-on-v6 port 53 { ::1; };             # <= コメントアウト
        version         "unknown";                  # <= 追加（bind バージョン情報非表示）
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        #allow-query     { localhost; localnets; };  # <= 変更(ローカルホスト、ローカルネットからの問合せのみ許可)
        #recursion yes;
        empty-zones-enable no;   # <= 追加（起動時の空ゾーンエラーログの出力無効化）

        dnssec-enable no;        # <= 変更（DNSSEC 非対応なので）
        dnssec-validation no;    # <= 変更（        〃         ）
        #dnssec-lookaside auto;  # <= コメントアウト（        〃         ）

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.iscdlv.key";
        forwarders{            # <= 追加
                aaa.bbb.ccc.ddd;  # <= 追加（ルータの IP アドレス）
                aa2.bb2.cc2.dd2;  # <= 追加（ISPの IP アドレス）
        };                     # <= 追加

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
#        category lame-servers { null; };  # <= 追加（エラーログの出力制御）
};

view "internal" {						
        match-clients {localnets; 192.168.0.0/24; aa4.bb4.cc4.0/20;};						
        recursion yes;						
        zone "." IN {
                type hint;
                file "named.ca";
        };
        zone "xxx-domain" {						
                type master;						
                file "/var/named/xxx-domain-in.db";						
                allow-update {none;};						
        };
        zone "0.168.192.in-addr.arpa" {						
                type master;						
                file "/var/named/0.168.192.in-addr.arpa.db";						
                allow-update {none;};						
        };						
        include "/etc/named.rfc1912.zones";						
	include "/etc/named.root.key";
};

view "external" {                                   # <= 追加（固定 IP 環境なので）
        match-clients { any; };                     # <= 追加（固定 IP 環境なので）
        match-destinations { any; };                # <= 追加（固定 IP 環境なので）
        recursion no;                               # <= 追加（固定 IP 環境なので）
        zone "xxx-domain" {
                type master;
                file "xxx-domain-ex.db";
                allow-query { any; };
        };
        zone "ccc.bbb.aaa.in-addr.arpa" {
                type master;
                file "ccc.bbb.aaa.in-addr.arpa.db";
                allow-query { any; };
        };
};

include "/etc/rndc.key";								
```

### ルートゾーン最新化
```
dig . ns @198.41.0.4 +bufsize=1024 > /var/named/chroot/var/named/named.ca
```

### 正引きゾーンデータベース
* /var/named/chroot/var/named/xxx-domain-in.db設定（権限、GID要注意）
```
$TTL    86400
@       IN      SOA     ns1.xxx-domain.  root.ns1.xxx-domain.(
                                      2016071501 ; Serial
                                      7200       ; Refresh
                                      7200       ; Retry
                                      2419200    ; Expire
                                      86400 )    ; Minimum
        IN NS    ns1.xxx-domain.
        IN MX 10 ns1.xxx-domain.
ns1     IN A     aa2.bb2.cc2.dd2
www     IN CNAME     ns1
mail    IN CNAME     ns1
```

* /var/named/chroot/var/named/xxx-domain-ex.db設定（権限、GID要注意）
```
$TTL    86400
@       IN      SOA     ns1.xxx-domain.  root.xxx-domain.(
                                      2016071501 ; Serial
                                      7200       ; Refresh
                                      7200       ; Retry
                                      2419200    ; Expire
                                      86400 )    ; Minimum
        IN NS    ns1.xxx-domain.
        IN MX 10 ns1.xxx-domain.
ns1     IN A     aaa.bbb.ccc.ddd
www     IN CNAME     ns1
mail    IN CNAME     ns1
#xxx-domain. IN TXT "v=spf1 ip4:aaa.bbb.ccc.ddd ~all"
```

### 逆引きゾーンデータベース
* /var/named/chroot/var/named/0.168.192.in-addr.arpa.db
```
$TTL    86400
@       IN      SOA     ns1.xxx-domain. root.ns1.xxx-domain.(
                        2016071501      ; serial
                        3600            ; refresh (1 hour)
                        900             ; retry (15 minutes)
                        604800          ; expire (1 week)
                        86400           ; negative (1 day)
)
        IN      NS              ns1.xxx-domain.
        IN      A               255.255.255.0
dd2     IN      PTR             ns1.xxx-domain.
```

* /var/named/chroot/var/named/ccc.bbb.aaa.in-addr.arpa.db
```
$TTL    86400
@       IN      SOA     ns1.xxx-domain. root.ns1.xxx-domain.(
                        2016071501      ; serial
                        3600            ; refresh (1 hour)
                        900             ; retry (15 minutes)
                        604800          ; expire (1 week)
                        86400           ; negative (1 day)
)
        IN      NS              ns1.xxx-domain.
        IN      A               255.255.255.240
ddd     IN      PTR             ns1.xxx-domain.
```


## DNS 設定（内向け）
------------------------------------------------------------------------
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
local-zone: "net03.xxx.local." static
local-data: "indns.net03.xxx.local. IN A 172.16.0.dd6"
local-data-ptr: "172.16.0.dd6 indns.net03.xxx.local."

local-zone: "0.168.192.in-addr.arpa." transparent

stub-zone:
    name: "xx-domain."
    stub-addr: 192.168.0.dd3
    #stub-host: xx-domain.
stub-zone:
    name: "0.168.192.in-addr.arpa."
    stub-addr: 192.168.0.4

# Enable IPv6, "yes" or "no".
do-ip6: no

# hide-version, "yes" or "no".
hide-version: yes
```
