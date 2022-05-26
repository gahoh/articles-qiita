<!--
title:   【入門】SQLWorkbench/Jを用いたJDBC経由のGridDB接続とデータベース操作
tags:    Docker,GridDB,JDBC,SQL,sqlworkbench
id:      8954bb2cd7947441c42f
private: false
-->
GridDBは、NoSQLとSQLのデュアルインタフェースを備えているデータベースです。リアルタイム性の高い登録や参照にはNoSQLインタフェース、大規模なデータ集約・加工にはSQLインタフェースというようにうまく使い分けることができるようです。

今回、Docker DesktopのCentOSコンテナで、JDBC (Java Database Connectivity) ドライバ経由で動作するオープンソースの SQL クライアントツール SQLWorkbench/Jをインストールして、JDBC経由でのGridDBデータベースに接続する方法についてステップバイステップで紹介します。また、典型的なGridDBのデータモデルの具体例を用いて、SQL構文を解説しながら、SQLWorkbench/JでのGridDBデータベースのSQL操作についても紹介します。

- 前提条件
- 事前準備
 - GridDB 構築済みの CentOS コンテナ のビルド・起動
 - GridDBサーバの起動
 - GridDB JDBCドライバーの構築
- SQL Workbench/J の入手とインストール
- SQL Workbench/J でのJDBC経由によるGridDBとの接続確認
- データベース操作をしてみる

# 前提条件

今回は、すでに Docker Desktop で CentOS コンテナの環境が構築され その上にGridDB Community Edition のセットアップされ動作確認されていることを前提としています。これらを環境を用意する場合は、次の記事を参照していただければと思います。

[【入門】はじめての Docker Desktop の CentOS コンテナでの GridDB Community Edition のセットアップ](2021-01-12_CentOS_Docker_DockerDesktop_GridDB_a8fbb6911551bc98c041.md)

以降、ここでは、GridDB Community Edition のセットアップ・動作確認された CentOS のイメージの名前をcentos:gahoh とします。

# 事前準備

## GridDB 構築済みの CentOS コンテナ のビルド・起動

GridDB Community Edition のセットアップ・動作確認された CentOS のイメージ centos:gahoh からCentOS コンテナをビルドして起動します。

```
$ docker run -it --name="centos" centos:gahoh /bin/bash
#
```

## GridDB サーバの起動

```
# su - gsadm
Last login: Mon Jan 18 09:44:28 UTC 2021 on pts/0
$ cd
$ pwd
/var/lib/gridstore
$ gs_startnode -u admin/admin -w
.
Started node.
$ gs_joincluster -c myCluster -u admin/admin -w
..
Joined node
$ gs_stat -u admin/admin | egrep Status
        "clusterStatus": "MASTER",
        "nodeStatus": "ACTIVE",
        "partitionStatus": "NORMAL",
```

### 設定内容

「[【入門】はじめての Docker Desktop の CentOS コンテナでの GridDB Community Edition のセットアップ](2021-01-12_CentOS_Docker_DockerDesktop_GridDB_a8fbb6911551bc98c041.md)」による GridDB サーバの設定内容は、クラスタ名以外は、デフォルト値の設定のままになっています。次に主な設定項目を示します。

|設定項目|値|
|:----|:----|
|クラスタ名|myCluster|
|マルチキャストアドレス|239.0.0.1 (デフォルト値)|
|マルチキャストポート番号|31999 (デフォルト値)|
|ユーザ名|admin (デフォルト値)|
|ユーザパスワード|admin (デフォルト値)|
|SQLマルチキャストアドレス|239.0.0.1 (デフォルト値)|
|SQLマルチキャストポート番号|41999 (デフォルト値)|

もし、これらの設定内容を変更している場合は、必要に応じてパラメータなどの修正が必要となりますので注意願いまます。

## GridDB JDBCドライバーの構築

git clone コマンドを用いて、Gitのリポジトリ https://github.com/griddb/jdbc をディレクトリ griddb_jdbc に複製します。

```
$ git clone https://github.com/griddb/jdbc.git griddb_jdbc
Cloning into 'griddb_jdbc'...
remote: Enumerating objects: 208, done.
remote: Total 208 (delta 0), reused 0 (delta 0), pack-reused 208
Receiving objects: 100% (208/208), 403.07 KiB | 365.00 KiB/s, done.
Resolving deltas: 100% (43/43), done.
$
```

### ビルド

Javaのビルドツール Apache Antを用いて GridDB JDBCドライバをビルドします。

```
$ cd griddb_jdbc/
$ ant
Buildfile: /var/lib/gridstore/griddb_jdbc/build.xml

compile:
    [mkdir] Created dir: /var/lib/gridstore/griddb_jdbc/obj/main
    [javac] Compiling 65 source files to /var/lib/gridstore/griddb_jdbc/obj/main
    [javac] warning: [options] bootstrap class path not set in conjunction with -source 1.6
    [javac] Note: Some input files use or override a deprecated API.
    [javac] Note: Recompile with -Xlint:deprecation for details.
    [javac] Note: Some input files use unchecked or unsafe operations.
    [javac] Note: Recompile with -Xlint:unchecked for details.
    [javac] 1 warning

mainJar:
    [mkdir] Created dir: /var/lib/gridstore/griddb_jdbc/bin
      [jar] Building jar: /var/lib/gridstore/griddb_jdbc/bin/gridstore-jdbc.jar

callLoggingCompile:
    [mkdir] Created dir: /var/lib/gridstore/griddb_jdbc/obj/call_logging
    [javac] Compiling 3 source files to /var/lib/gridstore/griddb_jdbc/obj/call_logging
    [javac] warning: [options] bootstrap class path not set in conjunction with -source 1.6
    [javac] 1 warning

callLoggingJar:
      [jar] Building jar: /var/lib/gridstore/griddb_jdbc/bin/gridstore-jdbc-call-logging.jar

jar:

BUILD SUCCESSFUL
Total time: 5 seconds
$
```

これで、bin/ フォルダの下に次のファイルが作成されます。

　JDBCドライバー　gridstore-jdbc.jar
　JDBCログイングドライバー　gridstore-jdbc-call-logging.jar

```
$ ls bin
gridstore-jdbc-call-logging.jar  gridstore-jdbc.jar
```

次に、JDBCドライバファイル gridstore-jdbc.jar を /usr/share/java/ の下にコピーしておきます。

```
$ sudo cp bin/gridstore-jdbc.jar /usr/share/java/

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for gsadm:
$
```

### サンプルプログラムの実行

JDBCドライバーがちゃんと構築されているか確かめるため、サンプルプログラムを実行してみます。

```
import java.net.URLEncoder;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.Properties;

public class JDBCSelect {

	public static void main(String[] args){
		try {
			//===============================================
			// クラスタに接続する
			//===============================================
			// JDBCの接続情報
			String jdbcAddress = "239.0.0.1";
			String jdbcPort = "41999";
			String clusterName = "myCluster";
			String databaseName = "public";
			String username = "admin";
			String password = "admin";
			String applicationName = "SampleJDBC";

			// クラスタ名とデータベース名はエンコードする
			String encodeClusterName = URLEncoder.encode(clusterName, "UTF-8");
			String encodeDatabaseName = URLEncoder.encode(databaseName, "UTF-8");

			// URLを組み立てる (マルチキャスト方式)
			String jdbcUrl = "jdbc:gs://"+jdbcAddress+":"+jdbcPort+"/"+encodeClusterName+"/"+encodeDatabaseName;

			Properties prop = new Properties();
			prop.setProperty("user", username);
			prop.setProperty("password", password);
			prop.setProperty("applicationName", applicationName);

			// 接続する
			Connection con = DriverManager.getConnection(jdbcUrl, prop);

			//===============================================
			// コンテナ作成とデータ登録
			//===============================================
			{
				Statement stmt = con.createStatement();

				// (存在した場合は削除する)
				stmt.executeUpdate("DROP TABLE IF EXISTS SampleJDBC_Select");

				// コンテナ作成
				stmt.executeUpdate("CREATE TABLE IF NOT EXISTS SampleJDBC_Select ( id integer PRIMARY KEY, value string )");
				System.out.println("SQL Create Table name=SampleJDBC_Select");

				// ロウ登録
				int ret1 = stmt.executeUpdate("INSERT INTO SampleJDBC_Select values "+
											"(0, 'test0'),(1, 'test1'),(2, 'test2'),(3, 'test3'),(4, 'test4')");
				System.out.println("SQL Insert count=" + ret1);

				stmt.close();
			}

			//===============================================
			// 検索の実行
			//===============================================
			// (1)ステートメント作成
			Statement stmt = con.createStatement();

			// (2)SQL実行
			ResultSet rs = stmt.executeQuery("SELECT * from SampleJDBC_Select where ID > 2");

			// (3)結果の取得
			while( rs.next() ){
				int id = rs.getInt(1);
				String value = rs.getString(2);
				System.out.println("SQL row(id=" + id + ", value=" + value + ")");
			}

			//===============================================
			// 終了処理
			//===============================================
			stmt.close();
			con.close();
			System.out.println("success!");

		} catch ( Exception e ){
			e.printStackTrace();
		}
	}
}
```




なお、起動した GridDB サーバのクラスタ名、ユーザ名、パスワードなどの設定内容を変更している場合は、ソースコード JDBCSelect.javaを編集して、クラスタ名、ユーザ名、パスワードなどの設定パラメータを修正する必要があります。

```
$ cd sample/ja/jdbc
$ javac -encoding UTF-8 JDBCSelect.java
$ export CLASSPATH=:/usr/share/java/gridstore-jdbc.jar
$ java JDBCSelect
SQL Create Table name=SampleJDBC_Select
SQL Insert count=5
SQL row(id=3, value=test3)
SQL row(id=4, value=test4)
success!
$
```

これでJDBCドライバーがちゃんと構築されていることが確認できました。次は、SQL Workbench/J のインストールです。

# SQL Workbench/J のインストール

[SQL Workbench/J] (https://www.sql-workbench.eu/) は、 Thomas Kellerer により開発・提供されている JDBC ドライバ経由で動作するオープンソースの SQL クライアントツールです。

Javaランタイム上で動くOS非依存のJavaアプリケーションで、ライセンスの利用規約としては一部（特定の国の政府機関など）利用を制限しています。日本での利用では特に問題はありません。

## SQL Workbench/Jの入手方法とインストール方法

SQL Workbench/Jは、専用ホームページの[ダウンロード画面](https://www.sql-workbench.eu/downloads.html)より入手してください。
[ダウンロード画面](https://www.sql-workbench.eu/downloads.html)には、数種類のダウンロードリンクがありますが「Generic package for all systems including all optional libraries (sha1) https://www.sql-workbench.eu/Workbench-Build127-with-optional-libs.zip が、現在の安定版であり、jarファイル、マニュアル（HTMLおよびPDF）、アプリケーションを起動するためのLinux / Unixベースのシステム（MacOSを含む）のシェルスクリプト、 およびWindows®ランチャーとサンプルXSLTスクリプトが含まれています。
ダウンロードしたファイルは、ZIP形式で圧縮されていますので、任意の場所に解凍してください。これで、SQL Workbench/Jのインストールは終了です。

具体的なインストールの大まかな流れは次の通りです。

```
# wget https://www.sql-workbench.eu/Workbench-Build127-with-optional-libs.zip
# mkdir sqlworkbench
# cd sqlworkbench
# unzip ../Workbench-Build127-with-optional-libs.zip
# chmod +x *.sh
# ./sqlwbconsole.sh
```
では、詳細を見ていきましょう。

### ダウンロード

wgetコマンドで、URLを指定してダウンロードします。

```
$ exit
#
# wget https://www.sql-workbench.eu/Workbench-Build127-with-optional-libs.zip
--2021-01-18 06:23:01--  https://www.sql-workbench.eu/Workbench-Build127-with-optional-libs.zip
Resolving www.sql-workbench.eu (www.sql-workbench.eu)... 217.160.0.56, 2001:8d8:100f:f000::20b
Connecting to www.sql-workbench.eu (www.sql-workbench.eu)|217.160.0.56|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 28852920 (28M) [application/zip]
Saving to: 'Workbench-Build127-with-optional-libs.zip'

100%[==============================================================================>] 28,852,920  2.53MB/s   in 12s

2021-01-18 06:23:14 (2.26 MB/s) - 'Workbench-Build127-with-optional-libs.zip' saved [28852920/28852920]
#
```

### 解凍

```
# mkdir sqlworkbench
# cd sqlworkbench
# unzip ../Workbench-Build127-with-optional-libs.zip
Archive:  ../Workbench-Build127-with-optional-libs.zip
  inflating: sqlworkbench.jar
   creating: ext/
  inflating: ext/commons-codec.jar
  (途中省略)
  inflating: workbench32.png
   creating: manual/
  inflating: manual/SQLWorkbench-Manual.pdf
  (途中省略)
  inflating: manual/workspace-usage.html
   creating: xslt/
  inflating: xslt/jdbctypes2oracle.xslt
  (途中省略)
  inflating: xslt/wbreport2proc.xslt
#
```

これで、SQL Workbench/Jのインストールは完了です。これでコマンドライン SQLWorkbench/Jを実行できるようになりました。

# SQL Workbench/J でのJDBC経由によるGridDBとの接続確認

## 起動

コマンドライン SQLWorkbench/J を起動します。

```
su - gsadm
Last login: Mon Jan 25 05:36:08 UTC 2021 on pts/0
$ cd /sqlworkbench/
$ ./sqlwbconsole.sh

SQL Workbench/J (127) console interface started.
Enter exit to quit.
Enter WbHelp for a list of SQL Workbench/J specific commands
Config directory: /var/lib/gridstore/.sqlworkbench

SQL>
```

## GridDBへの接続

WbConnectコマンドでGridDBに接続します。

> WbConnect -driver=com.toshiba.mwcloud.gs.sql.Driver -driverJar=/usr/share/java/gridstore-jdbc.jar -url=jdbc:gs<b/></b>://239.0.0.1:41999/myCluster/public -username=admin -password=admin;

パラメータとして、JDBCドライバのクラス名、jarファイル、JDBCの接続URL、DBMSのユーザ名、そのパスワードを指定します。

|パラメータ|説明|値|
|:----|:----|:----|
|-driver|JDBCドライバーの完全なクラス名を指定します|com.toshiba.mwcloud.gs.sql.Driver|
|-driverJar|JDBCドライバーを含む.jarファイルへのフルパス名を指定します|/usr/share/java/gridstore-jdbc.jar|
|-url|JDBC接続時の URL 形式|jdbc:gs ://239.0.0.1:41999/myCluster/public|
|-username|DBMSのユーザ名を指定します|admin (デフォルト値)|
|-password|ユーザパスワードを指定します|admin (デフォルト値)|

#### 補足 パラメータ -url

接続方法によって、パラメータ -urlの形式が異なります。デフォルトのマルチキャスト方式でクラスタ内のノードに自動接続の場合は、次のようになります。

`jdbc:gs://(multicastAddress):(portNo)/(clusterName)/(databaseName)`

- multicastAddress：GridDBクラスタとの接続に使うマルチキャストアドレス。(デフォルト: 239.0.0.1)
- portNo：GridDBクラスタとの接続に使うポート番号。(デフォルト: 41999)
- clusterName：GridDBクラスタのクラスタ名
- databaseName：データベース名。省略した場合はデフォルトデータベース(public)に接続します。

固定リスト方式、プロバイダ方式などで接続する場合は、「[GridDB SQLリファレンス](https://github.com/griddb/docs-ja/blob/master/manuals/GridDB_JDBC_Driver_UserGuide/overview.md#%E6%8E%A5%E7%B6%9A%E6%99%82%E3%81%AEurl%E5%BD%A2%E5%BC%8F)」を参照願います。

```
SQL> WbConnect -driver=com.toshiba.mwcloud.gs.sql.Driver -driverJar=/usr/share/java/gridstore-jdbc.jar -url=jdbc:gs://239.0.0.1:41999/myCluster/public -username=admin -password=admin;

Connection to "User=admin, URL=jdbc:gs://239.0.0.1:41999/myCluster/public" successful
```

## データの読み込み

先のサンプルプログラムで作成したデータを読み込みます。

>  SELECT * FROM SampleJDBC_Select;

```
SQL> SELECT * FROM SampleJDBC_Select;

id | value
---+------
 0 | test0
 1 | test1
 2 | test2
 3 | test3
 4 | test4

(5 Rows)
SELECT executed successfully
```

これで動作確認は終わりです。

# データベース操作をしてみる

今回のSQL Workbench/J を用いたデータベース操作の大まかな流れは、次の通りとなります。

- テーブルの作成
- データの登録（格納）
- データの取得（検索）
- テーブルの削除

実際にSQL Workbench/J を用いてデータベースを操作する前に、今回の操作するデータベース (例)の内容とGridDBでサポートされているSQLについて紹介します。

## データベース (例)の内容

GridDBにデータを登録し検索するには、データを格納する**コンテナ (テーブル)** を作成する必要があります。なお、NoSQLインタフェースで操作する場合はコンテナ、NewSQLインタフェース (SQLインタフェース) で操作する場合はテーブルと呼ぶようです。今回、SQLで操作するので、以降、テーブルと呼びます。
テーブルには、一般的なさまざまなデータを管理するための**コレクションテーブル**と時系列データを管理するのに適した**時系列テーブル**の2つのタイプがあります。

今回、ある施設で、各階の各部屋に装置が置かれ、その装置がアラートを検知すると、その検知時刻とアラートレベル、アラート詳細情報をデータベースに登録するような例を用います。それには次のデータソースが存在することとします。

### 装置管理情報

装置毎の定義情報を表現するものであり、装置毎に割り当てられた装置ID、装置タイプおよびその装置が設置されている階数とその設置されている部屋番号で構成されます。

|装置ID:id|装置タイプ:type|設置階数:floor|設置部屋番号:room|
|:-------|:-------------|:------------|--------------|

装置管理情報は、一般のロウとカラムから構成されるコレクションテーブルである装置管理情報テーブル（テーブル名：equipTable)に格納します。

### アラート履歴情報

検知されたアラート情報を表現するものであり、発生したアラートの日付時刻、アラートのレベル、およびアラート詳細情報で構成されます。

|検知時刻:date|装置ID:id|アラートレベル:level|アラート詳細情報:detail|
|:-----------|:-------|:-----------------|:-------------------|

アラート履歴情報は、発生するデータを発生した時刻とともに管理するのに適したタイプの時系列テーブルであるアラート履歴情報テーブル（テーブル名：alertTable)に格納します。

## GridDBでサポートされている SQL


GridDBでサポートされるSQL文は、次の通りです。


| コマンド                             | 概要                                                |
|-------------------------------------|----------------------------------------------------|
| [CREATE DATABASE](#create-database) | データベースを作成する。                             |
| [CREATE TABLE](#create-table)       | テーブルを作成する。                                 |
| [CREATE INDEX](#create-index)       | 索引を作成する。                                     |
| [CREATE VIEW](#create-view)         | ビューを作成する。                                   |
| [CREATE USER](#create-user)         | 一般ユーザを作成する。                               |
| [DROP DATABASE](#drop-database)     | データベースを削除する。                             |
| [DROP TABLE](#drop-table)           | テーブルを削除する。                                 |
| [DROP INDEX](#drop-index)           | 索引を削除する。                                     |
| [DROP VIEW](#drop-view)             | ビューを削除する。                                   |
| [DROP USER](#drop-user)             | 一般ユーザを削除する。                               |
| [ALTER TABLE](#alter-table)         | テーブルの構造を変更します。                         |
| [GRANT](#grant)                     | 一般ユーザにデータベースへのアクセス権を設定する。     |
| [REVOKE](#revoke)                   | 一般ユーザからデータベースへのアクセス権を削除する。   |
| [SET PASSWORD](#set-password)       | 一般ユーザのパスワードを変更する。                   |
| [SELECT](#select)                   | データを取得する。                                   |
| [INSERT](#insert)                   | テーブルに行を挿入する。                             |
| [DELETE](#delete)                   | テーブルから行を削除する。                           |
| [UPDATE](#update)                   | テーブルにある行を更新する。                         |
| [コメント](#comment)                | コメントを表記する。                                 |
| [ヒント](#hint)                     | 実行計画を制御する。                                 |

GridDBのSQLは基本的にSQL-92がベースになっているようです。
SQL文としては、SELECT文、INSERT文、UPDATE文、DELETE文などのデータ操作言語 (DML)、CREATE文やDROP文などのデータ定義言語(DDL) や GRANT文、REVOKE文などのデータ制御言語 (DCL) をサポートしています。
句としては、FROM句、GROUP BY句、HAVING句、ORDER BY句、WHERE句、LIMIT句/OFFSET句、JOIN句、UNION/INTERSECT/EXCEPT句、OVER句をサポートしてます。特にJOIN については、内部結合 [NATURAL] [INNER] JOIN、左外部結合 [NATURAL] LEFT [OUTER] JOIN、クロス結合 [NATURAL] CROSS JOINに対応しているよう複数のテーブルを跨いだJOINが可能なようです。
また、もちろん、演算子、関数、CASEなどもサポートしています。
詳細は「[GridDB SQLリファレンス](https://github.com/griddb/docs-ja/blob/master/manuals/GridDB_SQL_Reference/sql-commands-supported.md#sql_commands_supported_by_griddb_ae)」を参照してください。


## テーブル作成 (CREATE TABLE)

では、装置を管理するコレクションテーブルである装置管理情報のテーブル (テーブル名：equipTable) とアラート履歴を管理する時系列テーブルであるアラート履歴情報のテーブル (テーブル名：alertTable ) を作成します。

### 装置管理情報テーブル (テーブル名：equipTable) の作成

コレクションテーブルのタイプの装置管理情報テーブル (テーブル名：equipTable) は次の通りです。

|装置ID:id|装置タイプ:type|設置階数:floor|設置部屋番号:room|
|:-------|:--------------|:------------|:-----------------|


テーブル equipTable のカラムは、装置ID:id、装置タイプ:type、設置階:floor、設置部屋番号:room とします。また、テーブルのロウの一貫性を保証するために、主キー (プライマリキー) を装置ID:idに設定します。なお、プライマリキーを設定したカラム 装置ID:id には、NULL値を許容しません。

SQL文は次の通りです。

>CREATE TABLE equipTable (
>  id INTEGER PRIMARY KEY,
>  type STRING,
>  floor INTEGER,
>  room INTEGER
>);

では、実際に SQLWorkbench/Jで、テーブル equipTable を作成します。

```
SQL> CREATE TABLE equipTable (
..>   id INTEGER PRIMARY KEY,
..>   type STRING,
..>   floor INTEGER,
..>   room INTEGER
..> );

Table equipTable created
```

テーブル equipTable が作成されました。

### アラート履歴情報テーブル (テーブル名：alertTable) の作成

時系列テーブルのタイプのアラート履歴情報テーブル (テーブル名：alertTable)は次の通りです。

|検知時刻:date|装置ID:id|アラートレベル:level|アラート詳細情報:detail|
|:-----------|:--------|:------------------|:--------------------|

テーブル alertTable のカラムは、検知時刻:date、装置ID:id、アラートレベル:alertLebel、詳細メッセージ:detail とします。時系列テーブルの場合は、プライマリキーは先頭カラムに設定しなければなりません。なお、GridDBでは、カラムを0番から数えるため、カラム番号0に設定することになります。プライマリーキーは、TIMESTAMP型で、指定は必須です。プライマリーキーに設定したカラムには、カラムの型に応じてあらかじめ既定された、デフォルトの索引が設定されるようです。

GridDBのアプリケーションを高速化するには、処理するデータをできるだけメモリに配置することが重要とのことです。 テーブルパーティショニングを用いると、テーブルのデータを分割して配置することで、CPUやメモリリソースを効率的に活用した処理が可能のようです。分割したデータは、データパーティションという複数の内部コンテナに格納します。データをどのデータパーティションに格納するかは、テーブル作成時に指定されたパーティショニングキーのカラムの値を基に決定されるようです。

テーブルパーティショニングの方法として、ハッシュパーティショニング、インターバルパーティショニング、インターバル-ハッシュパーティショニングの3種類があります。ここでは、指定された一定の値間隔（今回は分割幅値：1日）でデータを分割して、データパーティションに格納するインターバルパーティショニングテーブルとして作成します。ある一定の範囲の値を持つデータを同じデータパーティション上に格納するので、登録するデータが連続値の場合や、特定期間の範囲の検索を行う場合などに、より少ないメモリで処理できます。
また、期限解放を使用します。パーティション期限解放として、今回は保持期間：2日としています。

ロウの保持期限は、データパーティションのロウ格納期間の最終日時に保持期間を足した日時です。同じデータパーティションに格納されているロウは、全て同じ保持期限になります。

SQL文は次の通りです。

>CREATE TABLE alertTable (
>  date TIMESTAMP PRIMARY KEY,
>  id INTEGER,
>  level INTEGER,
>  detail STRING
>) USING TIMESERIES WITH (
>  expiration_type='PARTITION',
>  expiration_time=2,
>  expiration_time_unit='DAY'
>) PARTITION BY RANGE (date) EVERY (1, DAY);

では、実際に SQLWorkbench/Jで、テーブル alertTable を作成します。

```
SQL> CREATE TABLE alertTable (
..> date TIMESTAMP PRIMARY KEY,
..> id INTEGER,
..> level INTEGER,
..> detail STRING
..> ) USING TIMESERIES WITH (
..> expiration_type='PARTITION',
..> expiration_time=2,
..> expiration_time_unit='DAY'
..> ) PARTITION BY RANGE (date) EVERY (1, DAY);

Table alertTable created
```

これで、テーブル alertTable を作成されました。

## データ登録 (INSERT)

では、データ登録をします。

### 装置管理情報テーブル (テーブル名：equipTable) へのデータ登録

装置管理情報テーブルに次のデータを登録します。

|装置ID:id|装置タイプ:type|設置階数:floor|設置部屋番号:room|
|:--|:-------|:------|:----|
| 1 | CAMERA |     1 |    1|
| 2 | THERMO |     1 |    1|
| 3 | THERMO |     4 |    3|
| 4 | THERMO |     6 |    2|
| 5 | WATT   |     1 |    1|
| 6 | WATT   |     6 |    1|

そのためのSQL文は次の通りです。

> INSERT INTO equipTable VALUES(1, 'CAMERA', 1, 1);
> INSERT INTO equipTable VALUES(2, 'THERMO', 1, 1);
> INSERT INTO equipTable VALUES(3, 'THERMO', 4, 3);
> INSERT INTO equipTable VALUES(4, 'THERMO', 6, 2);
> INSERT INTO equipTable VALUES(5, 'WATT', 1, 1);
> INSERT INTO equipTable VALUES(6, 'WATT', 6, 1);

では、実際に SQLWorkbench/Jで、テーブル equipTable にデータ登録します。

```
SQL> INSERT INTO equipTable VALUES(1, 'CAMERA', 1, 1);

INSERT INTO equipTable successful
1 row affected

SQL> INSERT INTO equipTable VALUES(2, 'THERMO', 1, 1);

INSERT INTO equipTable successful
1 row affected

SQL> INSERT INTO equipTable VALUES(3, 'THERMO', 4, 3);

INSERT INTO equipTable successful
1 row affected

SQL> INSERT INTO equipTable VALUES(4, 'THERMO', 6, 2);

INSERT INTO equipTable successful
1 row affected

SQL> INSERT INTO equipTable VALUES(5, 'WATT', 1, 1);

INSERT INTO equipTable successful
1 row affected

SQL> INSERT INTO equipTable VALUES(6, 'WATT', 6, 1);

INSERT INTO equipTable successful
1 row affected
```

### アラート履歴情報テーブル  (テーブル名：alertTable) へのデータ登録

アラート履歴情報テーブルに現在の時間を起点に次のようなデータを登録してみます。

|検知時刻:date            |装置ID:id|アラートレベル:<br>level|アラート詳細情報:<br>detail|
|:----------------------|:-------|:-----------------|:-------------------|
|96時間前の年-月-日 時:分:秒|  2      |                 1 | 96 hours ago        |
|72時間前の年-月-日 時:分:秒|  1      |                 2 | 72 hours ago        |
|48時間前の年-月-日 時:分:秒|  6      |                 3 | 48 hours ago        |
|24時間前の年-月-日 時:分:秒|  5      |                 4 | 24 hours ago        |
|18時間前の年-月-日 時:分:秒|  4      |                 1 | 18 hours ago        |
|12時間前の年-月-日 時:分:秒|  3      |                 2 | 12 hours ago        |
|9時間前の年-月-日 時:分:秒|  2     |                 3 | 9 hours ago        |
|6時間前の年-月-日 時:分:秒|  2     |                 4 | 6 hours ago        |
|3時間前の年-月-日 時:分:秒|  2     |                 3 | 3 hours ago        |
|現在の年-月-日　　時:分:秒|  1      |                 4 |                     |


SQL文は次の通りです。

> SELECT NOW();
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -96), 2, 1, '96 hours ago' );
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -72), 1, 2, '72 hours ago' );
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -48), 6, 3, '48 hours ago' );
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -24), 5, 4, '24 hours ago');
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -18), 4, 1, '18 hours ago');
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -12), 3, 2, '12 hours ago');
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -9), 2, 3, '9 hours ago');
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -6), 2, 4, '6 hours ago');
> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -3), 2, 3, '3 hours ago');
> INSERT INTO alertTable VALUES(NOW(), 1, 4, '');

ここで、注意をしておきたいことは、NOW関数で、現在の日時（タイムスタンプ）を取得しておきます。
なぜならば、デーブル alertTable は期限解放（パーティション期限解放として、今回は保持期間：2日）を指定しているので、検索する日時で、データの検索結果（数）が異なるからです。(すなわち、以降の検索での結果では、本記事の例と実際が異なる場合があります）

では、実際に SQLWorkbench/Jで、デーブル alertTable にデータの登録をします。

```
SQL> SELECT NOW();

Col1
-------------------
2021-02-26 16:32:50

(1 Row)
SELECT executed successfully

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -96), 2, 1, '96 hours ago' );

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -72), 1, 2, '72 hours ago' );

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -48), 6, 3, '48 hours ago' );

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -24), 5, 4, '24 hours ago');

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -18), 4, 1, '18 hours ago');

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -12), 3, 2, '12 hours ago');

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -9), 2, 3, '9 hours ago');

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -6), 2, 4, '6 hours ago');

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(TIMESTAMP_ADD(HOUR, NOW(), -3), 2, 3, '3 hours ago');

INSERT INTO alertTable successful
1 row affected

SQL> INSERT INTO alertTable VALUES(NOW(), 1, 4, '');

INSERT INTO alertTable successful
1 row affected
```

## データ検索

### 全件データ検索

SELECT * FROM table_name;

ロウおよびカラムの全件データ検索のSQL文は次の通りです。

> SELECT * FROM equipTable;
> SELECT * FROM alertTable;

では、実際に SQLWorkbench/Jで、全件データ検索します。


```
SQL> SELECT * FROM equipTable;

id | type   | floor | room_no
---+--------+-------+--------
 1 | CAMERA |     1 |       1
 2 | THERMO |     1 |       1
 3 | THERMO |     4 |       3
 4 | THERMO |     6 |       2
 5 | WATT   |     1 |       1
 6 | WATT   |     6 |       1

(6 Rows)
SELECT executed successfully
```

```
SQL> SELECT * FROM alertTable;

date                | id | level | detail
--------------------+----+-------+-------------
2021-02-24 16:32:49 |  6 |     3 | 48 hours ago
2021-02-26 04:32:50 |  3 |     2 | 12 hours ago
2021-02-26 07:32:50 |  2 |     3 | 9 hours ago
2021-02-26 10:32:50 |  2 |     4 | 6 hours ago
2021-02-26 13:32:50 |  2 |     3 | 3 hours ago
2021-02-26 16:32:51 |  1 |     4 |
2021-02-25 16:32:50 |  5 |     4 | 24 hours ago
2021-02-25 22:32:50 |  4 |     1 | 18 hours ago

(8 Rows)
SELECT executed successfully
```

検索されるロウの順序はタイミングによって異なります。
ところで、10ロウを登録したはずなのに8ロウしか検索されません。理由は、テーブル alertTableに、作成時に設定した保持期間：２日間を超えたロウデータを、検索や削除などの操作対象から外して参照不可とした後、データベースから物理的に削除する期限解放機能が指定されているからです。 これにより、データベースサイズ肥大化によるパフォーマンス低下を防ぐことができます。

### 取得するデータをソート：ORDER BY句

ORDER BY column_name_1 [{ASC|DESC}] [, column_name_2 [{ASC|DESC}] ...]

ORDER BY句を用いて、テーブル alertTableの検知時刻:dateの値に応じて、昇順 (ASC)で ロウを並べ替えます。

具体的なSQL文は次の通りです。

> SELECT * FROM alertTable order by date ASC;

ASCは省略することができます。
では、実際に SQLWorkbench/Jで、並べ替えて見ます。

```
SQL> SELECT * FROM alertTable order by date ASC;

date                | id | level | detail
--------------------+----+-------+-------------
2021-02-24 16:32:49 |  6 |     3 | 48 hours ago
2021-02-25 16:32:50 |  5 |     4 | 24 hours ago
2021-02-25 22:32:50 |  4 |     1 | 18 hours ago
2021-02-26 04:32:50 |  3 |     2 | 12 hours ago
2021-02-26 07:32:50 |  2 |     3 | 9 hours ago
2021-02-26 10:32:50 |  2 |     4 | 6 hours ago
2021-02-26 13:32:50 |  2 |     3 | 3 hours ago
2021-02-26 16:32:51 |  1 |     4 |

(8 Rows)
SELECT executed successfully
```

### 取得するデータの条件を設定：WHERE句

WHERE search_conditions

WHERE句を用いて、データを登録した日（今回の例では、2021-02-26）に絞って検索してみます。SQL文は次の通りです。

> SELECT * FROM alertTable WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') ;

では、実際に SQLWorkbench/Jで、期間：2021-01-26に絞って検索してみます。

```
SQL> SELECT * FROM alertTable WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') ;

date                | id | level | detail
--------------------+----+-------+-------------
2021-02-26 04:32:50 |  3 |     2 | 12 hours ago
2021-02-26 07:32:50 |  2 |     3 | 9 hours ago
2021-02-26 10:32:50 |  2 |     4 | 6 hours ago
2021-02-26 13:32:50 |  2 |     3 | 3 hours ago
2021-02-26 16:32:51 |  1 |     4 |

(5 Rows)
SELECT executed successfully
```

### データの最大値を求める：MAX関数

テーブル alertTableのアラート値:levelの最大値を求める場合は、 MAX関数を用いて、次のようにSQLを書きます。

> SELECT MAX(level) AS max FROM alertTable WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') ;

なお、このサンプルは分かりやすいようにAS句を使用して列に別名をつけています。
では、実際に SQLWorkbench/Jで、テーブル alertTableのアラート値:levelの最大値を求めてみます。

```
SQL> SELECT MAX(level) AS max FROM alertTable WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') ;

max
---
  4

(1 Row)
SELECT executed successfully
```

### データをグループ化する：GROUP BY句

GROUP BY column_name_1 [, column_name_2 ...]

GROUP BY句を用いてグループ毎に集計してみます。
SEGROUP BY句で装置ID:id 毎にアラートレベル:level の最大を求めて表示するSQL文は次のようになります。

> SELECT id, MAX(level) AS max FROM alertTable GROUP BY id;

```
SQL> SELECT id, MAX(level) AS max FROM alertTable GROUP BY id;

id | max
---+----
 1 |   4
 2 |   4
 3 |   2
 4 |   1
 5 |   4
 6 |   3

(6 Rows)
SELECT executed successfully
```

where句でデータを登録した日（今回の例では、2021-02-08）で範囲を絞り、GROUP BY句で装置ID:id 毎にアラートレベル:level の最大を求めて表示するSQL文は次のようになります。

> SELECT id, MAX(level) AS max FROM alertTable
> WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') GROUP BY id;

```
SQL> SELECT id, MAX(level) AS max FROM alertTable
..> WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') GROUP BY id;

id | max
---+----
 1 |   4
 2 |   4
 3 |   2

(3 Rows)
SELECT executed successfully
```

### 異なるテーブル同士を結合：JOIN句

table_name1 [INNER] JOIN table_name2 ON table_name1.row_name = table_name2.row_name

テーブル alertTableをGROUP BY句で装置ID:id 毎にアラートレベル:levelの最大値を集計した結果を テーブル equipTable と 装置ID:idをキーにしてJOIN句を用いて結合し、装置ID:id、装置タイプ:type、設置階:floor、設置部屋番号:room、それとアラートレベル:levelの最大値を表示します。

>SELECT equipTable.id, type, floor, room, max FROM equipTable JOIN
>  (SELECT id, MAX(level) AS max FROM alertTable GROUP BY id) t ON equipTable.id = t.id;

```
SQL> SELECT equipTable.id, type, floor, room, max FROM equipTable JOIN
..> (SELECT id, MAX(level) AS max FROM alertTable GROUP BY id) t ON equipTable.id = t.id;

id | type   | floor | room | max
---+--------+-------+------+----
 1 | CAMERA |     1 |    1 |   4
 2 | THERMO |     1 |    1 |   4
 3 | THERMO |     4 |    3 |   2
 4 | THERMO |     6 |    2 |   1
 5 | WATT   |     1 |    1 |   4
 6 | WATT   |     6 |    1 |   3

(6 Rows)
SELECT executed successfully
```

テーブル alertTableをwhere句で範囲付けして、さらにGROUP BY句で装置ID:id 毎にアラートレベル:levelの最大値を集計した結果を テーブル equipTable と 装置ID:idをキーにしてJOIN句を用いて結合し、装置ID:id、装置タイプ:type、設置階:floor、設置部屋番号:room、それとアラートレベル:levelの最大値を表示します。

> SELECT equipTable.id, type, floor, room, max FROM equipTable JOIN
>  (SELECT id, MAX(Level) AS max FROM alertTable
>  WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') GROUP BY id) t
>  ON equipTable.id = t.id;

```
SQL> SELECT equipTable.id, type, floor, room, max FROM equipTable JOIN
..> (SELECT id, MAX(Level) AS max FROM alertTable
..> WHERE date >= TIMESTAMP('2021-02-26T00:00:00Z') AND date < TIMESTAMP('2021-02-27T00:00:00Z') GROUP BY id) t
..> ON equipTable.id = t.id;

id | type   | floor | room | max
---+--------+-------+------+----
 1 | CAMERA |     1 |    1 |   4
 2 | THERMO |     1 |    1 |   4
 3 | THERMO |     4 |    3 |   2

(3 Rows)
SELECT executed successfully
```

### 任意の順でソートする：ORDER BY句 CASE式

テーブル equipTable で３階以上で、CAMERA(1)→THERMO(2)→WATT(3)→AMP(4)の順にソートされた結果が返します。

> SELECT * FROM equipTable WHERE floor >= 3 ORDER BY CASE type
>                        WHEN 'CAMERA' THEN 1
>                        WHEN 'THERMO' THEN 2
>                        WHEN  'WATT' THEN 3
>                        WHEN  'AMP' THEN 4
>                        ELSE 5
>                        END;

CAMERA(1)→THERMO(2)→WATT(3)→AMP(4)の順にソートされた結果が返ってきます。

```
SQL> SELECT * FROM equipTable WHERE floor >= 3 ORDER BY CASE type
..>                         WHEN 'CAMERA' THEN 1
..>                         WHEN 'THERMO' THEN 2
..>                         WHEN  'WATT' THEN 3
..>                         WHEN  'AMP' THEN 4
..>                         ELSE 5
..>                         END;

id | type   | floor | room
---+--------+-------+-----
 3 | THERMO |     4 |    3
 4 | THERMO |     6 |    2
 6 | WATT   |     6 |    1

(3 Rows)
SELECT executed successfully
```

## テーブルの削除

テーブル equipTable、alertTableを削除します。

> DROP TABLE equipTable;
> DROP TABLE alertTable;

```
SQL> DROP TABLE equipTable;

Table equipTable dropped

SQL> DROP TABLE alertTable;

Table alertTable dropped
```


# さいごに

今回、Docker DesktopのCentOSコンテナで、JDBCドライバ経由で動作する SQLWorkbench/Jをインストールして、JDBC経由でのGridDBへと方法についてステップバイステップで紹介し、また、具体的なGridbDBのデータモデルの例を用いて、SQLWorkbench/JによるGridDBデータベースのSQL操作についても記述したつもりです。
記述について誤りがあったり、気になることがあれば、編集リクエストやコメントでフィードバックしていただけると助かります。