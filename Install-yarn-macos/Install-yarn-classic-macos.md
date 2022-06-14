<!--
title:   macOS に Yarn (Yarn Classic) をインストールする方法
tags:    YARN,macOS,インストール
id:      d3a7271c3c251a9bd32d
private: true
-->
macOSにパッケージ管理システム Yarn をインストールして確認するまでの手順を備忘録としてまとめました。参考にして頂ければ幸いです。

- 前提条件（動作環境）
- yarnのインストール
- インストール完了の確認
なお、modenバージョン とclassicバージョンがありますが、今回はclassicバージョン インストールです。

# 前提条件（動作環境）

yarn をインストールするための前提条件は次の通りです。

- Node.js がインストールされ、動作すること
- npm がインストールされ、動作すること

これらの前提がまだ満たせていない場合はまずは次の記事を参考に前提部分の環境構築を完了させてください。

https://qiita.com/gahoh/items/d2004f711748bf493f6a

なお、検証した動作環境は次の通りです。

- OS: macOS Monterey バージョン 12.3.1
- Node.js: v16.15.0
- npm: 8.5.5

# yarnのインストール

yarn のインストール方法は非常に簡単で npm を使ってインストールします。

brew を使ってインストールをすることもできますが、依存関係的に Node をインストールして npm 経由の方が開発環境としては間違いが発生しないので、brew を使うパターンには触れません。

yarn のインストールには以下のコマンドを実行します。

```
npm install -g yarn 
```

```
$ npm install -g yarn 
npm ERR! code EACCES
npm ERR! syscall mkdir
npm ERR! path /usr/local/lib/node_modules/yarn
npm ERR! errno -13
npm ERR! Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules/yarn'
npm ERR!  [Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules/yarn'] {
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'mkdir',
npm ERR!   path: '/usr/local/lib/node_modules/yarn'
npm ERR! }
npm ERR! 
npm ERR! The operation was rejected by your operating system.
npm ERR! It is likely you do not have the permissions to access this file as the current user
npm ERR! 
npm ERR! If you believe this might be a permissions issue, please double-check the
npm ERR! permissions of the file and its containing directories, or try running
npm ERR! the command again as root/Administrator.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/gahoh/.npm/_logs/2022-05-23T10_58_25_292Z-debug-0.log
$ 
```

```
$ sudo npm install -g yarn 
Password:
Sorry, try again.
Password:

added 1 package, and audited 2 packages in 1s

found 0 vulnerabilities
npm notice 
npm notice New minor version of npm available! 8.5.5 -> 8.10.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v8.10.0
npm notice Run npm install -g npm@8.10.0 to update!
npm notice 
$ 
```

# インストール完了の確認

処理が終了したら正常にインストールが完了しているかを確認するために下記のコマンドを実行します。

```
$ yarn -v
1.22.18
$
```

これで、macOS への Yarn (Yarn Classic) のインストールは完了です。
