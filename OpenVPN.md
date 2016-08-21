# OpenVPN
  - [OpenVPNで使用できる認証方法](http://www.openvpn.jp/document/authentication-methods/)
  - [OpenVPN 認証方法についてのまとめ - 前編](http://yamatamemo.blogspot.jp/2013/02/openvpn.html)
  - [OpenVPN 認証方法についてのまとめ - 後編](http://yamatamemo.blogspot.jp/2013/02/openvpn_18.html)

## ポイント
  - `UDP/1194`
  - VPNクライアントにDHCPでIPを払い出す際に、DNSサーバのIPアドレスを渡したい
    - `push "dhcp-option DNS <IPアドレス>"`

## OpenVPN 設定ファイル
  - `/etc/openvpn/server.conf`
  - `/etc/openvpn/client.conf`

## 認証方法

OpenVPNでクライアントからVPN接続する際の認証方法について記載します。

### 静的鍵(Static Key)

 OpenVPNサーバにて生成した鍵(静的鍵)をクライアントでも保持し、接続時に使用する認証方式です。

 ```sh
openvpn --genkey --secret static.key
```

### 証明書認証

 認証機関を自前で設置（easy-rsaなどを使用）し、証明書と秘密鍵で認証する方法で、サーバーごと、クライアントごとに個別の秘密鍵、証明書を生成します。

### ID/パスワード認証

 OpenVPNサーバ上の設定ファイルでユーザ名とパスワードを設定しておきます。

### 二要素認証(PKCS#12)



