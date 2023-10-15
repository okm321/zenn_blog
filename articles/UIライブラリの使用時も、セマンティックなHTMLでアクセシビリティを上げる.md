---
title: "UIライブラリの使用時も、セマンティックなHTMLでアクセシビリティを上げる"
emoji: "🫶"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["react", "mui", "アクセシビリティ"]
published: false
---

お疲れ様です。最近めっきり涼しくなってきてテンション爆上がりです。

# はじめに

最近、所属している会社で[Web アプリケーションアクセシビリティ ── 今日から始める現場からの改善 (WEB+DB PRESS plus)](https://www.amazon.co.jp/Web%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E2%94%80%E2%94%80%E4%BB%8A%E6%97%A5%E3%81%8B%E3%82%89%E5%A7%8B%E3%82%81%E3%82%8B%E7%8F%BE%E5%A0%B4%E3%81%8B%E3%82%89%E3%81%AE%E6%94%B9%E5%96%84-WEB-DB-PRESS-plus/dp/4297133660)の輪読会をしたり、Web アクセシビリティに造詣が深いエンジニアの方が参画してくれたおかげで。自分自身もアクセシビリティについて考える機会が増えてきました。そのこともあってか、まだまだアクセシビリティ初心者の私でも業務中に『あ、こうするだけでアクセシビリティが改善されるんや！』と体験したことがあったので、そのことについてメモがてら書き残そうと思います。

# Web アクセシビリティとは

Web アクセシビリティを簡単に説明しますと、ウェブサイトやウェブアプリケーションが全ての人々、特に視覚障害・聴覚障害・運動障害・認知障害などのような障害を持った人々や高齢者にとっても利用しやすくなるように設計・開発されることを指します。

より詳しく知りたい方は、以下のリンクの **Web エンジニアとしていま知っておきたい Web アクセシビリティ** をご覧ください。自分もとても参考になりました。
https://zenn.dev/ymrl/articles/7f41ad2f39f714

それか、[Web アプリケーションアクセシビリティ ── 今日から始める現場からの改善 (WEB+DB PRESS plus)](https://www.amazon.co.jp/Web%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E2%94%80%E2%94%80%E4%BB%8A%E6%97%A5%E3%81%8B%E3%82%89%E5%A7%8B%E3%82%81%E3%82%8B%E7%8F%BE%E5%A0%B4%E3%81%8B%E3%82%89%E3%81%AE%E6%94%B9%E5%96%84-WEB-DB-PRESS-plus/dp/4297133660)を読んでみることをお勧めします。

:::message
本記事では、スクリーンリーダーを使用した際に特に体感できる Web アクセシビリティの改善点に焦点を当てています
:::

次章からはどのように、改善できたかを見ていきます。

# 改善前の実装内容

React で テキストフィールドを動的に増やしたり、減らしたりできる機能を実装していました。UI ライブラリには[Mui](https://mui.com/material-ui/)、フォームライブラリに[React Hook Form](https://react-hook-form.com/)を使用しています。コードは以下です。

```tsx:RhfTextFieldItem.tsx
import { FC } from "react";
import { Stack, TextField, IconButton } from "@mui/material";
import { Controller, Control } from "react-hook-form";
import { TrashCan as TrashCanIcon } from "mdi-material-ui";
import { FormType } from "../App";

type Props = {
  control: Control<FormType>;
  index: number;
  onRemove: () => void;
}

export const TextFieldItem: FC<Props> = ({
  control,
  index,
  onRemove
}) => {
  return (
    <Stack spacing={2} direction="row">
      <Controller
        name={`textFields.${index}.text`}
        control={control}
        render={({ field }) => {
          return (
            <TextField
              inputRef={field.ref}
              value={field.value}
              onBlur={field.onBlur}
              onChange={field.onChange}
              name={field.name}
            />
          );
        }}
      />
      <IconButton sx={{ alignSelf: "center" }} onClick={() => onRemove()}>
        <TrashCanIcon />
      </IconButton>
    </Stack>
  )
}
```

```tsx:RhfTextFieldList.tsx
import { FC } from "react";
import { Stack, Button } from "@mui/material";
import { useFieldArray, Control } from "react-hook-form";
import { FormType } from "../App";
import { RhfTextFieldItem } from "./RhfTextFieldItem";

type Props = {
  control: Control<FormType>;
};

export const RhfTextFieldList: FC<Props> = ({ control }) => {
  const { fields, append, remove } = useFieldArray({
    control,
    name: "textFields"
  });

  return (
    <>
      <Stack spacing={2}>
        {fields.map((field, index) => {
          return (
            <RhfTextFieldItem
              key={field.id}
              control={control}
              index={index}
              onRemove={() => remove(index)}
            />
          )
        })}
      </Stack>
      <Button
        sx={{
          marginTop: 2
        }}
        onClick={() => append({ text: "" })}
      >
        追加
      </Button>
    </>
  )
}

```

```tsx:App.tsx
import { useForm } from "react-hook-form";
import { Box } from "@mui/material";
import { RhfTextFieldList } from "./components/RhfTextFieldList";

export type FormType = {
  textFields: {
    text: string;
  }[];
};

export default function App() {
  const { control } = useForm<FormType>({
    defaultValues: {
      textFields: [
        {
          text: "サンプルテキスト"
        },
        {
          text: "サンプルテキスト"
        },
        {
          text: "サンプルテキスト"
        }
      ]
    }
  });
  return (
    <div className="App">
      <Box sx={{ p: 2 }}>
        <RhfTextFieldList control={control} />
      </Box>
    </div>
  );
}
```

`RhfTextFieldItem`は、テキストフィールドと削除アイコンをまとめたコンポーネントです。このコンポーネントは、ユーザーが入力したテキストを表示し、そのテキストを削除するためのアイコンボタンを提供します。`React-Hook-Form` の `Controller` を使用して、フォームの値の管理を効率的に行っています。

`RhfTextFieldList`は、複数の`RhfTextFieldItem`コンポーネントをリストとして表示するコンポーネントです。このコンポーネントは、`useFieldArray`フックを使用して、動的にテキストフィールドのリストを管理します。また、新しいテキストフィールドを追加するための「追加」ボタンも提供しています。

`App.tsx`ファイルで`RhfTextFieldList`を呼び出すようにしています。

実際に動くものを codesandbox で用意しまいた。
@[codesandbox](https://codesandbox.io/embed/muiakusesibiriteiyi-shi-nasi-49k9mw?fontsize=14&hidenavigation=1&theme=dark)

これで機能は満たしていますが、**これだとアクセシビリティ的に不十分なのです。**

# アクセシビリティ的な問題点

では、先の実装のどういう部分がアクセシビリティ的に不十分なのか、見ていきましょう。

## スクリーンリーダーの起動

私は Mac を使っているので、スクリーンリーダーに VoiceOver を使用します。VoiceOver の起動はデフォルトだと `⌘+F5キー`です。諸々の操作方法は以下の Qiita の記事がとても参考になったので、ご覧ください。
https://qiita.com/tsmd/items/3d8e265ae60dfb1e187d

## スクリーンリーダーで移動してみる

では、先ほど実装したものをスクリーンリーダーで移動してみます。
![voiceOverで移動してみた](/images/no_a11y.gif)

## 問題点

一見、問題ないように思えるかもしれませんが、以下の 2 点が問題として挙げられます。

1. **スクリーンリーダーのカーソルが何個目のテキストフィールドに当たっているか認識できない**
2. **そのボタンが何を意図するボタンなのかが理解することができない**

---

これらの対策は難しいように思うかもしれませんが、セマンティックな HTML を意識するだけで大幅に改善されます。

# セマンティックな HTML とは

セマンティックは「意味や目的を持たせる」という意味で使用されます。要するに、セマンティックな HTML とは、**<h1>や<ul>のような HTML タグを正しく使い分ける** ことを指します。セマンティックな HTML が適切に使用されていると、スクリーンリーダーはページの構造やコンテンツの意味を正確に伝えることができます。

# 改善内容

先の章で挙げた問題点をどのように改善していくかを見ていきましょう！

## 1.スクリーンリーダーのカーソルが何個目のテキストフィールドに当たっているか認識できない

`RhfTextFieldList`のコードを確認します。

```tsx:RhfTextFieldList.tsx
export const RhfTextFieldList: FC<Props> = ({ control }) => {
  // ...
  return (
    <>
      <Stack spacing={2}>
        {fields.map((field, index) => {
          return (
            <RhfTextFieldItem
              key={field.id}
              control={control}
              index={index}
              onRemove={() => remove(index)}
            />
          );
        })}
      </Stack>
      {/* ... **/}
    </>
  )
}
```

Mui の[Stack](https://mui.com/material-ui/react-stack/)コンポーネントは、要素を垂直または水平に一定のスペーシングで配置できる便利なコンポーネントです。これを使って`RhfTextFieldItem`を垂直に一定間隔で配置しています。ただ、このコンポーネントは画像のようにデフォルトだと **`<div>`要素としてレンダリングされます。**。
![Stackを使用した際の、devtool](/images/stack_devtool.png)

`<div>`要素でレンダリングされているので、見た目はリストのようになっていても HTML の構造的には、単純なブロックが置いてあるだけと認識されるので、スクリーンリーダーで読み上げる時に何個目のテキストフィールドかわからないのです。では、これをセマンティックな HTML に変えてみましょう！

```diff tsx:RhfTextFieldList.tsx
export const RhfTextFieldList: FC<Props> = ({ control }) => {
  // ...
  return (
    <>
-     <Stack spacing={2}>
+     <Stack
+       spacing={2}
+       component="ul"
+       sx={{
+         listStyleType: "none",
+         padding: 0,
+         margin: 0
+       }}
+     >
        {fields.map((field, index) => {
          return (
+           <li key={field.id}>
              <RhfTextFieldItem
-               key={field.id}
                control={control}
                index={index}
                onRemove={() => remove(index)}
              />
+           </li>
          );
        })}
      </Stack>
      {/* ... **/}
    </>
  )
}
```

`Stack`コンポーネントの`component`プロパティを使用することで、異なる HTML 要素や React コンポーネントに変更することができるので、リストにするために`<ul>`要素を設定し、`RhfTextFieldItem`を`<li>`要素で括ることによってリストの各項目を定義します。このようにすることで、`<ul>`要素`<li>`要素でレンダリングされます。
![Stackを<ul>でレンダリングするようにした際の、devtool](/images/stack_devtool_after.png)

これでセマンティックな HTML になりました。ここで、もう一度 voiceOver を使用してみて、改善前のものと見比べてみましょう。
| 改善前 | 改善後 |
| ---- | ---- |
| ![改善前のものでvoiceOverで移動してみた](/images/no_a11y.gif) | ![改善前のものでvoiceOverで移動してみた](/images/list_a11y.gif) |

改善後の voiceOver ではリストの場所にカーソルが入ってきた時に、**「リスト 3 項目」と読み上げられます**。そして、リストのアイテムにカーソルが入る時に、**「3 の 1」と読み上げられ、今カーソルが当たっているリストの場所がわかるようになります。** 🎉

## 2. **そのボタンが何を意図するボタンなのかが理解することができない**

`RhfTextFieldItem`の削除アイコン部分のコードを確認します。

```tsx:RhfTextFieldItem.tsx
// ...
export const RhfTextFieldItem: FC<Props> = ({ control, index, onRemove }) => {
  return (
    <Stack spacing={2} direction="row">
      {/** ... */}
      <IconButton
        sx={{ alignSelf: "center" }}
        onClick={() => onRemove()}
      >
        <TrashCanIcon />
      </IconButton>
    </Stack>
  );
};
```

`IconButton`の children に`<TrashCanIcon />`を設定しています。視覚的にはアイコンからこのアイコンをクリックしたら何が起こるか推測できますが、スクリーンリーダーのユーザーにはその情報は伝わりません。voiceOver だと **「ボタン」としか読み上げられません。**
![voiceOverでIconButtonを読み上げる時](/images/icon_button_no_a11y.png)

これをスクリーンリーダーユーザーにも伝わるようにするには、`aria-label`というものを使います。`aria-label`を簡単に説明しますと、要素の目的や機能をテキスト形式で明示的に示すためのものです。

```diff tsx:RhfTextFieldItem.tsx
// ...
export const RhfTextFieldItem: FC<Props> = ({ control, index, onRemove }) => {
  return (
    <Stack spacing={2} direction="row">
      {/** ... */}
      <IconButton
        sx={{ alignSelf: "center" }}
        onClick={() => onRemove()}
+       aria-label="削除"
      >
        <TrashCanIcon />
      </IconButton>
    </Stack>
  );
};
```

このように`aria-label`を付与すると、以下のようにスクリーンリーダーに読み上げられます。改善前と比較してみます。
| 改善前 | 改善後 |
| ---- | ---- |
| ![改善前のvoiceOverでIconButtonを読み上げる時](/images/icon_button_no_a11y.png) | ![改善後のvoiceOverでIconButtonを読み上げる時](/images/icon_button_a11y.png) |

アイコンボタンにカーソルが当たった時に、**「削除、ボタン」** と読み上げられるようになり、スクリーンリーダーで伝わるようになります！

# おわりに

以上が、アクセシビリティの改善内容です。自分の感覚ですが、UI ライブラリを使用する際、ライブラリが提供するコンポーネントをそのまま利用することで、簡単に目的のスタイルを実現できるため、セマンティックな HTML の重要性を見失いがちです。しかし、UI ライブラリを使用する時こそ、手を抜かずにセマンティックな HTML を追求することが、アクセシビリティの向上に直結すると思いました。

まだまだ、アクセシビリティについては初心者で知識が不足していますが、最低限セマンティックな HTML は目指していき、これからも学んでいけたらなと思います。

最後に、アクセシビリティを意識した codesandbox と、そうでない codesandbox を貼っておきますので、スクリーンリーダーを使って確かめてみてください！読んでいただき、ありがとうございました。

## アクセシビリティ意識なし

@[codesandbox](https://codesandbox.io/embed/muiakusesibiriteiyi-shi-nasi-49k9mw?fontsize=14&hidenavigation=1&theme=dark)

## アクセシビリティ意識あり

@[codesandbox](https://codesandbox.io/embed/muiakusesibiriteiyi-shi-ari-tyzvhj?fontsize=14&hidenavigation=1&theme=dark)

# 参考記事、書籍

https://zenn.dev/ymrl/articles/7f41ad2f39f714
https://www.amazon.co.jp/Web%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E2%94%80%E2%94%80%E4%BB%8A%E6%97%A5%E3%81%8B%E3%82%89%E5%A7%8B%E3%82%81%E3%82%8B%E7%8F%BE%E5%A0%B4%E3%81%8B%E3%82%89%E3%81%AE%E6%94%B9%E5%96%84-WEB-DB-PRESS-plus/dp/4297133660
https://qiita.com/tsmd/items/3d8e265ae60dfb1e187d
