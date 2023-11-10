---
title: Next.js에서 CSS-in-JS 방식의 문제점
author: Roen
date: 2023-11-03 15:13:00 +0800
categories: [개발, Next.js, Styled-Components]
tags: [Next.js, Styled-Components]
pin: false
math: true
mermaid: true
---

## 개요

최근에 기존 내용을 다듬고, 코드를 새롭게 정리하는 작업을 진행하고 있습니다.
<br/>[TailwindCSS와 Styled-Components의 사용법을 비교하는 글](https://roen77.github.io/posts/tailwindcss/)에서 작업했던 내용을 새롭게 Next.js를 사용하여 재작업하는 과정에서 겪은 문제를 정리하기 위해 글을 작성하게 되었습니다.
<Br/>

## 문제

[TailwindCSS와 Styled-Components의 사용법을 비교하는 글](https://roen77.github.io/posts/tailwindcss/)에서 하나의 웹 페이지를 가각 `TainwindCSS`와 `Styled-Components`
를 사용하여 레아아웃과 스타일링 작업을 진행했습니다.
<br/>이 때는 React를 사용하여 작업을 진행했기 때문에 별다른 문제는 없었지만, Next.js 프레임워크로 재작업을 진행하면서 `Styled-Components` 라이브러리를 사용했을 때 새로고침시마다 [Flash of Unstyled Content (FOUC) 현상](https://dev.to/lyqht/what-the-fouc-is-happening-flash-of-unstyled-content-413j)이 발생했습니다.

## 문제 파악

[Next.js](https://nextjs.org/) 공식 문서를 보면 Next.js는 React 프레임워크라고 설명되어 있습니다.<Br/>그런데 React에서는 문제가 없는데 `Next.js`에서만 문제가 발생하는 이유가 무엇일까요?
<Br/>그 이유를 알기 위해 [`Next.js` 공식 문서](https://nextjs.org/learn-pages-router/foundations/how-nextjs-works/rendering)를 파헤쳐보았습니다.
<br/>

### Next.js 작동 방식

{: .prompt-info }

> 공식 문서를 발췌하여 번역하고 정리한 글입니다.

React에서 작성한 코드를 HTML로 표현하여 UI를 보여주기 위한 작업 과정이 존재하는데, 이 프로세스를 랜더링이라고 합니다.
<br/>랜더링은 서버나 클라이언트에서 발생할 수 있고, 이는 빌드 시 미리 발생하거나 런타임 시 모든 요청에서 발생할 수 있습니다.
Next.js는 Server-Side Rendering(서버 사이드 랜더링)[^Server-Side-Rendering] , Static Site Generation(정적 사이트 생성)[^Static-Site-Generation], Client-Side Rendering(클라이언트 사이드 랜더링)[^Client-Side-Rendering] 세 가지 유형의 랜더링 방법을 사용할 수 있습니다.
<Br/>

<b style="font-size:20px;">Client-Side Rendering vs. Pre-Rendering</b>
<Br/>

<b style="font-size:18px;">Client-Side Rendering</b>
<br/>

React 애플리케이션에서 브라우저는 서버로부터 UI를 구성하기 위한 자바스크립트과 함께 빈 HTML를 받습니다.<br/>
초기 렌더링 작업이 사용자 장치에서 발생하므로 이를 Client-Side Rendering 이라고 합니다.

<b style="font-size:18px;">Pre-Rendering</b>
<br/>

결과를 클라이언트에 전송하기 전에 외부 데이터를 가져오고 반응 구성 요소를 HTML로 변환하기 때문에 이를 Pre-Rendering이라고 합니다.
<br/>즉 서버에서 미리 생성되어 클라이언트에 전송해주기 때문에 내용이 담긴 HTML를 온전히 받아 볼 수 있게 됩니다.
<br/>Pre-Rendering에는 SSR 과 SSG 두가지 유형이 존재합니다.

<b>SSR</b>란,

각 요청에 대해 페이지의 HTML이 서버에서 생성됩니다.
<Br/>이렇게 생성된 HTML은 해당 페이지에 필요한 자바스크립트와 연결하는 작업을 거치게 되는데(이벤트 핸들러를 버튼에 부착하는 것 등의 작업) 이를 hydration이라고 합니다.
<br/>이러한 과정을 거쳐 브라우저에 랜더링 되면 사용자가 페이지에 동적으로 상호작용 할 수 있습니다.
<br/>

<b>SSG</b>란,

HTML이 서버에서 생성되지만 SSR과 달리 런타임에는 서버가 없습니다.
<br/>HTML을 빌드 시에 한번 생성되고, 요청이 올때마다 이미 생성된 HTML를 재사용하여 반환해줍니다.

<Br/>
기본적으로 `Next.js`는 모든 페이지를 Pre-Rendering합니다.
즉 서버에서 이미 필요한 데이터를 가져오고, HTML를 생성하여 그것을 클라이언트에 보내주게 됩니다.
<Br/>클라이언트에서는 이 HTML를 받고 그 후에 자바크스립트를 로딩하고 실행시키는데, `Styled-Components` 같은 CSS-in-JS 방식은 자바스크립트가 실행되는 런타임에 스타일이 적용되기 때문에 자바스크립트가 실행되기 전에는 스타일이 적용되지 않는 문제가 발생하게 됩니다.
 <Br/>공식 문서를 살펴보면 아래와 같이 설명해주고 있습니다.

{: .prompt-warning }

> 런타임 자바스크립트가 필요한 CSS-in-JS 라이브러리는 현재 Server Components에서 지원되지 않습니다.
> <br/>서버 컴포넌트를 스타일링하려면 CSS 모듈이나 PostCSS 또는 Tailwind CSS와 같은 CSS 파일을 출력하는 다른 솔루션을 사용하는 것이 좋습니다.

그렇다면 해결 방법이 없는 것일까요?
<br/>
`Next.js` 와 `Styled-Components` 공식 문서를 통해 해결의 실마리를 잡을 수 있습니다.

## 해결

[Next.js](https://nextjs.org/docs/app/building-your-application/styling/CSS-in-JS) 와 [Styled-Components](https://styled-components.com/docs/advanced#server-side-rendering) 를 보면 알겠지만,
`Next.js` 의 버전이 무엇인지, app 라우터 방식을 사용했는지 page 라우터 방식을 사용했는지에 따라 해결 방법이 조금씩 다릅니다.
<Br/>

먼저 [Styled-Components](https://styled-components.com/docs/advanced#server-side-rendering)에서는 stylesheet rehydration을 통해 SSR을 지원합니다.
<br/>
React에서는 `ServerStyleSheet`를 작성하고 컨텍스트 API를 통해 스타일을 허용하도록 설정해줄 수 있습니다.
<Br/>그렇다면 `Next.js`에서는 어떻게 해야 할지 정리해보도록 하겠습니다.

<table>
<thead>
<tr>
<th>분류</th>
<th>참고 예제</th>
<th>비고</th>
</tr>
</thead>
<tbody>
<tr><th>1. babel과 함께 사용할 경우</th>
<td><a href="https://github.com/vercel/next.js/tree/canary/examples/with-styled-components-babel">해당 예제를 참고</a>
</td></tr>
<tr><th>2. Next.js 버전이 12이상인 경우</th>
<td><a href="https://github.com/vercel/next.js/tree/canary/examples/with-styled-components">해당 예제를 참고</a></td>
<td>Next.js 버전이 12이상인 경우<br/>SWC라는 Rust 컴파일러를 사용합니다.<br/>
Babel 플러그인을 사용하지 않는 경우에<br/>next.config.js에 compiler 옵션을<br/>따로 추가해주는 작업이 필요합니다.</td>
</tr>
<tr><th>3. Next.js 버전이 13이상이고,<br/>앱 라우터를 사용할 경우</th>
<td><a href="https://nextjs.org/docs/app/building-your-application/styling/CSS-in-JS#styled-components">해당 예제를 참고</a></td>
<td>사용하는 Styled-Components 라이브러리 버전이<br/>6이상이어야 합니다.</td>
</tr>
</tbody>
</table>

<br/>
여기서 저는 <b>3번</b>에 해당되기 때문에 해당 방법으로 수정해보도록 하겠습니다.
<br/>

먼저 `style-components API`를 사용하여 렌더링 중에 생성된 모든 CSS 스타일 규칙을 가져오도록 글로벌 레지스트리 구성 요소와 해당 규칙을 반환하는 함수를 만듭니다.<br/>그런 다음 serverInsertedHTML hook을 사용하여 레지스트리에 수집된 스타일을 루트 레이아웃의 <head> HTML 태그에 주입합니다.

```tsx
// lib/registry.tsx
"use client";

import React, { useState } from "react";
import { useServerInsertedHTML } from "next/navigation";
import { ServerStyleSheet, StyleSheetManager } from "styled-components";

export default function StyledComponentsRegistry({
  children
}: {
  children: React.ReactNode;
}) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet());

  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement();
    styledComponentsStyleSheet.instance.clearTag();
    return <>{styles}</>;
  });

  if (typeof window !== "undefined") return <>{children}</>;

  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  );
}
```

```tsx
// app/layout.tsx
import StyledComponentsRegistry from "./lib/registry";

export default function RootLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <StyledComponentsRegistry>{children}</StyledComponentsRegistry>
      </body>
    </html>
  );
}
```

서버를 렌더링하는 동안 스타일은 글로벌 레지스트리로 추출되고 HTML의 <head>에 플러시됩니다.
<br/>이렇게 하면 스타일 규칙을 사용할 수 있는 컨텐츠보다 먼저 스타일 규칙이 배치됩니다.
<br/>스타일 규칙을 더 효율적으로 추출해주기 위해 최상위 요소에 `Client Component`를 사용해야 하므로 `"use client"`라고 선언해주어야 합니다.

<br/>실제로 `Styled-Components`를 사용하고 있는 page에서 `"use client"`를 선언해주지않으면 오류가 납니다.

![Desktop View](https://github.com/Roen77/Roen77.github.io/assets/65719512/0637dce7-c306-4d77-9a6f-938c6df066c9){: width="972" height="589" }

<!-- https://github.com/Roen77/Roen77.github.io/assets/65719512/0637dce7-c306-4d77-9a6f-938c6df066c9 -->

<!-- <br/>이러한 방법으로 스타일 규칙을 추출하는 것이 효과저 최상위 요소에 `Client Component`를 사용해주기 위해 `"use client"`라고 선언해주어야 합니다. -->

<br/><br/>
위 방법을 통해 수정 전과 수정 후 화면을 비교한 모습입니다.
<br/>
<br/>
<b style="fontsize:18px">수정 전</b>
![Desktop View](https://github.com/Roen77/Roen77.github.io/assets/65719512/3588031d-ceb7-4b30-97c8-a6c42ce5e534){: width="972" height="589" }

- 보다시피 새로고침 시 스타일이 적용되지 않은 날것의 콘텐츠들이 보이는 것을 확인할 수 있습니다.

<br/>
<b style="fontsize:18px">수정 후</b>
![Desktop View](https://github.com/Roen77/Roen77.github.io/assets/65719512/ad4142b3-3114-4765-8856-68073736cfa7){: width="972" height="589" }

- 새로고침에도 기존에 적용시킨 스타일이 유지되는 것을 확인할 수 있습니다.

## 정리

`Next.js`의 작동 원리를 파악한다면 `CSS-in-JS`가 어떠한 문제점을 가지고 있는지 이해할 수 있습니다.
<br/>위의 방법대로 문제를 파악하고 개선하긴 했지만, `Next.js` 프레임워크를 사용한다면 별도의 세팅이나 구성해주어야 하는 작업이 추가적으로 들어가기 때문에 설정이 번거롭지 않고 스타일링에 유틸리티 기능을 제공해주는 `TailwindCSS`를 사용하는 것이 나아보입니다.
<br/>처음 `Next.js`로 프레임워크를 설치하고 세팅할 때, 기본적으로 `TailwindCSS` 사용 옵션을 선택할 수 있기 때문에 설치 과정도 간편하다는 이점도 있고 스타일링에도 많은 도움을 주는 기능이 있어 이러한 프레임워크를 적극적으로 사용하는 것도 도움이 될 것 같습니다.

{: .prompt-info }

> <span style="color:red">(2023.11 추가 완료)</span> [참고 코드 링크](https://github.com/Roen77/Styling-Next)

{: .prompt-info }

> <span style="color:red">(2023.11 추가 완료)</span> [사이트 링크](https://roen77.github.io/Styling-Next/)

{: .prompt-info }

> [연관된 글 참고](https://roen77.github.io/posts/tailwindcss/)

<br/>
<span style="background-color:#E3F2FD; padding:4px; border-radius:2px;">주석</span>

[^Server-Side-Rendering]: SSR로 축약
[^Static-Site-Generation]: SSG로 축약
[^Client-Side-Rendering]: CSR로 축약
