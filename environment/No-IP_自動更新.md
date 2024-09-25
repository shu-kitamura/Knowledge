# curl と cron で No-IP の IPアドレスを自動更新

## 概要

自宅でサーバを運用している際に、IPアドレスが動的に変更されることがあります。  
常に同じホスト名でアクセスできるようにするため、Dynamic DNS サービスの [No-IP](https://www.noip.com/) を使用しています。  
No-IP では定期的にIPアドレスの更新が必要ですが、それを自動で更新できるようにした方法を記載します。  

## 自動更新方法

No-IP では、IPアドレスを更新するための Web API が提供されています。  
参考 : https://www.noip.com/integrate/request

curl コマンドで Web API を叩くシェルスクリプトを作成し、IPアドレスを更新するようにしました。  
また、cron でシェルスクリプトを定期的に実行することで、IPアドレスを自動で更新することができました。  

### シェルスクリプト  

以下のシェルスクリプトを作成しました。  

```
#!/bin/sh 

USERNAME="YOUR USERNAME" # No-IPのユーザ名
PASSWORD="YOUR PASSWORD" # ユーザのパスワード
HOSTNAME="YOUR HOSTNAME" # 更新するホスト名
URL="https://$USERNAME:$PASSWORD@dynupdate.no-ip.com/nic/update?hostname=$HOSTNAME"

curl $URL
```

`YOUR USERNAME`, `YOUR PASSWORD`, `YOUR HOSTNAME`を各自の環境に合わせて変更してください。

### cron の設定  

毎日 00:00 にシェルスクリプトを実行するように設定しました。  
ファイルパスは各自の環境に合わせて変更してください。  

```
0 0 * * * root /home/user/noip/update_address.sh
```


