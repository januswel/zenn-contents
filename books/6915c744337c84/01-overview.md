---
title: "第 1 章: React / React Nativeの概要とその背景"
---

## React とは

React は JavaScript を用いてビュー層を記述するための OSS ライブラリーです。Facebook が主体となって開発されています。

2020 年 3 月現在、JavaScript でビュー層を構築する際に世界で最も使われているパッケージです。すべてのパッケージの日ごとのダウンロード数がわかる[npm trends](http://www.npmtrends.com/angular-vs-react-vs-vue)では、その他のフレームワークの 3 倍以上のシェアを持っていることが確認できます。

React はどのようなものか、[公式ページ](https://reactjs.org/)の言葉を借りると次の特徴を持っています。

- Declarative（宣言的）
- Component-Based（コンポーネントベース）
- Learn Once, Write Anywhere（一度学べば、どのプラットフォームでも書ける）

これらの特徴をひとつひとつ見ていきましょう。

### 特徴 1. Declarative

Declarative とは「宣言的」という意味です。次のように、表示すべき情報を書き下していくことでビュー層を構築できます。なお例としてコードを出していますが、細かい点については後の章でくわしく説明するので、本章では雰囲気を感じてください。

```jsx
function Profile(props) {
  return (
    // ①
    <div style={{ backgroundColor: props.isRegular ? "red" : "white" }}>
      <span>{props.name}</span>
      <img src={props.photo} />
    </div>
    // ①ここまで
  );
}
```

① の HTML に似た部分は JSX と呼ばれ、ビュー層を書きやすくするために Facebook が JavaScript の構文を拡張したものです。

これに対し、既存のライブラリーの一部もしくはライブラリーなしの JavaScript では手続きとして、次のようにビュー層を記述する必要があります。

```javascript
function renderProfile(props) {
  var span = document.createElement("span");
  span.appendChild(document.createTextNode(props.name));

  var div = document.createElement("div");
  div.style.backgroundColor = props.isRegular ? "red" : "white";
  div.appendChild(span);

  var img = document.createElement("img");
  img.src = props.photo;
  div.appendChild(img);

  return div;
}
```

このコードでは最終的な DOM の構造が一目でわかりません。多重にネストしたり要素が複数並んだりといった複雑な構造を作ることは可能ですが、非常にメンテナンスしにくいことがわかります。

対して宣言的なコードでは一目見れば構造がわかるようになっており、要素の追加/変更/削除が直感的にできることが特徴です。

### 特徴 2. Component-Based

Component（コンポーネント）は「部品」という意味です。Component-Based（コンポーネントベース）は「部品を基礎とした」という意味合いになります。画面を構成する要素をコンポーネントとして分割/作成し、それらを組み合わせて画面を作成していくという手法です。

この手法は既存手法の課題を解決するために考案されました。 HTML と CSS に JavaScript で動作をつけていく従来の手法で画面を作成していく際、次の課題がありました。

- 要素の粒度が細かすぎる
- HTML はアプリの画面作成のための語彙ではない
- 状態管理が煩雑になる

#### 要素の粒度が細かすぎる

`div` や `span` 、 `a` などの HTML 要素は、アプリに必要な構造を書き表すための十分な表現力を持っています。しかし、すべての画面でこれらのみを組み合わせていくのは、骨が折れる作業です。さながら手頃な大きさの木の板だけで超高層ビルを建築するようなものです。

#### HTML はアプリの画面作成のための語彙ではない

また、 HTML は文書作成用の言語なのでアプリの画面を作成するための語彙を持っていない、という点もハードルを上げている要因です。次のように `div` 要素や `span` 要素に対して `class` 属性で意味を付与できますが、視認性に優れているとは言えません。

```html
<div class="profile">
  <span class="name">Takagi Kensuke</span>
  <img class="photo" src="https://example.com/photos/takagi-kensuke.png" />
</div>
```

#### 状態管理が煩雑になる

画面は、状態を持つことがあります。テキストボックスに入力されている文字列やラジオボタンの選択状態など、フォーム要素はわかりやすい例でしょう。ユーザーの状態によって表示項目が変わることもあります。ユーザーの行動履歴から嗜好が分析できている場合はそれに沿ったサジェストを、そうでない場合は汎用の広告を出すなどです。

こういったものが画面内に 1 つ 2 つ程度しか存在しない場合は問題ないでしょう。しかし、 10 個 20 個の状態を管理しなければならないとした場合、管理が煩雑になりメンテナンスしづらいものとなってしまいます。

#### コンポーネントによる解決

コンポーネントは、これら 3 つの課題を解決します。コンポーネントによって表示および動作を意味づけし、その意味付けした単位で画面を構築可能となります。たとえばユーザープロフィールという概念のコンポーネントは、冒頭に紹介した次のコードとなるでしょう。

```jsx
function Profile(props) {
  return (
    <div style={{ backgroundColor: props.isRegular ? "red" : "white" }}>
      <span>{props.name}</span>
      <img src={props.photo} />
    </div>
  );
}
```

この定義があることで、任意の場所に次のように書くだけでユーザープロフィールが表示可能となります。

```jsx
<Profile
  isRegular={true}
  name="Takagi Kensuke"
  photo="https://example.com/photos/takagi-kensuke.png"
/>
```

前述した課題に対して、次のように解決されています。

- 扱いやすい粒度となっている
- アプリにとって意味のある名前を用いることができる
- 内部状態を気にかける必要がない

#### 使い回しが効く

画面要素がコンポーネントとして定義されていると、必要な場所すべてに対してそのコンポーネントを配置することで共通の UI を提供できます。たとえば複数の画面にユーザープロフィールを用意する必要がある場合、前述した Profile コンポーネントを必要な画面に配置するだけで画面が作成できてしまいます。これは開発効率に非常に大きく寄与します。

### Learn Once, Write Anywhere

「一度学べばどのプラットフォームでも書ける」という意味です。 Java では "Write Once, Run Anywhere" という標語を掲げていますが、それをもじったものです。「一度 React を学ぶことで、どのプラットフォームにおいてもアプリ開発ができるようになる」という思想です。

勘違いしやすいのですが、_ワンソースでクロスプラットフォーム対応するためのものではありません_。 Android と iOS では使えるコンポーネントや属性に差があるうえ、各プラットフォーム向けに作り込んでいくとワンソースでは済まなくなります。

しかし、基本的なコンポーネントの挙動は似ていますし、実行時にどのプラットフォーム上で動作しているかを判別して挙動を振り分けることも可能です。この思想はプラットフォームごとの開発を強制するものではありません。

もちろん、各プラットフォームに対してユーザー調査を繰り返した結果、全く別の UX とすべきという結論が出ることもあるでしょう。そのような場合に既存の開発チームのみで対応可能という点こそが React Native のメリットです。

React の開発主体である Facebook では、各プラットフォームごとに最適化された見た目・感触・機能を重視しているようです。ただし、現実はプラットフォームごとに開発で必要とされるスキルセットが異なります。[その差を埋めるために React を用いているようです](https://code.facebook.com/posts/1014532261909640/react-native-bringing-modern-web-techniques-to-mobile/)。次は React によってカバーされているプラットフォームの一覧です。

| プラットフォーム        | ライブラリー・フレームワーク                                                                                                                      |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Web SPA         | [ReactDOM](https://ja.legacy.reactjs.org/docs/react-dom.html) / [React Native for Web](https://necolas.github.io/react-native-web/) |
| Web server side | [Next.js](https://nextjs.org/)                                                                                                      |
| 静的サイト           | [React Static](https://github.com/react-static/react-static)                                                                        |
| Android         | [React Native](https://reactnative.dev/)                                                                                            |
| iOS             | [React Native](https://reactnative.dev/)                                                                                            |
| Windows         | [React Native Windows](https://github.com/microsoft/react-native-windows#getting-started)                                           |
| macOS           | [React Native macOS](https://github.com/microsoft/react-native-macos)                                                               |
| PDF             | [react-pdf](https://react-pdf.org/)                                                                                                 |
| CLI             | [ink](https://github.com/EvanBacon/react-native-ink)                                                                                |

Linux は対応していないのですが、 ReactDOM で開発したアプリを Electron を使ってパッキングすることにより、単体で動作させることが可能です。

## React が必要とされた背景

[公式ブログ](https://reactjs.org/blog/2013/06/05/why-react.html)において、Facebook は React を開発した理由を次のように説明しています。

- React isn't an MVC framework
- React doesn't use templates
- Reactive updates are dead simple

### React isn't an MVC framework

MVC とは設計パターンのひとつです。

MVC という言葉が使われる場合、おおまかに 2 種類のうちどちらかの意味合いで使われます。 1979 年に発表されたオリジナルの MVC と、 [Web サービス用に変更された JSP model 2 architecture](https://en.wikipedia.org/wiki/JSP_model_2_architecture) です。ここでは MVC という言葉を JSP model 2 architecture を指すものとして扱います。

次の要素で構成されており、それぞれの頭文字をとって MVC と呼ばれています。

| MVC の構成要素  | 説明                                    |
|------------|---------------------------------------|
| Model      | アプリの本質的なデータ構造と処理を表す層                  |
| View       | Model をユーザーにわかりやすく表示する層               |
| Controller | View からの入力を受け Model を生成し View を表示し直す層 |

Model、View、Controller それぞれがどのように関連するかは次の図を見ると一目瞭然でしょう。

![JSP model 2 architecture](https://upload.wikimedia.org/wikipedia/commons/thumb/7/72/JSP_Model_2.svg/440px-JSP_Model_2.svg.png =400x)

MVC モデルの動作は次のステップをたどります。

1. ユーザーが URL に対して HTTP リクエストを送信する
2. Controller が HTTP リクエストを受ける
3. Controller が適切な Model を生成する
4. Controller が生成した Model を View に注入してレスポンスを生成する
5. Controller が生成したレスポンスを返却する

ブラウザーと Controller は HTTP で繋がれています。そのためリクエストごとに View を生成する必要があることと、 HTTP が状態を持てないことから、このようなアーキテクチャーが考案されました。

当初 React のトップページには、マーケティングの都合から「MVC の V として使われることが多い」という記述がありましたが、 [2016 年 6 月にその記述が取り払われました](https://twitter.com/dan_abramov/status/741462507861233665)。このことから、実現するものが同じでも考え方は違うことが伺えます。

MVC の各層の責務が React でどのように実現されるかの対応は次のようになります。

| MVC の構成要素  | React での実現方法                                                        |
|------------|---------------------------------------------------------------------|
| Model      | props によって外部からコンポーネントに注入される                                         |
| View       | コンポーネントそのもの                                                         |
| Controller | props によって注入された Model をコンポーネントが操作する、<br/>もしくは注入された関数を使い Model を操作する |

props については後の章で詳しく説明します。ここではコンポーネントに外部から注入されるデータもしくは関数があることを理解できれば問題ありません。

この表からわかることは「MVC で実現できることは React でも実現できる」ということです。

さらに、 React では「画面の状態」に対する責務が明確になっています。「画面の状態」とは、アプリが動作するために必要な、しかしアプリが解決したい本質的な課題とは関連のない状態を指しています。タブ表示における現在アクティブなタブなどです。

MVC では「画面の状態」を Model に持たせるか、 Controller で管理する必要がありました。 Model に持たせるとアプリの本質的な状態以外を Model が抱え込むことになります。対して Controller が管理すると、 Model の操作以外の処理を担当することになります。いずれも Fat Model や Fat Controller と呼ばれ、期待されている以上の責務を負うことになっていました。この状態ではプロジェクトや開発者ごとに画面の状態を扱う場所が変わってしまい、保守性が担保しにくいという欠点がありました。

React ではコンポーネントごとに state と呼ばれる状態を持つことが可能です。ここに画面の状態を持たせることで前述した問題は解決されます。[MVC では曖昧だった責務分担がより明確になったわけです](https://medium.freecodecamp.org/is-mvc-dead-for-the-frontend-35b4d1fe39ec)。[^1]

また、 MVC ではすべての画面において Controller が必要な Model を生成し、適切に View へ渡す必要がありました。さらにユーザーからの入力など View の状態が変化することに対しても Controller が適切にハンドリングしなければなりませんでした。これは画面構成が複雑になるほど Controller の実装難度が上がる要因となっていました。

対して React では props によって親コンポーネントから子コンポーネントへ単方向に変更が伝播するという性質を持っているため、状態把握がかんたんになっています。この前提があるおかげで _state と props の中身をどう画面に表示するかという観点のみに注意を払えばよく、 MVC より開発しやすくなっています_。

[^1]: 主張はほぼ同じだが、元記事では Flux が Model 部分の責務を負うとしていたのに対し、必須ではないと考えた。また、 Business Logic が Controller の責務となっている点は元記事のおかしい点なので整理して記述している。

### React doesn't use templates

ビュー層を構築する際、テンプレートというアプローチを用いることが多くありました。テンプレートとはまさにビューの雛形を指し、 HTML Template という仕様を用いて次のように定義・使用します。

```html
<!-- HTML -->
<table id="search-result">
  <thead>
    <tr>
      <th>name</th>
      <th>summary</th>
    </tr>
  </thead>
  <tbody />
</table>

<template id="search-result-row-template">
  <tr>
    <td />
    <td />
    <td><button>detail</button></td>
    <!-- ① -->
  </tr>
</template>
```

```javascript
// javascript
var searchResults = [
  {
    name: "Alex",
    summary: "fontend engineer",
    // ②
  },
  {
    name: "Barbara",
    summary: "backend engineer",
  },
];

// ⑤
var rowTemplate = document.querySelector("#search-result-row-template");
var tbody = document.querySelector("tbody");
searchResults.forEach((searchResult) => {
  var row = document.importNode(rowTemplate.content, true);

  var tds = row.querySelectorAll("td");
  tds[0].textContent = searchResult.name;
  tds[1].textContent = searchResult.summary;

  // ③

  var button = row.querySelector("button");
  button.addEventListener("click", () => {
    alert(searchResult.name);
  });

  // ④

  tbody.appendChild(row);
});
```

この画面を拡張する場合を考えてみましょう。次のようになります。

1. テンプレートへの要素の追加

- HTML テンプレートの ① 付近に要素を追加

2. 表示する内容の追加

- JavaScript コードの ②、`searchResults` の定義にデータを追加
- JavaScript コードの ③ でデータを設定するロジックを追加

3. イベントハンドラーの設定

- JavaScript コードの ④ でインタラクションのためのコードを追加

また、⑤ で `querySelector` が実行されるまでエラーとなるかわからないことにも注意しましょう。

テンプレートではなく React を使うと、次のメリットがあります。

- 機能拡張しやすい
- 静的解析などであらかじめエラーを検出できる

比較のために同様の表示をするコードを React で書くと次となります。

```html
<!-- HTML -->
<div id="app" />
```

```javascript
// javascript
import React from 'react';
import ReactDOM from 'react-dom';

function SearchResultRow(props) {
  // ②
  return (
    <tr>
      <td>{props.content.name}</td>
      <td>{props.content.summary}</td>
      <td><button onClick={() => alert(props.content.name)}>detail</button></td>
    </tr>
  );
}

function App(props) {
  return (
    <table>
      <thead>
        <tr>
          <th>name</th>
          <th>summary</th>
          <th>detail</th>
        </tr>
      </thead>
      <tbody>
        {props.searchResults.map(
          searchResult => <SearchResultRow
            content={searchResult}
            key={searchResult.name}
          />
        );}
      </tbody>
    </table>
  );
}

// ①
var searchResults = [
  {
    name: 'Alex',
    summary: 'fontend engineer',
  },
  {
    name: 'Barbara',
    summary: 'backend engineer',
  },
];

ReactDOM.render(<App searchResults={searchResults} />, document.getElementById('app'));
```

HTML テンプレートを用いた場合に比べ、機能拡張は次の 2 点で済みます。*`SearchResultRow` の `props` と JSX 部分の把握のみが作業者が必要なことになっているため、負担が減っている*ことがわかります。

1. ① の `searchResults` にデータを追加
2. ② の `SearchResultRow` に表示、動作を追加

また、文字列を指定している箇所がないため、後述する静的解析や型チェックによって実行前にエラーがわかるという利点があります。たとえば `props.content.name` を `props.content.naem` と打ち間違えてしまった際、実行前にツールがエラーを検出してくれます。

これは React を使って書くコードがすべて JavaScript となることによる利点です。*一見 JSX は HTML と思いがちですが、実は Facebook による JavaScript の言語拡張です。*上記のコードのうち、 `SearchResultRow` は実行時には次のように変換されます。

```javascript
var SearchResultRow = function SearchResultRow(props) {
  return React.createElement(
    "tr",
    null,
    React.createElement("td", null, props.content.name),
    React.createElement("td", null, props.content.summary),
    React.createElement(
      "td",
      null,
      React.createElement(
        "button",
        {
          onClick: function onClick() {
            return alert(props.content.name);
          },
        },
        "detail"
      )
    )
  );
};
```

見やすいように適宜改行を追加しました。 `React.createElement` は React によって提供されている、要素を生成するための関数です。引数には要素名と属性が並び、最後に子要素が連なっていることがわかります。

### Reactive updates are dead simple

表示していたデータが変更された際、表示を更新する必要があります。これをそのまま実現する場合、まずデータが変更されたことを検出する必要があります。さらにその変更をどのように表示するかを指定する処理が必要です。

次は変更を検知し、表示を更新するコードの一例です。

```javascript
function init(props) {
  var count = document.createElement("span");
  count.innerText = props.count.toString(10);
  count.id = "count";
  document.getElementById("root").appendChild(count);
}

function update(props) {
  var countDom = document.getElementById("count");
  countDom.innerText = props.count.toString(10);
}

var proxyHandlers = {
  set: (target, key, newValue) => {
    target[key] = newValue;
    update(target);
  },
};

var target = { count: 0 };
var p = new Proxy(target, proxyHandlers);
init(target);

p.count = 42;
++p.count;
--p.count;
p.count += 100;
```

データの変更検出に関して、 JavaScript ではこのように Proxy を利用する他、[setter を利用する](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/set)などの手法があります。データをラップして変更の検出処理を挟み込もうという思想です。表示については最初の表示を `init` 関数が、データが変更された際の更新を `render` 関数が担当しています。

さて、この方法は十分に機能しますが、拡張性に乏しいコードです。すべてのデータに Proxy を生成し、データの初期表示と更新の方法を定義していかなければなりません。対象のデータが 10 倍、画面数が 10 倍となった際、メンテナンスできなくなってしまいます。

これに対して、 React ではコンポーネントに渡されたデータ props および内部状態 state のうち変更のあった部分だけを検出し、描画し直すというアプローチをとっています。このアプローチによって次の利点を生み出しています。

1. データの変更を検出するしくみが不要
2. 初期表示と表示の更新という場合分けが不要
3. 変更のあったものだけ更新するため描画が速い

:::details コラム: 表示速度と仮想 DOM
変更があったものだけを更新するしくみとは言え、非常に複雑な画面では表示速度が落ちてしまうことが容易に想像できます。 React ではこの問題を回避するため、仮想 DOM というアイデアが使われています。

仮想 DOM という名の通り、 DOM そのものではないですが DOM として扱っても問題ないものです。具体的には画面の全部または一部を描画するために必要な情報、およびその情報から実際の DOM ツリーを描画するための仕組みを指しています。

画面を表現するために必要な情報とは何でしょうか？HTML を思い浮かべてみてください。まず `div` や `span` などに代表される要素名が必要です。

```javascript
{
  name: 'div',
}
```

さらに `style` や `id` など、その要素が持つ属性の情報があれば要素単体は過不足なく表せそうです。

```javascript
{
  name: 'div',
  attributes: {
    style: { backgroundColor: 'red' }
  }
}
```

さらに DOM はツリー構造をとっているので、それを表現するために子要素のリストが必要となります。これを `children` という名前で表すこととしましょう。また、要素にとって属性や子要素は必須でないので、その場合は `attributes` や `children` の指定をしなければいいこととしましょう。

```javascript
{
  name: 'div',
  attributes: {
    style: { backgroundColor: 'red' }
  },
  children: [
    {
      name: 'span',
      children: [ 'Takagi Kensuke' ],
    },
    {
      name: 'img',
      attributes: {
        src: 'https://example.com/photos/takagi-kensuke.png'
      },
    },
  ]
}
```

これだけの情報があれば HTML を生成し画面を表示できるでしょう。しかし、仮想 DOM を固定のデータとして定義しても仕方ありません。パーツとして使いやすいように、外からパラメーターを渡せるようにしましょう。

```javascript
function Profile(props) {
  return {
    name: "div",
    attributes: {
      style: { backgroundColor: props.isRegular ? "red" : "white" },
    },
    children: [
      {
        name: "span",
        children: [props.name],
      },
      {
        name: "img",
        attributes: {
          src: props.photo,
        },
      },
    ],
  };
}
```

ここまでで仮想 DOM と同等のデータが定義できました。

比較のために、冒頭に紹介した次のコードを実行可能な状態に変換します。

```jsx
function Profile(props) {
  return (
    <div style={{ backgroundColor: props.isRegular ? "red" : "white" }}>
      <span>{props.name}</span>
      <img src={props.photo} />
    </div>
  );
}
```

すると次の出力が得られます。細部は異なりますが、我々が定義した `Profile` 関数ととても似ていますね。

```javascript
var Profile = function Profile(props) {
  return React.createElement(
    "div",
    {
      style: { backgroundColor: props.isRegular ? "red" : "white" },
    },
    React.createElement("span", null, props.name),
    React.createElement("img", {
      src: props.photo,
    })
  );
};
```

仮想 DOM は次から成り立っています。

- 仮想 DOM ノードの名前
- 属性名とその指定値
- 子仮想 DOM ノードの一覧
- パラメーターを流し込める機能

上述したコードを見ると、 `span` 要素で表示している名前の部分は `props.name` が変更された際にのみ再描画すれば常に適切な表示を保てることがわかります。この仕組みは React によって提供されており、差分レンダリングと呼ばれています。表示の高速化という恩恵が得られる、仮想 DOM の大きな強みです。

仮想 DOM は React に特有のものではありません。しかし React では[差分検出アルゴリズムをチューニング](https://reactjs.org/docs/faq-internals.html)しており [Reconciliation](https://reactjs.org/docs/reconciliation.html) として説明されています。また、差分レンダリングについても [React Fiber](https://speakerdeck.com/koba04/ready-for-async-rendering) として最適化が施されています。

## React Native が必要とされた背景

React Native は、 React を用いて Android と iOS のアプリ開発が可能なフレームワークです。前述した React の利点を享受できます。また、モバイルアプリの開発においても発達した JavaScript エコシステムの恩恵を受けられることがメリットです。

多くのアプリが React Native で作成されています。たとえば、 [Facebook 公式アプリや Microsoft Office アプリなどは React Native 製](https://reactnative.dev/showcase)です。日本国内では Togetter や pickss などが React Native で作成されています。

次の理由から React Native は生み出されました。既存の課題解決以上に、アプリ開発に際して多大なメリットがあります。

- ハイブリッドアプリに比べて速度問題が生じにくい
- ネイティブ UI を使ったユーザー体験を提供できる
- アプリ開発が必要なプラットフォームに対して React に慣れた開発者であれば開発ができる
- 従来のアプリ開発に比べて開発効率が高い

これらのメリットをもたらす要素については [Facebook のブログ](https://code.facebook.com/posts/1014532261909640/react-native-bringing-modern-web-techniques-to-mobile/)では、以下のように説明されています。

- Native Experience
- Learn Once, Write Anywhere
- Keeping Development Velocity

### Native Experience

各プラットフォームを提供している企業は、そのプラットフォームで最上の体験ができるよう、各 UI を作り込んでいます。たとえば iOS では Apple が、 Android では Google がユーザーのために日々アップグレードをしてくれています。そのためアプリを作る場合、ユーザーに提供できる体験を向上させるにはプラットフォームごとのネイティブ UI を使うことが近道です。

対して Web システムをアプリとしてパッキングして提供するハイブリッドアプリという手法がありますが、ブラウザーによって提供されるものや自ら作り込んだものを UI として提供することになります。しかし、この手法は次の課題を抱えています。

- ネイティブ UI とまったく同じものとはならない
- ネイティブ UI に似せるための開発コストが大きい
- ネイティブ UI に変更があった際、追従コストが発生する

ネイティブ UI と異なる場合、ユーザーは違和感を感じます。さながら言葉はそのままなのに、全く違う文化圏のジェスチャーを交えて会話しているような感覚を覚えるでしょう。これはユーザーに UI の使い方についての学習を強いてしまい、最悪使われなくなってしまいます。

また、ネイティブ UI に比べるとどうしても速度の問題がついてまわります。ネイティブ UI を直接操作するわけではなく、アプリ内ブラウザーを用いて擬似的に UI を描画していくわけですから描画速度はあがりません。

React Native ではこの課題を解決するためにネイティブ UI をそのまま使う形式をとっています。

React のプログラミングモデルである Declarative が幸いし、ネイティブ層の状態について考えなくてもビュー構築が可能という利点があります。ネイティブ層をコンポーネントという部品で扱うという思想です。

[Facebook はかつて HTML5 へ舵を切った後、またネイティブアプリへ回帰しています](https://techcrunch.com/2012/09/11/mark-zuckerberg-our-biggest-mistake-with-mobile-was-betting-too-much-on-html5/)。この理由は[ユーザー体験が貧弱なことと開発ツールが揃っていないこと](https://www.infoq.com/jp/news/2012/09/Facebook-HTML5-Native)だったようです。 React Native はこの点をクリアするために開発されています。

### Learn Once, Write Anywhere

前述したとおり、「一度 React の書き方を学ぶと、どのプラットフォームでもアプリ開発ができる」ということです。 React での開発経験をプラットフォームをまたいで持ち越すことが可能です。

[現在、デスクトップマシンやラップトップマシンなど他の形態のデバイスを抑え、スマートフォンの使用時間が最も長いという調査結果が出ています](http://www.soumu.go.jp/johotsusintokei/whitepaper/ja/h29/html/nc111210.html)。ターゲットにもよりますが、これを踏まえてモバイルアプリを開発すべきという結論に至ることは珍しくないでしょう。

ただし、通常 iOS のネイティブアプリを開発する際に使う言語は Swift もしくは Objective-C です。ビュー層では xib や Storyboard 、 SwiftUI などを用いて記述します。 Android のネイティブアプリ開発では Kotlin や Java を言語として用い、 xml ファイルでビュー層を記述します。

つまり、プラットフォームごとに習熟が必要ということを示しています。前述したモバイルアプリを出したいという状況であってもふたつのプラットフォームに習熟したメンバーが必要となるわけです。

さらに、大体の状況において管理用のシステム開発が必要になるでしょう。その場合、開発コストやユースケースを鑑みて Web アプリが選択される傾向にあります。また、ユーザーへ Web アプリ版を提供するという決定がなされることもあるでしょう。習熟すべきことが増えてしまい、開発メンバーの負荷は非常に高くなるか、チームをいくつも維持することになり金銭的なコストが高くなるでしょう。

この課題に対して、 React Native では同じ技術によって全プラットフォーム上で開発可能となるメリットがあります。 JSX を使ってビュー層を書き下していくことにより、どのプラットフォーム上でも React というスキルセットを持ったチームによって開発ができるのです。

### Keeping Development Velocity

従来のモバイルアプリ開発では画面の微調整をするために都度コンパイルが必要でした。ボタンの位置を少し変えたり色の微調整をしたり、アニメーションのテンポを変えたりなどといったことをするためにコンパイル時間を待たなくてはいけませんでした。

対して Web システムの開発では何かしらコードに変更を加えた際、その結果を見るためにはブラウザーをリロードするだけで可能でした。最近は合間にトランスパイルやバンドルといった処理がある場合もありますが、それも長くて数秒で済みますし自動的にリロードしてくれるツール群があります。

React Native ではこのテンポ感をアプリ開発に持ち込んでいます。ビュー層の定義を変更した際、やはりトランスパイルとバンドルが必要となりますが、各プラットフォームでのコンパイル時間に比べるとその所要時間は非常に短時間です。変更を検知して反映してくれる Fast Refresh と呼ばれる機能を使った場合、体感としては書き換えていくそばから反映される状態です。

これは Web システム開発者にとっては当たり前のことですが、ネイティブ開発者にとっては朗報です。今まで時間がかかっていた UI の開発効率が高くなります。
