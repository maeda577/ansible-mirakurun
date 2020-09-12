# Mirakurun構築用Ansible Playbook

[Mygica S270とRaspberryPi3とWSLで録画環境 | メモ帳.scr](https://maeda577.github.io/2020/08/23/s270.html) で書いたmirakurunの構築手順をAnsibleのロールにしたものです。以下2つのロールから構成されます。
* common
* mirakurun

### common
* 目的
    * 鍵認証を有効化したりタイムゾーンを設定したりホスト名を設定したりする
        * 初期設定が済んでいれば不要
* 必要な変数
    * ansible_user: ssh接続のユーザー名
    * hostname: 設定するホスト名

### mirakurun
* 目的
    * mirakurunを導入する
* 必要な変数
    * ansible_user: ssh接続のユーザー名
    * nodejs_version: 導入するnode.jsのバージョン番号
        * 未定義の場合はデフォルトで14を導入する

動作環境
-----------------------
Raspberry Pi 3 model B上のUbuntu 20.04でのみ動作確認しています。他の環境は不明です。パッケージインストールの部分がUbuntuのものしか書かれていないため、他のディストリビューションでは動きません。

使い方
-----------------------
* Mirakurun側はSSHが繋がる程度にはセットアップしておく
* inventories/sample/hosts の中のIPアドレスを修正する
* WSLのUbuntu 20.04で以下を実行。18.04だとaptで入るansibleが古いのでpip経由で入れる
``` shell
# Ansible導入
sudo apt install ansible sshpass

# 接続先のホストに一度は繋いでknown_hostsに足し、アップデートする
ssh ubuntu@192.168.0.100 sudo apt update
ssh ubuntu@192.168.0.100 sudo apt upgrade

# 疎通確認
ansible -i inventories/sample/ all -m ping --ask-pass

# SSH Keyを作る
ssh-keygen -t rsa -b 4096

# Playbook実行 初回はSSH Keyが登録されていないので--ask-passをつける
ansible-playbook tuner_servers.yml -i inventories/sample/ --ask-pass -v

# 2回目移行はask-pass無し
ansible-playbook tuner_servers.yml -i inventories/sample/ -v
```