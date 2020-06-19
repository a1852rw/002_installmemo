# CentOS-8にEC-Cubeの検証環境を構築
## 検証環境
- 仮想化ツール1：Vagrant 2.2.7
- 仮想化ツール2：Oracle VirtualBox 6.1.4
- ゲストOS：Cent OS 8.2.2004 (bento/centos-8.0)
https://app.vagrantup.com/bento/boxes/centos-8.0
ツール1：Apache 2.4.37
ツール2：PHP 7.2.24

## コマンドメモ
### 環境構築
- vagrant init bento/centos-8.0
- Vagrantfileを編集(メモ帳/テキストエディタ等で書き換え)
```txt
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
```
- vagrant up

### 初期設定
- vagrant ssh
- sudo yum update -y
- uname -a
- cat /etc/redhat-release
    - 更新後のバージョンを確認
- sudo yum install -y unzip vim tree
- sudo vim /etc/vimrc (ページ末尾に以下を追記)
```txt
set number
set title
set smartindent
```

### ここで保存
- exit
- vagrant snapshot save savepoint_001 --force
- vagrant snapshot list

### Apacheインストール
- sudo yum install -y httpd
    - httpd -v
- sudo systemctl start httpd.service
- sudo systemctl enable httpd.service
- ブラウザで動作確認
    - 127.0.0.1:2080
    - ウェルカムページが表示されたら成功

### ここで保存
- exit
- vagrant snapshot save savepoint_002 --force
- vagrant snapshot list

### PHPインストール
- sudo yum install -y epel-release
- sudo yum --enablerepo=epel install -y php php-mbstring php-xml php-xmlrpc php-gd php-pdo php-mysqlnd php-json php-pgsql php-pecl-apcu php-pecl-zendopcache php-mbstring php-intl php-zip php-phar php-zlib php-ctype php-session php-libxml php-openssl php-curl php-fileinfo
- php -v
    - バージョン情報が表示されれば成功
- sudo touch /var/www/html/info.php
- sudo chmod 766 /var/www/html/info.php
- sudo echo '<?php phpinfo(); ?>' > /var/www/html/info.php
- sudo systemctl restart httpd.service
- ブラウザで動作確認
    - 127.0.0.1:2080/info.php
    - PHPの情報ページが表示されれば成功
- sudo rm /var/www/html/info.php
    - ファイル削除

### ここで保存
- exit
- vagrant snapshot save savepoint_003 --force
- vagrant snapshot list

### MariaDBの設定
- sudo yum install -y mariadb mariadb-server
- sudo systemctl enable mariadb.service
- sudo systemctl start mariadb.service
- mysql_secure_installation
    - password: ec-cube
- mysql -uroot -p
    - create database eccube;
    - grant all privileges on eccube.* to eccube@localhost IDENTIFIED BY 'ec-cube';
    - exit
- mysql -u eccube -p
    - password：ec-cube
    - show databases;
    - テーブル「ec-cube」が表示されれば成功
- exit

### ここで保存
- exit
- vagrant snapshot save savepoint_004 --force
- vagrant snapshot list

### EC-Cubeのインストール
- sudo wget http://downloads.ec-cube.net/src/eccube-4.0.4.zip
- sudo unzip eccube-4.0.4.zip
- sudo mv ~/eccube-4.0.4/* /var/www/
- sudo mv ~/eccube-4.0.4/.htaccess /var/www/
- sudo chown -R apache:apache /var/www/*
- sudo vim /etc/httpd/conf/httpd.conf
    - 以下の通りApache2の設定ファイルを155行目付近を書き換える
```txt
<Directory "/var/www/html">
#AllowOverride None
AllowOverride All
```
    - 以下の通り167行目付近のルートディレクトリ設定を書き換える
```txt
<IfModule dir_module>
     DirectoryIndex index.html index.php
</IfModule>
```
- sudo systemctl restart httpd.service
- ブラウザで動作確認
    - 127.0.0.1:2080/index.php
    - EC-CUBEのインストール画面が表示されれば成功



## 参考ページ
- Server World 初期設定 : リポジトリを追加する2019
    - https://www.server-world.info/query?os=CentOS_8&p=initial_conf&f=7
- Server World Apache httpd : PHP スクリプトを利用する
    - https://www.server-world.info/query?os=CentOS_8&p=httpd&f=6
- CentOS 8にEC-CUBE 4をインストールした時の自分用メモ
    - https://qiita.com/okazy/items/6069f58345c0c42de439
- CentOS7にEC-Cube3をインストール（yumのみ）
    - https://labo-study.com/2016/01/26/centos7%E3%81%ABec-cube3%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%EF%BC%88yum%E3%81%AE%E3%81%BF%EF%BC%89/
- EC-CUBE3　超簡単インストールのご紹介 Linux
    - https://sys-guard.com/post-6825/
- 【初心者向け】EC-CUBE のインストール方法、失敗しない手順を図で解説します
    - https://www.yamatofinancial.jp/learning/pre-opening/how-to-install-ec-cube.html
- さくらVPS(CentOS 7)にEC-CUBE 3.0をインストールする
    - https://risa-webstore.com/blog/?p=71
