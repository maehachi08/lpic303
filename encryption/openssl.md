# OpenSSL

## SSL証明書 と CA(認証局)

  WebサーバなどでSSL通信を行う場合、サーバ側では **サーバ証明書** と **秘密鍵** を準備する必要があります。**サーバ証明書とは、秘密鍵と対になる公開鍵をCA(認証局)の秘密鍵で署名することで生成される証明書** を指します。


### 認証局(CA)の構築
  - [認証局（CA：Certification Authority）とは？](https://jp.globalsign.com/knowledge/ca.html)
  - Webサーバ等の公開鍵に署名し発行元が正しいことを証明する電子証明書を発行する
  - 自己署名の場合、Webサーバ等の秘密鍵を流用することが多いが、今回はCAサーバ環境を構築する

#### 環境準備

 1. CA用にディレクトリを作成
   - `private` ディレクトリは秘密鍵が格納されるため、パーミッションを `700` で作成

 ```sh
mkdir -p  /etc/pki/maepachiCA/{certs,private,crl,newcerts}
chmod 700 /etc/pki/maepachiCA/private 
```

 ```sh
[root@app001 maepachiCA]# ls -ld  /etc/pki/maepachiCA/{certs,private,crl,newcerts}
drwxr-xr-x. 2 root root 6 Aug 22 23:53 /etc/pki/maepachiCA/certs
drwxr-xr-x. 2 root root 6 Aug 22 23:53 /etc/pki/maepachiCA/crl
drwxr-xr-x. 2 root root 6 Aug 22 23:53 /etc/pki/maepachiCA/newcerts
drwx------. 2 root root 6 Aug 22 23:53 /etc/pki/maepachiCA/private
```

 1. `/etc/pki/maepachiCA/serial` を作成
   - 署名した発行順に証明書に付与されるシリアル番号を記録する
   - 本ファイルに記録されている値を次回の署名時に付与する

 ```sh
echo "01" > /etc/pki/maepachiCA/serial
```

 1. 証明書データベースを初期化

 ```sh
touch /etc/pki/maepachiCA/index.txt
```

#### 認証局用の証明書と秘密鍵を生成
  - RSA 2048ビットで鍵を生成
  - CA証明書は `cacert.pem`
  - 秘密鍵は `private/cakey.pem`

  1. 証明書の生成

 ```sh
cd /etc/pki/maepachiCA
openssl req -new -x509 -newkey rsa:2048 -out cacert.pem -keyout private/cakey.pem -days 1825
```

 ```sh
[root@app001 maepachiCA]# openssl req -new -x509 -newkey rsa:2048 -out cacert.pem -keyout private/cakey.pem -days 1825
Generating a 2048 bit RSA private key
..................................+++
...................................................................................+++
writing new private key to 'private/cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:Tokyo
Locality Name (eg, city) [Default City]:Arakawa-ku
Organization Name (eg, company) [Default Company Ltd]:maepachi
Organizational Unit Name (eg, section) []:test
Common Name (eg, your name or your server's hostname) []:app001.maepachi.local
Email Address []:maehachi08@gmail.com
```

  1. `private/cakey.pem` はパーミッションを変更

 ```sh
chmod 400 private/cakey.pem
```

  1. 証明書が生成されたことを確認

 ```sh
[root@app001 maepachiCA]# ls -l cacert.pem private/cakey.pem 
-rw-r--r--. 1 root root 1456 Aug 23 00:29 cacert.pem
-r--------. 1 root root 1834 Aug 23 00:29 private/cakey.pem
```
  1. 証明書の内容を確認

 ```sh
openssl x509 -in cacert.pem -text
```

#### 認証局の設定(openssl.cnf)
  - `/etc/pki/tls/openssl.cnf`
  - http://moca.wide.ad.jp/notes/ca_doc/openssl.html

 ```
[ CA_default ]
dir             = /etc/pki/maepachiCA    # Where everything is kept
certs           = $dir/certs             # Where the issued certs are kept
crl_dir         = $dir/crl               # Where the issued crl are kept
database        = $dir/index.txt         # database index file.
new_certs_dir   = $dir/newcerts          # default place for new certs.

certificate     = $dir/cacert.pem        # The CA certificate
serial          = $dir/serial            # The current serial number
crl             = $dir/crl.pem           # The current CRL
private_key     = $dir/private/cakey.pem # The private key
```

 `/etc/pki/tls/openssl.cnf` の修正前後での差分は以下の通りです。

 ```diff
--- /etc/pki/tls/openssl.cnf.original   2014-06-24 06:00:38.000000000 +0900
+++ /etc/pki/tls/openssl.cnf    2016-08-23 08:50:51.402742280 +0900
@@ -39,7 +39,7 @@
 ####################################################################
 [ CA_default ]
 
-dir            = /etc/pki/CA           # Where everything is kept
+dir             = /etc/pki/maepachiCA   # Where everything is kept
 certs          = $dir/certs            # Where the issued certs are kept
 crl_dir                = $dir/crl              # Where the issued crl are kept
 database       = $dir/index.txt        # database index file.
```

## 証明書の作成

### サーバ側で証明書と証明書要求(CSR)を生成

 1. サーバの秘密鍵を生成

 ```s
mkdir /etc/ssl/https
cd /etc/ssl/https
openssl genrsa -out server.key 
```

 1. 認証局へ渡す証明書要求(CSR)を生成
   - 証明書要求には公開鍵を含む。公開鍵を生成するために秘密鍵を読み込ませる必要がある。

 ```sh
openssl req -new -keyout server.key -out server-csr.pem
```

 生成された `server-csr.pem` の登録情報を確認する場合は以下コマンドで確認します。

 ```sh
openssl req -text -noout -in server-csr.pem
```

### 自己署名証明書を作成する(LPIC303対策)
  - **LPIC 303ではここ重要**
  - 自己署名証明書を **サーバ証明書** を記載することもある

#### すでに作成済みの秘密鍵とCSRに署名する

 ```sh
# openssl x509 -in <証明書要求> out <自己署名証明書> -req -signkey <秘密鍵> -days <期限>
openssl x509 -in /etc/ssl/https/server-csr.pem -out /etc/ssl/https/server-cert.pem -req -signkey /etc/ssl/https/server.key -days 365
```

 または、認証局を構成している場合は `openssl ca`コマンドでも可能。

 ```sh
openssl ca -out server-cert.pem -infiles server-csr.pem
```

#### 秘密鍵作成,CSR作成,自己署名を一度にする

 ```sh
# 暗号化する
openssl req -x509 -new -keyout /etc/ssl/https/server.key -out /etc/ssl/https/server.crt -days 365

# 暗号化しない
openssl req -x509 -nodes -new -keyout /etc/ssl/https/server.key -out /etc/ssl/https/server.crt -days 365
```

### 認証局(CA)で証明書要求に署名を行う
  - http://stacktrace.hatenablog.jp/entry/2015/12/12/102945

 ```sh
openssl ca  \
  -in /etc/ssl/https/server-csr.pem \
  -keyfile /etc/pki/maepachiCA/private/cakey.pem \
  -cert -cert /etc/pki/maepachiCA/cacert.pem \
  -out /etc/ssl/https/server-cert.pem 
```

## Apacheへの証明書組み込み
  - `/etc/httpd/conf.d/ssl.conf`

 ```
SSLCertificateFile /etc/httpd/conf.d/server.crt
SSLCertificateKeyFile /etc/httpd/conf.d/privkey.key
```

### opensslコマンドでの接続確認

 ```sh
 openssl s_client -connect <HOST>:443
```




