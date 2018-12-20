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

## Kali linux設定
### Virtualbox操作
- ISOイメージダウンロード
- https://www.kali.org/downloads/
	- kali-linux-light-2018.4-amd64.iso
	- SHA1：43c84be8ba616ea92a6d305738cde86fe58c12a4

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

[ここでスナップショット撮影：savepoint1]

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

### Virtualboc Guest Additionsのインストール：省略
- mkdir /media/cdrom
- mount -r /dev/cdrom /media/cdrom
- sh /media/cdrom/VBoxLinuxAdditions.run
- umount /media/cdrom
- これやったあとVirtualBoxを更新したら見事エラー
	- VirtialBoxからの起動もできなくなったため作り直し
- この手順は省略することにした

### キャッシュクリア
- ここの操作も「vagrant」で実行
- sudo apt clean all
- sudo apt-get autoclean
- sudo apt-get clean

### Macアドレスマッピングの無効化
- ln -s -f /dev/null /etc/udev/rules.d/70-persistent-net.rules
	- これhはCentOS7での設定よって今回は省略

[ここでもう一回保存：savepoint2]

### その他
- VirtialBox操作でポートフォワーディング設定を削除する

###　Windows PowerShell操作に戻る
- mkdir Documents\vagrant\001_createtest
	- パッケージ作成用にフォルダを作成
- vagrant package --base kali_lite
	- パッケージを生成

### Vagrant Boxに登録
- vagrant box add --name kali_light package.box
  - 「kali_light」という名前て登録
- vagrant box list
	- 登録を確認

### 動作確認
- vagrant init kali_light
- vagrant up

## 結果
### 問題点1
- できたことはできた
	- 画面が表示されない
	- rootでは問題ないが「vagrant」で入った時のキーボードがおかしい
	- 共有フォルダの設定が完了していない

### 対処1：前半
- 起動と同時に画面を表示するため下記を追記(コメントアウトを解除して「end」を書き込んだ)
```rb
config.vm.provider "virtualbox" do |vb|
#   # Display the VirtualBox GUI when booting the machine
vb.gui = true
end
```
	- 表示されたはいいがログイン画面が出てきて入力を求められる

### スナップショット作成1
- おおむねうまくいっているのでとりあえずここで保存しておく
- vagrant shapshot save savepoint1
- vagrant snapshot list
    - savepoint1が保存されたことを確認する

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

### 対処2：後半
- sudo vim /etc/lightdm/lightdm.conf
	- 該当部分を下記のように書き換える(コメントアウトの解除とログインするユーザーの指定)
```conf
[Seat:*]
autologin-user=vagrant
```
- sudo groupadd -r autologin
- sudo gpasswd -a vagrant autologin
