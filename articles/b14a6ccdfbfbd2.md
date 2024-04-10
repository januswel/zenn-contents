---
title: "Go 言語ですべてのテストを実行する"
emoji: "🍡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["golang", "test"]
published: false
---

Go でテストをすべて実行したいときは次のコマンドを実行する。

```
go test ./...
```

[`...` はパッケージ指定の際のワイルドカードにあたるものらしい](https://pkg.go.dev/cmd/go#hdr-Package_lists_and_patterns)。

テストがたくさんあってもデフォルトで CPU 数を同時実行するようなのであんまり遅くならない。

https://zenn.dev/koron/articles/ea783d5f202ef9fb68d7
