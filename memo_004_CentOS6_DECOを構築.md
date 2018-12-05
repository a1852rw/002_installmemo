# DECO構築メモ(CentOS6)
## 検証環境
- 仮想化ツール1：Vagrant
- 仮想化ツール2：Oracle VirtualBox
- ゲストOS：Cent OS 6.10
    - https://app.vagrantup.com/bento/boxes/centos-6.9
- ツール1：DECO 1.1.4
    - https://www.deco-project.org/deco
- ツール2：Ruby 1.9.3
- ツール3：Rails 3.2.6
- ツール4：MySQL 5.1.66
- ツール5：Passenger 3.0.17 3.0.18 3.0.19

# 手順を整理
- 手順開始前
    - boxをインストール
    - CentOSの初期設定
- savepoint1
    - mySQLの初期設定
- savepoint2
    - Ruby On Railsの初期設定
- savepoint3
    - Apache2のインストール
- savepoint4
    - passengerのインストールと初期設定(apache2も含む)
    - gembundlerのインストールと設定
- savepoint5
    - Decoのインストールと設定

## 初期設定
- sudo yum -y update
    - uname -a
    - cat /etc/redhat-release
    - 更新後のOSバージョンを確認
- sudo yum -y install vim vim-common vim-enhanced tree
- sudo vim /etc/vimrc
    - set number
    - hi Comment ctermfg=gray

## スナップショット保存：1
- exit
    - vagrant halt
    - vagrant snapshot save savepoint1
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

## mySQLのインストール
- sudo yum -y install mysql mysql-devel mysql-server
- sudo /etc/init.d/mysqld start
- sudo chkconfig mysqld on

### mySQLの初期設定
- sudo vim /etc/my.cnf の「[mysqld]」内に以下を追加する
```my.cnf
character-set-server=utf8
```
- 下記ページの手順を参考に作成
    - https://www.server-world.info/query?os=CentOS_6&p=mysql57&f=1
- mysql_secure_installation 
    - Press y|Y for Yes, any other key for No: y
    - New password: XXXXX
    - Re-enter new password: XXXX
    - Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
    - Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
    - Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
    - Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y

### mySQLのデーブル操作など
- mysql -u root -p 
    - GRANT ALL PRIVILEGES ON deco_production.* TO deco@'127.0.0.0/255.255.255.0' IDENTIFIED BY '********';
    - GRANT ALL PRIVILEGES ON deco_production.* TO deco@'localhost' IDENTIFIED BY '********';
    - create database deco_production;
    - exit

## スナップショット保存：2
- exit
    - vagrant halt
    - vagrant snapshot save savepoint2
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

## 別の手順でやりのしてみた
- sudo yum update -y
- sudo yum install -y openssl openssl-devel git gcc gcc-c++ readline readline-devel libcurl-devel zlib-devel sqlite-devel libicu-devel cmake
- 参考ページ
    - https://qiita.com/tinaba/items/067f4530742358a9dc6e
    - https://qiita.com/shinyashikis@github/items/3501c5f7f71a8e345c3d
- git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
    - echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    - echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
    - exec $SHELL -l
    - rbenv --version
- git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
    - rbenv install --list
    - rbenv install -v 2.5.1
    - rbenv global 2.5.1
    - rbenv rehash
- gem install rails --version="3.2.6"
    - rbenv rehash
    - ruby -v
    - gem -v
    - rails -v

## スナップショット保存：3
- exit
    - vagrant halt
    - vagrant snapshot save savepoint3
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh
 
## yamlのインストール
- cd /usr/local/src
- sudo wget http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
- sudo tar zxvf yaml-0.1.4.tar.gz
- cd yaml-0.1.4
- sudo ./configure
- sudo make
- sudo make install

<!---
## Ruby on Railsのインストール
- 参考ページ
    - https://qiita.com/tinaba/items/067f4530742358a9dc6e
    - https://qiita.com/shinyashikis@github/items/3501c5f7f71a8e345c3d
- sudo yum install -y openssl openssl-devel git gcc gcc-c++ readline readline-devel libcurl-devel zlib-devel sqlite-devel libicu-devel cmake
- git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
    - echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    - echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
    - exec $SHELL -l
    - rbenv --version
- git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
    - rbenv install --list
        - 表示されたものの中でから最新版を選択する(今回は 2.4.0)
    - rbenv install -v 2.4.0
    - rbenv global 2.4.0
    - rbenv rehash
- gem install rails
    - rbenv rehash
    - ruby -v
    - gem -v
    - rails -v
-->

## Calmavのインストール
- sudo rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
- sudo yum -y install clamav clamav-server clamav-server-systemd clamav-update clamav-scanner clamd clamav-devel clamav-db
- sudo vim /etc/clamd.conf
    - 207行目付近の「User clamav」を「User root」に書き換え
- sudo /etc/init.d/clamd start
- sudo chkconfig clamd on
- sudo freshclam

## Apachi2インストールと設定
- sudo yum -y install httpd httpd-devel
- sudo /etc/rc.d/init.d/httpd start
- sudo chkconfig httpd on 

## スナップショット保存：4
- exit
    - vagrant halt
    - vagrant snapshot save savepoint4
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

2018/10/18 ここまで作業した

## Passengerインストール
- gem install passenger
- rbenv rehash
- sudo chmod o+x "/home/vagrant"
- passenger-install-apache2-module
- Load Mobuleから始まる文字列をコピー
```text
LoadModule passenger_module /home/vagrant/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/passenger-5.3.5/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
    PassengerRoot /home/vagrant/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/passenger-5.3.5
    PassengerDefaultRuby /home/vagrant/.rbenv/versions/2.5.1/bin/ruby
</IfModule>
```

<!---
---
LoadModule passenger_module /home/vagrant/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/passenger-5.3.5/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
    PassengerRoot /home/vagrant/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/passenger-5.3.5
    PassengerDefaultRuby /home/vagrant/.rbenv/versions/2.4.0/bin/ruby
</IfModule>
---
-->


## Passenger設定
- sudo vim /etc/httpd/conf/httpd.conf
<!---
- sudo vim /etc/httpd/conf.d/rails.conf
--->
    - 【コピー】の文字列を張り付け
- sudo vim /etc/httpd/conf/httpd.conf
```conf
<VirtualHost *:80>
  ServerName deco
  DocumentRoot /usr/local/deco/public
  RailsEnv production
  <Directory /usr/local/deco/public>
    AllowOverride all
    Options -MultiViews
  </Directory>
</VirtualHost>
```
- sudo wget --no-check-certificate https://tn123.org/mod_xsendfile/mod_xsendfile.c
- sudo apxs -cia mod_xsendfile.c
- sudo chmod 755 /usr/lib64/httpd/modules/mod_xsendfile.so
- sudo vim /etc/httpd/conf.d/xsendfile.conf
```conf
<IfModule mod_xsendfile.c>
    XsendFile on
    XsendFilePath /var/deco/files
</IfModule>
```
- sudo /etc/rc.d/init.d/httpd restart 


## スナップショット保存：5
- exit
    - vagrant halt
    - vagrant snapshot save savepoint5
    - vagrant snapshot list
        - 保存されていることを確認
    - vagrant up
    - vagrant ssh

## DECOのインストール
- cd /usr/local
- sudo wget http://www.deco.vc/wp-content/uploads/deco_1.1.4.tar.gz
- sudo tar xzvf deco_1.1.4.tar.gz
- sudo chmod 777 /usr/local/deco -rf
- cd deco
- sudo rm Gemfile.lock
- gem install bundler rake
- bundle install
- sudo vim /usr/local/deco/config/database.yml
```yml
production:
  adapter: mysql
  encoding: utf8
  database: deco_production
  pool: 5
  host: localhost
  username: deco
  password: ********
  socket: /var/lib/mysql/mysql.sock
```
- rake db:migrate RAILS_ENV=production
- rake db:seed RAILS_ENV=production
- sudo vim config/environments/production.rb
```rb
  ActionMailer::Base.smtp_settings[:address] = "localhost"
  ActionMailer::Base.smtp_settings[:domain] = "example.com"
```
- rake assets:precompile RAILS_ENV=production
- sudo chown -R apache:apache /usr/local/deco

## その他設定
- sudo mkdir -p /var/deco/files
- sudo chown -R apache:apache /var/deco/files
- sudo /etc/rc.d/init.d/httpd restart
- 以下のURLで管理画面で設定を行う
    - (DECOサーバのURL)/sys_top
- 設定が完了したら以下のURLでシステムにアクセス
    - (DECOサーバのURL)/

## 備考
- リフレッシュの手順
    - sudo /etc/rc.d/init.d/httpd restart
    - rbenv rehash
    - sudo /etc/init.d/mysqld restart
- bundle install時のエラーの対処法
    - sudo rm Gemfile.lock
    - sudo chown vagrant  /usr/local/deco