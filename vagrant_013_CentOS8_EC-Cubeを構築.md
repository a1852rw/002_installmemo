# EC-Cubeの検証環境を構築
## 環境設定
- OS：CentOS 8

## コマンドメモ
### 環境構築
- vagrant init bento/centos-8.0
- Vagrantfileを編集
```txt
# config.vm.network "public_network"
config.vm.network "forwarded_port", guest: 80, host: 2080   # HTTP
config.vm.network "forwarded_port", guest: 443, host: 20443  # HTTPS
```
- vagrant up

### 初期設定
- vagrant ssh
- sudo yum update -y
- sudo yum install -y unzip

### ここで保存
- exit
- vagrant snapshot save savepoint_001
- vagrant snapshot list

### Apacheインストール
- sudo yum install -y httpd
    - httpd -v
- sudo systemctl start httpd.service
- sudo systemctl enable httpd.service
- ブラウザで動作確認
    - 127.0.0.1:2080
    - ウェルカムページが表示されたら成功

### ここで保存
- exit
- vagrant snapshot save savepoint_002
- vagrant snapshot list



