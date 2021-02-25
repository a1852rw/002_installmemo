# Lubuntu20.04にEC-Cubeの検証環境を構築
## 検証環境
- 仮想化ツール1：Vagrant 2.2.7
- 仮想化ツール2：Oracle VirtualBox 6.1.12
- ゲストOS： Lubuntu 18.04 (bento/ubuntu-18.04)
https://app.vagrantup.com/bento/boxes/ubuntu-18.04
- Vagrantプラグイン：vagrant-vbguest
- ツール1：Apache 2.4.41
- ツール2：PHP 7.4.3

構築済みのイメージをVagrant Cloudの以下に保存した。
  
- ユーザー：
- イメージ名：
- URL： []()

## コマンドメモ
### 環境構築
- vagrant init bento/ununtu-18.04
- vagrant up

### 初期設定
初期設定の内容については、以下URLを参照  
URL： [https://github.com/a1852rw/005_vagrantfile/tree/master/005_Lubuntu_18.04](https://github.com/a1852rw/005_vagrantfile/tree/master/005_Lubuntu_18.04)

URL内にて初期設定として実施している作業は以下の通り。

#### ゲスト OSの更新
- sudo apt-get update -y

#### デスクトップ環境のインストール
- sudo yes Y | sudo apt-get install -y --no-install-recommends lubuntu-desktop
- sudo sleep 20s

#### GUIモードの設定
- sudo systemctl set-default graphical.target
- sudo sleep 20s

#### 再度のゲストOS更新
- sudo apt-get update -y

#### 日本語環境の設定
- sudo apt-get install -y fonts-ipafont fonts-ipaexfont
- sudo apt-get install -y language-pack-ja-base language-pack-ja ibus fcitx fcitx-mozc 
- sudo apt-get install -y firefox-locale-ja language-pack-gnome-ja
- sudo update-locale LANG=ja_JP.UTF-8 LANGUAGE=”ja_JP:ja” LC_ALL=ja_JP.UTF-8
- sudo localectl set-keymap jp106
- sudo fc-cache -fv
- fc-match IPAGothic
    
#### 時刻の設定
- sudo timedatectl set-timezone Asia/Tokyo
- sudo systemctl enable systemd-timesyncd.service

#### その他必要なソフトの追加
- sudo apt-get install -y firefox vim wget curl tree git featherpad

#### vim設定カスタマイズ
- sudo echo " " >> /etc/vimrc
- sudo echo "set number" >> /etc/vimrc
- sudo echo "set title" >> /etc/vimrc
- sudo echo "syntax on" >> /etc/vimrc
- sudo echo "set smartindent" >> /etc/vimrc
- sudo echo " " >> /etc/vimrc
- sudo echo "highlight Comment ctermfg=Green" >> /etc/vimrc
- sudo echo "highlight Constant ctermfg=Red" >> /etc/vimrc
- sudo echo "highlight Identifier ctermfg=Cyan" >> /etc/vimrc
- sudo echo "highlight Statement ctermfg=Yellow" >> /etc/vimrc
- sudo echo "highlight Title ctermfg=Magenta" >> /etc/vimrc
- sudo echo "highlight Special ctermfg=Magenta" >> /etc/vimrc
- sudo echo "highlight PreProc ctermfg=Magenta" >> /etc/vimrc
- sudo echo " " >> /etc/vimrc
- sudo echo "set tabstop=2" >> /etc/vimrc
- sudo echo "set shiftwidth=2" >> /etc/vimrc

#### ゲストOSの再起動
- sudo shutdown -r now

### Libre Officeのインストール
- sudo apt-get install -y libreoffice libreoffice-l10n-ja libreoffice-help-ja
- sudo apt-get install -y libreoffice-gtk

### セーブポイント作成
- extit
- vagrant snapshot save savepoint_001
- vagrant snapshot list
    - savepoint_001 が表示されれば保存成功
- vagrant ssh

### Apache/PHPのインストールと設定
- ApacheはPHPインストール時に自動的にインストールされる
- sudo apt-get install -y php
- sudo apt-get install -y php-mbstring php-xml php-xmlrpc php-gd php-pdo php-mysqlnd php-json php-pgsql php-mbstring php-intl php-zip php-phar php-ctype php-curl php-fileinfo
- PHP/Apacheのインストールを確認。バージョン情報が表示されれば成功。
    - php -v
    - apache2 -v
- Apache2の自動起動を設定
    - sudo systemctl restart apache2
    - sudo systemctl enable apache2
- 仮想デスクトップ内でブラウザ(インストール済みのFirefox)を開き動作確認
    - http://localhost
    - Apacheが表示されれば成功

### セーブポイント作成
- exit
- vagrant snapshot save savepoint_002
- vagrant snapshot list
    - savepoint_002 が表示されれば保存成功
- vagrant ssh

### PHP/Apaceh初期設定
- sudo touch /var/www/html/info.php
- sudo chmod 766 /var/www/html/info.php
- sudo echo '<?php phpinfo(); ?>' > /var/www/html/info.php
- sudo systemctl restart apache2
- ブラウザで動作確認
    - 127.0.0.1/info.php
    - PHPの情報ページが表示されれば成功
- sudo rm /var/www/html/info.php
    - ファイル削除

### セーブポイント作成
- exit
- vagrant snapshot save savepoint_003
- vagrant snapshot list
    - savepoint_003 が表示されれば保存成功
- vagrant ssh

### MariaDBの設定
- sudo apt-get install -y mariadb-server
- sudo vim /etc/mysql/mariadb.conf.d/50-server.cnf
```cnf
[mysqld] セクション内に追記
character-set-server=utf8
```
- sudo systemctl enable mariadb
- sudo systemctl start mariadb
- sudo mysql -uroot -p
    - password:vagrant
    - MariaDB> create user wordpress identified by 'wordpresspasswd';
    - MariaDB> create database wordpress;
    - MariaDB> grant all privileges on wordpress.* TO 'wordpress';
    - MariaDB> flush privileges;
    - MariaDB> exit;
- sudo mysql -u wordpress -p
    - password：wordpresspasswd
    - MariaDB> show databases;
    - テーブル「wordpress」が表示されれば成功
- exit

### セーブポイント作成
- extit
- vagrant snapshot save savepoint_004
- vagrant snapshot list
    - savepoint_004 が表示されれば保存成功
- vagrant ssh

### WordPress4.9のインストール
- sudo wget https://ja.wordpress.org/wordpress-4.9-ja.zip
- sudo unzip wordpress-4.9-ja.zip
- sudo chmod 775 -R /var/www/
- sudo cp -r ~/wordpress/. /var/www/html/wordpress
- sudo chown -R www-data:www-data /var/www/*
- sudo systemctl restart apache2
- ブラウザで動作確認
    - http://127.0.0.1/wordpress/index.php
    - http://127.0.0.1/wordpress/wp-admin/setup-config.php に遷移しwordpressの設定画面が表示されれば成功

<!---
- sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.org 
- sudo vim /etc/httpd/conf/httpd.conf
```cnf
DocumentRoot "/var/www/wordpress"
<Directory "/var/www/wordpress">
    AllowOverride All
</Directory>
```
--->

### セーブポイント作成
- extit
- vagrant snapshot save savepoint_005
- vagrant snapshot list
    - savepoint_005 が表示されれば保存成功
- vagrant ssh

### WordPressの設定
- ブラウザでアクセス
    - http://127.0.0.1/wordpress/index.php
- データベースの設定
    - データベース名：wordpress
    - ユーザー名：wordpress
    - パスワード：wordpresspasswd
    - データベースのホスト名：localhost
- ブログの設定
	- タイトル：wordpress the world ブログ(仮)
	- 管理者ユーザ名：wordpress_2021
	- パスワード：wordpress_2021
	- メールアドレス：test@test.jp
	- 検索エンジンがこのサイトをインデックスしないようにする：有効

### 設定後の挙動
ゲスト環境のブラウザから以下の通り接続することができるようになる。

- 管理画面：http://127.0.0.1/wordpress/wp-admin
- ユーザ画面：http://1276.0.0.1/wordpress/


### セーブポイント作成
- extit
- vagrant snapshot save savepoint_006
- vagrant snapshot list
    - savepoint_006 が表示されれば保存成功
- vagrant ssh

### EC-CUBEインストール後の設定
#### Firefoxの設定
以下をブックマークに追加
- 管理画面：http://127.0.0.1/wordpress/wp-admin
- ユーザ画面：http://1276.0.0.1/wordpress/

#### 自動アップデートの停止
演習環境が変更されることを防ぐため自動アップデートのポップアップ表示を停止(GUI上で手動の操作)。

### パッケージ出力
とりあえずここでBOXファイルを出力してVagrant BOXにアップ
動作確認はまた後日。

### 参考ページ
以下のページを参考に手順を組み立てた。  
- Server World Apache2 : インストール
    - https://www.server-world.info/query?os=Ubuntu_19.04&p=httpd&f=1
- Server World Apache2 : PHPスクリプトを利用する
    - https://www.server-world.info/query?os=Ubuntu_19.04&p=httpd&f=3
- Server World Mariadb
    - https://www.server-world.info/query?os=Ubuntu_19.04&p=mariadb&f=1
- sedでこういう時はどう書く?
    - https://qiita.com/hirohiro77/items/7fe2f68781c41777e507
- テキストの置換処理を得意とするスクリプト言語 sed
    - https://bi.biopapyrus.jp/os/linux/sed.html
- Ubuntu 18.04 LTS に WordPress 5.3 をインストール
    - https://qiita.com/cherubim1111/items/265cfbbe91adb44562d5