# Redmineサーバ環境構築  

Rasberry Pi（Ubuntu）にRedmineをインストール

## 環境

### ハードウェア

|ハードウェア名 |モデル|
| ---- | ---- |
|Rasberry Pi|4B(メモリ4GB)|

### ソフトウェア

|ソフトウェア種別 |ソフトウェア |バージョン |
| ---- | ---- | ---- |
| OS | Ubuntu | 22.04 |
| データベース | PostgreSQL | 14.11 |
| Webサーバ | Apache | 2.4.52 |

## 前提条件  

  *  PostgreSQL 14が入っていること

## 実施手順  

### ①日本語ロケールの対応

```
sudo apt install language-pack-ja
```

### ②必要なパッケージのインストール

```
$ sudo apt update
$ sudo apt install -y build-essential zlib1g-dev libssl-dev libreadline-dev libyaml-dev libcurl4-openssl-dev libffi-dev
$ sudo apt install -y libpq-dev
$ sudo apt install -y apache2 apache2-dev
$ sudo apt install -y imagemagick fonts-takao-pgothic
$ sudo apt install -y subversion git
```

### ③Rubyのインストール

Ruby 3.3.1(作業日の最新バージョン)をインストール  
```
$ curl -O https://cache.ruby-lang.org/pub/ruby/3.3/ruby-3.3.1.tar.gz
$ tar xvf ruby-3.3.1.tar.gz
$ cd ruby-3.3.1
$ ./configure --disable-install-doc
$ make
$ sudo make install
$ cd ..
```
（数十分かかった）

インストールされたことを確認
```
$ ruby -v
ruby 3.3.1 (2024-04-23 revision c56cd86388) [aarch64-linux]
```

### ④PostgreSQLの設定

1. ユーザ作成  
   ```
   sudo -i -u postgres createuser -P redmine
   ```  
   パスワードの入力が求められるため、 _redmine_ を入力  
1. データベース作成  
   ```
   sudo -i -u postgres createdb -E UTF-8 -l ja_JP.UTF-8 -O redmine -T template0 redmine
   ```

### ⑤Redmineのインストール  

```
$ sudo mkdir /var/lib/redmine
$ sudo chown shusei /var/lib/redmine
$ sudo -u shusei svn co https://svn.redmine.org/redmine/branches/5.0-stable /var/lib/redmine
```

### ⑥データベース接続設定 
 
以下の内容でdatabase.ymlを作成する。  
```
$ cat config/database.yml
production:
  adapter: postgresql
  database: redmine
  host: localhost
  username: redmine
  password: "redmine"
  encoding: utf8
```
※データベース名、ユーザ名、パスワードは適宜変更する。

### ⑦設定ファイル作成  

以下の内容で作成する。  
```
$ cat config/configuration.yml
production:
  email_delivery:
    delivery_method: :smtp
    smtp_settings:
      address: "localhost"
      port: 25
      domain: "ras-pi-nas"

  rmagick_font_path: /usr/share/fonts/truetype/takao-gothic/TakaoPGothic.ttf
```

### ⑧gemパッケージのインストール

```
sudo bundle install --without development test
```

コマンドを実行したところ、以下のエラーが出力された。
```
Your Ruby version is 3.3.1, but your Gemfile specified >= 2.5.0, < 3.2.0
```

Gemfileの以下の2点を修正し、再実行した。
```
$ sudo vi /var/lib/redmine/Gemfile
修正1
	修正前 : ruby '>= 2.5.0', '< 3.2.0'
	修正後 : ruby '>= 2.5.0', '< 3.3.2'

修正2
	修正前 : gem "pg", "~> 1.2.2", :platforms => [:mri, :mingw, :x64_mingw]
	修正後 : gem "pg", "~> 1.5.5", :platforms => [:mri, :mingw, :x64_mingw]
```

### ⑨Redmineの初期設定

1. セッション改ざん防止用の秘密鍵の作成  
```
$ bin/rake generate_secret_token
```
1. データベースにテーブルを作成  
```
$ RAILS_ENV=production bin/rake db:migrate
```

### ⑩Passengerのインストール

以下のコマンドを実行。
（下のコマンドに１５分ぐらい時間かかった）
```
sudo gem install passenger -N
sudo passenger-install-apache2-module --auto --languages ruby
```

### ⑪Apacheの設定

1. Apacheに追加する設定を確認  
```
$ passenger-install-apache2-module --snippet
```  
1. 設定ファイル( _etc/apache2/conf-available/redmine.conf_ )を作成  
attachment:redmine.conf
1. 設定反映  
> sudo a2enconf redmine  
> apache2ctl configtest  
> sudo systemctl reload apache2

## 接続確認  

ブラウザに以下を入力して、Redmineにアクセスできることを確認。  
```
http://<ホスト名 or IPアドレス>/redmine
```

## 初期設定

以下に従って初期設定を実行。  
> https://redmine.jp/tech_note/first-step/admin/

※「プロジェクト管理のための準備」の「1. ステータスの追加」～「4. ワークフローの設定」までは使用しないため、飛ばした。

## 参考  

  *  日本語ロケールについて : https://qiita.com/valzer0/items/db7639d8231bf5121297
  *  Redmineインストール方法 : https://blog.redmine.jp/articles/5_0/install/ubuntu/
  *  Rubyインストール方法 : https://www.ruby-lang.org/ja/downloads/