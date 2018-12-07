# 作業手順メモ：CentOS7にWordPessをインストール
## 概要
- とにかく確実にWordPressをインストールする手順をまとめた
- その後これを使って各種プラグインの動作を確認する
- ただしSSL化や外部DBの接続は考慮していない。この手順で構築したのちそれをすると正常に動作しない可能性がある。
	- そういった構成をとる場合は他の資料を参照することをお勧めする

## 検証環境
- 仮想化ツール1：Vagrant 2.1.2
- 仮想化ツール2：Oracle VirtualBox 5.2.22
- ゲストOS：Cent OS 7.5 (bento/centos-7.5 v201811.25.0)
    - https://app.vagrantup.com/bento/boxes/centos-7.5
- ツール1：Apache 2.4.6
- ツール2：PHP 5.4.16
- ツール3：MariaDB 15.1 
- ツール2：WordPress 5.0

## boxのインストールと設定
- ダウンロードサイト
	- https://app.vagrantup.com/bento/boxes/centos-7.5
- vagrant init bento/centos-7.5
- Vagrantfile を編集(メモ帳などで直接書き換える)
```
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
```
- vagrant plugin install vagrant-vbguest
- vagrant up
- vagrant ssh

## 初期設定
- sudo yum -y update
	- uname -a
	- cat /etc/redhat-release
	- 更新後のOSバージョンを確認
- sudo yum -y install vim vim-common vim-enhanced tree
- sudo vim /etc/vimrc
	- 参考サイト
	- https://ymyk.wordpress.com/2010/06/25/vim%E3%81%AE%E8%89%B2%E8%A8%AD%E5%AE%9A/
```
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
- sudo timedatectl set-timezone Asia/Tokyo
    - timedatectl status
- sudo localectl set-locale LANG=ja_JP.utf8
    - localectl status
- sudo vim /etc/bashrc
```bash
HISTTIMEFORMAT="%F %T  "
HISTFILESIZE=2000
```
- sudo reboot
- date
    - 日本語で表示されることを確認する

# 参考ページ
- WordPressをインストールする（CentOS7.4）
- https://qiita.com/kaikusakari/items/f3358855e0d21a1f4e99

## PHPインストール(Apache付属)
- sudo yum -y install php php-mysql
	- php -v
	- httpd -v
- sudo cp /etc/php.ini /etc/php.ini.org
- sudo vim /etc/php.ini
```ini
date.timezone = "Asia/Tokyo"
```
- sudo systemctl start httpd
    - sudo systemctl enable httpd
    - sudo systemctl list-unit-files | grep httpd

## MariaDBインストール
- sudo yum install -y mariadb-server mariadb-client
	- mysql --version
	- mariadbのバージョンが表示されれば成功
- sudo vim /etc/my.cnf
```cnf
[mysqld] セクション内に追記
character-set-server=utf8
```
- sudo systemctl start mariadb
- sudo systemctl enable mariadb

## mariaDB初期設定
- sudo mysql_secure_installation
    - Set root password? [Y/n] y
        - New password:wordpres_2018
        - Re-enter new password:wordpress_2018
    - Remove anonymous users? [Y/n] y
    - Disallow root login remotely? [Y/n] y
    - Remove test database and access to it? [Y/n] y
    - Reload privilege tables now? [Y/n] y

## MariaDB WordPree用設定
- mysql -u root -p
		- Enter Password: wordpress_2018
    - MariaDB> create user wordpress identified by 'wordpresspasswd';
    - MariaDB> create database wordpress;
    - MariaDB> grant all privileges on wordpress.* TO 'wordpress';
    - MariaDB> flush privileges;
    - MariaDB> exit;

## WordPressのインストール
- cd /tmp
- sudo wget https://wordpress.org/latest.tar.gz
- sudo tar -zxvf latest.tar.gz -C /var/www/
- cd /var/www
- sudo chown -R apache:apache *
- sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.org 
- sudo vim /etc/httpd/conf/httpd.conf
```cnf
DocumentRoot "/var/www/wordpress"
<Directory "/var/www/wordpress">
    AllowOverride All
</Directory>
```
- sudo systemctl restart httpd
- ブラウザでアクセス
    - http://localhost:2080/wp-admin/install.php
    - ブラウザ上で設定する

## WordPressの設定
- ブラウザでアクセス
    - http://localhost:2080/wp-admin/install.php
- データベースの設定
    - データベース名：wordpress
    - ユーザー名：wordpress
    - パスワード：wordpresspasswd
    - データベースのホスト名：localhost
- ブログの設定
	- タイトル：wordpress the world ブログ(仮)
	- 管理者ユーザ名：wordpress_2018
	- パスワード：wordpress_2018
	- メールアドレス：test@test.jp
	- 検索エンジンがこのサイトをインデックスしないようにする：有効

## 追加：Digest認証の導入
- ここまでの手順ではWordPressのセキュリティ設定が皆無である。そのため(余裕があれば)ログイン画面にダイジェスト認証を導入する
- sudo htdigest -c /etc/httpd/conf/htdigest wordpress wps_admin
	- Mew Password:wps_2018
	- re-type New Password:wps_2018
- sudo vim /etc/httpd/conf/httpd.conf
```
DocumentRoot "/var/www/wordpress"内の129行目あたりに下記を追記する
<Directory "/var/www/wordpress/wp-admin">
    AuthType Digest
    AuthName wordpress
    AuthUserFile /etc/httpd/conf/htdigest
    Require valid-user
</Directory>
```
- apachectl configtest
    - httpd.confの構文チェック
- sudo systemctl stop httpd
- sudo systemctl start httpd
- ブラウザ操作
    - http://localhost:2680/
        - WorpPressのトップページが認証なしで表示されることを確認
    - http://localhost:2680/wp-admin/
        - WordPressの設定ページが認証ありで表示されることを確認
        - ログインできることを確認

