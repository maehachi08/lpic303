# nmap

ネットワーク調査およびセキュリティ監査を行うためのオープンソースのツールである。主にポートスキャンに使用されるが、OSバージョンやサービスなどの情報を取得することもできる。

## インストール

 ```sh
yum install -y nmap
```

## スキャンテクニック
 - https://nmap.org/man/jp/man-port-scanning-techniques.html


### SYNスキャン
  - `-sS` オプションで使用した場合の挙動であり、デフォルト動作
  - `SYN`パケットを送り、その応答によってポート開放状況を確認する
  - 相手の `SYN/ACK`へ応答しない **ハーフオープン**

 [公式ページ](https://nmap.org/man/jp/man-port-scanning-techniques.html)からの抜粋

 ```
SYNスキャンはデフォルトであり、正当な理由で最もよく使用されるスキャンオプションである。強制的なファイアウォールによる妨害のない、高速なネットワーク上では、数千ポート毎秒という高速なスキャンを実行できる。SYNスキャンは、TCPコネクションを確立しないため、比較的秘匿性が高い。また、NmapのFin/Null/Xmas、Maimon、Idleスキャンのように特定のプラットフォームの特質に左右されることはなく、規格準拠のTCPスタックなら何に対しても機能する。さらには、open、closed、およびfilteredというポートの状態を明確かつ確実に区別することができる。

この技法は、完全なTCPコネクションを開くわけではないので、 ハーフオープン(half-open)スキャンと呼ばれることも多い。あたかも実際にコネクションを開くつもりがあるかのように、SYNパケットを送信し、応答を待つ。SYN/ACKの応答は、ポートが待ち受け状態(open)であることを示し、またRST(reset)は、待ち受け状態にないことを示している。数回再送信しても何の応答もない場合、ポートはfilteredと見なされる。また、ICMP到達不能エラー(タイプ 3、コード 1、2、3、9、10、13)が送り返された場合も、ポートはfilteredと見なされる。
```

### connect() スキャン
  - `-sT` オプション
  - SYNスキャンが実行できない場合のデフォルト動作
  - *BerkeleyソケットAPI* の onnect()システムコールを発行し接続を検証する

 [公式ページ](https://nmap.org/man/jp/man-port-scanning-techniques.html)からの抜粋

 ```
TCP Connect()スキャンは、SYNスキャンを選択できない場合のデフォルトのTCPスキャンタイプである。ユーザが生パケットの権限を持たないか、IPv6ネットワークをスキャンする場合がこれにあてはまる。Nmapは、他のほとんどのスキャンタイプのように生パケットに書き込むのではなく、connect()システムコールを発行して、ターゲットのマシンやポートにとのコネクションを確立するよう下位OSに要求する。これは、Webブラウザ、P2Pクライアント、その他ほとんどのネットワーク対応アプリケーションがコネクションを確立するために使用するのと同じ高レベルのシステムコールである。これは、「BerkeleyソケットAPI」というプログラミングインターフェースの一部である。Nmapは、生パケットの応答を回線から読み込むのではなく、このAPIを使って、接続を試みるたびにステータス情報を入手する。
```

### UDP スキャン
  - `-sU`オプション
  - UDPポートスキャンを実施する
  - `-sUT` などと指定することでTCP/UDP 両ポートスキャンを同時実行可能である

### Nullスキャン
  - `-sN`オプション
  - 何のビットも設定しない(tcpヘッダのフラグは0)

### FINスキャン
  - `-sF`オプション
  - TCP FINビットだけを設定する

### Xmasスキャン
  - `-sX`オプション
  - FIN、PSH、URGのフラグをすべて設定し、クリスマスツリーのようにパケットをライトアップする

### ACK スキャン
  - `-sA`オプション
  - ファイアウォールのルールセットを明らかにするために用いられ、ファイアウォールがステートフルか否か、どのポートがフィルタされているかなどを決定する

### ウィンドウスキャン
  - `-sW`オプション
  - RSTが返されたら常にunfilteredと分類するのではなく、特定のシステムの実装に関する情報を用いて、openポートとclosedポートを識別する

### Maimon スキャン
  - `-sM`オプション
  - 発見者であるUriel Maimon氏の名前にちなんで名付けられた
  - プローブがFIN/ACKであるという点以外は、Null、FIN、Xmasスキャンとまったく同じもの

### カスタム TCP スキャン
  - `--scanflags`オプション
  - 任意のTCPフラグを指定することで、ユーザ独自のスキャンを設計することができる

### Idle スキャン
  - `-sI` オプション
  - 対象ホストに対して完全に匿名でTCPポートスキャンを実行できる
    - スキャンする側の実IPアドレスからは、対象ホストにパケットが送信されない

### IP プロトコル スキャン
  - `-sO`オプション
  - ターゲットマシン上でどのIPプロトコル(TCP、ICMP、IGMPなど)がサポートされているかを特定できる
  - TCP や UDPのポート番号ではなくて、IPプロトコル番号なので、厳密にはポートスキャンとは言えない

### FTP バウンス スキャン
  - `-b`オプション
  - プロキシFTP接続に対応

## スキャンの実施

 - デフォルト動作

 ```sh
# nmap 127.0.0.1

Starting Nmap 6.40 ( http://nmap.org ) at 2016-08-12 15:36 JST
Nmap scan report for app001.maepachi.local (127.0.0.1)
Host is up (0.00012s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 90.09 seconds
```

 - `-sS`(SYNスキャン)

 ```sh
# nmap -sS 127.0.0.1

Starting Nmap 6.40 ( http://nmap.org ) at 2016-08-12 15:45 JST
Nmap scan report for app001.maepachi.local (127.0.0.1)
Host is up (0.000020s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 3.62 seconds
```

 - `-sT`(connect()スキャン)

 ```sh
```

 - `-sU`(UDP)

 ```sh
# nmap -sU 127.0.0.1

Starting Nmap 6.40 ( http://nmap.org ) at 2016-08-12 16:15 JST
Nmap scan report for app001.maepachi.local (127.0.0.1)
Host is up (0.00012s latency).
Not shown: 998 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
5353/udp open|filtered zeroconf

Nmap done: 1 IP address (1 host up) scanned in 187.39 seconds
```




