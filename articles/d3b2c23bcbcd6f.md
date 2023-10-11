---
title: "ThoughtWorks Technology Radar とは何か"
emoji: "📡"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [technologyradar]
published: true
publication_name: "beingish"
---

## TL;DR

一言でいうと、 ToughtWorks 社によるオレオレ技術ランキング。次の URL で公表されている。

https://www.thoughtworks.com/radar

## ThoughtWorks 社とはどういう企業なのか？

日本で真っ先にヒットするのは次の書籍だ。かなり古いが、ソフトウェア開発に関して、今でも通用する様々な話題が語られている。

https://www.oreilly.co.jp/books/9784873113890/

[Martin Fowler](https://martinfowler.com/) 氏が所属する、というとその凄さがわかる方もいるだろう。ソフトウェア開発におけるおおよそすべての話題、例えばプロジェクト運営や設計論、コーディングテクニックなどのアイディアや実践の第一人者だ。

[主力事業は技術に関するコンサルティング](https://www.thoughtworks.com/about-us/our-purpose)だが、[オープンソースや周辺コミュニティの形成に力を入れたり](https://www.thoughtworks.com/insights/topic/open-source)、[書籍執筆のバックアップをしていたりする](https://www.thoughtworks.com/insights/books)。

逆に Technology Radar を出している企業、という認識の方もいるだろう。最近の日本における認知だとそれが一番自然だ。

ではその Technology Radar について詳細に見ていこう。

## ThoughWorks Technology Radar とは何か？

https://www.thoughtworks.com/radar/faq-and-more

年に 2 回、大体 4 月頃と 10 月頃に発表されている。

ThoughtWorks 社が実務で使って得た知見をまとめ、各技術の現状をまとめてくれている。もちろん、すべての技術を網羅できているわけではなく、主に今ホットな技術が記載されていることに注意。こういった特性から、過去には言及があったものの最新では記載されていない技術も存在する。

### 見方

[FAQ にすべて記載されている](https://www.thoughtworks.com/radar/faq-and-more)が、割と量があるのでざっくりとまとめる。

Radar の名の通り、実際のレーダーを模して表現されている。次のような画面を見たことがある方もいるだろう。

![レーダー](/images/radar.png =320x)

レーダー上に 4 つの光点がある。これを blip と言う。 Technology Radar でも同様に blip がたくさんあり、それぞれが各技術に対応している。

レーダーの中心からいくつか同心円がひかれている。これらは blip との距離を測るために使われる。 Technology Radar ではその距離に応じて 4 段階の状態を表している。

| 段階   | 説明                                                           |
| ------ | -------------------------------------------------------------- |
| Adopt  | いますぐ採用を検討すべき                                       |
| Trial  | いますぐ検証すべき                                             |
| Assess | まだ触らなくて良い                                             |
| Hold   | 近寄ってはならない。採用している場合は今すぐ代替策を講じるべき |

中心から最も近い領域が Adopt 、最も遠いものが Hold だ。同じ段階でも、より中心に近ければ、より有用だったり素早く検証すべき技術となる。

一般的なレーダーは 360 度の空間を投影しているが、 Technology Radar では 4 等分し、それぞれ別領域の表現に割り当てている。

- Programming Languages and Frameworks
- Tools
- Platforms
- Techniques

### 見る順番

目的によって異なるが、エンジニアリングに関わる方であれば最低限 Adopt と Hold だけ抑えておけば十分だろう。

CTO など中長期的な技術選択を迫られている方は Trial にも目を通し、余裕があれば Assess でひっかかるものを見てみると良い。

### 注意点

あくまで ThoughtWorks 社の立ち位置からの評価だということに注意する必要がある。 Adopt の blip だからといって鵜呑みにして採用してはならず、自らの環境において有用かどうかは必ず検討する必要がある。

## 自分で作りたい場合

たとえば自社の技術スタックを整理したい、採用すべき技術を関係者間で議論したいなどのニーズはどこでもあるだろう。 ThoughtWorks 社と同様の Technology Radar を自社でも作れればそういった目的に寄与すると考えるのは自然なことだ。

ThoughtWorks 社はそういった場合に使えるツールを提供してくれている。

https://www.thoughtworks.com/radar/byor

Google Spread で所定のカラムとデータを作成すると、それを元に生成してくれる。試しに作ってみた。

https://radar.thoughtworks.com/?documentId=https%3A%2F%2Fdocs.google.com%2Fspreadsheets%2Fd%2F1V4cD0enRtnKwuQs_VNXT1ymONuVB5vaCym3nI4CUoKY

サンプルとして提供されている Google Spread は使いづらいため、テンプレートも作成してみた。コピーして使うと良い。

https://docs.google.com/spreadsheets/d/1V4cD0enRtnKwuQs_VNXT1ymONuVB5vaCym3nI4CUoKY

このツールはオープンソースとして提供されており、改造したり自前でホストすることが可能。

https://github.com/thoughtworks/build-your-own-radar
