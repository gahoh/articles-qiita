<!--
title:   【入門】GridDB Cloud にPostmanを使ってWeb APIで触れてみよう！
tags:    GridDB,GridDBCloud,Postman,WebAPI,入門
id:      f45141ef56e90030d453
private: false
-->
記事「[【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」では、ビッグデータ・IoT向けデータベース GridDB をDB as a Serviceとして提供されるマネージドサービスである [GridDB Cloud](http://cloud.griddb.com/) の無料トライアル を入手し、Web APIでデータ登録・確認をおこなうまでの一連の手順について紹介しました。そこでは、Web APIの実行と確認 (疎通確認)には、URLシンタックスを用いてファイルを送信または受信するcurlコマンドを用いました。しかし、curlコマンドは、手軽にさまざまなプロトコルに対応したデータを転送することができたりして便利ですが、コマンドラインツールなので生産性はあまりよくないと言えます。

Postmanは、GUIでどのようなリクエストをおこなうか簡単に設定できリクエスト可能な便利なツールです。
![スライド4.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/155295db-d605-27d0-8189-18aac388246c.jpeg)
今回、curlコマンドではなく、Postmanを用いた GridDB CloudのWeb APIの実行と確認（疎通確認）の一連の手順について、はじめからていねいにまとめてご紹介します。また、Postmanのインストールから使い方までPostmanの詳細についても詳しく説明していくので、Postmanを初めて使う方にも参考になると思います。

本記事の流れは次の通りです。

- 事前準備
- 運用管理GUIへのログイン
- Postmanインストールの入手とインストール
- WebAPIの実行と確認

なお、本記事は、記事「[【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」で、curlコマンドを用いて説明している「Web APIの実行と確認（疎通確認）」の内容をcurlコマンドを用いる代わりにPostmanを用いて説明したものとなります。
よって、GridDB  Cloudの「[【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」の「[事前準備](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#%E4%BA%8B%E5%89%8D%E6%BA%96%E5%82%99)」、「[運用管理GUIへのログイン](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#%E9%81%8B%E7%94%A8%E7%AE%A1%E7%90%86gui%E3%81%B8%E3%81%AE%E3%83%AD%E3%82%B0%E3%82%A4%E3%83%B3)」を参照願います。

今回、Postmanをインストールしたデバイスの仕様とWindowsの仕様は次の通りです。

- プロセッサ Intel(R) COre(TM) i5-6300U CPU @ 2.40GHz 2.50DHz
- 実装RAM 8.00 GB
- システムの種類 64 ビット オペレーティング システム、x64 ベース　プロセッサ-
- エディション Windows 10 Pro

# 事前準備

事前準備は、GridDB Cloudのトライアルの入手です。
記事「[【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」の「[事前準備](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#%E4%BA%8B%E5%89%8D%E6%BA%96%E5%82%99)」を参照してください。

# 運用管理GUIへのログイン

ブラウザで電子メールでが送付されてきたGridDB CloudサービスのURLを開き、運用管理GUIへのログインします。
記事「[【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」の「[運用管理GUIへのログイン](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#%E9%81%8B%E7%94%A8%E7%AE%A1%E7%90%86gui%E3%81%B8%E3%81%AE%E3%83%AD%E3%82%B0%E3%82%A4%E3%83%B3)」を参照参照してください。

# Postmanの入手とインストール

次に、Postmanを入手し、インストールします。
まずは、公式ページのダウンロードページ [Download Postman](https://www.postman.com/downloads/) からダウンロードします。
![11.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/7b357ced-6ae6-f769-c393-6ba7c481d55a.png)

「Download the App」を左マウスボタンでクリックすると「Windows 32-bit」と「Windows 64-bit」の選択ができるようになるので,選択します。今回のデバイスの仕様は、「64 ビット オペレーティング システム」なので、「Windows 64-bit」を選択します。
![12.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/fa133cf8-4d71-ad2c-d4fe-7894efec1459.png)

 `Postman-win64-8.11.1-Setup.exe` のダウンロートが開始されます。
![13.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/210dc55d-1887-aae9-113b-47fc7e183238.png)

![14.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/6e1d8adb-b754-2ced-6f6b-2f00cb357ba3.png)

ダウンロードされたインストーラ `Postman-win64-8.11.1-Setup.exe` をダブルクリックすると、次のような画面が表示されインストールの実行が開始されます。
![20.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/f046c924-60b6-60c2-9e51-8bcc99dcd8c8.png)

すると、次のようなアカウント作成画面が表示されます。今回は、「Create Free Accout」を選択しました。「×」で閉じてスキップしての問題なく使用できるようです。
![22.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/7b5acd0d-ea7c-bb15-eba6-197e3ff2e7b5.png)

![23.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/a6f54e4b-f334-94f8-6843-7ab3282191a6.png)

![25.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/eda6c91e-271d-3316-394f-d1c28768d3d3.png)

メールアドレスの登録もしくはGoogleアカウントで、アカウントを作成できます。今回はメールアドレス、ユーザ名とパスワードの登録でアカウントを作成しました。
![30.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/bab5dfe7-5227-b13d-c69e-eae9ca1f8f2d.png)

「Continue」をクリックします。
![32.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/221eccc7-d65f-869c-c4fd-0aa7944df65b.png)

これでインストールは完了です。

# WebAPIの実行と確認

## 接続情報の確認

Web API のベースURLとクラスタ名を確認します。
詳細は、記事「[【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！)](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」の「[接続情報の確認](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#%E6%8E%A5%E7%B6%9A%E6%83%85%E5%A0%B1%E3%81%AE%E7%A2%BA%E8%AA%8D)」を参照してください。

### Web API のベースURLの確認

今回のWeb API のベースURLは`https://cloud1.griddb.com/trial001/griddb/v2/` とします。

### クラスタ名の確認

今回のクラスタ名は`gs_clustertrial001`とします。

## データベースユーザの作成

次にデータベースユーザを作成します。運用管理GUIで、「セキュリティ」画面に遷移し、「データベースユーザの作成」ボタンをクリックして、任意のユーザ名とパスワードを設定し、「作成」ボタンをクリックします。

ここでは、ユーザ名：「test_user1」、パスワード：「test_user1」とします。

詳細は、記事「[【入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！)](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」の「[データベースユーザの作成](https://qiita.com/gahoh/items/6c766e64c2c2c7aab81d#%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E3%83%A6%E3%83%BC%E3%82%B6%E3%81%AE%E4%BD%9C%E6%88%90)」を参照してください。

## Web APIの実行

では、本題のPostmanを用いたWeb APIの実行（疎通確認）に入ります。今回のWeb APIの実行は次のような流れでおこないます。

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

GridDB Cloud が提供するWeb APIの仕様の詳細については「[GridDB Web APIリファレンス](https://www.toshiba-sol.co.jp/pro/griddbcloud/docs-jp/v1_1/GridDB_Web_API_Reference.html?_ga=2.262652684.1202608860.1628578825-1814166202.1624951817)」を参照してください。

### Postmanの起動

PostmanでWeb APIの実行（疎通確認）するために、まずは、Postmanを起動します。Postmanを起動すると次のような画面が表示されます。
もし、表示されない場合は、「Home」をクリックしてください。
![32.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/418328b4-faec-00e6-cccc-33c4f5124d16.png)

「Get started with Postman」の「Started with something new」の「Create New > 」をクリックします。
![41.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/0da97c9f-2ca6-4f25-4491-12e846025f0d.png)

「Create New」ウィンドウで「HTTP Request」をクリックしてください。
![42.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/0551cd17-9a0b-16ca-df5a-029a7320c02a.png)

次のようなウィンドウが表示されます。
![43.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/30bc2e1d-0bca-aa77-2391-8547b7608b54.png)

上部のバーにはリクエストメソッド（デフォルトでは GET）、その右横にはリクエストURLを入力します。

次のセクション(Params/Authorization/Headers/Body/Pre-request Script/Tests/Setting)には7つのタブがあります。

- Params：リクエストに引数がある場合、引数を指定する
- Authorization：Basic AuthやOAuth2など、リクエストの認証方法を指定する
- Headers：Authorizationやcontent-typeなど、リクエストヘッダーを指定する
- Body：PostやPUTなどの場合に指定するリクエストボディーを指定する
- Pre-request Script：リクエスト前に実行するJavaScriptコードを指定する
- Tests：Postman API リクエストのテストスクリプトをJavaScriptコードで指定する
- Setting：リクエストに関する設定を指定する

準備ができたら、Sendボタンをクリックしてリクエストを開始します。

下のボタンのセクション(Response)には、レスポンスの情報（status、time、size）が表示されます。

### リクエストヘッダ

GridDBのWeb APIへの接続には次のAuthorizationとContent-Typeのヘッダが必須です。これらは、すべてのリクエストに付与します。

|  ヘッダ  |  説明  |
| ---- | ---- |
| Authorization | ベーシック認証(基本認証)でGridDBへアクセスするユーザとパスワードを指定します。<br>今回はユーザ名：test_user1とパスワード：test_user1となります|
| Content-Type |「application/json; charset=UTF-8」を指定します|

#### Authorization (認証)

Postmanは[多くの認証方式をサポート](https://learning.postman.com/docs/sending-requests/authorization/)しています。今回は、ヘッダーで設定する[ベーシック認証(基本認証)](https://learning.postman.com/docs/sending-requests/authorization/#basic-auth)を使います。

セクション「Params/Authorization/Headers/Body/Pre-request Script/Tests/Setting)」からタブ「Authorization」をクリックします。
![50.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/fc71f4f0-bda7-f597-91d6-c7fb1bbe4eda.png)

「Inherit」を左クリックし、プルダウンメニューを表示し「Basic Auth」を選択します。
![51.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/475b0084-4c8c-6cfb-486c-bd6eeb94a02a.png)

「データベースユーザの作成」で作成したユーザ名とパスワードを入力します。ここでは、「Username」に「test_user1」、「Password」に「test_user1」を入力します。
![52.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/46d8a022-ab74-8b07-f96f-d8f992b6495a.png)

#### Content-Type (コンテンツタイプ)

<!--
![60](./images/60.PNG)
![61](./images/61.PNG)
![62](./images/62.PNG)
![63](./images/63.PNG)
![64](./images/64.PNG)
![65](./images/65.PNG)
![66](./images/66.PNG)
![67](./images/67.PNG)
-->

「Headers」を左クリックします。
![68.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/c89d0c33-4195-9f41-0d83-ed474b87babf.png)

「Key」に「Content-Type」、「Value」に「application/json; charset=UTF-8」を入力します。
![69a.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/512456ae-0b89-5fc1-df30-d22697fe766e.png)

![69b.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/ba5947f5-f3a3-119b-1f12-7a16c532b381.png)

### データベース接続確認

まずは、手始めにデータベースへの接続確認をしてみましょう。データベース接続確認のHTTPトメソッドとリクエストURLは次の通りです。

>`GET ベースURL + クラスタ名/dbs/データベース名/containers/checkConnection`

今回の場合は、クラスタ名：「gs_clustertrial001」、データベース名：「public」となります。
よって、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/checkConnection
```

![53.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/cbfa022a-4ae2-610c-f615-88dfbada4d2e.png)

「Send」をクリックします。

次のようにHTTPレスポンスのステータスが200になっていれば成功です。
![54.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/62d42bb3-32d3-6076-ab9b-d0d5ec204d63.png)

## コンテナの作成

次は、TIMESTAMP型, BOOL型, DOUBLE型の3カラムで構成される時系列コンテナpoint01を作成します。

コンテナの作成のHTTPトメソッドとリクエストURLは次の通りです。

>`POST ベースURL + クラスタ名/dbs/データベース名/containers`

よって、POSTをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/containers
```

![71.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/8a9532c7-0d99-506d-161f-d17385534946.png)

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

画面で、 「Body」 を選択します。
![74.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/eecba68c-b5d1-72a9-6825-3ceecb65325d.png)
そうするとデフォルトが「none」となっているので、「raw」を選択してください。
![75.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/d19ba9c5-4ad0-8717-3db9-5058deff0043.png)
入力エリアが表れ、デフォルトの「JSON」となります。
今回の送信するリクエストボディはJSON形式なので、そのまま送信するリクエストボディを入力します。
![76.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/de4d847b-00f0-9a85-aee9-3cbadcedc9c4.png)
そして、「Send」をクリックします。
![77.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/49863150-a5cc-37b8-ae5c-cc721e6132fa.png)

「[GridDB Cloud クイックスタートガイド　5.2 コンテナの作成](https://www.toshiba-sol.co.jp/pro/griddbcloud/docs-jp/v1_1/GridDB_Cloud_QuickStartGuide.html?_ga=2.176392486.226577107.1630393155-1814166202.1624951817&_gac=1.251793275.1629693042.CjwKCAjw64eJBhAGEiwABr9o2IVtMxFQKPSoTGdmgxtuhOOB3duKhEGiTOXT-xzVVRGpM-8BCURTuBoC9qsQAvD_BwE#section-19)」では、「HTTPレスポンスのステータスが200(OK)になっていれば成功です。」と記述されていますが、実際は、HTTPレスポンスのステータスが201になっていれば成功です。

## コンテナ一覧取得

コンテナ一覧取得のHTTPトメソッドとリクエストURLは次の通りです。

>`GET ベースURL + クラスタ名/dbs/データベース名/containers/`

よって、GETをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/containers?limit=10&offset=0
```

![78.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/826947ba-6fed-7b9d-abe5-91f01aa847f3.png)

「Body」は「none」を指定します。
![78a.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/0584d16c-f51b-8162-2a63-ab5109a6d5a3.png)

「Send」をクリックします。
「[GridDB Web APIリファレンス 1.7 コンテナ一覧取得](https://www.toshiba-sol.co.jp/pro/griddbcloud/docs-jp/v1_1/GridDB_Web_API_Reference.html?_ga=2.117083919.84295391.1630990578-1814166202.1624951817&_gac=1.79816293.1629693042.CjwKCAjw64eJBhAGEiwABr9o2IVtMxFQKPSoTGdmgxtuhOOB3duKhEGiTOXT-xzVVRGpM-8BCURTuBoC9qsQAvD_BwE#section-16)」 では、「limitで0を指定した場合は、条件に合致するすべてのロウが返ります。」と記載されていますが、limit 0を指定した場合、取得数が0になるため、コンテナ名は取得されないようです。

![79.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/d6d88b0b-f5f4-bf4e-2489-f78d3bb3e442.png)

コンテナ一覧取得が実行されました。

## コンテナ情報取得

コンテナやテーブルの情報を取得してみましょう。

コンテナ情報取得のHTTPトメソッドとリクエストURLは次の通りです。

>`GET ベースURL + クラスタ名/dbs/データベース名/containers/コンテナ名/info`

よって、GETをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/containers/point01/info
```

![80.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/6ec116ef-f6ce-243c-2c19-dbaed1d7503b.png)


「Send」をクリックします。
![81.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/69d33fa3-5368-8ab1-d575-ca140faa5b7b.png)

レスポンスボディに次のようなコンテナ情報が取得でき、HTTPレスポンスのステータスが200になっていれば成功です。

```
{
  "container_name":"point01",
  "container_type":"TIME_SERIES",
  "rowkey":true,
  "columns":[
    {
      "name":"timestamp",
      "type":"TIMESTAMP",
      "index":[]
    },
    {
      "name":"active",
      "type":"BOOL",
      "index":[]
    },
    {
      "name":"voltage",
      "type":"DOUBLE",
      "index":[]
    }
  ]
}
```

## ロウの登録

次に、作成したコンテナpoint01にロウを登録します。

登録するロウは、JSON形式で指定します。ひとつのコンテナに複数のロウを登録することもできます。

ロウの登録ののHTTPトメソッドとリクエストURLは次の通りです。

>`PUT ベースURL + クラスタ名/dbs//データベース名/containers/コンテナ名/rows`

今回、例として次のリクエストボディを送信します。

```
[
["2021-06-27T10:30:00.000Z", false, "100"],
["2021-06-28T10:30:00.000Z", false, "100"]
]
```

よって、PUTをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/containers/point01/rows
```

![85.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/218383e0-53f1-68a9-cd0a-39c95310d0ae.png)

今回はHTTPレスポンスのステータスが200になっていて、レスポンスボディが {"count":2} のようになっていれば成功です。

## ロウの取得

次に、コンテナpoint01に登録したロウを取得します。

ロウの取得のHTTPトメソッドとリクエストURLは次の通りです。

>`POST ベースURL + クラスタ名/dbs//データベース名/containers/コンテナ名/rows`

よって、PUTをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/containers/point01/rows
```

今回、取得対象は最大100件で、2021年6月28日 4時30分以降のデータを取得するという条件でロウの取得をします。
timestampは時刻型なので、TQLの時刻型演算関数 TIMESTAMP(str)で時刻の文字列表現を時刻型に変換して指定します。
リクエストボディは次の通りです。

```
{
  "limit"  : 100,
  "condition" : "timestamp >= TIMESTAMP('2021-06-28T04:30:00.000Z')"
}
```

実行すると次のような画面になります。

![87.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/4cb3c7dd-7244-9fe0-ee41-0b85ed3b79ed.png)


## TQL実行

次にTQL文を実行します。

TQL実行のHTTPトメソッドとリクエストURLは次の通りです。

>`POST ベースURL + クラスタ名/dbs/データベース名/tql`

よって、POSTをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/tql
```

今回、コンテナpoint01に対して TQL文 `select * ` を実行します。

```
[
  {"name" : "point01", "stmt" : "select * "}
]
```

![88.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/8de27a8c-49fd-da23-0d34-6ae0d22abbcc.png)



## ロウ削除

ロウ削除のHTTPトメソッドとリクエストURLは次の通りです。

> `DELETE ベースURL + クラスタ名/dbs//データベース名/containers//コンテナ名/rows`

ロウキー "2021-06-28T10:30:00.000Z"を指定してロウを削除してみます。

```
["2021-06-28T10:30:00.000Z]
```

よって、DELETEをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/containers/point01/rows
```

![89.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/117e62f6-e2c9-6bd3-756f-f7c9d820c668.png)


HTTPレスポンスのステータスが204になっていれば成功です。

### SQL SELECT文実行

実際、ロウキー "2021-06-28T10:30:00.000Z"のロウが削除されたか確認するために、SQL SELECT文 "select * from point01"を実行してみます。

SQL SELECT文実行のHTTPトメソッドとリクエストURLは次の通りです。

>`POST ベースURL + クラスタ名/dbs/データベース名/sql/select`

よって、POSTをチョイスし、「Enter request」に次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/sql
```

SQL SELECT文を次のようなJSON形式で指定します。

```
[
  {"type" : "sql-select", "stmt" : "select * from point01"}
]
```

![90.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/577a61ed-a637-771e-3639-58d8a99a5964.png)

HTTPレスポンスのステータスが200になっていれば成功です。

## コンテナ削除

では、最後に今回Web APIの疎通確認するために作成したコンテナ (コンテナ名:point01)を削除します。

コンテナ削除のHTTPトメソッドとリクエストURLは次の通りです。

>`DELETE ベースURL + クラスタ名/dbs/データベース名/containers`

よって、DELETEをチョイスし、Enter requestに次のように入力します。

```
https://cloud1.griddb.com/trial001/griddb/v2/gs_clustertrial001/dbs/public/containers
```

削除するコンテナ名として次のリクエストボディを 送信します。

```
["point01"]
```

![91.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/c7100935-08b5-319c-e959-de514628a0a0.png)

HTTPレスポンスのステータスが204になっていれば成功です。しかし、GridDB Cloud v1.1では、削除するコンテナ名の指定がタイプミスで誤っていたりして存在しないコンテナを指定しても、現状HTTPレスポンスのステータスが204になってしまうようです。注意が必要です。

# さいごに

ビッグデータ・IoT向けデータベース GridDB をDB as a Serviceとして提供されるマネージドサービスである [GridDB Cloud](http://cloud.griddb.com/) の無料トライアルを入手し、Postmanを用いて、Web APIでデータ登録・確認をおこなう一連の動作を確かめました。
今回の動作確認に用いたPostman の基本機能はほんの一部です。 Postmanには今回用いた機能以外、便利な機能がまだたくさんあります。

なお、記述について誤りがあったり、気になることがあれば、編集リクエストやコメントでフィードバックしていただけると助かります。