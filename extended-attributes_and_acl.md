# ファイルシステムの拡張属性(extended attributes)

## 拡張属性への対応について

Linuxのカーネル設定でlibattr機能が有効であれば使用可能。
  - ext2
  - ext3
  - ext4
  - JFS
  - ReiserFS
  - XFS

## 属性の定義方法
  - 属性情報は、ファイルごとに *名前* と *データ*の組みをリストで持つ。
  - *名前* には **名前空間**が必要

### 名前空間
  1. user
    - user名前空間には名前の付け方と内容に関する制限がない
  1. system
    - カーネルが主にアクセス制御リストとして使用する
  1. trusted
    - 信頼できるプロセスのみが利用
  1. security
    - たとえば、SELinuxで利用

### attrパッケージ

`setfattr` や `getfattr` はattrパッケージに含まれています。

 ```sh
yum install -y attr
```

### 属性の設定

 1. user名前空間での拡張属性を設定

 ```sh
setfattr -n user.version -v '0.0.1' testfile 
setfattr -n user.description -v 'extended attributes test' testfile 
getfattr -d testfile
```

 1. 対象ファイルの拡張属性を確認

 ```sh
# getfattr -d testfile 
# file: testfile
user.description="extended attributes test"
user.version="0.0.1"
```

 1. 対象ファイルの拡張属性を削除

 ```sh
setfattr -x user.version testfile
```

 削除されたことをgetfattrコマンドで確認します。

 ```sh
# getfattr -d testfile 
# file: testfile
user.description="extended attributes test"
```

## aclパッケージ

### インストール

 ```sh
yum install -y acl
```

### 拡張ACLの設定

 - user01に対してtestfileファイルに対して読み書き権限を設定します。

 ```sh
setfacl -m user:<ユーザ名>:<権限> <ファイル>
setfacl -m user:<グループ>:<権限> <ファイル>
```

 ```sh
[root@app001 ~]# getfacl testfile
# file: testfile
# owner: root
# group: root
user::rw-
group::r--
other::r--

[root@app001 ~]# setfacl -m user:user01:rw testfile
[root@app001 ~]# getfacl testfile
# file: testfile
# owner: root
# group: root
user::rw-
user:user01:rw-
group::r--
mask::rw-
other::r--
```

### 拡張ACLの確認

 ```sh
getfacl testfile
```

 ```sh
[root@app001 ~]# getfacl testfile
# file: testfile
# owner: root
# group: root
user::rw-
user:user01:rw-
group::r--
mask::rw-
other::r--
```

### 設定した拡張ACLをすべて削除

 ```sh
setfacl -b <ファイル>
```

 ```sh
[root@app001 ~]# setfacl -m user:user01:rw testfile
[root@app001 ~]# setfacl -m user:user02:rwx testfile
[root@app001 ~]# setfacl -m user:user03:r testfile
[root@app001 ~]# getfacl testfile
# file: testfile
# owner: root
# group: root
user::rw-
user:user01:rw-
user:user02:rwx
user:user03:r--
group::r--
mask::rwx
other::r--

[root@app001 ~]# setfacl -b testfile
[root@app001 ~]# getfacl testfile
# file: testfile
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

### 拡張ACLの削除

 ```sh
setfacl -x user:<削除したいユーザ> <ファイル>
```

 ```sh
[root@app001 ~]# setfacl -m user:user01:rw testfile
[root@app001 ~]# getfacl testfile
# file: testfile
# owner: root
# group: root
user::rw-
user:user01:rw-
group::r--
mask::rw-
other::r--

[root@app001 ~]# setfacl -x user:user01 testfile
[root@app001 ~]# getfacl testfile
# file: testfile
# owner: root
# group: root
user::rw-
group::r--
mask::r--
other::r--
```

