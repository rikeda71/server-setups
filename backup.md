# backup

- 実験データやプログラムを個人で管理できているかどうかには個人差がある．
- そのため，共用サーバでは，`/home`以下とデータベースのバックアップを取り，データを誤って削除してしまったとしても復旧できるようにしておく



## ハードディスクのマウント

1. OSがハードディスクを認識しているか確認する（以下はchocoの場合）
   - OSが入っているハードディスクとは別のハードディスクを認識しているか確認（以下の場合だと`/dev/sdb`）

```bash
$ sudo fdisk -l
Disk /dev/loop0: 89.1 MiB, 93454336 bytes, 182528 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 88.5 MiB, 92778496 bytes, 181208 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: E8DB4D2A-2730-438B-A4B4-9A3B20101360


Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 7CEFA5A3-C9D4-48B2-9E2B-13BC48218E5D

Device       Start        End    Sectors  Size Type
/dev/sda1     2048    1050623    1048576  512M EFI System
/dev/sda2  1050624 1953521663 1952471040  931G Linux filesystem
```

2. 新しいハードディスクの場合，パーティションが作られていないので，パーティションを作る必要がある

```bash
$ parted /dev/sdb  # /dev/sdbは任意
(parted) mklabel gpt  # パーティションテーブルの設定
(parted) mkpart primary  # パーティションの作成
File system type?  [ext2]? ext4
Start? 0%
End? 100%
(parted) p
Model: ATA WDC WD40EFRX-68N (scsi)
Disk /dev/sdb: 4001GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  4001GB  4001GB  ext4         primary

(parted) quit
Information: You may need to update /etc/fstab.

$ sudo mkfs.ext4 /dev/sdb1  # /dev/sdb1は任意
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 976754176 4k blocks and 244195328 inodes
Filesystem UUID: 8bc25132-68a2-4e98-ab60-a116b533215c
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848, 512000000, 550731776, 644972544

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done

$ sudo fdisk -l
.
.
.

Device     Start        End    Sectors  Size Type
/dev/sdb1   2048 7814035455 7814033408  3.7T Linux filesystem

.
.
.
# ... は表示が1.と同じ部分．ハードディスクのパーティションが増えていればOK
```



3. ハードディスクのマウント

```bash
$ sudo mount マウント元(例えば/dev/sdb1) マウント先(/media/*，chocoは/media/backup/chocoとしている)
```



## backupスクリプトの設定

1. backupスクリプトの用意

```bash
$ sudo mkdir -p /home/scripts  # スクリプトを置くディレクトリを作成．場所は任意
$ cd /home/scripts
$ sudo vim backup.sh
$ sudo vim dbdump.sh
```

- 参考までに，2019年時点で使っているファイルのバックアップとDBのバックアップを取るスクリプトのソースコードを示す
- backup.sh
  - 最大200件まで`/home`以下のバックアップを取るスクリプト

```bash
#!/bin/sh -eu

# バックアップ数の上限
BACKUP_MAX=200
DATE=`date "+%Y%m%d_%H%M%S"`
# バックアップ先
SYNC_DIR=/media/backup/biscuit
# 最新のディレクトリ名
CUR_DIR=CUR
# 世代管理ディレクトリ名
OLD_DIR=OLD
# ログファイル
LOG_FILE=$SYNC_DIR/backup.log

# 開始ログ作成
echo "$DATE: バックアップ開始" >> $LOG_FILE

# 世代管理ディレクトリのバックアップ数制御
while test `ls $SYNC_DIR/$OLD_DIR | wc -l` -ge $BACKUP_MAX
do
      rm -r $SYNC_DIR/$OLD_DIR/`ls $SYNC_DIR/$OLD_DIR | sort | head -n 1`
done

# rsyncの受け側と世代管理ディレクトリの設定
DEST_DIR=$SYNC_DIR/$CUR_DIR
BACKUP_DIR=$SYNC_DIR/$OLD_DIR/$DATE/$CUR_DIR
mkdir -p $BACKUP_DIR

# バックアップ対象ファイルの指定
sudo rsync -au --delete --backup --backup-dir=$BACKUP_DIR /home $DEST_DIR

# バックアップされたものが何もなければディレクトリを削除
if test `ls -a $BACKUP_DIR | wc -l` -eq 2 ; then
    rm -r $SYNC_DIR/$OLD_DIR/$DATE
    echo "バックアップ対象がありませんでした。" >> $LOG_FILE
fi

# 終了ログ作成
echo "正常終了" >> $LOG_FILE
```

- dbdump.sh
  - PostgreSQL や MySQLを使わない場合，このスクリプトは用意しなくても良い
  - 使っているDBに応じて，スクリプトの内容は変更する

```bash
#!/bin/sh -eu

SYNC_DIR=/media/backup/biscuit

# PostgreSQL
mv $SYNC_DIR/DB/postgres_dumpall.sql $SYNC_DIR/DB/postgres_dumpall_old.sql
sudo -u postgres pg_dumpall -f $SYNC_DIR/DB/postgres_dumpall.sql

# MySQL
mv $SYNC_DIR/DB/mysql_dumpall.sql $SYNC_DIR/DB/mysql_dumpall_old.sql
mysqldump -A --single-transaction --quick > $SYNC_DIR/DB/mysql_dumpall.sql
```



2. cronの設定

```bash
$ sudo crontab -u root -e
no crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 2  # vimを使うなら
No modification made
```

- 参考までに，2019年時点で使っているcronの設定を示す

```vim
# m h  dom mon dow   command
0 0-23/6 * * * /home/scripts/backup.sh
0 1 * * 0 /home/scripts/dbdump.sh
```
