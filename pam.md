# Pamモジュール

## パラメータ
  1. ライブラリタイプ
    - auth: ユーザ認証
    - account: アカウントの制約や有効性
    - password: パスワード関連の設定
    - session: 接続時の動作設定
  1. コントロール
    - optional: ステータスを無視する
    - requireed: 失敗時はログインに失敗するが処理は継続する
    - requisite: 失敗時は処理を終了する
    - 成功時はアクセスを許可する
  1. モジュール名
    - `pam_listfile.so`など
  1. モジュールの引数
    - `pam_listfile.so`の場合
      - item: ACL対象(user/rhosts)
      - onerr: listfile失敗時の処理( succeedi(処理を続ける) / fail(処理を終了する) )
      - sense: ACL解釈設定(allow/deny)

## pam_listfile.so

 リストファイルに記載されたユーザのサービスへのアクセスを許可/拒否することができる

### 設定
  1. ユーザを任意のファイルに記載する
    - ここでは `/etc/sshd/sshd.deny` というファイルにACL制限対象のユーザ名を記述する

 ```
user01
```

  1. /etc/pam.d/sshdに以下設定を追記
    - リストのユーザを拒否したい場合
      - `auth required pam_listfile.so item=user sense=deny file=/etc/ssh_users onerr=fail`
    - リストのユーザを許可したい場合
      - `auth required pam_listfile.so item=user sense=allow file=/etc/ssh_users onerr=fail`

### /var/log/secure ログ

 ```
Aug 27 19:13:21 app001 sshd[12368]: pam_listfile(sshd:auth): Refused user user01 for service sshd
Aug 27 19:13:24 app001 sshd[12368]: Failed password for user01 from 10.0.2.2 port 49307 ssh2
```

