# Lubuntu20.04にEC-Cubeの検証環境を構築
## 検証環境
- 仮想化ツール1：Vagrant 2.2.7
- 仮想化ツール2：Oracle VirtualBox 6.1.12
- ゲストOS： Lubuntu 18.04 (bento/ubuntu-18.04)
https://app.vagrantup.com/bento/boxes/ubuntu-18.04
- Vagrantプラグイン：vagrant-vbguest
- ツール1：Apache 2.4.41
- ツール2：PHP 7.4.3
- ツール3：EC-Cube 4.1.0
- ツール4：Selenium IDE
- ツール5：Selenium (Python)
- ツール6：Selenium WebDriver (Google Chrome)

構築済みのイメージをVagrant Cloudの以下に保存した。
  
- ユーザー：a1852rw
- イメージ名：Lubuntu-18.04_Selenium
- URL：[aiit-alpha-team/Lubuntu-18.04_Selenium](aiit-alpha-team/Lubuntu-18.04_Selenium)

また、当ファイルに記載の方法で作成しVagrant CloudにアップしたBOXファイルを使用するためのVagrantfileを以下に保存した。

- ディレクトリ名：作成中
- ファイル名：Vagrantfile
- URL：[https://github.com/a1852rw/005_vagrantfile/XXXX作成中](https://github.com/a1852rw/005_vagrantfile/XXXX作成中)

## コマンドメモ
### 環境構築
- vagrant init bento/ununtu-18.04
- vagrant up

元にしたイメージは「bento/ununtu-18.04」だが、実際には以下「初期設定」に記載の手順により初期設定完了している以下イメージを使用することが可能。  

- aiit-alpha-team/Lubuntu-18.04_Desktop
    - URL：[https://app.vagrantup.com/aiit-alpha-team/boxes/Lubuntu-18.04_Desktop](https://app.vagrantup.com/aiit-alpha-team/boxes/Lubuntu-18.04_Desktop)

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

## 自動テスト演習環境の構築
ここまで設定したLubutu18.04上のEC-Cube環境を進化させ、自動テスト演習環境にするため追加の手順を記載する。  
自動テストツールとしてSelenium IDEおよび作業用のエディタ等、動作を補助するツールをインストールした。

### Google Chromeのインストール
- FireFoxでGoogle Chrome配布サイトへアクセスし、インストールプログラムをダウンロードする
    - https://www.google.com/intl/ja_jp/chrome/
    - インストール後に以下コマンドを実行しパッケージからインストールを実施する
- sudo dpkg -i /tmp/mozilla_vagrant0/google-chrome-stable_current_amd64.deb
- sudo shutdown -r now
    - インストール直後は動作が遅いので一度再起動する。再起動すると動作が正常化する。
- GUI操作により以下をブックマークとホームページに追加する。これによりGoogle Chrome起動後すぐに演習を開始できるようになる。
    - 管理画面：http://127.0.0.1/index.php/adminconsole/
    - ユーザ画面：http://127.0.0.1/index.php/
    - Selenium逆引きサイト：https://www.seleniumqref.com/


### Selenium IDEの導入
- 演習環境内でGoogle Chromeを起動しSelenium IDEのストアページを開く
    - ページタイトル：Selenium IDE
    - URL：https://chrome.google.com/webstore/detail/selenium-ide/mooikfkahbdckldjjndioackbalphokd?utm_source=chrome-ntp-icon
- 「Chromeに追加」ボタンをクリックする。確認画面が表示される。
- 「拡張機能を追加」ボタンをクリックする。Selenium IDEのアドオンのインストールが開始される。
- Google Chrome右上に「Selenium IDE」ボタンが追加される。
- 「Selenium IDE」ボタンをクリックするとSelenium IDEが起動し、テストの記録/自動実行ができるようになる。

### Visual Studio Codeの導入
すでにFeatherPadが導入されているため不要と思われるが、テキストエディタのデファクトスタンダードであるため導入する。  
Seleniumのスクリプトを編集するため使用する。  

- sudo wget -O /home/vagrant/ダウンロード/vscode.deb https://go.microsoft.com/fwlink/?LinkID=760868
- sudo apt install -y /home/vagrant/ダウンロード/vscode.deb
- デスクトップ画面左下の「メニュー」ボタンから「アクセサリ」を選択し、「Visual Studio Code」をクリックする
- Visual Studio Codeが起動する
    - 日本語化プラグイン(プラグイン検索画面で「Japanese」で検索)をインストールする
        - https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja
    - Pythonプラグイン(プラグイン検索画面で「Python」で検索)をインストールする
        -  https://marketplace.visualstudio.com/items?itemName=ms-python.python

### Selenium WebDriverの導入
- sudo mkdir /home/vagrant/selenium_script
- sudo chown -hR  vagrant:vagrant /home/vagrant/selenium_script
- 以下ファイルを作成し、ディレクトリ「/home/vagrant/selenium_script」に保存する

```python (test_001.py)
from selenium import webdriver
 
#ChromeDriverのパスを引数に指定しChromeを起動
driver = webdriver.Chrome("/home/vagrant/selenium_script/chromedriver")
#指定したURLに遷移
driver.get("https://www.google.co.jp")
driver.quit()
```

```python (test_002.py)
#指定したURLに遷移
driver.get("https://www.google.co.jp")

element = driver.find_element_by_link_text("画像")
#画像のリンクをクリック
element.click()
driver.quit()
```

以下コマンドにより、WebDriverをインストールする。

- python3 --version
    - まずはPythnがインストールされていることを確認する。「Python 3.x」と表示されれば成功
- pip3 --version
    - 次にPIP3がインストールされていることを確認する。「pop X.XX」と表示されれば成功
- sudo apt-get install -y python3-selenium
- pip3 show selenium
    - インストールに成功していればSeleniumのバージョンが表示される
- google-chrome --version
    - GoogleChromeのバージョンを確認する
- GoogleChromeのバージョンにあったWebDriverを以下ページで確認し、コマンドに埋め込んでダウンロードする。
    - 参照ページ：https://chromedriver.chromium.org/downloads
    - wget -a /home/vagrant/selenium_script/chromedriver.zip [ここにURLを記載] 
    - 下記コマンドは、ページよりバージョン96のダウンロードURLを確認し、作成したもの。
        - wget -O /home/vagrant/selenium_script/chromedriver.zip https://chromedriver.storage.googleapis.com/96.0.4664.35/chromedriver_linux64.zip
        - sudo unzip -d /home/vagrant/selenium_script/ /home/vagrant/selenium_script/chromedriver.zip 
 
以下コマンドにより動作確認を行う

- python3 /home/vagrant/selenium_script/test_001.py
- python3 /home/vagrant/selenium_script/test_002.py

Google Chromeが動作すれば設定完了。


### パッケージ出力
追加手順を行ったBOXファイルを出力してVagrant Cloudにアップロードする。

- vagrant package

出力されたBOXファイルをWeb上の以下ページから再度アップロード。

- Vagrant Cloud
    - https://app.vagrantup.com/


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
- lubuntu 20.04 で自動ログインする方法　[PC]
    - https://staka.blog.ss-blog.jp/2021-02-26
- linuxBean ： Firefoxで日本語入力ができなくなった
    - https://ameblo.jp/karakurenainimizu/entry-12138877583.html
- Selenium IDE公式ページ：Get-Started
    - https://www.selenium.dev/selenium-ide/docs/en/introduction/getting-started
- Debian and Ubuntu based distributions
    - https://code.visualstudio.com/docs/setup/linux#_debian-and-ubuntu-based-distributions