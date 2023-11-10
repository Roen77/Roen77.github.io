---
title: TailwindCSS와 Styled-Components
author: Roen
date: 2023-08-19 20:13:00 +0800
categories: [개발, TailwindCSS, React, Styled-Components]
tags: [TailwindCSS, React, Styled-Components]
pin: false
math: true
mermaid: true
---

## 개요

웹이 발전함에 따라, 그에 맞춰 웹을 스타일링 하기 위한 도구들도 많이 생겨나고 발전해왔습니다.
<Br/>`Bootstrap` 같은 전통적인 CSS 프레임워크부터, CSS 전처리기인 `SASS`, CSS-in-JS 방식을 지원하는 `Emotion`이나`Styled-Components` 등 스타일링 프로세스에 도움을 주는 다양한 프레임워크나 라이브러리들이 존재합니다.
<Br/> 오늘 정리해 볼 내용은 이러한 수 많은 웹의 스타일링에 도움을 주는 도구 중에서 `TailwindCSS` 라는 프레임워크에 대해 알아보고자 합니다.
`TailwindCSS` 는 HTML 마크업에 직접 적용할 수 있는 클래스 세트를 제공하는 유틸리티 우선 프레임워크입니다.
<br/>특히 React, Vue 및 Angular와 같은 프론트 프레임워크에서 자주 사용되었던 방식인 `CSS-in-JS`와 비교하여 `TailwindCSS`가 무엇이고, 어떠한 장단점이 있는지 직접 사용해보고 비교하여 느낀점을 정리했습니다.

## 비교

우선 유틸리티 우선 프레임워크인 `TailwindCSS` 와 `CSS-in-JS` 방식을 제공해주는 라이브러리인 `Styled-Components` 으로 각각 웹의 스타일링을 진행하여 비교해보려 합니다.
<Br/> 굳이 이 둘을 비교하는 이유는 둘 다 웹의 스타일링에 도움을 주기 위한 궁극적으로 같은 목적을 가지고 있고, 실제 업무에서 `CSS-in-JS` 방식을 많이 사용해왔기에, 이렇게 익숙한 방식과 비교해본다면 더욱 장단점을 느낄 수 있다고 판단했습니다.

<br/>그럼 [해당 사이트](https://business-starter-template.webflow.io/)를 참고하여 레이아웃과 반응형을 가진 웹 페이지를 만든다는 가정하에 각각 비교해봅시다.

{: .prompt-info }

> free Template를 제공해주고 있고, 자주 사용되는 가로 정렬 레이아웃에서 세로 정렬 레이아웃으로 변경되는 반응형이나, 헤더 메뉴 모바일 처리 등 스타일링을 비교하기 적합하다고 판단하여 해당 Template를 참고하게 되었습니다.

### 1. 스타일 정의

- <b style="font-size:18px">Styled-Components</b>

`Styled-Components`는 `CSS-in-JS` 방식으로 스타일링을 주는 방식을 제공합니다.
<br/>`CSS-in-JS` 방식은 명칭에서 추측할 수 있듯이 자바스크립트 코드에서 CSS를 작성하는 방식을 말합니다.
<br/>템플릿 리터럴을 사용해 CSS를 정의한 자바스크립트 태그를 선언하여 스타일을 정의할 수 있습니다.
<br/>

```tsx
// theme.ts

// 전역적으로 사용할 스타일 테마 설정
const theme = {
  fontSizes: {
    sm: "12px",
    base: "16px"
  },
  letterSpacing: {
    wide: "0.0625rem",
    widest: "0.125em"
  },
  colors: {
    white: "#fff",
    black: "#000",
    primaryBlack: "#333",
    darkGray: "#222",
    buttonBlack: "#1a1b1f"
  }
};

// 컴포넌트에서 사용
import styled, { ThemeProvider } from "styled-components";
function CSSInJs() {
  return (
    // ThemeProvider 로 감싸주어 theme 사용 할 수 있도록 정의
    <ThemeProvider theme={theme}>
      <Header>
        <a className="logo" href="#">
          Business
        </a>
      </Header>
    </ThemeProvider>
  );
}

export default CSSInJs;

// props로 미리 정의된 theme 사용
const Header = styled.header`
  padding: 30px 50px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: ${({ theme }) => theme.fontSizes.sm};
  letter-spacing: ${({ theme }) => theme.letterSpacing.wide};
  position: relative;
`;
```

위의 예제처럼 미리 공통 테마를 지정해서 스타일을 커스텀하거나 확장시킬 수 있고, props 사용하여 동적으로 스타일을 지정해줄 수 있습니다.

<br/>

- <b style="font-size:18px">TailwindCSS</b>

`TailwindCSS`는 해당 프레임워크에서 제공해주는 클래스 명을 사용하여 스타일을 지정해줄 수 있습니다.

```tsx
// tailwindcss.config.js
import type { Config } from "tailwindcss";

// 전역적으로 사용할 스타일 테마 커스텀
const config: Config = {
    ...
  theme: {
    extend: {
      padding: {
        "30": "1.875rem",
      },
      colors: {
        "primary-black": "#333",
        "dark-gray": "#222",
        "button-black": "#1a1b1f",
      },
      letterSpacing: {
        wide: "0.0625rem",
        widest: "0.125em",
      },
    },
  },
};

export default config;

// 컴포넌트에서 사용
function TailwindCSS() {
  return (
    <header>
      <a
        className="font-semibold text-2xl border-b-4 border-b-black tracking-wide"
        href="#"
      >
        Business
      </a>
    </header>
  );
}
```

미리 정의된 규칙에 따라 클래스를 결합하여 스타일을 지정해줄 수 있고, 구성 요소를 커스텀하여 새로운 테마를 생성해 결합하여 사용할 수 있습니다.
<br/>

### 2. 반응형

- <b style="font-size:18px;">Styled-Components</b>

일반적인 [미디어 쿼리](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_media_queries/Using_media_queries)를 사용해 중단점(Breakpoint)을 지정하여 반응형 처리를 할 수 있습니다.

```tsx
const Header = styled.header`
  padding: 30px 50px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: ${({ theme }) => theme.fontSizes.sm};
  letter-spacing: ${({ theme }) => theme.letterSpacing.wide};
  position: relative;

  .contact-button {
    padding: 12px 25px;
    background: ${({ theme }) => theme.colors.buttonBlack};
    color: ${({ theme }) => theme.colors.white};
    letter-spacing: ${({ theme }) => theme.letterSpacing.widest};
  }
  // 화면이 990px보다 작아질경우 지정할 스타일
  @media screen and (max-width: 990px) {
    ul.menu,
    .contact-button {
      display: none;
    }
    .mobile-button {
      display: block;
    }
  }
`;
```

<br/>

- <b style="font-size:18px">TailwindCSS</b>

[`TailwindCSS`에서 제공해주는 기능](https://tailwindcss.com/docs/responsive-design#using-custom-breakpoints)으로 미디어 쿼리를 사용할 수 있습니다.

| Breakpoint prefix | Minimum width |                         CSS |
| :---------------- | :------------ | --------------------------: |
| sm                | 640px         |  (min-width: 640px) { ... } |
| md                | 768px         |  (min-width: 768px) { ... } |
| lg                | 1024px        | (min-width: 1024px) { ... } |
| xl                | 1280px        | (min-width: 1280px) { ... } |
| 2xl               | 1536px        | (min-width: 1536px) { ... } |

<br/>우리가 사용하는 중단점(Breakpoint)은 990px인데 `TailwindCSS`에서 제공하지 않더라도 커스텀하여 반응형 처리가 가능합니다.

```tsx
// tailwindcss.config.js
import type { Config } from "tailwindcss";

const config: Config = {
    ...
  theme: {
    // screen  중단점(Breakpoint) 지정
    screens: {
      tablet: "640px",
      // => @media (min-width: 640px) { ... }

      laptop: "990px",
      // => @media (min-width: 990px) { ... }

      desktop: "1280px",
      // => @media (min-width: 1280px) { ... }
    },
  },
};

export default config;


// 컴포넌트에서 사용
function TailwindCSS() {
  return (
// 화면이 990px 이하일 경우, flex-direction: column으로 수직 정렬, 그 이상일 경우 flex-direction: row로 수평 정렬
  <div className="flex laptop:flex-row flex-col py-30 gap-2">
    <div className="basis-1 laptop:basis-1/3">
      <h3 className="font-bold text-4xl py-5">Three Col 1</h3>
      <div>
        desktop publishing packages and web page editors now use Lorem
        Ipsum as their default model text, and a search for lorem ipsum
        will uncover many web sites still in their infancy.
      </div>
    </div>
    <div className="basis-1 laptop:basis-1/3">
      <h3 className="font-bold text-4xl py-5">Three Col 2</h3>
      <div>
        Various versions have evolved over the years, sometimes by
        accident, sometimes on purpose (injected humour and the like).
      </div>
    </div>
    <div className="basis-1 laptop:basis-1/3">
      <h3 className="font-bold text-4xl py-5">Three Col 3</h3>
      <div>
        It is a long established fact that a reader will be distracted
        by the readable content of a page when looking at its layout.
        The point of using Lorem Ipsum is that it has a more-or-less
        normal distribution of letters, as opposed to using Content here
        content here
      </div>
    </div>
  </div>
  );
}
```

<br/>

### 3. hover나 focus 같은 상태 정의

- <b style="font-size:18px">Styled-Components</b>
  <br/>

CSS에서 일반적으로 사용하는 방식으로 `hover` 되었을 때, 스타일링이 가능합니다.

```tsx
const Header = styled.header`
  ul.menu {
    display: flex;
    color: ${({ theme }) => theme.colors.darkGrary};

    li:hover > a {
      opacity: 1;
    }
    li > a {
      padding: 20px;
      opacity: 0.6;
    }
  }
`;
```

<br/>
- <b style="font-size:18px">TailwindCSS</b>
  <br/>

`TailwindCSS`에서 제공해준 규칙대로 지정하여 `hover` 되었을 때, 스타일링이 가능합니다.

```tsx
function TailwindCSS() {
  return (
    <ul className="flex  max-laptop:hidden">
      {menuList.map((menu, i) => (
        <li key={i}>
          <a
            //hover될때 글자 색상의 opacity를 1로 변경
            className="p-[20px] text-dark-gray/60 hover:text-dark-gray/100"
            href="#"
          >
            {menu}
          </a>
        </li>
      ))}
    </ul>
  );
}
```

## 정리

같은 사이트를 `TainwindCSS` 사용한 유틸리티를 우선하여 스타일링을 주는 방식과 `Styled-Components`를 사용하여 JS내에 스타일을 주입하는 방식으로 만들어보았습니다.
<Br/>각각의 방식을 사용하며 느낀 점을 정리하자면 아래와 같습니다.

<table>
  <thead>
    <tr>
      <th>분류</th>
      <th>TailwindCSS</th>
      <th>Styled-Components</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>장점</td>
      <td><ul>
          <li>
            HTML 마크업에 직접 인라인 형식으로 작성하여
            <br />
            빠른 UI 개발이 가능함
          </li>
          <li>
           CSS 리팩토링의 필요성을 줄여<br/>코드의 길이를 줄일 수 있음
          </li>
          <li>
         유틸리티 클래스를 결합하여<br/>스타일링을 확장하기 쉬움
          </li>
          <li>
        복잡한 미디어 쿼리를 간소화하여<br/>반응형 디자인 작업이 수월함
          </li>
        </ul></td>
      <td><ul>
          <li>
            className을 별도로 주지 않고<Br/>스타일링이 가능함
          </li>
          <li>
            props로 조건에 따라<br/>동적으로 스타일링 변경이 쉬움
          </li>
          <li>
            컴포넌트별로 스타일을<br/>분리하여 관리하기 쉬움
          </li>
        </ul></td>
    </tr>
    <tr>
       <td>단점</td>
      <td>
        <ul>
          <li>
           스타일이 많아질 경우, HTML 마크업에 코드가 많아져<Br/>가독성이 떨어질 수 있음
          </li>
          <li>
          처음 사용시, 제공해주는 클래스의 규칙을 파악하는데<Br/>시간이 걸릴 수 있음
          </li>
        </ul>
      </td>
      <td>
        <ul>
          <li>
           CSS를 정의한 자바스크립트 태그를 만들때마다<Br/>명칭을 따로 지정해주어 생성해주어야 하는 번거로움이 있음
          </li>
          <li>
          스타일을 적용하려면 사용자가 직접<Br/>CSS를 작성해주어야 함
          <br/>(반응형 디자인 시 미디어 쿼리를 직접 작성해주어 스타일링 필요)
          </li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

<br/>

성능에 관해서는 각각 공식 문서를 참고하여 정리했습니다.
<br/>
<br/>
<b>성능</b>

<table>
  <thead>
    <tr>
      <th>TailwindCSS</th>
      <th>Styled-Components</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><ul>
          <li>
            프로덕션용으로 빌드할 때 사용되지 않는 모든 CSS를<Br/>자동으로 제거하여 CSS 번들이 적다.<Br/>(실제로 대부분의 Tailwind 프로젝트는 <br/>클라이언트에 10kB 미만의 CSS를 제공한다.)
          </li>
        </ul></td>
      <td><ul>
          <li>
          페이지에 렌더링되는 구성 요소를 추적하고<br/>해당 스타일만 주입하여, 최소한의 코드로만 로드가 가능하지만,<br/>
            기본적으로 CSS-in-JS 방식으로<Br/>JS 런타임에 영향을 주기 때문에 성능 이슈가 발생할 수 있다.
          </li>
        </ul></td>
    </tr>
  </tbody>
</table>

<br/>
실제 업무에서 `Styled-Components` 나 `Emotion` 라이브러리를 사용하여 `CSS-in-JS`방식으로 작업을 해 왔는데요.
<br/>`TailwindCSS` 를 사용해보니, 반응형 디자인 작업이 월등히 수월했고, 실제 디자인 작업에도 개발 속도가 대략 1/4 더 단축되었습니다.
<Br/>물론 처음에는 익숙치 않는 클래스 규칙을 파악하느라 시간은 좀 걸렸지만, 공식 문서에도 자세히 설명되어 있기에 큰 문제는 되지 않았습니다.
<br/>프로젝트에 맞게 원하는 방식을 선택하여 개발을 하면 되겠지만, 미디어 쿼리 작업이 간소화되고 개발이 빠르다는 점에서 개인 프로젝트 진행시에는
`TailwindCSS` 사용할 것 같습니다.

<br/>

{: .prompt-danger }

> <span style="color:#E91E63;">(23년 11월 추가)</span> 코드를 정리하는 과정에서 SSR 환경에서 `Styled-Components`
> 사용시, 문제점을 발견하여 해당 [정리글](https://roen77.github.io/posts/nextjs-CSSInJs/)은 별도로 분리했습니다.
