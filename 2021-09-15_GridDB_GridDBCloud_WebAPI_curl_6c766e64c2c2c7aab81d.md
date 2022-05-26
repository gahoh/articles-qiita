<!--
title:   【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！
tags:    GridDB,GridDBCloud,WebAPI,curl,入門
id:      6c766e64c2c2c7aab81d
private: false
-->
> 2022年3月GridDB Cloud v1.3がリリースされ、今回GridDB Cloud v1.3のトライアルに触れる機会を得ましたので、GridDB Cloud v1.3をベースに内容を見直しました。
>
> なお、改訂履歴は次の通りです。<br>
> - 2022年3月 改訂 GridDB Cloud v1.3 のトライアルをベースに改訂
> - 2021年11月 改訂 GridDB Cloud v1.2 のトライアルをベースに改訂
> - 2021年09月 新規 GridDB Cloud v1.1 のトライアルをベースにして新規作成

記事「[【入門】GridDB Cloud に VNetを使って触れてみよう！](2021-05-14_Azure_GridDB_GridDBCloud_8dc8d81eec89a7c1bed1.md)」では、[GridDB Cloud](http://cloud.griddb.com/) の 無料トライアル を入手し、払い出された後に管理コンソール (以下、運用管理GUI) へログインするところから、Azureアカウントの保持を前提としたVNetピアリングを使って接続しサンプルプログラム (Java) でGridDBへのデータ登録・確認をおこなうまでの一連の手順を紹介しました。

GridDB Cloud v1.1以降では、VNET環境 だけではなく、WebAPIを通してGridDBと接続・通信ができるようになりました。これにより、Azureのアカウントを保持していないなど、VNET環境を利用できない場合もユーザアプリケーションとの連携ができるようになりました。

![スライド3.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/681310dc-11a9-b8ba-ede5-6693913ebf73.jpeg)

今回、GridDB Cloud v1.3の無料トライアルを入手し、Web APIでGridDBへのデータ登録・確認をおこなうまでの一連の手順について、はじめからていねいにまとめて、ご紹介します。
また、Web APIの実行には、URLシンタックスを用いてファイルを送信または受信する curlコマンドを使用します。curlコマンドについても詳しく説明していくので、初心者でWeb APIを実行して確認したいという方は、是非参考にしていただければ幸いです。

また、GridDB Cloud が提供するWeb APIの仕様について詳しく知りたい場合は、「[GridDB Web APIリファレンス](https://www.toshiba-sol.co.jp/pro/griddbcloud/docs-jp/v1_2/ja_md_reference_web_api/md_reference_web_api.html)」 を参照してください。

本記事の流れは次の通りです。

- 事前準備
- 運用管理GUIへのログイン
- Web APIの実行と確認（疎通確認）

なお、記事「[【入門】GridDB Cloud に VNetを使って触れてみよう！](2021-05-14_Azure_GridDB_GridDBCloud_8dc8d81eec89a7c1bed1.md)」をすでに読んで試していただいた方は「[事前準備](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#事前準備)」、「[運用管理GUIへのログイン](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#運用管理guiへのログイン)」を飛ばして、「[Web APIの実行と確認](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#web-apiの実行と確認)」から始めていただければと思います。

また、「[事前準備](https://qiita.com/gahoh/items/8dc8d81eec89a7c1bed1#事前準備)」の「[無料トライアルの申し込み](https://qiita.com/gahoh/items/8dc8d81eec89a7c1bed1#無料トライアルの申し込み)」で、送付されてくる電子メールのGridDB Cloudのトライアルアカウント情報のWeb APIのURLは、後で記述するの **「ベースURL」** になります。

# Web APIの実行と確認

## 接続情報の確認

Web API のベースURLとクラスタ名を確認します。

### Web API のベースURLの確認

Web API の **ベースURL** は、送付されてきた電子メールに記載されているので確認しメモしてください。

 例：

```
https://cloud1.griddb.com/trial1301/griddb/v2/
```

### クラスタ名の確認

Web API の URLに必要な、クラスタ名を確認します。「クラスタ」画面に遷移し、次の接続情報をメモしてください。

クラスタ名 （クラスタ詳細の左上）
![410.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/83f6dfbd-4356-5a59-c03c-f8f96ef1fbde.png)

今回の画面の例では、クラスタ名は「gs_clustertrial1301」となります。

## データベースユーザの作成

次にデータベースユーザを作成します。「GridDBユーザ」画面に遷移し、「データベースユーザの作成」ボタンをクリックしてください。
![431.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/20b2b55c-815e-53c2-2fa4-f3c4ab988bf6.png)

任意のユーザ名とパスワードを設定し、「作成」ボタンをクリックします。
![432.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/273bffd4-0200-d32a-31dd-9daf34b5c33b.png)

ここでは、：ユーザ名：「test_user1」、パスワード：「test_user1」としました。
![433.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/ac2e63b8-8a0f-8ad5-86fd-704d7954dda6.png)

「作成」をクリックします。
![444.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/46dc2c7d-4ea2-3a28-346b-68e20e31f80c.png)

データベースユーザ名とそのパスワードが設定されました。

## Web APIの実行

では、本題のWeb APIの実行（疎通確認）に入ります。今回のWeb APIの実行は次のような流れでおこないます。

1. データベース(デフォルトデータベースpublic)への接続の確認
2. コンテナ (コンテナ名:point01) を作成<br>TIMESTAMP, BOOL, DOUBLEの3カラムで構成
3. コンテナ一覧を取得
4. 作成したコンテナの情報を取得
5. ロウにデータを登録　<br>"2021-06-27T10:30:00.000Z", false, "100" <br>"2021-06-28T10:30:00.000Z", false, "100"
6. 最大100件で、2021年6月28日 4時30分以降のデータを取得するという条件でロウの取得
7. TQL実行
8. "2021-06-28T10:30:00.000Z"のロウを削除
9. SQL SELECTを文実行
10. コンテナ (コンテナ名:point01) を削除

GridDB Cloud が提供するWeb APIの仕様の詳細については「[GridDB Web APIリファレンス](https://www.toshiba-sol.co.jp/pro/griddb/docs-jp/v4_6/GridDB_Web_API_Reference.html)」を参照してください。

Web APIの実行には、URLシンタックスを用いてファイルを送信または受信するコマンドラインツール curlを使用します。 今回利用したcurlコマンドのバージョンは次の通りです。

```
$ curl -V
curl 7.61.1 (x86_64-redhat-linux-gnu) libcurl/7.61.1 OpenSSL/1.1.1g zlib/1.2.11 brotli/1.0.6 libidn2/2.2.0 libpsl/0.20.2 (+libidn2/2.2.0) libssh/0.9.4/openssl/zlib nghttp2/1.33.0
Release-Date: 2018-09-05
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz brotli TLS-SRP HTTP2 UnixSockets HTTPS-proxy PSL Metalink
$
```
なお、今回使用するcurlコマンドのオプションは次の通りです。

|  オプション  |  説明  |
| ---- | ---- |
| -d 'データ'|リクエストメソッドがPOST時にデータを指定する|
| -H |リクエストヘッダにその情報を追加もしくは変更する 。POSTのフォーマットがJSONの場合は -H "Content-Type: application/json" という指定をする|
| -i |HTTPヘッダを出力に含める |
| -u ユーザ名:パスワード |認証に用いるユーザ名とパスワードを送信します。デフォルトは、BASIC認証。|
| -X リクエストメソッド|GET、POST、PUT、DELETEなどのリクエストメソッドを指定する。<br>GETリクエストの際はXオプションを省略することができます。|
| -V |バージョンを表示する|
| -v |デバッグ: リクエストヘッダとレスポンスヘッダを標準エラー出力に表示する。|

### リクエストヘッダ

GridDBのWeb APIへの接続には次のContent-TypeとAuthorizationのヘッダが必須です。すべてのリクエストに付与してください。

|  ヘッダ  |  説明  |
| ---- | ---- |
| Content-Type |「application/json; charset=UTF-8」を指定します
| Authorization |GridDBへアクセスするユーザとパスワードをBasic認証の形式 (BASE64でエンコードした値) で指定します。<br>例えばユーザ名：test_user1とパスワード：test_user1の場合、「Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx」となります。|

##### :beginner:  補足 :beginner:

BASE64でエンコードした値を生成するには次のように行います。

> `echo "Basic $(echo -n ユーザ名:パスワード | base64)"`

```
echo "Basic $(echo -n test_user1:test_user1 | base64)"
```

では、実行してみます。

```
echo "Basic $(echo -n test_user1:test_user1 | base64)"
$ Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx
```

#### BASIC認証

Authorizationヘッダを使わず、同じことを curl コマンドオプションのオプション -u を使ってもできます。「:パスワード」を省略すると、curlコマンド実行時にプロンプトが出力されてパスワードを聞かれます。

### データベース接続確認

まずは、手始めにデータベースへの接続確認をしてみましょう。データベース接続確認のリクエスト送信先のURLは次の通りです。

>`GET ベースURL + クラスタ名/dbs/データベース名/containers/checkConnection`

今回の場合は、クラスタ名：「gs_clustertrial001」、データベース名：「public」となります。なお、GETリクエストメソッドの場合 `-X GET` を省略することができます。今後、省略します。

curlコマンドでを用いて、リクエストヘッダの認証情報をechoBasic認証の形式 (BASE64でエンコードした値) `-H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx'` で送信し、データベース接続確認をする指定方法は次の通りです。

```
curl -i -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/checkConnection
```

`-H 'Authorization:Basic $(echo -n test_user1:test_user1 | base64)'`のようにechoコマンドでBasic認証の形式 (BASE64でエンコードした値)に変換して、リクエストヘッダの認証情報を送信し データベース接続確認をする指定方法は次の通りです。

```
curl -i -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic $(echo -n test_user1:test_user1 | base64)' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/checkConnection
```

curlコマンドでを用いて、`-u test_user1:test_user1` でサーバー認証の送信し、データベース接続確認をする指定方法は次の通りです。

```
curl -i -H 'Content-Type:application/json;charset=UTF-8' -u test_user1:test_user1 https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/checkConnection
```

では、リクエストヘッダの認証情報をBasic認証の形式 (BASE64でエンコードした値) で送信してみます。

```
$ curl -i -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic
dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/checkConnection
HTTP/1.1 200
Date: Wed, 30 Mar 2022 01:42:52 GMT
Content-Length: 0
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

$
```

HTTPレスポンスのステータスが200になっていれば成功です。


### コンテナの作成

次は、TIMESTAMP型, BOOL型, DOUBLE型の3カラムで構成される時系列コンテナpoint01を作成します。

コンテナの作成のリクエスト送信先のURLは次の通りです。

>`POST ベースURL + クラスタ名/dbs/データベース名/containers`

ここでは、TIMESTAMP型, BOOL型, DOUBLE型の3カラムで構成される時系列コンテナpoint01を作成しますので、送信するリクエストボディは次の通りです。

```
{
  "container_name" : "point01",
  "container_type" : "TIME_SERIES",
  "rowkey" : true,
  "columns" : [
    {"name": "timestamp", "type": "TIMESTAMP"},
    {"name": "active", "type": "BOOL"},
    {"name": "voltage", "type": "DOUBLE"}
  ]
}
```

curlコマンドを用いた、TIMESTAMP型, BOOL型, DOUBLE型の3カラムで構成される時系列コンテナpoint01の作成の指定は次の通りです。

```
curl -i -X POST -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers -d '
{
  "container_name" : "point01",
  "container_type" : "TIME_SERIES",
  "rowkey" : true,
  "columns" : [
    {"name": "timestamp", "type": "TIMESTAMP"},
    {"name": "active", "type": "BOOL"},
    {"name": "voltage", "type": "DOUBLE"}
  ]
}
'
```

では、実際に curlコマンドでコンテナの作成を実行してみましょう。

```
$ curl -i
n:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers -d '
>
> {
>
>   "container_name" : "point01",
>
>   "container_type" : "TIME_SERIES",
>
>   "rowkey" : true,
>
>   "columns" : [
>
>     {"name": "timestamp", "type": "TIMESTAMP"},
>
>     {"name": "active", "type": "BOOL"},
>
>     {"name": "voltage", "type": "DOUBLE"}
>
>   ]
>
> }
>
> '
HTTP/1.1 201
Date: Wed, 30 Mar 2022 04:34:07 GMT
Content-Length: 0
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

$
```

HTTPレスポンスのステータスが201になっていれば成功です。

### コンテナ一覧取得

コンテナ一覧取得のリクエスト送信先のURLは次の通りです。

>`GET ベースURL + クラスタ名/dbs/データベース名/containers/`

curlコマンドを用いた、コンテナ一覧取得の指定は次の通りです。

```
curl -i -X GET -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers?limit=10\&offset=0
```

では、コンテナ一覧取得を実行してみましょう。

```
$ curl -i -X GET -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization
:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers?limit=10\&offset=0
HTTP/1.1 200
Date: Wed, 30 Mar 2022 04:37:06 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

{"names":["point01"],"total":1,"offset":0,"limit":10}
$
```

### コンテナ情報取得

コンテナやテーブルの情報を取得してみましょう。

コンテナ情報取得のリクエスト送信先のURLは次の通りです。

>`GET ベースURL + クラスタ名/dbs/データベース名/containers/コンテナ名/info`

curlコマンドを用いた、コンテナ情報取得の指定は次の通りです。

```
curl -i -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/info
```

では、実際に curlコマンドでコンテナ情報取得を実行してみましょう。

```
$ curl -i -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic
dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/info
HTTP/1.1 200
Date: Wed, 30 Mar 2022 04:39:34 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

{"container_name":"point01","container_type":"TIME_SERIES","rowkey":true,"columns":[{"name":"timestamp","type":"TIMESTAMP","index":[]},{"name":"ac
tive","type":"BOOL","index":[]},{"name":"voltage","type":"DOUBLE","index":[]}]}
$
```

HTTPレスポンスのステータスが200になっていれば成功です。

### ロウの登録

次に、作成したコンテナpoint01にロウを登録します。
登録するロウは、JSON形式で指定します。ひとつのコンテナに複数のロウを登録することもできます。

ロウの登録のリクエスト送信先のURLは次の通りです。

>`PUT ベースURL + クラスタ名/dbs//データベース名/containers/コンテナ名/rows`

今回、例として次のリクエストボディを送付します。

```
[
["2021-06-27T10:30:00.000Z", false, "100"],
["2021-06-28T10:30:00.000Z", false, "100"]
]
```

curlコマンドを用いた、ロウの今回のロウの登録の指定は次の通りです。

```
curl -i -X PUT -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/rows -d '
[
  ["2021-06-27T10:30:00.000Z", false, "100"],
  ["2021-06-28T10:30:00.000Z", false, "100"]
]
'
```

では、実際に curlコマンドでロウの登録を実行してみましょう。

```
$ curl -i -X PUT -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization
:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/rows -d '
>
> [
>
>   ["2021-06-27T10:30:00.000Z", false, "100"],
>
>   ["2021-06-28T10:30:00.000Z", false, "100"]
>
> ]
>
> '
HTTP/1.1 200
Date: Wed, 30 Mar 2022 04:42:24 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

{"count":2}
```

今回はHTTPレスポンスのステータスが200になっていて、レスポンスボディが `{"count":2}` のようになっていれば成功です。

### ロウの取得

次に、コンテナpoint01に登録したロウを取得します。

リクエスト送信先のURLは次の通りです。

>`POST ベースURL + クラスタ名/dbs//データベース名/containers/コンテナ名/rows`

今回、取得対象は最大100件で、2021年6月28日 4時30分以降のデータを取得するという条件でロウの取得をします。
timestampは時刻型なので、TQLの時刻型演算関数 TIMESTAMP(str)で時刻の文字列表現を時刻型に変換して指定します。
リクエストボディは次の通りです。

```
{
  "limit"  : 100,
  "condition" : "timestamp >= TIMESTAMP('2021-06-28T04:30:00.000Z')"
}
```

curlコマンドでのオプション -d で　リクエストボディを指定する場合、次のように
　①：ダブルクォーテーション部分をエスケープ ( " ⇒ \\" )
　②：シングルクォーテーション部分をダブルクォーテーションに修正 ( ‘ ⇒ " )
　　　※②のダブルクォーテーションにエスケープは不要です。
とします。

```
curl -i -X POST -H 'Content-Type: application/json;charset=UTF-8' -H 'Authorization: Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/rows -d "
{
  \"limit\"  : 100,
  \"condition\" : \"timestamp >= TIMESTAMP('2021-06-28T04:30:00.000Z')\"
}
"
```

では、実際に curlコマンドでロウの取得を実行してみましょう。

```
$ curl -i -X POST -H 'Content-Type: application/json;charset=UTF-8' -H 'Authorizati
on: Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/rows -d "
>
> {
>
>   \"limit\"  : 100,
>
>   \"condition\" : \"timestamp >= TIMESTAMP('2021-06-28T04:30:00.000Z')\"
>
> }
>
> "
HTTP/1.1 200
Date: Wed, 30 Mar 2022 04:43:43 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

{"columns":[{"name":"timestamp","type":"TIMESTAMP"},{"name":"active","type":"BOOL"},{"name":"voltage","type":"DOUBLE"}],"rows":[["2021-06-28T10:30
:00.000Z",false,100.0]],"offset":0,"limit":100,"total":1}
$
```

また、POSTするリクエストボディをreqbody.txtなどのテキストファイルに書き込み、標準入力から取り込ませてそのままPOSTさせるには、次のように指定できます。

```
cat reqbody.txt | curl -i -X POST -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/rows -d @-
```

データをファイルで指定したい場合はファイル名の先頭に @ を付けて、 @reqbody.txt のように指定できます。ファイルではなく標準入力を指定するには 上のように @- とします。

### TQL実行

次にTQL文を実行します。

TQL実行のリクエスト送信先のURLは次の通りです。

>`POST ベースURL + クラスタ名/dbs/データベース名/tql`

今回、コンテナpoint01に対して TQL文 `select * ‘を実行します。

```
[
  {"name" : "point01", "stmt" : "select * "}
]
```

curlコマンドを用いた、今回のTQL実行の指定は次の通りです。

```
curl -i -X POST -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorizatio
n:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/tql -d '
>
> [
>
>   {"name" : "point01", "stmt" : "select * "}
>
> ]
>
> '
```

では、実際に curlコマンドでTQLを実行してみましょう。

```
$ curl -i -X POST -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorizatio
n:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/tql -d '
>
> [
>
>   {"name" : "point01", "stmt" : "select * "}
>
> ]
>
> '
HTTP/1.1 200
Date: Wed, 30 Mar 2022 04:46:00 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

[{"columns":[{"name":"timestamp","type":"TIMESTAMP"},{"name":"active","type":"BOOL"},{"name":"voltage","type":"DOUBLE"}],"results":[["2021-06-27T1
0:30:00.000Z",false,100.0],["2021-06-28T10:30:00.000Z",false,100.0]],"offset":0,"limit":10000,"total":2}]$
```


### ロウ削除

では、ロウキー "2021-06-28T10:30:00.000Z"を指定してロウを削除してみます。

> `DELETE ベースURL + クラスタ名/dbs/データベース名/containers//コンテナ名/rows`

curlコマンドを用いた、ロウキー "2021-06-28T10:30:00.000Z"のロウの削除の指定は次の通りです。

```
curl -i -X DELETE -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/rows -d "[\""2021-06-28T10:30:00.000Z"\"]"
```

では、curlコマンドで実際に削除してみます。

```
$ curl -i -X DELETE -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorizat
ion:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers/point01/rows -d "[
\""2021-06-28T10:30:00.000Z"\"]"
HTTP/1.1 204
Date: Wed, 30 Mar 2022 04:49:06 GMT
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

$
```

HTTPレスポンスのステータスが204になっていれば成功です。


### SQL SELECT文実行

実際、ロウキー "2021-06-28T10:30:00.000Z"のロウが削除されたか確認するために、SQL SELECT文 "select * from point01"を実行してみます。

>`POST ベースURL + クラスタ名/dbs/データベース名/sql/select`

SQL SELECT文を次のようなJSON形式で指定します。

```
[
  {"type" : "sql-select", "stmt" : "select * from point01"}
]
```

curlコマンドを用いた、SQL SELECT文 "select * from point01の指定は次の通りです。

```
curl -i -X POST -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/sql -d '
[
  {"type" : "sql-select", "stmt" : "select * from point01"}
]
'
```

では、curlコマンドで実際に実行してみます。

```
$  curl -i -X POST -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorizatio
n:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/sql -d '
>
> [
>
>   {"type" : "sql-select", "stmt" : "select * from point01"}
>
> ]
>
> '
HTTP/1.1 200
Date: Wed, 30 Mar 2022 04:49:58 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

[{"columns":[{"name":"timestamp","type":"TIMESTAMP"},{"name":"active","type":"BOOL"},{"name":"voltage","type":"DOUBLE"}],"results":[["2021-06-27T1
0:30:00.000Z",false,100.0]]}]
```

HTTPレスポンスのステータスが200になっていれば成功です。

### コンテナ削除

 では、最後に今回Web APIの疎通確認するために作成したコンテナ (コンテナ名:point01)を削除します。

>`DELETE ベースURL + クラスタ名/dbs/データベース名/containers`

curlコマンドを用いた、コンテナ (コンテナ名:point01)の削除の指定は次の通りです。

```
curl -i -X DELETE -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorization:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers -d "[\"point01\"]"
```

では、実際に curlコマンドを用いて実行してみましょう。

```
$ curl -i -X DELETE -H 'Content-Type:application/json;charset=UTF-8' -H 'Authorizat
ion:Basic dGVzdF91c2VyMTp0ZXN0X3VzZXIx' https://cloud1.griddb.com/trial1301/griddb/v2/gs_clustertrial1301/dbs/public/containers -d "[\"point01\"]"
HTTP/1.1 204
Date: Wed, 30 Mar 2022 04:50:56 GMT
Connection: keep-alive
Server: Apache/2.4.6 (CentOS)

$
```

HTTPレスポンスのステータスが204になっていれば成功とのことです。しかし、削除するコンテナ名の指定がタイプミスで誤っていたりして存在しないコンテナを指定しても、現状HTTPレスポンスのステータスが204になってしまうようです。


# さいごに

今回、ビッグデータ・IoT向けデータベース GridDB をDB as a Serviceとして提供されるマネージドサービスである [GridDB Cloud](http://cloud.griddb.com/) の無料トライアルを入手し、コマンドラインツール curlを使用をしたWeb APIの実行（疎通確認）についての一連の手順を詳しく紹介しました。

コマンドラインツール curlではなく、Postmanを用いた GridDB CloudのWeb APIの実行と確認（疎通確認）の一連の手順については、 [【入門】GridDB Cloud にPostmanを使ってWeb APIで触れてみよう！](2021-10-13_GridDB_GridDBCloud_Postman_WebAPI_f45141ef56e90030d453.md) を参照していただければと思います。

なお、記述について誤りがあったり、気になることがあれば、編集リクエストやコメントでフィードバックしていただけると助かります。

------