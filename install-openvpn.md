# OpenVPN サーバの構築

 CentOS7を使って、OpenVPNサーバを構築します。

## 構築フロー
  1. `openvpn` インストール
  1. `easy-rsa` のインストール
  1. サーバ設定
    - 認証局を構築しCA証明書を生成する
    - サーバ証明書の生成
    - DHパラメータの生成
    - TLS認証鍵の生成
    - 設定ファイル

### `openvpn` インストール

 EPELレポジトリからインストールします。

 ```sh
yum install -y epel-release
yum install -y openvpn openssl-devel lzo-devel pam-devel
```

### `easy-rsa` のインストール

 [easy-rsa](https://github.com/OpenVPN/easy-rsa) を導入して認証局を構築する

 ```sh
wget https://github.com/OpenVPN/easy-rsa/archive/master.zip
unzip master.zip
cp -r easy-rsa-master/easyrsa3/ /etc/openvpn/
rm -rf easy-rsa-master/
rm -f master.zip
```

### サーバ設定

#### 認証局を構築しCA証明書を生成する

 ```sh
# 認証局(easy-rsa)ディレクトリへ移動
cd /etc/openvpn/easyrsa3/

# 認証局を構築
./easyrsa init-pki

# CA証明書を生成
./easyrsa build-ca

# CA署名書をOpenVPN用のディレクトリへコピー
cp pki/ca.crt /etc/openvpn/
```

 実行ログ

 ```sh
[root@vpn001 easyrsa3]# ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easyrsa3/pki
[root@vpn001 easyrsa3]# ./easyrsa build-ca
Generating a 2048 bit RSA private key
......+++
...........+++
writing new private key to '/etc/openvpn/easyrsa3/pki/private/ca.key.cjWzqk8ueV'
Enter PEM pass phrase: <パスフレーズ>
Verifying - Enter PEM pass phrase: <パスフレーズ>
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]: <サーバホスト名>

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easyrsa3/pki/ca.crt
```

#### サーバ証明書の生成
  - 証明書への署名も一緒にやってくれる

 ```sh
./easyrsa build-server-full server nopass
cp pki/issued/server.crt /etc/openvpn/
cp pki/private/server.key /etc/openvpn/
```

 実行ログ

 ```sh
[root@vpn001 easyrsa3]# ./easyrsa build-server-full server nopass
Generating a 2048 bit RSA private key
..................+++
........+++
writing new private key to '/etc/openvpn/easyrsa3/pki/private/server.key.Gl10VI5QMC'
-----
Using configuration from /etc/openvpn/easyrsa3/openssl-1.0.cnf
Enter pass phrase for /etc/openvpn/easyrsa3/pki/private/ca.key: <CA証明書の生成時に設定したパスフレーズ>
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :PRINTABLE:'server'
Certificate is to be certified until Aug 19 04:28:00 2026 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
[root@vpn001 easyrsa3]# cp pki/issued/server.crt /etc/openvpn/
[root@vpn001 easyrsa3]# cp pki/private/server.key /etc/openvpn/
```

#### DHパラメータの生成

 ```sh
./easyrsa gen-dh
cp pki/dh.pem /etc/openvpn/
```

 実行ログ

 ```sh
[root@vpn001 easyrsa3]# ./easyrsa gen-dh
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
....................................

DH parameters of size 2048 created at /etc/openvpn/easyrsa3/pki/dh.pem

[root@vpn001 easyrsa3]# cp pki/dh.pem /etc/openvpn/
```

#### TLS認証鍵の生成

 ```sh
openvpn --genkey --secret /etc/openvpn/ta.key
cp /usr/share/doc/openvpn-2.3.6/sample/sample-config-files/server.conf /etc/openvpn/
```

 実行ログ

 ```sh
[root@vpn001 easyrsa3]# openvpn --genkey --secret /etc/openvpn/ta.key
[root@vpn001 easyrsa3]# cp /usr/share/doc/openvpn-2.3.11/sample/sample-config-files/server.conf /etc/openvpn/
```

#### 設定ファイル

 ```sh
cp -p /etc/openvpn/server.conf{,.orig}
vim /etc/openvpn/server.conf
```

 ```diff
--- /etc/openvpn/server.conf.orig       2016-08-21 13:35:41.359751935 +0900
+++ /etc/openvpn/server.conf    2016-08-21 14:31:06.560834968 +0900
@@ -82,7 +82,8 @@
 # Diffie hellman parameters.
 # Generate your own with:
 #   openssl dhparam -out dh2048.pem 2048
-dh dh2048.pem
+; dh dh2048.pem
+dh dh.pem
 
 # Network topology
 # Should be subnet (addressing via IP)
@@ -140,6 +141,7 @@
 # back to the OpenVPN server.
 ;push "route 192.168.10.0 255.255.255.0"
 ;push "route 192.168.20.0 255.255.255.0"
+push "route 192.168.0.0 255.255.255.0"
 
 # To assign specific IP addresses to specific
 # clients or if a connecting client has a private
@@ -241,7 +243,7 @@
 # a copy of this key.
 # The second parameter should be '0'
 # on the server and '1' on the clients.
-;tls-auth ta.key 0 # This file is secret
+tls-auth ta.key 0 # This file is secret
 
 # Select a cryptographic cipher.
 # This config item must be copied to
@@ -264,8 +266,8 @@
 #
 # You can uncomment this out on
 # non-Windows systems.
-;user nobody
-;group nobody
+user nobody
+group nobody
 
 # The persist options will try to avoid
 # accessing certain resources on restart
@@ -286,8 +288,8 @@
 # "log" will truncate the log file on OpenVPN startup,
 # while "log-append" will append to it.  Use one
 # or the other (but not both).
-;log         openvpn.log
-;log-append  openvpn.log
+log         openvpn.log
+log-append  openvpn.log
 
 # Set the appropriate level of log
 # file verbosity.
```

### クライアン設定

 ```sh
cd /etc/openvpn/easyrsa3
./easyrsa build-client-full client1
```

 実行ログ

 ```sh
[root@vpn001 easyrsa3]# ./easyrsa build-client-full client1
Generating a 2048 bit RSA private key
...........................................+++
..+++
writing new private key to '/etc/openvpn/easyrsa3/pki/private/client1.key.z8Ajo5BzIE'
Enter PEM pass phrase: <クライアント証明書のパスフレーズ>
Verifying - Enter PEM pass phrase: <クライアント証明書のパスフレーズ>
-----
Using configuration from /etc/openvpn/easyrsa3/openssl-1.0.cnf
Enter pass phrase for /etc/openvpn/easyrsa3/pki/private/ca.key: <サーバ証明書のパスフレーズ>
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :PRINTABLE:'client1'
Certificate is to be certified until Aug 19 05:32:53 2026 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
```




## ルーティング / ブリッジ

 OpenVPNサーバを構築する際に最初に






## ルーティング / ブリッジ

 OpenVPNサーバを構築する際に最初に


