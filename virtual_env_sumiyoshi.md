# 使用技術一覧
|  OS  |  DB  |  Webサーバ  |  サーバ言語  |  フレームワーク |
|  :--: | :--: | :--: | :--: | :--: |
|  CentOS7  |  MySQL5.7  |  Nginx  |  PHP7.3  |  laravel6.0  |

***
## Vagrantの使用方法
### Vagrantの導入

- まずはゲストOSのbox(イメージファイル)をダウンロードします。
```bash=
$ vagrant box add centos/7
```

- 実行すると以下のような選択肢が表示されるので今回使用する3) virtualboxを選択しましょう。

```bash=
1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```

- 次にappのディレクトリを作成し、移動。（今回はvagrant_laravelとします。）
```bash=
$ mkdir vagrant_laravel && cd vagrant_laravel
```

- vagrant_laravel内で先ほどのboxを使用します。
```bash=
$ vagrant init centos/7

# 実行後問題なければ以下のような文言が表示されます
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

### Vagrantfileの編集

- まず、ポートフォワーディングを行います。今回はゲストOSの80番ポートをホストOSの8080番に転送します。
```bash=
# config.vm.network "forwarded_port", guest: 80, host: 8080
# ↓　コメントイン
config.vm.network "forwarded_port", guest: 80, host: 8080
```
- プライベートネットワークの設定。（今回は192.168.33.19を使用します。）なお、192.168.33.19を使用したことがある場合は他のipを指定しましょう。
```bash=
#config.vm.network "private_network", ip: "192.168.33.10"
# ↓　コメントインと編集
config.vm.network "private_network", ip: "192.168.33.19"
```

### Vagrantプラグインのインストール

- vagrant-vbguestというBoxにインストールされているGuest AdditionsをVirtualBoxのバージョンに合わせて最適化するプラグインを導入します。
```bash=
vagrant plugin install vagrant-vbguest
```
- また、今回はvagrantの状態を保存、巻き戻しが出来るsarahaというプラグインを導入します。
```bash=
vagrant plugin install sahara
```
- vagrant-vbguestがインストールされているか確認しましょう。
```bash=
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

-　実行後、以下のような記述になっていればログインに成功しています。
```bash=
[vagrant@localhost ~]$
```

## 使用するバッケージのインストール
### 開発ツールなどのインストール
- gitなどの開発に必要なパッケージをインストールします。
```bash=
sudo yum -y groupinstall "development tools"
```
- yumだけではインストールできない物もあるので。パッケージをいれることによってインストールを可能にします。
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
```


