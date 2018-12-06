# OpenUpload構築メモ(CentOS)
## 検証環境
- 仮想化ツール1：Vagrant 2.1.5
- 仮想化ツール2：Oracle VirtualBox
- ゲストOS：Cent OS 6.8 (bento/centos-6.8)
- ポートフォワーディング設定
    - vagrantfile
    - config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
    - config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
- ツール1：Apache2
- ツール2：PHP 5.3.3
- ツール3：mySQL 14.14

## 基本設定
- sudo yum -y update
- sudo yum -y install vim vim-common vim-enhanced tree
- sudo vim ~/.vimrc
```
set number
highlight Comment ctermfg=Green 
highlight Constant ctermfg=Red 
highlight Identifier ctermfg=Cyan 
highlight Statement ctermfg=Yellow 
highlight Title ctermfg=Magenta 
highlight Special ctermfg=Magenta 
highlight PreProc ctermfg=Magenta

```

## スナップショット保存1：基本設定終了
- Vagrantスナップショット作成
    - exit
    - vagrant halt
    - vagrant snapshot save savepoint1
    - vagrant snapshot list

## ツールの取得
- sudo yum -y install httpd
    - httpd -v
- sudo yum -y install php php-cgi libapache2-mod-php php-common php-pear php-mbstring php-cli php-fpm php-dev php-zip php-pgsql php-mysql
    - php -v

## スナップショット保存2：基本ツールインストール完了
- Vagrantスナップショット作成
    - exit
    - vagrant halt
    - vagrant snapshot save savepoint2
    - vagrant snapshot list

## Webサーバ設定
- 参考ページ
    - https://www.server-world.info/

### Apache2の設定
- 参考ページ
    - https://www.server-world.info/query?os=CentOS_6&p=httpd&f=1
- sudo vim /etc/httpd/conf/httpd.conf
```
44行目：ServerTokens Prod
76行目：KeepAlive On
262行目：ServerAdmin watanabe@kingsoft.jp
292行目：DocumentRoot "/var/www/openupload"
338行目：AllowOverride All
402行目：DirectoryIndex index.html index.htm index.php
536行目：ServerSignature Off
```
- sudo /etc/rc.d/init.d/httpd restart
- sudo chkconfig httpd on 

### OpenUploadダウンロードと展開
- cd ~
- wget http://www.d-ip.jp/download/images/openupload-0.4rc1_dip.tar.gz
- sudo tar xvzf openupload-0.4rc1_dip.tar.gz -C /var/www
- sudo /etc/rc.d/init.d/httpd restart
- sudo chmod -R 777 /var/www

## スナップショット保存3：データベース以外完了
- Vagrantスナップショット作成
    - exit
    - vagrant halt
    - vagrant snapshot save savepoint3
    - vagrant snapshot list

## データベース設定：mySQL
- 参考ページ
    - https://www.server-world.info/query?os=CentOS_6&p=mysql&f=1

### インストール
- sudo yum -y install mysql-server
- sudo vim /etc/my.cnf
```
character-set-server=utf8
[mysqld]欄の最終行に追記
```
- sudo /etc/rc.d/init.d/mysqld start 
- sudo chkconfig mysqld on

### 初期設定
- sudo mysql_secure_installation 
    - Set root password? [Y/n]：y 
    - New password：test
    - Remove anonymous users? [Y/n]：y 
    - Disallow root login remotely? [Y/n]：y
    - Remove test database and access to it? [Y/n]：y
    - Reload privilege tables now? [Y/n]：y
    - exit
- sudo /etc/rc.d/init.d/mysqld restart

## ブラウザ操作
- 参考ページ
    - http://wkiki.seesaa.net/article/380244890.html
    - この手順に準拠して操作する
- データベースタイプ
    - mysqlを選択
- データベースオプション
    - ホスト：localhost
    - ユーザ名：user_test
    - パスワード：test
    - DB名：openipload
    - Create the database：OFF
    - Also create user?：OFF
    - DB管理者ユーザ：user_test
    - DB管理者パスワード：test
    - Populate database：Private Modes
- アプリケーションオプション
    - WebMaster E-mail：test@test
    - サイト E-mail：test@test
- ユーザ
    - Administrator：admin
    - Admin password：test
- プラグイン
    - 全部有効
- データベース初期化
    - 「Restart」ボタンをクリック
    - 「実行」ボタンを何回かクリック
- 設定保存
    - 「Save Configuration」ボタンをクリック
    - 「click here to start using your new site」ボタンをクリック (極めて重要)
        - http://localhost:2080/index.php

## 内容のまとめ
### Vagrant Box
- ここまでの手順を行い完成したVagrant Boxをパッケージ化し下記に保存した
- Box名：pesia_one/001_CentOS-6.8_OpenUpload
- https://app.vagrantup.com/pesia_one/boxes/001_CentOS-6.8_OpenUpload

### ブログ記事
- ここまでの手順を詳しくまとめたブログ記事を作成した
	- 【OpenUpload】CentOSへのインストール手順(成功)
	- https://www.pesia-one.com/entry/2018/09/22/070000

### 注意事項
- 検証環境ではアクセス先のアドレスが「http://localhost:2080」のようになる
- しかしファイルのダウンロードリンクは「http://localhost2380」のようになるため、手動で「:」を入力する必要がある
- PHP7.Xではエラーになりスクリプトが動作しない
	- CentOS6.8ではPHP5.3.3がインストールされるため問題はない。
	- Ubuntuなどで使う場合はPHP5.6などを指定する必要がある
