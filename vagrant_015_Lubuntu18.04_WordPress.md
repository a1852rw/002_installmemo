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
- sudo vim /etc/my.cnf
```cnf
[mysqld] セクション内に追記
character-set-server=utf8
```
- sudo systemctl enable mariadb
- sudo systemctl restart mariadb
- mysql -uroot -p
    - password:vagrant
    - MariaDB> create user wordpress identified by 'wordpresspasswd';
    - MariaDB> create database wordpress;
    - MariaDB> grant all privileges on wordpress.* TO 'wordpress';
    - MariaDB> flush privileges;
    - MariaDB> exit;
- mysql -u ecuser -p
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

### WordPressのインストール


<!---
### EC-Cubeのインストール
- sudo wget http://downloads.ec-cube.net/src/eccube-4.0.4.zip
- sudo unzip eccube-4.0.4.zip
- sudo chmod 775 -R /var/www/
- sudo cp -r ~/eccube-4.0.4/. /var/www/html/
- sudo chown -R www-data:www-data /var/www/*
- sudo systemctl restart apache2
- ブラウザで動作確認
    - 127.0.0.1/index.php
    - EC-CUBEのインストール画面が表示されれば成功
--->

### セーブポイント作成
- extit
- vagrant snapshot save savepoint_005
- vagrant snapshot list
    - savepoint_005 が表示されれば保存成功
- vagrant ssh

### ブラウザ操作によるEC-CUbeインストール
- P1 ようこそ
    - 「次へ進む」ボタンをクリック
- P2 権限ページ
    - 「次へ進む」ボタンをクリック
- P2 サイトの設定
    - あなたの店名：ECテスト演習店舗
    - メールアドレス：test@test.jp
    - 管理画面ログインID：admin
    - 管理画面パスワード：adminpasswd
    - 管理画面のディレクトリ名：adminconsole
    - それ以外の項目は操作しない
    - 「次へ進む」ボタンをクリック
- P3 データベースの設定
    - データベースの種類：MySQL
    - データベースのホスト名：localhost
    - データベースのポート番号：空欄
    - データベース名：ecdata
    - ユーザ名：ecuser
    - パスワード：ec-cube
- P4 データベースの初期化
    - 「次へ進む」ボタンをクリック
- P5 インストール完了
    - 「管理画面を表示」ボタンをクリック
- ログイン画面
    - ログインID：admin
    - パスワード：adminpasswd
- EC-CUBEの管理画面が表示されれば成功

### 設定後の挙動
ゲスト環境のブラウザから以下の通り接続することができるようになる。

- 管理画面：http://127.0.0.1/index.php/adminconsole/
- ユーザ画面：http://127.0.0.1/index.php/


### セーブポイント作成
- extit
- vagrant snapshot save savepoint_006
- vagrant snapshot list
    - savepoint_006 が表示されれば保存成功
- vagrant ssh

### EC-CUBEインストール後の設定
#### Firefoxの設定
以下をブックマークに追加
- 管理画面：http://127.0.0.1/index.php/adminconsole/
- ユーザ画面：http://127.0.0.1/index.php/

#### 自動アップデートの停止
演習環境が変更されることを防ぐため自動アップデートのポップアップ表示を停止(GUI上で手動の操作)。

### パッケージ出力
とりあえずここでBOXファイルを出力してVagrant BOXにアップ

なぜかメモ帳以外は日本語入力で紀伊問題があったり、構築手順が怪しかったりするが現時点では目をつぶる。

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

<!---
EC-CUBEのインストール手順はいずれも実行するとエラーが発生し最後まで進めることができない。実際にコマンドを入力しての動作確認を怠っていると思われる。  
  
- Server World 初期設定 : リポジトリを追加する2019
    - [https://www.server-world.info/query?os=CentOS_8&p=initial_conf&f=7](https://www.server-world.info/query?os=CentOS_8&p=initial_conf&f=7)
- Server World Apache httpd : PHP スクリプトを利用する
    - [https://www.server-world.info/query?os=CentOS_8&p=httpd&f=6](https://www.server-world.info/query?os=CentOS_8&p=httpd&f=6)
- CentOS 8にEC-CUBE 4をインストールした時の自分用メモ
    - [https://qiita.com/okazy/items/6069f58345c0c42de439](https://qiita.com/okazy/items/6069f58345c0c42de439)
- CentOS7にEC-Cube3をインストール（yumのみ）
    - [https://labo-study.com/2016/01/26/centos7%E3%81%ABec-cube3%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%EF%BC%88yum%E3%81%AE%E3%81%BF%EF%BC%89/](https://labo-study.com/2016/01/26/centos7%E3%81%ABec-cube3%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%EF%BC%88yum%E3%81%AE%E3%81%BF%EF%BC%89/)
- EC-CUBE3　超簡単インストールのご紹介 Linux
    - [https://sys-guard.com/post-6825/](https://sys-guard.com/post-6825/)
- 【初心者向け】EC-CUBE のインストール方法、失敗しない手順を図で解説します
    - [https://www.yamatofinancial.jp/learning/pre-opening/how-to-install-ec-cube.html](https://www.yamatofinancial.jp/learning/pre-opening/how-to-install-ec-cube.html)
- さくらVPS(CentOS 7)にEC-CUBE 3.0をインストールする
    - [https://risa-webstore.com/blog/?p=71](https://risa-webstore.com/blog/?p=71)
- Vimの置換コマンドまとめ
    - [https://qiita.com/lightning5x5/items/e5162cb3e4b6d38b439d](https://qiita.com/lightning5x5/items/e5162cb3e4b6d38b439d)
--->
