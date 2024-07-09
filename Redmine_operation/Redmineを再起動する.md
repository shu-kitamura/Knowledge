# Redmineを再起動する

{{toc}}

## 概要

設定変更を反映するなど、Redmineを再起動が必要になることがある。  
再起動の方法を記載する。

## 方法

*  apache2を再起動する。
``` shell
$ sudo systemctl restart apache2
```

*  restart.txtを作成して、Redmineにアクセスする。　　
``` shell
$ touch /var/lib/redmine/tmp/restart.txt
```
restart.txtは再起動後に自動で削除される。
（削除されない場合があるらしいので、その際は手動で削除する）

## 参考  

  *  https://www.redmine.org/boards/2/topics/15827
  *  https://chusotsu-program.com/redmine-restart/