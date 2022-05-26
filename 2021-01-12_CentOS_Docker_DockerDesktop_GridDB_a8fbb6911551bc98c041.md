<!--
title:   【入門】はじめての Docker Desktop の CentOS コンテナでの GridDB Community Edition のセットアップ
tags:    CentOS,Docker,DockerDesktop,GridDB,入門
id:      a8fbb6911551bc98c041
private: false
-->
---
【入門】はじめての Docker Desktop の CentOS コンテナでの GridDB Community Edition のセットアップ
---

Docker Desktop 上の CentOS コンテナ (仮想環境)で、オープンソースのGridDB Community Editionをセットアップする方法について、はじめからていねいにまとめて紹介します。また、途中で Dockerコマンドなどについても詳しく説明していくので、  Docker初心者で、DockerのCentOS コンテナに GridDB Community Edition をセットアップしたいという方には、参考になると思います。

具体的には、ホストOS macOS に Docker for Mac がインストールされ CentOS のDockerコンテナがビルドされた１台の MacBook に GridDB Community Edition をインストール・環境設定し、GridDBサーバの起動/停止やサンプルプログラムを実行し動作確認をします。

なお、Docker Desktop for Windows の環境の場合でも手順などはほとんど同じです。

# 流れ

- 事前準備
- GridDB インストール
- 環境設定
- GridDB サーバの起動
- サンプルプログラムの実行
- GridDB サーバの停止
- セットアップ済みのコンテナの保存

# 事前準備

## CentOS のDockerコンテナを用意

今回は、すでにDocker Desktop の環境が構築され、その上に動作確認済の CentOS のDockerイメージが用意されていることを前提としています。これから用意する場合は、次の２つの記事を参照して頂ければと思います。

「[【入門】はじめての Docker Desktop  for Mac のインストールと CentOS の仮想環境構築のセットアップ](2020-12-15_CentOS_Docker_DockerDesktop_92217e0a887bb81e3155.md)」
「[【入門】はじめての Docker Desktop for Windows のインストールと CentOS の仮想環境構築のセットアップ](2019-09-11_CentOS_Docker_DockerDesktop_7b21377b5c9e3ffddf4a.md)」

なお、今回の動作確認済みのCentOSは、CentOS　7.9.2009 imagesを用いています。

以降、ここでは、動作確認済の CentOS のイメージの名前は centos:gahoh とします。
ターミナル を起動して、docker images コマンドを用いて、ローカルに保存した Docker イメージの一覧を表示します。
次のようなCentOS のDocker イメージが用意されているものとします。

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
centos       gahoh     8beaf0d882f9   53 minutes ago   663MB
centos       centos7   8652b9f0cb4c   6 weeks ago      204MB
```
CentOS のDockerイメージ centos:gahoh からコンテナをビルドして起動します。

### CentOS コンテナ の作成・起動

CentOS のイメージからCentOS コンテナをビルドして起動するには　docker run コマンドを用います。
次のように docker run コマンドを実行して、コンテナをビルド・起動します。

```
$ docker run -it --name="centos" centos:gahoh /bin/bash
#
```

ここでは、CentOS のDockerイメージ centos:gahoh から コンテナをビルドし、同時にコンテナも起動し、そのタイミングで自動的にログインし、そのまま bash シェルに接続します。
ここでは新しく作成するコンテナに "centos" という名前を付けています。

# GridDB のインストール

GridDB Community Editionのパッケージのインストールは次の通りです。

```
(CentOS)
$ sudo rpm -ivh griddb-X.X.X-(X.)linux.x86_64.rpm

(Ubuntu)
$ sudo dpkg -i griddb_X.X.X_amd64.deb

(openSUSE)
$ sudo rpm -ivh griddb-X.X.X-opensuse.x86_64.rpm

※ X.X.X-(X.) または X.X.X はバージョンを意味します。
```

今回、Docker Desktop for Mac の CentOS上で、GitHubのサイトの Release の Assets のRPMパッケージを用いて直接インストールします。RPMパッケージのURLは、次の通りです。

>https://github.com/griddb/griddb/releases/download/vX.X.X/griddb-X.X.X(-X)-linux.x86_64.rpm

X.X.X または X.X.X-(X) はバージョンを意味します。

今回、ここでインストールするバージョンは、v4.5.2とします。よって、インストールに用いるRPMパッケージのURLは、次のようになります。

>https://github.com/griddb/griddb/releases/download/v4.5.2/griddb-4.5.2-linux.x86_64.rpm

では、インストールします。
RPMパッケージをインストールする／アンインストールする rpm コマンドを用います。

<!--
proxy経由の通信をする必要がある場合、CentOS全体にproxyを適用の設定をしておいてください。
proxy サーバが yourProxy、proxy ポートが proxyPort の場合：

```
# echo PROXY="'yourProxy:proxyPort'" >> /etc/profile
# echo "export http_proxy=$PROXY" >> /etc/profile
# echo "export https_proxy=$PROXY" >> /etc/profile
# source /etc/profile
```
-->

rpm コマンド でインストールします。

```
# rpm -ivh https://github.com/griddb/griddb/releases/download/v4.5.2/griddb-4.5.2-linux.x86_64.rpm
```

インストールが開始されます。

```
# rpm -ivh https://github.com/griddb/griddb/releases/download/v4.5.2/griddb-4.5.2-linux.x86_64.rpm
Retrieving https://github.com/griddb/griddb/releases/download/v4.5.2/griddb-4.5.2-linux.x86_64.rpm
Preparing...                          ################################# [100%]

------------------------------------------------------------
Information:
  User gsadm and group gridstore have been registered.
  GridDB uses new user and group.
------------------------------------------------------------

Updating / installing...
   1:griddb-4.5.2-linux               ################################# [100%]
#
```
これで、インストールは完了です。

## インストール後のユーザとグループ

このRPMパッケージをインストールすると、CentOSのユーザ gsadm とグループ gridstore が自動的に作成されます。ユーザ gsadmは、GridDBを運用するための 管理ユーザとして使用します。
ユーザ gsadm でログインすると環境変数 GS_HOMEとGS_LOGが自動的に設定されます。また、運用コマンドの場所が環境変数 PATHに設定されます。

### インストール後のディレクトリ構成
インストールされたGridDBノードのディレクトリ構成を確認します。
ノードの定義ファイルやデータベースファイルなどを配置するGridDBホームディレクトリと、インストールしたファイルを配置するインストールディレクトリが作成されます。

# 環境設定

## 管理ユーザ gsadm の設定

管理ユーザは、運用コマンドの実行や、管理者権限のみ許可されている操作を実行する際に用います
管理ユーザ gsadm にはパスワードが設定されていないので、パスワードを設定してください。

```
# passwd gsadm
Changing password for user gsadm.
New password: （パスワードを入力します）
Retype new password: （パスワードを入力します）
passwd: all authentication tokens updated successfully.
# gpasswd -a gsadm wheel
Adding user gsadm to group wheel
```

ここでは、CentOSのパスワードポリシーに準じたパスワードを設定してください。
とりあえず GridDB4admin としました。

### 管理ユーザ gsadm のパスワード設定

管理ユーザ gsadm のパスワードが設定されていない場合、サーバは起動できないため必ずパスワードを設定してください。
ユーザ gsadm でログインし、gs_passwd コマンドを使用して、管理ユーザのパスワードの設定します。

```
# su - gsadm
$ gs_passwd admin
Password: （パスワードを入力します）
Retype password: （パスワードを入力します）
$
```

ここでは、パスワードを とりあえず admin と入力して設定しました。
なお、環境変数 `$GS_HOME`, `$GS_LOG` が設定されます。

## クラスタ名の設定

インストール後に GridDB を動作させるためには、アドレスやクラスタ名などのノードのパラメータの初期設定が必要です。ここでは、必須項目の クラスタ名 のみ設定を行い、それ以外はデフォルト値を用います。
クラスタの クラスタ名 をクラスタ定義ファイルに記述します。クラスタ定義ファイルは、/var/lib/gridstore/conf/gs_cluster.json です。
"clusterName":"" の部分にクラスタ名を記載します。ここでは、myCluster という名前を用います。

```
$ vi conf/gs_cluster.json

{
        "dataStore":{
                "partitionNum":128,
                "storeBlockSize":"64KB"
        },
        "cluster":{
                "clusterName":"",
                "replicationNum":2,
                "notificationAddress":"239.0.0.1",
                "notificationPort":20000,
                "notificationInterval":"5s",
                "heartbeatInterval":"5s",
                "loadbalanceCheckInterval":"180s"
        },
        "sync":{
                "timeoutInterval":"30s"
        },
        "transaction":{
                "notificationAddress":"239.0.0.1",
                "notificationPort":31999,
                "notificationInterval":"5s",
                "replicationMode":0,
                "replicationTimeoutInterval":"10s"
        },
        "sql":{
                "notificationAddress":"239.0.0.1",
                "notificationPort":41999,
                "notificationInterval":"5s"
        }
}
```

”clusterName”:””の部分にクラスタ名 myCluster を挿入して保存します。

```
$ vi conf/gs_cluster.json

{
        "dataStore":{
                "partitionNum":128,
                "storeBlockSize":"64KB"
        },
        "cluster":{
                "clusterName":"myCluster",
                "replicationNum":2,
                "notificationAddress":"239.0.0.1",
                "notificationPort":20000,
                "notificationInterval":"5s",
                "heartbeatInterval":"5s",
                "loadbalanceCheckInterval":"180s"
        },
        "sync":{
                "timeoutInterval":"30s"
        },
        "transaction":{
                "notificationAddress":"239.0.0.1",
                "notificationPort":31999,
                "notificationInterval":"5s",
                "replicationMode":0,
                "replicationTimeoutInterval":"10s"
        },
        "sql":{
                "notificationAddress":"239.0.0.1",
                "notificationPort":41999,
                "notificationInterval":"5s"
        }
}
```

# サーバの起動

GridDBノードの起動／停止、クラスタの開始／停止の操作をやってみましょう。起動や停止の操作方法はいくつかあるようですが、ここでは、運用コマンドを使ってみます。なお、運用コマンドは gsadm ユーザで実行で実行します。

## サーバを起動する

### ノードを起動する
ノードを起動するは、運用コマンドの gs_startnode コマンドを用います。
ユーザ認証オプション -u には管理ユーザ のユーザ名: admin とパスワード (ここでは、admin) を指定し、ノードの起動を待ち合せる -w オプションを指定します。

```
$ gs_startnode -u admin/admin -w
.
Started node.
```

### クラスタを開始する

クラスタの開始するには、運用コマンドの gs_joincluster コマンドを用います。 ユーザ認証オプション-uには管理ユーザのユーザ名 admin とパスワード(ここでは、admin) を指定し、クラスタの開始を待ち合せる -w オプションを指定します。クラスタ名を -c オプションで指定します.

```
$ gs_joincluster -u admin/admin -c myCluster –w
..
Joined node
```

クラスタが開始されているかなどのクラスタの状態を確認するのは、運用コマンドの gs_stat コマンドを用います。
ユーザ認証オプション -u には管理ユーザ admin のユーザ名とパスワードを指定します。また、クラスタのステータスを確認するために、"Status"の表記の行のみを grep で抽出した方がよいでしょう。

```
$ gs_stat -u admin/admin | egrep Status
        "clusterStatus": "MASTER",
        "nodeStatus": "ACTIVE",
        "partitionStatus": "NORMAL",
```

"clusterStatus"、"nodeStatus"、"partitionStatus"が上のように表示されていれば、正常に起動している状態で、アプリケーションからクラスタにアクセスが可能になります。

# サンプルプログラムの実行

```
$ cd
$ export CLASSPATH=${CLASSPATH}:/usr/share/java/gridstore.jar:.
$ mkdir gsSample
$ cp /usr/griddb-4.5.2/docs/sample/program/Sample1.java gsSample/.
$ javac gsSample/Sample1.java
$ java gsSample/Sample1 239.0.0.1 31999 myCluster admin admin
Person:  name=name02 status=false count=2 lob=[65, 66, 67, 68, 69, 70, 71, 72, 73, 74]
```

JDKがインストールされていない場合、 CentOS の標準の yum リポジトリに java-1.8.0-openjdk という名前があります。のOpenJDKの Java 8 (JDK) 開発環境のパッケージです。

```
# sudo yum -y install java-1.8.0-openjdk-devel
```

# サーバの停止

## サーバを停止する

起動の流れとは逆に、クラスタを安全に停止してから、各ノードを停止します。クラスタを停止した段階で、アプリケーションからはクラスタにアクセスできなくなります。

### クラスタを停止する

運用コマンドの gs_stopcluster コマンドを実行します。クラスタ停止コマンドを実行した時点で、アプリケーションからはクラスタにアクセスできなくなります。
ユーザ認証オプション -u には管理ユーザ admin のユーザ名とパスワードを指定し、クラスタの停止を待ち合せる -w オプションを指定します。

```
$ gs_stopcluster -u admin/admin -w
.
Stopped cluster
```

### ノードを停止する

運用コマンドの gs_stopnodeコマンドを実行し、ノードを停止（シャットダウン）します。必ず、クラスタを停止してからノードを停止します。
ユーザ認証オプション -u には管理ユーザ admin のユーザ名とパスワードを指定し、クラスタの停止を待ち合せる -w オプションを指定します。

```
$ gs_stopnode -u admin/admin -w
Stopping node
.
Stopped node
```

# セットアップ済みの環境の保存

## セットアップした動作環境を新たな Docker イメージとして保存

コンテナで変更した作業の内容を新たなイメージとして保存するコマンドとして docker commit というコマンドが用意されています。これを使って GridDB をインストール・環境設定するために変更を加えたコンテナをイメージとして保存しておきます。
いったん、コンテナから抜けて、docker commit コマンドで"centos"というコンテナを"centos:gahoh"という名前のイメージに上書き保存します。

今後、これで、GridDBの動作環境を楽に使えるようにします。

### 保存するコンテナを停止する

コンテナから抜けます。

```
$ exit
logout
# exit
exit
$
```
これで、コンテナ centos は停止しました。
docker ps コマンドで、 -a オプションをつけて、停止中のコンテナを含めたすべてのコンテナを表示します。

```
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                     PORTS     NAMES
cd115c8ce57b   centos:gahoh   "/bin/bash"   37 minutes ago   Exited (0) 2 minutes ago             centos
$
```

### 停止したコンテナをイメージに保存する

その前に、docker images コマンドで、現在取得したイメージを確認します。

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
centos       gahoh     8beaf0d882f9   2 hours ago   663MB
centos       centos7   8652b9f0cb4c   6 weeks ago   204MB
```

セットアップした動作環境のCentOS コンテナの"centos"を docker commit コマンドで、イメージ "centos:gahoh" という名前のイメージに上書き保存します。

```
$ docker commit centos centos:gahoh
sha256:9d94ca66706c330a7b2c7ee49c8c29ccc528e3ae54cd757085e0e17b21e27ddd
$
```

一応、docker imagesコマンドにて、イメージ "centos:gahoh" が更新されたことを確認しておきましょう。

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
centos       gahoh     9d94ca66706c   9 seconds ago   709MB
centos       centos7   8652b9f0cb4c   6 weeks ago     204MB
$
```

これで、GridDB のインストール・環境設定をおこなったコンテナが イメージ centos:gahoh に上書き保存されたことが確認できます。

# さいごに

Docker Desktop for Mac 上の CentOS コンテナ (仮想環境)で、GridDB Community Editionを構築・動作確認をしながら、一連の手順を詳細に記述したつもりです。 もし、記述について誤りがあったり、気になることがあれば、編集リクエストやコメントでフィードバックしていただけると助かります。