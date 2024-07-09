# PostgresユーザにログインしたときにPostgreSQL Serverを起動する

## 概要

Postgresユーザにログインしたときに、PostgreSQL Serverが起動されていない場合のみ、起動する方法を記載する。  

## 方法

以下の内容で *.bash_profile* を作成する。  
``` shell
# Define environment variables
export PGDATA=~/data
export PATH=$PATH:/usr/lib/postgresql/16/bin

# Start PostgreSQL Server(if necessary)
pg_isready > /dev/null

if [ $? -ne 0 ]; then
        pg_ctl start
fi
```

## 説明

### 環境変数の定義

以下の2行で環境変数を定義している。
``` shell
export PGDATA=~/data
export PATH=$PATH:/usr/lib/postgresql/16/bin
```

環境変数を設定する意図は以下。  
* PGDATA → 後のコマンドで、データベースクラスタのディレクトリを指定しなくてよくなる
* PATH → 後のコマンドをフルパスで実行する必要がなくなる

### PostgreSQL Serverの起動状態の確認

以下で確認している。
``` shell
pg_isready > /dev/null
```

上記のコマンドは、 **PostgreSQLが実行されている場合は0を返し** 、 **それ以外の場合は別の数値を返す** （実機では2が返ってきた）  

### PostgreSQL Serverの起動

以下で起動している。
``` shell
if [ $? -ne 0 ]; then
        pg_ctl start
fi
```

*pg_isready* コマンドの返り値が0以外の場合、PostgreSQL Serverを起動している。



