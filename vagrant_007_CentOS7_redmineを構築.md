# CentOS7にRedmineバージョン4.0をインストール

## 事前の準備
### 検証環境
- 仮想化ツール1：Vagrant 2.1.2
- 仮想化ツール2：Oracle VirtualBox 5.2.22
- ゲストOS：Cent OS 7.5 (bento/centos-7.5 v201811.25.0)
    - https://app.vagrantup.com/bento/boxes/centos-7.5
		- ツール1：Apache 2.4.6
		- ツール2：MariaDV 5.5.60
		- ツール3：Redmine 4.0.0
		- ツール：Rails 5.2.2

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
- sudo yum install -y vim tree git
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
```
- sudo timedatectl set-timezone Asia/Tokyo
- timedatectl status

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
- CentOS7 Redmineのインストール
	- https://www.unix-power.net/networking/post-728
- Ruby 2.5 インストール
	- https://www.server-world.info/query?os=CentOS_7&p=ruby25
- bundleコマンドのパスを通す方法
	- https://qiita.com/shyamahira/items/6c986eb3e21471b35af2

### 必要なツールのインストール
- sudo yum groupinstall -y "Development Tools"
- sudo yum install -y openssl-devel readline-devel zlib-devel curl-devel libyaml-devel libffi-devel
- sudo yum install -y mariadb-server mariadb-devel mariadb
- sudo yum install -y httpd httpd-devel
- sudo yum install -y ImageMagick ImageMagick-devel ipa-pgothic-fonts 

### Rubyのインストール
- sudo yum install -y yum-plugin-priorities
- sudo sed -i -e "s/\]$/\]\npriority=1/g" /etc/yum.repos.d/CentOS-Base.repo
- sudo yum install -y centos-release-scl-rh centos-release-scl
- sudo yum --enablerepo=centos-sclo-rh -y install rh-ruby24 rh-ruby24-ruby-devel
- sudo scl enable rh-ruby24 bash
- ruby -v
    - バージョン表示でインストールを確認
- sudo vim /etc/profile.d/rh-ruby24.sh
```sh
#!/bin/bash

source /opt/rh/rh-ruby24/enable
export X_SCLS="`scl enable rh-ruby24 'echo $X_SCLS'`"
```
- gem install bundler --no-rdoc --no-ri

### スナップショット2
- vagrant halt
- vagrant snapshot save savepoint2
- vagrant snapshot list
- vagrant list

### MariaDBの設定
- sudo vim /etc/my.conf
```
[mysqld]のセクションに下記を追記
character-set-server=utf8

```
- sudo systemctl enable mariadb.service
- sudo systemctl start mariadb.service
- mysql_secure_installation
	- パスワードを「redmine_2018」に設定、それ以外はすべて「y」(yesの意味)で設定
- mysql -u root -p
	- Password: redmine_2018
	- create database redmine default character set utf8;
	- grant all on redmine.* to redmine@localhost identified by 'redmine_passwd';
	- exit;

### Redmineのインストール
- sudo wget http://www.redmine.org/releases/redmine-4.0.0.tar.gz
	- ファイル名はバージョン番号によって変わる。最新版は下記URLから取得。
	- http://www.redmine.org/projects/redmine/wiki/Download
- tar zxvf redmine-4.0.0.tar.gz
- sudo mv redmine-4.0.0 /var/lib/redmine.
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

### Apache初期設定：ここで詰まっていると思われる(Rubyアプリケーションが起動しない)
- sudo vim /etc/httpd/conf.d/redmine.conf
```conf
# Redmineの画像ファイル・CSSファイル等へのアクセスを許可する設定。
# Apache 2.4のデフォルトではサーバ上の全ファイルへのアクセスが禁止されている。
<Directory "/var/lib/redmine/public">
  Require all granted
</Directory>

# Passengerの基本設定。
# passenger-install-apache2-module --snippet で表示された設定を記述。
# 環境によって設定値が異なるため以下の5行はそのまま転記せず、必ず passenger-install-apache2-module --snippet で表示されたものを使用すること。
LoadModule passenger_module /usr/local/lib/ruby/gems/2.4.0/gems/passenger-5.1.5/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
  PassengerRoot /usr/local/lib/ruby/gems/2.4.0/gems/passenger-5.1.5
	PassengerDefaultRuby /usr/local/bin/ruby
</IfModule>

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

