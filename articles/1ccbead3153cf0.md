---
title: "TypeScriptのinterfaceとtype、どっち使う？ジェネリック制約でハマる意外な落とし穴"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# TypeScript の`interface`と`type`、どっち使う？ジェネリック制約でハマる意外な落とし穴

TypeScript を触っている皆さん、`interface`と`type`の使い分けで悩んだことはありませんか？どちらを使っても同じように書けることが多いので、深く考えずに使っている方もいるかもしれません。

普段は意識しなくても問題ないことが多いのですが、実は**ジェネリック型制約**、特に`Record<string, unknown>`のような型が登場すると、予期せぬエラーに遭遇することがあります。

この記事では、この「あるある」なエラーの具体的な事例から、その原因、そして明日から使える解決策までを解説します。

---

## 1\. 事例で理解するジェネリック型制約の落とし穴

まずは、実際にエラーが発生する状況を見てみましょう。
最近のフロントエンド開発では、Storybook のようなツールを使ってコンポーネントを独立して開発・テストすることが一般的です。その際、コンポーネントの Props に Storybook 関連のユーティリティ型を追加するケースがあります。

例えば、以下のような`AppendStorybookProps`というユーティリティ型を考えてみてください。これは、あるコンポーネントの Props に Storybook で使うためのプロパティを追加することを想定しています。

```typescript
// src/types/storybook.ts
// この型は、受け取ったProps型 P が Record<string, unknown> の構造を持っていることを期待しています。
type AppendStorybookProps<
  TOriginalProps,
  TAddedProps extends Record<string, unknown>
> = TOriginalProps & {
  // ここにStorybookで使用する追加のプロパティを定義する
  storybookControls?: TAddedProps;
  // 実際にはもっと複雑なロジックがあるかもしれませんが、例として簡略化
};
```

この`AppendStorybookProps`を使って、とあるコンポーネントの Props を拡張しようとしたときに、以下のようなエラーに遭遇することがあります。

```typescript
// src/components/FuelCostRegister.tsx

// FuelCostRegister.tsx の Props を interface で定義
export interface FuelCostRegisterProps {
  onClose: () => void;
  // その他のプロパティ...
}

// ❌ エラーが発生する使い方
// FuelCostRegisterProps を AppendStorybookProps の第二引数に渡す
export type StorybookEnhancedFuelCostRegisterProps = AppendStorybookProps<
  unknown, // 実際には何らかのPropsが入りますが、ここでは省略
  FuelCostRegisterProps // ここでエラー！
>;
```

このコードを実行すると、次のようなエラーが出力されます。

```
型 'FuelCostRegisterProps' は制約 'Record<string, unknown>' を満たしていません。
型 'string' is missing in type 'FuelCostRegisterProps' のインデックス シグネチャがありません。
```

「あれ？`FuelCostRegisterProps`は普通のオブジェクト型なのに、なぜ`Record<string, unknown>`を満たさないの？」「`string`型のインデックスシグネチャがないってどういうこと？」と頭を抱えてしまうかもしれません。

---

## 2\. `interface`と`type`の構造的互換性の違い

なぜこのエラーが起きるのでしょうか？その鍵は、TypeScript における\*\*`interface`と`type`の構造的互換性の判断基準の違い\*\*にあります。

まず、エラーメッセージに出てきた`Record<string, unknown>`とは何でしょうか？
これは、\*\*「任意の文字列をキーとして持ち、その値が`unknown`型であるオブジェクト」\*\*を表現するユーティリティ型です。つまり、以下のように書いた型とほぼ同等です。

```typescript
type WithIndexSignature = {
  [key: string]: unknown; // これが「インデックスシグネチャ」
};
```

では、なぜ`FuelCostRegisterProps`（`interface`で定義された型）がこの`Record<string, unknown>`という制約を満たさないのでしょうか。

### `interface`ではなぜエラーになるのか？

TypeScript の`interface`は、主にオブジェクトの\*\*「形状（シェイプ）」**を定義するために使われます。そして、`interface`には**「宣言のマージ (Declaration Merging)」\*\*という強力な特性があります。同じ名前の`interface`を複数回宣言すると、それらは自動的に結合（マージ）されます。

この「宣言のマージ」という特性があるため、コンパイラは`interface`に対してより慎重になります。もし`interface`が暗黙的に`Record<string, unknown>`のようなインデックスシグネチャを持つと判断してしまうと、後からマージされた別の`interface`のプロパティと衝突する可能性が出てきてしまいます。

そのため、`interface`が`Record<string, unknown>`のような**任意の文字列キーを受け入れる型として扱われるためには、明示的にインデックスシグネチャ `[key: string]: unknown;` を定義する**必要があります。明示的に書かれていない場合、TypeScript は「この`interface`は、定義されたプロパティ以外の文字列キーを持つことを想定していない」と判断し、エラーを発生させます。

### `type`ではなぜエラーにならないのか？

一方、`type`エイリアスは宣言のマージを持ちません。同じ名前の`type`エイリアスを複数回宣言すると、それはエラーになります。

この特性のおかげで、`type`エイリアスは`interface`よりも**構造的互換性の判断が柔軟**になります。`type`で定義されたオブジェクト型は、明示的なインデックスシグネチャがなくても、その構造が`Record<string, unknown>`と**構造的に互換性がある**と TypeScript コンパイラによって判断されやすいのです。つまり、定義されたプロパティが存在する限り、TypeScript は暗黙的に「追加のプロパティも存在しうる」と解釈することがある、ということです。

### 実際に違いを見てみよう

この挙動の違いを、よりシンプルなコードで確認してみましょう。

```typescript
// Record<string, unknown>制約をテストするためのユーティリティ型
type RequiresStringIndex<T extends Record<string, unknown>> = T;

// ------------------------------------

// ❌ interface で定義した場合
interface InterfaceProps {
  onClose: () => void;
  name: string;
}

// InterfaceProps は Record<string, unknown> 制約を満たさないためエラー
type InterfaceTest = RequiresStringIndex<InterfaceProps>;
/*
型 'InterfaceProps' は制約 'Record<string, unknown>' を満たしていません。
型 'string' is missing in type 'InterfaceProps' のインデックス シグネチャがありません。
*/

// ------------------------------------

// ✅ type で定義した場合
type TypeProps = {
  onClose: () => void;
  name: string;
};

// TypeProps は Record<string, unknown> 制約を満たすためエラーなし
type TypeTest = RequiresStringIndex<TypeProps>; // OK！
```

この検証コードから、`interface`と`type`でジェネリック型制約の挙動が明確に異なることが分かりますね。

---

## 3\. 解決策：ジェネリック制約がある場合は`type`を使おう！

ここまでくれば解決策は明確です。`AppendStorybookProps`のように\*\*`Record<string, unknown>`制約を持つジェネリック型に Props を渡す場合**は、**`interface`ではなく`type`を使用する\*\*ようにしましょう。

先ほどのエラーが発生していたコードを`type`に変更すると、無事にコンパイルが通ります。

```typescript
// src/components/FuelCostRegister.tsx

// FuelCostRegister.tsx の Props を type で定義に変更！
export type FuelCostRegisterProps = {
  // <-- interface から type に変更
  onClose: () => void;
  // その他のプロパティ...
};

// ✅ エラーなし
// FuelCostRegisterProps を AppendStorybookProps の第二引数に渡す
export type StorybookEnhancedFuelCostRegisterProps = AppendStorybookProps<
  unknown,
  FuelCostRegisterProps
>;
```

### プロジェクトでの対応方針

今回のケースのように、外部から提供されるユーティリティ型（例：Storybook のユーティリティ、特定のフレームワークの型ヘルパーなど）が`Record<string, unknown>`のような制約を持つ場合、**そのユーティリティ型に渡すコンポーネントの Props は`type`で定義する**ことをお勧めします。

```typescript
// ✅ 推奨：ジェネリック型制約との互換性を考慮し、type で定義
export type MyComponentProps = {
  prop1: string;
  prop2: number;
};

// ❌ 非推奨（AppendStorybookPropsなどでエラーになる可能性）
// export interface MyComponentProps {
//   prop1: string;
//   prop2: number;
// }
```

もちろん、`interface`が適しているユースケース（例えば、クラスの`implements`として使いたい場合や、宣言のマージを利用して既存の型を拡張したい場合など）もたくさんあります。しかし、今回のようなジェネリック制約でのハマりどころを避けるためには、適切な使い分けが重要になります。

---

## 4\. まとめと Next Steps

この記事では、TypeScript の`interface`と`type`が、`Record<string, unknown>`のようなジェネリック型制約に対して異なる挙動を示すことについて解説しました。

重要なポイントは以下の通りです。

- `interface`と`type`は似ていますが、**ジェネリック型制約、特に`Record<string, unknown>`を含むケースでは挙動が異なります。**
- **`interface`は宣言のマージ特性を持つため、インデックスシグネチャに関してより厳密**であり、明示的な定義がないと`Record<string, unknown>`とみなされません。
- **`type`は宣言のマージを持たないため、構造的互換性の判断がより柔軟**で、暗黙的に`Record<string, unknown>`と互換性があるとみなされやすいです。
- 解決策は、**該当するユースケースでは`interface`ではなく`type`を使用して Props を定義する**ことです。

この知識があれば、TypeScript の型エラーで悩む時間が減り、より堅牢で理解しやすいコードが書けるようになるはずです。

### 参考文献

- [TypeScript Handbook - Interfaces](https://www.google.com/search?q=https://www.typescriptlang.org/docs/handbook/2/interfaces.html)
- [TypeScript Handbook - Type Aliases](https://www.google.com/search?q=https://www.typescriptlang.org/docs/handbook/2/everyday-types.html%23type-aliases)
- [GitHub - Issue \#15300: Index signature is missing in type (only on interfaces, not on type alias)](https://github.com/microsoft/TypeScript/issues/15300)

---

この記事が皆さんの TypeScript 学習の一助となれば幸いです。もし他に TypeScript で疑問に感じていることがあれば、ぜひ教えてください！
