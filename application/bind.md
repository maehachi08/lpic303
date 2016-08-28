# Bindセキュリティ

## Bindサーバの保護
  1. namedデーモンを非rootユーザで実行する
  1. chroot jailで任意のディレクトリをルートと見せかける
  1. アクセス制限(IPアドレス)
    - allow-query
    - allow-recursion
    - allow-query-cache
    - allow-transfer
    - allow-notify
    - allow-update
    allow-update-forwarding
  1. ACLを構成する

## `named.conf` の構文チェック

 `named-checkconf` コマンド

## DNSの脆弱性
  - DNSキャッシュ ポイズニング
    - DNSサーバに誤ったテーブル情報を記憶させ、DNSクライアントに誤ったIPアドレスを返す
  - 中間者攻撃
    - `man in the middle` 攻撃
    - 通信しているホスト間に割り込んで通信を盗聴すること
  - Smurf 攻撃
    - IPMPエコーリクエストパケットの送信元を偽装してDNSサーバに大量にパケット送信する

## TSIG( Transaction Signature )
  - DNSサーバのゾーンデータの転送時のプライマリサーバのIPアドレス偽装を防ぐ
  - プライマリとセカンダリ間で共通の秘密鍵を保有します
  - DNSメッセージ全体に署名をおこなう
  - メッセージの完全性の保証やリクエスト認証を可能にします
  - TSIGを設定すればdigコマンドでゾーン転送の確認(AXFR)はできない

### TSIGの設定
  1. 共通鍵を生成する
    - `dnssec-keygen` コマンド
    - `/usr/sbin/dnssec-keygen -a HMAC-MD5 -b 512 -n HOST example.jp`
  1. .key ファイルから共有鍵の文字列を抜き出す
  1. `named.conf` に抜き出した文字を埋め込む
    - `secret 文字列`



