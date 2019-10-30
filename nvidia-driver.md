# NVIDIAドライバの導入

1. `lspci  | grep -i nvidia`
  - ここで何も表示されなければ環境構築不可
  - 搭載されているGPUが表示できるか
2. 既にnvidia-driverやCUDAが入っていないか確認．以下のコマンドでライブラリが表示されたら表示されなくなるまでライブラリを削除する

```
dpkg -l | grep nvidia
dpkg -l | grep cuda

# 削除するコマンド
$ sudo apt --purge remove nvidia-*
$ sudo apt --purge remove libnvidia-*
```

3. XServerを止める

```
sudo service lightdm stop
pkill Xorg
```

4. nvidia-driver用のリポジトリを追加する
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```

5. [NVIDIA公式](https://www.nvidia.co.jp/Download/Find.aspx?lang=jp)でPCに搭載されているGPUに対応したNVIDIAドライバを探す．基本的には対応しているかつ最新のものを入れておけば良さそう．
6. やり方は2つある．後者の方が確実
	- `sudo apt install nvidia-***`
  - `***`は5で見つけたバージョンを入力
	- NVIDIA公式から見つけたバージョンのドライバを`wget`などでダウンロード．その後，`sudo sh drivername`でドライバのインストールが実行される
7. 再起動
8. `nvidia-smi`でGPU使用率が表示されたら成功
9. CUDAは[公式](https://developer.nvidia.com/cuda-downloads)通りにやればミスはなくて済みそうだが，DLライブラリやそのバージョンによって依存するCUDAのバージョンが違うので，[nvidia-docker2](https://github.com/NVIDIA/nvidia-docker)を使うことを推奨



## UEFIのセキュアブート機を対象とする場合

- 近年のPCはBIOSがUEFIの場合が多い
- chocoのBIOSもUEFIで，上記の手順通りにやってもドライバが入らなかったため，ドライバをインストールのメモを以下に残しておく．
- セキュアブートかどうかの確認
  - 以下のように，`Secure boot enabled`と出たら，セキュアブートが有効化されている．以下が出なければ，上記の手順でインストールできているはず．．．

```bash
$ dmesg | grep Secure
[    0.000000] secureboot: Secure boot enabled
```

- https://qiita.com/arc279/items/99f08b549c95881007b9 を参考に，セキュアブートの鍵を登録して，rebootすることでnvidiaドライバのインストールができた．
