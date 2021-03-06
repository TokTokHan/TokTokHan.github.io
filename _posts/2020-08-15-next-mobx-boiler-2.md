---
layout: post
title: "Next와 Mobx 정리 - [시작하기]"
author: jangwon.seo
date: 2020-08-15 14:48
tags: [react, next, mobx, frontend]
---

[Next.js 시작하기]
Next.js에 대한 이해와 Mobx와 같은 라이브러리들을 사용해 웹을 개발하기 위한 기본 세팅 작업을 boilerplate로 만들어보는 과정에 대한 내용을 정리했습니다.

## 목차

1. [Next.js 란?](https://tech.toktokhan.dev/2020/07/01/next-mobx-boiler-1/)
2. **Next.js 시작하기**
3. [Next.js 라이브러리 세팅](https://tech.toktokhan.dev/2020/07/10/next-mobx-boiler-3/)
4. [Next.js app & document 커스텀](https://tech.toktokhan.dev/2020/07/15/next-mobx-boiler-4/)

## Next.js 시작하기

[[Github Project 링크](https://github.com/wkddnjset/next-mobx-boilerplate)]

### 프로젝트 생성

Next.js에서도 React에서 사용하는 `creat-react-app`과 같은 `creact-next-app`이라는 CLI를 제공하고 있으므로 꼭! 사용하는 것을 권장한다.

> 물론 메뉴얼 세팅 방법이 있지만, 다루지 않았으며, 하단에 `참고 사이트`를 참고 해주세요.

```bash
$ npx create-next-app
# or
$ yarn create next-app
```

이미 공식적으로 제공되는 `boilerplate` 들이존재하고, 프로젝트를 설치 할 때 자동 세팅을 지원하는 듯 하지만, `Default Mode`를 사용해서 설치하였습니다.

### 디렉토리 구조

```
next-mobx-boilerplate
├─ node_modules
    └─ ...
├─ pages
    └─ api
        └─ hello.js
    └─ index.js
├─ public
├─ node_modules
└─ pages
```

#### Page

Next.js에서 **page**는`pages`폴더에 리액트 컴포넌트 형태로 저장되어 있으며, 각 **page**들은 파일명을 기준으로 router와 연결되어 있습니다.

> 예시로 만약 `pages/about.js` 파일을 생성하면, `/about`의 url 경로로 접속이 가능하게 됩니다.

### 동적 라우팅

- `pages/post/[pid].js`

```javascript
import { useRouter } from "next/router";

const Post = () => {
  const router = useRouter();
  const { pid } = router.query;

  return <p>Post : {pid}</p>;
};

export default Post;
```

`/post/abc`

```json
{ "pid": "abc" }
```

`/post/abc?foo=bar`

```json
{ "foo": "bar", "pid": "abc" }
```

route 파라미터와 queryString이 같은 경우
`/post/abc?pid=123`

```json
{ "pid": "abc" }
```

`pages/post/[pid]/[comment].js`

```json
{ "pid": "abc", "comment": "a-comment" }
```

#### 모든 라우트 불러오기

`pages/post/[...slug].js` 파일명을 다음과 같이 만들면, `post/a` 뿐만아니라 `post/a/b` 도 인식한다.

`post/a/b`

```json
{ "slug": ["a", "b"] }
```

### 정리

- `pages/post/create.js` - Will match `/post/create`
- `pages/post/[pid].js` - Will match `/post/1`, `/post/abc`, etc. But not `/post/create`
- `pages/post/[...slug].js` - Will match `/post/1/2`, `/post/a/b/c`, etc. But not `/post/create`, `/post/abc`

## 참고 사이트

- [Next.js 사이트](https://nextjs.org/docs/getting-started)
- [[똑똑한개발사 웹사이트](https://toktokhan.dev/)]
