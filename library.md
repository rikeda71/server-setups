# ライブラリのインストール

- python3（ビルドする場合）

  - OSに入っているpython3のバージョンは最新でないことがほとんど

  - よって，ソースからビルドすると良い

  - `netplan`(ネットワークの設定をするコマンド)がOSのpython3を消したら無くなるので（OSのpython3に`netplan`が依存している），OSのpython3を消さずに入れる必要がある

  - インストールはhttps://www.python.jp/install/ubuntu/index.htmlを参考にする

    ```bash
    $ sudo apt install build-essential libbz2-dev libdb-dev \
      libreadline-dev libffi-dev libgdbm-dev liblzma-dev \
      libncursesw5-dev libsqlite3-dev libssl-dev \
      zlib1g-dev uuid-dev tk-dev
    $ wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tgz  # python3.8.0の場合
    $ tar xzf Python-3.8.0.tgz
    $ cd Python-3.8.0
    $ ./configure --enable-shared
    $ make
    $ sudo make install
    $ sudo sh -c "echo '/usr/local/lib' > /etc/ld.so.conf.d/custom_python3.conf"
    $ sudo ldconfig
    $ python3.8
    ```

- python3（OSに入っているものを使う場合）

  - 関連するライブラリをインストールしておく

  ```bash
  $ sudo apt install build-essential libbz2-dev libdb-dev \
    libreadline-dev libffi-dev libgdbm-dev liblzma-dev \
    libncursesw5-dev libsqlite3-dev libssl-dev \
    zlib1g-dev uuid-dev tk-dev
  $ sudo apt install python3-venv python3-dev
  ```

- mecab，mecab-neologd

  ```bash
  sudo apt install mecab libmecab-dev mecab-ipadic-utf8 git make curl xz-utils file
  ```

  - neologdの導入

  ```bash
  $ git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd.git
  $ cd mecab-ipadic-neologd
  $ sudo ./bin/install-mecab-ipadic-neologd -n -a # 結果を確認する画面で`yes`と入力
  ```

- Postgresql

  - 研究室では，共用サーバ（biscuitやcandy）のDBにpostgresqlを採用している
  - 自分のクライアントに，MySQLやRDBでないもの（MongoDB）などを導入するのはOK

  ```bash
  $ sudo apt install postgresql-client libpq-dev  # clientだけ導入したい場合（サーバー上にDBを持たない場合）
  $ sudo apt install postgresql libpq-dev  # postgresqlを導入したい場合
  ```

  - DBの設定

    - クライアント名（s1○t2○○; 学籍番号名）と同じ名前のユーザとデータベースを作成し，運用する
    - `psql`と入力すると，`psql -U clientname -h localhost -d clientname`と解釈されるので，クライアント名と同じ名前のユーザとデータベースを作成することで，ユーザが簡単にデータベースを導入することができる．
    - ユーザはクライアント名と同じデータベース内でテーブルを作成し，研究を行う．
    - クライアント名と違う名前のデータベースを作成したい場合は要望に応じてあげてください．

    ```sql
    # ログイン後（ログインは`psql -U postgres -h localhost`などのコマンドで）
    postgres=> CREATE ROLE s1○t2○○ with login password `password`;  # (`password`は任意のパスワード)
    postgres=> CREATE DATABASE s1○t2○○ owner s1○t2○○;
    CREATE DATABASE
    ```

    - 設定ファイルは`/etc/postgresql/*/main/`にある（*はpostgresqlのバージョンが入ったり入らなかったり）
    - 研究室では，`pg_hba.conf`を以下のように編集している
      - `133.92.147.0/24`（研究室のネットワークアドレス）からの接続を許可しているため，研究室のクライアントPCや研究室のネットワークに接続したノートPCからデータベースに接続できるようにしている

    ```less
      local   all             postgres                                peer

      # TYPE  DATABASE        USER            ADDRESS                 METHOD

      # "local" is for Unix domain socket connections only
      local   all             all                                     trust
      # IPv4 local connections:
      host    all             all             127.0.0.1/32            trust
      host    all             all             133.92.147.0/24         md5
      # IPv6 local connections:
      host    all             all             ::1/128                 trust
    ```

- graphviz （matplotlibが依存）

  ```bash
  $ sudo apt install graphviz
  ```

- docker

  - 参考（https://docs.docker.com/install/linux/docker-ce/ubuntu/）

  ```bash
  $ sudo apt install \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg-agent \
      software-properties-common
  $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  $ sudo apt-key fingerprint 0EBFCD88
  $ sudo add-apt-repository \
     "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) \
     stable"
  $ sudo apt update
  $ sudo apt install docker-ce docker-ce-cli containerd.io
  # 標準でdockerはroot権限が必要．root権限を持たないユーザが使用する場合は以下を実行
  $ sudo usermod -aG docker `username`(`username`はdockerの使用権限を持たせたいユーザ名)
  ```

- zsh

  ```bash
  $ sudo apt install zsh
  ```

