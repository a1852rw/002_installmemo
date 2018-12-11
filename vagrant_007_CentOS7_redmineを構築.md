# CentOS7にRedmineバージョン4.0をインストール

## 事前の準備
### 検証環境
- 仮想化ツール1：Vagrant 2.1.2
- 仮想化ツール2：Oracle VirtualBox 5.2.22
- ゲストOS：Cent OS 7.5 (bento/centos-7.5 v201811.25.0)
    - https://app.vagrantup.com/bento/boxes/centos-7.5
		- ツール1：Apache 2.4.6
		- ツール2：MySQL
		- ツール3：Redmine 3.4
		- ツール：Rails 4.2

### Vagrantの初期設定
- vagrant init bento/centos-7.5
- Vagrantfileを編集(メモ帳などで直接書き換える)
```txt
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
```

### CentOS7の初期設定
- sudo yum -y update
- uname -a
- cat /etc/redhat-release
    - 更新後のOSバージョンを確認
- sudo yum -y install vim tree git
- sudo vim /etc/vimrc
```text
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
sudo timedatectl set-timezone Asia/Tokyo
timedatectl status
```

## インストール作業
### 参考ページ
- Redmineのインストール
    - http://guide.redmine.jp/RedmineInstall/

### スナップショット1
- vagrant halt 
- vagrant snapshot save savepoint1
- vagrant snapshot list
- vagrant up

### 




