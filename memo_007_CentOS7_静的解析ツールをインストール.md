# 作業メモ：CentOS7に静的解析ツールをインストール
- 内容
  - Ratsをインストールする
  - Flaw Finderをインストールする

## Ratsのインストール
### expatをインストールする
- expatはRatsインストールの前提となる
- cd ~
- wget https://sourceforge.net/projects/expat/files/expat/2.2.6/expat-2.2.6.tar.bz2
- tar xf expat-2.2.6.tar.bz2
- cd expat-2.2.6/
- ./configure
- make 
- sudo make install

### Ratsをインストールする
- cd ~
- wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/rough-auditing-tool-for-security/rats-2.4.tgz
- tar xf rats-2.4.tgz 
- cd rats-2.4/
- ./configure
- make
- sudo make install
- rats -h
  - バージョン情報が表示されれれば成功

### 使い方
- rats XXXXX(対象のコード)
- 例
  - rats unbounded-copy-by-gets.c
- 実行後に画面上に解析結果が表示される

## FlawFinderのインストール
### インストール手順
- cd ~
- wget https://dwheeler.com/flawfinder/flawfinder-2.0.7.tar.gz
- tar xvzf flawfinder-*.tar.gz 
- cd flawfinder-2.0.7
- sudo make prefix=/usr install

### 使い方
- flawfinder XXXXX(対象のコード)
- 例
  - flawfinder unbounded-copy-by-gets.c 
- 実行後に画面上に解析結果が表示される

## FlawFinderの解析結果をブラウザで確認する
- Apache2をインストールする
- (Vagrantを使っているという想定で)ポートフォワーディングを設定する
- flawfinderで解析結果をHTML形式で出力する

### Apacheのインストールと設定
- cd ~
- sudo yum install -y httpd
- httpd -v
    - バージョンが表示されApacheがインストールされたことを確認
- sudo systemctl start httpd
- sudo chkconfig httpd on

### ポートフォワーディングの設定
- Vagrantfileを編集(メモ帳などで変種してかまわない)
```
#config.vm.network "forwarded_port", guest: 80, host: 8080
コメントアウトを解除し上書き保存
```
- Vagrant reload で再起動し ブラウザから localhost:8080 にアクセス
- ページにアクセスできることを確認

### 使い方
- sudo touch /var/www/html/index.html
    - sudo chmod 777 /var/www/html/index.html
    - 出力先ファイルを作成して利用可能に設定
- flawfinder --html XXXX(対象のコード) > /var/www/html/index.html
    - 例：flawfinder --html ~/source/lecture_004/unbounded-copy-by-gets.c > /var/www/html/index.html
- ブラウザから localhost:8080 にアクセス
    - 解析結果をブラウザで確認する
    - ターミナル画面よりは読みやすい
    - ブラウザの機能をつかって結果をPDF出力したりHTMLで保存することもできる

## 備考
- この設定を行った仮想イメージをVagrantファイルのBoxとして保存した
    - Windows環境でVagrantを起動しChromeで動作を確認した
- Box名： aiit-alpha-team/CentOS-7.5_2018_SecurePrograming 
- https://app.vagrantup.com/aiit-alpha-team/boxes/CentOS-7.5_2018_SecurePrograming

