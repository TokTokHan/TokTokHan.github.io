---
layout: post
title: "Next와 Mobx 정리 - [app & document 커스텀]"
author: jangwon.seo
date: 2020-07-15 18:00
tags: [react, next, mobx]
---

[Next.js app & document 커스텀]
Next.js에 대한 이해와 Mobx와 같은 라이브러리들을 사용해 웹을 개발하기 위한 기본 세팅 작업을 boilerplate로 만들어보는 과정에 대한 내용을 정리했습니다.

## 목차

1. [Next.js 란?](https://tech.toktokhan.dev/2020/07/01/next-mobx-boiler-1/)
2. [Next.js 시작하기](https://tech.toktokhan.dev/2020/07/05/next-mobx-boiler-2/)
3. [Next.js 라이브러리 세팅](https://tech.toktokhan.dev/2020/07/10/next-mobx-boiler-3/)
4. **Next.js app & document 커스텀**

## Next.js app & document 커스텀

[[Github Project 링크](https://github.com/wkddnjset/next-mobx-boilerplate)]

### `App` 커스텀

`App`은 가장 최상위 컴포넌트 입니다. 아래 사항들을 커스텀할때 사용할 수 있습니다.

- 페이지 별로 잦은 Layout 변경이 필요할 경우
- 페이지가 이동 될 때 마다 State 값이 유지되어야 할 경우
- 에러 처리를 커스텀 하고 싶을 경우
- 페이지에 추가적인 데이터를 입력하고 싶을 경우
- global CSS를 추가할 경우

`/pages/_app.js`를 생성하면 커스텀을 진행할 수 있습니다.

```javascript
import * as React from "react";
import { AppProps } from "next/app";
import { Provider } from "mobx-react";
import { createGlobalStyle } from "styled-components";
import ScrollToTop from "components/ScrollToTop";
import stores from "stores";

// CSS Reset Code
const GlobalStyle = createGlobalStyle`
  ...
`;

const MyApp = ({ Component, pageProps }: AppProps) => (
  <Provider {...stores}>
    <ScrollToTop>
      {/* <GlobalStyle /> */}
      <title>Next.js & Mobx BoilerPlate</title>
      <Component {...pageProps} />
    </ScrollToTop>
  </Provider>
);

MyApp.getInitialProps = async ({ Component, ctx }) => {
  let pageProps = {};
  if (Component.getInitialProps) {
    pageProps = await Component.getInitialProps(ctx);
  }
  return { pageProps };
};

export default MyApp;
```

저는 `Mobx Provider`, `global CSS`에 대한 설정, `ScrollToTop` 컴포넌트를 이용해 페이지 이동시 스크롤 이동에 대한 컨트롤을 커스텀 하였습니다.

### `Document` 커스텀

`Document` 커스텀은 `<html>`과 `<body>` 태그를 커스텀 할때 대부분 사용 됩니다. 해당 기능이 필요한 이유는 Next.js page는 주변 파일들에서 정의한 `마크업`을 무시하기 때문 이라고 합니다.

또한 `getInitialProps` 기능을 포함하고 있어 비동기로 서버사이드 렌더링된 데이터를 불러올때 사용됩니다.

`/pages/_document.js`를 생성하면 커스텀을 진행할 수 있습니다.

```javascript
import * as React from "react";
import { GetStaticProps } from "next";
import Document, { Head, Main, NextScript } from "next/document";
import { ServerStyleSheet } from "styled-components";

const SITE_NAME = "SITE_NAME";
const SITE_TITLE = "SITE_TITLE";
const SITE_DESCRIPTION = "SITE_DESCRIPTION";
const SITE_IMAGE = "SITE_IMAGE";

function Document(props) {
  return (
    <html lang="ko">
      <Head>
        <meta
          name="viewport"
          content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no"
        />
        <meta property="og:type" content="website" />
        <meta property="og:site_name" content={SITE_NAME} />
        <meta property="og:title" content={SITE_TITLE} />
        <meta property="og:description" content={SITE_DESCRIPTION} />
        <meta property="og:image" content={SITE_IMAGE} />
        <meta name="twitter:card" content="summary_large_image" />
        <meta name="twitter:site" content={SITE_NAME} />
        <meta name="twitter:title" content={SITE_TITLE} />
        <meta name="twitter:description" content={SITE_DESCRIPTION} />
        <meta property="twitter:image" content={SITE_IMAGE} />
        <meta
          name="format-detection"
          content="telephone=no, address=no, email=no"
        />
        <link rel="shortcut icon" href="/static/favicon.ico" />
        {props.styleTags}
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </html>
  );
}

Document.getInitialProps = async ({ renderPage }) => {
  const sheet = new ServerStyleSheet();
  const page = renderPage((App) => (props) =>
    sheet.collectStyles(<App {...props} />)
  );
  const styleTags = sheet.getStyleElement();
  return { ...page, styleTags };
};

export default Document;
```

저는 `styled-component`에서 적용된 스타일들을 `getInitialProps`를 사용해서 페이지가 로드되기 전에 서버에서 해당 데이터를 불러와서 스타일적용을 시키기 위해서 `Document` 커스텀을 사용했습니다.

## 참고 사이트

- [Next.js 사이트](https://nextjs.org/docs/getting-started)
- [Next.js with Mobx](https://www.themikelewis.com/post/nextjs-with-mobx)
- [[똑똑한개발사 웹사이트](https://toktokhan.dev/)]
