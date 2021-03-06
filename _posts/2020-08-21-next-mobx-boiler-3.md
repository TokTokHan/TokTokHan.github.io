---
layout: post
title: "Next와 Mobx 정리 - [라이브러리 세팅]"
author: jangwon.seo
date: 2020-08-21 18:12
tags: [react, next, mobx, frontend]
---

[Next.js 라이브러리 세팅]
Next.js에 대한 이해와 Mobx와 같은 라이브러리들을 사용해 웹을 개발하기 위한 기본 세팅 작업을 boilerplate로 만들어보는 과정에 대한 내용을 정리했습니다.

## 목차

1. [Next.js 란?](https://tech.toktokhan.dev/2020/07/01/next-mobx-boiler-1/)
2. [Next.js 시작하기](https://tech.toktokhan.dev/2020/07/05/next-mobx-boiler-2/)
3. **Next.js 라이브러리 설치 및 Bebel 세팅**
4. [Next.js app & document 커스텀](https://tech.toktokhan.dev/2020/07/15/next-mobx-boiler-4/)

## Next.js 라이브러리 세팅

[[Github Project 링크](https://github.com/wkddnjset/next-mobx-boilerplate)]

### 라이브러리 설명

- babel-plugin-module-resolver

> 아래 코드 처럼 Import path를 간단하게 만들어주는 바벨 라이브러리 입니다.

```
// Use this:
import MyUtilFn from 'utils/MyUtilFn';
// Instead of that:
import MyUtilFn from '../../../../utils/MyUtilFn';
```

- styled-components [[참고](https://styled-components.com/)]
- babel-plugin-styled-components

> styled-component를 컴파일 해주는 바벨 라이브러리 입니다.

- mobx [[참고](https://mobx.js.org/README.html)]
- @babel/plugin-proposal-decorators

> [Mobx에서 사용]데코레이터 기능을 사용할 수 있도록 설정해주는 바벨 라이브러리 입니다.

- @babel/plugin-proposal-class-properties

> [Mobx에서 사용]정적 클래스의 속성을 초기화 된 속성 값으로 변환하는 바벨 라이브러리 입니다.

### babel-plugin-module-resolver

- Install

```bash
$ yarn add -D babel-plugin-module-resolver
```

- Babel Setting

`/.babelrc`

```json
{
  "presets": ["next/babel"],
  "plugins": [
    [
      "module-resolver",
      {
        "root": ["./"],
        "alias": {
          "api": "./api/*",
          "component": "./component/*",
          "containers": "./containers/*"
        }
      }
    ]
  ]
}
```

### styled-components & babel-plugin-styled-components

- Install

```bash
$ yarn add styled-components
$ yarn add -D babel-plugin-styled-components
```

- Babel Setting

`/.babelrc`

```json
{
  "plugins": [
    ["module-resolver", {
      ...
    }],
    // 추가
    ["babel-plugin-styled-components"]
  ]
}
```

### mobx

- Install

```bash
$ yarn add mobx mobx-react
$ yarn add -D @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators
```

- Babel Setting

`/.babelrc`

```json
{
  "plugins": [
    ...// 추가
    ["@babel/plugin-proposal-decorators", { "legacy": true }],
    ["@babel/plugin-proposal-class-properties", { "loose": true }]
  ]
}
```

## 참고 사이트

- [Next.js 사이트](https://nextjs.org/docs/getting-started)
- [Next.js with Mobx](https://www.themikelewis.com/post/nextjs-with-mobx)
- [[똑똑한개발사 웹사이트](https://toktokhan.dev/)]
