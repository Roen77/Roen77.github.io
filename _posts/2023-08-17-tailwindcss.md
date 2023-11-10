---
title: TailwindCSS와 Styled-Components
date: 2023-8-17 20:10:00 +0800
categories: [개발, TailwindCSS, React, Styled-Components]
tags: [TailwindCSS, React, Styled-Components]
pin: false
math: true
mermaid: true
---

## 개요

웹이 발전함에 따라, 그에 맞춰 웹을 스타일링 하기 위한 도구들도 많이 생겨나고 발전해왔습니다.
<br/>`Bootstrap` 같은 전통적인 CSS 프레임워크부터, CSS 전처리기인 `SASS`, CSS-in-JS 방식을 지원하는 `Emotion`이나`Styled-Components` 등 스타일링 프로세스에 도움을 주는 다양한 프레임워크나 라이브러리들이 존재합니다.
<br/> 오늘 정리해 볼 내용은 이러한 수 많은 웹의 스타일링에 도움을 주는 도구 중에서 `TailwindCSS` 라는 프레임워크에 대해 알아보고자 합니다.
`TailwindCSS` 는 HTML 마크업에 직접 적용할 수 있는 클래스 세트를 제공하는 유틸리티 우선 프레임워크입니다.
<br/>특히 React, Vue 및 Angular와 같은 프론트 프레임워크에서 자주 사용되었던 방식인 `CSS-in-JS`와 비교하여 `TailwindCSS`가 무엇이고, 어떠한 장단점이 있는지 직접 사용해보고 비교하여 느낀점을 정리했습니다.
