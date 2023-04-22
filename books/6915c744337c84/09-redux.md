---
title: "データフローの設計および実装"
---

## Redux とは

極論を言うと、アプリケーションとはユーザーにとって有用なデータを適切なタイミングで引き出し、更新するための一連の枠組みのことです。この章ではアプリケーションの作成にあたって重要となるデータの取扱いを実装していきます。

Redux とはアプリケーションの状態を管理するためのライブラリーです。Facebook の Dan Abramov 氏と Andrew Clark 氏によって開発されました。[^1][^2][^3]

データ変更と非同期処理は実践的なアプリケーションを作成する際に必要となりますが、その組み合わせは取り扱いづらく、爆発的に複雑になることがわかっています。Redux はいくつかの制限を導入することで、データ変更と非同期処理に対して「状態の変更を予測可能」とすることを目的としています。

この目的を達成するため、次の特徴を持っています。[^4]

- Single source of truth
- State is read-only
- Changes are made with pure functions

[^1]: https://redux.js.org/
[^2]: https://github.com/gaearon
[^3]: https://github.com/acdlite
[^4]: https://redux.js.org/introduction/three-principles

### 特徴 1. Single source of truth

Redux ではアプリケーションの状態を Object のインスタンスに格納します。これを Store と呼んでいます。Store とは貯蔵する場所という意味合いです。データのみが格納され、メソッドを持ちません。以降、本書ではこのデータ型を「プレインオブジェクト」と呼ぶことにします。

たとえば、Praiser で管理する Todo をデータとして表現すると次のようになります。

```typescript
{
  todos: [
    {
      id: '568a2cb1-b769-4be0-9fca-31039e808969',
      title: 'install Redux',
      detail: 'yarn add --exact redux',
      createdAt: 1575106837382,
      updatedAt:  1575193155087,
      completedAt: 1575193155087,
    },
    {
      id: '44f815f8-57ca-4e62-a7f2-411c808c14b8',
      title: 'create Reducers',
      detail: 'See Redux documents'
      createdAt: 1575193390669,
      updatedAt: 1575193390669,
      completedAt: null,
    }
  ],
}
```

Redux で Store をこのように設計しているのは次の 3 つの理由があるからです。

- Store にのみ状態が格納されているので、デバッグしやすい
- シリアライズ可能なのでキャッシュしやすい・サーバーとのデータのやりとりをシンプルに実装可能
- Store の状態をひとつ過去にもどすことで Undo を、ひとつ未来に進めることで Redo を実装可能

特に、1 の理由は開発効率に直結します。バグが発生した際、まず Store が想定通りか確認し、そこからビュー層の実装が想定どおりか確認するという流れで調査の大半が完了します。個人のスキルや直感によらず、調査方法を平準化できることは非常に大きな利点です。

同時にプレインオブジェクトという単純なデータ構造なので、求められる前提が少ない点もチーム開発に向いていると言えるでしょう。一般的なプレインオブジェクトの取扱いをそのまま適用できるため、JavaScript での開発経験がそのまま流用できます。逆に Redux での経験が JavaScript 習熟の一助となることもメリットのひとつでしょう。

シリアライズとは、メモリー上に散在するデータを連続するデータとしてパッキングすることを言います。シリアライズされた有名なデータ形式のひとつに JSON があります。シリアライズの目的はファイルやデータベースへの保存およびネットワーク通信のペイロードとして使用することです。

Redux を用いたアプリでは、ユーザーによる操作などによって Store の状態が刻々と更新されていきます。状態が時系列で連続しているのです。ということは状態をひとつ古いものへ巻き戻すとアプリ全体が Undo の挙動を取ることになります。逆に巻き戻した状態から新しいものへ戻すと Redo となります。

### 特徴 2. State is read-only

状態の読み取りは `getState` というインタフェースを用います。状態を表すプレインオブジェクトを取得できます。

```typescript
const appState = store.getState();
```

このコードを実行すると `appState` 変数には Store が持つすべての状態が格納されます。が、あくまでもこれは参照専用です。データを更新したいからといって次のようなコードを書いてはいけません。

```typescript
appState.todos[1].completedAt = Date.now();
```

状態を更新するためのインタフェースとして `dispatch` というメソッドが提供されています。「書類や荷物などを発送する」という意味合いです。状態更新の指示が書かれた書類を送るイメージです。次は特定の TODO を完了にするよう指示するコードです。

```typescript
store.dispatch({
  type: "todo/complete",
  id: 1,
  completedAt: Date.now(),
});
```

予め Store に対して `subscribe` でイベントリスナーを登録しておくことで、`dispatch` 呼び出し時にイベントリスナーを呼び出してくれます。この通知によって常に現在の状態をビュー層が表現することを担保しています。例として書くと次のようになるでしょう。

```typescript
const unsubscribe = store.subscribe(() => {
  // stateをログに出す
  console.log(store.getState());

  // 他になにかする
});
```

前述の `getState` メソッドによって取得した状態を書き換える方法では、状態変化の通知がなされません。状態を直接書き換えても、アプリの見た目が更新されないのです。これは望む動作ではないでしょう。

`dispatch` によって発送される状態更新の指示を、Redux では Action と呼んでいます。Action に求められる条件は `type` というプロパティを持っているプレインオブジェクトであることのみです。`type` はどういった種類の更新なのかを指定するためのものです。わかりやすいよう文字列定数を用いることが一般的となっていますが、開発者が把握できればどんな値でも問題ありません。`type` プロパティによって指定される値を Action Type と呼びます。

実際の Action がどのような構造を持っているか確認しましょう。次はカウンターの値をひとつ増やす Action を `dispatch` しています。

```typescript
const action = {
  type: "counter/increment",
};
store.dispatch(action);
```

Action は `type` 以外のプロパティを自由に持ってよいため、状態更新に必要な情報を同時にもたせられます。特定の TODO を完了とする Action では「どの TODO を対象とするか」と「完了時刻」の指定が必要です。

```typescript
store.dispatch({
  type: "todo/complete",
  id: 1,
  completedAt: Date.now(),
});
```

こういった付属データの指定を考慮すると、必要となるすべての Action をリテラルで定義するのは不可能です。次のように Action を生成する関数を使用しましょう。

```typescript
function setSearchKeywords(id: number) {
  return {
    type: "todo/complete",
    id: 1,
    completedAt: Date.now(),
  };
}

store.dispatch(actionCreator("1"));
```

こういった、Action を生成する関数を Action Creator と呼びます。Action の生成を共通化することでミスが少なくなるだけでなく、Action Creator をテストすることで正しく Action が生成されるかを確かめやすくなるメリットがあります。

Action をプレインオブジェクトとして表現することの利点は以下の 3 つです。

- Store と同様、プレインオブジェクトの中身を検査していくことが可能になり、デバッグがかんたんになる
- Action を JSON などの形式で記録しやすい
- 初期状態と Action すべてを発行された順とともに記録しておけば、任意の状態を再現できる

最後に挙げたメリットは `dispatch` がひとつずつ、同期的に処理されることから生じています。これは原理的にデータ更新に関する競合が発生しないことと同義です。現在の状態と `dispatch` する Action さえわかれば一意に状態が定まるため、バグの発生する余地がありません。

#### Flux Standard Action

Redux Action は `type` プロパティを持つプレインオブジェクトであること以外は規定されていません。これでは実装者によって差異が出てしまい、効率的な開発の妨げとなっていました。この課題への対応として、Action の細かいルールを定める方法が挙げられます。Flux Standard Action（FSA）は有名なルールのひとつです。[^7]

FSA は次のルールを持っています。

1. プレインオブジェクトでなければならない
2. `type` プロパティを持っていなければならない
3. `payload` プロパティを持つことができる
4. `error` プロパティを持つことができる
5. `meta` プロパティを持つことができる
6. `type`、`payload`、`error`、`meta` 以外のプロパティを持ってはならない

1 つめと 2 つめのルールは Redux Action の決めごとと同様です。

`payload` プロパティには Action として必要なデータを詰め込みます。どのようなデータでも構いません。

```typescript
// 特定のTODOを完了にするAction
const completeTodoAction = {
  type: "todo/complete",
  payload: {
    id: 1,
    complatedAt: Date.now(),
  },
};
```

`error` プロパティは何かしらのエラーを `dispatch` する場合に指定します。`true` 以外の場合、エラーとして扱ってはいけません。`true` の場合、`payload` には Error インスタンスを指定します。

```typescript
// 存在しないidを指定された場合のエラーを通知するAction
const action = {
  type: "todo/complete",
  error: true,
  payload: {
    error: new Error("specified id is not found"),
  },
};
```

`meta` プロパティは `payload` プロパティ以外で Action に載せたいデータを詰め込みます。ライブラリーなどで使用されます。

```typescript
// my-libへの指示を含んだAction
const action = {
  type: "todo/complete",
  payload: {
    id: 1,
    complatedAt: Date.now(),
  },
  meta: {
    library: "my-lib",
    someFlag: true,
  },
};
```

本書では FSA に則った Redux Action を使用します。

[^7]: https://github.com/redux-utilities/flux-standard-action

### 特徴 3. Changes are made with pure functions

#### 純粋関数とは

pure function は日本語では純粋関数と訳されています。関数とは数学で定義されている関数を指しています。本書を読む上では「入力されたデータを処理し、何らかの出力をするもの」と捉えておけば問題ありません。純粋関数とは次の 3 つの条件を持つ関数のことです。

- 返り値が引数のみに依存していること。言い換えると、同じ引数であれば常に同じ返り値を出力すること
- 副作用を持たないこと
- 純粋関数以外を呼び出さないこと

副作用とは、コンピューターの状態を変化させることを指します。次は状態と副作用の具体的な一例です。

| 状態                 | 副作用                   |
| -------------------- | ------------------------ |
| グローバル変数       | グローバル変数の変更     |
| データベース         | データベース更新         |
| ネットワーク通信状態 | ネットワーク通信         |
| 周辺機器との通信状態 | 周辺機器との入出力       |
| 参照渡しされた引数   | 参照渡しされた引数の変更 |

通信状態についてはネットワーク・周辺機器ともに、対向するサーバー・デバイスから受け取ったデータを一時的にコンピューター内部に蓄える必要があります。この部分が状態です。

最後に挙げた例について、JavaScript および TypeScript ではオブジェクトや配列は参照渡しとなるため、引数を変更すると副作用を生んでしまいます。TypeScript では次のように書くことで不注意による引数の変更をある程度防ぐことが可能です。

```typescript
interface Rect {
  width: number;
  length: number;
}

function foo(rect: Readonly<Rect>) {
  rect.width = 3; // エラー！

  // エラーとならない
  rect = {
    width: 3,
    length: 4,
  };
}

function bar(numbers: Readonly<Array<number>>) {
  numbers[0] = 1; // エラー！
  numbers = [0, 1, 2]; // エラーとならない
}
```

引数が持つプロパティや要素の書き換えについてはエラーを出力させられますが、引数自体の書き換えは防げないので注意しましょう。

#### Reducer

`dispatch` された Action は Reducer と呼ばれる純粋関数によって、状態を更新するために使用されます。Reducer の役割は現在状態と Action から次の状態を作り出すことです。次のコードはカウンターを実装した Reducer の例です。

```typescript
function createInitialState() {
  return 0;
}

function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    case "counter/increment":
      return state + 1;
    case "counter/decrement":
      return state - 1;
    default:
      return state;
  }
}
```

現在の状態 `state` と `action` を引数に取り、次の状態を返す関数となっています。`state` のデフォルト引数は `createInitialState` 関数によって生成された state の初期状態です。

Reducer が満たすべき条件はひとつです。ハンドリング可能な Action が渡された場合、必ず新しい状態を生成し返すようにすることです。 `state` が引数として渡されていますが、Reducer は純粋関数でなければならないので、これを変更してはいけません。

ではなぜ Reducer は純粋関数である必要があるのかを考えてみましょう。反例として Reducer が純粋関数でないと仮定してみましょう。次は Reducer において最終更新時刻を書き換えている例です。

```typescript
function createInitialState() {
  return {
    lastUpdatedAt: undefined,
  };
}

function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    case "xxx":
      return {
        ...state,
        lastUpdatedAt: Date.now(),
      };
    default:
      return state;
  }
}
```

このような Reducer では、全く同じ Action が `dispatch` されたとしても、その時々でアプリケーションの状態が異なってしまいます。Reducer が呼ばれた時刻に依存した値で `lastUpdatedAt` の更新がなされるからです。この状況でもし `lastUpdatedAt` に依存して分岐する処理があった場合、どの分岐が実行されるかはソースのみからでは読み取れなくなってしまいます。

これは Redux の目的である「開発者が状態変更について予測可能である」ことに反しています。そのため、Reducer を純粋関数に限定するという制限が設けられているのです。

ちなみに上述のように、時刻を状態として持ちたい場合は次のように Action に詰め込む設計となります。この Action を何回 `dispatch` しても同一の状態となるからです。

```typescript
const action = {
  type: "xxx",
  payload: {
    lastUpdatedAt: Date.now(),
  },
};

function reducer(state = initialState, action: Action) {
  switch (action.type) {
    case "xxx":
      return {
        ...state,
        lastUpdatedAt: action.payload.lastUpdatedAt,
      };
    default:
      return state;
  }
}
```

#### Reducer の分割

Reducer の原理はかんたんですが、処理すべき Action Type が増えると複雑さも上昇します。関心事によって Reducer を分割すべきです。たとえば Todo 管理もできるポモドーロアプリでは、Todo を管理する Reducer とタイムウォッチを管理する Reducer に分割が可能でしょう。

```typescript
// todos.ts
export function createInitialState() {
  // （中略）
}

export function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    case "todo/add":
    // （中略）
  }
}
```

```typescript
// timer.ts
export function createInitialState() {
  // （中略）
}
export function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    case "timer/reset":
    // （中略）
  }
}
```

ただし Store はひとつのため指定可能な Reducer はひとつです。そのために Redux では `combineReducers` というユーティリティ関数が提供されています。

```typescript
// reducer.ts
import { combineReducers } from "redux";

import * as Todos from "./todos";
import * as Timer from "./timer";

export interface AppState {
  readonly todos: ReturnType<typeof Todos.createInitialState>;
  readonly timer: ReturnType<typeof Timer.createInitialState>;
}

export function createInitialState(): AppState {
  return {
    todos: Todos.createInitialState(),
    timer: Timer.createInitialState(),
  };
}

export default combineReducers({
  todos: Todos.reducer,
  timer: Timer.Reducer,
});
```

`ReturnType` は TypeScript から提供されているユーティリティー型です。Generic パラメーターに指定した関数の戻り値の型を返すものです。ここではアプリの状態を示す `AppState` 型を定義するために使用しています。

`combineReducers` で結合した場合、各 Reducer が持っていた `state` は引数に指定したプレインオブジェクトのそれぞれのプロパティの値として格納されます。上述したコードでは次のようになります。

```javascript
{
  todos: /* todos.tsで定義したcreateInitialStateによって生成された初期状態 */,
  timer: /* timer.tsで定義したcreateInitialStateによって生成された初期状態 */,
}
```

`dispatch` された Action はすべての Reducer に渡されます。このため、`type` 定義が重複していると思わぬ不具合の原因となりますので注意してください。Reducer の名前をプレフィクスとしてつけるなどして一意性を担保するとよいでしょう。

実際にアプリを構築する際、`combineReducers` は必須となるでしょう。本書でも使用します。

以上が Redux の構成要素です。次の図はアプリがどのように Redux を使用するかを表した図です。

![アプリがどのようにReduxを使用するか](../images/09-redux-overview.jpg)

:::detals コラム: Flux
Redux は Flux と呼ばれる設計パターンを参考にして作られています。

Flux はそれ以前の設計パターンの代表であった MVC、つまり JSP model 2 architecture と比べると、データの流れが一方向に限定されていることが特徴です。大規模な MVC アプリケーションでは Controller を介して Model と View が複雑に絡み合うため、データの流れを追いづらく、適切に変更することが難しいという問題がありました。

![MVC complexity](https://res.infoq.com/news/2014/05/facebook-mvc-flux/ja/resources/flux-react-mvc.png)

この問題に対して Flux ではデータの流れを一方向に限定するという解決策をとっています。これは公式サイトの図を見ると一目瞭然でしょう。

![flux diagram](https://github.com/facebook/flux/raw/master/docs/img/flux-diagram-white-background.png)

次の点を除いて Redux と構成は同一です。

- Store が 1 つではない
- Redux では Store のメソッドである dispatch が Dispatcher として独立している
- Reducer が Callback という名前になっている
- Callback は純粋関数でなくてよい

また、データフローを扱う数多くのライブラリーから参考にされています。有名なものは次のとおりですが、思想が明快なこともあり様々な方が様々な思想で同様のライブラリーを作成されています。

- Facebook が開発した flux-utils
- Vue.js での使用を前提に開発された VueX
- より簡易に書けることを目指した MobX

Redux と比べた際の利点は既存 Web システムへの導入が容易なことです。URL 単位で MVC を Flux に置換していくといったことが可能なためです。残念ながら React Native では、Web システムに比べ規模が小さいこと、Redux とモバイルアプリの親和性が高いことなどから、Flux を選択するインセンティブが働きづらいでしょう。
:::

:::details コラム: Redux 以外の Store 実装
MobX など類似のデータフロー系ライブラリーでは、Store に相当するものの中身がクラスインスタンスとして提供されています。それぞれのクラスインスタンスに getter/setter が定義されており、getter をカスタマイズ可能であったり、setter を呼び出すことで状態の変更を各所に通知します。

このアプローチは単純なデータ構造のアプリにおいては、把握すべき事柄が少なく抑えられるため、開発者の負担が減ることは確かです。しかし、複雑なデータ構造を持つアプリでは getter がデータを隠蔽してしまっているため、デバッグ効率が著しく減退してしまいます。

また、Store が複数作成されるものでは、シリアライズすべき対象に抜け漏れが発生する可能性を排除できません。シリアライズ対象となるすべてのデータに対して気を使わなければならない点は実装効率にも影響するでしょう。Undo/Redo についても同様に、Store の変更履歴を別途記録しておく必要があり、実現が困難となります。
:::

:::details コラム: 決定性有限オートマトン ミーリ・マシン
Redux を端的に言い表すと、「現在の状態と入力を受け取って新しい状態を作り出すもの」となります。これは計算機科学では有限オートマトン、より詳しくいうとミーリ・マシンという数理モデルとして定義されています。[^5]

ミーリ・マシンの構成要素は次表の左の列に挙げている定義がなされています。Redux においてそれらに対応するものを右の列に記載しました。

| ミーリ・マシン   | Redux                            |
| ---------------- | -------------------------------- |
| 状態の有限集合 S | Store の中の State               |
| 初期状態 S0      | initialState                     |
| 入力の有限集合 Σ | Action                           |
| 出力の有限集合 Λ | React コンポーネントの props     |
| 遷移関数 T       | Reducer                          |
| 出力関数 G       | react-redux の `mapStateToProps` |

ミーリ・マシンは実際の製品でも幅広く使われています。実績のある非常に強力なモデルです。

- ストップウォッチ
- 自動販売機
- 信号機
- バーコードスキャナー

[^5]: https://en.wikipedia.org/wiki/Mealy_machine

:::

### どのような場合に Redux を使うのが有効か

Redux の特徴からメリット・デメリットを考えると次のようになるでしょう。

| 特徴                                 | メリット                                           | デメリット                                     |
| ------------------------------------ | -------------------------------------------------- | ---------------------------------------------- |
| Single source of truth               | アプリケーション状態のシリアライズが容易           | 状態を分割しなければならない場合に使用できない |
| State is read-only                   | 状態の把握・予測が容易                             | 特になし                                       |
| Changes are made with pure functions | アプリケーション状態の設計をコードから理解しやすい | 書き方のルールを学ぶための労力が必要           |

状態を保存する必要のあるアプリケーションでは Redux が非常に有効です。Store から取り出したプレインオブジェクトをシリアライズし、永続化することで目的がかないます。React Native ではモバイルアプリという性質上、ユーザーごとの設定を持つことも多いため、親和性が高いと言えます。

また、アプリケーションの状態が大きくなってもメンテナンスしやすいこと、チームメンバー間で書き方にブレが生じにくいことから大規模開発にも向いています。Redux の厳格な状態更新の手順は多少面倒ですが、それだけです。状態の更新方法が複数ある場合、選択が異なることを防ぐためにメンバー間で認識をあわせるコストが生じます。Redux の状態更新の煩雑さは、アプリケーションが持つ本来の複雑性に対処するための、必要な複雑性と理解できます。

既存アプリケーションがすでに大きくなっており、一度に書き換えることが難しい場合、Redux の導入は非常にハードルが高くなります。徐々に置き換えていくというアプローチがとれないからです。

さらに、新しく Redux を学ぶ余裕がない状況でも導入は困難になります。アプリケーション状態を設計するためのライブラリーという性格上、開発開始時に導入を決める必要がありますが、技術選定時は運用も含めての知見が求められるからです。パイロットプロジェクトなどでチームが使いこなせるかを見極める必要があります。

ただし、Redux は世界的にも有名かつ、使われているライブラリーです。公式サイトの記述も丁寧ですし、日本語での解説も多く、教材に事欠きません。採用しているプロジェクトの GitHub リポジトリー上で実際のコードを読みながら学ぶことが可能です。比較的学習しやすいと言えるでしょう。

以上から、次の条件が Redux を使うための前提となります。これらを満たせない場合、採用すべきではありません。

- 新規開発であること
- チームメンバーが新しく技術を学習する余裕がある

また、次の条件に多く合致するほど Redux のメリットを享受できます。

- 状態の保存が必要となる
- チーム開発である
- 複雑化もしくは大規模化する可能性がある

言い換えると、これらの条件に当てはまらない場合はそもそも Redux を使う必要はありません。しかし、アプリケーションはどのように成長するかわからないものです。業務で使う場合、こういった条件が発生しやすいため、最初期に導入を検討することが必要でしょう。

対して Redux のデメリットは次になります。

- 追加パッケージであるため、別途インストールが必要であること
- 状態を更新するためのコードが多いこと

React のみでアプリケーションの要件を満たせる場合は使用しないほうがよいでしょう。不必要な複雑性は開発効率を低下させます。

### React State の限界

4 章で説明したように、React では State という状態を管理する仕組みがありました。Redux も状態管理のための仕組みですが、同じ仕組みを改めて導入したのはなぜでしょうか？

React の State はコンポーネントに状態を持たせて管理するものです。これを用いてアプリケーションを構築する場合、次のどちらかを選択する必要があります。

1. アプリケーションの状態を各コンポーネントにもたせる
2. ルートコンポーネントにすべての状態を集約させる

#### アプリケーションの状態を各コンポーネントにもたせる

React コンポーネントは UI 部品であるという性質上、このアプローチはアプリケーションの状態が UI に束縛されてしまいます。わかりやすく管理するために画面ごと、もしくは機能を持つペインごとなどに状態をわけたとしても、どうしても UI との関連を考えなければなりません。

うまく分割できたとしても、どこにどの状態を持っているのかの把握が大変になります。メンテナンス性・開発効率が下がってしまう可能性があります。

さらに大きいデメリットとして、状態を保存することが難しくなります。各コンポーネントがシリアライズ用のメソッドを備え、それらをもれなく呼び出すことが必要になります。

また、アプリケーション全体にまたがる状態についてはこのアプローチは使えません。次のアプローチが必要になります。

#### ルートコンポーネントにすべての状態を集約させる

ルートコンポーネントとは React において最初にマウントするコンポーネントを指しています。`AppRegistry.registerComponent` で指定するものです。より具体的には、7 章で説明したナビゲーションコンポーネントが一般的でしょう。単画面のアプリの場合、単純な React コンポーネントとなります。

ルートコンポーネントに状態をもたせた場合、必要なコンポーネントへ必要な状態を渡していくことに関して課題が出てきます。React コンポーネント間でデータを渡す場合、2 つの方法があります。

```typescript
// ①明示的に渡すもの指定する
<MyComponent a={this.state.a} b={this.state.b} />

// ②すべてを渡す
<MyComponent {...this.state} />
```

① の書き方では明示的に渡すデータを書いていく必要があります。そのデータを直接使わない中間層に対しても書いていく必要があるため、コンポーネントが密結合になってしまいます。また、何より面倒です。

② の書き方について上述した欠点はないものの、子コンポーネントすべてにおいて再描画されてしまい、パフォーマンスが落ちてしまいます。単純なアプリであれば許容できる場合も多いでしょうが、大規模化してきた際に顕在化しやすく、修正のコストが非常に高くついてしまいます。

以上の問題から、大規模なアプリにおいて React コンポーネントの State にアプリ状態をもたせることは現実的に不可能です。React の State による管理は 1 画面で、それほど複雑でないアプリに限定すべきでしょう。

### Redux の基本

では Redux に触れてみましょう。かんたんのため、題材はカウンターとします。インクリメントとデクリメントのボタンによってカウンターの数値が上下するアプリを作ります。エッセンスをわかりやすくするため、まっさらな状態から組み上げます。

```console
# 作業ディレクトリーに移動してください
npx react-native init ReduxCounter --template react-native-template-typescript@6.3.16
cd ReduxCounter
yarn add --exact redux react-redux
yarn add --dev --exact @types/react-redux
```

react-redux パッケージは React の世界と Redux の世界をつなぐためのものです。

#### Reducer のベース作成と初期状態の設計

まずカウンターの初期状態を設計します。State は Reducer によって変化しますが、`createInitialState` を最初に定義し、その戻り値の構造に則った Reducer を書くことで設計しやすくなります。

Redux の流儀として Reducer を記述するファイルに初期状態も定義することになっています。`src/reducers/counter.ts` を作り、現在のカウントを表す `current` を定義します。

```typescript
// /src/reducers/counter.ts
export function createInitialState() {
  return {
    current: 0,
  };
}

export type State = ReturnType<typeof createInitialState>;
```

次に Reducer のベースを作成しましょう。同じファイルに引き続いて記述してください。

```typescript
// /src/reducers/counter.ts
export default function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    default:
      return state;
  }
}
```

見てのとおり `switch` によって Action Type ごとに処理を切り分けるものです。`default` はそのまま `state` を返していることが重要です。自分が処理できる Action Type 以外は何もせず、そのままの State を返すようにします。

`Action` 型は Action の設計後に定義しましょう。

#### Action の設計

続いて Action を設計していきましょう。インクリメント、デクリメントの 2 種類の Action が必要になります。対応する Action Type も 2 つ定義しますが、扱いやすくするため定数化しましょう。`src/constants/counter.ts` に次を定義します。

```typescript
// /src/constants/counter.ts
export const INCREMENT = "counter/increment" as const;
export const DECREMENT = "counter/decrement" as const;
```

それぞれの定義において末尾に書かれている `as const` は TypeScript 3.4 で導入された const assertions です。変数 `INCREMENT` の型を `string` ではなく `'counter/increment'` としてくれるものです。この指定は重要ですので忘れないようにしましょう。

これを用いて Action を設計します。インクリメント、デクリメントそれぞれ、次のようになります。`type` のみを持つ Action です。

```typescript
// カウンターの値を1増加させるアクション
{
  type: INCREMENT,
}

// カウンターの値を1減少させるアクション
{
  type: DECREMENT,
}
```

ここまでシンプルであれば直接 `dispatch` してもよいですが、汎用的な作りを学ぶため、Action Creator を定義しましょう。`src/actions/counter.ts` を次のように記述してください。

```typescript
// /src/actions/counter.ts
import { INCREMENT, DECREMENT } from "../constants/counter";

export function increment() {
  return {
    type: INCREMENT,
  };
}
export function decrement() {
  return {
    type: DECREMENT,
  };
}

export type Action =
  | ReturnType<typeof increment>
  | ReturnType<typeof decrement>;
```

関数として定義した `increment`、`decrement` をそれぞれ実行すると Action が手に入ります。ふたつの Action の型の Union Type を `Action` として定義し、`export` しています。Reducer の第 2 引数に指定するための定義です。

さて、Reducer にこれらの Action を `dispatch` したときの処理を記述していきましょう。先ほどの Reducer の定義を次のように書き換えましょう。

```typescript
// /src/reducers/counter.ts
import { INCREMENT, DECREMENT } from "../constants/counter";
import { Action } from "../actions/counter";

// （中略）

export default function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    case INCREMENT:
      return {
        ...state,
        current: state.current + 1,
      };
    case DECREMENT:
      return {
        ...state,
        current: state.current - 1,
      };
    default:
      return state;
  }
}
```

これで State、Reducer、Action の設計が完了しました。この一連の流れは Redux を用いたプロダクトのコアの設計になります。実際のプロダクト開発ではそれぞれ行きつ戻りつしながら設計することになるでしょう。

#### Store の作成

Reducer を定義できたので Store を作成しましょう。`src/store.ts` を次のように記述します。

```typescript
// /src/store.ts
import { createStore } from "redux";

import counterReducer, { createInitialState } from "./reducers/counter";

const store = createStore(counterReducer, createInitialState());

export default store;
```

Store はアプリにひとつのみで限定したいため、作成したものを `export` する作りとしています。

#### Redux の世界と React の世界をつなぐ

これで Redux 側の準備が整いました。まず Counter コンポーネントを `src/components/Counter.tsx` に定義しましょう。

```typescript
// /src/components/Counter.tsx
import React from "react";
import { Button, StyleSheet, Text, View } from "react-native";

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  counter: {
    fontSize: 20,
  },
});

interface Props {
  count: number;
  actions: {
    increment: () => void;
    decrement: () => void;
  };
}

export default function Counter(props: Props) {
  return (
    <View style={styles.container}>
      <Text style={styles.counter}>{props.count}</Text>
      <Button onPress={props.actions.increment} title="increment" />
      <Button onPress={props.actions.decrement} title="decrement" />
    </View>
  );
}
```

React コンポーネントは Redux の世界のことを全く知らないことに注目してください。props に渡される各種データによってのみ成り立っています。

では react-redux パッケージを用いて Redux の世界のデータを React 側へ props として渡しましょう。`/src/containers/Counter.tsx` を作成し、編集します。

```typescript
// /src/containers/Counter.tsx
import React from "react";
import { useSelector, useDispatch } from "react-redux";

import { State } from "../reducers/counter";
import { increment, decrement } from "../actions/counter";
import Counter from "../components/Counter";

export default function ConnectedCounter() {
  const count = useSelector((state: State) => state.current);
  const dispatch = useDispatch();
  const actions = React.useMemo(
    () => ({
      increment() {
        dispatch(increment());
      },
      decrement() {
        dispatch(decrement());
      },
    }),
    [dispatch]
  );

  return <Counter count={count} actions={actions} />;
}
```

`Provider` コンポーネントを使い、アプリ全体に対して Store を流し込めば完了です。ここで使用する `Counter` コンポーネントは `containers` ディレクトリーの下にあるものです。`/App.tsx` を `/src/App.tsx` に移し、編集します。

```typescript
// /src/App.tsx
import React from "react";
import { Provider } from "react-redux";

import Counter from "./containers/Counter";
import store from "./store";

export default function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

`/index.js` を次のように編集します。

```typescript
import { AppRegistry } from "react-native";
import App from "./src/App";
import { name as appName } from "./app.json";

AppRegistry.registerComponent(appName, () => App);
```

![カウンターアプリ](../images/09/counter-app.png)

:::details コラム: useSelector、useDispatch と connect の比較
`useSelector` と `useDispatch` が提供されたのは比較的最近です。以前は `connect` 関数を使用していました。次はカウンターアプリでの `connect` の使用例です。

```typescript:/src/containers/counter.ts
// /src/containers/counter.ts
import { connect } from 'react-redux'
import { Dispatch } from 'redux'

import { State } from '../reducers/counter'
import { increment, decrement } from '../actions/counter'
import Counter from '../components/Counter'

const mapStateToProps = (state: State) => ({
  count: state.current,
})

const mapDispatchToProps = (dispatch: Dispatch) => ({
  actions: {
    increment() {
      dispatch(increment())
    },
    decrement() {
      dispatch(decrement())
    },
  },
})

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```

`mapStateToProps` は Store の中の State を React の props にマッピングするための関数定義です。`state` を引数とし、props として受け取りたいプレインオブジェクトを返す関数を定義するものです。

対して `mapDispatchToProps` は Action Creator を React props にマッピングするための関数です。引数として渡ってくる `dispatch` を用いて Action を適宜 `dispatch` する関数群を書く場所です。

`useSelector` と `useDispatch` を使う場合に比べ、処理内容が想像しにくい印象を受けますね。理由がない限り、`connect` の使用は避けるとよいでしょう。
:::

### Redux におけるディレクトリー構造

Redux は公式サイトのサンプルにおいて、`/src` ディレクトリー直下を次のように分けています。

| 名前       | 用途                                    |
| ---------- | --------------------------------------- |
| actions    | Action Creator                          |
| components | React コンポーネント                    |
| constants  | 定数定義                                |
| containers | React コンポーネントを connect したもの |
| reducers   | Reducer                                 |

それぞれの役割が明確な分、ディレクトリー数が多くなっています。

### Ducks

Redux 公式のディレクトリー構成をそのまま真似すると多くの同名ファイルが作成されることになります。たとえばカウンターアプリを実現する場合、次のファイルツリーとなるでしょう。

```
/
  src/
    App.tsx
    store.ts
    actions/
      counter.ts
    components/
      Counter.tsx
    constants/
      counter.ts
    containers/
      Counter.tsx
    reducers/
      counter.ts
```

ところが Action Type、Action Creator、Reducer は 3 つ揃うことではじめて意味のあるものとなります。これを 1 つのファイルにまとめてしまうことで見通しよく記述できる、Ducks という書き方が提案されています。[^6]

Ducks には 4 つのルールがあります。

1. Reducer をデフォルトエクスポートする
2. Action Creator を名前付きエクスポートする
3. Action Type は「npm モジュールもしくはアプリ名/reducer 名/action type 名」という形式とする
4. 同一ファイル外の Reducer やライブラリーなどが使用できるように Action Type を大文字のスネークケースとしてエクスポートしてよい

これに則ってカウンターアプリを実装してみると次のようになります。

```typescript
export function createInitialState() {
  return {
    count: 0,
  };
}
export type State = ReturnType<typeof createInitialState>;

// Action Types
// 3つ目と4つ目のルール
export const INCREMENT = "myapp/counter/increment";
export const DECREMENT = "myapp/counter/decrement";

// 2つ目のルール
export function increment() {
  return {
    type: INCREMENT,
  };
}

export function decrement() {
  return {
    type: DECREMENT,
  };
}

type Action = ReturnType<typeof increment> | ReturnType<typeof decrement>;

// Reducer
// 1つ目のルール
export default function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    case INCREMENT:
      return {
        ...state,
        count: state.count + 1,
      };
    case DECREMENT:
      return {
        ...state,
        count: state.count - 1,
      };
    default:
      return state;
  }
}
```

Action と Reducer が同一ファイルにあるので見通しがよくなっています。開発効率を重視して、本書では Ducks を使って実装します。この時点のディレクトリー構成は次になります。

```
/
  src/
    components/ # Reactコンポーネントを格納する
    containers/ # ReactコンポーネントとRedux Storeを接続するファイルを格納する
    modules/    # Ducks moduleファイルを格納する
```

[^6]: https://github.com/erikras/ducks-modular-redux

## Redux における非同期処理

カウンターの実装を通して単純なデータの変更を学習しました。では実践的な例として、ネットワーク通信の結果を Redux Store に格納する処理を書いてみましょう。

Reducer は純粋関数でなければなりません。また、Action Creator は同期的に Action を返却する関数でなければなりません。つまり、これまでに登場した概念では非同期処理を書く場所はありません。少し書き方を考えてみましょう。

やりたいことはネットワーク通信の結果を Action に載せ、 `dispatch` することです。素直に書き下すと次のようになるでしょう。ユーザー一覧を API 経由で取得する想定です。

```typescript
// /src/modules/users.ts
import { Dispatch } from "redux";

interface User {
  id: number;
  name: string;
}

export function createInitialState(): User[] {
  return [];
}

const SET_USERS = "users/set" as const;

export function setUsers(users: User[]) {
  return {
    type: SET_USERS,
    payload: {
      users,
    },
  };
}

export async function retrieveUsers(dispatch: Dispatch) {
  const response = await fetch("https://example.com/api/users");
  const users = await response.json();
  dispatch(setUsers(users));
}

type Action = ReturnType<typeof setUsers>;

export default function reducer(state = createInitialState(), action: Action) {
  switch (action.type) {
    case SET_USERS:
      return action.payload.users;
    default:
      return state;
  }
}
```

ネットワーク通信のレスポンスが返ってくるまで待つために `retrieveUsers` 関数には `async` 、 `await` が必要となります。Action の生成についてはきちんと Action Creator を定義し移譲しています。

`dispatch` を引数にとっている点は変則的です。`dispatch` は Store のインスタンスメソッドのように見えますが、実はただの関数です。今回のように引数に渡すことが可能です。

この関数を使用する際は次のように props にマップします。

```typescript
import React from "react";
import { useSelector, useDispatch } from "react-redux";

import Component from "../components/Component";
import { retrieveUsers } from "../modules/users";

export default function ConnectedComponent() {
  const dispatch = useDispatch();
  const actions = React.useMemo(
    () => ({
      retrieveUsers() {
        retrieveUsers(dispatch);
      },
    }),
    [dispatch]
  );

  return <Component actions={actions} />;
}
```

React コンポーネントからの呼び出しは次となります。

```typescript
interface Props {
  actions: {
    retrieveUsers: () => void;
  };
}

export default function App(props: Props) {
  return (
    <View style={styles.container}>
      <Button title="get users" onPress={props.actions.retrieveUsers} />
    </View>
  );
}
```

問題なく非同期処理が実現できていますが、`dispatch` の取り扱いが変則的です。また、`retrieveUsers` 関数は Redux のどの概念にも当てはまらないものです。この不整合を解消するため、Redux では middleware という仕組みを用います。

### Redux middleware

Redux における middleware は、一般的なミドルウェアと同様のものです。ミドルウェアとは、OS とアプリケーションの間で特定の目的を実現するソフトウェアを指します。アプリケーション構築において必須もしくはあるとうれしい機能を取り扱いやすい粒度で実現することが多いです。身近なものとしては Web サーバーや DBMS などが挙げられます。 OS が提供する機能を用いて、Web サーバーでは HTTP 通信のためのインターフェイスを、DBMS では SQL を用いたデータ操作インターフェイスをそれぞれ提供します。

Redux では非同期処理を middleware によって実現します。

#### Redux middleware の仕組み

Redux middleware は任意の Action を `dispatch` した後、その Action が Reducer によって処理される前に任意の処理を実行する仕組みです。

middleware 実装にあたっては次の作法を守る必要があります。

1. `store` を引数とする関数内で `next` を引数とする関数を返す
2. `next` を引数とする関数内で `action` を引数とする関数を返す
3. 2 の関数の中で自らの責務を果たしたら引数を `action` として `next` を実行し、その結果を `return` する

次がこれらの条件を満たす最低限のコードです。

```typescript
import { Middleware } from "redux";

export const nop: Middleware = (_store) => (next) => (action) => {
  return next(action);
};
```

#### middleware を実装する

非同期処理を可能とする middleware を作成してみましょう。非同期処理の結果を `dispatch` できるように関数を Action として渡せるようにするものです。

```typescript
// /src/middlewares.ts
import { Middleware } from "redux";

export const functionExecutor: Middleware = (store) => (next) => (action) => {
  if (typeof action === "function") {
    return action(store.dispatch, store.getState);
  }
  return next(action);
};
```

Action が関数の場合に `dispatch` と `getState` を渡して実行しているだけです。これを設定するために `applyMiddleware` という関数を使用します。Store 作成時に middleware を設定する形です。

```typescript
// /src/store.ts
import { createStore, applyMiddleware } from "redux";

import appReducer from "./modules";
import { functionExecutor } from "./middlewares";

const store = createStore(appReducer, applyMiddleware(functionExecutor));
```

これで関数を `dispatch` 可能となりました。次の例はこの middleware を用いて、先ほどのユーザー情報を取得するコードを書き換えたものです。

```typescript
async function retrieveUsers() {
  return function (dispatch: Dispatch) {
    try {
      const response = await fetch("https://example.com/api/todo/");
      const todos = response.json();
      dispatch(setUsers(todos));
    } catch (e) {
      dispatch(setError(e));
    }
  };
}

dispatch(retrieveUsers());
```

非同期処理を Redux の枠組み内で処理できるようになりました。

:::details コラム: 便利な middleware
非同期処理を実現する middleware の他に、アプリケーションを構築するにあたってあると嬉しいものは何でしょうか？構築するアプリケーションによって異なりますが、どのアプリケーションにおいても有用なものは次になるでしょう。

1. Action 適用前後の Store の状態のロギング
2. クラッシュレポート

1 番は開発する上で Reducer の変遷を確認できますので、Reducer の想定外の挙動を発見しやすくなります。

2 番のクラッシュレポートとは、ユーザーの端末や使用状況による不具合情報を収集するための仕組みを言います。モバイルアプリケーション開発においてはこれらの情報がないと問題を再現できず、不具合を修正できないため、クラッシュレポートを活用することは顧客満足のために非常に重要となります。

#### Action 適用前後の Store をロギングする middleware

```typescript
// /src/middlewares.ts
import { Middleware } from "redux";

export const logger: Middleware = (store) => (next) => (action) => {
  console.log(action);
  console.log(store.getState());
  const result = next(action);
  console.log(store.getState());
  return result;
};
```

`next` 関数による Reducer 処理の前後に `store.getState()` の実行結果を出力するコードを挟んでいるだけです。

#### クラッシュレポートする middleware

```typescript
// /src/middlewares.ts
import { Middleware } from "redux";
import CrashReport from "crash-report";

export const crashReporter = (store) => (next) => (action) => {
  try {
    return next(action);
  } catch (e) {
    console.log(e);
    CrashReport.send(e);
    throw e;
  }
};
```

`crash-report` パッケージはクラッシュレポートサービスから提供されているものという前提のコードです。

#### 複数の middleware を同時に指定する

```typescript
import { createStore, applyMiddleware } from "redux";

import appReducer from "./modules";
import { logger, crashReporter } from "./middlewares";

const store = createStore(appReducer, applyMiddleware(logger, crashReporter));
```

`applyMiddleware` 関数の引数の指定順序に注意してください。先に指定したものが先に実行されます。この例では次のいずれかにおいてエラーが発生した場合、クラッシュレポートが送信されます。

- Reducer による処理
- crashReporter 以外の middleware による処理

逆の指定とした場合、もし `logger` でエラーがおきてもクラッシュレポートされなくなってしまいます。気をつけましょう。
:::

#### redux-thunk とは

非同期処理のために関数を Action として渡せるようにする middleware は、redux-thunk というパッケージとして提供されています。thunk とは関数に別の関数を注入することを指すプログラミングテクニックのことです。[^8]毎回自分で middleware を実装するのは大変ですので、redux-thunk を使いましょう。

[^8]: https://github.com/reduxjs/redux-thunk

##### redux-thunk のセットアップ

まずインストールしましょう。

```console
yarn add --exact redux-thunk
```

続いて Store に設定します。

```typescript
// /src/store.ts
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";

import appReducer, { createInitialState } from "./modules/async";

export default createStore(
  appReducer,
  createInitialState(),
  applyMiddleware(thunk)
);
```

これでセットアップは完了です。

#### Usecase パターン

関数を `dispatch` することが可能となりました。改めてこれがどういう意味を持つか考えてみましょう。

関数には Action の `dispatch` を含めた、任意の処理を記述できます。単発の `dispatch` ではなく、一連の流れを踏まえた `dispatch` が可能になったことが最大のメリットです。

また、Redux から React へマッピングされた関数は主にユーザーによって駆動されるか、時間など特定の条件下で駆動されるかの 2 つに大別されます。こういった何らかの主体によって駆動される一連の処理は、一般的にユースケースと呼ばれます。ユースケースというと UML によって規定されている言葉が有名ですが、ここでは単純にシステムとそれ以外の何かとのインタラクションをステップとして記述したもの、という定義とします。[^10]

Praiser における「Todo 一覧を取得する」というユースケースの処理は次のようになります。

1. サーバーに Todo 一覧取得をリクエストする
2. サーバーから返却された Todo 一覧を payload に持つ Action を `dispatch` する

以上から、Redux Store へ関数を `dispatch` する場合、ユースケースの粒度とするのが良さそうです。これを本書では Usecase パターンと名前をつけ使っていきます。ユースケースを記述するディレクトリーを `/src` 直下に配置し、次の構成としましょう。

```
/
  src/
    components/ # Reactコンポーネントを格納する
    containers/ # ReactコンポーネントとRedux Storeを接続するファイルを格納する
    modules/    # Ducks moduleファイルを格納する
    usecases/   # ユースケースを格納する
```

[^10]: https://en.wikipedia.org/wiki/Use_case

### Redux における Store 設計

ここでどういったデータを Store に持てばよいかを考えてみましょう。

メンテナンス性を考慮すると、管理対象のデータは少ないほうがよいはずです。すると必要最低限のデータはなにかという問題になります。

Redux Store のデータを使用するのはユースケースと React コンポーネントです。ユースケースについては上述したので、ここでは React コンポーネントについて考えましょう。

最終的に React コンポーネントで必要となるデータはおおまかに 3 つに分けられます。

1. アプリケーションが対象とする問題領域を表現するデータ
2. 表示用のデータ
3. アプリケーションが動作する環境や状況の文脈データ

1 は非常に本質的なデータです。アプリケーションは通常、何らかの課題解決のために作成されますが、その解法を定式化したものになります。Todo アプリでは Todo 管理、動画共有アプリでは動画共有という課題への解法を表します。

対して 2 はアプリの使いやすさを向上するために必要となるデータを指します。1 から導出可能なデータ群で、完了済み Todo の個数や動画の視聴数ランキングなどがあたります。それぞれ、すべての Todo データや動画の視聴履歴から計算可能なデータです。

3 はユーザーごとにカスタマイズ可能な設定情報やアプリケーションのモードなどです。ユーザーの認証情報や使用言語、アプリケーションの外観テーマ、A/B テストの設定やキャンペーンなどが該当します。アプリケーションにおけるグローバル情報としてどの機能からもアクセスしたいものになります。

#### アプリケーションが対象とする問題領域を表現する

アプリケーションを有用なものとするためには、そのアプリケーションが解決しようとしている課題を適切に捉えることが必須です。しかし、これは非常に難しいことです。仮説検証を繰り返し、もっともらしい課題から順に解いて探し当てるステップが必要となります。

仮説検証を繰り返すための手法のひとつとして、問題領域を表現し、深化させていく DDD（Domain Driven Design、ドメイン駆動設計）というアプローチがあります。DDD では対象とする問題領域をドメインと呼び、実行可能なプログラミング言語によって書き表していくことでドメインをモデリングするというアプローチを取ります。[^12]

本書では Redux Store の設計に DDD を適用しますが、一点だけ変更を加えます。オブジェクト指向の文脈で語られることが多いのですが、クラスを設計していくアプローチではなく、Redux に適用しやすいようにプレインオブジェクトと純粋関数を用いて設計していくアプローチを取ります。例として次のコードに示す、クラスとオブジェクトおよび関数による表現は全く同じものです。

```typescript
// クラスによる表現
class Todo {
  title: string;
  isCompleted: boolean = false;
  createdAt: number = Date.now();

  constructor(title: string) {
    this.title = title;
  }

  complete() {
    this.isCompleted = true;
  }

  uncomplete() {
    this.isCompleted = false;
  }
}

// オブジェクトおよび関数による表現
interface Model {
  readonly title: string;
  readonly isCompleted: boolean;
  readonly createdAt: number;
}

function factory(title: string): Model {
  return {
    title,
    isCompleted: false,
    createdAt: Date.now(),
  };
}

function complete(todo: Model): Model {
  return {
    ...todo,
    isCompleted: true,
  };
}

function uncomplete(todo: Model): Model {
  return {
    ...todo,
    isCompleted: false,
  };
}
```

`factory` はデザインパターンで紹介されている Factory パターンです。クラスインスタンスの構築手順が複雑な際、それを関数にパッキングする手法です。この例は特に複雑な構築手順ではないのですが、様々なプレインオブジェクトの構築手段を標準化するためこのようにしています。

`factory` によって構築されるプレインオブジェクトの型を `Model` と定義しています。この型名はドメインモデルからとっています。

また、ドメインモデルの振る舞いとして定義した `complete` と `uncomplete` はそれぞれ、第一引数にプレインオブジェクトを受け取り新しいプレインオブジェクトを返しています。これはドメインモデリングに純粋関数のみを使うという制約を設けているためです。`Model` 型のすべてのプロパティに `readonly` をつけていますが、直接ドメインモデルを変更するような次のコードに対し、エラーを出力するためです。

```typescript
function complete(todo: Model): void {
  todo.isCompleted = true; // エラー！
}
```

[^12]: https://domainlanguage.com/ddd/

#### ドメインモデリング

ドメインモデリングの例として飲料の自動販売機を取り上げます。まず飲料のドメインモデルは簡単に次のようにしてみました。名前、価格、在庫の他に一意に識別するために ID を追加しています。

```console
yarn add --exact uuid react-native-get-random-values
cd ios/
pod install --repo-update
```

```typescript
// /src/domain/drink.ts
import "react-native-get-random-values";
import { v4 as generateUuid } from "uuid";

export interface Model {
  readonly id: string;
  readonly name: string;
  readonly price: number;
  readonly remains: number;
}

export function factory(name: string, price: number, remains: number): Model {
  return {
    id: generateUuid(),
    name,
    price,
    remains,
  };
}
```

ID の生成に UUID v4 を用いていますが、その他のプロパティは `factory` に指定された値をそのまま保持するのみです。

このドメインモデルの永続化を責務とするコードは次になります。

```typescript
// /src/domain/drink-repository.ts
import * as Drink from "./drink";

const master: Readonly<Array<Drink.Model>> = [
  Drink.factory("water", 100, 30),
  Drink.factory("orange juice", 120, 30),
  Drink.factory("coffee", 120, 30),
];

export function getAll() {
  return master;
}
```

インターフェイスとして `getAll` を定義しています。今回は自動販売機のマスターデータを定義し、そのまま返すコードとしています。実際はデータベースなどと接続し、データを取得するコードとなるでしょう。書き込みや上書きのためのインターフェイスなどは必要となった際に定義していきましょう。これは Repository パターンとして DDD で紹介されているモデリングパターンの一種です。

飲料の次は自動販売機をモデリングしましょう。非常に簡単ですが「お金を投入する」ことと「飲料を販売する」ことをモデルとして表現しています。

```typescript
// /src/domain/vending-machine.ts
import assert from "../lib/assert";
import assertIsDefined from "../lib/assert-is-defined";
import * as Drink from "./drink";
import * as DrinkRepository from "./drink-repository";

export interface Model {
  inventory: Readonly<Array<Drink.Model>>;
  chargedMoney: number;
}

export function factory(): Model {
  return {
    inventory: DrinkRepository.getAll(),
    chargedMoney: 0,
  };
}

export function charge(vendingMachine: Model, money: number): Model {
  return {
    ...vendingMachine,
    chargedMoney: vendingMachine.chargedMoney + money,
  };
}

export function sell(vendingMachine: Model, id: string): Model {
  const selected = vendingMachine.inventory.find((item) => item.id === id);
  assertIsDefined(selected);
  assert(selected.remains <= 0, `Selected drink is sold out: ${selected.name}`);
  assert(
    vendingMachine.chargedMoney < selected.price,
    `Too few charged money for selected drink: ${vendingMachine.chargedMoney} < ${selected.price}`
  );

  return {
    ...vendingMachine,
    inventory: vendingMachine.inventory.map((item) => {
      if (item === selected) {
        return {
          ...selected,
          remains: selected.remains - 1,
        };
      }
      return item;
    }),
    chargedMoney: vendingMachine.chargedMoney - selected.price,
  };
}
```

ここまでのコードは React や Redux とは独立したコードとなっていますが、アプリケーションの本質である「お金を投入する」ことや「飲料を売る」ことを表現できていますね。ドメイン層のコードを読めばアプリが何を目的としているか、どのように実現しようとしているかは一目瞭然です。

実世界でこのモデルを使うには、お金やお釣りのモデリングが足りないでしょう。しかし、モデルを育てる土台が整っているため、必要なものを追加していく作業はそれほど困難とはならないでしょう。徐々に改良していけばよいのです。

#### Redux にドメインモデルを組み込む

ドメインモデルを Redux で操作する際は次のようにします。Redux の文脈にのるように Action types、Reducer、Action Creators を定義しているだけです。

```typescript
// /src/modules/vending-machine.ts
import * as VendingMachine from "../domain/vending-machine";

export function createInitialState() {
  return VendingMachine.factory();
}

export const CHARGE = "vending-machine/charge" as const;
export const SELL = "vending-machine/sell" as const;

export function charge(chargedMoney: number) {
  return {
    type: CHARGE,
    payload: {
      chargedMoney,
    },
  };
}

export function sell(id: string) {
  return {
    type: SELL,
    payload: {
      id,
    },
  };
}

export type Action = ReturnType<typeof charge> | ReturnType<typeof sell>;

export default (state = createInitialState(), action: Action) => {
  switch (action.type) {
    case CHARGE:
      return VendingMachine.charge(state, action.payload.chargedMoney);
    case SELL:
      return VendingMachine.sell(state, action.payload.id);
    default:
      return state;
  }
};
```

Redux の作法に則ってドメイン層のコードを呼び出しているだけですね。

### 表示用データの生成

さて、たとえば Todo アプリにおいていままでに完了したタスクとその総数を表示したい場合はどうすればよいでしょうか？これまでの知識を用いると、React コンポーネントで計算すればよいでしょう。

```typescript
// /src/components/Component.tsx
import React from "react";
import { FlatList, Text } from "react-native";

interface Todo {
  title: string;
  isCompleted: boolean;
}

interface Props {
  todos: Array<Todo>;
}

export default function Component(props: Props) {
  const completedTodos = React.useMemo(
    () => props.todos.filter((todo) => todo.isCompleted),
    [props.todos]
  );
  const numofCompleted = React.useMemo(
    () => completedTodos.length,
    [completedTodos]
  );

  return (
    <>
      <Text>{numofCompleted}</Text>;
      <FlatList
        data={completedTodos}
        renderItem={({ item }) => <Text>{item.title}</Text>}
      />
    </>
  );
}
```

このコードは正しく動作するのですが、他のコンポーネントでも使いたい場合にこのロジックを切り出す必要があります。その際の切り出し先なのですが、この計算は本来ドメイン層に存在すべきロジックではないでしょうか？早速ドメイン層に切り出してみます。

```typescript
// /src/domain/todos.ts
export function getCompletedTodos(todos: Model) {
  return todos.filter((todo) => todo.isCompleted);
}

export function getNumofCompleted(todos: Model) {
  return getCompletedAll(todos).length;
}
```

このロジックを React コンポーネントで呼び出せば必要な場所で計算結果を使用できます。ただ、React コンポーネントの責務は props および state から JSX を組み立てることです。できる限り簡素にし、計算などさせないようにしたほうがよいでしょう。

Redux の世界と React の世界をつなぐ際、`useSelector` という Redux Store から `props` への写像がありました。ここで計算させてみましょう。

```typescript
// /src/containers/Component.tsx
import React from 'react'
import { useSelector } from 'react-redux'

import Component from '../components/Component'
import * as Todos from '../domain/todos'

export function ConnectedComponent() {
  const completedTodos = useSelector(Todos.getCompletedTodos)
  const numofCompleted = useSelector(Todos.getNumofCompleted);

  return <Component numofCompleted={numofCompleted}>;
}
```

先ほどの React コンポーネントは次のようになります。

```typescript
// /src/components/Component.tsx
import React from "react";
import { FlatList, Text } from "react-native";

interface Todo {
  title: string;
  isCompleted: boolean;
}

interface Props {
  completedTodos: Array<Todo>;
  numofCompleted: number;
}

export default function Component(props: Props) {
  return (
    <>
      <Text>{props.numofCompleted}</Text>
      <FlatList
        data={completedTodos}
        renderItem={({ item }) => <Text>{item.title}</Text>}
      />
    </>
  );
}
```

非常にシンプルになりました。各層の責務もはっきりしましたし、これで良さそうです。

ところが、このコードはパフォーマンス上の問題点があります。`useSelector` はプリミティブ値については計算結果をキャッシュし、変更がなければ前回の結果を返却してくれるのですが、配列やオブジェクトについては対象外です。

このままでは Store に関するどのデータが変更されても `completedTodos` と `numofCompleted` の再計算が走ってしまいます。無駄な計算が増え、アプリケーションが重くなってしまう原因となります。

これを防ぐために、配列やオブジェクトについても計算結果をキャッシュするように変更しましょう。すべての計算結果をキャッシュしていくとメモリーの使用量が多くなってしまい、OS によってアプリをシャットダウンされてしまいます。`useSelector` にならい、直前の計算結果のみをキャッシュしておき次回の呼び出し時の引数が全く同一のものであれば直前の計算結果を返す、という仕組みとします。この仕組みを `cacheOnce` 関数として実装したものを使って書き直すと次のようになるでしょう。

```typescript
// /src/containers/Component.tsx
import React from "react";
import { useSelector } from "react-redux";

import Component from "../components/Component";
import * as Todos from "../domain/todos";
import { cacheOnce } from "../lib";

export function ConnectedComponent() {
  const getCompletedTodos = cacheOnce(
    (state) => [state.todos],
    (todos) => Todos.getCompletedTodos(todos)
  );
  const getNumofCompleted = cacheOnce(
    (state) => getCompletedTodos(state),
    (complatedTodos) => Todos.getNumofCompleted(completedTodos)
  );

  const completedTodos = useSelector(getCompletedTodos);
  const numofCompleted = useSelector(getNumofCompleted);

  return (
    <Component
      completedTodos={completedTodos}
      numofCompleted={numofCompleted}
    />
  );
}
```

`cacheOnce` 関数の第 1 引数に指定している関数は、Redux Store から計算に必要なプロパティ群を抽出するためのものです。第 1 引数の評価結果はそのまま第 2 引数の関数の引数として渡されます。`cacheOnce` 関数は第 1 引数の評価結果と第 2 引数の評価結果の組を覚えておき、第 1 引数の評価結果が直前の呼び出しから変化していた場合、第 2 引数の関数を実行します。それ以外の場合はキャッシュしていた値を返します。この挙動は次のコードで実現可能です。

```typescript
// /src/lib/cache-once.ts
import assert from "./assert";

function equals<T>(a: Array<T>, b: Array<T>) {
  assert(a.length === b.length);

  for (let i = 0; i < a.length; ++i) {
    if (a[i] !== b[i]) {
      return false;
    }
  }

  return true;
}

const table = new Map();

interface Arg {
  [key: string]: any;
}
type Memoizee = (...args: Array<any>) => any;
type Memoized = (arg: Arg) => any;
type Selector = (arg: Arg) => Array<any>;

export default (selector: Selector, memoizee: Memoizee): Memoized =>
  (arg: Arg) => {
    if (!table.has(memoizee)) {
      table.set(memoizee, { args: null, result: null });
    }
    const cache = table.get(memoizee);

    const args = selector(arg);
    if (equals(cache.args, args)) {
      return cache.result;
    }

    cache.args = args.slice();
    cache.result = memoizee.apply(null, args);
    return cache.result;
  };
```

直前の呼び出しにおける引数と今回の呼び出しにおける引数が全く同一というチェックは `equals` 関数で実現しています。この中で JavaScript の演算子 `!==` を使用していることがポイントです。これは数値や文字列などプリミティブ値であれば値同士を比較しますが、Object のインスタンスなどにおいてはその参照が異なるかどうかをチェックします。次はこの演算子の動作の一例です。[^11]

```typescript
1 !== 1;  // false
1 !== 2;  // true
'React Native' !== 'React Native';  // false

{name: 'React Native'} !== {name: 'React Native'};  // true: 参照が異なる

const obj = {name: 'React Native'};
obj !== obj;  // false: 参照が等しい
```

このチェック方法は Redux Store において State が変更されたかどうかをチェックする方法と同一です。Reducer によって State の一部が新しい参照となっている場合、`cacheOnce` 関数は引数の参照が異なっていないかをチェックして再計算の是非を判定します。Redux と親和性が高い仕組みです。

注意点として、この仕組みを適用するために、対象の関数は純粋関数である必要があります。非純粋関数の場合、引数が同じでも同じ値を返すとは限らない可能性があるためです。

[^11]: https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Identity

#### reselect

さて、やりたいことは問題なくできましたが、毎回 `cacheOnce` 関数を定義するのは面倒かつミスのもとです。npm にはすでに同様のことを実現する reselect パッケージがあります。活用しましょう。

```console
yarn add --exact reselect
```

続いて、State から表示用データへ計算する関数を定義しましょう。

```typescript
// /src/selectors/todos.ts
import { createSelector } from "reselect";

import { AppState } from "../modules";
import * as Todos from "../domain/todos";

function getTodos(state: AppState) {
  return state.todos;
}

export const getCompletedTodos = createSelector([getTodos], (todos) =>
  Todos.getCompletedTodos(todos)
);

export const getNumofCompleted = createSelector(
  [getCompletedTodos],
  (completedTodos) => Todos.getNumofCompleted(completedTodos)
);
```

`selectors` ディレクトリーには Redux Store から表示用の各種データへの計算ロジックを保存しましょう。

これを `useSelector` で使用します。

```typescript
// /src/containers/Component.tsx
import React from "react";
import { useSelector } from "react-redux";

import Component from "../components/Component";
import * as Todos from "../selectors/todos.ts";

export function ConnectedComponent() {
  const completedTodos = useSelector(Todos.getCompletedTodos);
  const numofCompleted = useSelector(Todos.getNumofCompleted);

  return (
    <Component
      completedTodos={completedTodos}
      numofCompleted={numofCompleted}
    />
  );
}
```

### アプリケーションが動作する環境や状況の文脈データ

ユーザーごとのカスタマイズは文脈データに該当するわかりやすい例でしょう。表示言語や見た目の設定など、アプリケーションを使用する前提となる設定が具体例です。これらは UI に関わってくるため、React の Context API で読み書き可能とすることが自然です。

```typescript
// /src/contexts/user-settings.ts
import React from "react";

export interface UserSettings {
  locale: string;
  theme: string;
}

export function createInitialContext() {
  return {
    locale: "en-US",
    theme: "light",
  };
}

export default React.createContext<UserSettings>(createInitialContext());
```

作成したユーザー情報の文脈を、ルートコンポーネントで次のように設定します。

```typescript
// /src/App.ts
import React from "react";

import Home from "./components/Home";
import UserSettingsContext, {
  createInitialContext,
} from "./contexts/user-settings";
import { loadUserSettings } from "../lib";

export default function App() {
  const [userSettings, setUserSettings] = React.useState(
    createInitialContext()
  );

  React.useEffect(() => {
    async function load() {
      const userSettings = await loadUserSettings();
      setUserSettings(JSON.parse(userSettings));
    }
    load();
  });

  return (
    <UserSettingsContext.Provider value={userSettings}>
      <Home />
    </UserSettingsContext.Provider>
  );
}
```

各コンポーネントでは `UserSettingsContext` を `import` し、`useContext` で参照可能とするだけです。

```typescript
// /src/components/Home.tsx
import React from "react";
import { StyleSheet, Text, View } from "react-native";

import UserSettingsContext from "../contexts/user-settings";

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
});

export default function Home() {
  const { locale, theme } = React.useContext(UserSettingsContext);
  return (
    <View style={styles.container}>
      <Text>Home</Text>
      <Text>locale: {locale}</Text>
      <Text>theme: {theme}</Text>
    </View>
  );
}
```

このコードは 4 章で紹介した `useState` と Context API の併用パターンをユーザー設定の保持と使用に当てはめたものです。

非常に強力な Context API ですが、表示項目のカスタマイズや特定機能の ON/OFF などを文脈データとして扱う場合は注意が必要です。テスト工数が増加することに加え、何よりもユーザーにアプリケーションに関する習熟を強制することになります。代替案を検討したり、ユーザーインタビューや A/B テストなどで本当に必要かどうかを確認しましょう。

### ここまでのディレクトリー構成

いろいろな概念が出てきたので、最後に `/src` ディレクトリー直下の各ディレクトリーを整理します。

| ディレクトリー | 使用パッケージ | 責務                                       |
| -------------- | -------------- | ------------------------------------------ |
| domain         | なし           | 課題を解決するためのモデルを表現する       |
| modules        | redux          | アプリケーションのデータフローを実現する   |
| usecases       | redux-thunk    | ユースケースを実現する                     |
| selectors      | reselect       | 本質的なデータから表示用のデータを生成する |
| containers     | react-redux    | React の世界と Redux の世界をつなぐ        |
| contexts       | react          | アプリケーションに文脈を与える             |
| components     | react          | UI を提供する                              |
