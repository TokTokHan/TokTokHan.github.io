---
layout: post
title: "Next와 Mobx 정리 - [알아보기]"
description: "Next.js에 대한 이해와 Mobx와 같은 라이브러리들을 사용해 웹을 개발하기 위한 기본 세팅 작업을 boilerplate로 만들어보는 과정에 대한 내용을 정리했습니다."
author: jangwon.seo
date: 2020-08-01 18:00
tags: [react, next, mobx, frontend]
---

[Next.js 알아보기]
Next.js에 대한 이해와 Mobx와 같은 라이브러리들을 사용해 웹을 개발하기 위한 기본 세팅 작업을 boilerplate로 만들어보는 과정에 대한 내용을 정리했습니다.

## 목차

1. **Next.js 란?**
2. [Next.js 시작하기](https://tech.toktokhan.dev/2020/07/05/next-mobx-boiler-2/)
3. [Next.js 라이브러리 세팅](https://tech.toktokhan.dev/2020/07/10/next-mobx-boiler-3/)
4. [Next.js app & document 커스텀](https://tech.toktokhan.dev/2020/07/15/next-mobx-boiler-4/)

## Next.js 란?

[[Github Project 링크](https://github.com/wkddnjset/next-mobx-boilerplate)]

기본적으로는 리액트를 기반으로 한 SPA를 빠르게 빌드할 수 있도록 도와주는 프레임워크 입니다. Next에서는 많은 기능을 제공해주지만 리액트에 대한 기본적인 지식이 있는 상황이라면 `client-side routing`, `page layout` 두가지만 배우면 빠르게 Next를 사용할 수 있습니다.

### SSR vs CSR

![ssrvscsr](https://raw.githubusercontent.com/wkddnjset/wkddnjset.github.io/master/_posts/images/2020-07/ssrvscsr.png)

먼저 일반적인 React 렌더링 방식은 `render()` 함수가 먼저 실행되고, 그다음에 `ComponentDidMount()` 함수를 통해 데이터를 가져와서 다시 한 번 렌더링이 됩니다. 반면에 `Next`는 `getInitialProps()` 라는 함수를 사용하기 때문에 데이터를 먼저 가져와서 한 번에 렌더링을 해줍니다.

이미지에서 `getInitialProps()`는 실제 `page` 단위로 진행하게 되며, 아래 설명할 `코드 스플리팅`으로 해당 기능이 구현되어져 있습니다.(아마도) 그렇기 때문에 page 이동 시에 `getInitialProps()`를 실행하게 되어 CSR보다 느리게 동작할 수 있습니다. CSR은 첫 페이지 로딩 때 모든 파일을 내려받은 다음 페이지 이동시 필요한 데이터만 불러서 사용하기에 초기 로딩 속도는 느리지만 page 이동시 속도가 빠릅니다.

> CSR 에서도 [코드 스플리팅](https://reactjs.org/docs/code-splitting.html)을 구현해서 초기 로딩 속도를 개선할 수 있다. [블로그 참고](https://medium.com/little-big-programming/spa-%EC%B4%88%EA%B8%B0-%EB%A1%9C%EB%94%A9-%EC%86%8D%EB%8F%84-%EA%B0%9C%EC%84%A0%ED%95%98%EA%B8%B0-9db137d25566)

### Next.js의 핵심기능

#### 코드 스플리팅

기존 react는 별도로 [셋업](https://reactjs.org/docs/code-splitting.html)을 해줘야하는 반면 Next는 `코드 스플리팅` 셋업이 되있습니다. 규모가 커지고, 용량이 커지면 로딩속도가 지연될 수 있는 문제점을 개선해서 필요에 따라 파일을 불러올 수 있게 여러 개의 파일로 분리하는 것이 `코드 스플리팅` 입니다. 폴더 구조를 보시면 `pages 폴더` 안에 각 page 즉, 라우트들이 들어가며, Components 폴더에는 React 컴포넌트들이 들어가게 되는데요. 예를 들어, 브라우저가 실행되고, 사용자가 접속을 하게 되면, 첫 페이지인 `index page`만 불러오게 되고, 그 이후에 다른 페이지로 넘어갔을 때는 해당 페이지만 불러오게 됩니다.

#### 간단한 클라이언트 사이드 라우팅 제공

- **/page/index.js**

```jsx
import Link from "next/link";
import Router from "next/router";

class Home extends Component {
  toDetail = () => {
    Router.push("/detail");
  };
  render() {
    return (
      <div>
        // Link
        <Link
          as={`/book/${this.props.book.id}`}
          href={`/book?id=${this.props.book.id}`}
        >
          <div>Book</div>
        </Link>
        // Router
        <div onClick={this.toDatil}>Book</div>
      </div>
    );
  }
}
```

사용방법은 `Router`와 `Link`를 모두 import 해서 사용할 수 있는데요. 먼저 `Link`에서는 href와 as props가 있는데 이 href는 해당 페이지로 이동해 주는 역할을 하고, as는 href의 URL을 조금 더 직관적으로 만들어주는 역할을 해줍니다. `Router`는 링크와 동일하게 해당 페이지로 이동해주는 역할을 하지만 개발자에게 조금 더 제어권을 넘겨줘서 쉽게 `Redirect`도 가능합니다.

> href props는 필수로 입력해야 하며, as는 선택적으로 사용할 수 있다. 위 예제 코드에서 유저가 보게되는 url은 `/book/1` 이지만 Next에서는 `/book?id=1`로 queryString을 받아서 처리할 수 있다.

#### 커스텀 API 서버 (as - 라우트 마스킹)

- **/server.js**

```jsx
const next = require("next");
const app = next({ dev: process.env.NODE_ENV === "production" });
const express = require("express");
const handle = app.getRequestHandeler();

app
.prepare()
.then(() => {
    const server = express();

    server.get("/book/:id", (req, res) => {
        const actualPage = "/book";
        const queryParams = { id:req.params.id };
        app.render(req, res, actualPage, queryParaams);
    })
    ...
})
```

만약에 `Link` 컴포넌트에서 as를 사용하게 되면 실제 페이지가 없는 곳에 요청하게 됩니다. 그래서 직접 커스텀 서버를 생성해서 as의 URL이 href를 바라볼 수 있게 처리를 따로 해줘야 새로 고침이나 뒤로 갔을 때도 렌더링이 가능해집니다.

#### getInitialProps()

Next의 핵심기능인 `getInitialProps` 함수를 통해 데이터를 가져올 수 있습니다. 밑에 코드를 보시면 React의 `componentDidMount`와 setState를 통해 렌더링이 두 번 되지만, Next에서는 데이터를 미리 갖고 오기 때문에 `한 번에 렌더링이 가능합니다`.

> 위에 이미지에서 확인해주세요! 비동기 처리를 통해 `componentDidMount` 한번에 렌더링으로 처리가능하다.

- **React Example**

```jsx
class Home extend Component {
    state = {
        books: []
    }
    // 렌더링이 된 다음 다시 state를 변경했기 때문에 렌더가 2번 됨
    async componentDidMount() {
        const res = await axios.get(`....url`);
        const books = res.data
        this.setState({ books })
    }
    ...
}
```

- **Next Example**

```jsx
class Home extend Component {
    static async getInitialProps () {
        const res = await axios.get(`....url`);
        const books = res.data

        return {
            books: books
        }
    }
    state = {
        books: this.props.books
    }
    ...
}
```

- getInitialProps

  - `getInitialProps`은 서버에서 가져온 데이터를 `props`로 넘기는 비동기 함수입니다.
  - 항상 서버에서 동작하며, 만약 페이지가 `<Link>`를 사용하여 호출된다면 클라이언트측에서만 호출됩니다. (라우트마스킹을 해줘야함)
  - `getInitialProps`을 부모 컴포넌트에서 진행했을 때, 자식 컴포넌트에서 사용할 수 없다. [참고](https://github.com/vercel/next.js/issues/159#issuecomment-257318970)

- componentDidMount
  - `page` 뿐만아니라 `component`에서도 사용할 수 있습니다.
  - `async/await`을 사용해서 `getInitialProps`와 유사하게 사용되어질 수 있습니다.

`getInitialProps` 와 `componentDidMount` 을 적재적소에 사용하는 것은 어려움이 있으며, [여기](https://stackoverflow.com/questions/47461803/next-js-componentwillmount-vs-getinitialprops)에서는 아래 방식으로 하고 있다고 합니다.

- `page`에서 데이터를 불러와서 props로 사용하게 될 경우 `getInitialProps`을 사용
- 자식 컴포넌트 혹은 `state`를 업데이트 하는 API를 호출하게 될 경우 `componentDidMount` 사용

## 참고 사이트

- [Next.js 사이트](https://nextjs.org/)
- [Next.js 필요한 것만 빨리 배우기
  ](https://velog.io/@jakeseo_me/Next.js-%EB%B9%A8%EB%A6%AC-%EB%B0%B0%EC%9A%B0%EA%B8%B0-y0jz9oebn0)
- [Next.js 제대로 알고 쓰자
  ](https://medium.com/@msj9121/next-js-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90-8727f76614c9)
- [[똑똑한개발자 웹사이트](https://toktokhan.dev/)]
