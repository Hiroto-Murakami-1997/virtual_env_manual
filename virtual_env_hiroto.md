# vagrant + centOS7 + nginx による環境構築マニュアル(Windows10)

## 使用する製品

|製品  |バージョン  |
|:---:|:---:|
|Virtual Box  |6.0.14  |
|Vagrant  |2.2.18  |
|centOS |7|
|MySQL|5.7|
|Nginx|1.21.1|
|PHP|7.3|
|Laravel |6.0|
## 目次
[①仮想環境に接続するディレクトリを作成](#①仮想環境に接続するディレクトリを作成)  
[②仮想環境にて使用するosを立ち上げる](#②仮想環境にて使用するosを立ち上げる)  
[③Vagrantfileの編集](#③vagrantfileの編集)  
[④Vagrantプラグインのインストール](#④vagrantプラグインのインストール)  
[⑤VagrantでゲストOSを起動](#⑤vagrantでゲストosを起動)  
[⑥ゲストOSにログイン](#⑥ゲストosにログイン)  
[⑦パッケージのインストール](#⑦パッケージのインストール)  
[⑧PHPのインストール](#⑧phpのインストール)  
[⑨composerのインストール](#⑨composerのインストール)  
[⑩Laravelアプリケーションのコピー作成](#⑩laravelアプリケーションのコピー作成)  
[⑪webサーバ(Nginx)のインストール](#⑪webサーバnginxのインストール)  
[⑫データベースのインストールと起動](#⑫データベースのインストールと起動)  
[⑬データベースの作成](#⑬データベースの作成)  
[⑭DBをLaravelに接続](#⑭dbをlaravelに接続)  
[⑮Laravelを動かす](#⑮laravelを動かす)  
[⑯アプリケーション画面が正しく表示されなかった場合](#⑯アプリケーション画面が正しく表示されなかった場合)  
[◎環境構築の所感](#環境構築の所感)  
[◎参考サイト](#参考サイト)

## ①仮想環境に接続するディレクトリを作成

分かりやすければどこでもいいですが、今回は例としてCドライブ直下に作成して下さい。 

```shell
$ mkdir vagrant  
$ cd vagrant
```

## ②仮想環境にて使用するOSを立ち上げる
 作業中のディレクトリが①で作成したvagrantであることを確認し、下記コマンドを実行。
```
$ vagrant init centos/7
```
 以下の表示が確認できたら次に進んでください。
```
A 'Vagrantfile' has been placed in this directory. You are now
ready to 'vagrant up' your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
'vagrantup.com' for more information on using Vagrant.
```
## ③vagrantfileの編集
1. 以下2点のコメントアウトを外し、 ip: "192.168.33.10"⇒ip: "192.168.33.19"に変更
```ruby
config.vm.network "forwarded_port", guest: 80, host: 8080
config.vm.network "private_network", ip: "192.168.33.19"
```
2. 以下のテキストを編集 
```ruby
# 編集前 
config.vm.synced_folder ".data", "/vagrant_data"
# 編集後
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
## ④Vagrantプラグインのインストール
```
$ vagrant plugin install vagrant-vbguest
```  
このプラグインは、Guest Additionsを常に自動で最新の状態に更新してくれる役割があります。  
※Guest Additionsとは、VirtualBoxの操作性を向上させるためのモジュールで、具体的には、ホストOSとゲストOSとの間での共有フォルダの作成ができるようになる、といった機能がある。  
コマンド実行後、プラグインか正常にインストールされているか確認のため、以下のコマンドを実行してください。
```
$ vagrant plugin list
```
## ⑤VagrantでゲストOSを起動
```
$ vagrant up
```
処理に少々時間がかかります。  

## ⑥ゲストOSにログイン
1. RLoginを起動
2. 「新規」をクリック
3. ホスト名(サーバーIPアドレス)を**127.0.0.1**に設定
4. TCPポートを**2222**に設定
5. ログインユーザー名を**vagrant**に設定
6. SSH認証鍵を選択すると、エクスプローラーが開くので、***vagrant/.vagrant/machines/default/virtualbox*** 直下の***private_key*** を指定して開くボタンをクリック。
7.  「OK」ボタンをクリック
8. 最後に、作成したvagrantディレクトリにてゲストOSにログイン  
```
$ vagrant ssh
# ログインが成功すると、以下の表示になります。  
[vagrant@localhost ~]$
```  
ここで一度共有フォルダがvagrantの名前で認識されているか一度確認をします。
```
[vagrant@localhost ~]$ cd /
[vagrant@localhost /]$ ls 
```
すると、おそらくvagrantという名前のディレクトリが見当たらないはずです。  
ここで一度exitでゲストOSからログアウトし、下記コマンドを実行。
```
$ vagrant reload
```
これでvagant upが再度行われます。
すると、
```
No Guest Additions
```
という表示が出るはずです。
これは、No Guest Additionsそのものがまだインストールされておらず、ゲストＯＳとホストＯＳでフォルダの共有をする機能を持ったモジュールがない状態を表しています。
そこで、Guest Additionsのインストールを行います。
```
[vagrant@localhost ~]$ yum install kernel-devel
[vagrant@localhost ~]$ yum -y update kernel
```
この後、再度exitして
```
$ vagrant reload
$ vagrant ssh
[vagrant@localhost ~]$ cd /
[vagrant@localhost /]$ ls 
```
こうすることで、ルートディレクトリにてlsコマンドを打つと、共有フォルダがvagrantの名前で表示されるかと思います。

## ⑦パッケージのインストール
### パッケージ、パッケージ管理システムとは
- パッケージとは実行ファイル、設定ファイル、ライブラリなどをがまとめられたもの。
- パッケージは個々に、どのパッケージがないとどのパッケージが動かないといった依存関係をもっているため、パッケージ管理システムによってパッケージの依存関係を自動で解決している。
### インストールの実行
```
[vagrant@localhost ~]$ sudo yum -y groupinstall "development tools"
```
## ⑧PHPのインストール
```
[vagrant@localhost ~]$ sudo yum -y install epel-release wget ※外部パッケージツール   
[vagrant@localhost ~]$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm  
[vagrant@localhost ~]$ sudo rpm -Uvh remi-release-7.rpm  
[vagrant@localhost ~]$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip  
```
最後にバージョン確認でこのように表示されれば成功です。
```
[vagrant@localhost ~]$ php -v
PHP 7.3.30 (cli) (built: Aug 24 2021 10:03:17) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.30, Copyright (c) 1998-2018 Zend Technologies
```
※yum - 依存関係の管理、解決  
※rpm - 依存関係管理のみ  
※wget - 指定先URLのファイルをダウンロード

## ⑨composerのインストール
**comporser** - PHPのパッケージ管理ツール  
```
# composer-setup.php をダウンロード  
[vagrant@localhost ~]$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"  
# ダウンロードした composer-setup.php を実行  
[vagrant@localhost ~]$ php composer-setup.php  
# composert-setup.phpを削除  
[vagrant@localhost ~]$ php -r "unlink('composer-setup.php');"  
# どのディレクトリでもcomposerコマンドが使えるようにする  
[vagrant@localhost ~]$ sudo mv composer.phar /usr/local/bin/composer  
[vagrant@localhost ~]$ composer -v
```

## ⑩Laravelアプリケーションのコピー作成
一旦、exitコマンドでゲストOSからログアウトしてください。  
```
$ cd vagrant  
$ cp -r laravel_login(ログイン機能を実装したアプリケーションディレクトリまでの絶対パス) ./
```

## ⑪webサーバ(Nginx)のインストール
### 1. vagrantディレクトリにてゲストOSにログイン(**vagrant ssh**)
### 2. nginxインストールの前に、nginxパッケージリポジトリをセットアップ。
```
[vagrant@localhost ~]$ sudo vi /etc/yum.repos.d/nginx.repo
# 以下を書き込み、保存。  
[nginx]  
name=nginx repo  
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/  
gpgcheck=0  
enabled=1
```
こうすることで、/etc/yum.repos.d/nginx.repoファイルが新規作成、保存されます。
### 3. nginxインストール実行  
```
[vagrant@localhost ~]$ sudo yum install -y nginx  
[vagrant@localhost ~]$ nginx -v
```
### 4. Nginxの起動  
```
[vagrant@localhost ~]$ sudo systemctl start nginx
```
ブラウザに入力（http://192.168.33.19)  
⇒NginxのWelcomeページが表示されれば成功です。

## ⑫データベースのインストールと起動
### 1. Mysql 5.7のインストール  
```
[vagrant@localhost ~]$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm  
[vagrant@localhost ~]$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm  
[vagrant@localhost ~]$ sudo yum install -y mysql-community-server  
[vagrant@localhost ~]$ mysql --version
```
### 2. Mysqlの起動 
- Mysqlの起動  
```
[vagrant@localhost ~]$ sudo systemctl start mysqld
```
- ログインするために初期設定のパスワードを表示  
```
[vagrant@localhost ~]$ sudo cat /var/log/mysqld.log | grep 'temporary password'
# 実行結果の例  
[vagrant@localhost ~]$ 2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```  
hogehogeの部分がパスワードとなっているので、コピーする。
- コピーしたパスワードでログイン  
```
[vagrant@localhost ~]$ mysql -u root -p  
Enter password:
```
※Enter password:には入力しても何も表示されませんが実際は入力されているので、ログインに失敗した場合、しっかりコピーできていない、もしくは正しいパスワードの入力ができていないという事になりますので、何度か試してみてください。  
ログインに成功すると、下記が表示されます。  
```
mysql >
```
- パスワードの変更    
```sql
mysql > set password = "新たなpassword";
```
※大文字小文字の英数字 + 記号かつ8文字以上  
変更後、新しいパスワードで再ログインできればパスワードの変更は成功です。
## ⑬データベースの作成
```sql
mysql > create database laravel_login;  
mysql > show databases; ※データベースが正しく作成されたか確認
```
## ⑭DBをLaravelに接続
```shell
$ cd laravel_login  
$ vi .env  

DB_PASSWORD=登録したパスワードに編集
```
編集後、保存が完了したら、laravel_logniディレクトリにて、マイグレーションを実行。  
```
[vagrant@localhost laravel_login]$ php artisan migrate
```  
再度Mysqlにログインし、マイグレーションか成功したか確認。  
```sql
mysql > use laravel_login;
mysql >show tables;
```
成功していれば、下記の3つが追加されているはずです。  
- failed_jobs   
- password_resets  
- users
## ⑮Laravelを動かす
### 1. Nginxの設定ファイルを編集  
```
[vagrant@localhost ~]$ sudo vi /etc/nginx/conf.d/default.conf

# 以下、編集内容

server {
  listen       80;
  server_name  192.168.33.19; # Vagranfileでコメントアウトした箇所のipで、デフォルトだと192.168.33.10ですが、今回は192.168.33.19に変更したので、server_nameも192.168.33.19に変更してください。
  root /vagrant/laravel_login/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }
```  
### 2. php-fpm の設定ファイルを編集  
```
[vagrant@localhost ~]$ sudo vi /etc/php-fpm.d/www.conf

# 以下編集内容

# 24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```  
### 3. nginxの再起動  
```
[vagrant@localhost ~]$ sudo systemctl restart nginx  
[vagrant@localhost ~]$ sudo systemctl start php-fpm
```
ブラウザにhttp://192.168.33.19 を入力し、laravelの画面が表示されるか確認。
laravel画面右上のsigninとregisterから登録とサインインができたら完成です。

## ⑯アプリケーション画面が正しく表示されなかった場合
### 1. 画面自体が表示されない場合  
ファイアウォールの設定  
 Vagrantfileの編集をした際、に編集した```config.vm.network "forwarded_port", guest: 80, host: 8080```の、```guest: 80```はhttp通信を行うための通路番号を表しており、ファイアウォールに対してこのポートを経由でhttp通信することを許可する必要があります。
 ```
 # ファイアウォールの起動
 [vagrant@localhost ~]$ sudo systemctl start firewalld.service
 # http通信の許可
 [vagrant@localhost ~]$ sudo firewall-cmd --add-service=http --zone=public --permanent
 [vagrant@localhost ~]$ sudo firewall-cmd --reload
 # nginxの再起動
 [vagrant@localhost ~]$ sudo systemctl restart nginx
 ```
### 2. 403 Forbiddenが表示された場合
SELinuxの設定を変更する  
SELinuxの予備知識
- SELinux : Linuxに強制アクセス制御(MAC)機能を追加するモジュールで、Linuxをより安全に運用することができる。
- SELinux コンテキスト : 通常Linuxで扱われるパーミッションのほかにSELinuxが持っているパーミッション。  
今回はこの機能を無効化することで、パーミッションの不一致を解消します。
```
[vagrant@localhost ~]$ sudo vi /etc/selinux/config

# 下記を編集

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
# 編集前 SELINUX=enforcing  
SELINUX=disabled #編集後
```
編集後、ゲストOSを再起動。
```
[vagrant@localhost ~]$ exit
$ vagrant reload
$ vagrant ssh
[vagrant@localhost ~]$ sudo systemctl start nginx
```
# 環境構築の所感
## 1. マニュアル通りにはいかないこと
サーバーレッスンに入ってからvagrantやVirtualBoxをインストールする時点で、既にギズテックの通りにうまく行かない事が多々あり、自分で解決方法を探るのに苦労しました。それでも、一つ一つ解決方法を模索していく中で自分に知識がついたと思います。例えば、ゲストOSとの共有フォルダの設定がなかなかうまくいかなかった時に、カーネルモジュールをインストールしなければいけないという答えにたどり着き、その道中でVagrantやVirtualBoxに対する理解が深まったと実感できました。
## 2. エラーに対する姿勢
サーバーに関しては今の時点の自分の知識量では、エラーが一つ起きたとして、そのエラーを検索すると、数倍の情報が帰ってきます。その情報の一つ一つを理解するのは一見途方もない作業に見えますが、少しずつ確実に理解すると、どこかで見た内容がちらほら見え始めて、点と線が一本につながったような体験を何度かしました。この体験から、自分はエラーやバグに対してよりポジティブに向かい合うことができるようになったと思います。
# 参考サイト
- [VirtualBoxを使おう - Hint & Tips](https://www.linuxmania.jp/virtualbox_02.html)  
- [Vagrantのフォルダ共有機能Guest Additionsでのトラブル対応](https://www.shibuya24.info/entry/trouble_guest_additions)  
- [パッケージ管理システムとは？】Linuxでのパッケージ管理の使い方まとめました](https://eng-entrance.com/category/linux/linux-package)  
- [理由がわかれば怖くない！SELinux とのつきあい方](https://blog.fenrir-inc.com/jp/2016/09/selinux.html)










