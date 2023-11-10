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
