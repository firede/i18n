# ビルド手順 (macOS)

macOS 版 Electron のビルドについては、以下のガイドラインに従ってください。

## 必要な環境

- macOS 10.11.6 以上
- [Xcode](https://developer.apple.com/technologies/tools/) 8.2.1 以上
- [Node.js](https://nodejs.org) (外部)
- TLS 1.2 に対応した Python 2.7

## Python

システムと Python のバージョンが少なくともTLS 1.2をサポートしていることも確認してください。 これはあなたの macOS と Python のバージョンの両方に依存します。 クイックテストを実行するには以下を実行します。

```sh
$ npm run check-tls
```

スクリプトが古い構成のセキュリティプロトコルを使用していると応答した場合、macOS を High Sierra に更新するか、新しいバージョンの Python 2.7.x をインストールすることができます。Python をアップグレードするために、以下では [Homebrew](https://brew.sh/) を使用します。

```sh
$ brew install python@2 && brew link python@2 --force
```

Homebrew で提供される Python を使用している場合、以下の Python モジュールのインストールも必要です。

- [pyobjc](https://pythonhosted.org/pyobjc/install.html)

## macOS SDK

Electron を開発していて独自の Electron ビルドを再配布する予定がない場合は、このセクションを飛ばして構いません。

特定の機能 (ピンチズームなど) を正しく機能させるには、macOS 10.10 SDK をターゲットにする必要があります。

公式の Electron ビルドは、デフォルトで 10.10 SDK を含まない [Xcode 8.2.1](http://adcdownload.apple.com/Developer_Tools/Xcode_8.2.1/Xcode_8.2.1.xip) でビルドされています。 入手するには、まず [Xcode 6.4](http://developer.apple.com/devcenter/download.action?path=/Developer_Tools/Xcode_6.4/Xcode_6.4.dmg) ディスクイメージをダウンロードしてマウントしてください。

次に、Xcode 6.4 ディスクイメージが `/Volumes/Xcode` にマウントされ、Xcode 8.2.1 のインストールが `/Applications/Xcode.app` にあるとすると、以下を実行します。

```sh
cp -r /Volumes/Xcode/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/
```

更に 10.10 SDK に対してビルドするには、以下の手順で Xcode を有効にする必要があります。

- `/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Info.plist` を開く
- `MinimumSDKVersion` を `10.10` にセットする
- ファイルを保存する

## コードを取得する

```sh
$ git clone https://github.com/electron/electron
```

## ブートストラップ

ブートストラップスクリプトはビルドに必要な全ての依存関係をダウンロードし、ビルドプロジェクトファイルを作成します。 なお、Electron のビルドには [ninja](https://ninja-build.org/) を用いているため、Xcode プロジェクトが生成されないことに注意してください。

To bootstrap for a static, non-developer build, run:

```sh
$ cd electron
$ npm run bootstrap
```

Or to bootstrap for a development session that builds faster by not statically linking:

```sh
$ cd electron
$ npm run bootstrap:dev
```

言語サーバを基にした [JSON コンパイルデータベース](http://clang.llvm.org/docs/JSONCompilationDatabase.html) をサポートしているエディタを使用している場合、以下で生成できます。

```sh
$ ./script/build.py --compdb
```

## ビルド

To build both `Release` and `Debug` targets:

```sh
$ npm run build
```

You can also build either the `Debug` or `Release` target on its own:

```sh
$ npm run build:dev
```

```sh
$ npm run build:release
```

ビルド完了後、`out/D` 下に `Electron.app` が見られます。

## 32 ビット OS のサポート

Electron は macOS 上では 64bit ターゲットしかビルドできず、今後 32bit macOS をサポートする予定はありません。

## クリーン

以下でビルドファイルをクリーンします。

```sh
$ npm run clean
```

以下で `out` と `dist` ディレクトリだけをクリーンします。

```sh
$ npm run clean-build
```

**注釈:** どちらのクリーンコマンドもビルド前に `ブートストラップ` を再度実行する必要があります。

## テスト

[ビルドシステム概要: テスト](build-system-overview.md#tests) を参照してください。