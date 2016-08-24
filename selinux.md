# SELinux

## アクセス制御の種類

 SELinuxの本題に入る前に、ファイルへのアクセス制御の種類について記載します。

### 任意アクセス制御(DAC)
  - 任意アクセス制御(DAC:Discretionary Access Control)
  - リソース所有者にアクセス制御を任せる(任意)方式
  - 主に、Linuxのパーミッション(読み,書き,実行)やACLなどを指す

### 強制アクセス制御(MAC)
  - 強制アクセス制御(MAC:Mandatory Access Control)
  - リソース所有者の意図に関らず、システムの管理者により一定のアクセス制御を強制する方式
  - アクセス制御ルールは管理者権限を持ったユーザに対しても適用される
  - リソース所有者の権限では変更できない
  - 主に、SELinux、TOMOYO Linux、Trusted BSD や Trusted Solarisなどを指す

### ロールベースのアクセス制御(RBAC)
  - ロールベースのアクセス制御(RBAC:Role-based access control)
  - SELinuxで採用されている
  - 「ロール」と呼ばれる複数のドメインを束ねたものを設定し、ユーザに付与する方式
  - ユーザは付与されたロール内のドメイン権限でアクセス権を持つ
  - rootユーザを含む全てのユーザに対してアクセス制限が可能

## SELinuxの有効( 強制 / 許容 ) / 無効
 - [5.4. SELinux の有効化および無効化](https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Working_with_SELinux-Enabling_and_Disabling_SELinux.html)

  1. Permissiveモード

    有効ではあるがSELinuxコンテキストに基づいたアクセス制限は強制されない

  1. Enforcingモード

    SELinuxコンテキストに基づいたアクセス制限を強制する

  1. Disable

    SELinuxは動作していない。*setenforceコマンドでは設定できない*

### コマンドでの設定
  - **setenforce** コマンドで設定
    - **0 : Permissiveモード**
    - **1 : Enforcingモード**
  - **getenforce** コマンドで確認

 ```sh
[root@app001 ~]# getenforce 
Enforcing
[root@app001 ~]# setenforce 0
[root@app001 ~]# getenforce 
Permissive
[root@app001 ~]# setenforce 1
[root@app001 ~]# getenforce 
Enforcing
```

### 設定ファイルでの設定
  - `/etc/sysconfig/selinux` の **SELINUX** 
  - 変更後はシステム再起動することで反映される

 ```
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
```

## SELinux管理コマンド

### semanage

`policycoreutils-python` パッケージに内包されているコマンドです。

 ```sh
yum install -y policycoreutils-python
```

#### SELinux user object typeのリスト

 ```sh
semanage user -l
```

 ```sh
[root@app001 ~]# semanage user -l

                Labeling   MLS/       MLS/                          
SELinux User    Prefix     MCS Level  MCS Range                      SELinux Roles

guest_u         user       s0         s0                             guest_r
root            user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
staff_u         user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
sysadm_u        user       s0         s0-s0:c0.c1023                 sysadm_r
system_u        user       s0         s0-s0:c0.c1023                 system_r unconfined_r
unconfined_u    user       s0         s0-s0:c0.c1023                 system_r unconfined_r
user_u          user       s0         s0                             user_r
xguest_u        user       s0         s0                             xguest_r
```

#### SELinux login user objectのリスト

 ```sh
semanage login -l
```

 ```sh
[root@app001 ~]# semanage login -l

Login Name           SELinux User         MLS/MCS Range        Service

__default__          unconfined_u         s0-s0:c0.c1023       *
root                 unconfined_u         s0-s0:c0.c1023       *
system_u             system_u             s0-s0:c0.c1023       *
```

## SELinuxにおけるアクセス制御
 - refs http://www.itmedia.co.jp/enterprise/articles/0502/17/news033.html

  1. 強制アクセス制御(MAC:Mandatory Access Control)
    - セキュリティ管理者が設定したアクセス制御ルールに、たとえroot特権ユーザでも強制させる
  1. 最小特権
    1. TE(Type Enforcement)
      - ディレクトリやファイル(タイプ)ごとにアクセス制限が可能
      - FTPやHTTPといったプロセス(ドメイン)ごとにアクセス制限が可能
    1. ドメイン遷移
      - SELinuxのドメインは通常子プロセスに継承されるが、親プロセスから子プロセスを起動する際に別のドメインで起動することができる機能
      - initプロセス(`init_t`)から起動するhttpdプロセスがinitと同ドメインでは権限が適切ではないので、`httpd_t` ドメインで起動する
    1. RMAC(ロールベース)
      - rootユーザを含む全てのユーザに対してアクセス制限が可能
  1. 監査ログ
    - アクセス拒否された場合に`/var/log/messages` にログを記録する
    - アクセス許可された場合にログを記録することも可能

## 設定

### コンテキストの確認方法

 ```
# <SELinux User>:<SELinux Role>:<Type Enforcementの属性>:<レベル>
system_u:object_r:httpd_config_t:s0
```

#### タイプリソース
  - lsコマンドの `-Z` オプションでSELinuxのコンテキストを表示できる

 ```sh
[root@app001 ~]# ls -dZ /root /etc/nginx/conf.d/
drwxr-xr-x. root root system_u:object_r:httpd_config_t:s0 /etc/nginx/conf.d/
dr-xr-x---. root root system_u:object_r:admin_home_t:s0 /root
```

#### ドメインリソース
  - psコマンドの `-Z` オプションでSELinuxのコンテキストを表示できる

 ```sh
[root@app001 ~]# ps auxfwwZ | egrep "[s]ystemd-journald|[n]ginx"
system_u:system_r:syslogd_t:s0  root       433  0.0  0.2  32692  1316 ?        Ss   Aug23   0:00 /usr/lib/systemd/systemd-journald
system_u:system_r:httpd_t:s0    root      1411  0.0  0.0 109896     4 ?        Ss   Aug23   0:00 nginx: master process /usr/sbin/nginx
system_u:system_r:httpd_t:s0    nginx     1420  0.0  0.0 110320     4 ?        S    Aug23   0:00  \_ nginx: worker process
```

## SELinuxコンテキストを設定する

### OSが持つ情報に基づいてSELinuxコンテキストを設定する
  - https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Working_with_SELinux-SELinux_Contexts_Labeling_Files.html

  正しいSELinuxコンテキストを再設定する場合は **restorecon** コマンドを使う。
  設定するコンテキストは`/etc/selinux/targeted/contexts/files/` 以下のファイルを参照する。

 ```sh
[root@app001 ~]# touch testfile
[root@app001 ~]# ls -Z ~/testfile 
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 /root/testfile
[root@app001 ~]# restorecon -RFv ~/testfile 
restorecon reset /root/testfile context unconfined_u:object_r:admin_home_t:s0->system_u:object_r:admin_home_t:s0
[root@app001 ~]# ls -Z ~/testfile 
-rw-r--r--. root root system_u:object_r:admin_home_t:s0 /root/testfile
```

### 一時的にSELinuxコンテキストを変更する

 一時的にSELinuxコンテキストを変更する場合は **chcon** コマンドを使う。
   - User => `-u`オプション
   - Role => `-r`オプション
   - Type => `-t`オプション

 ```sh
chcon -u <SELinux User> -r <SELinux Role> -t <SELinux Type>
```

 ```sh
[root@app001 ~]# touch testfile2 
[root@app001 ~]# ls -Z testfile2 
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 testfile2
[root@app001 ~]# chcon -u system_u -r object_r -t httpd_config_t testfile2 
[root@app001 ~]# ls -Z testfile2 
-rw-r--r--. root root system_u:object_r:httpd_config_t:s0 testfile2
```

### chconコマンドでの変更はrestoreconコマンドで再設定される

 chconコマンドで `httpd_config_t` タイプを設定したのですが、restoreconコマンドを実行したことで `admin_home_t` タイプに変更されました。

 ```sh
[root@app001 ~]# restorecon -RFv testfile2 
restorecon reset /root/testfile2 context system_u:object_r:httpd_config_t:s0->system_u:object_r:admin_home_t:s0
[root@app001 ~]# ls -Z testfile2 
-rw-r--r--. root root system_u:object_r:admin_home_t:s0 testfile2
```

