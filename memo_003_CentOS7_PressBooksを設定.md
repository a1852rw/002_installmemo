# 作業手順メモ：CentOS7にPressBooksを設定
## 概要
- PressBooksとはWordPressのプラグインの一種
    - WorPressを電子書籍のエディタとして使うことができる
    - 電子書籍と同様の形式でWeb上で公開/ダウンロードできるようにする
- これを利用することでWPSのマニュアルをオンライン上で管理することができる
    - 同時に電子書籍としてのAmazonへの掲載なども容易になると期待される
- 現状のWORD形式での管理を改善できると考えシステムの構築を試す
- とにかく確実にWordPressをインストールする手順をまとめた

## 検証環境
- 仮想化ツール1：Vagrant
- 仮想化ツール2：Oracle VirtualBox
- ゲストOS：Cent OS 7.5
    - https://app.vagrantup.com/bento/boxes/centos-7.5
- ツール1：Apache2
- ツール2：PHP
- ツール3：MariaDB 
- ツール2：WordPress

## boxのインストールと設定
- ダウンロードサイト
    - https://app.vagrantup.com/bento/boxes/centos-7.5
- vagrant init bento/centos-7.5
- vagrantfile
    -   # config.vm.network "public_network"
    - config.vm.network "forwarded_port", guest: 80, host: 2680   # HTTP
    - config.vm.network "forwarded_port", guest: 443, host: 26443  # HTTPS
- vagrant plugin install vagrant-vbguest
- vagrant up
- vagrant ssh

## 初期設定
- sudo yum -y update
    - uname -a
    - cat /etc/redhat-release
    - 更新後のOSバージョンを確認
- sudo yum -y install vim vim-common vim-enhanced tree
- 参考サイト
    - https://ymyk.wordpress.com/2010/06/25/vim%E3%81%AE%E8%89%B2%E8%A8%AD%E5%AE%9A/
- sudo vim /etc/vimrc
```vimrc
set number
highlight Comment ctermfg=Green 
highlight Constant ctermfg=Red 
highlight Identifier ctermfg=Cyan 
highlight Statement ctermfg=Yellow 
highlight Title ctermfg=Magenta 
highlight Special ctermfg=Magenta 
highlight PreProc ctermfg=Magenta 
```

## スナップショット保存：1
- exit
    - vagrant halt
    - vagrant snapshot save savepoint1
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

## 参考ページ
- WordPressをインストールする（CentOS7.4）
- https://qiita.com/kaikusakari/items/f3358855e0d21a1f4e99

## PHPインストール(Apache付属)
- sudo yum -y install php php-mysql
    - php -v
- sudo cp /etc/php.ini /etc/php.ini.org
- sudo vim /etc/php.ini
    - date.timezone = "Asia/Tokyo"
- sudo systemctl start httpd
    - sudo systemctl enable httpd
    - sudo systemctl list-unit-files | grep httpd

## MariaDBインストール
- sudo yum install -y mariadb-server mariadb-client
- sudo vim /etc/my.cnf
    - [mysqld] セクション内に追記
    - character-set-server=utf8
- sudo systemctl start mariadb
- sudo systemctl enable mariadb

## スナップショット保存：2
- exit
    - vagrant halt
    - vagrant snapshot save savepoint2
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

## mariaDB初期設定
- mysql_secure_installation
    - Set root password? [Y/n] y
        - New password:wordpress_2018
        - Re-enter new password:wordpress_2018
    - Remove anonymous users? [Y/n] y
    - Disallow root login remotely? [Y/n] y
    - Remove test database and access to it? [Y/n] y
    - Reload privilege tables now? [Y/n] y

## MariaDB WordPree用設定
- mysql -u root -p
    - MariaDB> create user wordpress identified by 'wordpresspasswd';
    - MariaDB> create database wordpress;
    - MariaDB> grant all privileges on wordpress.* TO 'wordpress';
    - MariaDB> flush privileges;
    - MariaDB> exit;

## WordPressのインストール
- cd /tmp
- sudo wget http://ja.wordpress.org/wordpress-4.9.8-ja.tar.gz
- sudo tar -zxvf wordpress-4.9.8-ja.tar.gz -C /var/www/
- cd /var/www
- sudo chown -R apache:apache *
- sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.org 
- sudo vim /etc/httpd/conf/httpd.conf
    - DocumentRoot "/var/www/wordpress"
    - <Directory "/var/www/wordpress">
    - AllowOverride All
    - </Directory>
- sudo systemctl stop httpd
- sudo systemctl start httpd
- ブラウザでアクセス
    - http://localhost:2680/wp-admin/install.php
    - ブラウザ上で設定する

## スナップショット保存：3
- exit
    - vagrant halt
    - vagrant snapshot save savepoint3
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

## WordPressの設定
- ブラウザでアクセス
    - http://localhost:2680/wp-admin/install.php
- データベースの設定
    - データベース名：wordpress
    - ユーザー名：wordpress
    - パスワード：wordpresspasswd
    - データベースのホスト名：localhost
- ブログの設定
    - タイトル：ブックブログ(仮)
    - 管理者ユーザ名：wordpress_admin
    - パスワード：wordpress_2018
    - メールアドレス：watanabe@kingsoft.jp

## スナップショット保存：4
- exit
    - vagrant halt
    - vagrant snapshot save savepoint4
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

## PressBooksのインストール
- 参考URL 
    - Pressbooks Open Source Plugin
    - https://pressbooks.com/pressbooks-open-source-plugin/
    - github
    - https://github.com/pressbooks/pressbooks
- githubからzipをダウンロードして読み込んだ
    - エラー発生で失敗
- 

