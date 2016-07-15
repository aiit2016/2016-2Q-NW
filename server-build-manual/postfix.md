
## Postfix設定
--------------------------------------------------------------------------------

### /etc/postfix/main.cf　←　Postfix設定ファイル編集
```
# INTERNET HOST AND DOMAIN NAMES
#
# The myhostname parameter specifies the internet hostname of this
# mail system. The default is to use the fully-qualified domain name
# from gethostname(). $myhostname is used as a default value for many
# other configuration parameters.
#
#myhostname = host.domain.tld
#myhostname = virtual.domain.tld
myhostname = mail.centossrv.com　←　追加(自FQDN名を指定)

# The mydomain parameter specifies the local internet domain name.
# The default is to use $myhostname minus the first component.
# $mydomain is used as a default value for many other configuration
# parameters.
#
#mydomain = domain.tld
mydomain = centossrv.com　←　追加(自ドメイン名を指定)

# SENDING MAIL
#
# The myorigin parameter specifies the domain that locally-posted
# mail appears to come from. The default is to append $myhostname,
# which is fine for small sites.  If you run a domain with multiple
# machines, you should (1) change this to $mydomain and (2) set up
# a domain-wide alias database that aliases each user to
# user@that.users.mailhost.
#
# For the sake of consistency between sender and recipient addresses,
# myorigin also specifies the default domain name that is appended
# to recipient addresses that have no @domain part.
#
#myorigin = $myhostname
#myorigin = $mydomain
myorigin = $mydomain　←　追加(ローカルからのメール送信時の送信元メールアドレス@以降にドメイン名を付加)

# The inet_interfaces parameter specifies the network interface
# addresses that this mail system receives mail on.  By default,
# the software claims all active interfaces on the machine. The
# parameter also controls delivery of mail to user@[ip.address].
#
# See also the proxy_interfaces parameter, for network addresses that
# are forwarded to us via a proxy or network address translator.
#
# Note: you need to stop/start Postfix when this parameter changes.
#
#inet_interfaces = all
#inet_interfaces = $myhostname
#inet_interfaces = $myhostname, localhost
inet_interfaces = localhost
↓
inet_interfaces = all　←　変更(外部からのメール受信を許可)

# The mydestination parameter specifies the list of domains that this# machine considers itself the final destination for.
#
# These domains are routed to the delivery agent specified with the
# local_transport parameter setting. By default, that is the UNIX
# compatible delivery agent that lookups all recipients in /etc/passwd
# and /etc/aliases or their equivalent.
#
# The default is $myhostname + localhost.$mydomain.  On a mail domain
# gateway, you should also include $mydomain.
#
# Do not specify the names of virtual domains - those domains are
# specified elsewhere (see VIRTUAL_README).
#
# Do not specify the names of domains that this machine is backup MX
# host for. Specify those names via the relay_domains settings for
# the SMTP server, or use permit_mx_backup if you are lazy (see
# STANDARD_CONFIGURATION_README).
#
# The local machine is always the final destination for mail addressed
# to user@[the.net.work.address] of an interface that the mail system
# receives mail on (see the inet_interfaces parameter).
#
# Specify a list of host or domain names, /file/name or type:table
# patterns, separated by commas and/or whitespace. A /file/name
# pattern is replaced by its contents; a type:table is matched when
# a name matches a lookup key (the right-hand side is ignored).
# Continue long lines by starting the next line with whitespace.
#
# See also below, section "REJECTING MAIL FOR UNKNOWN LOCAL USERS".
#
mydestination = $myhostname, localhost.$mydomain, localhost
↓
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain　←　変更(自ドメイン宛メールを受信できるようにする)
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,
#       mail.$mydomain, www.$mydomain, ftp.$mydomain

# The relayhost parameter specifies the default host to send mail to
# when no entry is matched in the optional transport(5) table. When
# no relayhost is given, mail is routed directly to the destination.
#
# On an intranet, specify the organizational domain name. If your
# internal DNS uses no MX records, specify the name of the intranet
# gateway host instead.
#
# In the case of SMTP, specify a domain, host, host:port, [host]:port,
# [address] or [address]:port; the form [host] turns off MX lookups.
#
# If you're connected via UUCP, see also the default_transport parameter.
#
#relayhost = $mydomain
#relayhost = [gateway.my.domain]
#relayhost = [mailserver.isp.tld]
#relayhost = uucphost
#relayhost = [an.ip.add.ress]
relayhost = [mail.asahi-net.or.jp]　←　ISPのSMTPサーバー名を指定（左記例はASAHIネットの場合）

# DELIVERY TO MAILBOX
#
# The home_mailbox parameter specifies the optional pathname of a
# mailbox file relative to a user's home directory. The default
# mailbox file is /var/spool/mail/user or /var/mail/user.  Specify
# "Maildir/" for qmail-style delivery (the / is required).
#
#home_mailbox = Mailbox
#home_mailbox = Maildir/
home_mailbox = Maildir/　←　追加(メールボックス形式をMaildir形式にする)

# SHOW SOFTWARE VERSION OR NOT
#
# The smtpd_banner parameter specifies the text that follows the 220
# code in the SMTP server's greeting banner. Some people like to see
# the mail version advertised. By default, Postfix shows no version.
#
# You MUST specify $myhostname at the start of the text. That is an
# RFC requirement. Postfix itself does not care.
#
#smtpd_banner = $myhostname ESMTP $mail_name
#smtpd_banner = $myhostname ESMTP $mail_name ($mail_version)
smtpd_banner = $myhostname ESMTP unknown　←　追加(メールサーバーソフト名の隠蔽化)

以下を最終行へ追加(SMTP-Auth設定)
smtpd_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_mechanism_filter = plain
smtpd_sasl_local_domain = $myhostname
smtpd_recipient_restrictions =
    permit_mynetworks
    permit_sasl_authenticated
    reject_unauth_destination

以下を最終行へ追加(受信メールサイズ制限)
message_size_limit = 10485760　←　追加(受信メールサイズを10MB=10*1024*1024に制限)
```

### SMTP-Auth設定
* サービス起動
```
SMTP-Auth用ユーザ名、パスワードにシステムのユーザ名、パスワードを使用する場合)
yum -y install cyrus-sasl　←　cyrus-saslインストール
systemctl start saslauthd　←　saslauthd起動※CentOS7の場合
systemctl enable saslauthd　←　saslauthd自動起動設定※CentOS7の場合
chkconfig saslauthd on　←　saslauthd自動起動設定※CentOS6,5の場合
```

* 認証ファイル
/etc/postfix/sasl_passwd　←　ISPのSMTP認証ファイル作成
```
[mail.asahi-net.or.jp] xxxxxxxx:xxxxxxxx　←　「[ISPのSMTPサーバー名] ユーザー名:パスワード」を指定（左記例はASAHIネットの場合）
```

postmap /etc/postfix/sasl_passwd　←　ISPのSMTP認証ファイルハッシュ化

SMTP-Auth用ユーザ名、パスワードとシステムのユーザ名、パスワードを別々にする場合
/etc/sasl2/smtpd.conf　←　SMTP-Auth認証設定ファイル編集※CentOS7,6の場合
```
pwcheck_method: saslauthd
↓
pwcheck_method: auxprop　←　変更
```

### Maildir形式メールボックス作成
Postfixのメール格納形式は共有ディレクトリ形式（｢/var/spool/mail/ユーザ名｣というファイルに全てのメールが蓄積されていく形式）だが、アクセス性能改善及びセキュリティ強化の観点からMaildir形式へ移行する。
* 【既存ユーザ対処】
既存ユーザのホームディレクトリにMaildir形式のメールボックスを作成して、蓄積済のメールデータを当該メールボックスへ移行する⇒メールデータ移行を参照
* 【新規ユーザ対処】
新規ユーザ追加時に自動でホームディレクトリにMaildir形式のメールボックスが作成されるようにする
```
mkdir -p /etc/skel/Maildir/{new,cur,tmp}　←　新規ユーザ追加時に自動でMaildir形式メールボックス作成
chmod -R 700 /etc/skel/Maildir/　←　メールボックスパーミッション設定
```

### Postfix起動
```
systemctl restart postfix　←　Postfix再起動※CentOS7の場合
systemctl enable postfix　←　Postfix自動起動設定※CentOS7の場合

### ポート25番のOPEN
```
firewall-cmd --add-service=smtp --permanent firewall-cmd --reload
```
