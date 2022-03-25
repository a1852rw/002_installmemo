# Lubuntu20.04にEC-Cubeのセキュリティ演習環境を構築
## 検証環境
- 仮想化ツール1：Vagrant 2.2.7
- 仮想化ツール2：Oracle VirtualBox 6.1.12
- ゲストOS： Lubuntu 20.04 (bento/ubuntu-20.04)
https://app.vagrantup.com/bento/boxes/ubuntu-20.04
- Vagrantプラグイン：vagrant-vbguest
- ツール1：Apache v2.4.41
- ツール2：PHP v7.4.3
- ツール3：EC-Cube v4.1.0
- ツール4：OWASP ZAP v2.11.1
- ツール5：Maria DB v15.1
- ツール6：JAVA v11.14
- ツール7：Burp Suite Community Edition v2022.2.4

## 環境構築演習
上記環境を構築する。  
仮想デスクトップを起動しターミナルから以下操作を順に実施する。

### ターミナルの起動
- 「システムツール」内「QTerminal」をクリックする。ターミナルが起動する。

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
    - http://127.0.0.1/index.php
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

### PHP my adminのインストール
- sudo apt-get install -y phpmyadmin
- Apacheを選択
    - 「はい」を選択
- 「了解」を選択
    - パスワード：phpmyadmin_test
    - パスワードの確認：phpmyadmin_test
- Apache側の設定
    - sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
    - sudo a2enconf phpmyadmin.conf
    - sudo systemctl restart apache2

- 動作確認
    - http://127.0.0.1/phpmyadmin
    - 以下アカウントでログインし、データベース「ecdata」が表示されることを確認する
        - ID：ecuser
        - パスワード：ec-cube

    - 以下のphpmyadmin用のアカウントでもログイン可(EC-Cubeのデータは見れない)
        - ID：phpmyadmin
        - パスワード：phpmyadmin_test


### OWASP ZAPによる演習環境の構築
#### JREのダウンロードとインスト―ル
sudo add-apt-repository -y ppa:openjdk-r/ppa && sudo apt-get update
sudo apt-get install -y openjdk-11-jre

#### OWASP ZAPのダウンロードとインスト―ル
sudo wget https://github.com/zaproxy/zaproxy/releases/download/v2.11.1/ZAP_2_11_1_unix.sh -P /home/vagrant/Downloads
sudo chmod +x /home/vagrant/Downloads/ZAP_2_11_1_unix.sh
sudo /home/vagrant/Downloads/ZAP_2_11_1_unix.sh


### Burp Suite Commyunity Editionのダウンロードとインストール
- Firefoxを起動し以下のURLを開き、インストールプログラムをダウンロードする。
    - https://portswigger.net/burp/communitydownload
    - Firefoxでリンク「Go straight to downloads」をクリック
    - 遷移したページで以下を選択し「Download」ボタンをクリック
        - Burp Suite Commyunity Edition
        - Linux(64-Bit)
- 仮想デスクトップ上でターミナルを起動し以下コマンドを実行
    - sudo chmod +x /home/vagrant/Downloads/burpsuite_community_linux_v*.sh
    - sudo /home/vagrant/Downloads/burpsuite_community_linux_v*.sh
    - 「Yes」もしくは「Enter」を連打する
- スタートメニューから以下操作を実施して初期設定
    - 「その他」内「Burp Suite Community Edition」をクリックする。初期設定画面が表示される。
    - ライセンス画面右下「I Accept」ボタンをクリックする。プロジェクト設定が画面が表示される。
    - 設定を変更せず画面の右下「Next」ボタンをクリックする。設定ファイルの選択画面が表示される。
    - 画面の右下「StartBurp」ボタンをクリックする。初期設定が行われBurp Suiteのダッシュボードが表示される。
- プロキシ設定を実施
    - ダッシュボードのメニューバー上から二列目「Proxy」をクリックする。メニューバー上から三列目にプロキシに関連するメニューが表示される。
    - ダッシュボードのメニューバー上から三列目「Options」をクリックする。詳細設定が表示される。
    - 以下のように設定を書き換える。
        - 変更前：127.0.0.1:8080
        - 変更後：127.0.0.1:18080


### Firefoxの設定変更(手作業)
#### OWASP ZAPの証明書をインポート
FireFoxにOWAP ZAPで生成した証明書をインポートする

- OWASP ZAPで証明書を生成
    - 「その他」内「OWASP ZAP」をクリックする。OWASP ZAPが起動する。
    - 画面上側メニューバー「ツール」内「オプション」をクリックする。オプション画面が表示される。
    - 画面右側のリストより「ダイナミックSSL証明書」をクリックする。画面右側にSSL証明書の生成画面が表示される。
    - 「生成」ボタンをクリックする。画面右下「保存」ボタンをクリックする。証明書データが出力される。以下要領で保存する。
        - ファイル名：owasp_zap_root_ca.cer
        - 保存先：Desktop等わかりやすい場所を選択

- Firefoxに証明書をインポート
    - Firefoxを起動する。
    - 画面右上「メニュー」ボタン内「設定」をクリックする。設定画面が表示される。
    - 画面左側「プライバシーとセキュリティ」をクリックする。画面右側にブラウザープライバシーの設定画面が表示される。
    - 画面下側にスクロールし「証明書」欄の「証明書を表示」ボタンをクリックする。「証明書マネージャー」が表示される。
    - 「認証局証明書」をクリックする。認証局証明書の設定画面が表示される。
    - 「インポート」ボタンをクリックする。ファイル選択画面が表示される。
    - OWASP ZAPで生成したファイル「owasp_zap_root_ca.cer」を選択しインポートする。

※OWASP ZAPで生成した証明書情報は有効期限が当日のみに設定されている。必要に応じて上記手順を再度実施し更新する。

#### FoxyProxyの設定
- アドオンFoxy ProxyをFireFoxにインストール 
    - Lubuntu上でFiredoxを起動し、以下URLにアクセスする
    - https://addons.mozilla.org/ja/firefox/addon/foxyproxy-standard/
    - アドオンをダウンロードしFirefoxにインストールする
- FoxyProxyの設定
    - 以下を設定する。
    - OWASP ZAPの設定
        - 設定タイトル：OWASP ZAP:8080
        - Proxy IP address or DNS name：localhost
        - Port：8080
    - Burp Suite Commyunity Editionの設定
        - 設定タイトル：Burp Suite:18080
        - Proxy IP address or DNS name：localhost
        - Port：18080

#### その他アドオンのインストール
- Lununtu上でFireFoxを起動し、以下URLにアクセス。アドオン User-Agent Switcherをインストールする。
    - https://addons.mozilla.org/ja/firefox/addon/user-agent-switcher-revived/
    - 設定は特になし
- Lununtu上でFireFoxを起動し、以下URLにアクセス。アドオン Wappalyzerをインストールする。
    - https://addons.mozilla.org/ja/firefox/addon/wappalyzer/
    - 設定は特になし

### 参考ページ
- 【Ubuntu 20.04 LTS Server】phpMyAdminをインストール   
    - https://www.yokoweb.net/2020/08/16/ubuntu-20_04-phpmyadmin-install/
- OWASP ZAP ルート証明書を更新する
    - https://techtech-note.com/636/
- OWASP ZAPで脆弱性診断 設定編
    - https://techtech-note.com/610/
- https://ja.linux-console.net/?p=963
    - https://ja.linux-console.net/?p=963