---
layout: post
title: Intersection Observer API로 화면 요소 감지하기
date: 2023-01-18 20:13 +0900

lastmod: 2024-01-02 13:38:56 +0900
categories: [JS]
tags: [javascript, intersection-observer]
image:
  path: /assets/gifs/intersection-observer.gif
  alt: thumbnail
---

웹에서는 다양한 화면 크기와 장치에 대응하여 요소들의 가시성을 관리해야 합니다. 특히, `Intersection Observer API`는 페이지 로딩 시에 모든 이미지나 미디어를 한 번에 로딩하지 않고 화면에 특정 이미지나 미디어가 나타날 때 로딩하는 방식을 채택 하여 성능을 개선하거나 사용자가 특정 광고 영역으로 스크롤할 때 해당 광고의 노출 여부를 감지하고 트래킹할 수 있는 기능, 이 밖에도 무한로딩, 스크롤 기반 애니메이션 등 다양한 상황에서 쓰일 수 있습니다.

따라서 프론트엔드 개발자라면 `Intersection Observer API`에 대한 기본적인 사용법을 익혀두고 나중에 실무나 사이드 프로젝트에 응용해서 쓸 수 있어야 한다고 생각 합니다.

또한 이 글에서는 `Intersection Observer API`가 탄생하게된 배경 또는 정의를 쓰는 글이라기보다 작은문제가 주어졌을 때 제가 `API`를 어떻게 활용해아할 지를 고민한 글 입니다.

{: .prompt-info }

> 이 글은 **2024-01-02** 에 업데이트 되었습니다.

{: .prompt-warning }

> 이 글은 HTML, CSS , JavaScript에 대한 기본적인 지식을 알고 있어야 합니다.

## 프로젝트 요구 사항 정의

![Alt text](/assets/imgs/intersection-observer.png)

저는 api나 다른 새로운 기술을 익힐 때 간단하게라도 프로젝트를 생각해서 만들어보고 적용해 보는 것을 선호합니다.
따라서 이 글에서도 아주아주 간단하게 위와 같은 **`Intersection Observer API`로 요소를 감지하면 `pink`였던 배경색이 아름다운 `skyblue` 색이 되는 프로젝트를 만들어 보겠습니다.**

{: .prompt-tip }

> [결과 확인](#결과)의 CodePen에서 모든 코드와 실행화면을 확인하실 수 있습니다!

## 기본적인 사용법

간단한 프로젝트에 `Intersection Observer API`를 사용하기 이전에 저는 아래와 같은 3단계를 지정하여 사용법을 이해했습니다.

**1.콜백함수와 옵션 작성하기**

```js
let options = {
  // 대상 객체의 가시성을 확인할 때 사용되는 뷰포트 요소입니다. 조상 요소여야함
  // 등록하지 않으면 자동으로 뷰포트 기준으로 됩니다.
  root: document.querySelector("조상요소"),
  // root 가 가진 여백을 지정할 수 있습니다. CSS의 margin 속성과 유사하게
  // 상,우 하,좌 로 설정이 가능합니다.
  rootMargin: "0px",

  // observer의 콜백이 실행될 대상 요소의 가시성 퍼센티지를 나타내는 단일 숫자 혹은 숫자 배열입니다.
  // 만일 50%만큼 요소가 보여졌을 때를 탐지하고 싶다면, 값을 0.5로 설정하면 됩니다.
  threshold: 1.0,
};

let callback = (entries, observer) => {
  entries.forEach((entry) => {
    // 각 항목들 중에서 관찰된 하나의 교차 변경을 설명합니다.
    //   entry.boundingClientRect
    //   entry.intersectionRatio
    //   entry.intersectionRect
    //   entry.isIntersecting
    //   entry.rootBounds
    //   entry.target
    //   entry.time
  });
};
```

**2. `IntersectionObserver`를 등록하기 (콜백함수와 옵션이 매개변수로!)**

```js
const io = new IntersectionObserver(callback, options);
```

**3. 관찰할 대상을 선언하고 관찰시키기**

```js
const observer = document.querySelectorAll(".box");
divEl.forEach((el) => {
  io.observe(el);
});
```

### 전체 코드

전체 코드 다음과 같습니다. callback함수의 entry옵션의 설명이 궁금하시면 [여기](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry)를 클릭하세요!

```js
// 1. 콜백함수와 옵션 작성하기

let options = {
  // 대상 객체의 가시성을 확인할 때 사용되는 뷰포트 요소입니다. 조상 요소여야함
  // 등록하지 않으면 자동으로 뷰포트 기준으로 됩니다.
  root: document.querySelector("#조상요소"),
  // root 가 가진 여백을 지정할 수 있습니다. CSS의 margin 속성과 유사하게
  // 상,우 하,좌 로 설정이 가능합니다.
  rootMargin: "0px",

  // observer의 콜백이 실행될 대상 요소의 가시성 퍼센티지를 나타내는 단일 숫자 혹은 숫자 배열입니다.
  // 만일 50%만큼 요소가 보여졌을 때를 탐지하고 싶다면, 값을 0.5로 설정하면 됩니다.
  threshold: 1.0,
};

let callback = (entries, observer) => {
  entries.forEach((entry) => {
    // 각 항목들 중에서 관찰된 하나의 교차 변경을 설명합니다.
    //   entry.boundingClientRect
    //   entry.intersectionRatio
    //   entry.intersectionRect
    //   entry.isIntersecting
    //   entry.rootBounds
    //   entry.target
    //   entry.time
  });
};

// 2. `IntersectionObserver`를 등록하기
const io = new IntersectionObserver(callback, options);

// 3. 관찰할 대상을 선언하고 관찰시키기
const observer = document.querySelectorAll(".box");
divEl.forEach((el) => {
  io.observe(el);
});
```

## 프로젝트에 적용

위에서 사용법을 알았으니 이제 우리 프로젝트에 적용해 봅시다.

우리가 사용할 요소인 `box`라는 클래스를 가진 `div`요소를 여러개 생성하고 `CSS`로 크기와 색을 스타일링 해 줍니다.

```html
<body>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
</body>
```

```css
.box {
  width: 200px;
  height: 200px;
  background-color: pink;
  margin: 5rem auto;
  border-radius: 10px;
  transition: all 0.5s ease;
}

.active {
  background-color: skyblue;
}
```

위에 설명한 사용법과 같이 **단계 1**을 작성했습니다. 단계 1에서 `root`와 `rootMargin`은 생략해도 되는 값이기에 생략해 주었고 뷰포트에서 보여지는 기준인 `threshold`를 `0.5`로 지정하였습니다.

콜백 함수는 뷰포트의 요소들의 배열인 `entries` 들을 기준으로 가시비율인 `intersectionRatio`가 0.5 이상이라면 `active` 클래스를 아니라면 제거해 주었습니다.

```js
// 1. 콜백함수와 옵션 작성하기
let options = {
  threshold: 0.5,
};

let changeColor = (entries, observer) => {
  entries.forEach((entry) => {
    if (entry.intersectionRatio > 0.5) {
      entry.target.classList.add("active");
    } else {
      entry.target.classList.remove("active");
    }
  });
};
```

이제 **단계 2**와 **단계 3**은 위에서 설명한 것처럼 옵저버를 선언하고 관찰할 대상을 관찰 시키면 됩니다.

```js
// 2. 옵저버 등록하기
const io = new IntersectionObserver(changeColor, { threshold: 0.5 });

// 3. 관찰할 대상을 선언하고 관찰시키기
const boxEl = document.querySelectorAll(".box");
boxEl.forEach((el) => {
  io.observe(el);
});
```

### 결과

스크롤을 천천히 내려보세요!

<p class="codepen" data-height="500" data-default-tab="html,result" data-slug-hash="rNQbxzx" data-user="osh6006" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/osh6006/pen/rNQbxzx">
  인터섹션 옵저버 사용하기</a> by osh6006 (<a href="https://codepen.io/osh6006">@osh6006</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

## 결론

`Intersection Observer API`는 DOM조작이나 스크롤 이벤트를 구현할 때 자주 사용되는 방법 중 하나 입니다. 이번 포스트 에서는 `javascript`에서 사용하는 방법을 알아 봤으니 다음에는 `react`에서 커스텀 훅을 사용해서 랜더링 최적화 하는 간단한 프로젝트도 추후에 포스팅 해봐야 겠다는 생각이 들었습니다.

## 참고 사이트

https://developer.mozilla.org/ko/docs/Web/API/Intersection_Observer_API
https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry
