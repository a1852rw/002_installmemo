# CentOS7にRedmineバージョン4.0をインストール

## 事前の準備
### 検証環境
- 仮想化ツール1：Vagrant 2.1.2
- 仮想化ツール2：Oracle VirtualBox 5.2.22
- ゲストOS：Cent OS 7.5 (bento/centos-7.5 v201811.25.0)
    - https://app.vagrantup.com/bento/boxes/centos-7.5
		- ツール1：Apache 2.4.6
		- ツール2：MySQL
		- ツール3：Redmine 3.4
		- ツール：Rails 4.2

### Vagrantの初期設定
- vagrant init bento/centos-7.5
- Vagrantfileを編集(メモ帳などで直接書き換える)
```txt
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
```

### CentOS7の初期設定
- sudo yum -y update
- uname -a
- cat /etc/redhat-release
    - 更新後のOSバージョンを確認
- sudo yum -y install vim tree git
- sudo vim /etc/vimrc
```text
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
sudo timedatectl set-timezone Asia/Tokyo
timedatectl status
```
### スナップショット1
- vagrant halt
- vagrant snapshot save savepoint1
- vagrant snapshot list
- vagrant up

## インストール作業
### 参考ページ
- Redmineのインストール
    - http://guide.redmine.jp/RedmineInstall/
- Redmine 3.4をCentOS 7.3にインストールする手順
    - http://blog.redmine.jp/articles/3_4/install/centos/

### 必要なツールのインストール
- sudo yum groupinstall -y "Development Tools"
- sudo yum install -y openssl-devel readline-devel zlib-devel curl-devel libyaml-devel libffi-devel
- sudo yum install -y postgresql-server postgresql-devel
- sudo yum install -y httpd httpd-devel
- sudo yum install -y ImageMagick ImageMagick-devel ipa-pgothic-fonts 

### Ruvyのインストール
- sudo wget https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.1.tar.gz
- sudo tar xvf ruby-2.4.1.tar.gz
- cd ruby-2.4.1
- sudo ./configure --disable-install-doc
- sudo make
- sudo make install
- cd ~
- ruby -v
    - バージョン表示でインストールを確認
		- gem install bundler --no-rdoc --no-ri

### Postgre SQLの設定
- sudo postgresql-setup initdb
- sudo vim  /var/lib/pgsql/data/pg_hba.conf
```conf
# Put your actual configuration here
# ----------------------------------
#
# If you want to allow non-local connections, you need to add more
# "host" records.  In that case you will also need to make PostgreSQL
# listen on a non-local interface via the listen_addresses
# configuration parameter, or via the -i or -h command line switches.
host    redmine         redmine         127.0.0.1/32            md5
host    redmine         redmine         ::1/128                 md5
```
- sudo systemctl start postgresql.service
- systemctl enable postgresql.service
- cd /var/lib/pgsql
- sudo -u postgres createuser -P redmine
    - Pass Word：redmine_2018
		- これによりユーザー「redmine」とパスワード「redmine_2018」が設定される
- sudo -u postgres createdb -E UTF-8 -l ja_JP.UTF-8 -O redmine -T template0 redmine
- cd ~

### Redmineのインストール
- svn co https://svn.redmine.org/redmine/branches/3.4-stable /var/lib/redmine
    - インストール先のディレクトリは任意に指定が可能
- sudo vim /var/lib/redmine/config/database.yml
```yml
production:
adapter: postgresql
database: redmine
host: localhost
username: redmine
password: redmine_2018
encoding: utf8
``` 
- sudo vim /var/lib/redmine/config/configuration.yml
```yml
production:
email_delivery:
delivery_method: :smtp
smtp_settings:
address: "localhost"
port: 25
domain: "example.com"

rmagick_font_path: /usr/share/fonts/ipa-pgothic/ipagp.ttf
```
- cd /var/lib/redmine
- bundle install --without development test --path vendor/bundle

### Redmine初期設定
- bundle exec rake generate_secret_token
- RAILS_ENV=production bundle exec rake db:migrate
- RAILS_ENV=production REDMINE_LANG=ja bundle exec rake redmine:load_default_data

### Passengerインストール
- gem install passenger -v 5.1.12 --no-rdoc --no-ri
- passenger-install-apache2-module --auto --languages ruby
- passenger-install-apache2-module --snippet

### Apache初期設定
- sudo vim /etc/httpd/conf.d/redmine.conf
```conf
# Redmineの画像ファイル・CSSファイル等へのアクセスを許可する設定。
# Apache 2.4のデフォルトではサーバ上の全ファイルへのアクセスが禁止されている。
<Directory "/var/lib/redmine/public">
  Require all granted
	</Directory>

# Passengerの基本設定。
# passenger-install-apache2-module --snippet で表示された設定を記述。
# 環境によって設定値が異なるため以下の5行はそのまま転記せず、必ず
# passenger-install-apache2-module --snippet で表示されたものを使用すること。
#
LoadModule passenger_module /usr/local/lib/ruby/gems/2.4.0/gems/passenger-5.1.5/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
  PassengerRoot /usr/local/lib/ruby/gems/2.4.0/gems/passenger-5.1.5
	PassengerDefaultRuby /usr/local/bin/ruby
</IfModule>

# 必要に応じてPassengerのチューニングのための設定を追加（任意）。
# 詳しくはPhusion Passenger users guide(https://www.phusionpassenger.com/library/config/apache/reference/)参照。
PassengerMaxPoolSize 20
PassengerMaxInstancesPerApp 4
PassengerPoolIdleTime 864000
PassengerStatThrottleRate 10

Header always unset "X-Powered-By"
Header always unset "X-Runtime"
```
- sudo systemctl start httpd.service
- sudo systemctl enable httpd.service

### ApacheでRedmineを動作させる設定
- sudo chown -R apache:apache /var/lib/redmine
- sudo vim /etc/httpd/conf/httpd.conf 
```conf
DocumentRoot "/var/www/html"
↓
DocumentRoot "/var/lib/redmine/public"
```
- sudo service httpd configtest
- sudo systemctl restart httpd.service

### ブラウザでの操作
- http://localhost:2080

