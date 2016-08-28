# キーワード
## ntop

### ntopのデプロイメントシナリオ
  1. シンプルホスト
  1. ボーダゲートウェイ
  1. ミラーライン

## Nagios
  1. `notification_inerval`
    - 特定のイベントに対してNagiosが1つ以上のアラート送信しないように0に設定するもの
  1. `retry_check_interval`
    - システムの異常検出時の監視間隔を設定

## nmap
  1. nmapに出てくる `RST`
    - RSTパケット(reset packet)
    - TCPで接続を中断・拒否する際に送られるパケット
      - SYNスキャンでポートが閉じている場合などに送信される

## OpenSSH
  1. OpenSSH関連コマンド
    - `ssh-vulnkey` : Openssh鍵の安全性チェック
    - `ssh-keygen` : 認証用の鍵生成・管理
    - `ssh-agent` : SSH認証エージェント
  1. 鍵認証のみにする
    - `PasswordAuthentication`
    - `PubkeyAuthentication`
  1. known_hostsのチェックを行わない
    - `StrictHostKeyChecking = no`
  1. `/etc/ssh/ssh_known_hosts`
    - SSH接続時に接続先サーバの公開鍵情報を保持する



