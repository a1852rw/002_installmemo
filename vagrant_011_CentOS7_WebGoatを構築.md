# CentOS7にWebGoatをインストール

## 事前の準備
### 検証環境
- 仮想化ツール1：Vagrant 2.2.2
- 仮想化ツール2：Oracle VirtualBox 6.0.2
- ゲストOS：Cent OS 7.5 (bento/centos-7.5 v201811.25.0)
    - https://app.vagrantup.com/bento/boxes/centos-7.5
- ツール1：java

### Vagrantの初期設定
- vagrant init bento/centos-7.5
- Vagrantfileを編集(メモ帳などで直接書き換える)
```txt
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 8080, host: 8080   # HTTP
config.vm.network "forwarded_port", guest: 9090, host: 9090  # HTTPS
```
- 以下はメモリの割り当てを4GBに増加させる記述。リソースに余裕がなければ不要です。
```txt
 config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  # Customize the amount of memory on the VM:
  vb.memory = "4092"
  end
```
- vagrant up

### CentOS7の初期設定
- sudo yum -y update && sudo yum -y upgrade
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

### スナップショット1
- vagrant halt 
- vagrant snapshot save savepoint1
- vagrant snapshot list
- vagrant up

## インストール作業
### 参考ページ
- WebGoatのインストールから起動までのメモ
    - https://qiita.com/weasel/items/1e71df51cd77e0065018
- ServerWorld OpenJDK 8 インストール
    - https://www.server-world.info/query?os=CentOS_7&p=jdk8&f=2
- CentOS 7 に Java 8 (OpenJDK) を yum インストールする手順
    - https://weblabo.oscasierra.net/installing-openjdk8-on-centos7/
- LinuxのJavaをバージョンアップする手順。alternativesコマンドでバージョン管理も完璧。
    - https://shimi-dai.com/install-java-on-linux/
- OWASP WebGoat：Git-hubのリポジトリ
    - https://github.com/WebGoat
- CentOS 7にApache Tomcatをインストールする(yum)
    - https://internalservererror.info/2018/05/01/centos-7にapache-tomcatをインストールするyum/
- JavaによるWebアプリケーションの仕組みをざっくり説明
    - https://qiita.com/mkdkkn/items/5c8b5b0ce549ac5d9014
- javaを使ってWebアプリを作るまでに結局なにが必要なのか。仕組みと学習に必要なものをザックリ説明
    - https://qiita.com/shimatter/items/c69ffd67a8ae4390b94f

### JREのインストール
- sudo yum install -y java-11-openjdk
    - ここを「java-18.0.0-openjdk」にするとWebGaotが起動しない

### WebGoatのインストール
- sudo mkdir ~/WebGoat-workspace
- cd ~/WebGoat-workspace/
- sudo wget https://github.com/WebGoat/WebGoat/releases/download/v8.0.0.M25/webgoat-server-8.0.0.M25.jar
    - git-hubからバージョン8.0.0.M24をダウンロードして保存した
    - https://github.com/WebGoat/WebGoat/releases/

### WebGoatの実行
- sudo java -jar webgoat-server-8.0.0.M25.jar

### ブラウザで操作
- ブラウザを開き下記にアクセス。ページが表示されることを確認する。
    - http://localhost:8080/WebGoat/login

## 備考
- よく見たらGithub上にVagrantでWebGoat環境を作るスクリプトが置かれていた
    - https://github.com/WebGoat/WebGoat/blob/develop/webgoat-images/vagrant-training/Vagrantfile
- このファイルを保存してvagrant up すれば今回まとめた手順は不要(ただしバージョンM21と古い)


<!---
java -jar webgoat-server-8.0.0.M25.jar --server.address=0.0.0.0
--->
