# ファイルシステムの暗号化

## ブロックデバイスの暗号化
  - ファイルの暗号化ではなく、ファイルシステムレベルで暗号化します
  - linux 2.6以降ではブロックデバイスを論理デバイスにマッピングする**Device-mapper** が提供されています
  - Device-mapperで管理する論理デバイスを暗号化対象とする仕組みを**dm-crypt** と呼びます
  - **dm-cryptでのファイルシステムの暗号化を実施するためのコマンドとしてcryptsetupが用意されている**

## cryptsetupを使った暗号化ファイルシステム
  - LUKS(Linux Unified Key Setup)
    - LUKSとは、暗号化ファイルシステムの標準的な仕様を指します。
    - https://ja.wikipedia.org/wiki/LUKS
  - `cryptsetup` コマンド
    - **初期化はluksFormat**
    - **暗号化ファイルシステムを開くのはluksOpen**
    - **暗号化ファイルシステムを閉じるのはluksClose**
  - `cryptsetup` コマンドでは `luksFormat` でパスワードを設定する
  - パスワードの設定箇所をcryptsetupでは **キースロット** と呼ぶ
  -  `cryptsetup` コマンドで複数のパスワードを設定できる
    - **パスワードを追加するのは luksAddKey**
    - **パスワードを無効(削除)するのはluksKillSlot**
  - 暗号化ファイルシステムの状態(キースロットなども)を確認するのは**luksDump**
    - **status** もあるが情報量は **luksDump** のほうが豊富

### cryptsetup ツールのインストール

 ```sh
yum install -y cryptsetup
```

### 暗号化ファイルシステムの作成

 ```sh
[root@app001 ~]# cryptsetup -c aes-xts-plain -s 256 luksFormat /dev/sdb1 

WARNING!
========
This will overwrite data on /dev/sdb1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase: 
Verify passphrase: 
```

### 暗号化ファイルシステムのマウント/アンマウント
  - **cryptmount** コマンド
