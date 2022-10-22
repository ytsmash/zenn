---
title: "docker-compose.yamlはマージできる"
emoji: "👋"
type: "tech"
topics: ["docker", "docker-coompose"]
published: true
---
# docker composeのデフォルトの挙動
デフォルトでは、docker-compose.yamlをベースとして読み込み、docker-compose.override.yamlがあればそれで上書きします。
```bash
# docker-compose.yamlをベースに起動
# docker-compose.override.yamlがあればそれで上書きしたもので起動
docker compose up
```
> デフォルトでは、 Compose は２つのファイルを読み込みます。１つは docker-compose.yml で、もう１つはオプションで docker-compose.override.yml ファイルです。習慣として、 docker-compose.yml に基本となる設定を入れます。 上書きoverride ファイルは、その名前が意味する通り、既存のサービスや新しいサービス全体の設定を上書きする設定を含められます。

:::details デフォルトで読み込むdocker-compose.yamlの変更
以下の記事が詳しいです。
https://qiita.com/hoto17296/items/a8a85d5244f46c119278
:::

# docker composeの-fオプション
それに対して、docker composeコマンドを実行するときに-fオプションで設定ファイルを指定できます。
```bash
docker compose -f docker-compose.yaml up      # 本番環境用の設定ファイルで起動
docker compose -f docker-compose.dev.yaml up  # dev環境用の設定ファイルで起動
docker compose -f docker-compose.test.yaml up # test環境用の設定ファイルで起動
```
のようにすることで環境ごとに設定ファイルを分けることができます。

:::details docker-composeコマンドとdocker composeコマンドについて
以下の記事が参考になります。基本的にはあまり変わらないみたいです。
https://qiita.com/kotobuki5991/items/e2af78c11b4571f904e5
:::

# -fオプションを複数指定するとマージできる
ここで、-fオプションを複数指定するとそれらを全てマージしたものを設定ファイルとしてコンテナ群を起動することができます。
```bash
# 本番環境とdev環境の設定ファイルをマージして起動
docker compose -f docker-compose.yaml -f docker-compose.dev.yaml up
```

# マージの仕様について
-fオプションに複数指定する場合は、以下のような点がポイントです。
- **1つ目の-fオプションに指定する設定ファイルはdocker-compose.yamlファイルとして正しいものでなければいけない**
- 逆に**2つ目以降は正しくなくていい**(docker-compose.yamlの破片でいい)
- あとに指定した設定ファイルが優先される

> 複数の上書きファイルを使う場合や、異なる名前の上書きファイルを使う場合は、 -f オプションを使ってリストやファイルを指定できます。コマンドライン上で指定した順番で、Compose はそれらのファイルを統合します。 -f を使う詳しい情報は docker-compose コマンドリファレンス をご覧ください。
>
> 複数の設定ファイルを使う場合、全ファイルのパスが基本となる Compose ファイル（ -f で１番目に指定した Comopse ファイル）からの相対パスになるので、注意が必要です。注意が要るのは、上書きするファイルは正しい Comopse ファイルである必要がないためです。サービスの一部を追跡するにあたり、相対パスは複雑で混乱するため、パスを分かりやすくするためには、全てのパスを基本ファイルからの相対パスとして指定すべきです。

https://docs.docker.jp/compose/extends.html


# 実際の使用例
```yaml:docker-compose.yaml
# ベースとなる内容を書く
version: '3'
services:
  web:
    image: node:16
```
```yaml:docker-compose.dev.yaml
# ベースからの変更点だけ書く
services:
  web:
    tty: true
```
みたいにすると以下のような挙動になるはずです。
```bash
# 起動後すぐ終了する
docker compose -f docker-compose.yaml up

# 起動したまま
docker compose -f docker-compose.yaml -f docker-compose.dev.yaml up

# 単体では実行できない
docker compose -f docker-compose.dev.yaml up
```

# 終わりに
最近知った機能だったのでまとめてみました。

上書きできるところまでは知っていたのですが、2つ目以降のdocker-compose.yamlファイルは一部のみの記述でいいことは結構イケてると思いました。これを使うと、ベースを1つ用意して、他環境は差分のみ書けばいいのでファイルを見た時にわかりやすいです。

# 参考
https://docs.docker.jp/compose/extends.html
https://qiita.com/hoto17296/items/a8a85d5244f46c119278
https://qiita.com/kotobuki5991/items/e2af78c11b4571f904e5