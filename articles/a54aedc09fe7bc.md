---
title: "React コンポーネントの書き方をチームで統一する"
emoji: "⚛️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [React, TypeScript]
published: true
publication_name: "beingish"
---

次の記事を見てなるほどと思ったものの、コンポーネント集作成の視点で書かれている。我々はアプリを作る機会が多いので、その際はどういう書き方が楽なのか、チームで共有するために類型を洗い出し、比較した。

https://kray.jp/blog/dont-have-to-use-react-fc-and-react-vfc/

## TL;DR

関数での書き方を覚えれば良いと判断した。

```typescript
function MyComponent(props: React.PropsWithChildren<Props>) {
  if (props.title.length % 2 === 0) {
    return null;
  }
  return <div>{props.children}</div>;
}
```

ここからやりたいことによって不要なものを削除していく。例えば props が必要ない場合、シグネチャーから props 定義を削除する。子要素の指定が必要ない場合は `React.PropsWithChildren` の使用をやめる。 `null` を返す必要がない場合は `if` 節を削除する。

以降でなぜこの書き方に落ち着いたのかを記載する。

## 類型

類型を MECE で考えるため、 2 つの条件をそれぞれ 3 パターン出し、掛け合わせて 9 パターンの定義を見てみた。

次の表は横方向が props の形状、縦方向が型の付け方という条件になっている。

|                       | N: props なし | P: props あり | C: 子要素あり |
| --------------------- | ------------- | ------------- | ------------- |
| F: 関数               | FN            | FP            | FC            |
| V: React.V?FC         | VN            | VP            | VC            |
| R: React.ReactElement | RN            | RP            | RC            |

名前からどういった条件なのかがわかるようにアルファベット 2 文字で名前付けをしている。

`React.VFC` については将来的に `React.FC` になるため、 `React.V?FC` という表記にしている。

https://ikesyo.hatenablog.com/entry/2020/12/18/141737

## 前提

以降のすべてのコンポーネント定義は次の定義を前提としている。

```typescript
interface Props {
  title: string;
}
```

また、コンポーネントはすべて `null` を返す可能性があるものとして考える。

## 9 パターン

[TypeScript PlayGround での定義](https://www.typescriptlang.org/play?ssl=36&ssc=1&pln=73&pc=1#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgcilQ3wChTgA7GJKTdJOABRzAGc4BvUuOGYGABskALjhsYUKgHNSAX3IB6AFSkAYmMwBXShmARKpAGpiA7gIAWiYjAB0RgPxqAwqQRnL19HeTeAosIgSNTkAHJilBBwYKxspEweMFYxEOykzolWaBbAggAmRIbKiuTauvwGcGqhABQAlFw8cMCYcDUAsihJtlAolHm49Q3cvLxEMFpQlHCUWoKCANxNCmNIE1NwADx5wABuAHwALABMm4o7B0sKZXqVakw1KexiLKlsw00tbU9stvxCSFswko0iScAApHBjnAALxwuAABg+ozg40m01m8yWvBWqLW6K2F32nB+fwEwjkZyJV1INwq0xcj1iYl8GFsr3YAHVLM4cvlCpsOWx9sjmq0mW8yQCgcFQVZIdC4TDEaLVusMXNFssmmiNts9sTSdlcgVgpTzgaaaQ0AYJHAjOEvGyjC5YW0GjD9o1eF8Ol0LD0+gMQENvaNdRqsdq1QT9QcTlTLfIljbKHajAknXYXc5BbEvcqfrCvSMxd9YlLhDKQWCFbD4UiwzGNpitTidfi9USSRX-hTE5dk9bbfAjBks-YXJtWXYhdykryTQKhfsC9FYsWw77SX3AcC5RCofXlY3S83I224LiI4SDT3Jcb+WaB-srkth2n4AhHfVNzPbDOARIEE1BwAAPjMmpur+npbuKnTdL0-SDHUqp4uqkFRu2563vGpwWoOcgpiOiCZhKzzMLEHpev+gGBMEMB5m8XoQa2bpFrBpbbr25J7rKtZHkqKpNuhBKttiV4dhhcaGjxALmtSQ6pnaCDjuRbAsjY7KxPOFiLk+lBMewIp-lpdHAQxRnCuBmGCOxG6cZ84o7rx1YHnWQmnk0OHidGoldneRp8qalAKUmRFKIocAALSxeQSAAB6QLAcB5Eg9BzFgOi3NMACCYBgL+pY3jU3lbPsZW8Js1RwIoFUolV9x8LxMIAESYBAECtbV9UNdVzjNQCbUdV1+wAEYoFAZwuBVlVbA6PVzZsGaDcIw2dd1dVLWOq1IOto0TVNihjrNDVbN+i1ndOTC7ftm29Si04Dbud3jZNZyqQ9WxbbwdTyEAA)。

### FN: 関数 & props なし

```typescript
function NF() {
  if (Math.random()) {
    return null;
  }
  return <div>42</div>;
}
```

### FP: 関数 & props あり

```typescript
function FP(props: Props) {
  if (props.title.length % 2 === 0) {
    return null;
  }
  return <div>{props.title}</div>;
}
```

### FC: 関数 & 子要素あり

```typescript
function FC(props: React.PropsWithChildren<Props>) {
  if (props.title.length % 2 === 0) {
    return null;
  }
  return <div>{props.children}</div>;
}
```

### VN: React.V?FC & props なし

```typescript
const VN: React.VFC = () => {
  if (Math.random()) {
    return null;
  }
  return <div>42</div>;
};
```

### VP: React.V?FC & props あり

```typescript
const VP: React.VFC<Props> = (props) => {
  if (props.title.length % 2 === 0) {
    return null;
  }
  return <div>{props.title}</div>;
};
```

### VP: React.V?FC & 子要素あり

```typescript
const VC: React.VFC<React.PropsWithChildren<Props>> = (props) => {
  if (props.title.length % 2 === 0) {
    return null;
  }
  return <div>{props.children}</div>;
};
```

### RN: React.ReactElement & props なし

```typescript
const RN: () => React.ReactElement | null = () => {
  if (Math.random()) {
    return null;
  }
  return <div>42</div>;
};
```

### RP: React.ReactElement & props あり

```typescript
const RP: (props: Props) => React.ReactElement<Props> | null = (props) => {
  if (props.title.length % 2 === 0) {
    return null;
  }
  return <div>{props.title}</div>;
};
```

### RC: React.ReactElement & 子要素あり

```typescript
const RC: (
  props: React.PropsWithChildren<Props>
) => React.ReactElement<Props> | null = (props) => {
  if (props.title.length % 2 === 0) {
    return null;
  }
  return <div>{props.children}</div>;
};
```

## 比較

`React.ReactElement` を用いるパターンは煩雑に感じる。特に `null` が返りうることを明示するところがいまいち。 React コンポーネントの型が `React.ReactElement` であることさえ覚えてしまえば、この書き方は忘れても良い。

関数を用いるものが一番シンプルだが返り値の型を指定していない。試しに指定すると `React.ReactElement` を用いるパターンと同じく煩雑になってしまう。

```typescript
function NF(): React.ReactElement | null {
  // snip
}
```

では返り値の型指定なしで実害は出ないのか？多分出ない。 [`React.Element` 以外の型を持つ値をコンポーネントから返すと、型エラーとなる](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgcilQ3wChTMBXAOw2AmrgFEAKASjgG9S44iZKURgBYATAG5SAX3JIAHpFhwAJkkwpKAGyw06DOAEEwYdlx58kAoXAA8TOAHoAfNKA)ためだ。

```typescript
import React from "react";

function E() {
  return 42;
}

export default function App() {
  return <E />; // 'E' cannot be used as a JSX component.
  // Its return type 'number' is not a valid JSX element.(2786)
}
```

人間が勘違いしないか、という点でも問題ないはずだ。現在ではコンポーネントを特定のディレクトリー以下へ集約することが暗黙の了解になっている。また、大抵の場合コンポーネントの名前を PascalCase でつける慣習に従っているだろう。これらのことから今見ている関数定義がコンポーネントなのかどうかは比較的判別しやすいはずだ。

では関数と `React.V?FC` と、どちらを使って書くのがいいのか？多分に好みが分かれるところだが、チームでの書き方統一という観点では覚えることが少ない関数を用いた書き方が良いだろうと判断した。

:::message
現時点では将来的に `React.VFC` から `React.FC` への書き換えが必要となる点も関数を使った書き方を選択するモチベーションになるだろう。
:::

`React.V?FC` を用いる場合、多少タイプ数が増える代わりに TypeScript のコンパイル時間短縮の可能性があることだ。巨大なアプリを作成するときにはベンチマークを取りながら検討しても良いだろう。
