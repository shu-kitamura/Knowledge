# PythonコンテナからMySQLコンテナにクエリを実行する

{{toc}}


##  概要

Docker上にPython, MySQLのコンテナを起動している環境で、  
PythonコンテナからMySQLコンテナに対して、クエリを実行する方法を記載する。

##  使用環境

|ソフトウェア |バージョン |
| ---- | ---- |
| Docker Desktop | 4.0.1 |
| Doker Engine | 20.10.8 |
| Python | 3.10.2 |
| MySQL | 8.0.28 |

## 手順  

以下の手順で、PythonコンテナからMySQLコンテナにクエリを実行する。  

### 1. コンテナとネットワークの作成

#### コンテナイメージの取得

以下のコマンドでpython, mysqlのコンテナイメージを取得
```
docker pull python
```
```
docker pull mysql
```

#### ネットワークの作成

以下のコマンドで **python-mysql** という名前のネットワークを作成  
```
docker network create python-mysql
```

作成されたことを確認  
```
PS C:\Users\Programs\docker\mysql> docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
a51d54c0b2c9   bridge         bridge    local
faaad3171f2c   host           host      local
c11908b83263   none           null      local
0f5d7dd03e03   python-mysql   bridge    local
```

####  コンテナの起動

以下のコマンドでコンテナを起動。  
起動時のコマンドラインの引数 _--network_ に **python-mysql** を指定。

  *  MySQLコンテナ  
コンテナ名 **mysql_con** でコンテナを起動
```
docker run --name mysql_con  --network python-mysql  -e MYSQL_ROOT_PASSWORD=mysql_pass -d -p 33306:3306 mysql
```

  *  Pythonコンテナ  
コンテナ名 **python_con** でコンテナを起動
```
docker run --name python_con -it  --network python-mysql  -v C:\Users\Programs\docker\mysql:/var/opt  python:latest /bin/bash
```

コンテナの起動ができていることを確認
```
PS C:\Users\Programs\docker\mysql> docker container ls
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                                    NAMES
92bf3f48f0f7   python:latest   "/bin/bash"              37 minutes ago   Up 37 minutes                                                            python_con
8337420b4cad   mysql           "docker-entrypoint.s…"   38 minutes ago   Up 30 minutes   33060/tcp, 0.0.0.0:33306->3306/tcp, :::33306->3306/tcp   mysql_con
```

### 2. MySQLの設定

#### パスワード設定の変更

MySQLのバージョンが8の場合、以下の手順を実行する
1. MySQLのコンテナにログイン
1. 以下のコマンドを実行   
```
echo "default_authentication_plugin= mysql_native_password" >> /etc/mysql/my.cnf
```
1. コンテナをstopして、start(再起動)

#### データベース作成

MySQLにrootでログインしてデータベースを作成する
1. 以下のコマンドでMySQLにrootでログイン  
（パスワードはmysql_pass）  
```
mysql -u root -p
```
1. 以下のSQL文を実行  
```
CREATE DATABASE test_work;
```

### 3. Pythonの設定

#### ライブラリをインストール

Pythonコンテナで **mysql-connector-python** をインストール
```shell
pip3 install mysql-connector-python
```

### 4. Pythonのプログラムを作成・実行

#### MySQLコンテナのIPアドレスを確認  

以下のコマンドを実行して、MySQLコンテナのIPアドレスの情報が必要なので確認  
```
docker network inspect python-mysql
```

以下のように出力される。  
```
PS C:\Users\Programs\docker\mysql> docker network inspect python-mysql
[
    (略)

        "Containers": {
            "8337420b4cad7317a6916a5d61ea86cdcec34f61d87547d1d8610eba5e35ae6b": {
                "Name": "mysql_con",
                "EndpointID": "1e09387938126cebdc69203f930d0d0f06ab044c34be10f6c588396b1459b1f8",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
            "92bf3f48f0f79449798ce64b92b84ca406b56a381da9ba6eeef83ce989af528c": {
                "Name": "python_con",
                "EndpointID": "231043087d4bc176bf2d8df2961cdd31802caf7afcb0df1cec982b020d0ee7e2",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
            }
        },
    (略)
]
```
確認するのは **name** が **mysql_con** のオブジェクトの **IPv4Address** の値  
今回の場合は **172.18.0.2**

#### プログラムを作成  

MySQLにテーブルを作成するPythonプログラムを作成  
```python
# -*- coding: utf-8 -*-

import mysql.connector

db=mysql.connector.connect(host="172.18.0.2", port="3306",  user="root", password="mysql_pass")

cursor=db.cursor()

cursor.execute("USE test_work")
db.commit()
cursor.execute("""CREATE TABLE IF NOT EXISTS docker_make(
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                fruits VARCHAR(32),
                value INT);""")
db.commit()
```

##  実行結果の確認

Pythonコンテナでプログラムを実行し、MySQLコンテナでテーブルが作成されていることを確認する。  

mysqlにrootユーザーでログインして以下の文を実行
```
USE test_work;
SHOW TABLES;
```

プログラムで指定したテーブルが作成されていることが確認できた。
```
mysql> use test_work;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------+
| Tables_in_test_work |
+---------------------+
| docker_make         |
+---------------------+
1 row in set (0.01 sec)
```

## まとめ

  *  コンテナ起動時のコマンドラインの引数 _--network_ に同じネットワークに指定することで、
コンテナのIPアドレスを使用して、コンテナ間を接続できる。  

  *  今回実行したクエリはDDL（CREATE TABLE）だが、クエリであれば何でもできそう。