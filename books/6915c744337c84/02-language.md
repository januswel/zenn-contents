---
title: "TypeScript と ECMAScript 2015 の基本を抑える"
---

## TypeScript とは

TypeScript とは ECMAScript を元に静的型付けを可能としたことが特徴の、 Microsoft が開発した言語です。 React Native は 2018 年 5 月に TypeScript をサポートしました。[^1]

[^1]: https://reactnative.dev/blog/2018/05/07/using-typescript-with-react-native

### 型があることによる 2 つのメリット

TypeScript は既存の言語に型付けを可能としたのみですが、型付けできることによってどういったメリットが生じるのでしょうか。大きくふたつに集約されます。

1. 開発時にエラーがわかる
2. 動くドキュメントとして開発者やツールの助けとなる

#### 開発時にエラーがわかる

次の JavaScript コードは、実行した際にはじめてエラーがわかります。

```javascript
function getPrefecture(profile) {
  return profile.residence.prefecture;
}

console.log(
  getPrefecture({
    residence: {
      city: "新宿区",
    },
  })
);
```

このコードを実行すると次のエラーが出力されます。

```
// TypeError: Cannot read property 'prefecture' of undefined
```

エラーが実行時にしかわからない理由は、JavaScript がインタープリターというしくみを採用した言語だからです。インタープリターは実装時やコンパイル時ではなく、実行時に実際のデータを検査し、存在するべきデータが存在しない場合などにエラーを出力します。これでは実行するまでエラーの有無さえわかりません。できればコードを書いているそのときにこのエラーを検出したいという要求は自然なものでしょう。

では型がある状態、つまり TypeScript で同様の処理を書いてみましょう。

```typescript
interface Profile {
  residence: {
    prefecture: string;
    city: string;
  };
}
function getPrefecture(profile: Profile) {
  return profile.residence.prefecture;
}

console.log(
  getPrefecture({
    residence: {
      city: "新宿区",
    },
  })
);
```

`interface` など TypeScript の書き方についてはこのあと触れるので、ここでは型を付与していることが感じられれば十分です。このコードでは TypeScript によるコンパイル時に `prefecture` というプロパティが存在しないことを指摘されます。

```
// Property 'prefecture' is missing in type '{ city: string; }' but required in type '{ prefecture: string; city: string; }'.
```

実行時からコンパイル時にエラーの検出時期を早めることができました。さらに、エディターもしくは IDE の助けを借りることでコードの編集中にエラーを検出できるようになります。つまり、実装する際にエラーを作り込まないで済むようになります。

型によるエラー検出は非常に有用ですが、すべてに対して有効なわけではありません。ネットワーク通信によってデータを受け取る際などプログラム外とのやりとりが生じる場合、実行時までエラーがあるかわかりません。それ以外のエラーを早めに検出し対処することで、後々のデバッグ工数の短縮に繋がり開発効率が非常に高くなります。

#### 動くドキュメントとして開発者の助けとなる

コードに書かれている型は、そのコードを理解するためのヒントとなります。

```typescript
function getAll(): Array<number> {
  // （中略）
}
```

関数名は `getAll` としか書かれておらず処理内容もわからない状態です。引数には何も指定されておらず、戻り値の型として `Array<number>` と書かれています。これは引数に何も受け取らず、数値の配列を返す関数の定義です。これらの情報から次のように使うことが可能だとわかります。

```typescript
getAll().map((n) => n.toPrecision(3));
```

`map` は配列が持っているインスタンスメソッド、 `toPrecision` は数値が持っているインスタンスメソッドです。

同様のコードで型情報がない場合を見てみましょう。引数に何も指定できないことはわかりますが、戻り値の型がわからないため、どのように使ってよいか推測できません。

```typescript
function getAll() {
  // （中略）
}
```

こういった型による情報伝達は開発者同士のコミュニケーションにも役立ちます。たとえば上述した `getAll` 関数を定義する方とそれを使う方が異なる場合、型による情報はどう使われたいかを適切に伝えてくれます。裏を返せば、使う側は関数を適切に呼び出さなければならない責務を持つわけです。これはチーム内など近しい状況を持つ開発者同士についても、OSS のようにまったく異なる状況の開発者同士についても、同様にコミュニケーション可能なドキュメントとして働きます。

### TypeScript の構文

本書を読むために必要となる TypeScript の構文を紹介します。

それぞれの構文を試すには TypeScript Playground を利用するとよいでしょう。型チェックと JavaScript コードへのコンパイル結果を確認しながら TypeScript コードを書けるツールです。TypeScript の公式サイトで利用できます。[^2]

[^2]: http://www.typescriptlang.org/play/

#### 明示的な型付け

TypeScript では開発者が明示的に型付けしていくことが可能です。

変数名に続いてコロン `:` と型を書いていくことで変数の型を指定可能です。

```typescript
let a: number;
a = 42;
a = "42"; // エラー！
```

変数 `a` は `number` 型だと明示しているコードです。 `number` 型である `42` は代入可能ですが、 `string` 型である `'42'` を代入できません。

```typescript
function double(n: number): number {
  return 2 * n;
}

function errorFunc(n: number): number {
  return n.toString(); // エラー！
}
```

関数の引数は変数の型付けと同様の方法で型を指定可能です。関数の戻り値の型を明示する場合は、引数を取り囲むカッコの直後にコロン `:` と型を書きましょう。

関数 `double` および関数 `errorFunc` の戻り値を `number` 型としています。関数 `double` ではその宣言どおり `number` 型を返していますが、関数 `errorFunc` では `string` 型を返しているためエラーとなります。

#### 暗黙的な型付け

すべての型を明示的に指定するのは大変な作業です。 TypeScript ではコードから型を推測し、最も妥当な型を変数や関数に付与してくれます。

```typescript
let a = 42;
a = "42"; // エラー！
```

この例では変数 `a` が `42` という数値、すなわち `number` 型で初期化されています。この時点で `a` は `number` 型に確定となり、続く `string` 型 `'42'` の代入は型が異なるためエラーとなります。

```typescript
function double(n: number) {
  return 2 * n;
}

let a = double(1);
// a = 2
a = true; // エラー！
```

こちらは関数の戻り値の型を指定していないコードです。この場合、関数 `double` の戻り値は関数内のコードから `number` 型と推測されます。 `double` を使用した結果を受ける変数 `a` についても `number` 型として確定され、 `number` 型以外の代入はエラーとなります。

このように、TypeScript は型を推測しながらエラーがないかをチェックします。すべての型を開発者が明示する必要はありません。こういった手軽さが TypeScript をはじめやすい点の 1 つです。

#### 基礎的な型

標準で使える型は次の表のとおりです。

| 型        | 意味                         |
| --------- | ---------------------------- |
| boolean   | 真偽値                       |
| number    | 数値                         |
| string    | 文字列                       |
| any       | 型チェックしないことを示す   |
| null      | null                         |
| undefined | undefined                    |
| void      | 他のどの型でもないことを示す |
| Array     | 配列                         |
| Tuple     | 組み合わせ                   |
| Object    | オブジェクト                 |

##### boolean

`true` もしくは `false` のどちらかをとる型です。 `if` 文の条件式や三項演算子の 1 つ目の式の評価結果は `boolean` となります。

##### number

JavaScript が扱える数値を表す型です。10 進数だけでなく 2 進数、8 進数、16 進数を取り扱うことが可能です。 `6.63e-34` など指数表記を用いた小数や、 `NaN` 、 `Infinity` も `number` 型となります。

##### string

ダブルクォーテーション `"` やシングルクォーテーション `'` で囲った文字列は `string` 型となります。後述する Template Literals も `string` 型と解釈されます。

##### any

`any` は型チェックを必要としない、という意思表示となります。

```typescript
function double(n: any) {
  return 2 * n;
}

var a = double("42"); // ①エラーとならない
```

① で関数 `double` を引数 `'42'` を指定して呼び出していますが、これはエラーとなりません。引数の型チェックをしないよう、 `any` を用いて TypeScript に依頼しているからです。これは実行時にもエラーとならず、 `NaN` が返るコードとなります。このままではデバッグが大変となりそうです。

`any` は型チェックの穴を作り出します。せっかく TypeScript を使って開発効率の向上や安心を得ようとしているのに、 `any` を使用することで台無しになってしまいます。影響が限定される箇所や、熟慮した上でのみ使うようにしましょう。

##### null と undefined

`null` 型と `undefined` 型はそれぞれ JavaScript における `null` と `undefined` に対応します。TypeScript 標準の動作では、これらの型はすべての型に対して代入可能となっています。

```typescript
var a: number = null; // エラーとならない
var b: number = undefined; // エラーとならない
```

しかし、これでは各変数について `null` もしくは `undefined` でないことを確かめてからでないと使えません。そのような冗長なコードを避けるため、オプション `strictNullChecks` をオンにすることでそれぞれの型もしくは `any` を代入可能とできます。

```typescript
var a: number = null; // エラー！
var b: number = undefined; // エラー！
```

公式サイトの記述においても `strictNullChecks` オプションはオンとするようアドバイスされています。具体的な設定方法については後述します。

##### void

`void` は他のどの型でもないことを示す型です。

```typescript
function log(a: any) {
  console.log(a);
}
```

このコードは返り値がありません。コンパイルして型宣言を出力させると次のようになります。

```typescript
function log(a: any): void;
```

返り値の型として `void` が付与されています。何も返ってこないことを明示しているわけです。

##### Array と Tuple

Array とは配列で、長さが特に決まっていないものを指します。また、配列の中身はひとつの型のみしか指定できません。 `Array` 型に続けて中身の型を `<` と `>` で囲んで指定するか、中身の型に続けて `[]` を書くことで配列の型を表すことができます。

```typescript
var array01: Array<number> = [1, 2, 3];
var array02: number[] = [1, 2, 3];
```

対して Tuple は長さを明示する必要がありますが、要素それぞれに異なる型を指定可能です。

```typescript
var tuple: [number, string] = [1, "foo"];
```

##### Object

オブジェクトも型として表すことが可能です。

```typescript
function area(rect: { length: number; width: number }) {
  return rect.length * rect.width;
}

var a = area({ length: 3, width: 4 });
var b = area({ length: 3 }); // エラー！
var c = area({ width: 4 }); // エラー！
var d = area({ length: 3, width: 4, extra: "foo" }); // エラー！
```

引数の型として `{length: number, width: number}` を指定しています。 `number` 型である `length` プロパティと `number` 型である `width` プロパティのみを持つオブジェクトを受け付ける、ということです。コードの最後 3 行に示したように、必要なプロパティが足りない場合、および型定義に存在しないプロパティが存在する場合にエラーとなります。

#### 新たな型を作成する

基礎的な型を用いて新しい型を定義できます。

##### interface

```typescript
interface Rect {
  length: number;
  width: number;
}

function area(rect: Rect) {
  return rect.length * rect.width;
}

var a = area({ length: 3, width: 4 });
var b = area({ length: 3 }); // エラー！
var c = area({ width: 4 }); // エラー！
var d = area({ length: 3, width: 4, extra: "foo" }); // エラー！
```

`number` 型である `length` プロパティと `number` 型である `width` プロパティを持つ、新しい型 `Rect` を定義しています。定義した型は基礎的な型と同じように使用できます。また、定義したプロパティは過不足なく持っていなければなりません。これでは使用しづらい場面があるため、指定が任意であるプロパティを定義したくなるでしょう。

```typescript
interface NetworkOptions {
  timeout?: number;
  numofRetry?: number;
}

var options01: NetworkOptions = {};
var options02: NetworkOptions = {
  timeout: 3000,
};
var options03: NetworkOptions = {
  numofRetry: 3,
};
var options04: NetworkOptions = {
  timeout: 3000,
  numofRetry: 3,
};
var options05: NetworkOptions = {
  timeout: 3000,
  numofRetry: 3,
  foo: 1, // エラー！
};
```

ネットワーク通信の設定用オブジェクトを定義しています。 `timeout` プロパティは通信がタイムアウトするまでの時間を、 `numofRetry` はリトライ回数を指定するためのプロパティという想定です。これらが指定されない場合、それぞれのデフォルト値を用いてネットワーク通信するものとします。

`timeout` プロパティおよび `numofRetry` プロパティそれぞれの末尾に `?` が書かれています。指定が任意であることを指示するための書き方です。 `NetworkOptions` 型を使用する際、 `timeout` プロパティと `numofRetry` プロパティについては任意の指定が可能ですが、型定義に存在しないプロパティの指定はエラーとなります。

関数を引数として取る関数を定義する際は関数の型定義をしたくなるでしょう。

```typescript
interface NumericalMap {
  (n: number): number;
}

function doubleCall(n: number, numericalMap: NumericalMap) {
  return numericalMap(numericalMap(n));
}

function double(n: number) {
  return 2 * n;
}

doubleCall(3, double);

function concat(str: string) {
  return str + str;
}

doubleCall(3, concat); // エラー！
```

`NumericalMap` 型は `number` 型を受け取り `number` 型を返す関数です。 `interface` の中カッコの中に名前を指定せず、引数定義に続いて `:` で返り値の型を指定することで関数の型定義となります。検証用として、合わせて `NumericalMap` 型を引数として受け取る `doubleCall` 関数を定義しました。 `doubleCall` 関数に関数 `double` および関数 `concat` を渡していますが、関数 `concat` では `NumericalMap` 型と定義が合わず、エラーとなっています。

##### Type Aliasing

```typescript
type Id = number;
```

このように型に別名をつけることも可能です。ここでは `Id` 型を `number` 型の別名として定義しています。

自分で型を定義することの利点は、概念に名前をつけることが可能なことです。この例では `number` 型に `Id` という名前をつけました。以後、何らかの ID を表すデータは `Id` 型を付与することで人間に理解しやすいコードとなります。

##### リテラル型

ある特定の数値や文字列も型として使用できます。

```typescript
type One = 1;
type Foo = "foo";

var one: One = 2; // エラー！
var foo: Foo = "bar"; // エラー！
```

`One` 型は `1` 型の別名、 `Foo` 型は `'foo'` 型の別名として定義しました。それぞれ `1` と `'foo'` 以外の値は代入できません。

これをリテラル型と呼びます。次に説明する Union Types と併用することで便利となる型です。

##### Union Types

Union Types は集合論における和集合を定義できます。

```typescript
type Action =
  | {
      type: "increment";
    }
  | {
      type: "add";
      payload: number;
    };

function increaser(state: number, action: Action) {
  switch (action.type) {
    case "increment":
      return state + 1;
    case "add":
      return state + action.payload;
  }
}
```

ここでは操作を表す `Action` を、 `type` プロパティが `'increment'` 型であるものと `'add'` 型であるもののいずれかをとる型として定義しています。片方にのみ `payload` プロパティが定義されていることに注意してください。

関数 `increaser` の内部では `switch` によって処理を分岐しています。このとき TypeScript は Union Types を構成する型のうち、その文脈における型を判別してくれます。たとえば `case 'increment'` の節では `payload` プロパティが存在しないことを検知し、 `action.payload` が存在しないプロパティである旨のエラーを出力してくれます。

##### Generics

```typescript
function double(n: number) {
  return 2 * n;
}
function concat(str: string) {
  return str + str;
}

// 関数 doubleCall を定義して次のように使いたい
doubleCall(2, double);
doubleCall("abc", concat);
```

関数 `double` は `number` 型を受け取り `number` 型を返し、関数 `concat` は `string` 型を受け取り `string` 型を返します。これらの関数を 2 回呼ぶ関数 `doubleCall` をどのように定義すればよいでしょうか？

```typescript
interface SameTypeFunction<T> {
  (src: T): T;
}

function doubleCall<T>(src: T, func: SameTypeFunction<T>) {
  return func(func(src));
}

function double(n: number) {
  return 2 * n;
}
function concat(str: string) {
  return str + str;
}
function toString(n: number) {
  return n.toString();
}

doubleCall<number>(2, double);
// 型推論によって明示的なパラメーター指定が必要なくなっている
doubleCall(2, double);

doubleCall("abc", concat);
doubleCall(2, toString); // エラー！
```

こういった場合に Generics を使います。関数名の直後において角カッコ `<>` で型パラメーター名をくくり、型指定をすべき箇所に型パラメーター名を指定します。 `SameTypeFunction` 型は何らかの型 `T` を受け取り `T` を返す関数を定義しています。 `doubleCall` 関数の第 2 引数の型として `SameTypeFunction` 型を指定していますが、この型パラメーターも Generics によって指定可能としています。

Generics を用いた型を使用する際は通常、角カッコ `<>` にパラメーターを指定する必要があります。ただし、 `doubleCall` 関数では第 1 引数によって `T` 型が推論可能であるため、型パラメーターの指定無しでの呼び出しも可能です。

`number` 型をとり `string` 型を返す `toString` 関数を用いた `doubleCall` 関数の呼び出しはエラーとなることに気をつけてください。 `SameTypeFunction` による制約に違反しているためです。

#### noUnusedParameters オプションとアンダーバー `_`

TypeScript 標準では使用しない引数を定義してもエラーとなりません。

```typescript
function double(n: number, extra: string) {
  return 2 * n;
}
```

`extra` 変数を使用していませんが、特に警告やエラーが表示されません。しかし、これではコードの見通しが悪くなるため、 `noUnusedParameters` オプションを指定することでエラーを出力するよう設定できます。

ただし、引数のチェックをしてほしくない場合が存在します。

```typescript
// エラー！
var evens = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10].filter(function (value, index) {
  return index % 2 === 0;
});
```

JavaScript 標準の `Array.prototype.filter` 関数を利用して、偶数番目の要素だけ抜き出すコードです。これは使われていない引数 `value` があるというエラーが出力されてしまいます。 `Array.prototype.filter` の型定義は次となっています。

```typescript
filter(callbackfn: (value: T, index: number, array: T[]) => unknown, thisArg?: any): T[];
```

最初の引数 `value` は配列の要素ですが、今回のコードではこの引数を使わずともよい処理内容です。こういった場合、仮引数名にアンダーバー `_` を付与することで、型チェックせずともよいことを TypeScript に伝えられます。

```typescript
var evens = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10].filter(function (_value, index) {
  return index % 2 === 0;
});
```

### Optional Chaining

TypeScript 3.7 から使用可能になりました。

```typescript
interface Options {
  actions?: {
    callback: () => void;
  };
}

function area(props: Options) {
  if (props.actions) {
    props.actions.callback();
  }
}
```

`Options` 型ではプロパティ `actions` 型が存在する場合と存在しない場合があります。こういった場合、上述したコードのように `actions` プロパティが存在するか確かめてから `callback` 関数を呼び出すようなコードとしなければなりませんでした。

```typescript
function area(props: Options) {
  props.actions?.callback();
}
```

Optional Chaining は明示的なチェックなしでプロパティの値を参照可能とするものです。 `?.` の左側に指定されているプロパティが定義されていなければ何もしません。

### Assertion Functions

TypeScript 3.7 で追加された機能です。

データが `null` でないことを担保したい場合があります。その場合、 `if` 文を用いて次のように書く必要がありました。

```typescript
interface Item {
  id: number;
  name: string;
}

function getName(items: Array<Item>, id: number) {
  const target = items.find((item) => item.id === id);
  if (target == null) {
    throw new Error("this path must not be reached.");
  }
  return target.name;
}
```

`Array.prototype.find` は `null` が返ってくる可能性があるため、null チェックが必須です。しかし、この書き方は本質的でない null チェック部分が長く、見通しが悪くなってしまいます。この課題は Assertion Functions によって解決可能です。まず事前に次の関数を定義しておきます。

```typescript
function assertIsDefined<T>(value: T): asserts value is NonNullable<T> {
  if (value === undefined || value === null) {
    throw new Error(`Expected 'value' to be defined, but received ${value}`);
  }
}
```

これを次のように使用します。

```typescript
function getName(items: Array<Item>, id: number) {
  const target = items.find((item) => item.id === id);
  assertIsDefined(target);
  return target.name;
}
```

最初のコードに比べ非常にすっきりと書けるようになります。TypeScript は厳密に null チェックをしてくれるのですが、Assertion Functions 導入以前は `if` 文によるチェックが必要でした。より見通しのよいコードとなるよう、活用しましょう。

より一般化した `assert` 関数は次のように定義可能です。データが特定の条件を満たす必要があることを明示する際に使いましょう。

```typescript
function assert(condition: any, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message);
  }
}

function area(length: number, width: number) {
  assert(0 < length);
  assert(0 < width);
  return length * width;
}
```

:::details コラム: 効率的な型付け

関数の戻り値や変数の型は TypeScript による推測に任せ、関数の引数およびどうしても明示的な型付けが必要となる場合にのみ、型を書いていくのがよいでしょう。

関数の戻り値を明示的に型付けしなければならない場合、その関数が複数の責務を持ってしまっている場合が多いです。適切な責務に切り分け、ひと目で処理内容がわかるような大きさの関数とすべきです。

たとえば長方形に関する処理をライブラリーとして提供する場合を考えましょう。

```typescript
export interface Rect {
  width: number;
  length: number;
}

export type Kind = "name" | "area";

export default function getInformation(
  rect: Rect,
  kind: Kind
): string | number {
  if (kind === "name") {
    if (rect.width === rect.length) {
      return "square";
    }
    return "rectangle";
  }
  return rect.width * rect.length;
}
```

このコードでは `kind` 引数の値によって `getInformation` 関数の戻り値の型が変わります。この関数に機能追加をする場合など、コードを変更する際には `return` しているデータの型が意図通りであることを保証するため、戻り値に明示的な型付けをしたほうが望ましいでしょう。しかし、四角形の名前を返す部分と面積計算部分に分け、使用する側で適宜呼び出す設計とすることでそもそもこのような心配がなくなります。

```typescript
export interface Rect {
  width: number;
  length: number;
}

export function getName(rect: Rect) {
  if (rect.width === rect.length) {
    return "square";
  }
  return "rectangle";
}

export function calculateArea(rect: Rect) {
  return rect.width * rect.length;
}
```

この場合、戻り値の型はコードを見ればすぐにわかります。さらにコードを変更する際も 1 つの型だけを意識すればよいので、特に戻り値の型を明示する必要性は薄くなります。

ただし、ライブラリー作成時に関数の戻り値を書いていない場合、そのライブラリーの使用者にも関数の内部処理を読ませるのか？という問題が出てきます。が、これについては TypeScript によるコンパイル時に `declaration` オプションをオンにすることで、動作するコードと型宣言とを分離できますのでそれぞれを提供すればよいでしょう。コンパイルによって型宣言には関数の戻り値が明示されますのでライブラリーの使用者は型宣言のみを見て使い方を学習可能となります。さらに、そのライブラリーは JavaScript からも利用可能となります。

TypeScript コードから、型定義がどのように分離されるのか見てみましょう。tsconfig.json は次とします。

```json
{
  "compilerOptions": {
    "declaration": true,
    "outDir": "dist",
    "esModuleInterop": true,
    "target": "ESNext",
    "module": "CommonJS"
  }
}
```

次のコードを `index.ts` として保存し、コンパイルしましょう。

```typescript
export function double(n: number) {
  return 2 * n;
}
```

コンパイルするためのコマンドは次となります。2 行目まではコンパイルのための準備、3 行目が実際にコンパイルするコマンドですので、複数回コンパイルしたい場合は 3 行目のみを実行してください。

```console
yarn init
yarn add --dev --exact typescript
yarn tsc index.ts
```

すると `dist` ディレクトリーが生成され、いくつかファイルが出力されます。型定義は `index.d.ts` として、次のように生成されます。関数の戻り値が明示されていますね。

```typescript
export declare function double(n: number): number;
```

JavaScript コードは `index.js` として次が出力されます。

```javascript
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
function double(n) {
  return 2 * n;
}
exports.double = double;
```

これは Node.js でも読み込むことが可能な CommonJS 形式です。JavaScript からも利用可能なライブラリーが作成できました。
:::

## ECMAScript 2015 の新記法

ECMAScript 2015 は、TC39 という組織が 2015 年に制定した ECMAScript のバージョンを指しています。TC39 は Technical Committee 39 の略で ECMAScript を専門に議論している技術的な委員会です。ECMAScript は毎年アップデートされ、現在 ECMAScript 2020 が制定されている最中です。一般的に JavaScript と呼ばれている言語は、現在 ECMAScript のいずれかのバージョンもしくはすべてと同様の意味となっています。過去はより細かな差異があったのですが標準化が進んだ現在、実用上、区別する必要がなくなりました。

ECMAScript 2015 はもともと、ECMAScript 5 の次のバージョンにあたる ECMAScript 6 として策定が進んでいたものです。が、一年ごとに ECMASCript をバージョンアップするとの決定に伴い、いつのバージョンなのかをわかりやすく明示するために ECMAScript 2015 とあらためられました。

ECMAScript 2015 で可能となった書き方をかんたんに確認しておきましょう。

### `let` と `const` による変数定義

ECMAScript 5 では変数を宣言する際に `var` というキーワードを使っていました。

```typescript
function myFunc() {
  console.log(foo); // ②
  var foo = 0; // ①
  console.log(foo); // ③
}
myFunc();
```

① で `var` を使い変数 `foo` を宣言しています。しかしそれ以前の ② で変数 `foo` を使っています。予想に反してこのコードは実行可能です。実行すると次の結果が得られます。

```
undefined
0
```

`var` による変数宣言では、すべての変数は関数の先頭で宣言され、実際に宣言した行まで指定した値が代入されません。そのため ② の時点では `undefined` が出力されます。③ では ① による代入の後なので、 `0` が出力されるというわけです。これを変数の巻き上げと呼びます。

この挙動はときにバグを生み出す可能性があったため、ECMAScript 2015 で新たに導入された `let` および `const` が推奨となりました。

```typescript
function myFunc() {
  console.log(foo); // ①エラー！
  let foo = 0;
  console.log(foo);
}
myFunc();
```

① において変数 `foo` の宣言を `let` を使うように変更しています。 `let` および `const` を用いた場合、変数宣言後でなければその変数へのアクセスはエラーとなるため、思わずバグを作り込んでしまうことがなくなりました。 `const` もこの挙動は同様です。

`const` による変数宣言では、初期化が必須です。また、値の代入が禁止されています。

```typescript
const a = 42;
a = 0; // エラー！
```

ただし、オブジェクトや配列などを `const` で宣言した場合、代入そのものは禁止されていますがその中身については変更可能です。

```typescript
const obj = { a: 42 };
obj = { a: 0 }; // エラー！
obj.a = 0; // エラーとならない

const array = [];
array = ["foo"]; // エラー！
array.push("foo"); // エラーとならない
```

また、中カッコ `{}` によって囲まれた部分をブロックと呼びますが、ブロックごとに同名の変数を宣言可能です。

```typescript
const a = 42;
console.log(a);
{
  const a = "in block";
  console.log(a);
}
console.log(a);
```

このコードの実行結果は次のようになります。

```typescript
42 in block;
42;
```

ブロックの中で定義した変数 `a` がブロックの外で定義した変数 `a` を覆い隠していることに注意してください。この挙動はバグを生む可能性がありますので、変数は重複しない名前で宣言していくとよいでしょう。

### Arrow Functions

通常、関数定義は `function` キーワードを用いて次のように書く必要がありました。

```typescript
function divideName(name: string) {
  const names = name.split(/ /);
  return {
    familyName: names[0],
    personalName: names[1],
  };
}

divideName("Takagi Kensuke");
```

Arrow Functions では同様の関数定義を次のように書くことが可能です。

```typescript
const divideName = (name: string) => {
  const names = name.split(/ /);
  return {
    familyName: names[0],
    personalName: names[1],
  };
};

divideName("Takagi Kensuke");
```

引数定義に続いて `=>` を書き、その後に関数ブロックを書く形となります。また、名前をつけることができないため、変数に代入することであとから参照および使用が可能となります。ここでは `divideName` という変数に代入しています。JavaScript では数値や文字列などと同様、関数をデータとして扱うことができるため、こういった書き方が可能です。 `=>` が矢印に見えることから Arrow Functions という名前がつきました。

関数の処理内容が `return` 文ひとつで済むような場合、 `return` で返すべき値を `=>` に続けて書くことが可能です。

```typescript
functoin double01(n: number) {
  return 2 * n;
}

const double02 = (n: number) => 2 * n;
```

`double01` および `double02` はまったく同様の挙動となります。オブジェクトを返す際は少し工夫する必要があります。

```typescript
const makeRect = (length: number, width: number) => ({
  length: length,
  width: width,
});
```

`=>` に中カッコ `{}` を続けてしまうと関数ブロックと解釈されてしまいます。これを回避するため、カッコ `()` でくくってあげなければなりません。

Arrow Functions を用いて定義された関数は、 `function` キーワードを使って関数定義をするコードに比べ少ないタイプ数で書けるコードとなっています。その理由は Arrow Functions がもともと `Array.prototype.filter` など関数を引数として取る関数を使用する際に簡便に書くために用意されたものだからです。

```typescript
const evens01 = [1, 2, 3].filter(function (n: number) {
  return n % 2 === 0;
});

const evens02 = [1, 2, 3].filter((n: number) => n % 2 === 0);
```

上述のコードはまったく同じ結果を返しますが、後者のほうが短く、読みやすいコードとなっています。

#### 通常の関数定義と Arrow Function の使い分け

これまでの解説で、全く同じであればどのように使い分ければよいのかという疑問を持った方もいらっしゃるでしょう。厳密には違いがあるのですが、React Native を書く上ではどちらを使用してもなんら違いは生まれません。

別の視点として、コードの見通しや開発効率といった側面で考えてみましょう。

今まで見てきたように、Arrow Functions はコードが短くなる反面、名前をつける際に変数を用いる必要があります。使っているツールによっては変数として認識してしまい、関数としてのサポートを受けられない可能性があります。対して `function` を用いた場合、きちんと関数として扱われますが、コードが長くなります。

これらを考慮すると、次のように使い分けるのがよいでしょう。

| 書き方          | 場合分け                       |
| --------------- | ------------------------------ |
| Arrow Functions | その場でしか使わない関数の定義 |
| 通常の関数定義  | それ以外の用途                 |

#### 関数の型

Arrow Function の書き方は関数の型定義としても使用可能です。

```typescript
type NumericalMap = (n: number) => number;

function doubleCall(n: number, numericalMap: NumericalMap) {
  return numericalMap(numericalMap(n));
}

function double(n: number) {
  return 2 * n;
}

doubleCall(3, double);

function concat(str: string) {
  return str + str;
}

doubleCall(3, concat); // エラー！
```

Type Aliasing を用い、引数定義、 `=>` に続いて型を指定することで関数の定義となります。 `interface` を用いた定義に比べると簡素化されていることがわかります。

### Default Parameter Values

関数を呼び出す際の引数にデフォルト値を定義可能となっています。

```typescript
function round(src: number, digit: number = 1) {
  const base = Math.pow(10, digit - 1);
  return Math.round(src * base) / base;
}

round(1.23); // 1
round(1.23, 2); // 1.2
```

数値を任意の桁で四捨五入する関数 `round` を定義しました。第 2 引数 `digit` で四捨五入する桁を指定しますが、関数定義ではこのデフォルト値を `1` としています。呼び出し時に第 2 引数を指定せずとも使用できています。

TypeScript でコンパイルし、型定義を出力すると次となります。

```typescript
function round(src: number, digit?: number): number;
```

第 2 引数に `?` が付与され、任意指定であることが明示されています。ここで、第 1 引数にデフォルト値を定義し、第 2 引数にはデフォルト値を定義しない場合はどうなるでしょうか？ `round` 関数の引数をひっくり返した `altRound` 関数を定義します。

```typescript
function altRound(digit: number = 1, src: number) {
  const base = Math.pow(10, digit - 1);
  return Math.round(src * base) / base;
}
```

このコードをコンパイルすると次の定義が出力されます。

```typescript
function altRound(digit: number | undefined, src: number): number;
```

第 1 引数が `digit` の型が `number | undefined` となっています。第 1 引数に `undefined` を指定するとデフォルト値が使われるようになっています。

```typescript
altRound(undefind, 1.23); // 1
altRound(2, 1.23); // 1.2
```

これは使いづらいと感じるでしょう。特別な理由がない限り、デフォルト値を定義したい引数は、値指定が必須である引数の後ろで定義しましょう。

### Template Literals

文字列を構築する際、変数の内容を利用したい場合があります。

```typescript
const url = "https://" + domain + ":" + portNumber;
```

読みづらいですね。Template Literals はこういったコードを見やすくする、文字列定義中に変数を書ける機能です。次のコードは上述したものとまったく同様の挙動となります。

```typescript
const url = `https://${domain}:${portNumber}`;
```

まず `'` や `"` ではなく `` ` `` で文字を囲みます。その中で利用したい変数などは `${` と `}` によって囲むことで展開されます。対象は変数のみではなく、式が利用可能です。

```typescript
const url = `https://${domain}:${portNumber}/users/${encodeURIComponent(
  userName
)}`;
```

変数 `userName` の内容として URL に使用可能な文字以外を含む場合を想定しています。問題なく動作します。

### Property Shorthand

すでに定義された変数を用いてオブジェクトリテラルを定義する際、ECMAScript 5 では次のように書く必要がありました。

```typescript
const length = 3;
const width = 4;

const rect = {
  length: length,
  width: width,
};
```

ECMAScript 2015 ではもう少し短く書くことが可能です。

```typescript
const length = 3;
const width = 4;

const rect = {
  length,
  width,
};
```

各プロパティの定義が変数のみとなっています。この書き方は上述した ECMAScript 5 での書き方と全く同様のコードにコンパイルされます。変数 `length` の名前がプロパティ名、値がプロパティ値として定義されます。コードをすっきりさせ、読み書き両面でメリットが得られます。

### Computed Property Names

ECMAScript 5 以前で、オブジェクトのプロパティ名に変数の値を使いたい場合、次のように書く必要がありました。

```javascript
const extraProperty = "name";
const rect = {
  length: 3,
  width: 4,
};
rect[extraProperty] = 42;
```

TypeScript で同様のコードを書く場合、変数 `rect` に対して型付けし、プロパティ `name` を持ちうることを明示してあげなければなりません。

```typescript
interface Rect {
  length: number;
  width: number;
  name?: string;
}
const extraProperty = "name";
const rect: Rect = {
  length: 3,
  width: 3,
};
rect[extraProperty] = "square";
```

冗長ですね。ECMAScript 2015 で導入された Computed Property Names を使うことで以下のようにかんたんに書くことが可能です。

```typescript
const extraProperty = "name";
const rect = {
  length: 3,
  width: 3,
  [extraProperty]: "square",
};
```

プロパティ名の位置に変数名を角カッコ `[]` で囲み記述しています。プロパティ名として使用可能な型はいくつかあるのですが、現状 `string` 型を指定することが最良です。

### Destructuring

オブジェクトや配列、Tuple など構造化されたデータにアクセスする際、添字やプロパティ名の指定が必要でした。

```typescript
const obj = { foo: 42 };
const array = [1, 2, 3];
const tuple: [number, string] = [1, "foo"];

obj.foo + array[0] + tuple[0];
```

Destructuring とは「非構造化」という意味合いです。構造化されたデータの一部を変数に割り当て、扱いやすくする機能です。

```typescript
const obj = { foo: 42 };
const array = [1, 2, 3];
const tuple: [number, string] = [1, "foo"];

const { foo: a } = obj;
const [, b] = array;
const [c] = tuple;
a + b + c;
```

それぞれ次のように変数を割り当て、使用しています。

- オブジェクトの `foo` プロパティの値を変数 `a`
- 配列の 2 つめの要素を変数 `b`
- Tuple の 1 つめの要素を変数 `c`

Destructuring は関数引数に対しても有効です。

```typescript
interface Rect {
  length: number;
  width: number;
}

function area({ length, width }: Rect) {
  return length * width;
}

area({ length: 3, width: 4 });
```

関数 `area` の引数定義の際に Destructuring を使用しています。関数呼び出し時の引数の指定はオブジェクトですが、関数内部ではオブジェクトを意識せず使えます。

また、オブジェクトを Destructuring する際、変数名を割り当てなければオブジェクトのプロパティ名がそのまま変数名となります。

深い階層の Destructuring は次のように書きましょう。

```typescript
interface Profile {
  name: string;
  contacts: {
    phoneNumber: string;
    mailAddress: string;
  };
}

function mailBox({ name, contacts: { mailAddress } }: Profile) {
  return `${name}<${mailAddress}>`;
}

mailBox({
  name: "Takagi",
  contacts: {
    phoneNumber: "090-0000-0000",
    mailAddress: "takagi.kensuke@example.com",
  },
});
```

### Spread Operator

既存のオブジェクトや配列の内容を新しいオブジェクトや配列の内容として展開する書き方です。

```typescript
const foo = { foo: 1 };
const bar = { bar: 2 };
const newObject = {
  ...foo,
  ...bar,
  buz: 3,
};
```

`newObject` は次の内容となります。

```typescript
{
  foo: 1,
  bar: 2,
  buz: 3,
}
```

同時に一部のプロパティを変更可能です。

```typescript
const rect = {
  length: 3,
  width: 4,
};
const newRect = {
  ...src,
  width: 42,
};
```

このように書くと `newRect` の内容は次となります。

```typescript
{
  length: 3,
  width: 42,
}
```

配列についても使用可能です。

```typescript
const numbers = [1, 2, 3];
const newNumbers = [0, ...numbers, 4];
```

`newNumbers` の内容は次となります。

```typescript
[0, 1, 2, 3, 4];
```

Spread Operator によって作成されたオブジェクトや配列は、新しいオブジェクトや配列を生成します。

```typescript
const rect = {
  length: 3,
  width: 4,
};
const numbers = [1, 2, 3];

console.log(rect === { ...rect }); // false
console.log(numbers === [...numbers]); // false
```

### class キーワード

以前の ECMAScript でもクラスを定義可能でしたが、 `class` キーワードによるクラス定義が可能となったのは ECMAScript 2015 からです。

```typescript
class Rect {
  // ①
  length: number;
  width: number;

  // ②
  constructor(length: number, width: number) {
    this.length = length;
    this.width = width;
  }

  // ③
  area() {
    return this.length * this.width;
  }
}

// ④
const rect = new Rect(3, 4);
rect.area();
```

TypeScript でクラス定義する場合、① のようにオブジェクトが持つデータの型を明示する必要があります。オブジェクト生成時の初期化処理は ② のように、振る舞いは ③ のように定義します。使用する際は ④ のように `new` キーワードを用います。

クラス定義についてはとてもたくさんのトピックがありますが、本書を読むために必要な知識はここで紹介したもののみです。

### 非同期処理

#### コールバック関数

ネットワークやハードウェアとの通信において、応答が返ってきた際に処理を実行したい場合があります。その場合、これまでは応答が返ってきた際に実行したい関数を指定する、という方法をとっていました。この関数をコールバック関数と呼びます。

```javascript
get("https://example.com/api/names/1", ({ status, text }) => {
  // textには'Takagi'が格納されている
  if (status === 200) {
    console.log(text); // ①
  }
});
console.log("getting data..."); // ②
```

`get` 関数はネットワーク通信によってデータを取得する関数と仮定しています。 `get` 関数の第 2 引数に応答が返ってきた際に呼び出されるコールバック関数を指定しています。このコードの実行結果は次のようになります。

```
getting data...
Takagi
```

通常、コードは上から下へと実行されていきますが、ここでは ① より先に ② が実行されています。ネットワーク通信の応答を受け取るまでコールバック関数は実行されないためです。こういったコードの実行順が見た目と異なる処理を非同期処理と呼びます。

この処理は単純ですが、たとえば通信結果によってさらに別のコールバック関数を必要とする処理が連なる場合、以下のように読みづらいコードになってしまいます。

```typescript
get("https://example.com/apis/names/1", ({ status, text: name }) => {
  if (status !== 200) {
    throw new Error("can not get name");
  }
  get(`https://example.com/apis/books/${name}`, ({ status, text: book }) => {
    if (status !== 200) {
      throw new Error("can not get book");
    }
    get(`https://example.com/apis/pages/${book}`, ({ status, text: page }) => {
      if (status !== 200) {
        throw new Error("can not get page");
      }
      // さらに通信が続く
    });
  });
});
console.log("getting data...");
```

#### Promise

コールバック関数による処理は有用ですが、多段階の呼び出しが必要な場合、コードの可読性を保てない欠点があります。代わりに ECMASCript 2015 で導入された Promise を使いましょう。

```typescript
get("https://example.com/apis/names/1")
  .then(({ status, text: name }) => {
    if (status !== 200) {
      return Promise.reject("can not get name");
    }
    return get(`https://example.com/apis/books/${name}`);
  })
  .then(({ status, text: book }) => {
    if (status !== 200) {
      return Promise.reject("can not get book");
    }
    return get(`https://example.com/apis/pages/${book}`);
  })
  .catch((e) => {
    console.log(e);
  });
```

同様の処理を `Promise` を用いて書いていますが、 `get` 関数が `Promise` オブジェクトを返すことを仮定しています。

この場合、コールバック関数を `get` 関数へ指定する代わりに、 `get` 関数の実行結果である `Promise` オブジェクトが持つ `then` メソッドで応答が返った際の関数を指定しています。複数の `then` を連ねることで非同期処理を順次実行可能です。また、通信が失敗した場合の処理を `catch` メソッドに指定することでエラー処理しています。失敗とみなされる場合、つまり `throw` していた箇所が `Promise.reject` 関数の結果の `return` に変わっていることにも注意してください。

React Native ではライブラリーで提供されている関数の返り値として `Promise` 型が定義されていることがあります。それらの関数を使用する際は次の書き方を参考にしましょう。

```typescript
fetch("https://example.com").then(onFullfilled).catch(onRejected);

function onFullfilled(response) {
  // fetchが成功した場合の処理
}
function onRejected(error) {
  // fetchが失敗した場合の処理
}
```

#### async/await

React Native では ECMAScript 2017 で導入された `async` / `await` も使うことが可能です。次のコードはこれまでと同様の処理を `async` / `await` を用いて書いたものです。

```typescript
async function getData() {
  try {
    const { status: statusName, text: name } = await get(
      "https://example.com/apis/names/1"
    );
    if (statusName !== 200) {
      throw new Error("can not get name");
    }
    const { status: statusBook, text: book } = await get(
      `https://example.com/apis/books/${name}`
    );
    if (statusName !== 200) {
      throw new Error("can not get book");
    }
    const { status: statusPage, text: page } = await get(
      `https://example.com/apis/pages/${book}`
    );
  } catch (e) {
    console.log(e.message);
  }
}

getData();
```

Promise を用いた場合に比べて同期的な書き方に非常に似ていますが、次の 5 つの違いがあります。

- `getData` 関数の定義の先頭に `async` キーワードが付与されている
- `get` 関数の呼び出しの前に `await` キーワードが付与されている
- `get` 関数の実行結果を変数に代入している
- 失敗とみなしたい場合に例外を送出している
- `try` / `catch` を用いている

`Promise` 型を意識しなくてよいコードとなっています。非同期処理が必要な場合、基本的に `async` / `await` を用いて書いていくとよいでしょう。

Arrow Functions においても `async` / `await` を使用可能です。

```typescript
const getData = async () => {
  const { status, text } = await get("https://example.com/apis/names/1");
  // （中略）
};
```

### ES Modules

ひとつのファイルにアプリケーションを構築するすべてのコードが書かれていてはメンテナンスが難しくなります。ECMAScript 2015 ではファイルにコードを分割し、互いに読み込める仕組みとして ES Modules が定義されています。

```typescript
// circle.ts
export const PI = 3.141592;
export interface Circle {
  radius: number;
}
export function area({ radius }: Circle) {
  return PI * Math.pow(radius, 2);
}
export default (a: Circle, b: Circle) => a.radius === b.radius;
```

モジュールから何かしらのデータを提供する場合、 `export` キーワードを用います。提供可能なものはすべてのデータおよび型定義です。関数はデータに含まれるので `export` 可能です。コードでは変数、型定義、関数を `export` しています。

```typescript
// app.ts
import equals, { PI, Circle, area } from "./circle";

const circleA = { radius: 3 };
area(circleA);

function circumference({ radius }: Circle) {
  return 2 * PI * Math.pow(radius, 2);
}

equals(circleA, {
  radius: 4,
});
```

`export` したものを他のファイルで使うためには `import` する必要があります。

`import` に続けてそのファイルで使いたいものを指定します。 `import` のすぐ右側に指定されている `equals` は `circle.ts` において `export default` されている `equals` 関数です。また、その他のデータは中カッコ `{}` に囲われていますが、これらは `circle.ts` 内で定義されている名前を明示的に指定する必要があります。前者を default export、後者を named export と呼びます。

#### default export/named export

default export では名前を意識する必要がないことに注意してください。

```typescript
// module.ts
exprt default function echo(target: any) {
  console.log(target);
}
```

```typescript
// app.ts
import something from "./module";

something(42);
```

`module.ts` では `echo` 関数を default export していますが、 `app.ts` では `something` という名前で `import` し使っています。 `export` 時に名前が付けられている必要がないため、リテラル値や Arrow Functions などを指定可能です。ただし、1 つのファイルにつき 1 つしか default export できません。2 つ以上の指定はどちらを `import` すべきかわからなくなるからです。

対して named export では `export` されている名前を意識する必要がありますが、いくつでも `import` 可能です。

```typescript
// module.ts
export const foo = 42;
export const Foo = 0;
export bar = () => 'bar';
export buz = [1, 2, 3];
```

```typescript
// app.ts
import { foo, Foo, bar } from "./module";

(foo + Foo).toString() + bar;
```

`export` されているすべてのものを `import` する必要はありません。また、大文字小文字は区別されます。named export を個別に指定せず、後からまとめて `export` できます。

```typescript
const foo = 42;
const bar = "bar";

export { foo, bar };
```

このコードでは `foo` と `bar` を named export しています。次の書き方をしてしまうと `foo` プロパティと `bar` プロパティを持つオブジェクトを default export していることになってしまいます。違いに注意しましょう。

```typescript
const foo = 42;
const bar = "bar";

export default { foo, bar };
```

default export されたものと named export されたものを同時に `import` 可能です。

```typescript
import foo, { bar, buz } from "./module";
```

named export されたものの指定は必ず default export の後でなければなりません。

#### パス指定

`from` の右側にはファイルを指定しますが、拡張子は書きません。相対パスか絶対パスかで探しに行く先が変わります。

| 書き方   | 例         | 探す先                                          |
| -------- | ---------- | ----------------------------------------------- |
| 相対パス | './circle' | `import` 文が書かれているファイルからの相対パス |
| 絶対パス | 'circle'   | `node_modules` 直下                             |

複数のディレクトリー階層をまたいで指定できます。

```typescript
import Icon from "react-native-vector-icons/FontAwesome";
import { assertIsDefined } from "../../lib";
```

#### DefinitelyTyped

React Native 開発において使用するモジュールは npm から取得しますが、TypeScript が世の中に出たときすでに膨大な数の npm パッケージが存在していました。そのため、型定義を持たないパッケージも多いのです。かくいう React Native も型定義は同梱されていません。

そういった npm パッケージをモジュールとして読み込む際、有志によって定義されている型を使用する形になります。npm パッケージ名の前に `@types/` をつけたものが型定義のパッケージです。React Native の場合は `@types/react-native` になります。これらは DefinitelyTyped という GitHub リポジトリーで管理され、日々型定義が追加、更新されています。 TypeScript が型定義を読み込めないエラーを出力した場合は当該パッケージの先頭に `@types/` をつけた npm パッケージを探してみるとよいでしょう。
