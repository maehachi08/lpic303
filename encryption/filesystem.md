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

 1. 暗号化ファイルシステムを作成

 ```sh
[root@app001 ~]# cryptsetup -c aes-xts-plain -s 256 luksFormat /dev/sdb1 

WARNING!
========
This will overwrite data on /dev/sdb1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase: 
Verify passphrase: 
```

 1. 作成した暗号化ファイルシステムの情報を表示する
   - `luksFormat` 時にパスフレーズを設定しているので `Key Slot 0: ENABLED` である
   - `Key Slot 1` 以降はまだ設定していないので `DISABLED` である

 ```sh
[root@app001 ~]# cryptsetup luksDump /dev/sdb1 
LUKS header information for /dev/sdb1

Version:        1
Cipher name:    aes
Cipher mode:    xts-plain
Hash spec:      sha1
Payload offset: 4096
MK bits:        256
MK digest:      46 67 a8 84 5d 6b e4 a4 3a e8 93 a5 20 60 e8 14 b8 4f 45 73 
MK salt:        53 79 b4 33 e3 f8 ea e0 95 b3 4d 25 db d9 f1 c3 
                29 8f 3d 0f a6 82 11 8e 15 1d ae 19 64 ba 47 8e 
MK iterations:  51500
UUID:           0c8cf9e4-6f83-4d4e-a2b8-1df71a476024

Key Slot 0: ENABLED
        Iterations:             207455
        Salt:                   f4 17 6e d3 46 8c b1 04 0a 8b 5a 43 d2 95 4c 57 
                                b1 6b 50 ce 61 8d a7 d5 b1 b2 1f dc b5 14 ac e0 
        Key material offset:    8
        AF stripes:             4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

### 空きキースロットにパスフレーズを登録する
 - 設定後のluksDumpで `Key Slot 1: ENABLED` となっていること

 ```sh
[root@app001 ~]# cryptsetup luksAddKey /dev/sdb1 
Enter any existing passphrase: 
Enter new passphrase for key slot: 
Verify passphrase: 
[root@app001 ~]# cryptsetup luksDump /dev/sdb1 
LUKS header information for /dev/sdb1

Version:        1
Cipher name:    aes
Cipher mode:    xts-plain
Hash spec:      sha1
Payload offset: 4096
MK bits:        256
MK digest:      46 67 a8 84 5d 6b e4 a4 3a e8 93 a5 20 60 e8 14 b8 4f 45 73 
MK salt:        53 79 b4 33 e3 f8 ea e0 95 b3 4d 25 db d9 f1 c3 
                29 8f 3d 0f a6 82 11 8e 15 1d ae 19 64 ba 47 8e 
MK iterations:  51500
UUID:           0c8cf9e4-6f83-4d4e-a2b8-1df71a476024

Key Slot 0: ENABLED
        Iterations:             207455
        Salt:                   f4 17 6e d3 46 8c b1 04 0a 8b 5a 43 d2 95 4c 57 
                                b1 6b 50 ce 61 8d a7 d5 b1 b2 1f dc b5 14 ac e0 
        Key material offset:    8
        AF stripes:             4000
Key Slot 1: ENABLED
        Iterations:             205787
        Salt:                   fc 0b 44 b6 19 06 f5 70 e7 95 5c 1e f3 a8 b0 8c 
                                8e 3d f7 14 40 72 9d ab 98 02 f2 49 8e 2c b3 e4 
        Key material offset:    264
        AF stripes:             4000
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

### 暗号化ファイルシステムを開く

 ```
[root@app001 ~]# cryptsetup luksOpen /dev/sdb1 luks2
Enter passphrase for /dev/sdb1: 
[root@app001 ~]# ls -l /dev/mapper/luks2
lrwxrwxrwx. 1 root root 7 Aug 19 23:50 /dev/mapper/luks2 -> ../dm-2
```

### luksOpenでエラー

 ```sh
[root@app001 ~]# cryptsetup luksOpen /dev/sdb1 luks2
cryptsetup: relocation error: /lib64/libcryptsetup.so.4: symbol dm_task_get_info_with_deferred_remove, version Base not defined in file libdevmapper.so.1.02 with link time reference
```

 [このIsuue](https://github.com/docker/docker/issues/17379)を参考にdevice-mapper-libsパッケージをアップデートすることで解決しました。

 ```sh
yum erase -y lvm2
yum update -y device-mapper-libs
yum install -y lvm2
```

### 暗号化ファイルシステムをフォーマットしてマウントしてみる

 1. フォーマット

 ```sh
[root@app001 ~]# mkfs.xfs /dev/mapper/luks2 
meta-data=/dev/mapper/luks2      isize=256    agcount=4, agsize=327424 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0
data     =                       bsize=4096   blocks=1309696, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

 1. マウント

 ```sh
[root@app001 ~]# mount /dev/mapper/luks2 /data 
```

 1. 確認

 ```sh
[root@app001 ~]# df -h /data
Filesystem         Size  Used Avail Use% Mounted on
/dev/mapper/luks2  5.0G   33M  5.0G   1% /data
```

### 暗号化ファイルシステムのマウント/アンマウント
 - http://cryptmount.sourceforge.net/
 - **cryptmount** コマンド

#### インストール

 ```sh
yum install -y https://copr-be.cloud.fedoraproject.org/results/kiso49j/cryptmount/epel-7-x86_64/00160174-cryptmount/cryptmount-5.2-1.el7.centos.x86_64.rpm
```

#### 設定
  - 設定ファイルは `/etc/cryptmount/cmtab`
    - sampleがある( `/etc/cryptmount/cmtab.example` )
  - 必要なカーネルモジュールが `/etc/modules-load.d/cryptmount.conf` に記載

##### 自動マウントに使用するパスフレーズを設定
 1. 乱数で `/boot/luks_key` を作成
 1. パーミッションを600に変更
 1. `/boot/luks_key` からキースロットにパスフレーズを登録

 ```sh
dd if=/dev/urandom of=/boot/luks_key bs=1 count=1024
chmod 600 /boot/luks_key

cryptsetup luksAddKey /dev/sdb1 /boot/luks_key
```

##### /etc/cryptmount/cmtab 作成

 ```
crypt_basic {
    dev=/dev/mapper/luks2
    dir=/data
    fstype=xfs         mountoptions=defaults
    cipher=aes-xts-plain
    keyfile=/boot/luks_key
    keyformat=keyfile
}
```

#### cryptmountサービスの起動

 ```sh
systemctl enable cryptmount
systemctl start cryptmount
```

#### 手動でcryptmountを実行する

 ```sh
cryptmount crypt_basic
```

