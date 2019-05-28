# CentOS7にRedmineバージョン4.0をインストール

## 事前の準備
### 検証環境
- 仮想化ツール1：Vagrant 2.1.2
- 仮想化ツール2：Oracle VirtualBox 5.2.22
- ゲストOS：Cent OS 7.5 (bento/centos-7.5 v201811.25.0)
    - https://app.vagrantup.com/bento/boxes/centos-7.5
- ツール1：Apache 2.4.6
- ツール2：Python3.7

### Vagrantの初期設定
- vagrant init bento/centos-7.5
- Vagrantfileを編集(メモ帳などで直接書き換える)
```txt
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
```

### CentOS7の初期設定
- sudo yum -y update && sudo yum -y upgrade
- uname -a
- cat /etc/redhat-release
    - 更新後のOSバージョンを確認
- sudo yum install -y vim tree git wget
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
- apache2でpythonによるcgiを動かすまでのメモまとめ
    - https://qiita.com/shita_fontaine/items/05c7f47f588491c0212f
- Server World Apache httpd :Server World 
    - https://www.server-world.info/query?os=CentOS_7&p=httpd&f=17

### 必要なツールのインストール
- sudo yum install -y httpd python3 pip3

### Apache初期設定
- grep -n "^ *ScriptAlias" /etc/httpd/conf/httpd.conf 
  - cgiの実行が許可されていることを確認する
  - 「247:    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"」と表示されれば成功
```text
CGI の実行はデフォルトで「/var/www/cgi-bin/」配下で許可されている。
「/var/www/cgi-bin/index.py」のように配置することで、「http://(httpd サーバー)/cgi-bin/index.py」へアクセス可能となる。
注：この設定は「/var/www/cgi-bin/」配下のファイルを全て CGI と扱うため、CGI 以外のファイルは表示不可である。
```
- sudo mkdir /var/www/html/cgi-enabled/
- sudo vim /var/www/html/cgi-enabled/index.py
- sudo chmod 705 /var/www/html/cgi-enabled/index.py 
```py
#!/usr/bin/env python

print "Content-type: text/html\n\n"
print "<html>\n<body>"
print "<div style=\"width: 100%; font-size: 40px; font-weight: bold; text-align: center;\">"
print "PythonによるWebページ構築成功\n"
print "</div>\n</body>\n</html>"
```
### ブラウザでの操作
- 下記にアクセスしスクリプトが表示されることを確認
- http://localhost:2080/cgi-enabled/index.py

### スナップショット2
- vagrant halt 
- vagrant snapshot save savepoint2
- vagrant snapshot list
- vagrant up