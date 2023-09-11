---
title: "はじめに「未経験転職は難しい。だからこそ」"
---

## この章でやること

Next.js でのデザイン実装を効率化するため、`Material-UI`と`emotion`を導入します。

## Material-UI とは？

React向けのコンポーネントライブラリの一つです。`MUI`とも表記されます。

様々な汎用性の高いコンポーネント（Webページを構成する部品）を簡単に自作アプリに盛り込むことができます。

特に今回は、当記事執筆時点での最新バージョンである`MUI@v5`を使用します。

## emotion とは？

CSS（スタイル）管理を行うためのライブラリです。

React.jsにおけるスタイある管理はいくつかの流派に分けられるのですが、 emotion はそのうちの`CSS-in-JS`に分類されます。

`CSS-in-JS`とは、CSSをJavaScriptコードの中に直接書くという流派です。スタイル（CSS）をコンポーネント（HTML構造）を1つのファイルにまとめられる（カプセル化する）ので、コンポーネントの再利用や管理やしやすくなるというメリットがあります（初学者にとっても取っ付きやすいと思います）

MUI@v5は、スタイリングのために内部で`CSS-in-JS`のためのライブラリを使用しています。

## 参考

- [【始め方】Next.js + TypeScript + ESLint + Prettier Stylelint + Material UI + Emotion + depcheck + dependabot + Vercel で快適開発](https://qiita.com/hasehiro0828/items/ef1736e871b85b039212)

- Material-UI公式リポジトリ（Next.jsへの導入サンプルも記載）

https://github.com/mui/material-ui/tree/next/examples/nextjs-with-typescript

## 手順

### インストール

`next`コンテナに入り、`MUI`と`emotion`をインストールします。

```sh:nextコンテナ
npm install @mui/material @mui/icons-material @emotion/react @emotion/styled @emotion/cache @emotion/server
```

`package.json`と`package-lock.json`が更新されていることを確認してください。

### 設定ファイルの追加

ここから、`Next.js`で`MUI@v5`と`emotion`を動作させるための諸々の設定を行います。コードの全てを理解するのは難しいと思いますので、いったんは大まかな理解で構いません。

#### theme.ts を新規作成

`theme.ts`は、`MUI`コンポーネント全体のデザインテーマを管理するファイルです。

例えば、Webアプリケーションでは、その世界観を構築するようなテーマカラーを設定することが多いです（zennであれば、青色がそれに該当するかと思います）。

そういったテーマカラーを当ファイルで定義しておくことで、`MUI`コンポーネントの配色を決定する際にそれを参照するようにすることで、アプリ全体でのカラーデザインの統一できるようになります。

`next/src/styles/theme.ts`を新規作成し、以下を記述してください。

```ts:next/src/styles/theme.ts
import { red } from '@mui/material/colors'
import { createTheme } from '@mui/material/styles'

// Create a theme instance.
const theme = createTheme({
  palette: {
    primary: {
      main: '#3EA8FF',
    },
    secondary: {
      main: '#19857b',
    },
    error: {
      main: red.A400,
    },
  },
})

export default theme
```

最も主要なテーマカラー(`primary`)を`#3EA8FF`を、次点で主要なテーマカラー(`secondary`)からを`#19857b`に設定しています（カラーコードで直接指定）。

また、エラー表示で汎用的に使用するカラーとして、`red.A400`を設定しています（`MUI`が提供するカラーライブラリから選定）。

テーマカラーは好きに選択していただいてもOKです。

#### createEmotionCache.ts を新規作成

キャッシュ関連の共通処理を記載するファイルです。ひとまず「`MUI`と`emotion`を`Next.js`で動作させるための設定ファイルのひとつ」くらいの理解度で問題ありません。

`next/src/styles/createEmotionCache.ts`を新規作成し、以下を記述してください。

```ts:next/src/styles/createEmotionCache.ts
import createCache, { EmotionCache } from '@emotion/cache'

export default function createEmotionCache(): EmotionCache {
  return createCache({ key: 'css' })
}
```

#### theme.ts と createEmotionCache.ts を _document.tsx に適用

`_document.tsx`は`Next.js`で元々用意されているファイルです。サーバー側でのみレンダリングされ、静的なHTMLマークアップと初期データを生成するために使用されます。

Webページの骨格である`<html>`、`<head>`、`<body>` タグをカスタマイズするために使用されます。通常、このファイルで設定されるタグやデータは各ページで変わらず、アプリケーション全体を通して一貫したものになります。

当ファイルに、先ほど新規作成した`theme.ts`と`createEmotionCache.ts`を読み込ませ、`Next.js`全体で`MUI`と`emotion`が動作できるようにします。

```tsx:next/src/pages/_document.tsx
/* eslint-disable @typescript-eslint/no-explicit-any */
import createEmotionServer from '@emotion/server/create-instance'
import { RenderPageResult } from 'next/dist/shared/lib/utils'
import Document, {
  Html,
  Head,
  Main,
  NextScript,
  DocumentInitialProps,
} from 'next/document'
import * as React from 'react'

import createEmotionCache from '@/styles/createEmotionCache'
import theme from '@/styles/theme'

export default class MyDocument extends Document {
  render(): JSX.Element {
    return (
      <Html lang="ja">
        <Head>
          {/* PWA primary color */}
          <meta name="theme-color" content={theme.palette.primary.main} />
          <link
            rel="stylesheet"
            href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap"
          />
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

// `getInitialProps` belongs to `_document` (instead of `_app`),
// it's compatible with static-site generation (SSG).
MyDocument.getInitialProps = async (ctx): Promise<DocumentInitialProps> => {
  // Resolution order
  //
  // On the server:
  // 1. app.getInitialProps
  // 2. page.getInitialProps
  // 3. document.getInitialProps
  // 4. app.render
  // 5. page.render
  // 6. document.render
  //
  // On the server with error:
  // 1. document.getInitialProps
  // 2. app.render
  // 3. page.render
  // 4. document.render
  //
  // On the client
  // 1. app.getInitialProps
  // 2. page.getInitialProps
  // 3. app.render
  // 4. page.render

  const originalRenderPage = ctx.renderPage

  // You can consider sharing the same emotion cache between all the SSR requests to speed up performance.
  // However, be aware that it can have global side effects.
  const cache = createEmotionCache()
  const { extractCriticalToChunks } = createEmotionServer(cache)

  ctx.renderPage = (): RenderPageResult | Promise<RenderPageResult> =>
    originalRenderPage({
      enhanceApp:
        (App: any) =>
        // eslint-disable-next-line react/display-name
        (props): JSX.Element =>
          <App emotionCache={cache} {...props} />,
    })

  const initialProps = await Document.getInitialProps(ctx)
  // This is important. It prevents emotion to render invalid HTML.
  // See https://github.com/mui-org/material-ui/issues/26561#issuecomment-855286153
  const emotionStyles = extractCriticalToChunks(initialProps.html)
  const emotionStyleTags = emotionStyles.styles.map((style) => (
    <style
      data-emotion={`${style.key} ${style.ids.join(' ')}`}
      key={style.key}
      // eslint-disable-next-line react/no-danger
      dangerouslySetInnerHTML={{ __html: style.css }}
    />
  ))

  return {
    ...initialProps,
    // Styles fragment is rendered after the app and page rendering finish.
    styles: [
      ...React.Children.toArray(initialProps.styles),
      ...emotionStyleTags,
    ],
  }
}
```

#### MUI と emotion を _app.tsx でインポート

`_app.tsx`は`Next.js`で元々用意されているファイルで、全ページで共通するレイアウトや状態を管理する役割を持っています。

ここにも MUI と emotion の設定を追加していきます。

```tsx:next/src/pages/_app.tsx
import { CacheProvider, EmotionCache } from '@emotion/react'
import CssBaseline from '@mui/material/CssBaseline'
import { ThemeProvider } from '@mui/material/styles'
import { AppProps } from 'next/app'
import * as React from 'react'

import createEmotionCache from '@/styles/createEmotionCache'
import theme from '@/styles/theme'

// Client-side cache, shared for the whole session of the user in the browser.
const clientSideEmotionCache = createEmotionCache()

interface MyAppProps extends AppProps {
  emotionCache?: EmotionCache
}

export default function MyApp(props: MyAppProps): JSX.Element {
  const { Component, emotionCache = clientSideEmotionCache, pageProps } = props
  return (
    <CacheProvider value={emotionCache}>
      <ThemeProvider theme={theme}>
        {/* CssBaseline kickstart an elegant, consistent, and simple baseline to build upon. */}
        <CssBaseline />
        <Component {...pageProps} />
      </ThemeProvider>
    </CacheProvider>
  )
}
```

#### Link コンポーネントの実装

https://github.com/mui/material-ui/blob/next/examples/nextjs-with-typescript/src/Link.tsx

`Next.js`に`MUI`を導入する際に発生する問題の一つとして、両方で`Link`コンポーネントが用意されてしまっていることで、命名の競合が発生してしまうことがあります。

`Next.js`における`Link`は、`Next.js`の機能をフルに活かした高速のページ遷移を実現できるコンポーネントで、機能として必要なものなります（今回の開発でもフルに活用します）。

一方、`MUI`における`Link`は、あくまでもデザインとしてのリンクコンポーネントを提供するだけであり、機能としては単なる`<a>`タグでのリンクになります。

`Next.js`に`MUI`を導入するケースにおいては、両者を上手いこと組み合わせた新しい`Link`コンポーネントを実装し、それを開発に使用していくアイディアがあります（機能とデザインの両方をいいとこ取りします）。

`next/src/components`配下に`Link.tsx`を作成してください。

```tsx:next/src/components/Link.tsx
/* eslint-disable @typescript-eslint/no-unused-vars */
/* eslint-disable @typescript-eslint/no-explicit-any */
import MuiLink, { LinkProps as MuiLinkProps } from '@mui/material/Link'
import clsx from 'clsx'
import NextLink, { LinkProps as NextLinkProps } from 'next/link'
import { useRouter } from 'next/router'
import * as React from 'react'

interface NextLinkComposedProps
  extends Omit<React.AnchorHTMLAttributes<HTMLAnchorElement>, 'href'>,
    Omit<NextLinkProps, 'href' | 'as'> {
  to: NextLinkProps['href']
  linkAs?: NextLinkProps['as']
  href?: NextLinkProps['href']
}

export const NextLinkComposed = React.forwardRef<
  HTMLAnchorElement,
  NextLinkComposedProps
>(function NextLinkComposed(props, ref) {
  const {
    to,
    linkAs,
    href,
    replace,
    scroll,
    passHref,
    shallow,
    prefetch,
    locale,
    ...other
  } = props

  return (
    <NextLink
      href={to}
      prefetch={prefetch}
      as={linkAs}
      replace={replace}
      scroll={scroll}
      shallow={shallow}
      passHref={passHref}
      locale={locale}
      ref={ref}
      {...other}
    ></NextLink>
  )
})

export type LinkProps = {
  activeClassName?: string
  as?: NextLinkProps['as']
  href: NextLinkProps['href']
  noLinkStyle?: boolean
} & Omit<NextLinkComposedProps, 'to' | 'linkAs' | 'href'> &
  Omit<MuiLinkProps, 'href'>

// A styled version of the Next.js Link component:
// https://nextjs.org/docs/#with-link
const Link = React.forwardRef<HTMLAnchorElement, LinkProps>(function Link(
  props,
  ref,
) {
  const {
    activeClassName = 'active',
    as: linkAs,
    className: classNameProps,
    href,
    noLinkStyle,
    role, // Link don't have roles.
    ...other
  } = props

  const router = useRouter()
  const pathname = typeof href === 'string' ? href : href.pathname
  const className = clsx(classNameProps, {
    [activeClassName]: router.pathname === pathname && activeClassName,
  })

  const isExternal =
    typeof href === 'string' &&
    (href.indexOf('http') === 0 || href.indexOf('mailto:') === 0)

  if (isExternal) {
    if (noLinkStyle) {
      return (
        <a
          className={className}
          href={href as string}
          ref={ref as any}
          {...other}
        />
      )
    }

    return (
      <MuiLink
        className={className}
        href={href as string}
        ref={ref}
        {...other}
      />
    )
  }

  if (noLinkStyle) {
    return (
      <NextLinkComposed
        className={className}
        ref={ref as any}
        to={href}
        {...other}
      />
    )
  }

  return (
    <MuiLink
      component={NextLinkComposed}
      linkAs={linkAs}
      className={className}
      ref={ref}
      to={href}
      {...other}
    />
  )
})

export default Link
```

#### .babelrcの新規追加

`MUI`に関しては設定が完了していますが、最後`emotion`でCSSスタイルを定義できるための設定を行います。

`next`コンテナに入り、下記を実行してください。

```sh:nextコンテナ
npm install --save-dev @emotion/babel-plugin
```

`package.json`と`package-lock.json`が更新されていることを確認してください。

↓

`next/.babelrc`を以下のように新規作成してください。

```:next/.babelrc
{
  "presets": [
    [
      "next/babel",
      {
        "preset-react": {
          "runtime": "automatic",
          "importSource": "@emotion/react"
        }
      }
    ]
  ],
  "plugins": ["@emotion/babel-plugin"]
}
```

↓

`next/tsconfig.json`に以下を追記してください。

```diff json:next/tsconfig.json
{
  "compilerOptions": {
    .
    .
    "jsx": "preserve",
+   "jsxImportSource": "@emotion/react",
    .
    .

  },
  .
  .
}
```

## 動作確認

ここまでで設定が完了しましたので、動作確認をしながらMUIやemotionの使い方を学んでいきます。

`pages/hello_mui.tsx`を以下の通りに新規作成してください。

```tsx:next/src/pages/hello_mui.tsx
import { Button } from '@mui/material'
import type { NextPage } from 'next'

const HelloMui: NextPage = () => {
  return (
    <>
      <Button>Hello Mui@v5!</Button>
    </>
  )
}

export default HelloMui
```

nextサーバーが起動している状態で、 http://localhost:8000/hello_mui にアクセスします。

```sh:nextコンテナ
# nextサーバー起動
npm run dev
```

以下の画面が表示されていればOKです。また、背景と同化して一見分かりにくいですが、触ってみるとクリックができるボタンであることが分かると思います。

![](https://storage.googleapis.com/zenn-user-upload/d2ed3b1e2215-20230628.png)

## Material-UIの使い方

先ほどのコードを見てみると、`@mui/material`ライブラリから、`Button`というReactコンポーネントをインポートしています。

```tsx
import { Button } from '@mui/material'
```

Material-UIは、ボタンに代表されるようないくつかの汎用的なコンポーネントを提供してくれるライブラリです。自力でデザインを実装をしなくても、Material-UIが提供してくれるコンポーネントをうまく組み合わせることで、かなり本格的なデザインの画面を作ることができます。

`Button`をもう少し触ってみましょう。例えば以下のようにして props に値を渡してあげることで、デザインを切り替えることができます。

```tsx:
<Button variant="contained">Hello Mui@v5!</Button>
```

![](https://storage.googleapis.com/zenn-user-upload/6a940c74eb7a-20230628.png)

```tsx:
<Button variant="outlined">Hello Mui@v5!</Button>
```

![](https://storage.googleapis.com/zenn-user-upload/dbd2bf2bd196-20230628.png)

```tsx:
<Button variant="contained" color="error">
  Hello Mui@v5!
</Button>
```

![](https://storage.googleapis.com/zenn-user-upload/28dea25fbfec-20230628.png)

数えきれないほどのデザインパターンが存在しており、その内容は公式ドキュメントから調べることができます。

- [MUI公式ドキュメント Button](https://mui.com/material-ui/react-button/)

Material-UIでどれくらいのカスタマイズ性が用意されているのか、いきなりその全てを把握することは難しいので、ここは触りながらだんだんと慣れていきましょう。少なくとも現時点では、**propsに特定の値を渡すことでデザインを切り替えられる**ということだけ分かっていればOKです。

### sxプロパティによるスタイル定義

また、用意されたデザインから外れて、独自にスタイルを適用したい場合は`sx`プロパティを使用します。

```tsx:
      <Button
        variant="contained"
        sx={{ p: 6, ml: 2, mt: 3, color: 'white', textTransform: 'none' }}
      >
        Hello Mui@v5!
      </Button>
```

![](https://storage.googleapis.com/zenn-user-upload/08e83400beca-20230806.png)

デフォルトで設定されていたスタイルを上書きして、独自デザインに仕上げることができました。

`sx`プロパティにおいては、`padding`と`margin`は省略表記が設定されています。

`p`が`padding`、`m`が`margin`、各方向は`t`(top)、`r`(right)、`b`(botom)、`l`（left）です。また、数値は1単位あたり`8px`を表しています。

詳しくは以下を参照ください。

- [MUI公式ドキュメント Spacing](https://mui.com/system/spacing/)

さらに、**sxプロパティ**の優秀なところは、MUIの世界で定義されたブレイクポイントに応じたレスポンシブデザインを簡単に実現できる点です。先の`<Button>`を、以下のように書き換えてください。

```tsx:
      <Button
        variant="contained"
        sx={{
          p: 6,
          ml: 2,
          mt: 3,
          color: { xs: 'white', md: 'red' },
          textTransform: 'none',
        }}
      >
        Hello Mui@v5!
      </Button>
```

`color: { xs: 'white', md: 'red' }`が、ウィンドウサイズに応じたスタイルの切り替えを表現しています。MUIにおいては、`xs`、`sm`、`md`、`lg`、`xl`がブレイクポイントを表現する単語として定義されています。

- xs, extra-small: 0px
- sm, small: 600px
- md, medium: 900px
- lg, large: 1200px
- xl, extra-large: 1536px

https://mui.com/material-ui/customization/breakpoints/

つまり`color: { xs: 'white', md: 'red' }`は、ウィンドウサイズが0~899pxの範囲では`color: "white"`、900px~の範囲では`color: "red"`として振る舞います。

![](https://storage.googleapis.com/zenn-user-upload/10d098b02039-20230806.png)

![](https://storage.googleapis.com/zenn-user-upload/1443cf74aa18-20230806.png)

## emotion によるスタイル定義

emotion を使うことで、スタイルを複数箇所で再利用できる形に共通化することができます。

例えば、複数のMUIコンポーネント`<Button>`に対して、同じスタイルを適用させたい場合、ひとつひとつの`sx`プロパティ全てに同じ定義を記述することでも実装は行えます。

```tsx:next/src/pages/hello_mui.tsx
const HelloMui: NextPage = () => {
  return (
    <>
      <Button variant="contained" sx={{ p: '24px' }}>
        Button1
      </Button>
      <Button variant="outlined" sx={{ p: '24px' }}>
        Button2
      </Button>
      <Button variant="contained" color="error" sx={{ p: '24px' }}>
        Button3
      </Button>
    </>
  )
}

export default HelloMui
```

![](https://storage.googleapis.com/zenn-user-upload/5adf9b9975d6-20230628.png)

emotionを利用することで、以下のように共通化することができます。

```tsx:next/src/pages/hello_mui.tsx
import { css } from '@emotion/react'
import { Button } from '@mui/material'
import type { NextPage } from 'next'

const buttonCss = css({
  padding: '24px',
})

const HelloMui: NextPage = () => {
  return (
    <>
      <Button variant="contained" css={buttonCss}>
        Button1
      </Button>
      <Button variant="outlined" css={buttonCss}>
        Button2
      </Button>
      <Button variant="contained" color="error" css={buttonCss}>
        Button3
      </Button>
    </>
  )
}

export default HelloMui
```

`@emotion/react`ライブラリから`css`関数をインポートし、`buttonCss`を定義しています。そしてそれを、`css`プロパティにセットしています。

こちらでも同じスタイルが当たっていることが確認できると思います。

![](https://storage.googleapis.com/zenn-user-upload/5adf9b9975d6-20230628.png)

## 当開発アプリでのCSS戦略

まとめると、今回のアプリにおいてはCSSを設定する方法が二つ存在することになります。

1. MUIの**sxプロパティ**
2. emotionの**cssメソッド**

どちらを利用してもおおよそのスタイル定義は実現できるはずです。それ故に、両者の使い分けのルールを決めておくことで、コーディングの一貫性を損なわない開発が行えます（チーム開発においては、そのルールを明文化することが極めて重要とされています）

今回は、以下のルールで両者を使い分けていこうと思います。

- 原則、**sxプロパティ**でスタイル定義を行う。
- 複数の React コンポーネントで共通利用することが想定される一部スタイルのみ、**emotion**を利用する。

基本的には、MUIの**sxプロパティ**を活用してスタイル定義を進めていくことにします。理由は、以下の点を**sxプロパティ**のメリットとして重要視したためです。

- 「JSX構造」と「スタイル定義」の定義場所を一致させられるので、初学者にとって直感的で分かりやすいデザイン実装ができる。
- MUIのブレイクポイントを利用したレスポンシブデザインが手軽に実装できる。

一方、複数の React コンポーネントが何度も定義することが分かっているような一部のスタイルについては、**emotion**を用いたスタイルの共通化を行います。

複数の React コンポーネントで利用する前提なので、定義場所は特定の React コンポーネントではなく、独立したスタイルファイルとします。これについては、実際に利用する場面で改めて説明をしていきます。
