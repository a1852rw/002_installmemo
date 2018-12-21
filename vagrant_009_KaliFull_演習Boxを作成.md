li Linux Boxイメージの作成

## 参考URL
- Vagrant BOX の作成
  - http://pg.4696.info/vagrant/vagrant-box.html#OSISO
- Kali Linux 2018.4 導入と日本語化
	- https://doruby.jp/users/r357_on_rails/entries/Kali-Linux-2017-3
- Vagrant box 作成手順(CentOS 6.7)
	- https://qiita.com/KisaragiZin/items/64b664c8b20ea39a6cd1
- Ubuntu 16.04のVagrantで使うboxを作成する
	- http://yakubouzu.hatenablog.com/entry/2016/12/17/182737
- SSHで接続するとキーボードの挙動が変になる
	- https://forums.ubuntulinux.jp/viewtopic.php?id=5621
- arch wiki:LightDM
	- https://wiki.archlinux.jp/index.php/LightDM

## Kali linux設定
### Virtualbox操作
- ISOイメージダウンロード
- https://www.kali.org/downloads/
	- kali-linux-2018.4-amd64.iso
	- SHA1：7c65d6a319448efe4ee1be5b5a93d48ef30687d4e3f507896b46b9c2226a0ed0

### コマンドライン操作で初期設定
- sudo apt-get install -y openssh-server
	- /etc/init.d/ssh restart
	- systemctl enable ssh.service
- useradd -m vagrant
- passwd vagrant
	- retype: vagrant
	- ここでSSHによる接続が可能になる(今回はWSLのUbuntuで接続)
	- VirtualBoxのポートフォワーディング設定 ホスト：2222 ゲスト22に設定
- gpasswd -a vagrant sudo
	- Windows Power Shellで操作(WSLでも可)
	- ssh vagrant@127.0.0.1 -p 2222
- sudo apt-get update -y && sudo apt-get upgrade -y

### ツールのインインストール
- sudo apt-get install -y wireshark
- sudo apt-get install -y tree curl vim

### スナップショット作成1
- VirtualBox上のマウス操作でスナップショットを保存
	- savepoint1

### sudoの設定変更
- visudo
```
%sudo   ALL=(ALL:ALL) ALL：これを下記に書き換え
↓↓↓↓
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```
- ここはなぜかemacsで読み込まれるの注意

### SSHキーの設置
- ここはユーザー「vagrant」で実行
- mkdir /home/vagrant/.ssh
- chmod 700 /home/vagrant/.ssh
- cd /home/vagrant/.ssh
- curl -k -L -o authorized_keys 'https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub'
- chmod 600 /home/vagrant/.ssh/authorized_keys
- chown -R vagrant.vagrant /home/vagrant/.ssh

### 追加でVimの設定も変更
- /etc/vim/vimrc に追記する
- sudo vim /etc/vim/vimrc
```vimrc
set number
set title
syntax on
set smartindent

highlight Comment ctermfg=Green
highlight Constant ctermfg=Red
highlight Identifier ctermfg=Cyan
highlight Statement ctermfg=Yellow
highlight Title ctermfg=Magenta
highlight Special ctermfg=Magenta
highlight PreProc ctermfg=Magenta

set tabstop=2
set shiftwidth=2
```

### その他設定
- GUI上からアカウント「vagrant」の自動ログインを「有効」に設定
- su vagrant
	- sudo ln -sf /bin/bash /bin/sh
	- キーボード挙動がおかしい問題に対処

### キャッシュクリア
- ここの操作も「vagrant」で実行
- sudo apt clean all
- sudo apt-get autoclean
- sudo apt-get clean

### 動作確認
- sudo shutdown -r now

### スナップショット作成2
- VirtualBox上でスナップショットを手動で作成
	- savepoint2
- 作成後にVirtualBoxでのマウス操作で仮想環境の電源OFF

###　Windows PowerShell操作に戻る
- mkdir Documents\vagrant\011_KaliFull
	- パッケージ作成用にフォルダを作成
- vagrant package --base kali_full
	- パッケージを生成

### Vagrant Boxに登録
- vagrant box add --name kali_full package.box
  - 「kali_full」という名前て登録
- vagrant box list
	- 登録を確認

### 動作確認
- vagrant init kali_full
- vagrant up

### VagrantFileの編集
- 起動と同時に画面を表示するため下記を追記(コメントアウトを解除して「end」を書き込んだ)
```rb
config.vm.provider "virtualbox" do |vb|
#   # Display the VirtualBox GUI when booting the machine
vb.gui = true
end
```

### パッケージ作成
- vagrant halt
- vagrant package
- 出力されるファイル「package.box」をVagrant Cloudにアップ


## その他
- sudo apt-get install -y  virtualbox-guest-dkms
	- これでゲストOS側の準備も完了となる
- VirtualBox画面で操作し「Insert Guest Additions CD image」を行う
	- sudo sh /media/cdrom/VBoxLinuxAdditions.run
	- 自動的にマウントが行われるのでこのコマンドを実行してシステムを再起動するのみで手順は完了する
	- 再起動後にVirtualBox画面でこのCD-ROMイメージを取り出す
	- デスクトップ上のCDアイコンを右クリックして「EjectImage」でマウントを解除できる
	- ここはできなかったがそれ以外は解決。
		 - ファイル共有については後日検討する

### スナップショット作成4
- vagrant shapshot save savepoint1
- vagrant snapshot list
    - savepoint1が保存されたことを確認する
