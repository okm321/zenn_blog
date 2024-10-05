---
title: "React Testing Libraryの複数の要素で作られたテキストのテストで手こずった話"
emoji: "🐩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "testinglibrary", "jest"]
published: true
---

## はじめに

React Testing Library でテストを書いている際、`<h2>こんにちは<strong> 世界</strong></h2>` のように複数の要素で構成されたテキストのテストに手こずったため、備忘録として残しておきたいと思います。

## どの場合にテキストの取得が難しいか

`<h1>合計¥10,000です</h1>`のような単一の要素で作成されたテキストの取得は以下のように`getByRole`, `getByText`で問題なく取得することが可能です。

```tsx
const SingleElementText = () => {
  return <h1 data-testid="single-element-text">合計¥10,000です</h1>;
};

// getByRole
expect(
  screen.getByRole("heading", { name: "合計¥10,000です" }),
).toBeInTheDocument();
// getByText
expect(screen.getByText("合計¥10,000です")).toBeInTheDocument();
```

@[stackblitz](https://stackblitz.com/edit/vitejs-vite-ogqhyc?embed=1&file=src%2Fcomponents%2FSingleElementText.test.tsx&view=editor)

---

では、テストの内容は変更せずにコンポーネントを複数の要素で構成された`<h1>合計<span>¥10,000</span>です</h1>`に変更して、再度テストを実行してみましょう！

```diff tsx
const SingleElementText = () => {
- return <h1>合計¥10,000です</h1>;
+ return <h1>合計<span>¥10,000</span>です</h1>;
};
```

@[stackblitz](https://stackblitz.com/edit/vitejs-vite-bwakkm?embed=1&file=src%2Fcomponents%2FMultipleElementText.test.tsx&view=editor)

![](/images/getText/multiple-element-text-test-fail.png)

**`getByRole`、`getByText`のテストが落ちてしまいました。。** `<h1>合計<span>¥10,000</span>です</h1>`の`<h1>`要素にテキストが含まれていますが、その中にさらに`<span>`要素で囲まれた部分も含まれています。この構造がテスト結果に影響を与えています。

### `getByRole`と`getByText`が失敗する理由

#### getByRole

`getByRole`は要素のアクセシビリティロール（この場合は"heading"）を使って要素を取得し、テキスト全体を `"合計¥10,000です"` として一致させようとしますが、実際の DOM では `"合計"`, `"¥10,000"`, `"です"` が別々のテキストノードとして扱われます。また、DOM 構造上、`<h1>` 内に書かれたテキストが複数の要素（`"合計"`, `<span>¥10,000</span>`, `"です"`）に分かれているため、ブラウザは要素間の改行やインデントを空白（ホワイトスペース）として解釈します。具体的には、以下のようにブラウザはそれらのテキストをレンダリングします。

- `"合計"` の後に改行とインデントがあるため、ここに空白が挿入されます。
- `</span>` の後にインデントと改行があるため、ここにも空白が挿入されます。

![](/images/getText/multiple-element-dom.png)

その結果、実際にレンダリングされるテキストは `"合計 ¥10,000 です"` となります。このため、空白を含まない `"合計¥10,000です"` という文字列では一致しません。

#### getByText

`getByText` は、要素のテキストコンテンツを文字列として一致させることを試みます。こちらも失敗理由は同じで、`<h1>` 要素の中のテキストが `"合計"`, `"¥10,000"`, `"です"` と 3 つの異なるテキストノードとして扱われ、そのノード間の改行やインデントが空白として認識されるため、期待するテキスト `"合計¥10,000です"` とは一致しません。

---

では、このような複数の要素で構成されたテキストをテストする場合の回避方法を見ていきましょう！

## 回避方法1: `toHaveTextContent`を使用する

複数の要素で構成されたテキストをテストするには、`getByTestId`や`render`メソッドから取得できる`container`を使って取得した要素に対して、`toHaveTextContent`を使用することが有効的です。`toHaveTextContent`は、要素のテキストコンテンツを文字列として一致させることができます。このため、`<h1>合計<span>¥10,000</span>です</h1>` のテキストを `"合計¥10,000です"` として一致させることができます。

```tsx:NestedTypography.tsx
const NestedTypography = () => {
  return (
    <h2 data-testid="nested-typography">
      合計
      <strong>¥10,000</strong>
      です
    </h2>
  );
};
```

```tsx:NestedTypography.test.tsx
import { render, screen } from '@testing-library/react';
import { NestedTypography } from './NestedTypography'

describe('NestedTypography', () => {
  test('テキストが正しいか', () => {
    const { container } = render(<NestedTypography />)
    expect(container).toHaveTextContent("合計¥10,000です")
  })
  // or
  test('テキストが正しいか', () => {
    render(<NestedTypography />)
    expect(getByTestId('nested-typography')).toHaveTextContent("合計¥10,000です")
  })
})
```

## 回避方法2: `getByText`のコールバック関数を使用する

回避方法1に加えて、以下の GitHub のディスカッションで見つけた `getByText` のコールバック関数を使用してテキストを取得する方法もあります。`getByText` のコールバック関数の第一引数は要素内のテキストコンテンツを、第二引数はその要素自体を取得します。

<!-- prettier-ignore -->https://github.com/testing-library/dom-testing-library/issues/410#issuecomment-1536238708<!-- prettier-ignore -->

```tsx:NestedTypographyWithButton.tsx
const NestedTypographyWithButton = () => {
  return (
    <>
      <h2>
        合計
        <strong>¥10,000</strong>
        です
      </h2>
      <button>確定！</button>
    </>
  )
}
```

```tsx:NestedTypographyWithButton.test.tsx
import { render, screen } from '@testing-library/react';
import { NestedTypographyWithButton } from './NestedTypographyWithButton'

describe('NestedTypographyWithButton', () => {
  test('テキストが正しいか', () => {
    render(<NestedTypographyWithButton />)

    expect(screen.getByText((content, node) => {
      const hasText = (node) => node.textContent === "合計¥10,000です";
      const nodeHasText = hasText(node);
      const childrenDontHaveText = Array.from(node.children).every(
      (child) => !hasText(child)
    );

    return nodeHasText && childrenDontHaveText;
    })).toBeInTheDocument()
  })
})
```

`hasText` 関数は、引数として渡されたノードが指定したテキストを持っているかどうかを判定します。`nodeHasText` はノード自体が指定したテキストを持っているかどうか、`childrenDontHaveText` はノードの子要素がそのテキストを持っていないかどうかを判定します。最終的に、`nodeHasText` と `childrenDontHaveText` の両方が `true` であれば、`getByText` は指定したテキストを持つ要素を返します。これにより、親要素が正しいテキストを持ち、子要素がそれを持っていないかどうかを確認することができます！

または、関数化して呼び出すのも 🙆

```ts:textContentMatcher.ts
function textContentMatcher(textMatch: string | RegExp) {
  const hasText =
    typeof textMatch === "string"
      ? (node: Element) => node.textContent === textMatch
      : (node: Element) => textMatch.test(node.textContent);

  const matcher = (_content: string, node: Element) => {
    if (!hasText(node)) {
      return false;
    }
    return Array.from(node?.children || []).every((child) => !hasText(child));
  };

  matcher.toString = () => `textContentMatcher(${textMatch})`;

  return matcher;
}
```

```diff tsx:NestedTypographyWithButton.test.tsx
import { render, screen } from '@testing-library/react';
import { NestedTypographyWithButton } from './NestedTypographyWithButton'

describe('NestedTypographyWithButton', () => {
  test('テキストが正しいか', () => {
    render(<NestedTypographyWithButton />)

-   expect(screen.getByText((content, node) => {
-     const hasText = (node) => node.textContent === "合計¥10,000です";
-     const nodeHasText = hasText(node);
-     const childrenDontHaveText = Array.from(node.children).every(
-       (child) => !hasText(child)
-     );
-
-     return nodeHasText && childrenDontHaveText;
-   })).toBeInTheDocument()
+   expect(screen.getByText(textContentMatcher('合計¥10,000です'))).toBeInTheDocument()
  })
})
```

## まとめ

以上が、React Testing Libraryで複数の要素で構成されたテキストをテストする方法でした。`toHaveTextContent`や`getByText`のコールバック関数を使用することで、複数の要素で構成されたテキストをテストすることができます。個人的には回避方法2のほうがしっくりきたので、回避方法2で対応しました！

これらの方法を使って、テストを書く際に手こずることなく、スムーズにテストを書くことができるようになると思います。
