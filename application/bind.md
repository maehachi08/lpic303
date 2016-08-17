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
  1. ACL

## `named.conf` の構文チェック

 `named-checkconf` コマンド

## DNSの脆弱性
  - DNSキャッシュ ポイズニング
  - 中間者攻撃
  - Smurf 攻撃

