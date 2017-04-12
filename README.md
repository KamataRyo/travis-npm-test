# Travis CIを使ってnpmを継続的に公開・メンテナンスするよ

これは以下のポストのクローンであり、同時にポスト内で例示されているリポジトリになります。

http://qiita.com/KamataRyo/items/6e795c6734f9a775f5a6

[![Build Status](https://travis-ci.org/KamataRyo/travis-npm-test.svg?branch=master)](https://travis-ci.org/KamataRyo/travis-npm-test)

npmで公開予定のライブラリを想定し、CIサービスを中心にしてリリース管理を行う方法について書きます。

## ローカルマシンに必要なライブラリをインストールする

[Git](https://git-scm.com/)、[Node.js](https://nodejs.org/)、[Travis CI Client](https://github.com/travis-ci/travis.rb)をインストールします。お好きな方法でお使いの端末にインストールしてください。以下は[Homebrew](https://brew.sh/)がインストールされているMacのターミナル上での操作の例です。

```shell
$ brew install git
$ brew install node
$ gem install travis
```

## 各サービスのサインアップ・ログイン

それぞれのサービスにサインアップしておきます。

- [GitHubへのサインアップ](https://github.com/signup/)
- [npmjs.comへのサインアップ](https://www.npmjs.com/signup)
- [Travis CIへのサインアップ](https://travis-ci.org/) (トップページからGitHubアカウントでサインインできます)

## Travis CI Clientでのログイン

GitHubのアカウントを使い、Travis CI Client上でログインを行います。以下のコマンドを使うと、GitHubユーザー名、パスワード、2要素認証のコード（設定している方のみ）をインタラクティブに入力してログインすることができます。画像にある標準出力のメッセージによればGitHubのアクセストークンを使って認証する方法もあるようです。

```shell
$ travis login
```

![スクリーンショット 2017-04-11 17.46.21.png](https://qiita-image-store.s3.amazonaws.com/0/125062/585955c8-207c-7bff-7cad-9a96701ccd09.png "スクリーンショット 2017-04-11 17.46.21.png")

## npmクライアントでのログイン

npmのクライアントはNode.jsと一緒にインストールされているはずです。npmjs.comのアカウントを使い、クライアント上でログインを行います。`npm login`コマンドを使うと、ユーザー名、パスワード、公開Eメールアドレスをインタラクティブに入力してログインすることができます。

```shell
$ npm login
```

![スクリーンショット 2017-04-11 18.11.22.png](https://qiita-image-store.s3.amazonaws.com/0/125062/f8fcd0a8-a4bb-c8e5-2bcd-c39e0a8eb8f0.png "スクリーンショット 2017-04-11 18.11.22.png")

## プロジェクトのセットアップ

対象となるプロジェクトをセットアップします。プロジェクトはGitの管理下に置き、またnpmプロジェクトとしての初期化（`package.json`の作成）を行っておいてください。

```shell
$ cd path-to-the-project
$ git init
$ npm init -y
```

また、このプロジェクトはGitHub上で公開プロジェクトとしてホストできるようにしておく必要があります。今回は、[kamataryo/travis-npm-test](https://github.com/KamataRyo/travis-npm-test)というリポジトリで例を用意してあります。

```shell
$ git remote add origin git@github.com:kamataryo/travis-npm-test.git
```

## Travis CIのセットアップ

`travis init`コマンドを使うと `.travis.yml`が作成され、また、このリポジトリの存在をTravis CIに伝えることができます。この際は、正しいリポジトリかどうか、プロジェクトのプログラミング言語は何かをインタラクティブに聞かれますので、正しい答えを入力してください。`package.json`があるにもかかわらず、よくrubyに間違われます。

```shell
$ travis init
```

![スクリーンショット 2017-04-11 17.56.55.png](https://qiita-image-store.s3.amazonaws.com/0/125062/0e79dce4-4dc0-9dba-d32b-597f76e5960a.png "スクリーンショット 2017-04-11 17.56.55.png")

ここまでの設定を行うと、GitHubへのプッシュをトリガーに、Travis CIでのビルドが発火する状態になっているはずです。

## npmへのデプロイのセットアップ

`.travis.yml`を編集してデプロイ設定を行います。ファイルに以下の値を追加してください。

```.travis.yml
deploy:
  provider: npm
  email: "test@example.com"
  on:
    tags: true
```

`on.tags = true`は、タグのコミットの場合のみデプロイが発火する設定です。この他、特定のブランチのプッシュにフックすることもできるようです。詳細は以下のドキュメントなどを参照してください。

https://docs.travis-ci.com/user/deployment/npm/


## npmの認証トークンを確認する

`npm login`コマンドが成功していると、`~/.npmrc`に認証トークンが記載されるはずです。これを確認してください。

```shell
$ cat ~/.npmrc

//registry.npmjs.org/:_authToken=[認証トークン]
```

`travis encrypt`コマンドを使い、得られた認証トークンを暗号化します。addオプションでカレントディレクトリにある`.travis.yml`に値を追記してくれます。ただ、この時にyamlファイルが勝手に整形されてしまうので、これが嫌な場合はaddオプションを外して標準出力をコピペするのがいいかもしれません。

```shell
$ travis encrypt [認証トークン] --add deploy.api_key
```

以上までの設定を行うと、`.travis.yml`は以下のような内容になっているはずです。

```.travis.yml
language: node_js
node_js:
- '7'
deploy:
  provider: npm
  email: 'test@example.com'
  on:
    tags: true
  api_key:
    secure: [暗号化された認証トークン]
```

## npmで公開する

以上でTravis CIの設定が完了したので、GitHubにプッシュしてみます。

```shell
$ git add .
$ git commit -m "Init repo"
$ git push origin master
```

タグを付け、プッシュすることでリリースします。

```shell
$ npm version patch
$ git push origin v1.0.1
```

`npm version`コマンドは色々な仕事をしてくれていているようで、

- package.jsonの編集（バージョン番号のインクリメント）
- 新しいバージョンタグの作成
- masterブランチへのamendコミット（?）

などをしてくれているみたいです。あとはタグをプッシュするだけでOKになります。`patch`引数を与えるとバージョンの `v0.0.x`の部分、それとは別に`minor`と`major`引数を与えるとそれぞれ `v0.x.0`、`vx.0.0`の部分がインクリメントされます。

Travis CIでのテストが成功すると、ビルドはdeployセクションに移行して`npm publish`コマンドを自動で実行してくれます。以下のリンクは成功したビルドの履歴になります。

https://travis-ci.org/KamataRyo/travis-npm-test/builds/220894443

![スクリーンショット 2017-04-11 18.51.30.png](https://qiita-image-store.s3.amazonaws.com/0/125062/b68e869f-78c2-993d-0db0-73b479eb2786.png "スクリーンショット 2017-04-11 18.51.30.png")


## オプション

`npm version`コマンドのポストスクリプトを以下のように定義しておくと、`npm version`コマンドの後でタグをプッシュしなくていいので便利かもしれません。最新のタグを自動でプッシュします。

```package.json
{
  ...
  "scripts": {
     ...
  	 "postversion": "git push origin $(git tag -l | tail -n1)",
  	 ...
  }
}
```

## まとめ
今回例示したリポジトリは以下のものになります。
https://github.com/KamataRyo/travis-npm-test

また、このリポジトリのTravis CIでのビルドログは以下から閲覧できます。
https://travis-ci.org/KamataRyo/travis-npm-test/builds


CIツール・サービスを中心にしたワークフローを採ることで、タグを付けてプッシュするだけでライブラリの公開・更新を行うことができます。テストに失敗するとデプロイが発火しないため、ヒューマンエラーによりテストが落ちる状態でプッシュしてしまったとしても、リリースを防ぐことができます。
