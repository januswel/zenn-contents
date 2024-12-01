---
title: "GPG 鍵を別マシンに移植する"
emoji: "✉️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [git, github, gpg, pgp]
published: false
---

次の補足。

https://zenn.dev/beingish/articles/5beb088b807399

GPG 鍵は生成されたマシン内に暗号化の上、保存されている。そのため、別マシンに移すにはいくつかステップが必要となる。

gpg コマンドはちょっとマニュアルが見づらいのでここにまとめておく。

# 1. 秘密鍵をエクスポートする

[前回の一番最後](https://zenn.dev/beingish/articles/5beb088b807399#%E9%8D%B5%E3%82%92%E4%BF%9D%E7%AE%A1%E3%81%99%E3%82%8B)に記載しているのでそちらを参照。

インポートしたいマシンにコピーする。

# 2. 新マシンで GPG の設定をする

[macOS のやり方だが、前回の記事に書いている](https://zenn.dev/beingish/articles/5beb088b807399#%E5%BF%85%E8%A6%81%E3%81%AA%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B)。

# 3. インポートする

次のコマンドで秘密鍵をインポートする。 `<key path>` は実際の秘密鍵ファイルへのパスを指定する。

```sh
gpg --import <key path>
```

作成時に指定したパスワードが求められるので入力。

うまくいくと、次のコマンドでインポートした鍵が見えるようになっている。

```sh
gpg --list-secret-keys --keyid-format long
```

# 4. インポートした鍵の信頼度を 5 にする

インポート直後は鍵の信頼度が不明: unknown となっている。が、自分で作ったものなので信頼して良い。このためにいくつか操作が必要となる。

https://unix.stackexchange.com/questions/407062/gpg-list-keys-command-outputs-uid-unknown-after-importing-private-key-onto

:::details 信頼度とは
信頼度は 5 段階あり、信頼のチェインを形成する際の評価に用いられるらしい。

https://www.gnupg.org/gph/en/manual/x334.html

多分 Git commit 時に署名だけであれば信頼度設定は関係ないはず。
:::

具体的なコマンドは次。 `<mail address>` は信頼度を変更したい鍵に設定したメールアドレス。

```sh
gpg --edit-key <mail address>
```

対話形式で設定する。 `trust` と打ち込んで、 `5` を選択、 `y` で決定する。最後に `save` で保存。

:::details 実際のログ
手元のログを貼っておく。

```sh
gpg> trust
sec   ed25519/24A01C981CDE28D4
     作成: 2024-03-25  有効期限: 2027-03-25  利用法: SC
     信用: 充分        有効性: 不明の
ssb   cv25519/E836A4E4B71FE7F4 2024-03-25  利用法: E
[  不明  ] (1). NANASHINO Gonbee <nanashino.gonbee@example.com>

他のユーザの鍵を正しく検証するために、このユーザの信用度を決めてください
(パスポートを見せてもらったり、他から得たフィンガープリントを検査したり、などなど)

  1 = 知らない、または何とも言えない
  2 = 信用し ない
  3 = まぁまぁ信用する
  4 = 充分に信用する
  5 = 究極的に信用する
  m = メーン・メニューに戻る

あなたの決定は? 5
本当にこの鍵を究極的に信用しますか? (y/N) y

sec   ed25519/24A01C981CDE28D4
     作成: 2024-03-25  有効期限: 2027-03-25  利用法: SC
     信用: 充分        有効性: 不明の
ssb   cv25519/E836A4E4B71FE7F4 2024-03-25  利用法: E
[  不明  ] (1). NANASHINO Gonbee <nanashino.gonbee@example.com>
プログラムを再起動するまで、表示された鍵の有効性は正しくないかもしれない、
ということを念頭においてください。

gpg> save
鍵は無変更なので更新は不要です。
```

:::

次のコマンドで `uid` の右側に `究極` と表示されていることを確認。

```sh
gpg --list-secret-keys --keyid-format long
```

これでインポートは完了。
