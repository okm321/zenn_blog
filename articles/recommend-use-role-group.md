---
title: "data-testidをrole='group'に置き換えてみたら、テスタビリティとアクセシビリティが向上した話"
emoji: "🦑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["jest", "react", "testinglibrary", "a11y"]
published: false
---

# はじめに

テストを書いている時に、よく「このコンポーネントが表示されているか確かめたいな」、「この範囲内にある DOM に特定の要素があるか」などを検証したい時が多々あると思います。その時に、`data-testid`を使用していた箇所を`role='group'`指定することによって、テスタビリティが向上し、さらにマシンリーダビリティも向上して、アクセシビリティの改善に繋がったことについて書いていこうと思います！

# Testing Library の基本理念

テストには、`testing-library`を使用しています。その`testing-library`の思想として、**ユーザーの視点に立ったテストを書くこと**が挙げられています。

> We try to only expose methods and utilities that encourage you to write tests that closely resemble how your web pages are used.

また、`testing-library`には[クエリの優先度](https://testing-library.com/docs/queries/about#priority)が存在し、次のように決められています。

1. 視覚的なユーザーや支援技術を使用するユーザーの体験を反映しているクエリ（`getByRole`や`getByLabelText`）
2. HTML5 や ARIA 規格に準拠したセレクターを使用するクエリ（`getByAltText`）
3. 他の方法で取得できない場合や意味がない場合に使用するクエリ（`getByTestId`）

詳しいクエリ優先度については、以下の記事が大変参考になったので、そちらを覗いてみてください！
https://zenn.dev/tnyo43/articles/39e4caa321d0aa

# 具体的な実装例

以下のような、ラジオボタンの選択項目によって表示されるコンポーネントが切り替わる UI のテストを行います。
![](/images/radio_food.gif)

コードは以下のようにしています。

```tsx:Fruits.tsx
import { FC } from 'react';

export const Fruits: FC = () => {
  return (
    <div>
      <ul
        style={{
          listStyle: 'none',
          margin: 0,
          padding: 0,
        }}
      >
        <li>りんご</li>
        <li>ばなな</li>
        <li>みかん</li>
      </ul>
    </div>
  );
};

```

```tsx:DisplayGroupSwitcher.tsx
import { FC } from 'react';
import { Fruits } from './Fruits';
import { Vegetable } from './Vegetable';
import { Protein } from './Protein';

type DisplayGroupSwitcherProps = {
  selectedValue: string;
};

export const DisplayGroupSwitcher: FC<DisplayGroupSwitcherProps> = ({
  selectedValue,
}) => {
  switch (selectedValue) {
    case 'fruits':
      return <Fruits />;
    case 'vegetable':
      return <Vegetable />;
    case 'protein':
      return <Protein />;
    default:
      return null;
  }
};

```

```tsx:GroupSelectParts.tsx
import { Box } from '@mui/material';
import { InputRadioGroup } from './InputRadioGroup';
import { FC, useState } from 'react';
import { DisplayGroupSwitcher } from './DisplayGroupSwitcher';

const RADIO_OPTIONS = [
  {
    id: 'fruits',
    text: '果物',
  },
  {
    id: 'vegetable',
    text: '野菜',
  },
  {
    id: 'protein',
    text: 'タンパク質',
  },
];

export const GroupSelectParts: FC = () => {
  const [value, setValue] = useState('fruits');

  return (
    <Box p={1}>
      <InputRadioGroup
        label="食べ物"
        value={value}
        items={RADIO_OPTIONS}
        onChange={(e) => setValue(e.target.value)}
      />
      <DisplayGroupSwitcher selectedValue={value} />
    </Box>
  );
};

```

## テストの実装

このコードでラジオボタンで果物を選択した時に、`Fruits.tsx`が表示されているかをテストするのに、よくやる手法として`data-testid`を付与し、それを元に`getByTestId`でコンポーネントを取得する方法があると思います。

```diff tsx:Fruits.tsx
  return (
-    <div>
+    <div data-testid='fruits'>
      <ul
        style={{
          listStyle: 'none',
          margin: 0,
          padding: 0,
        }}
      >
        <li>りんご</li>
        <li>ばなな</li>
        <li>みかん</li>
      </ul>
    </div>
  );
};
```

```tsx:GroupSelectParts.test.tsx
import userEvent from '@testing-library/user-event';
import { render, screen } from '@testing-library/react';
import { GroupSelectParts } from './GroupSelectParts';

const user = userEvent.setup();

describe('GroupSelectParts', () => {
  describe('果物を選択した場合', () => {
    test('果物の種類が表示されるか', async () => {
      render(<GroupSelectParts />);

      await user.click(screen.getByRole('radio', { name: '果物' }));
      expect(screen.getByTestId('fruits')).toBeInTheDocument();
    });
  });
});

```

自分はよく上記の方法を用いてコンポーネントの表示/非表示の分岐のテストを書いていました。

# data-testid のデメリット

テスト実行自体は問題ないのですが、`data-tessid`を使用することで以下のようなデメリットが生じる場合があります。

- **実装の詳細に依存してしまう**
  テストが要素の具体的な実装に依存してしまうことにより、リファクタリングやスタイルの変更など、実際にはユーザーに影響しない変更でもテストが失敗してしまう可能性がある
- **アクセシビリティの考慮不足**
  ユーザーが要素をどのように見つけるかを模倣しないため、実際のユーザー体験に対するテストが不十分になる可能性がある
- **保守性の低下**
  `data-testid`を広範囲に使用してしまうと、HTML やコンポーネントの変更時に多数のテストを更新する必要が生じ、保守性が低下する可能性がある

以上のようなデメリットがあるので、他のどのクエリでも要素を取得できない時以外は個人的にあまり使用したくありません。`testing-library`の祖、Kent C. Dodds さんのブログにも同じようなことが書いてありました。

> Sometimes you can't reliably select an element by any of the other queries. For those, it's recommended to use `data-testid`
> 他のどのクエリでも要素を選択できないことが時としてあります。そのような場合に`data-testid`を利用することが推奨されます

これに加えて、以下も述べていました。

> (though you'll want to make sure that you're not forgetting to use a proper `role` attribute or something first)
> （ただ、最初に適切な `role` 属性の指定漏れなどないかを確かめる必要があります）

https://kentcdodds.com/blog/making-your-ui-tests-resilient-to-change#whats-with-the-data-testid-query

これをヒントに、使えそうな `role` 属性がないか探してみた結果、`group`が良さそうでした！

# group ロールについて

`group`ロールは WAI-ARIA で定義されているアクセシビリティ属性の一つです。`group`ロールはユーザーに関連する複数の要素を論理的なグループとしてまとめるために使用されます。これにより、スクリーンリーダーやその他支援技術が、これらの要素がグループ化されていることをユーザーに伝えることができます！エクセルやパワポで図をグループ化するノリで使用できます。詳しい`group`ロールの説明については以下のリンクをご確認ください。

https://www.w3.org/TR/wai-aria-1.1/#group
https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/Group_Role
https://waic.jp/translations/WCAG-TECHS/ARIA17.html

# `role='group'`を使用した実装

それでは、野菜のコンポーネント（Vegetable.tsx）に`role='group'`を使用して実装してみましょう。コードは以下の通りです。

```diff tsx:Vegetable.tsx
import { FC } from 'react';

export const Vegetable: FC = () => {
  return (
    <div
+      role="group"
+      aria-labelledby="vegetable"
    >
      <ul
        style={{
          listStyle: 'none',
          margin: 0,
          padding: 0,
        }}
      >
        <li>にんじん</li>
        <li>ピーマン</li>
        <li>キャベツ</li>
      </ul>
    </div>
  );
};

```

コンポーネントの一番外側のタグに`role='group'`を指定し、`aria-labelledby="vegetable"`を指定します。`aria-labelledby`は指定された ID を持つ要素の内容をこのグループのラベルとして読み上げる役割を持ちます。ここでは、ラジオボタンに渡されている`id`を使用しているため、「野菜」と読み上げられるようになります。

こうすることにより、テストは以下のようにクエリ優先度高めな`getByRole`で取得することができるようになります！

```diff tsx:GroupSelectParts.test.tsx
describe('GroupSelectParts', () => {
  describe('野菜を選択した場合', () => {
    test('野菜の種類が表示されるか', async () => {
      render(<GroupSelectParts />);

      await user.click(screen.getByRole('radio', { name: '野菜' }));
+      expect(screen.getByRole('group', { name: '野菜' })).toBeInTheDocument();
    });
  });
});
```

## マシンリーダビリティも向上

適切なロール属性で実装したので、マシンリーダビリティも向上しています！`data-testid`で実装した果物と`role='group'`で実装した野菜をスクリーンリーダーの VoiceOver を使用して比較すると以下のようになります。
| 果物(data-testid) | 野菜(role='group') |
| ---- | ---- |
| ![](/images/fruits.gif) | ![](/images/vegetable.gif) |

`role='group'`を使用している「野菜、グループ」と読み上げられるようになり、ユーザーに野菜に関するグループが表示されていることが伝わります！

---

`role='group'`は`button`や`ul`タグのように自動的に付与されることはありませんが、非常に有用なロールだと思います。特に、複数の関連要素を論理的にグループ化する場合に便利です。例えば、以下のように「タンパク質」を選択した時に表示されるコンポーネント内で、肉と魚の種類を表示する際に、role='group'を使うことで、各カテゴリをグループ化できます。

```diff tsx:Protein.tsx
export const Protein: FC = () => {
  return (
    <div
+     role="group"
+     aria-labelledby="vegetable"
      style={{
        backgroundColor: 'lightsalmon',
        padding: 16,
      }}
    >
+     <div role="group" aria-labelledby="meat">
        <h3
+         id="meat"
          style={{
            margin: 0,
          }}
        >
          肉
        </h3>
        <ul
          style={{
            listStyle: 'none',
            margin: 0,
            padding: 0,
            marginLeft: 16,
          }}
        >
          <li>牛肉</li>
          <li>豚肉</li>
          <li>鶏肉</li>
        </ul>
      </div>
+     <div role="group" aria-labelledby="fish">
        <h3
+         id="fish"
          style={{
            margin: 0,
          }}
        >
          魚
        </h3>
        <ul
          style={{
            listStyle: 'none',
            margin: 0,
            padding: 0,
            marginLeft: 16,
          }}
        >
          <li>サーモン</li>
          <li>マグロ</li>
          <li>鯖</li>
        </ul>
      </div>
    </div>
  );
};

```

テストもグループ単位で存在チェックができるので、複数の関連要素がある場合などで便利です。

```tsx:GroupSelectParts.test.tsx
describe('GroupSelectParts', () => {
  describe('タンパク質を選択した場合', () => {
    test('タンパク質の種類が表示されるか', async () => {
      render(<GroupSelectParts />);

      await user.click(screen.getByRole('radio', { name: 'タンパク質' }));
      expect(
        screen.getByRole('group', { name: 'タンパク質' })
      ).toBeInTheDocument();
    });

    test('魚のリストの中にマグロが存在するか', async () => {
      render(<GroupSelectParts />);

      await user.click(screen.getByRole('radio', { name: 'タンパク質' }));
      expect(
        within(
          within(screen.getByRole('group', { name: 'タンパク質' })).getByRole(
            'group',
            { name: '魚' }
          )
        ).getByText('マグロ')
      ).toBeInTheDocument();
    });
  });
});

```

VoiceOver も以下のように読み上げられるようになります。

![](/images/protein.gif)

# おわりに

以上が、`data-testid`を`role='group'`に置き換えてみたら、テスタビリティとアクセシビリティが向上した話についてでした。自分は`data-testid`を使わないことはないと思いますが、使う場面に差し掛かった時に、適切なロールが存在しないかチェックするようにしています。これからもテスタビリティ、アクセシビリティが共に高いコードをかけていければと思います。この記事で使用したサンプルコードを残しておきますので、コードの確認やスクリーンリーダー、テストランなどやってみてください！
@[stackblitz](https://stackblitz.com/edit/vitejs-vite-im7wds?embed=1&file=src%2Fcomponents%2FGroupSelectParts.test.tsx)

この記事がどなたかの一助になれば幸いです。読んでいただき感謝です 🙇

# 参考記事

https://zenn.dev/tnyo43/articles/39e4caa321d0aa
https://kentcdodds.com/blog/making-your-ui-tests-resilient-to-change#whats-with-the-data-testid-query
https://www.w3.org/TR/wai-aria-1.1/#group
https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/Group_Role
https://waic.jp/translations/WCAG-TECHS/ARIA17.html
