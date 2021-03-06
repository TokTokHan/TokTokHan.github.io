---
layout: post
title: "Next.js 코드 컨벤션"
description: "Next.js에 대한 이해와 Mobx와 같은 라이브러리들을 사용해 웹을 개발하기 위한 기본 세팅 작업을 boilerplate로 만들어보는 과정에 대한 내용을 정리했습니다."
author: minjung.kim
date: 2020-08-30 17:45
tags: [react, next, frontend]
---

## 시작하며

해당 글은 [똑똑한개발자](https://toktokhan.dev/) github에 있는 `next-ts`을 기준으로 작성되었습니다.<br/>
VSCode 기준으로 설정 되었으며, 다른 IDE를 쓸 경우 ESLint와 Prettier가 다르게 보일 수 있습니다.

#### 공통 사용 스택

```
- Next.js, PWA, TypeScript, styled-components, Storybook
- axios
```

- 현재(글 마지막 업데이트 기준) 프로젝트 패키지 버전

```json
{
  "dependencies": {
    "@material-ui/core": "^4.11.0",
    "@types/react-select": "^3.0.26",
    "@types/react-slick": "^0.23.4",
    "axios": "^0.20.0",
    "mobx": "^6.0.4",
    "mobx-react": "^6.2.2",
    "next": "^10.0.5",
    "next-pwa": "^3.1.4",
    "react": "^16.12.0",
    "react-device-detect": "^1.13.1",
    "react-dom": "^16.12.0",
    "react-flip-toolkit": "^7.0.13",
    "react-select": "^3.1.0",
    "react-slick": "^0.27.13",
    "slick-carousel": "^1.8.1",
    "styled-components": "^5.1.1",
    "styled-theming": "^2.2.0"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-proposal-decorators": "^7.10.1",
    "@storybook/addon-actions": "^6.0.28",
    "@storybook/addon-essentials": "^6.0.28",
    "@storybook/addon-links": "^6.0.28",
    "@storybook/addon-storysource": "^6.1.14",
    "@storybook/addons": "^6.1.10",
    "@storybook/react": "^6.0.28",
    "@storybook/theming": "^6.0.28",
    "@svgr/webpack": "^5.5.0",
    "@types/node": "12.12.21",
    "@types/react": "16.9.16",
    "@types/react-dom": "16.9.4",
    "@types/styled-components": "^5.1.0",
    "awesome-typescript-loader": "^5.2.1",
    "babel-loader": "^8.1.0",
    "babel-plugin-module-resolver": "^4.0.0",
    "babel-plugin-styled-components": "^1.10.7",
    "cross-env": "7.0.2",
    "eslint-config-prettier": "^7.0.0",
    "eslint-plugin-prettier": "^3.2.0",
    "eslint-plugin-react": "^7.21.5",
    "fork-ts-checker-webpack-plugin": "^5.2.1",
    "prettier": "^2.2.1",
    "react-docgen-typescript-loader": "^3.7.2",
    "react-is": "^17.0.1",
    "tsconfig-paths-webpack-plugin": "^3.3.0",
    "typescript": "3.7.3"
  }
}
```

- `mobx`는 원할 경우 v6로 업데이트 가능. 현재 글에서는 v5로 진행

#### 상황별 사용할 라이브러리

- Admin 페이지 같은 경우, material-ui 적극 활용
- material-ui/datepicker: 날짜를 달력으로 선택
  - [[Material DatePicker 커스텀(range로 사용하기) + Mobx](https://www.notion.so/Material-DatePicker-range-Mobx-0401fa643af7449c81f5ea4816884b22)]
- ~~moment:~~ 날짜<br/>
  ⇒ dayjs 사용, moment의 경우 라이브러리가 너무 크며 업데이트 중단

  - react-hook-form: form
  - react-use-gesture: 모바일 드래그 모션

- component는 스토리북을 열어 직접 확인 부탁드립니다. [[Storybook 활용법](https://www.notion.so/Storybook-202709a641774221a1fb5448dc06c979)]

_마지막 업데이트: 2021.01.25_

## ESLint, Prettier 설정

VSCode에 ESLint, Prettier Extension이 설치 되어 있어야 합니다.

- ESLint는 코드의 잠재적 에러나 오류 탐지, Prettier는 코드 포매터에 집중하여 코드를 깔끔하게 만들어주는 역할을 합니다.
- VSCode 설정 (window: `ctrl + ,` mac: `command + ,`)에서 `Format On Save`를 체크하면 저장할 때마다 바뀝니다.

  참고: [[VSCode Extension 추천 및 설정](https://www.notion.so/VSCode-Extension-4b7e3a1b02194d12a5027d6c475d102a)]

## `next-ts` 폴더 구조

![frontend_structure](/files/posts/2020_08/frontend_structure.png){: width="400px", marginRight="auto"}

- `.next`: 프로젝트 build로 생성되는 폴더
- `.storybook`: Storybook 설정 폴더
- `api`: 공통으로 사용 되는 api (Axios 커스텀)
- `components`: 공통으로 사용 되는 컴포넌트 모음
- `containers`: 해당 프로젝트에서 사용 되는 화면 구현
- `layout`: 레이아웃에서 공통으로 사용 되는 style
- `pages`: 파일 이름으로 라우터 생성
- `public`: 이미지, 아이콘 등
- `stores`: 전역상태관리 폴더
- `utils`: 프로젝트마다 공통으로 사용될 만한 hooks, format 등 모음
