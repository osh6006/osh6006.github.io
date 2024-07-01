---
layout: post
title: Next.js 의 Rendering 전략
date: 2024-02-20 17:13 +0900
lastmod: 2024-06-20 13:31:23 +0900
categories: [nextjs, typescript]
tags: [next, typescript, SSR]
image:
  path: https://nextjs.org/_next/image?url=%2Fstatic%2Fblog%2Fnext-15-rc%2Fcreate-next-app-dark.png&w=2048&q=75
---

3번의 프로젝트 동안 `Next.js` 를 사용하고 있었지만 `Next.js` 의 렌더링 전략의 개념이 살짝 헷갈리는 부분이 있었습니다. 그래서 이번 글에서는 확실히 그 개념을 정리해 보겠습니다.

{: .prompt-info }

> 이 글은 **2024-06-23** 에 업데이트 되었습니다.

{: .prompt-warning }

> 이 글은 **HTML, CSS , JavaScript, TypeScript, React, Next.js**에 우터를 사용하는 13버전 이상의 버전을 설명하고 있습니다.

## Next.js의 렌더링 전략

Next.js 에서 사용하는 공식적인 렌더링 전략은 기본적으로 작성된 컴포넌트가 `Server Components` 인지 `Client Components` 인지에 따라 달라집니다.

### Server Components

Server Components는 기본적으로 Pre-rendering 방식으로 빌드되며 Server Components는 모두 이 방식을 사용하고 있습니다. 그리고 **Pre-rendering 과정**에서 **`data`를 어떻게 `cache` 하느냐에 따라서** **Static Rendering, Dynamic Rendering, Streaming** 방식으로 나뉘게 됩니다.

#### Pre-rendering

![이미지](https://res.cloudinary.com/dxesudkxn/image/upload/v1719826067/blog/Next.js%20%EB%A0%8C%EB%8D%94%EB%A7%81%20%EC%A0%84%EB%9E%B5/h8kryigwtqcpfwapu8gq.png)

`Pre-rendering` 이란 서버에서 미리 렌더링된 HTML을 제공하는 방식입니다.

우선 유저가 요청을 보내면 Next.js는 서버에서 DOM 요소를 만들어 **미리 HTML 문서를 렌더링** 합니다. (이 때 유저는 미리 랜더링된 정적 페이지를 보고 있음)

그리고 나서 모든 데이터를 받고나면 **Hydration 과정을 거쳐 클라이언트에서 렌더링**을 완료합니다.
(페이지와 유저가 상호작용 할 수 있음)

{: .prompt-info }

> **Hydration 은 뭔가요?**
> 미리 렌더링 된 뼈대만 있는 HTML에 JavaScript를 결합하여 이벤트가 동작하도록 하는 과정 입니다.

#### Static Rendering

Static Rendering은 서버에서 미리 렌더링된 HTML을 제공하는 방식은 같지만 **빌드 시에 HTML을 만들고** 클라이언트에서 **요청 시 재사용**할 수 있는 방식 입니다. StaticSiteGeneration : SSG 라고도 하지만 공식 문서에서는 Static Rendering 이라고 쓰여져 있습니다.

Next.js는 기본적으로 페이지를 생성할 때 이런 방식을 사용하고 있습니다. 아래의 코드에서 `fetch`를 사용하여 데이터를 받아오고 있는데 이때 `cache`를 `defalut` 옵션인 `force-cache`로 설정하면 빌드 시 요청이 캐시되어 있기 때문에 다음에 다시 요청을 보내지 않고 재사용할 수 있습니다.

Static Rendering 방식은 빌드 시 HTML을 만들기 때문에 가장 빠른속도로 유저와 상호작용 할 수 있으며, 주로 정적 블로그 게시물이나 제품 페이지와 같이 경로에 사용자에게 맞춤화되지 않고 **빌드 시점에 알 수 있는 데이터가 있는 경우**에 유용합니다.

```tsx
async function getData() {
  const res = await fetch("https://api.example.com/...");
  return res.json();
}

export default async function Page() {
  const data = await getData();
  return <></>;
}
```

#### Dynamic Rendering

Dynamic Rendering은 Static Rendering과는 다르게 **클라이언트가 요청 시 마다 HTML을 생성하는** Pre-rendering 방식입니다. ServerSideRendering : SSR 이라고도 하며 공식 문서에는 Dynamic Rendering 이라고 쓰여져 있습니다.

Dynamic Rendering 방식은 `fetch`를 사용할 때 `cache` 옵션을 `no-store`로 설정하면 되는데 이렇게 설정하면 빌드시 요청하지 않고 클라이언트 요청마다 새로운 HTML을 만들어 제공합니다.

또는 `Dynamic Functions` 라는 함수를 사용하여 서버에 요청하게 되면 자동적으로 Dynamic Rendering 방식으로 동작합니다. `Dynamic Functions` 는 아래와 같습니다.

- `cookies()` and `headers()`: 서버 컴포넌트에서 사용하면 요청 시 전체 경로가 동적 렌더링으로 선택됩니다.
- `searchParams`: 페이지에서 searchParams 프로퍼티를 사용하면 요청 시 페이지가 동적 렌더링으로 선택됩니다.

Dynamic Rendering 방식은 사용자가 에게 맞춤화된 데이터가 있거나 쿠키 또는 URL의 SearchParams와 같이 **요청 시점에만 알 수 있는 정보가 있는 경우**에 유용합니다.

```tsx
async function getData() {
  const res = await fetch("https://api.example.com/...", { cache: "no-store" });
  return res.json();
}

export default async function Page() {
  const data = await getData();
  return <></>;
}
```

#### Streaming

### Client Components
