# メールセキュリティ

## sendmail

### セキュリティ化する
  1. TLS有効化
  1. VRFY無効化
  1. chroot環境でsendmailを動作させる

## Postfix メールセキュリティ

### 1. chroot環境でデーモンを起動する
  - `/etc/postfix/master.cf` にて設定する
  - Postfixが攻撃された時の影響範囲を狭めることが可能

#### proxymap( Postfix 検索テーブルプロキシ)

chroot環境ではローカルアドレス宛メールの宛先チェックで/etc/passwd を参照することができず(chroot監獄)、chroot内にpasswdファイルをコピーするのは運用上好ましくない。

このような場合に、proxymapサーバによってオープンテーブルを共有し、それを複数のプロセス(qmgrなど)で参照することを実現する。

 - `/etc/passwd`

 ```
local_recipient_maps =
  proxy:unix:passwd.byname $alias_maps
```

 - mysql

 ```
virtual_alias_maps =
  proxy:mysql:/etc/postfix/virtual_alias.cf
```

### EXPN/VRFYコマンドを無効化
  - `/etc/postfix/master.cf` にて設定する
  - `disable_vrfy_command = yes`

### Postfix のバージョンを非表示にする
  - `/etc/postfix/master.cf` にて設定する
  - `smtpd_banner = $myhostname ESMTP $mail_name`

### TLS有効化
  - `/etc/postfix/master.cf` にて設定する
  - `smtpd_tls_cert_file`
  - `smtpd_tls_key_file`
  - `smtpd_tls_CAfile`
  - `smtpd_use_tls`

### メールルールの違反の通知方法
  - 各設定はカンマ区切り
    - `rulename =ルール名`
    - `severity = $(SIG_HI)`
    - `emailto = mail;mail`
      - **メールアドレスはセミコロン繋ぎ**

 ```
 ( rulename = “Mail Configuration”, severity = $(SIG_HI),
 emailto = user@sample.com;test@sample.com )
 ```
 
