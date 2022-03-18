# Lubuntu20.04にEC-Cubeの検証環境を構築
## 検証環境
- 仮想化ツール1：Vagrant 2.2.7
- 仮想化ツール2：Oracle VirtualBox 6.1.12
- ゲストOS： Lubuntu 20.04 (bento/ubuntu-20.04)
https://app.vagrantup.com/bento/boxes/ubuntu-20.04
- Vagrantプラグイン：vagrant-vbguest
- ツール1：Apache 2.4.41
- ツール2：PHP 7.4.3
- ツール3：EC-Cube 4.1.0
- ツール4：OWASP ZAP <バージョンは後程確認> 

構築済みのイメージをVagrant Cloudの以下に保存した。
  
- ユーザー：a1852rw
- イメージ名 <保存前>
- URL：[<保存前>](<保存前>)

また、当ファイルに記載の方法で作成しVagrant CloudにアップしたBOXファイルを使用するためのVagrantfileを以下に保存した。

- ディレクトリ名：009_Lubuntu_20.04_Desktop_full
- ファイル名：Vagrantfile
- URL：[https://github.com/a1852rw/005_vagrantfile/tree/master/008_Lubuntu_20.04_Desktop_full](https://github.com/a1852rw/005_vagrantfile/tree/master/008_Lubuntu_20.04_Desktop_full)

手順を構築中のためコメントアウト 
<!---
## コマンドメモ
### 環境構築
- vagrant init bento/ununtu-20.04
- vagrant up

元にしたイメージは「bento/ununtu-20.04」だが、実際には以下「初期設定」に記載の手順により初期設定完了している以下イメージを使用することが可能。  

- aiit-alpha-team/Lubuntu-20.04_Desktop
    - URL：[https://app.vagrantup.com/aiit-alpha-team/boxes/Lubuntu-20.04_Desktop](https://app.vagrantup.com/aiit-alpha-team/boxes/Lubuntu-20.04_Desktop)

初期設定の内容については、以下URLを参照  
- 構築テキスト作成中

URL内にて初期設定として実施している作業は以下の通り。

### 初期設定
#### ゲスト OSの更新
- sudo apt-get update -y && sudo apt-get upgrade -y

#### デスクトップ環境のインストール
- sudo yes Y | sudo apt-get install -y lubuntu-core

#### GUIモードの設定
- sudo systemctl set-default graphical.target

#### 再度のゲストOS更新
- sudo apt-get update -y

#### 日本語環境の設定
- sudo apt-get install -y fonts-ipafont fonts-ipaexfont
- sudo apt-get install -y language-pack-ja-base language-pack-ja ibus fcitx-mozc 
- sudo apt-get install -y firefox-locale-ja language-pack-gnome-ja language-pack-kde-ja
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

#### メモ
以上で初期設定は完了。
--->

### Apache/PHPのインストールと設定
- ApacheはPHPインストール時に自動的にインストールされる
- sudo apt-get install -y php
- sudo apt-get install -y libonig-dev libxml2-dev
- sudo apt-get install -y php-mbstring php-xml php-xmlrpc php-gd php-pdo php-mysqlnd php-json php-pgsql php-intl php-zip php-phar php-ctype php-curl php-fileinfo php-opcache php-pdo php-fpm php-json php-cli php-common php-mysql
- PHP/Apacheのインストールを確認。バージョン情報が表示されれば成功。
    - php -v
    - apache2 -v
- Apache2の自動起動を設定
    - sudo systemctl restart apache2
    - sudo systemctl enable apache2
- 仮想デスクトップ内でブラウザ(インストール済みのFirefox)を開き動作確認
    - http://localhost
    - Apacheが表示されれば成功

### PHP/Apaceh初期設定
- sudo touch /var/www/html/info.php
- sudo chmod 766 /var/www/html/info.php
- sudo echo '<?php phpinfo(); ?>' > /var/www/html/info.php
- sudo systemctl restart apache2
- 仮想デスクトップ内でFirefoxを起動し動作確認
    - http://127.0.0.1/info.php
    - PHPの情報ページが表示されれば成功
- sudo rm /var/www/html/info.php
    - ファイル削除

### MariaDBの設定
- sudo apt-get install -y mariadb-server
- sudo systemctl enable mariadb
- sudo systemctl start mariadb
- sudo mysql -uroot -p
    - password:vagrant
    - MariaDB> create user ecuser identified by 'ec-cube';
    - MariaDB> create database ecdata;
    - MariaDB> grant all privileges on ecdata.* TO 'ecuser';
    - MariaDB> flush privileges;
    - MariaDB> exit;
- mysql -u ecuser -p
    - password：ec-cube
    - MariaDB> show databases;
    - テーブル「ecdata」が表示されれば成功
- exit

### EC-Cubeのインストール
- sudo wget http://downloads.ec-cube.net/src/eccube-4.1.0.zip
- sudo unzip eccube-4.1.0.zip
- sudo chmod 775 -R /var/www/
- sudo cp -r ~/ec-cube/. /var/www/html/
- sudo chown -R www-data:www-data /var/www/*
- sudo systemctl restart apache2
- ブラウザで動作確認
    - 127.0.0.1/index.php
    - EC-CUBEのインストール画面が表示されれば成功

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

管理画面で以下のユーザ情報をテスト用に登録。

- ユーザ名：test@test.jp
- パスワード：test_test


### EC-CUBEインストール後の設定
#### Firefoxの設定
GUI操作により以下をブックマークに追加する。これによりFirefox起動後すぐに演習を開始できるようになる。
- 管理画面：http://127.0.0.1/index.php/adminconsole/
- ユーザ画面：http://127.0.0.1/index.php/
- Selenium逆引きサイト：https://www.seleniumqref.com/

#### 自動アップデートの停止
演習環境が変更されることを防ぐため自動アップデートのポップアップ表示を停止(GUI上で手動の操作)。

---
### OWASP ZAPによる演習環境の構築
#### JREのダウンロードとインスト―ル
sudo add-apt-repository -y ppa:openjdk-r/ppa && sudo apt-get update
sudo apt-get install -y openjdk-11-jre

#### OWASP ZAPのダウンロードとインスト―ル
sudo wget https://github.com/zaproxy/zaproxy/releases/download/v2.11.1/ZAP_2_11_1_unix.sh -P /home/vagrant/Downloads
sudo chmod +x /home/vagrant/Downloads/ZAP_2_11_1_unix.sh

### Firefoxの設定変更(手作業y
#### OWASP ZAPの証明書をインポート委
- FireFoxにOWAP ZAPで生成した証明書をインストールする
<手順は後日記述する>

#### FoxyProxyの設定
- アドオンFoxy ProxyをFireFoxにインストール 
    - Lubuntu上でFiredoxを起動し、以下URLにアクセスする
    - https://addons.mozilla.org/ja/firefox/addon/foxyproxy-standard/
    - アドオンをダウンロードしFirefoxにインストールする
- FoxyProxyの設定
    - 
<手順は後日記述する>

#### その他アドオンのインストール
- Lununtu上でFireFoxを起動し、以下URLにアクセス。アドオン User-Agent Switcherをインストールする。
    - https://addons.mozilla.org/ja/firefox/addon/user-agent-switcher-revived/
    - 設定は特になし
- Lununtu上でFireFoxを起動し、以下URLにアクセス。アドオン Wappalyzerをインストールする。
    - https://addons.mozilla.org/ja/firefox/addon/wappalyzer/
    - 設定は特になし