# UbuntuにWebサーバとPHP環境を構築

## 事前の準備
### 検証環境
- 仮想化ツール1：Vagrant 2.2.2
- 仮想化ツール2：Oracle VirtualBox 6.0.2
- ゲストOS：Ubuntu 18.10 7.5 (bento/ubuntu-18.10)
    - https://app.vagrantup.com/bento/boxes/ubuntu-18.10
- ツール1：Apache 2.4.34
- ツール2：PHP 7.2

## Vagrantの初期設定
- vagrant init bento/ubuntu-18.10
- Vagrantfileを編集(メモ帳などで直接書き換える)
```txt
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
```

## Ubuntu18.10 の初期設定
### パッケージのアップデート
- sudo apt update -y && sudo apt upgrade -y
- uname -a
- cat /etc/os-release
    - 更新後のOSバージョンを確認

### 基本コマンドのインストールと設定
- sudo apt install -y vim tree git ntp ntpdate
- sudo vim /etc/vim/vimrc
```text
set number
set title
syntax on
set smartindent
set encoding=utf-8
set wrap

highlight Comment ctermfg=Green 
highlight Constant ctermfg=Red 
highlight Identifier ctermfg=Cyan 
highlight Statement ctermfg=Yellow 
highlight Title ctermfg=Magenta 
highlight Special ctermfg=Magenta 
highlight PreProc ctermfg=Magenta

set tabstop=2
set shiftwidth=2
sudo timedatectl set-timezone Asia/Tokyo
timedatectl status
```
- sudo vim ~/.bashrc (下記を書き加える)
```txt
export PS1="\[\e[1;32m\]\u@\h \W\$\[\e[m\] "
```

### 時刻の設定
- sudo systemctl stop ntp
- sudo ntpdate ntp.nict.jp
- sudo vim /etc/ntp.conf
```txt
# 21行目付近：デフォルト設定をコメントアウトし無効化する
#pool 0.ubuntu.pool.ntp.org iburst
#pool 1.ubuntu.pool.ntp.org iburst
#pool 2.ubuntu.pool.ntp.org iburst
#pool 3.ubuntu.pool.ntp.org iburst

# Use Ubuntu's ntp server as a fallback.
#pool ntp.ubuntu.com
server ntp.nict.jp iburst
server ntp1.jst.mfeed.ad.jp iburst
server ntp2.jst.mfeed.ad.jp iburst 

# 53行目付近：時刻同期を許可する範囲を書き込み
restrict 10.0.0.0 mask 255.255.255.0 nomodify notrap
```
- sudo service ntp start
    - sudo systemctl restart ntp
    - sudo systemctl enable ntp
- sudo ntpq -p
- timedatectl list-timezones
- sudo timedatectl set-timezone Asia/Tokyo 
- timedatectl 
- date
    - 日付時刻が日本時間になっていることを確認

### スナップショット1
- vagrant halt 
- vagrant snapshot save savepoint1
- vagrant snapshot list
- vagrant up

## Webサーバの設定
- 参考ページ
    - Apache2 : インストール
    - https://www.server-world.info/query?os=Ubuntu_18.04&p=httpd&f=1
    - Apache2 : PHPスクリプトを利用する
    - https://www.server-world.info/query?os=Ubuntu_18.04&p=httpd&f=3

### PHPのインストールと設定
- ApacheはPHPインストール時に自動的にインストールされる
- sudo apt-get install -y php apache2
- php -v
- apache2 -v
- sudo systemctl restart apache2
- sudo systemctl enable apache2
- ブラウザを開き動作確認
    - http://localhost:2080
    - Apacheが表示されれば成功

### スナップショット2
- vagrant halt 
- vagrant snapshot save savepoint2
- vagrant snapshot list
- vagrant up

### Apacheの設定
- 

### MariaDBの設定
### 動作確認