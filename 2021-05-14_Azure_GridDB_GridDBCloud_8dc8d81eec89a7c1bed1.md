<!--
title:   【入門】GridDB Cloud に VNetを使って触れてみよう！
tags:    Azure,GridDB,GridDBCloud,入門,初心者
id:      8dc8d81eec89a7c1bed1
private: false
-->
> 2022年3月GridDB Cloud v1.3がリリースされ、今回GridDB Cloud v1.3のトライアルに触れる機会を得ましたので、GridDB Cloud v1.3をベースに内容を見直しました。
>
> なお、改訂履歴は次の通りです。<br>
> - 2022年3月 改訂 GridDB Cloud v1.3 のトライアルをベースに改訂
> - 2021年11月 改訂 GridDB Cloud v1.2 のトライアルをベースに改訂
> - 2021年08月 改訂 GridDB Cloud v1.1 のトライアルをベースに改訂
> - 2021年05月 新規 GridDB Cloud v1.0 のトライアルをベースにして新規作成

ビッグデータ・IoT向けデータベース GridDB をDataBase as a Serviceとして提供されるマネージドサービスである [GridDB Cloud](http://cloud.griddb.com/) の 無料トライアル を試してみました。

アプリケーションとGridDBの間をインターネットを介すWeb APIによる接続の方法では、通信の暗号化が必須となり、ネットワークオーバヘッドが大きくなるため、GridDBの特長である高い処理能力を十分に生かすことができません。そこで、下図のようにGridDB Cloudでは、Web APIによる接続方法だけではなく、GridDBとアプリケーションをAzure 仮想プライベートネットワークであるVNet（Azure Virtual Network）ピアリングでも接続することが可能となっています。
![azure.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/b4baae38-364f-6a29-ccc8-3e4e97a392b0.jpeg)

VNetピアリングでAzureのバックボーンを用いて直接接続すると。アプリケーションのVNetとGridDBのVNetが直接接続され、セキュリティが確保できるため、通信の暗号化が不要となります。また、HTTPネットワークによる以外のオーバーヘッドが少ないJDBCやODBCなどのプロトコルも利用でき，GridDB本来の処理能力を生かすことができるとのことです。

今回、GridDB Cloud の無料トライアルを入手し、払い出された後に管理コンソール (以下、運用管理GUI) へログインするところから、VNetピアリングを使ってサンプルアプリケーション (Java)とGridDBを接続し、GridDBへのデータ登録・確認をおこなうまでの一連の手順を紹介していきたいと思います。

- 事前準備
- ログイン (運用管理GUIへのログイン)
- 接続設定
- サンプルプログラムの実行と確認

# 事前準備

## 無料トライアルの申し込み

[GridDB Cloud サイト](http://cloud.griddb.com/)で、[トライアル](https://form.ict-toshiba.jp/download_form_griddb_cloud/?_ga=2.52834538.339858682.1617928849-483230675.1616380623)を申し込むと数日後にGridDB Cloudの トライアルアカウント情報（URL、契約ID、ログインID、パスワード、Web API情報、利用期間情報など）が電子メールで送付されてきます。

例：
`
GridDB Cloud トライアルへようこそ！

〇〇〇〇〇〇〇
〇〇　〇〇　様

この度は、GridDB Cloudトライアルにお申し込みいただきまして、誠にありがとうございます。
お客様のトライアルアカウント情報をお知らせします。

URL       ： https://cloud1.griddb.com/portal/
契約ID     ： trial1301
ログインID ： Admin
パスワード  ： Password
Web API    ： https://cloud1.griddb.com/trial001/griddb/v2/
利用期間    ： YYYY/MM/DD ~ YYYY/MM/DD

GridDB Cloudトライアルのご利用方法はクイックスタートガイドをご確認ください。

また、WebAPIのIPアドレス制限を実施したい場合や
ご質問などがございましたら、griddb@ml.toshiba.co.jp までご連絡ください。
`

なお、無料トライアルの利用期間は、原則1回限り１カ月間。また、GridDB Cloud サービスの評価に限り、日本国内に限って無償で使用できるとのことです。詳細は、[GridDB Cloud トライアル利用規約](https://www.global.toshiba/content/dam/toshiba/jp/products-solutions/ai-iot/griddb/pdf/GridDBCloud_%E3%83%88%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%AB%E5%88%A9%E7%94%A8%E8%A6%8F%E7%B4%84_20210405a.pdf?elqTrackId=91b560430e444f1b981f35f6d3e88bcb&elqaid=827&elqat=2)をお読みください。

## Microsoft Azure環境の用意

クライアントAPI(Java/C API)を用いたアプリケーションを開発することでGridDB CloudのGridDBに接続できます。その場合、アプリケーションをAzureのプライベートネットワークVNetに配置する必要があります。そして、アプリケーションを配置するVNetとGridDBが動作するVNetを、VNetピアリングを使って接続します。

すなわち、ローカルにアプリケーションを配置し、GridDB CloudのGridDB へ接続するのではなく、事前にAzureサブスクリプションで、Azure上にVNet を設定し、アプリケーションを配置する仮想マシン (VM、Virtual Machine) 等のリソースを用意し、VNetピアリングを用いて接続します。VNetピアリング確立後は、プライベートIPアドレスを指定して通信できるようになります。VNetピアリングは従量課金のサービスですので、利用料金など詳細な仕様について知りたい場合は、Microsoftの公式ドキュメントを参照してください。

なお、Azureのアカウントを保持していないなどVNet環境を利用できない場合には、アプリケーションをGridDB Web APIを用いて開発し、そのアプリケーションをローカルなどに配置しGridDBインスタンスとWebAPI経由で接続することもできます。Web APIを利用した接続については「[入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」および「[【入門】GridDB Cloud にPostmanを使ってWeb APIで触れてみよう！](2021-10-13_GridDB_GridDBCloud_Postman_WebAPI_f45141ef56e90030d453.md)」を参照してください。

# ログイン (運用管理GUIへのログイン)

電子メールでが送付されてきたGridDB CloudサービスのURLをブラウザで開きます。今回の例では、https://cloud1.griddb.com/portal/ です。
![10.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/d1519383-53ec-f553-f191-448912464ae9.png)
契約ID、ログインID、パスワード入力し、GridDB Cloudにログインします。

ログインすると、次のような**運用管理GUI**の画面が表示されます。
![20.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/cbb1c195-e9ce-6f28-16b9-bf04a9be5fc8.png)

この画面では GridDB のクラスタの稼働状況の確認などができるようです。

詳細は、[GridDB Cloud 運用管理GUIリファレンス](https://www.toshiba-sol.co.jp/pro/griddbcloud/docs-jp/v1_3/cloud_reference_management_gui_html/GridDB_Cloud_ManagementGui_Reference.html)　に記述されているみたいなので、ここでは詳細は省略します。

# 接続設定

次にアプリケーション環境から GridDB Cloud の GridDB に接続するための設定をしてみます。

運用管理GUI でクラウドプロバイダの選択をおこない、VNetの情報を入力し、VNetピアリングを確立するための手順を入手します。そして、Virtual Machine上でその手順を実行し、VNetピアリングを確立し接続します。

### クラウドプロバイダの選択

![31.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/c770ab8f-62dc-f75c-6468-d5133a564f74.png)

まずは、運用管理GUI の左のナビゲーションメニューより「ネットワークアクセス」のアイコン![30.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/0998e17f-cc6e-a1ee-22f6-f80e41b95bd1.png)
をクリックします。

![32.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/03c696d3-f922-b14d-ece9-4e5d59666952.png)

次に、「ピアリング接続を作成」ボタンを選択しクリックします。
そうすると次のようなクラウドプロバイダを選ぶダイアログが表示されます。何も変更せず、「次へ」ボタンをクリックしてください。
![40.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/f25468d8-eabe-58c0-76ff-ed7a2757ac2c.png)

### VNetの情報入力

VNet Settingsの画面が表示されます。
![50.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/0d37ff29-b3d1-d485-6120-16ec87a41970.png)

ここではアプリケーションが配置される VNet の情報を入力します。具体的には次の情報を入力します。

- サブスクリプションID
- テナントID
- リソースグループ名
- VNet名

取得方法がわからない場合は、![51.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/5cc1fc23-7e97-0c14-f0f5-a4a280bf5483.png) をクリックして、取得方法のヒントを表示してください。

なお、ここでは、

- サブスクリプションID: cf0be29e-7532-459b-976e-beb79a6172cb
- テナントID: 1ba2ed3a-1a4b-43a6-bfd1-f6ce9507a6e7
- リソースグループ名: cloud-peering-to-heatrun
- VNet名: cloud-peering-to-heatrun-vnet

としています。
![60.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/8f8d67c0-b0a2-c9cf-b704-344b6577868b.png)

サブスクリプションID、テナントIDリソース、リソースグループ名、VNet名のすべて入力が終わったら、「次へ」ボタンを選択しクリックしてください。
![70.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/579f11da-7240-d93f-6429-583d72668601.png)

このようなVNetピアリングを確立するための手順のダイアログが表示されます。

### VNetピアリングの確立

次に、VNetピアリングを確立するために作成された手順を実行します。
次のいずれかの環境で実行可能です。

- Azureポータルから実行可能なAzure Cloud Shell
ポータル右上にある下記の形をしたアイコンをクリックして開けば、すぐにVNetピアリングを確立するための手順のダイアログのコマンドが実行可能です。
![icon1.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/8abf069e-145b-e372-8efc-5c9ede638242.png)
- Azure仮想マシン内で実行可能なAzure CLI
Azure CLIが未インストールの場合はインストールが必要です。VNetピアリングを確立するための手順のダイアログのコマンド実行前に、az loginコマンドなどによるログイン処理が必要です

Azure Cloud ShellやAzure CLIを用いて、表示された1.から4.のコマンドを順に実行します。なお、一部のコマンドにはAzureでの実行権限が必要です。詳しくは、手順の右側のツールチップにマウスカーソルを合わせ、内容を参照してください。

ここでは、Azure CLI を用いて、次のように実行しました。

##### 1.Azure CLIでこのコマンドを実行して、GridDB Cloudのサービスプリンシパルを作成してください

```
$ az ad sp create --id dceac839-02cf-4361-8fa0-bd8063c4396d
```
<!--
##### 実行結果例
```
$ az ad sp create --id dceac839-02cf-4361-8fa0-bd8063c4396d
{
  "accountEnabled": "True",
  "addIns": [],
  "alternativeNames": [],
  "appDisplayName": "dbaas-tenant-gs-spal-trial1301",
  "appId": "dceac839-02cf-4361-8fa0-bd8063c4396d",
  "appOwnerTenantId": "0ebed734-5b0f-4c26-832c-465b3a4aeffa",
  "appRoleAssignmentRequired": false,
  "appRoles": [],
  "applicationTemplateId": null,
  "deletionTimestamp": null,
  "displayName": "dbaas-tenant-gs-spal-trial1301",
  "errorUrl": null,
  "homepage": null,
  "informationalUrls": {
    "marketing": null,
    "privacy": null,
    "support": null,
    "termsOfService": null
  },
  "keyCredentials": [],
  "logoutUrl": null,
  "notificationEmailAddresses": [],
  "oauth2Permissions": [],
  "objectId": "658cf8fb-6606-4aeb-9527-8ba9d9faab0d",
  "objectType": "ServicePrincipal",
  "odata.metadata": "https://graph.windows.net/1ba2ed3a-1a4b-43a6-bfd1-f6ce9507a6e7/$metadata#directoryObjects/@
Element",
  "odata.type": "Microsoft.DirectoryServices.ServicePrincipal",
  "passwordCredentials": [],
  "preferredSingleSignOnMode": null,
  "preferredTokenSigningKeyEndDateTime": null,
  "preferredTokenSigningKeyThumbprint": null,
  "publisherName": "griddbcloud",
  "replyUrls": [],
  "samlMetadataUrl": null,
  "samlSingleSignOnSettings": null,
  "servicePrincipalNames": [
    "dceac839-02cf-4361-8fa0-bd8063c4396d"
  ],
  "servicePrincipalType": "Application",
  "signInAudience": "AzureADMultipleOrgs",
  "tags": [],
  "tokenEncryptionKeyId": null
}
$
```
-->

##### 2. このロール定義をpeering-role.jsonとしてカレントディレクトリに保存してください
viコマンドなどで peering-role.json ファイルに書き込みます。

##### 3. このコマンドを実行してpeering-role.jsonを使ってロールを作成してください
手順2で定義したロール
```
$ az role definition create --role-definition peering-role.json
```

<!--
今回、次のようなメッセージが出力しました。
`
$ az role definition create --role-definition peering-role.json
A custom role with the same name already exists in this directory. Use a different name.
`
これは、「作成対象のカスタムロールが既に存在する場合」に表示されます。
状況から、「V1.2のトライアル環境に接続する際に作成されたカスタムロール」が残っていた為、発生したものと考えられます。
ピアリング接続に用いる環境が同一となりますので、「今回作成するカスタムロール」と「V1.2のトライアル環境に接続する際に作成されたカスタムロール」に差異はないとのことです。
ゆえに、手順3は実施済の状態となるので、このまま手順4をおこなえば、ピアリング接続が可能となります。
-->

<!--
##### 実行結果例
`
$ az role definition create --role-definition peering-role.json
A custom role with the same name already exists in this directory. Use a different name.
$
`
エラーが発生したので、「![51.PNG](./images/51.PNG)」 をクリックして、次のカスタムロールの作成時にエラーが発生した場合を実行してみました。

![80.PNG](./images/80.PNG)
`
$ az role assignment create --role "Network Contributor" --assignee "dceac839-02cf-4361-8fa0-bd8063c4396d" --scope "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/resourceGroups/cloud-peering-to-heatrun/providers/Microsoft.Network/virtualNetworks/cloud-peering-to-heatrun-vnet"
{
  "canDelegate": null,
  "condition": null,
  "conditionVersion": null,
  "description": null,
  "id": "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/resourceGroups/cloud-peering-to-heatrun/providers/Microsoft.Netwo
rk/virtualNetworks/cloud-peering-to-heatrun-vnet/providers/Microsoft.Authorization/roleAssignments/9436cc6b-0b24-4ecc-8ee2-8d5
bbcbcc92e",
  "name": "9436cc6b-0b24-4ecc-8ee2-8d5bbcbcc92e",
  "principalId": "658cf8fb-6606-4aeb-9527-8ba9d9faab0d",
  "principalType": "ServicePrincipal",
  "resourceGroup": "cloud-peering-to-heatrun",
  "roleDefinitionId": "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/providers/Microsoft.Authorization/roleDefinitions/4
d97b98b-1d4f-4787-a291-c67834d212e7",
  "scope": "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/resourceGroups/cloud-peering-to-heatrun/providers/Microsoft.Ne
twork/virtualNetworks/cloud-peering-to-heatrun-vnet",
  "type": "Microsoft.Authorization/roleAssignments"
}
`
-->

##### 4. このコマンドを実行して、ステップ 1で作成したサービスプリンシバルにステップ 3のロールを割り当ててください
`
$ az role assignment create --role "GridDBCloudPeering/cf0be29e-7532-459b-976e-beb79a6172cb/cloud-peering-to-heatrun/cloud-peering-to-heatrun-vnet" --assignee "1a7718df-aaa5-4e8a-892c-ce4db570706f" --scope "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/resourceGroups/cloud-peering-to-heatrun/providers/Microsoft.Network/virtualNetworks/cloud-peering-to-heatrun-vnet"
`
<!--
##### 実行結果例
`
az role assignment create --role "GridDBCloudPeering/cf0be29e
-7532-459b-976e-beb79a6172cb/cloud-peering-to-heatrun/cloud-peering-to-heatrun-vnet" --assignee "dceac839-02cf-4361-8fa0-bd806
3c4396d" --scope "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/resourceGroups/cloud-peering-to-heatrun/providers/Micros
oft.Network/virtualNetworks/cloud-peering-to-heatrun-vnet"
{
  "canDelegate": null,
  "condition": null,
  "conditionVersion": null,
  "description": null,
  "id": "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/resourceGroups/cloud-peering-to-heatrun/providers/Microsoft.Netwo
rk/virtualNetworks/cloud-peering-to-heatrun-vnet/providers/Microsoft.Authorization/roleAssignments/4db39ddb-2e65-4e48-86db-cc2
960b6234b",
  "name": "4db39ddb-2e65-4e48-86db-cc2960b6234b",
  "principalId": "658cf8fb-6606-4aeb-9527-8ba9d9faab0d",
  "principalType": "ServicePrincipal",
  "resourceGroup": "cloud-peering-to-heatrun",
  "roleDefinitionId": "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/providers/Microsoft.Authorization/roleDefinitions/e
ecd66fd-0b4c-49e4-8038-449ffdeb5aca",
  "scope": "/subscriptions/cf0be29e-7532-459b-976e-beb79a6172cb/resourceGroups/cloud-peering-to-heatrun/providers/Microsoft.Ne
twork/virtualNetworks/cloud-peering-to-heatrun-vnet",
  "type": "Microsoft.Authorization/roleAssignments"
}
`
-->

##### 5. 接続がただしく動作するかどうかを検証してください

コマンドの実行が完了したら、運用管理GUIのダイアログに戻って左下の「検証」ボタンをクリックして接続状態を確認してください。

![81.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/fb31d428-51f4-3f0e-c08e-7bd9c5f107a6.png)

![110.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/d1ec9732-e767-d76d-0bc1-18f45664e0e4.png)

接続が確認できたら、「ピアリングを作成」ボタンを押して、VNet ピアリングを確立します。

### VNetピアリングの状態確認

「ピアリング接続を作成」をクリックし、ピアリング接続リストの一覧に戻り、「ステータス」が接続となっていれば VNet ピアリング確立が成功です。これで GridDB のプライベートIPアドレスを使った通信が可能となりました。

![111.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/d1a38a36-9d6c-bd51-eae7-e1a0a3feef16.png)

なお、AzureポータルからVNetピアリングを削除した場合などは、「ステータス」が未接続となります。再度接続したい場合は、新たにVNetピアリングを確立してください。

また、GridDBとアプリケーションの配置されるネットワークアドレスが競合する場合は VNetピアリングを確立することができません。その場合は、アプリケーションが配置されるネットワークアドレスを変更してください。

# サンプルプログラムの実行と確認

ここでは、アプリケーションプログラムからGridDBにデータを登録する方法について紹介します。

## アプリケーション開発 (クライアントAPI) ライブラリ

### アプリケーション開発(クライアントAPI) ライブラリの取得

運用管理GUIで、左のナビゲーションメニューより「サポート」を選択しますクリックします。
![123.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/8a8f7e03-20bf-f075-764a-fcd621061666.png)

「GridDB Cloud ライブラリ／プラグインダウンロード」の「ファイルダウンロード」のURLリンクからダウンロードをおこないます。
![130.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/af1c0e14-014c-fa17-18a4-427d26cd222a.png)

Azure CLIで、curlコマンドなどを用いて、Virtual Machineにダウンロードします。

```
$ curl 'https://dbaassharefilest.blob.core.windows.net/file/GridDB_Cloud_doc_lib.zip?sv=2020-02-10&ss=b&srt=o&sp=r&se=2026-04-06T14:59:59Z&st=2021-04-06T02:37:13Z&spr=https&sig=nsp9dFZeR3anSkZ4daqx1IWzx3fHz9iPH9jHvsrScb4%3D' -o GridDB_Cloud_doc_lib.zip
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current2021-04-06T02:37:13Z&spr=https&sig=nsp9dFZeR3an
                                 Dload  Upload   Total   Spent    Left  Speed
100 28.8M  100 28.8M    0     0  11.8M      0  0:00:02  0:00:02 --:--:-- 11.8M
$ ls
GridDB_Cloud_doc_lib.zip
$ unzip GridDB_Cloud_doc_lib.zip
Archive:  GridDB_Cloud_doc_lib.zip
   creating: GridDB_Cloud_doc_lib/
   creating: GridDB_Cloud_doc_lib/embulk-output-griddb/
  inflating: GridDB_Cloud_doc_lib/embulk-output-griddb/build.gradle
   creating: GridDB_Cloud_doc_lib/embulk-output-griddb/config/
  inflating: GridDB_Cloud_doc_lib/embulk-output-griddb/config.yml
(省略)
$
```

なお、GridDB_Cloud_doc_lib.zipを保存しているストレージは、匿名アクセスを無効にしているようです。だから、wgetコマンドを使用した場合、ストレージ上では匿名アクセスとして扱われ、結果として、アクセスを拒否されてしますのでご注意を願います。

GridDB_Cloud_doc_lib.zipを解凍すると次のようになります。

```
$ cd GridDB_Cloud_doc_lib/
$ ls -l
total 21748
drwxrwxr-x. 6 azgsadm azgsadm      230 Oct 15 10:34 embulk-output-griddb
drwxrwxr-x. 6 azgsadm azgsadm     4096 Mar  7 13:22 fluent-plugin-griddb
-rw-rw-r--. 1 azgsadm azgsadm    10793 Jul  1  2021 gdbConnectSample.zip
drwxrwxr-x. 8 azgsadm azgsadm     4096 Oct 15 10:37 grafana-input-plugin
-rw-rw-r--. 1 azgsadm azgsadm  1200152 Sep 13  2021 griddb-ee-c_lib-4.6.0-linux.x86_64.rpm
-rw-rw-r--. 1 azgsadm azgsadm  1452988 Sep 13  2021 griddb-ee-java_lib-4.6.0-linux.x86_64.rpm
-rw-rw-r--. 1 azgsadm azgsadm 19587096 Sep 14  2021 griddb-ee-webapi-4.6.0-linux.x86_64.rpm
drwxrwxr-x. 2 azgsadm azgsadm      218 Oct 15 11:24 JDBC
drwxrwxr-x. 5 azgsadm azgsadm      202 Mar  7 11:35 logstash-output-plugin
drwxrwxr-x. 4 azgsadm azgsadm       61 Oct 15 11:24 ODBC
drwxrwxr-x. 3 azgsadm azgsadm       88 Mar  7 11:31 telegraf-output-plugin
```

griddb-ee-java_lib-4.6.0-linux.x86_64.rpm がJava用クライアントAPIライブラリのrpmパッケージです。

### アプリケーション開発ライブラリ (クライアントAPI) のインストール

#### Java用クライアントAPIライブラリのインストール

今回、Java言語でコーディングされたサンプルプログラムを実行させますので、rpmコマンドを用いて Java サンプルプログラムのビルドで利用するJava用クライアントAPIのライブラリをインストールします。

```
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:griddb-ee-java_lib-4.6.0-linux   ################################# [100%]
$
```

## サンプルプログラムの作成

### サンプルプログラムの処理概要

サンプルプログラムは、接続情報を引数で受け取り、次のような流れでデータの登録・表示をおこなうものです。

1. 接続パラメータの設定
1. 接続用インスタンスの作成
1. 接続し、コンテナ(コンテナ名:point01)を作成
   TIMESTAMP, BOOL, DOUBLEの3カラムで構成
1. ロウを登録
   (現在時刻, false, 100.0)を登録
1. 過去6時間以内に登録されたロウを表示

### サンプルプログラムソースコード

サンプルプログラムのソースコードは次の通りです。このソースコードをコピーし、cloud_sample.javaというファイル名で任意のディレクトリに保存してください。

GridDBの接続方式には、マルチキャスト方式、固定リスト方式、プロバイダ方式の3種類がありますが、GridDB Cloudでは、サンプルプログラムで指定したプロバイダ方式 (Notification Provider URLを接続先として指定する方式) のみが有効で、他の方式には対応していません。

```
import java.util.Date;
import java.util.Properties;

import com.toshiba.mwcloud.gs.GSException;
import com.toshiba.mwcloud.gs.GridStore;
import com.toshiba.mwcloud.gs.GridStoreFactory;
import com.toshiba.mwcloud.gs.RowKey;
import com.toshiba.mwcloud.gs.RowSet;
import com.toshiba.mwcloud.gs.TimeSeries;
import com.toshiba.mwcloud.gs.TimestampUtils;
import com.toshiba.mwcloud.gs.TimeUnit;


// Storage and extraction of a specific range of time-series data
public class cloud_sample {

    static class Point {
        @RowKey Date timestamp;
        boolean active;
        double voltage;
    }

    public static void main(String[] args) throws GSException {

        // Acquiring a GridStore instance
        Properties props = new Properties();
        props.setProperty("notificationProvider", args[0]);
        props.setProperty("clusterName", args[1]);
        props.setProperty("user", args[2]);
        props.setProperty("password", args[3]);
        GridStore store = GridStoreFactory.getInstance().getGridStore(props);

        // Creating a TimeSeries (Only obtain the specified TimeSeries if it already exists)
        TimeSeries<Point> ts = store.putTimeSeries("point01", Point.class);

        // Preparing time-series element data
        Point point = new Point();
        point.active = false;
        point.voltage = 100;

        // Store the time-series element (GridStore sets its timestamp)
        ts.append(point);

        // Extract the specified range of time-series elements: last six hours
        Date now = TimestampUtils.current();
        Date before = TimestampUtils.add(now, -6, TimeUnit.HOUR);

        RowSet<Point> rs = ts.query(before, now).fetch();

        while (rs.hasNext()) {
            point = rs.next();

            System.out.println(
                    "Time=" + TimestampUtils.format(point.timestamp) +
                    " Active=" + point.active +
                    " Voltage=" + point.voltage);
        }

        // Releasing resource
        store.close();
    }
}
```

### サンプルプログラムのコンパイル

Java用クライアントAPIライブラリは、/usr/share/javaにインストールされているので、そのパスを通し、サンプルプログラムをコンパイルします。

`
$ export CLASSPATH=${CLASSPATH}:/usr/share/java/gridstore.jar
$ javac cloud_sample.java
$ ls
'cloud_sample$Point.class'   cloud_sample.class   cloud_sample.java
`
これで、サンプルプログラムが作成されました。次は、接続情報をメモし引数に入れてプログラムを実行します。


## 接続情報の確認

作成したプログラムを実行する前に、接続情報の確認と、データベースユーザの作成を行います。
まずは接続情報の確認をします。「クラスタ」画面に遷移し、次の接続情報をメモしてください。

- クラスタ名 （クラスタ詳細の左上）
- アドレスプロバイダのURL（クラスタ詳細の右下）
![410.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/ecfa4832-c8c4-d96e-b9ca-43c43dae4769.png)

今回の例では、クラスタ名：「gs_clustertrial1301」、アドレスプロバイダのURL：「http://dbaassharegssta.blob.core.windows.net/dbaas-share-griddb-blob/trial1301.json 」となります。

## データベースユーザの作成

データベースサンプルプログラムを実行するにあたり、データベースユーザを作成します。「GridDBユーザ」画面に遷移し、「データベースユーザの作成」ボタンをクリックしてください。
![431.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/20b2b55c-815e-53c2-2fa4-f3c4ab988bf6.png)

任意のユーザ名とパスワードを設定し、「作成」ボタンをクリックします。
![432.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/273bffd4-0200-d32a-31dd-9daf34b5c33b.png)

ここでは、：ユーザ名：「test_user1」、パスワード：「test_user1」としました。
![433.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/ac2e63b8-8a0f-8ad5-86fd-704d7954dda6.png)

「作成」をクリックします。
![444.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/46dc2c7d-4ea2-3a28-346b-68e20e31f80c.png)

データベースユーザ名とそのパスワードが設定されました。

## サンプルプログラムの実行

`
$ java cloud_sample <Notification Provider URL> <クラスタ名> <データベースユーザ名> <データベースユーザパスワード>
`
実際に、サンプルプログラムを実行してみます。

`
$ java cloud_sample http://dbaassharegssta.blob.core.windows.net/dbaas-share-griddb-blob/trial1301.json gs_clustertrial1301 test_user1 test_user1
Time=2022-03-23T09:25:03.933Z Active=false Voltage=100.0
$
`
このような実行結果が表示されれば成功です。

## サンプルプログラムによる登録データの確認

運用管理GUIで、サンプルプログラムによる登録データを確認してみましょう。

「クエリ」画面を開きます。右側にサンプルプログラムで作成したコンテナ point01があることを確認します。

### SQL実行

![500.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/2f13ac61-c871-6444-df8f-029cc8ceb68e.png)

次に、point01のデータをすべて取得するSQL文 ```SELECT * FROM point01``` を入力し、実行ボタン![520.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/13adaa22-1988-59f0-0270-e110ecce9988.png)
を押します。
![510.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/1e74e62e-a0d9-ef36-f012-a4230e28b8f6.png)

実行結果が次のように表示されていれば成功です。
![540.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/838618b6-ecfb-004f-ca3f-ff5064f9dff4.png)


### TQL実行

GridDB Cloud v1.1より、「クエリ」画面で、TQLが実行できるようになりました。
「クエリ」画面 の画面右側のSQL/TQLスイッチボタンでTQLを選択します。
![610.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/7bbf38fc-1957-6584-46b3-b96b3ea9edf7.png)

「コンテナを選択」で、コンテナ名としてpoint01を選択します。
![620.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/d873dabd-6506-fd3f-a259-9f55129cc3d7.png)

データをすべて取得するSQL文 ```SELECT * ``` が入力されているので、実行ボタン![520.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/d9012b46-8d52-c610-5a0d-2c65a42b6a1b.png)
を押します。
![640.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/04c30646-84d7-b9f4-8c77-673aeb74f44f.png)

SQL実行結果の上に同じ結果が上書きされます。

# さいごに

今回、ビッグデータ・IoT向けデータベース GridDB をDB as a Serviceとして提供されるマネージドサービスである [GridDB Cloud](http://cloud.griddb.com/) の無料トライアルを入手し、払い出された後に運用管理GUIへログインするところから、GridDBの特長である高い処理能力を十分に生かすために、Web APIによる接続方法だけではなく、GridDBとアプリケーションをVNet（Azure Virtual Network）ピアリングを使ってサンプルアプリケーション (Java)とGridDBを接続し、GridDBへのデータ登録・確認をおこなうまでの一連の手順を紹介しました。

ローカル環境などからのWeb APIを利用した接続については「[入門】GridDB Cloudにcurlを使ってWeb APIで触れてみよう！](2021-09-15_GridDB_GridDBCloud_WebAPI_curl_6c766e64c2c2c7aab81d.md)」および「[【入門】GridDB Cloud にPostmanを使ってWeb APIで触れてみよう！](2021-10-13_GridDB_GridDBCloud_Postman_WebAPI_f45141ef56e90030d453.md)」を参照していただければと思います。


なお、記述について誤りがあったり、気になることがあれば、編集リクエストやコメントでフィードバックしていただけると助かります。

------