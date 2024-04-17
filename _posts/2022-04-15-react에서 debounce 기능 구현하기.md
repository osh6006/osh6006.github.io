---
layout: post
title: react에서 debounce 기능 구현하기
date: 2022-04-15 13:11 +0900
lastmod: 2024-04-20 12:01:33 +0900

categories: [react, typescript]
tags: [react, typescript, debounce]
image:
  path: https://res.cloudinary.com/dxesudkxn/image/upload/v1713282581/blog/rtgrp1w2b8dc19mmcrwe.gif
---

최근 한 프로젝트에서 검색 기능을 구현하던 중 사용자가 입력을 할 때마다 서버에 불필요하게 많은 요청이 발생하는 문제를 발견했습니다. 이러한 비효율적인 처리는 서버 부하를 증가시키고, 전반적인 사용자 경험을 저하시키는 원인이 되었습니다. 이 문제를 해결하기 위해 검색 기능의 로직을 리팩토링하여 `debounce` 기술을 적용하기로 결정했습니다.

따라서 이번 글은 간단히 `debounce`에 대해 소개하고 `debounce`로 저의 프로젝트에서 만든 기능을 `codesandbox`로 공유해 보겠습니다.

{: .prompt-info }

> 이 글은 **2024-04-20** 에 업데이트 되었습니다.

{: .prompt-warning }

> 이 글은 **HTML, CSS , JavaScript, TypeScript, React**에 대한 기본적인 지식을 알고 있어야 합니다.

## Debounce 란?

![debounce](https://res.cloudinary.com/dxesudkxn/image/upload/v1713245898/blog/pgqjpcynhtybdn1qiql6.gif)

Debounce는 위의 이미지 처럼 사용자가 타이핑하거나 스크롤하는 등의 연속적인 이벤트에 대응하는 과정에서 모든 이벤트를 즉시 처리하지 않고 일정 시간 동안 기다린 후 마지막 이벤트만을 처리하는 방법입니다.

예를 들어, 사용자가 검색 창에 입력을 멈출 때까지 기다렸다가 검색을 실행하는 경우가 이에 해당합니다. 이 방법을 사용하면 불필요한 리소스 사용을 줄이고, 성능을 향상시키는 데 도움을 줍니다.

## 리팩토링 이전의 검색 기능

리팩토링 이전의 검색 기능은 단순하게 아래의 코드와 같이 사용자가 `input` 값을 변경한다면 이것을 감지하여 api 요청을 했습니다.

```tsx
import { useState } from "react";

export default function App() {
  const [value, setValue] = useState<string>("");

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.currentTarget.value);
  };

  useEffect(() => {
    // value로 서버에 value값으로 데이터 요청
  }, [value]);

  return (
    <div className="App">
      <h1>Hello Debounce</h1>
      <input
        type="text"
        value={value}
        onChange={handleChange}
        placeholder="something write"
      />
    </div>
  );
}
```

이렇게 로직을 작성하면 사용자가 변경할 때마다 요청을 보내기 때문에 `rapid api`로 하루 100회 제한된 요청을 받는 저의 프로젝트에서는 검색 한번당 수많은 요청을 보내야 했었습니다. 따라서 불필요한 `api` 요청을 줄이기 위해서 `debounce`를 적용하였습니다.

## 사용하기

사용법을 익히기 위해서 기본적인 프로젝트를 만들어서 익혀 보았습니다. 간단한 나라 검색 `api`를 이용하여 `debounce`를 적용하여 보겠습니다.

### 프로젝트 요구사항

프로젝트를 만들기 전 요구사항은 다음과 같습니다.

**필수**

- `https://restcountries.com/v3.1/` `api` 이용하여 국기 검색 사이트 만들기
- `debounce`를 사용하여 `api` 요청 최소화 하기
- `debounce`를 커스텀 훅으로 만들기

**선택**

- `debounce`를 이용하여 `back to the top` 버튼 만들기

### use Debounce hook

이제 `use debounce hook`을 만들어 보겠습니다. 보통 이 기능은 다른 곳에서도 많이 사용하기 때문에 커스텀 훅으로 만들어서 사용했고, 현 프로젝트에서는 검색 요청뿐 아니라 `back to the top` 버튼을 만들 때도 사용하였기 때문에 따로 `hook`을 만들어 주었습니다.

```tsx
import { useState, useEffect, useCallback } from "react";

// debounce 커스텀 훅
const useDebounce = (callback: Function, delay: number) => {
  const [timerId, setTimerId] = useState<NodeJS.Timeout | null>(null);

  const debounceCallback = useCallback(
    (...args: any[]) => {
      if (timerId) {
        clearTimeout(timerId);
      }
      const newTimerId = setTimeout(() => {
        callback(...args);
      }, delay);

      setTimerId(newTimerId);
    },
    [callback, delay, timerId]
  );

  useEffect(() => {
    // 컴포넌트가 언마운트되면 타이머 제거
    return () => {
      if (timerId) {
        clearTimeout(timerId);
      }
    };
  }, [timerId]);

  return debounceCallback;
};

export default useDebounce;
```

위의 코드는 입력된 콜백 함수를 지정된 시간만큼 딜레이된 후에 실행합니다. 이 과정에서 `useCallback` 훅을 사용하여 콜백 함수를 메모이제이션하고, 내부적으로 `debounce` 처리를 수행하는 함수를 반환 합니다. `useCallback`은 의존성 배열이 변경되지 않는 한 이전 콜백 함수를 재사용하므로, 타이머 ID 상태와 `delay` 값이 변경될 때만 `debounce` 처리된 새로운 콜백 함수가 생성됩니다.

### 검색 요청하기

이제 `useDebounce` 훅을 작성했으니 `api` 요청을 해야 합니다. 또한 간단하게 `loading`과 에러를 출력할 `error`도 `useState`를 이용하여 상태 관리를 해주었습니다.

```tsx
import "./App.css";
import { useState } from "react";
import useDebounce from "./use-debounce";

function App() {
  const [, setSearchValue] = useState<string>("");
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);
  const [imageInfo, setImageInfo] = useState<any>(null);

  // debounce 훅 사용
  const handleDebouncedSearch = useDebounce(async (value: string) => {
    try {
      setIsLoading(true);
      const res = await fetch(`https://restcountries.com/v3.1/name/${value}`);
      const countryData = await res.json();

      if (countryData.status === 404) {
        setIsError(countryData.message);
        return;
      }

      setImageInfo(countryData);
      setIsError(false);
    } catch (error) {
      console.error("API 호출 에러:", error);
      console.log(error);
      setIsError(true);
    } finally {
      setIsLoading(false);
    }
  }, 300); // 300밀리초(0.3초) 딜레이

  const handleChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const { value } = e.target;
    setSearchValue(value);

    if (value) {
      handleDebouncedSearch(value);
    }
  };

  console.log(imageInfo);

  return (
    <>
      <h1>Hello Debounce!</h1>
      <input
        type="text"
        placeholder="나라 이름을 입력하세요 ex) usa"
        onChange={handleChange}
      />
      {isLoading && <h2>Loading...</h2>}
      {isError && <h2>Something Error!</h2>}
      {!isLoading && !isError && imageInfo && (
        <ul>
          {imageInfo.length <= 0 && <h2>검색된 나라가 없습니다!</h2>}
          {imageInfo?.map((el: any) => (
            <img key={el.flags.alt} src={el.flags.png} alt={el.flags.alt} />
          ))}
        </ul>
      )}
    </>
  );
}

export default App;
```

위의 코드는 `setSearchValue` 로 사용자가 입력한 검색어를 저장하고, `isLoading`과 `isError`로 데이터의 상태를 나타내며, `imageInfo`는 검색된 나라의 이미지 정보를 저장합니다.

그리고 `useDebounce` 훅을 사용하여 검색어 입력을 딜레이된 형태로 처리합니다. 검색어가 변경될 때마다 해당 검색어를 사용하여 API를 호출하고, 결과를 가져와서 이미지 정보 상태를 업데이트합니다.

### back to the top 버튼 만들기

![back to the top](https://res.cloudinary.com/dxesudkxn/image/upload/v1713279395/blog/a8xr4n0wlucs9ipj5mcn.gif)

검색된 결과가 많을 경우 스크롤이 길어져 사용자가 다시 페이지의 상단으로 이동하는 것은 번거로운 일일 수 있습니다. 위의 이미지 처럼 이를 해결하기 위해 위로 스크롤하는 버튼을 추가하여 사용자 경험을 개선할 수 있습니다.

이 버튼은 페이지 하단에 고정되어 있고, 사용자가 스크롤을 아래로 내린 경우에만 나타납니다. 또한 사용자가 버튼을 클릭하면 페이지가 맨 위로 자연스럽게 스크롤되어야 합니다.

이 기능도 `debounce`를 활용해야 하는데, 그 이유는 **윈도우에서 `scroll` 이벤트를 등록하고 스크롤을 할 때 항상 호출되기 때문에** `debounce`를 사용해 주어야 합니다.

```tsx
import { useState, useEffect } from "react";
import useDebounce from "./use-debounce";

const BackToTheTop = () => {
  const [isOpen, setIsOpen] = useState(false);

  const handleScrollToTop = () => {
    // 뷰포트를 제일 위로 스크롤합니다.
    window.scrollTo({ top: 0, behavior: "smooth" });
  };

  const handleScroll = () => {
    console.log(window.scrollY);

    if (window.scrollY >= 400) {
      setIsOpen(true);
    } else {
      setIsOpen(false);
    }
  };

  const handleDebouncedScroll = useDebounce(handleScroll, 300);

  useEffect(() => {
    window.addEventListener("scroll", handleDebouncedScroll);
    return () => window.removeEventListener("scroll", handleDebouncedScroll);
  }, [handleDebouncedScroll]);

  if (!isOpen) {
    return null;
  }

  return (
    <button
      style={{
        position: "fixed",
        bottom: "10px",
        right: "10px",
        border: "1px solid 10px",
        background: "green",
        color: "white",
      }}
      onClick={handleScrollToTop}
    >
      back to the top
    </button>
  );
};

export default BackToTheTop;
```

위의 코드는 `useEffect` 훅으로 컴포넌트가 마운트 될 때 윈도우에 스크롤 이벤트를 걸고, `handleScroll` 이벤트를 `useDebounce` 훅으로 딜레이 처리시켜 주고, 윈도우 기준 상단에서 400 만큼 스크롤 했을 경우 `BackToTheTop` 버튼이 나타나게 됩니다.

## CodeSandbox에서 확인하기

지금까지 작성한 코드를 시험해 볼 수 있습니다. 콘솔도 찍어보고 여러가지 다른 더 좋은 방법을 연구해 보세요!

<iframe src="https://codesandbox.io/p/devbox/react-debounce-vite-zy9pf9?file=%2Fsrc%2Fback-to-the-top.tsx&embed=1&showConsole=true"
     style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-debounce-vite"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 결론

`debounce 는입력이나 이벤트와 같은 연속적인 액션에 대한 반복적인 작업을 효율적으로 관리하는 데 유용한 기술입니다. 이를 통해 너무 빈번한 작업을 방지하고 성능을 향상시킬 수 있었으며, 사용자 경험도 크게 개선할 수 있었습니다. 😎
