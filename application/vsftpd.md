# vsftpd

## chroot

vsftpdにおいてFTPアクセスするユーザがchroot環境をFTPのルートに設定されている場合、chrootのルートディレクトリより上位(それより上のディレクトリを辿らないといけないパス)に対してはアクセスができない。

### chroot環境でそれより上位ディレクトリへアクセスしたい場合
  - `/etc/vsftpd/vsftpd.conf`
    - `chroot_list_enable=YES`
    - `chroot_list_file=/etc/vsftpd/chroot_list`

### シンボリックリンクの代わりにmount –bind

 シンボリックリンクの代わりに`mount -bind` を使ことで、*chroot_list_enableで許可しなくてもリンク先にアクセスできる*

 ```sh
# mount --bind /home/B/org /home/A/lnk
```

