# Ubuntu最初の設定
- WSLでUbuntuをインストールした時に最初に行う設定の手順をまとめた

## 全部アップデート
- sudo apt-get update

## ツールをインストール
- sudo apt-get install tree
- sudo apt-get install vim
- sudo apt-get python3
- sudo apt-get git

## プロンプトの表示を変える
- sudo vim ~/.bashrc に追記する
```bash
export PS1="\[\e[1;32m\]\u@\h \W\$\[\e[m\]"
```

## vimの設定を変える
- sudo ./vimrc に追記する
```vimrc
set number
hi Comment ctermfg=gray
```

## git-hubのための設定
- ssh-keygen
- cd ~/.ssh
- cat id_rsa >> authorized_keys
- cat id_rsa.pub 表示された公開鍵をgit-hubのsshページに登録する
- cd ~
- mkdir ~/001_git-hub
- cd 001_git-hub
- git clone git-hub上に作ったリポジトリ
