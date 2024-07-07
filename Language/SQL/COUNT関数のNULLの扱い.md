# COUNT関数のNULLの扱い

## 概要

count(*)とcount(列名)でNULLの扱いが違う

|引数 |NULLの扱い |
|--|--|
| count(*) | 数える |
| count(列名) | 数えない |


## 実機確認

以下のテーブルに、countを実行する。  
```
mysql> SELECT * FROM number_tbl;
+------+
| val  |
+------+
|    1 |
|    3 |
|    5 |
| NULL |
+------+
4 rows in set (0.00 sec)
```

実行結果は以下。  
```
mysql> SELECT count(*) FROM number_tbl;
+----------+
| count(*) |
+----------+
|        4 |
+----------+
1 row in set (0.00 sec)

mysql> SELECT count(val) FROM number_tbl;
+------------+
| count(val) |
+------------+
|          3 |
+------------+
1 row in set (0.00 sec)
```