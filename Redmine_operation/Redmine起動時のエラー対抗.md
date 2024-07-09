# Redmine起動時のエラー対応

## 概要

Redmineを再起動した際に、http://192.168.3.15/redmine にはアクセスできたが、以下の文字が表示された。
> We're sorry, but something went wrong.

正常に戻した際に実施した対処を記載する。  

## 対処

_/var/log/apache2/error.log_ から、エラーの内容を確認した。  

以下のエラーが出力されていた。  
> [ E 2024-05-20 00:59:06.9385 165112/Tk age/Cor/App/Implementation.cpp:221 ]: Could not spawn process for application /var/lib/redmine: The application encountered the following error:   * No such file or directory @ rb_sysopen - /var/log/redmine/redmine.log  *  (Errno::ENOENT)

_/var/log/redmine/redmine.log_ が存在しないというエラー。  

ディレクトリ・ファイルを作成。
```
$ mkdir /var/log/redmine
$ touch /var/log/redmine/redmine.log
```

ディレクトリ・ファイルの作成後、エラーが以下に変わった。  

> [ E 2024-05-20 01:01:34.1368 165112/Tp age/Cor/App/Implementation.cpp:221 ]: Could not spawn process for application /var/lib/redmine: The application encountered the following error:   * Permission denied @ rb_sysopen - /var/log/redmine/redmine.log  *  (Errno::EACCES)

_/var/log/redmine/redmine.log_ への権限が足りないというエラー。

shusei（redmineプロセスの実行ユーザ）に権限を付与。
```
$ sudo chown shusei /var/log/redmine
$ sudo chown shusei /var/log/redmine/redmine.log
```

## まとめ

今回のエラーは
  *  ログを出力するよう設定したが、ディレクトリ・ファイルを作成していなかったこと  
  *  作成したディレクトリ・ファイルの権限設定が正しくなかったこと

が原因だった。  

## 参考  

  *  https://blog.nekonium.com/redmine-wrong-for-gemfile-lock/  
（発生した事象は違うが、見たログファイルは同じ）  