## Rails5 + PostgreSQL 開発環境セットアップ
### セットアップされるバージョンなど
* Linux: CentOS 7.3
* Ruby: 2.6.3
* Rails: 5.2.3
* NodeJS: 8.5.0
* PostgreSQL: 9.6

* ※ゲストOSへは192.168.12.10でアクセス可能になります  
※PostgreSQLにはローカルの5432ポートでもアクセス可能です  
※Railsのポート3000はローカルにフォワードしてません

### 用意するもの
* Windows10の64bit環境で検証済みです。  
⇒Linux, iOSでも多分ほとんど同じ手順でいけると思われますが、検証はしていません。  

* Git関連  
gitとか、git-bashとか必要なものはあらかじめ入っている想定です。

* Vagrantの最新版 (>= 2.2.3)  
[ダウンロードリンク](https://www.vagrantup.com/downloads.html)  
⇒パッケージインストールするだけです。  
(windowsでは、`C:\HashiCorp\Vagrant\bin`にセットアップされます)

* Virtual Box v6.0.6 (多分最新でも大丈夫、実績版はこれです)  
[ダウンロードリンク](http://download.virtualbox.org/virtualbox/6.0.6/)  
⇒これもパッケージインストールするだけです。  
※もし一度もVMを起動した事がないPCの場合は、BIOSの設定も確認して下さい。  
[参考リンク](https://support.bluestacks.com/hc/ja/articles/115003910391-PC%E3%81%A7%E3%83%90%E3%83%BC%E3%83%81%E3%83%A3%E3%83%A9%E3%82%A4%E3%82%BC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3-VT-%E3%82%92%E4%BD%9C%E5%8B%95%E3%81%95%E3%81%9B%E3%82%8B%E3%81%AB%E3%81%AF-)

* centos7.3のboxを追加

```
# git-bashで操作
$ vagrant box add --insecure bento/centos-7.3 https://app.vagrantup.com/bento/boxes/centos-7.3
==> box: Loading metadata for box 'https://app.vagrantup.com/bento/boxes/centos-7.3'
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) parallels
2) virtualbox
3) vmware_desktop

Enter your choice: 2    <- 2を選択
```
以上で前準備は終了です。


### 初回起動手順
#### ローカルにrepositoryをclone
以下、git-bashで`C:\setup-rails` にvagrantの起動環境を作成する想定として説明します。

```
# git-bashで操作
$ cd /c
$ git clone https://github.com/tom-kudo/setup-rails-with-postgresql.git setup-rails
$ cd setup-rails
```

#### パスワードなど変更
エディタを使って適当に設定されているパスワードを自分の好みに変更します。
* OSのpostgresユーザの設定：ansible\roles\postgresql\tasks\main.yml 23行目
* PostgreSQLのvagrantユーザの設定：ansible\roles\postgresql\files\vagrant.sql 1行目  
ansible\roles\setup_project\tasks\main.yml 23行目
* samba接続用vagrantユーザの設定：ansible\roles\setup_project\tasks\main.yml 7行目

#### Vagrant起動
初回はVMに必要packageを導入＆設定を追加するので10分ほど時間がかかります。最後に「faild=0」と表示されていればansibleによる自動設定は完了です。もしfaild=1となっている場合は、途中でsetupが異常終了してますので、原因を取り除いて再setup (vagrant provision) する必要があります。  
2回目以降はこの処理は動かないので1分程度でVMが起動します。

```
# git-bashで操作
$ cd /c/setup-rails
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'bento/centos-7.3'...

~~~ 省略 ~~~

==> default: Running provisioner: ansible_local...
    default: Installing Ansible...
Vagrant has automatically selected the compatibility mode '2.0'
according to the Ansible version installed (2.8.1).

    default: Running ansible-playbook...
PLAY [all] *********************************************************************

~~~ 省略 ~~~

PLAY RECAP *********************************************************************
default                    : ok=40   changed=33   unreachable=0    failed=0    skipped=0    rescued=0    ignored=4

$
```

#### ゲストOSへrailsリポジトリのcloneと、setupの実行
ホストOSのc:\setup-railsは、ゲストOSの/vagrantに自動でマウントされますので、ここに既存のrailsリポジトリをcloneするのが一番使い勝手が良いと思います。ここであればホストOSもゲストOSも同じファイルを気にせず共有できます。場合によってはゲストOSの/home/vagrantにチェックアウトしてsambaで共有するのもありだと思います。sambaはansibleで設定されています。  

その後bundle install -> rails db:create -> rails db:schema:load -> rails db:seedあたりを動かせばrails serverが動くようになっていると思います。  

参考) ゲストOSのsamba共有 (/home/vagrant) をwindowsにマウントする手順
```
#dosプロンプトで操作
DOS>  net use k: \\192.168.12.10\vagrant
'192.168.12.10' のユーザー名を入力してください: vagrant
192.168.12.10 のパスワードを入力してください: (vagrant)
コマンドは正常に終了しました。

DOS> k:
DOS> dir
```
