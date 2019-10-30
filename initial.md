# Server メモ

※ 全てUbuntu18.04を想定しています．他のOSだと，パッケージ管理システム（aptやyum）やサービスの管理方法が違うので，随時置き換えてください

- mirrorの設定
  - aptの接続先は通常は日本ではなく，イギリスやアメリカ？（要確認）
  - 日本のサーバにすることでダウンロードの速度が上がる

```bash
$ grep '^deb ' /etc/apt/sources.list | head  # このコマンドでどこのサーバに接続されているか確認できる
# 下のように表示される
deb http://archive.ubuntu.com/ubuntu bionic main restricted
deb http://archive.ubuntu.com/ubuntu bionic-updates main restricted
deb http://archive.ubuntu.com/ubuntu bionic universe
deb http://archive.ubuntu.com/ubuntu bionic-updates universe
deb http://archive.ubuntu.com/ubuntu bionic multiverse
deb http://archive.ubuntu.com/ubuntu bionic-updates multiverse
deb http://archive.ubuntu.com/ubuntu bionic-backports main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu bionic-security main restricted
deb http://security.ubuntu.com/ubuntu bionic-security universe
deb http://security.ubuntu.com/ubuntu bionic-security multiverse
```

- 以下のコマンドで，サーバを日本国内（富山大学）に変更できる

```bash
$ sudo perl -p -i.bak -e 's%https?://(?!security)[^ \t]+%http://jp.archive.ubuntu.com/ubuntu/%g' /etc/apt/sources.list
$ sudo apt update  # http://jp.archiveから始まるアドレスが表示されたらOK
```


- SSHの導入

  ```bash
  $ sudo apt install openssh-server
  ```

  - ポートなどの設定

  ```bash
  $ sudo vim /etc/ssh/sshd_config
  ```

  ```vim
  # 変更箇所
  Port ****（任意のポート番号．candyなどは3922としている）  # 13行目付近
  PermitRootLogin no  # 32行目付近．rootでのログインを禁止
  ```

  - sshの有効化

  ```bash
  $ sudo systemctl enable ssh
  $ sudo systemctl restart ssh
  ```

  - sshができるかの確認

    - サーバ側
      - ipアドレスを確認する（以下のinetに書かれているもの．香川大学なら *133.92* から始まればOK）

    ```bash
    # 以下は，このドキュメントを作成すると同時に設定したchocoのもの
    $ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 00:d8:61:33:60:a5 brd ff:ff:ff:ff:ff:ff
        inet 133.92.147.85/24 brd 133.92.147.255 scope global eno1
           valid_lft forever preferred_lft forever
        inet6 fe80::2d8:61ff:fe33:60a5/64 scope link
           valid_lft forever preferred_lft forever
    ```

    - クライアント側

      ```bash
      $ ssh usernamer@133.92.*.* -p portnumber（例えば3922）
      ```

- その他の設定

  - timezone

  ```bash
  $ sudo timedatectl set-timezone Asia/Tokyo
  ```

