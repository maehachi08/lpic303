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
    - 
  1. security

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

