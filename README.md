# Mirakurun構築用Ansible Playbook

[Mygica S270とRaspberryPi3とWSLで録画環境 | メモ帳.scr](https://maeda577.github.io/2020/08/23/s270.html) で書いたmirakurunの構築手順をAnsibleのロールにしたものです。以下のロールから構成されます。
* common
* mirakurun
* zabbix

### common
* 目的
    * 鍵認証を有効化したりタイムゾーンを設定したりホスト名を設定したりFWを加えたりする
        * 初期設定が済んでいれば不要
* 必要な変数
    * accept_tcp_ports: 許可するInputのポート(SSHの22番は常に許可)

### mirakurun
* 目的
    * mirakurunを導入する
* 必要な変数
    * nodejs_version: 導入するnode.jsのバージョン番号
        * 未定義の場合はデフォルトで14を導入する

### zabbix
* 目的
    * Zabbix + MySQL + Apacheを導入する
* 必要な変数
    * zabbix_version: 導入するZabbixのバージョン番号
        * 未定義の場合はデフォルトで5.0を導入する

動作環境
-----------------------
Raspberry Pi 3 model B上のUbuntu 20.04とRaspberry Pi OS Lite August 2020でのみ動作確認しています。他の環境は不明です。パッケージインストールの部分がUbuntu、Debianのものしか書かれていないため、他のディストリビューションでは動きません。

使い方
-----------------------
* Mirakurun側はSSHが繋がる程度にはセットアップしておく
* inventories/sample/hosts の中のIPアドレスを修正する
* WSLのUbuntu 20.04で以下を実行

``` shell
# Ansible導入
sudo apt install python3-pip sshpass
sudo pip3 install ansible ansible-lint

# 接続先のホストに一度は繋いでknown_hostsに足し、アップデートする
ssh ubuntu@192.168.0.100 sudo apt update
ssh ubuntu@192.168.0.100 sudo apt upgrade

# 疎通確認
ansible -i inventories/sample/ all -m ping --ask-pass

# SSH Keyを作る
ssh-keygen -t rsa -b 4096

# zabbix関連のロールを使うときは関連コレクションを入れる
ansible-galaxy install -r requirements.yml

# Playbook実行 初回はSSH Keyが登録されていないので--ask-passをつける
ansible-playbook site.yml -i inventories/sample/ --ask-pass --ask-become-pass -v

# 2回目移行はask-pass無し
ansible-playbook site.yml -i inventories/sample/ --ask-become-pass -v
```

事前チェック関連のメモ
-----------------------
``` shell
# 構文エラーの確認
ansible-playbook site.yml -i inventories/sample/ --syntax-check
# ドライラン
ansible-playbook site.yml -i inventories/sample/ --check

# コーディング規約チェック
ansible-lint site.yml
```
