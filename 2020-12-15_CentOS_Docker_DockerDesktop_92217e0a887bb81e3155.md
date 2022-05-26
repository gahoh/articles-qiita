<!--
title:   【入門】はじめての Docker Desktop for Mac のインストールと CentOS の仮想環境構築のセットアップ
tags:    CentOS,Docker,DockerDesktop,入門,初心者
id:      92217e0a887bb81e3155
private: false
-->
---
【入門】はじめての Docker Desktop for Mac のインストールと CentOS の仮想環境構築のセットアップ
---

最近では、Linuxの環境を利用したいときに 無償の Docker Desktop (for Mac and Windows ) で Linuxコンテナを使うというのがお手軽になりつつあります。
今回、MacBookに Doker Desktop for Mac をインストールし、CentOSの仮想環境を構築する手順について、はじめからていねいにまとめてご紹介します。

また、途中で Dockerコマンドなどについても詳しく説明していくので、 Docker初心者で CentOSの仮想環境を構築したいという方は、是非参考にしていただければ幸いです。

- 前提条件
- 事前準備
- Docker Desktop for Mac のインストール
- CentOS コンテナ (仮想環境) の構築
- CentOS コンテナ (仮想環境）の整備
- 作業済み CentOS コンテナ (仮想環境）の保存

なお、Windows 10のノートPCに Docker Desktop for Windows をインストールし、CentOSの仮想環境を構築する手順については「[【入門】はじめての Docker Desktop  for Windows のインストールと CentOS の仮想環境構築のセットアップ](2019-09-11_CentOS_Docker_DockerDesktop_7b21377b5c9e3ffddf4a.md)」の方を参照していただければと思います。

# 前提条件

システム要件の詳細は[こちら](https://docs.docker.jp/docker-for-mac/install.html#mac-what-to-know-before-you-install)を参照してください。

なお、今回インストールしたデバイスの仕様とmacOSの仕様は次の通りです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/9f970b20-f84a-5264-5805-72386915d18f.png)
- MacBook Pro (13-inch, 2016, Four Thunderbolt 3 Ports)
- プロセッサ 2.9 GHz デュアルコアIntel Core i5
- メモリ 16 GB 2133 MHz LPDDR3
- macOS Big Sur バージョン 11.0.1

# Docker Desktop for Mac のインストール

## インストール

### インストーラーをダウンロードする

インストールは次のサイトで Docker for Mac インストーラーをダウンロードしてインストールしていくことができます。
https://www.docker.com/products/docker-desktop

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/edee9387-e951-99e0-7a1d-7980221afda5.png)
[Download for Mac] のダウンロードボタンをクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/54f0bbe3-bc13-ac52-3140-295ed593f318.png)
これで、[Docker.dmg] ファイルがダウンロードできます。ファイルは1つだけです。

ここまでで MacBookマシンへのダウンロードまでが完了です。

### MacBookマシンへのインストールする

次はインストールを行います。ダウンロードした[Docker.dmg] ファイルをダブルクリックしてインストーラを実行します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/7170a11e-1370-8a72-dcc9-2fdb2fee44d7.png)
左側の「Docker」のアイコンを右側の「Appllications」フォルダにドラッグ＆ドロップします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/757d9f6c-f15d-1eb8-59bc-cc15b3e6ddf9.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/cf69e2fc-f49d-953e-26d9-d00c5ff784d9.png)
アプリケーション一覧を確認し、Docker.appが入っていたらインストール完了です。

### Dockerの起動

アプリケーションから「Docker.app」をクリックして起動します。

初回起動時にセキュリティ警告が出たら「開く」をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/4f79c144-06e7-7ae8-20c7-470837f92952.png)
ネットワークコンポーネントへのアクセスを問われるウィンドウが出たら「OK」を押して許可のためMacBook本体のパスワードを入力します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/c629d03b-5118-d644-d544-a8a96fc0ba72.png)
初回起動時にはダッシュボード上にチュートリアルが表示されます。とりあえず、「Skip tutorial」をクリックしてください。
![スクリーンショット 2020-12-09 21.49.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/07f847b7-6434-5159-1fb9-3fdce0082857.png)
次のような画面が通常のDocker Dashboardです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/336f1e45-7963-ac48-9f50-51f61a67ecea.png)
なお、ダッシュボードはDocker起動中であればナビゲーションからも起動できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/a64ef916-2fe3-2c6e-0129-ee7825f718bd.png)

## 動作確認

### Docker のバージョンの確認をしてみる

まずは、インストールされた Docker のバージョンを確認してみましょう。
ターミナル を起動して、docker version コマンドでバージョン情報が表示されます。

```ターミナル:
$ docker version
Client: Docker Engine - Community
 Cloud integration: 1.0.4
 Version:           20.10.0
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        7287ab3
 Built:             Tue Dec  8 18:55:43 2020
 OS/Arch:           darwin/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.0
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       eeddea2
  Built:            Tue Dec  8 18:58:04 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc:
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### Hello Worldを動かしてみる

では、早速、プログラマにはおなじみの hello world をやってみましょう。
docker run コマンドは、イメージからコンテナを起動するコマンドです。

```
$ docker run hello-world
```

上記のコマンドの場合、hello-world というイメージからコンテナを作成して起動するという意味になります。ただし、ローカルに hello-world イメージがないため、Docker デーモンが hello-world イメージを Docker Hub（Docker社が運営する、インターネット上でイメージを公開・共有したりする Docker Registry サービス）からダウンロードし、イメージからコンテナを起動します。一般的には、イメージはファイルシステムとアプリケーションやミドルウェア、実行時に必要とするパラメータから構成されます。

このコンテナは次のような標準出力を出して終了します。


```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:1a523af650137b8accdaed439c17d684df61ee4d74feac151b5b337bd29e7eec
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

$
```

#### :warning: 注意 :warning:

次のようなエラーが発生した場合、Proxyなどのネットワーク関連の設定の問題の場合があります。

```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io on 192.168.65.1:53: no such host.
See 'docker run --help'.
```

Docker Desktopメニューで[setting]を選択し[setting]ダイアログの[Resources]タブの[PROXIES]でProxyの設定をしてみてください。

![スクリーンショット 2020-12-14 23.02.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/112127/0e74bbf9-6afe-4a55-67cb-a61045eb3f8b.png)

## CentOS コンテナ (仮想環境) の構築

先ほどの hello-world の例で、コマンドラインでイメージの取得とコンテナの作成・起動・実行が簡単にできることがおわかりいただけたと思います。
では、本題の CentOS のイメージを取得し、CentOS のコンテナ (仮想環境) を作成・起動させてみましょう。

### CentOS の イメージの取得

#### イメージを取得する

CentOS のコンテナのイメージも Docker Hub の CentOS のリポジトリの中で、CentOS のバージョンごとにタグ付けされて管理されています。
それでは CentOS7 の Docker イメージを取得しましょう。 Docker イメージの取得は docker pull コマンドを利用します。なお、すでに Docker ID を取得しログインしているので、この　DokerHub サービスを利用することができます

```
$ docker pull centos:centos7
centos7: Pulling from library/centos
2d473b07cdd5: Pull complete
Digest: sha256:0f4ec88e21daf75124b8a9e5ca03c37a5e937e0e108a255d890492430789b60e
Status: Downloaded newer image for centos:centos7
docker.io/library/centos:centos7
$
```

これで CentOS イメージのダウンロードが終わりました。環境に依存しますが数十秒で終了します。なお、pull コマンドの「centos:centos7」の意味は、centosリポジトリの、centos7タグが付いたイメージという意味です。これを理解すればいろいろなリポジトリからいろいろなイメージをダウンロードできるようになります。

#### 取得した イメージを確認する

取得した Docker イメージは docker images コマンドで一覧表示できます。

```
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
centos        centos7   8652b9f0cb4c   6 weeks ago     204MB
hello-world   latest    bf756fb1ae65   11 months ago   13.3kB
$
```

centos リポジトリの centos7 タグが付いたイメージが IMAGE ID 8652b9f0cb4c で、そして Hello Worldを動かしてみる時に使用した hello-worldリポジトリのlatestタグが付いたイメージが IMAGE ID bf756fb1ae65 で保管されていることを示しています。

### CentOS コンテナ の作成・起動

#### コンテナ を作成・起動する

CentOS7 のイメージが取得できたら、さっそく CentOS コンテナを作成して起動してみましょう。

Docker イメージからコンテナを作成して起動するには docker run コマンドを利用します。次のように docker run コマンドを実行して、コンテナを作成・起動してみます。ここでは新しく作成するコンテナに "centos7f" という名前を付けています。CentOS7 のイメージから CentOS7 コンテナを作成し、同時にコンテナも起動し、そのタイミングで自動的にログインし、そのまま bash シェルに接続します。

```
$ docker run -it --name="centos7f" centos:centos7 /bin/bash
#
```

このように、いきなり Centos コンテナ起動し、ログインした状態になります。このまま exit コマンドを実行すれば、このコンテナは停止します。


```
# exit
exit
$
```

#### 起動または存在しているコンテナの確認をする

docker ps コマンドを実行してみてください。

docker ps コマンドは、コンテナリストを表示・取得するコマンドです。現在起動または存在しているコンテナをリストにしてくれます。オプションなしの場合は、現在起動中のコンテナを表示してくれます。

```
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
$
```

今回は、停止された状態なので、現在起動中のコンテナはありません。

下記のように -a、--allオプションをつけた場合は、停止中のコンテナを含めたすべてのコンテナを表示してくれます。

```
$ docker ps -a
CONTAINER ID   IMAGE            COMMAND       CREATED          STATUS                      PORTS     NAMES
b84088954f9f   centos:centos7   "/bin/bash"   30 seconds ago   Exited (0) 24 seconds ago             centos7f
770fe29e2224   hello-world      "/hello"      10 minutes ago   Exited (0) 10 minutes ago             festive_chatelet
$
```

イメージの centos:centos7 からコンテナが"centos7f"という名前で作成・起動され、bin/shコマンドを実行し、現在は、停止した状態であることを示しています。

### Centos コンテナ (仮想環境) の再開・接続

#### 停止中のコンテナを再開する

停止したコンテナを docker start コマンドで再開できます。

"centos7f"　を再開させます。

```
$ docker start centos7f
centos7f
$
```

docker ps コマンドでコンテナ"centos7f"が確かに再開していることを確認しましょう。

```
$ docker ps
CONTAINER ID   IMAGE            COMMAND       CREATED         STATUS          PORTS     NAMES
b84088954f9f   centos:centos7   "/bin/bash"   2 minutes ago   Up 13 seconds             centos7f
$
```

#### 起動中のコンテナへ接続する

起動させたコンテナに接続して操作する場合は、docker attach コマンドまたは docker execコマンド を用います。

docker attach コマンドの場合は、次のようになります。

```
$ docker attach centos7f
#
```

ただし docker attach コマンドでコンテナに接続後、exit コマンドで接続から抜けると、コンテナが停止してしまいます。 コンテナを稼動させたままにしたい場合は、Ctrl + P、Ctrl + Q を続けて押して抜ける必要があります。

ここでは、Ctrl + P、Ctrl + Q を続けて押して、コンテナを稼動させたまま抜けてみます。次のようなメッセージが出力されます。

```
# read escape sequence
$
```

ここで、docker ps コマンドでコンテナの状況を確認してみましょう。
CentOS コンテナ "centos7f" は稼働中です。

```
$ docker ps -a
CONTAINER ID   IMAGE            COMMAND       CREATED          STATUS                      PORTS     NAMES
b84088954f9f   centos:centos7   "/bin/bash"   3 minutes ago    Up About a minute                     centos7f
770fe29e2224   hello-world      "/hello"      13 minutes ago   Exited (0) 13 minutes ago             festive_chatelet
```

この状態であれば、再度、CentOS コンテナ (centos7f) に接続することができます。

今度は、docker attach コマンドではなく docker exec コマンドも用いてコンテナに接続して操作してみましょう。ただし、動作に微妙な違いがあります。

```
$ docker exec -it centos7f /bin/bash
#
```

docker attach コマンド と docker exec コマンドの動作の違いは次の通りです。

|attach コマンド|exec コマンド|
|----|----|
|稼働中のコンテナで起動している PID=1 のプロセスの標準入出力 (STDIN/STDOUT) に接続 (attach) する。|コンテナ内でシェルが動作していなければ接続することができない。|稼働中のコンテナで起動している PID=1 のプロセスの標準入出力 (STDIN/STDOUT) に接続 (attach) する。|稼働中のコンテナで起動している PID=1 のプロセスの標準入出力 (STDIN/STDOUT) に接続( attach) する。|
|attachコマンドで抜けるとコンテナが停止してしまう|exitコマンドで抜けるてもコンテナは停止しない|

## Centos コンテナ (仮想環境) の作成・起動 (バックグランド)

### コンテナを作成しバックグラウンドで起動する

別の ターミナル を起動します。
次のように docker run コマンドを -d で実行して、コンテナを作成しコマンドプロンプトの裏でバックグラウンド起動してみましょう。

ここでは新しく作成するコンテナに "centos7b" という名前をつけています。
centos7 イメージからコンテナを作成してバックグラウンドで起動します。

```
Last login: Mon Dec 14 19:43:55 on ttys000

The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
$ docker run -it -d --name="centos7b" centos:centos7 /bin/bash
019e36aeb68989da7abfaf6ae3a5dd5c4fc0de375a049eee79201d08a35fae10
$
```

## CentOS コンテナ (仮想環境) の停止・削除

コンテナの削除をするには、まずは動ているコンテナを停止させてから削除することになります。

では、docker ps コマンドで動作しているコンテナを確認します。

```
$ docker ps
CONTAINER ID   IMAGE            COMMAND       CREATED          STATUS          PORTS     NAMES
019e36aeb689   centos:centos7   "/bin/bash"   23 seconds ago   Up 23 seconds             centos7b
b84088954f9f   centos:centos7   "/bin/bash"   6 minutes ago    Up 4 minutes              centos7f
$
```

Ctrl + P、Ctrl + Q を続けて押して、稼動させたまま抜けたコンテナ "centos7f" と、先ほどバックグラウンドで起動したコンテナ "centos7b" を確認することができるはずです。

#### 起動しているコンテナの停止する

では、docker stop コマンドで、動作しているコンテナを停止させます。

```
$ docker stop centos7b centos7f
centos7b
centos7f
$
```

docker ps コマンドで、現在のコンテナの稼働状況を確認します。

先に述べた通り、-a、--allオプションをつけた場合は、停止中のコンテナを含めたすべてのコンテナを表示してくれます。

```
$ docker ps -a
CONTAINER ID   IMAGE            COMMAND       CREATED              STATUS                        PORTS     NAMES
019e36aeb689   centos:centos7   "/bin/bash"   About a minute ago   Exited (137) 16 seconds ago             centos7b
b84088954f9f   centos:centos7   "/bin/bash"   7 minutes ago        Exited (137) 16 seconds ago             centos7f
770fe29e2224   hello-world      "/hello"      17 minutes ago       Exited (0) 17 minutes ago               festive_chatelet
```

現在、存在しているコンテナは、"centos7b", "centos7f", "festive_chatelet" の3つであることが確認できるでしょう。"festive_chatelet" というコンテナの名前はイメージ hello-world からコンテナが作成・起動された時に一意的にたまたま決まったものです。なので、違う名前が表示されているかもしれません。

では、docker rm コマンドで、3つのコンテナを削除してみましょう。

#### コンテナ（仮想環境）を削除する

```
$ docker rm centos7b centos7f festive_chatelet
centos7b
centos7f
festive_chatelet
```

削除が完了しますと、削除対象のコンテナ名が出力されます。

```
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

これで、コンテナがすべて削除されました。

#### イメージを削除する

Docker イメージの削除は、docker rmi コマンドを用います。

その前にdocker images コマンドで、取得したDockerイメージを一覧表示できます。

```
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
centos        centos7   8652b9f0cb4c   6 weeks ago     204MB
hello-world   latest    bf756fb1ae65   11 months ago   13.3kB
$
```
docker rmi コマンドを用いて、取得したDockerイメージを削除してみます。

```
$ docker rmi centos:centos7 hello-world:latest
Untagged: centos:centos7
Untagged: centos@sha256:0f4ec88e21daf75124b8a9e5ca03c37a5e937e0e108a255d890492430789b60e
Deleted: sha256:8652b9f0cb4c0599575e5a003f5906876e10c1ceb2ab9fe1786712dac14a50cf
Deleted: sha256:174f5685490326fc0a1c0f5570b8663732189b327007e47ff13d2ca59673db02
Untagged: hello-world:latest
Untagged: hello-world@sha256:1a523af650137b8accdaed439c17d684df61ee4d74feac151b5b337bd29e7eec
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b
Deleted: sha256:9c27e219663c25e0f28493790cc0b88bc973ba3b1686355f221c38a36978ac63
```

これで、docker上からすべてのイメージが削除されました。

# CentOS コンテナ (仮想環境）の整備

centos:centos7 のイメージは、最小構成 (？)でインストールされた CentOS7 環境であるようなので、実際に使用するには、構築した CentOS コンテナに個別に必要なもののインストールや環境の整備の作業をしなければなりません。

まずは、Centos7 のコンテナやイメージなど削除してしまったので、復習をかねて、再度 Docker for Windows 上の CentOS コンテナ (仮想環境) を構築をします。

## CentOS コンテナ (仮想環境) の構築 【復習】

### CentOS の Docker イメージを取得

```
$ docker pull centos:centos7
centos7: Pulling from library/centos
2d473b07cdd5: Pull complete
Digest: sha256:0f4ec88e21daf75124b8a9e5ca03c37a5e937e0e108a255d890492430789b60e
Status: Downloaded newer image for centos:centos7
docker.io/library/centos:centos7
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
centos       centos7   8652b9f0cb4c   6 weeks ago   204MB
$
```

#### CentOS コンテナの作成・起動

```
$ docker run -it --name="centos7" centos:centos7 /bin/bash
#
```

## コマンドのパッケージのインストール

sudo, java, git, ant, wget, unzipなどのコマンドを入力したら、次のような"コマンドが見つかりません"という旨のメッセージが出力されました。

```
# sudo
bash: sudo: command not found
#
```

まずは yum update コマンドで CentOS のパッケージを最新の状態にしておきます。

```
# yum update
Loaded plugins: fastestmirror, ovl
Determining fastest mirrors
 * base: ftp.iij.ad.jp
 * extras: ftp.iij.ad.jp
 * updates: ftp.iij.ad.jp
base                                                     | 3.6 kB     00:00
extras                                                   | 2.9 kB     00:00
updates                                                  | 2.9 kB     00:00
(1/4): base/7/x86_64/group_gz                              | 153 kB   00:00
(2/4): extras/7/x86_64/primary_db                          | 222 kB   00:00
(3/4): updates/7/x86_64/primary_db                         | 4.7 MB   00:00
(4/4): base/7/x86_64/primary_db                            | 6.1 MB   00:01
Resolving Dependencies
--> Running transaction check
---> Package bind-license.noarch 32:9.11.4-26.P2.el7 will be updated
---> Package bind-license.noarch 32:9.11.4-26.P2.el7_9.3 will be an update
---> Package centos-release.x86_64 0:7-9.2009.0.el7.centos will be updated
---> Package centos-release.x86_64 0:7-9.2009.1.el7.centos will be an update
---> Package coreutils.x86_64 0:8.22-24.el7 will be updated
---> Package coreutils.x86_64 0:8.22-24.el7_9.2 will be an update
---> Package curl.x86_64 0:7.29.0-59.el7 will be updated
---> Package curl.x86_64 0:7.29.0-59.el7_9.1 will be an update
---> Package device-mapper.x86_64 7:1.02.170-6.el7 will be updated
---> Package device-mapper.x86_64 7:1.02.170-6.el7_9.3 will be an update
---> Package device-mapper-libs.x86_64 7:1.02.170-6.el7 will be updated
---> Package device-mapper-libs.x86_64 7:1.02.170-6.el7_9.3 will be an update
---> Package glib2.x86_64 0:2.56.1-7.el7 will be updated
---> Package glib2.x86_64 0:2.56.1-8.el7 will be an update
---> Package kpartx.x86_64 0:0.4.9-133.el7 will be updated
---> Package kpartx.x86_64 0:0.4.9-134.el7_9 will be an update
---> Package libcurl.x86_64 0:7.29.0-59.el7 will be updated
---> Package libcurl.x86_64 0:7.29.0-59.el7_9.1 will be an update
---> Package openssl-libs.x86_64 1:1.0.2k-19.el7 will be updated
---> Package openssl-libs.x86_64 1:1.0.2k-21.el7_9 will be an update
---> Package python.x86_64 0:2.7.5-89.el7 will be updated
---> Package python.x86_64 0:2.7.5-90.el7 will be an update
---> Package python-libs.x86_64 0:2.7.5-89.el7 will be updated
---> Package python-libs.x86_64 0:2.7.5-90.el7 will be an update
---> Package systemd.x86_64 0:219-78.el7 will be updated
---> Package systemd.x86_64 0:219-78.el7_9.2 will be an update
---> Package systemd-libs.x86_64 0:219-78.el7 will be updated
---> Package systemd-libs.x86_64 0:219-78.el7_9.2 will be an update
---> Package vim-minimal.x86_64 2:7.4.629-7.el7 will be updated
---> Package vim-minimal.x86_64 2:7.4.629-8.el7_9 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package               Arch      Version                       Repository  Size
================================================================================
Updating:
 bind-license          noarch    32:9.11.4-26.P2.el7_9.3       updates     91 k
 centos-release        x86_64    7-9.2009.1.el7.centos         updates     27 k
 coreutils             x86_64    8.22-24.el7_9.2               updates    3.3 M
 curl                  x86_64    7.29.0-59.el7_9.1             updates    271 k
 device-mapper         x86_64    7:1.02.170-6.el7_9.3          updates    297 k
 device-mapper-libs    x86_64    7:1.02.170-6.el7_9.3          updates    325 k
 glib2                 x86_64    2.56.1-8.el7                  updates    2.5 M
 kpartx                x86_64    0.4.9-134.el7_9               updates     81 k
 libcurl               x86_64    7.29.0-59.el7_9.1             updates    223 k
 openssl-libs          x86_64    1:1.0.2k-21.el7_9             updates    1.2 M
 python                x86_64    2.7.5-90.el7                  updates     96 k
 python-libs           x86_64    2.7.5-90.el7                  updates    5.6 M
 systemd               x86_64    219-78.el7_9.2                updates    5.1 M
 systemd-libs          x86_64    219-78.el7_9.2                updates    418 k
 vim-minimal           x86_64    2:7.4.629-8.el7_9             updates    443 k

Transaction Summary
================================================================================
Upgrade  15 Packages

Total download size: 20 M
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
warning: /var/cache/yum/x86_64/7/updates/packages/bind-license-9.11.4-26.P2.el7_9.3.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for bind-license-9.11.4-26.P2.el7_9.3.noarch.rpm is not installed
(1/15): bind-license-9.11.4-26.P2.el7_9.3.noarch.rpm       |  91 kB   00:00
(2/15): centos-release-7-9.2009.1.el7.centos.x86_64.rpm    |  27 kB   00:00
(3/15): curl-7.29.0-59.el7_9.1.x86_64.rpm                  | 271 kB   00:00
(4/15): device-mapper-1.02.170-6.el7_9.3.x86_64.rpm        | 297 kB   00:00
(5/15): device-mapper-libs-1.02.170-6.el7_9.3.x86_64.rpm   | 325 kB   00:00
(6/15): kpartx-0.4.9-134.el7_9.x86_64.rpm                  |  81 kB   00:00
(7/15): libcurl-7.29.0-59.el7_9.1.x86_64.rpm               | 223 kB   00:00
(8/15): glib2-2.56.1-8.el7.x86_64.rpm                      | 2.5 MB   00:00
(9/15): openssl-libs-1.0.2k-21.el7_9.x86_64.rpm            | 1.2 MB   00:00
(10/15): python-2.7.5-90.el7.x86_64.rpm                    |  96 kB   00:00
(11/15): coreutils-8.22-24.el7_9.2.x86_64.rpm              | 3.3 MB   00:00
(12/15): systemd-219-78.el7_9.2.x86_64.rpm                 | 5.1 MB   00:00
(13/15): systemd-libs-219-78.el7_9.2.x86_64.rpm            | 418 kB   00:00
(14/15): vim-minimal-7.4.629-8.el7_9.x86_64.rpm            | 443 kB   00:00
(15/15): python-libs-2.7.5-90.el7.x86_64.rpm               | 5.6 MB   00:00
--------------------------------------------------------------------------------
Total                                               14 MB/s |  20 MB  00:01
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-9.2009.0.el7.centos.x86_64 (@CentOS)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : systemd-libs-219-78.el7_9.2.x86_64                          1/30
  Updating   : 1:openssl-libs-1.0.2k-21.el7_9.x86_64                       2/30
  Updating   : coreutils-8.22-24.el7_9.2.x86_64                            3/30
  Updating   : libcurl-7.29.0-59.el7_9.1.x86_64                            4/30
  Updating   : python-libs-2.7.5-90.el7.x86_64                             5/30
  Updating   : centos-release-7-9.2009.1.el7.centos.x86_64                 6/30
  Updating   : systemd-219-78.el7_9.2.x86_64                               7/30
Failed to get D-Bus connection: Operation not permitted
  Updating   : 7:device-mapper-libs-1.02.170-6.el7_9.3.x86_64              8/30
  Updating   : 7:device-mapper-1.02.170-6.el7_9.3.x86_64                   9/30
  Updating   : kpartx-0.4.9-134.el7_9.x86_64                              10/30
  Updating   : python-2.7.5-90.el7.x86_64                                 11/30
  Updating   : curl-7.29.0-59.el7_9.1.x86_64                              12/30
  Updating   : glib2-2.56.1-8.el7.x86_64                                  13/30
  Updating   : 2:vim-minimal-7.4.629-8.el7_9.x86_64                       14/30
  Updating   : 32:bind-license-9.11.4-26.P2.el7_9.3.noarch                15/30
  Cleanup    : curl-7.29.0-59.el7.x86_64                                  16/30
  Cleanup    : kpartx-0.4.9-133.el7.x86_64                                17/30
  Cleanup    : 7:device-mapper-1.02.170-6.el7.x86_64                      18/30
  Cleanup    : 7:device-mapper-libs-1.02.170-6.el7.x86_64                 19/30
  Cleanup    : systemd-219-78.el7.x86_64                                  20/30
  Cleanup    : python-2.7.5-89.el7.x86_64                                 21/30
  Cleanup    : centos-release-7-9.2009.0.el7.centos.x86_64                22/30
  Cleanup    : 32:bind-license-9.11.4-26.P2.el7.noarch                    23/30
  Cleanup    : python-libs-2.7.5-89.el7.x86_64                            24/30
  Cleanup    : coreutils-8.22-24.el7.x86_64                               25/30
  Cleanup    : 1:openssl-libs-1.0.2k-19.el7.x86_64                        26/30
  Cleanup    : libcurl-7.29.0-59.el7.x86_64                               27/30
  Cleanup    : systemd-libs-219-78.el7.x86_64                             28/30
  Cleanup    : glib2-2.56.1-7.el7.x86_64                                  29/30
  Cleanup    : 2:vim-minimal-7.4.629-7.el7.x86_64                         30/30
  Verifying  : python-2.7.5-90.el7.x86_64                                  1/30
  Verifying  : kpartx-0.4.9-134.el7_9.x86_64                               2/30
  Verifying  : centos-release-7-9.2009.1.el7.centos.x86_64                 3/30
  Verifying  : 7:device-mapper-1.02.170-6.el7_9.3.x86_64                   4/30
  Verifying  : 32:bind-license-9.11.4-26.P2.el7_9.3.noarch                 5/30
  Verifying  : coreutils-8.22-24.el7_9.2.x86_64                            6/30
  Verifying  : libcurl-7.29.0-59.el7_9.1.x86_64                            7/30
  Verifying  : 1:openssl-libs-1.0.2k-21.el7_9.x86_64                       8/30
  Verifying  : curl-7.29.0-59.el7_9.1.x86_64                               9/30
  Verifying  : python-libs-2.7.5-90.el7.x86_64                            10/30
  Verifying  : 2:vim-minimal-7.4.629-8.el7_9.x86_64                       11/30
  Verifying  : 7:device-mapper-libs-1.02.170-6.el7_9.3.x86_64             12/30
  Verifying  : systemd-libs-219-78.el7_9.2.x86_64                         13/30
  Verifying  : systemd-219-78.el7_9.2.x86_64                              14/30
  Verifying  : glib2-2.56.1-8.el7.x86_64                                  15/30
  Verifying  : 2:vim-minimal-7.4.629-7.el7.x86_64                         16/30
  Verifying  : systemd-libs-219-78.el7.x86_64                             17/30
  Verifying  : glib2-2.56.1-7.el7.x86_64                                  18/30
  Verifying  : python-2.7.5-89.el7.x86_64                                 19/30
  Verifying  : kpartx-0.4.9-133.el7.x86_64                                20/30
  Verifying  : 32:bind-license-9.11.4-26.P2.el7.noarch                    21/30
  Verifying  : centos-release-7-9.2009.0.el7.centos.x86_64                22/30
  Verifying  : python-libs-2.7.5-89.el7.x86_64                            23/30
  Verifying  : 7:device-mapper-1.02.170-6.el7.x86_64                      24/30
  Verifying  : 7:device-mapper-libs-1.02.170-6.el7.x86_64                 25/30
  Verifying  : systemd-219-78.el7.x86_64                                  26/30
  Verifying  : coreutils-8.22-24.el7.x86_64                               27/30
  Verifying  : 1:openssl-libs-1.0.2k-19.el7.x86_64                        28/30
  Verifying  : libcurl-7.29.0-59.el7.x86_64                               29/30
  Verifying  : curl-7.29.0-59.el7.x86_64                                  30/30

Updated:
  bind-license.noarch 32:9.11.4-26.P2.el7_9.3
  centos-release.x86_64 0:7-9.2009.1.el7.centos
  coreutils.x86_64 0:8.22-24.el7_9.2
  curl.x86_64 0:7.29.0-59.el7_9.1
  device-mapper.x86_64 7:1.02.170-6.el7_9.3
  device-mapper-libs.x86_64 7:1.02.170-6.el7_9.3
  glib2.x86_64 0:2.56.1-8.el7
  kpartx.x86_64 0:0.4.9-134.el7_9
  libcurl.x86_64 0:7.29.0-59.el7_9.1
  openssl-libs.x86_64 1:1.0.2k-21.el7_9
  python.x86_64 0:2.7.5-90.el7
  python-libs.x86_64 0:2.7.5-90.el7
  systemd.x86_64 0:219-78.el7_9.2
  systemd-libs.x86_64 0:219-78.el7_9.2
  vim-minimal.x86_64 2:7.4.629-8.el7_9

Complete!
#
```

エラーが生じた場合、[こちら](https://qiita.com/gahoh/items/7b21377b5c9e3ffddf4a#コマンドのパッケージのインストール)を参考してください。

yum install コマンドで、sudo のパッケージをインストールします。

```
# yum -y install sudo
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: ftp.iij.ad.jp
 * extras: ftp.iij.ad.jp
 * updates: ftp.iij.ad.jp
Resolving Dependencies
--> Running transaction check
---> Package sudo.x86_64 0:1.8.23-10.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package        Arch             Version                   Repository      Size
================================================================================
Installing:
 sudo           x86_64           1.8.23-10.el7             base           842 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 842 k
Installed size: 3.0 M
Downloading packages:
sudo-1.8.23-10.el7.x86_64.rpm                              | 842 kB   00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : sudo-1.8.23-10.el7.x86_64                                    1/1
  Verifying  : sudo-1.8.23-10.el7.x86_64                                    1/1

Installed:
  sudo.x86_64 0:1.8.23-10.el7

Complete!
#
```

これで sudo コマンドのパッケージがインストールができました。

```
# sudo
usage: sudo -h | -K | -k | -V
usage: sudo -v [-AknS] [-g group] [-h host] [-p prompt] [-u user]
usage: sudo -l [-AknS] [-g group] [-h host] [-p prompt] [-U user] [-u user]
            [command]
usage: sudo [-AbEHknPS] [-r role] [-t type] [-C num] [-g group] [-h host] [-p
            prompt] [-T timeout] [-u user] [VAR=value] [-i|-s] [<command>]
usage: sudo -e [-AknS] [-r role] [-t type] [-C num] [-g group] [-h host] [-p
            prompt] [-T timeout] [-u user] file ...
#
```

sudo以外に java, git, ant, wget, unzipなどのコマンドのパッケージをインストールしておきます。

```
# yum -y install java-1.8.0-openjdk-devel
# yum -y install git
# yum -y install ant
# yum -y install wget
# yum -y install unzip
```

sudo などのコマンドをインストールした CentOS のコンテナの"centos7" を停止して抜けます。

```
# exit
exit
$
```

## 作業済み CentOS コンテナ (仮想環境）の保存

今回、CentOS コンテナ (仮想環境）に対して sudo コマンドをインストールをしました。

このように設定や更新などのカスタマイズ作業をおこなったコンテナの状態をイメージとして保存しておくことができます。
では、今回のカスタマイズ作業をおこなった CentOS コンテナ からイメージを作って保存をしてみます。

### 保存するコンテナを停止する

まず、 オプションなしで、現在起動中のコンテナを確認します。

```
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
$
```

今回は、sudo などのコマンドのインストールしたCentOS のコンテナの"centos7" は、停止状態のはずなので、現在起動中のコンテナはありません。

コンテナが動いている状態でコンテナに接続されていない場合は docker stop コマンドで停止させることができます。

次に -a オプションをつけて、停止中のコンテナを含めたすべてのコンテナを表示します。

```
$ docker ps -a
CONTAINER ID   IMAGE            COMMAND       CREATED          STATUS                            PORTS     NAMES
38cad34ec558   centos:centos7   "/bin/bash"   17 minutes ago   Exited (127) About a minute ago             centos7
$
```

sudo などのコマンドのインストールなどの作業が行われ、現在停止した状態である CentOS コンテナの"centos7"が表示されます。

### 停止したコンテナをイメージに保存する

docker images コマンドで、取得したイメージを確認します。

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
centos       centos7   8652b9f0cb4c   6 weeks ago   204MB
$
```

カスタマイズ作業をおこなった CentOS コンテナの"centos"を docker commit コマンドで、Docker イメージ "centos:gahoh" として保存します。

```
$ docker commit centos7 centos:gahoh
sha256:8beaf0d882f91e8d100b5b67696cc001ff57c743188024b6b2ead373aff28dee
$
```

なお、一時停止せず、コンテナが動いたままの状態でDockerイメージを作成する場合は「--pause=false」オプションを使用します。

docker imagesコマンドにて、Dockerイメージが作成されているか確認します

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
centos       gahoh     8beaf0d882f9   46 minutes ago   663MB
centos       centos7   8652b9f0cb4c   6 weeks ago      204MB
$
```

新たに Dokcer イメージ "centos:gahoh" が追加されたことがわかります。

#### 作成されたイメージの内容を確認す

先ほど作業した内容がちゃんと反映されている イメージなのか、確認してみましょう。

docker rm コマンドで、コンテナ "centos7" を削除します。

```
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
cd115c8ce57b   centos:gahoh   "/bin/bash"   46 minutes ago   Exited (0) 21 minutes ago             centos
$
$ docker rm centos7
centos7
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
$
```

これで、すべてのコンテナがなくなりました。

では、docker run コマンドで、イメージ "centos:gahoh" からコンテナ "centos7" を作成・起動してみます。

```
$ docker run -it --name="centos7" centos:gahoh /bin/bash
#
```

sudo コマンドを実行してみます。


```
# sudo
usage: sudo -h | -K | -k | -V
usage: sudo -v [-AknS] [-g group] [-h host] [-p prompt] [-u user]
usage: sudo -l [-AknS] [-g group] [-h host] [-p prompt] [-U user] [-u user]
            [command]
usage: sudo [-AbEHknPS] [-r role] [-t type] [-C num] [-g group] [-h host] [-p
            prompt] [-T timeout] [-u user] [VAR=value] [-i|-s] [<command>]
usage: sudo -e [-AknS] [-r role] [-t type] [-C num] [-g group] [-h host] [-p
            prompt] [-T timeout] [-u user] file ...
#
```


## さいごに

今回、MacBookに Doker Desktop for Mac をインストールし、CentOS の仮想環境を構築・設定・保存などの一連の手順を詳細に記述したつもりです。
もし、記述について誤りがあったり、気になることがあれば、編集リクエストやコメントでフィードバックしていただけると助かります。