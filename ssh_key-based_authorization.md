# ssh 公開鍵鍵認証

## ssh 公開鍵認証の必要性

- セキュリティの観点から，研究室のサーバには ssh 公開鍵認証によってアクセスしてもらいます．
  - 学内のサーバと言っても ssh で接続するために 22 番ポートは空いています．
  - 中国やロシアから恐ろしいほどのアクセスが試されています．
- ssh によるサーバへのアクセスを公開鍵認証のみに設定した場合，個人の秘密鍵が外部に漏れない限り，外部からアクセスされることはありません．
- 設定が面倒かもしれませんがご協力よろしくお願いします．

## 公開鍵認証の概要

- どのように認証されているか知りたい人は[この辺り](https://knowledge.sakura.ad.jp/3543/)を参考にしてください

## 設定方法

1. openssh をインストールする

   - [windows](https://docs.microsoft.com/ja-jp/windows-server/administration/openssh/openssh_install_firstuse)
     - wsl2 上で ssh 接続したい場合は，wsl2 の環境下で openssh をインストールする
   - mac は標準でインストールされているはず?

2. 以下のコマンドを実行

```shell-scripts
$ cd
$ ssh-keygen -t rsa
# いろいろ書かれているけどenterキーを押すだけでOK
Enter file in which to save the key (***/.ssh/id_rsa): (enter)
Enter passphrase (empty for no passphrase): (enter)
Enter same passphrase again: (enter)
$ ls .ssh # dir .ssh (windows)
id_rsa id_rsa.pub ...
```

3. 以下のコマンドで公開鍵を転送する

- ユーザ名 は s00t200 の形式
- ポート番号は任意の番号（わからなかったらサーバ管理者に問い合わせてください）

```shell-scripts
$ scp -P ポート番号 .ssh/id_rsa.pub ユーザ名@サーバのIPアドレス:/home/ユーザ名/.ssh/id_rsa.pub
```

4. サーバーに ssh 接続し，公開鍵を登録する

```shell-scripts
$ ssh ユーザ名@サーバのIPアドレス -p ポート番号
# ここではパスワード入力が必要なので，サーバ管理者にパスワードを入力してのsshを許可してもらう
# ログイン後
$ cat .ssh/id_rsa.pub >> .ssh/authorized_keys
$ rm .ssh/id_rsa.pub
$ exit
```

5. パスワードを使わずに ssh 接続できることを確認する

```shell-scripts
$ ssh ユーザ名@サーバのIPアドレス -p ポート番号
# パスワードを入力せずログイン可能
```

## ssh 公開鍵認証の利点

ここではサーバ管理者ではなくユーザから見た利点を記しておきます．

- 毎回パスワードを入力する必要なくログインできる
- (vscode ユーザのみ)vscode の remote development を有効活用できる
  - サーバ側のファイルを直接編集できる．もちろん補完も効く
