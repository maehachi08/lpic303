# アプリケーションセキュリティ

## Apache セキュリティ

### アクセス制限
  1. Order <allow/deny>
    - 許可/拒否の適用順序

  1. Allow from <from>
    - 許可ホストの設定

  1. Deny from <from>
    - 拒否ホストの設定

### Basic認証
  - 認証情報はhtpasswdコマンドで作成する
  - 設定には、AuthType,AuthName,AuthUserFile,Requireが必要

 ```
<Location "/path/to/basic/url">
    AuthType Basic
    AuthName "Secret Zone"
    AuthUserFile /etc/httpd/.htpasswd
    Require valid-user
</Location>
```

 またはディレクトリ指定。

 ```
<Directory "/path/to/basic/auth">
    AuthType Basic
    AuthName "Secret Zone"
    AuthUserFile /etc/httpd/.htpasswd
    Require valid-user
</Directory>
```

### .htaccess の無効化
  - `< Directory /> AllowOverride None </Directory>`

### SSL
  - http://www.lpi.or.jp/news/event/docs/20150711_01_report_01.pdf

#### サーバ証明書の作成

##### 作成フロー
  1. Webサーバで`openssl genrsa` コマンドで秘密鍵を生成
  1. Webサーバで`openssl req -new` コマンドで署名リクエスト(CSR)を生成
  1. CSRを認証局へ送り、認証曲の秘密鍵で署名したサーバ証明書を生成
    - `openssl ca -out <生成するサーバ証明書> ---infiles <CSRファイル>`

#### 設定
  - `SSLCertificateFile`
  - `SSLCertificateKeyFile`

 ```
SSLCertificateFile /etc/httpd/conf.d/server.crt
SSLCertificateKeyFile /etc/httpd/conf.d/privkey.key
```

