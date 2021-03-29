# 使用技術一覧
|  OS  |  DB  |  Webサーバ  |  サーバ言語  |  フレームワーク |
|  :--: | :--: | :--: | :--: | :--: |
|  CentOS7  |  MySQL5.7  |  Nginx  |  PHP7.3  |  laravel6.0  |

***
## 構築の流れ

1. ゲストOSの構築
vagrantを使用してゲストOSの構築を行います。
2. 使用するパッケージのインストール
ゲストOS内で使用するgitやphpなどのインストール。
3. Laravelアプリの作成
4. nginxのインストールと設定
今回はwebサーバーにnginxを使用します。
5. データベースの構築
今回はMySQLを使用します。
構築後Laravelとの接続を行います。

## ゲストOSの構築
### Vagrantの導入

- まずはゲストOSのbox(イメージファイル)をダウンロードします。
```shell=
$ vagrant box add centos/7
```

- 実行すると以下のような選択肢が表示されるので今回使用する3) virtualboxを選択しましょう。

```shell=
1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```

- 次にappのディレクトリを作成し、移動。（今回はvagrant_laravelとします。）
```shell=
$ mkdir vagrant_laravel && cd vagrant_laravel
```

- vagrant_laravel内で先ほどのboxを使用します。
```shell=
$ vagrant init centos/7

# 実行後問題なければ以下のような文言が表示されます
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

### Vagrantfileの編集

- まず、ポートフォワーディングを行います。今回はゲストOSの80番ポートをホストOSの8080番に転送します。
```shell=
# config.vm.network "forwarded_port", guest: 80, host: 8080
# ↓　コメントイン
config.vm.network "forwarded_port", guest: 80, host: 8080
```
- プライベートネットワークの設定。（今回は192.168.33.19を使用します。）なお、192.168.33.19を使用したことがある場合は他のipを指定しましょう。
```shell=
#config.vm.network "private_network", ip: "192.168.33.10"
# ↓　コメントインと編集
config.vm.network "private_network", ip: "192.168.33.19"
```

- ホストOSのカレントディレクトリ(vagrant_larevel)とゲストOSの/vagrantの同期を行います。
```bash=
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```

### Vagrantプラグインのインストール

- vagrant-vbguestというBoxにインストールされているGuest AdditionsをVirtualBoxのバージョンに合わせて最適化するプラグインを導入します。
```shell=
vagrant plugin install vagrant-vbguest
```
- また、今回はvagrantの状態を保存、巻き戻しが出来るsarahaというプラグインを導入します。
```shell=
vagrant plugin install sahara
```
- プラグインがインストールされているか確認しましょう。
```shell=
vagrant plugin list

# 下記のように表示されればOKです。
sahara (0.0.17, global)
vagrant-vbguest (0.29.0, global)
```
- saharaの使用方法(起動はゲストOSを立ち上げてから行ってください。)
```bash=
# 起動
vagrant sandbox on

# コミット
vagrant sandbox commit

# ロールバック
vagrant sandbox rollback

# 停止
vagrant sandbox off
```

### ゲストOSの起動

早速起動しましょう。
```bash=
vagrant up
```

- ゲストOSとホストOSのマウント（同期）がうまくいかない場合は下記を参照してください。
[https://qiita.com/mao172/items/f1af5bedd0e9536169ae](https://qiita.com/mao172/items/f1af5bedd0e9536169ae)
### ゲストOSへのログイン

- sshでログイン
```bash=
vagrant ssh
```

- 実行後、以下のような記述になっていればログインに成功しています。
```bash=
[vagrant@localhost ~]$
```

## 使用するパッケージのインストール
### 開発ツールなどのインストール
- gitなどの開発に必要なパッケージをインストールします。
```bash=
sudo yum -y groupinstall "development tools"
```
groupinstallを使用すれば複数のパッケージを一括でインストールできます。
- yumだけではインストールできない物もあるので、パッケージをいれることによってそれを可能にします。
```bash=
sudo yum -y install epel-release wget
```

### PHPのインストール
今回、PHPのバージョンは7.3を使用します。

- Remiリポジトリ（PHPをインストールする用のリポジトリ）のインストール。
```bash=
sudo yum -y install epel-release wget
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo rpm -Uvh remi-release-7.rpm
```
- PHPver7.3と拡張モジュールのインストール
```bash=
sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
```

- インストールの確認を行います。以下のように表示されればOKです。
```bash=
php -v

# 実行後
PHP 7.3.27 (cli) (built: Feb  2 2021 10:32:50) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.27, Copyright (c) 1998-2018 Zend Technologies
```

### composerのインストール

PHPのパッケージ管理ツールのcomposerをインストールします。

```bash=
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerを使用できるようにfileを移動させます。
sudo mv composer.phar /usr/local/bin/composer

# インストールの確認
composer -v

# 以下のような表記が出ればOK
Composer version 2.0.11 2021-02-24 14:57:23
```

## Laravelアプリの作成
- ゲストOSのvagrant直下にlaravel_appというプロジェクトを作成します。
```bash=
composer create-project --prefer-dist laravel/laravel test_app "6.*"

# インストールの確認
cd laravel_app
php artisan -v
```
- 今回はlaravel/uiを使って認証の実装を行います。
```bash=
$ composer require laravel/ui:^1.0 --dev
$ php artisan ui bootstrap --auth
```
以上で認証とある程度のUIを実装できます。


## nginxのインストールと設定

### インストール
- nginxをインストールするためのリポジトリを作成。
```bash=
$ sudo vi /etc/yum.repos.d/nginx.repo
```
下記の内容を書き込みます。
```bash=
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
- 保存したらインストールを行います。
```bash=
$ sudo yum install -y nginx

# インストールの確認
$ nginx -v

# 以下のような表記が出れば問題ないです。
nginx version: nginx/1.19.8
```
- インストールが確認できればNginxを起動します。
```bash=
$ sudo systemctl start nginx
```
ブラウザにて[http://192.168.33.19](http://192.168.33.19)(Vagrantofileで別のipを使用した場合は指定したip)と入力。
NginxのWelcomeページが表示されれば成功です。

 ### Laravelを動かす

 - Nginxの設定ファイルの編集
 ```bash=
 $ sudo vi /etc/nginx/conf.d/default.conf
 ```
 内容は下記の通りです。
 ```bash=
 server {
  listen       80;
  server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
  root /vagrant/laravel_app/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # root を除いたlocation { }のコメントアウトと編集
  # 矢印　←　に該当する部分のコメントアウトの忘れが多いので確認をしてください

  location ~ \.php$ { # ←
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  } # ←
 ```
 Nginxの設定は以上です。
 - php-fpmの設定ファイルの編集
 php-fpmはNginxでphpを動かすためのモジュールです。
 ```bash=
 $ sudo vi /etc/php-fpm.d/www.conf
 ```
 下記に変更
 ```bash=
 ;24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
 ```
- 以上で設定ファイルの編集は終了です。Nginxの再起動とphp-fpmの起動を行いましょう。
```bash=
$ sudo systemctl restart nginx
$ sudo systemctl start php-fpm
```

- 再度、[http://192.168.33.19](http://192.168.33.19)にアクセスしてみましょう。
すると、以下のようなエラーが発生すると思います。
```bash=
The stream or file "/vagrant/laravel_app/storage/logs/laravel.log" could not be opened: failed to open stream: Permission denied
```
先ほどphp-fpmの設定ファイルのuserとgropをnginxに変更しましたが、nginxに権限がないためパーミッションエラーが発生します。
nginxにも権限を付与してあげましょう。

- larevel_app直下で権限の操作を行います。
```bash=
$ cd /vagrant/laravel_app
$ sudo chmod -R 777 storage
```

これで再度[http://192.168.33.19](http://192.168.33.19)にアクセスするとLaravelの画面が表示されます。

## データベースの構築
### インストールと設定
- インストール
```bash=
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server

#インストールの確認
mysql --version
```
- インストールの確認ができましたら接続を行います。
ですが、初期パスワードが設定されているためパスワードを調べなけらばなりません。
下記コマンドで検索しましょう。
```bash=
sudo cat /var/log/mysqld.log | grep 'temporary password'
# 実行すると下記のようなログが出力されます。
2021-03-27T15:06:40.612424Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```
上記の[hogehoge]に該当する部分がパスワードになります。
- 一度、ログインできるか試しておきましょう。
```bash=
$ mysql -u root -p
$ Enter password:
mysql >
```
- 毎回先ほどのパスワードを使用するのは面倒なので、パスワードの再設定を行います。ですが、設定できるパスワードが厳格に決まっているため、それを解除し簡単なパスワードを設定できるようにします。
```bash=
# mysqlからログアウト
mysql < exit

$ sudo vi /etc/my.cnf
```
上記のファイルを編集します。
```bash=
# 省略

[mysqld]

# 省略

# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# 下記の一行を追加
validate-password=OFF
```
これで簡易的なパスワードを設定できます。
ですが、本来はセキュリティー上危険なので現場では厳格なパスワードを使用しましょう。
- 編集後はMySQLサーバーの再起動が必要となります。
```bash=
$ sudo systemctl restart mysqld
```
- そして先ほどのパスワードを使い再度ログインし、パスワードの再設定を行います。
```bash=
$ mysql -u root -p
$ Enter password:

mysql > set password = "新たなpassword";
```
### データベースの作成とLaravelへの接続
- Laravelで使用するデータベースの作成を行います。
```sql=
mysql > create database laravel_app;
```
Query OKと表示されたら作成は完了です。

- Laravelと接続します。laravel_appディレクトリの.envに設定用の変数が入っているので編集します。
```bash=
DB_DATABASE=laravel
# ↓ 以下に編集
DB_DATABASE=laravel_app

DB_PASSWORD=
# ↓ 以下に編集
DB_PASSWORD=登録したパスワード
```
これで設定は完了です。
- 最後にlaravel_app直下でphp artisan migrateを実行します。
```bash=
cd /vagrant/laravel_app
php artisan migrate
```
マイグレーションが成功すれば登録ができるようになります。
以上で環境の構築は完了です。

## 環境構築の所感
- エラーの検索方法はエラーメッセージを直接検索にかけるパターンと環境や使用ツールから検索するパターンの２つのパターンがある。（今までは前者でしか検索をかけていなかった）
- OSが違う場合でも「このツールはmacでいうとHomebrewだな」というように置換しながら考えれるとずいぶん理解が楽になる。
- 初見のものがいっぱいでやってる内容がわからない場合でも、一つずつ丁寧に調べていけば意外と理解はしやすい。
- 初めてのコマンドの実行や設定ファイルの編集は内容からある程度実行結果を想定していると、全体構造が掴みやすい。

## 参考サイト
[Vagrant + VirtualBOx で 最新のCentOS7 vbox(centos/7 2004.01)でマウントできない問題](https://qiita.com/mao172/items/f1af5bedd0e9536169ae)

[nginxのリポジトリを使用してnginxをインストール](https://www.server-memo.net/server-setting/nginx/nginx-repo.html#nginx)

[CentOSなどで使う、Remi Repositoryってなんだ？](https://qiita.com/charon/items/6d34ae798e9b05e8bd0a)

[【Laravel】「laravel-ui」について](http://www.code-magagine.com/?p=10606#:~:text=laravel%2Dui%E3%81%A8%E3%81%AF%EF%BC%9F,%E3%81%A7%E3%81%8D%E3%82%8B%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E3%81%AE%E4%BA%8B%E3%81%A7%E3%81%99%E3%80%82)

[saharaのgithub](https://github.com/jedi4ever/sahara)

