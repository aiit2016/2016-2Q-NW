
## Dovecot設定
--------------------------------------------------------------

### Dovecotインストール
```
yum -y install dovecot　←　Dovecotインストール
```

### Dovecot設定
* /etc/dovecot/conf.d/10-mail.conf　←　10-mail.conf編集
```
# Location for users' mailboxes. The default is empty, which means that Dovecot
# tries to find the mailboxes automatically. This won't work if the user
# doesn't yet have any mail, so you should explicitly tell Dovecot the full
# location.
#
# If you're using mbox, giving a path to the INBOX file (eg. /var/mail/%u)
# isn't enough. You'll also need to tell Dovecot where the other mailboxes are
# kept. This is called the "root mail directory", and it must be the first
# path given in the mail_location setting.
#
# There are a few special variables you can use, eg.:
#
#   %u - username
#   %n - user part in user@domain, same as %u if there's no domain
#   %d - domain part in user@domain, empty if there's no domain
#   %h - home directory
#
# See doc/wiki/Variables.txt for full list. Some examples:
#
#   mail_location = maildir:~/Maildir
#   mail_location = mbox:~/mail:INBOX=/var/mail/%u
#   mail_location = mbox:/var/mail/%d/%1n/%n:INDEX=/var/indexes/%d/%1n/%n
#
# 
#
#mail_location =
mail_location = maildir:~/Maildir　←　追加(メールボックス形式をMaildir形式とする)

# ':' separated list of directories under which chrooting is allowed for mail
# processes (ie. /var/mail will allow chrooting to /var/mail/foo/bar too).
# This setting doesn't affect login_chroot, mail_chroot or auth chroot
# settings. If this setting is empty, "/./" in home dirs are ignored.
# WARNING: Never add directories here which local users can modify, that
# may lead to root exploit. Usually this should be done only if you don't
# allow shell access for users. 
#valid_chroot_dirs =
valid_chroot_dirs = /home　←　追加※OpenSSH+Chrootを導入している場合のみ
```

* /etc/dovecot/conf.d/10-auth.conf　←　10-auth.conf編集
```
# Disable LOGIN command and all other plaintext authentications unless
# SSL/TLS is used (LOGINDISABLED capability). Note that if the remote IP
# matches the local IP (ie. you're connecting from the same computer), the
# connection is considered secure and plaintext authentication is allowed.
#disable_plaintext_auth = yes
disable_plaintext_auth = no　←　追加(プレインテキスト認証を許可)
※メールサーバー間通信内容暗号化(OpenSSL+Postfix+Dovecot)導入必須
```

* /etc/dovecot/conf.d/10-ssl.conf　←　10-ssl.conf編集
```
# SSL/TLS support: yes, no, required. 
# disable plain pop3 and imap, allowed are only pop3+TLS, pop3s, imap+TLS and imaps
# plain imap and pop3 are still allowed for local connections
ssl = no　←　SSL接続無効
※メールサーバー間通信内容暗号化(OpenSSL+Postfix+Dovecot)導入必須
```

### Dovecot起動
* Dovecot起動
```
systemctl start dovecot　←　Dovecot起動※CentOS7の場合
systemctl enable dovecot　←　Dovecot自動起動設定※CentOS7の場合
```

### ポート110番(POPの場合)または143番(IMAPの場合)のOPEN
ルーター側の設定でポート110番(POPの場合)または143番(IMAPの場合)をOPENする。
```
firewall-cmd --add-service=pop --permanent
firewall-cmd --add-service=imap --permanent
firewall-cmd --reload
```
