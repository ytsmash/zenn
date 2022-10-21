---
title: "watchコマンドでコンテナの一生を見届ける"
emoji: "🎉"
type: "tech"
topics:
  - "docker"
  - "linux"
  - "cli"
published: true
published_at: "2022-10-16 15:28"
---

# はじめに
## watchコマンドについて
watchコマンドって知ってますか？
僕は恥ずかしながら最近知りました。

> **watchコマンド**
> コマンドを一定の時間ごとに実行して結果を端末にスクロールさせずに固定で表示してくれるコマンド。

つまり、リアルタイムで実行結果が反映されるようにするコマンドです。
今回はwatchコマンドを使ってコンテナを作ったり壊したりしてみます。

## watchコマンドのインストール
```bash
brew install watch # homebrewでインストール
watch -v           # 正常にインストールされていることの確認
```
https://formulae.brew.sh/formula/watch

## watchコマンドの使い方
```bash
watch 定期実行したいコマンド
# 例: watch lsとかwatch docker psみたいな感じに
```
デフォルトでは、**2秒ごと**に実行されますが、この間隔を**nオプション**によって指定することができます

```bash
watch ls 　　　　　　　　　　#　2秒ごとに定期実行
watch -n 5 ls #　5秒ごとに定期実行
```

他のオプションは以下のような感じ:
| オプション | 説明 |
| ---- | ---- |
| -c | 色表示に対応してくれる|
| -e | コマンドがエラー終了したら定期実行をやめる |
| -g | コマンドの出力に変化があったらやめる |
| -d | 変化した部分を強調して表示してくれる |
| -n | 実行間隔を指定する。最小は0.1(秒) |
| -t | ヘッダーを非表示にする　|

# watchコマンドでdockerを監視する
とりあえずwatchコマンドで`docker ps`コマンドをします。
```bash
watch docker ps # 実行中のdockerをリスト表示する
```
> 実行結果
> ```bash
> Every 2.0s: docker ps
>
> CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
> # 最初は当然起動中のコンテナはなにもない
> ```

当然何も表示されていません。

次に、ターミナルの別ウィンドウで`docker run`してみます。
vscodeだと`Ctrl + Shift + 5`でターミナルを分割できます。
```bash
docker run nginx # nginｘイメージをもとにdockerコンテナを起動
```

するとwatchコマンドを実行してたターミナルに変化があると思います。
> 実行結果
> ```bash
> Every 2.0s: docker ps
> 
> CONTAINER ID   IMAGE     COMMAND     CREATED     STATUS    PORTS     NAMES
> 0e7b22e2be39   nginx     ... # 以下省略
> ```

次に、`docker run`を実行してたターミナルを `Ctrl + C`で一回止めてコンテナを停止します。
コンテナが停止したことで、`docker ps`に表示されなくなりました。
> 実行結果
>```bash
> Every 2.0s: docker ps 
>
> CONTAINER ID   IMAGE     COMMAND     CREATED     STATUS    PORTS     NAMES
> # なにも無くなった
> ```

今度はwatchコマンドを実行していたターミナルも一回`Ctrl + C`で止めて、今度は停止中のコンテナも含めて見てみます。つまり`docker ps -a`コマンドをwatchで実行します。
```bash
watch docker ps -a # 起動中または停止中のコンテナをリスト表示
```
> 実行結果
>```bash
> Every 2.0s: docker ps　-a
>
> CONTAINER ID   IMAGE     COMMAND     CREATED     STATUS    PORTS     NAMES
> 0e7b22e2be39   nginx     ... # 以下省略
> # 停止中のコンテナが表示される
> ```

またまた、別のウィンドウを開いて、今度は停止しているコンテナを削除してみます。
```bash
docker rm 0e # 0eから始まるコンテナIDのコンテナを削除(コンテナIDは実行ごとに異なる)
```

すると`docker ps -a`を実行中のターミナルからコンテナが消えます
> 実行結果
> ```bash
> Every 2.0s: docker ps　-a
> 
> CONTAINER ID   IMAGE     COMMAND     CREATED     STATUS    PORTS     NAMES
> # なにもなくなった
> ```

# 最後に
watchコマンドは色々使い道ありそうです。
今回は試しませんでしたが、cとかdオプション指定したものをaliasで登録しておいたりするのもよさそうだと思いました。