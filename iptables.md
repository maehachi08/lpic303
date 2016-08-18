# iptables

## iptablesルールセット
  - nat
  - filter
  - mangle
  - raw

## iptablesコマンドの処理内容を問う

 ```
iptables -t <table> <command> <match rule> <taget / jump>
```

### table
  - LPIC 303でいう **iptablesルールセット** を指す
    - nat
    - filter
    - mangle
    - raw

### command
  - -A(--append)
    - チェインのルールを最後に追加
  - -D(--delete)
    - チェインのルールを削除
  - -F(--flush)
    - チェインの内容を消去
  - -I(--insert)
    - チェインの指定ルール番号の箇所に挿入
  - -L(--list)
    - チェインのルールを表示
  - -N(--new-chain)
    - チェインの作成
  - -X(--delete-chain)
    - チェインの削除
  - -Z(--zero)
    - すべてのチェックインのパケットカウンタとバイトカウンタを0に設定

### match rule
  - -s(--source)
  - -d(--destination)
  - -i(--in-interface)
  - -p(--protocol)
  - -sport(--source-port)
  - -dport(--destination-port)

### target / jump
  - -j(--jump)


