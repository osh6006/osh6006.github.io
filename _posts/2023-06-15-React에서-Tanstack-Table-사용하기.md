---
layout: post
title: React에서 Tanstack Table 사용하기
date: 2023-06-15 11:13 +0900
lastmod: 2024-06-14 18:31:23 +0900

categories: [react, typescript]
tags: [react, typescript, tanstack-table]
image:
  path: https://tanstack.com/_build/assets/logo-color-600w-Bx4vtR8J.png
---

개인 프로젝트에서 축구팀과 선수들의 순위를 구현하기 위해 테이블을 사용해야 했습니다. 그러나 여러 가지 기능이 있는 테이블을 직접 구현하는 데는 시간이 많이 걸릴 수 있습니다. 그래서 빠르고 효율적으로 다양한 기능을 제공하는 라이브러리를 찾던 중, tanstack-table을 발견하게 되었습니다. 오늘은 tanstack-table을 간단히 소개하고, 사용법과 프로젝트에 어떻게 적용했는지에 대해 알아보겠습니다.

{: .prompt-info }

> 이 글은 **2024-06-14** 에 업데이트 되었습니다.

{: .prompt-warning }

> 이 글은 **HTML, CSS , JavaScript, TypeScript, React**에 대한 기본적인 지식을 알고 있어야 합니다.

## Tanstack Table 이란

`tanstack-table`은 `TS/JS`를 완벽하게 지원하고 `React`뿐 아니라 `Vue`, `Sevelt`등 다양한 프레임 워크에서도 사용이 가능하며, `Headless UI`를 지향하는 라이브러리 입니다.

{: .prompt-info }

> Headless UI란 뭔가요?? \
> \
> Headless UI란 UI 요소와 상호작용을 위한 로직, 상태, 처리 및 API를 제공하지만 마크업, 스타일 또는 사전 구축된 구현을 제공하지 않는 라이브러리 및 유틸리티를 일컫는 용어입니다. \
> \
> 즉, Headless UI 컴포넌트는 **UI의 기능적 부분만 제공하며, 디자인과 스타일링은 개발자가 원하는 대로 구현할 수 있습니다.** 이를 통해 재사용 가능한 컴포넌트를 만들 수 있습니다. 또한 **동일한 기능 로직을 여러 프로젝트나 컴포넌트에서 재사용할 수 있어, 유지보수와 확장이 용이**합니다.

기본적인 테이블 구성뿐만 아니라, 필터링, 정렬, 페이징 등의 고급 기능을 간단하게 구현할 수 있도록 도와주며, 공식문서도 아주 친절한 설명이 되어있기 때문에 최신 버전으로 마이그레이션 하거나, 다양한 예제를 찾아 프로젝트에 적용할 수 있습니다.

## 사용법

우선 `react`와 `typescript`로 프로젝트 세팅을 했다면 터미널에 아래의 코드를 입력하여 `npm` 패키지를 다운받습니다.

### 설치하기

```bash
npm install @tanstack/react-table
```

### 데이터와 타입 정의하기

이제 테이블에 사용할 데이터를 정의해 봅시다. 데이터는 제가 이전에 포스팅 했던

[ReactQueryKey Factory로 관리하기](https://osh6006.github.io/posts/React-Query-key-Factory/)의 예제로 사용했던 랜덤 유저 API의 정적 데이터를 가져와서 세팅해 주었습니다. 아래에는 데이터중 일부를 가져온 것 입니다. 확인하려면 이전의 포스팅에서 `CodeSandBox`의 예제를 통해 확인하실 수 있습니다.

이 데이터들 중에서 저는 **이름 , 사진, 위치, 이메일** 정보만 사용하였습니다.

```json
// 많은 유저 데이터 중 일부...

[
  {
    "gender": "female",
    "name": {
      "title": "Ms",
      "first": "Selma",
      "last": "Poulsen"
    },
    "location": {
      "street": {
        "number": 7629,
        "name": "Kærmindevej"
      },
      "city": "Sønder Stenderup",
      "state": "Hovedstaden",
      "country": "Denmark",
      "postcode": 38789,
      "coordinates": {
        "latitude": "-40.3337",
        "longitude": "66.8465"
      },
      "timezone": {
        "offset": "+3:00",
        "description": "Baghdad, Riyadh, Moscow, St. Petersburg"
      }
    },
    "email": "selma.poulsen@example.com",
    "login": {
      "uuid": "dde8c7d3-5d54-4e05-b778-1771daf92342",
      "username": "orangebutterfly511",
      "password": "hollywood",
      "salt": "2tq5cvnE",
      "md5": "816959871c58ff6d5dba6f3546814e99",
      "sha1": "5af54145902c5cdd03026d36d8c15e03248ad9b0",
      "sha256": "1f2ade42c14406c8f9a4f44c100b0b84633b59c62bbff94d3b6f42ef224d08fc"
    },
    "dob": {
      "date": "1971-12-10T22:47:14.418Z",
      "age": 52
    },
    "registered": {
      "date": "2003-09-08T21:18:23.756Z",
      "age": 20
    },
    "phone": "69512312",
    "cell": "36344005",
    "id": {
      "name": "CPR",
      "value": "101271-8945"
    },
    "picture": {
      "large": "큰 이미지",
      "medium": "중간 이미지",
      "thumbnail": "썸네일"
    },
    "nat": "DK"
  }
]
```

그리고 타입은 아래와 같이 지정해 주었습니다.

```ts
// type.ts
export interface User {
  gender: string;
  name: {
    title: string;
    first: string;
    last: string;
  };
  location: {
    street: {
      number: number;
      name: string;
    };
    city: string;
    state: string;
    country: string;
    postcode: number;
    coordinates: {
      latitude: string;
      longitude: string;
    };
    timezone: {
      offset: string;
      description: string;
    };
  };
  email: string;
  login: {
    uuid: string;
    username: string;
    password: string;
    salt: string;
    md5: string;
    sha1: string;
    sha256: string;
  };
  dob: {
    date: string;
    age: number;
  };
  registered: {
    date: string;
    age: number;
  };
  phone: string;
  cell: string;
  id: {
    name: string;
    value: string;
  };
  picture: {
    large: string;
    medium: string;
    thumbnail: string;
  };
  nat: string;
}
```

### Column 정의하기

테이블에 대한 열을 정의합니다. 이 때 열은 아래의 코드처럼 `thaed`에 들어가는 `header`와 `tbody`에 들어가는 `cell` `tfoot`에 들어가는 `footer를 옵션으로 지정할 수 있습니다.`

또한 저는 간단하게 `name, image, email, location` 만 사용할 것이기 때문에 `column` 타입도 4가지로 간단하게 줄여주었습니다.

마지막으로 `use-column` 이라는 훅을 만들어 테이블에 사용할 `columnHelper`와 `colunm`을 분리시켜 주었습니다.

```ts
// type.ts
export interface IUserColunm {
  name: string;
  image: string;
  email: string;
  location: string;
}
```

```ts
// use-colunm.tsx
// columnHelper 만들기

import { createColumnHelper } from "@tanstack/react-table";
import { IUserColunm } from "./type";

export default function useColumn() {
  const columnHelper = createColumnHelper<IUserColunm>();
  const columns = [
    columnHelper.accessor("name", {
      header: "Name",
      cell: (info) => {
        const newName = info.getValue();
        return newName;
      },
    }),
    columnHelper.accessor("image", {
      header: "ProfileImage",
      cell: (info) => {
        return <img src={info.getValue()} alt="thumbnail" />;
      },
    }),
    columnHelper.accessor("email", {
      header: "Email",
    }),
    columnHelper.accessor("location", {
      header: "Location",
    }),
  ];

  return { columnHelper, columns };
}
```

### 사용하기

이제 다시 `app.tsx`로 돌아와 `table` 을 정의하고 사용합니다. 이 때 `userToUserColumn`를 통해 원래의 데이터를 컬럼타입에 맞게 재 가공합니다. 그리고 `tanstack table` 에서 제공하는 `useReactTable`에 가공한 데이터, `useColunm` 으로 불불러온 `colunms`, `getCoreRowModel`등을 설정하면 간단하게 `table`이 완성 됩니다.

```tsx
import {
  flexRender,
  getCoreRowModel,
  useReactTable,
} from "@tanstack/react-table";
import { useReducer, useState } from "react";
import { userData } from "./data";
import "./styles.css";
import { IUser, IUserColunm } from "./type";
import useColumn from "./use-column";

export default function App() {
  const [data, _setData] = useState(() => userToUserColumn(userData));
  const rerender = useReducer(() => ({}), {})[1];
  const { columns } = useColumn();

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
  });

  return (
    <div className="App">
      <h1>Hello CodeSandbox</h1>
      <table
        style={{
          width: "100%",
        }}
      >
        <thead>
          {table.getHeaderGroups().map((headerGroup) => (
            <tr key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <th key={header.id}>
                  {header.isPlaceholder
                    ? null
                    : flexRender(
                        header.column.columnDef.header,
                        header.getContext()
                      )}
                </th>
              ))}
            </tr>
          ))}
        </thead>
      </table>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <tr key={row.id}>
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id}>
                {flexRender(cell.column.columnDef.cell, cell.getContext())}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </div>
  );
}

// 현제 데이터를 컬럼타입에 맞게 가공
function userToUserColumn(users: IUser[]): IUserColunm[] {
  return users.map((user) => ({
    name: user.name.title + user.name.first + user.name.last,
    email: user.email,
    location: user.location.city,
    image: user.picture.medium,
  }));
}
```

## Tanstack Table의 다양한 기능들

저는 개인 프로젝트에 테이블의 헤더에 오름차순 정렬과 같은 기능을 적용해야 했기 때문에 `tanstack-table`에서 소개하는 기능에는 어떤 기능이 있는지 알아보겠습니다.

### 정렬

`tanstack-table`에서의 정렬은 기본적으로 영/숫자, 대/소문자, 날짜 기준으로 정렬할 수 있고, 커스텀으로 사용자가 함수를 지정하고 자바스크립트의 `sort()` 처럼 `-1 or 0 or 1`을 반환하면서 정렬을 할 수 있습니다.

### 사용하기

### Colunm 기능 설정

정렬 기능을 사용하기 위해선 우리가 일단 우리가 사용하고 있던 `colunms`의 배열을 수정해야 합니다. `columHelper.accessor`에 각 `colunm`마다 다양한 `sort`옵션을 할 수 있습니다. 옵션은 아래의 코드에서 주석으로 설명하겠습니다.

저는 다양한 옵션 중에 이미지로는 정렬하는것만 방지하고 나머지는 기본 값을 그대로 사용하였습니다.

```tsx
// use-column.tsx
const columns = [
  columnHelper.accessor("name", {
    header: "Name",
    cell: (info) => {
      const newName = info.getValue();
      return newName;
    },
    // sortDescFirst?: boolean 정렬 시 내림차순으로 시작합니다.
    // enableSorting?: boolean 정렬을 활성화 하거나 비활성화 합니다.
    // enableMultiSort?: boolean 다중 정렬을 활성화 하거나 비활성화 합니다.
    // invertSorting?: boolean 반전 정렬 (1이 가장 큰수) 로 정렬합니다.
    // sortUndefined : default :1
    // - first : 정의되지 않은 값은 목록의 맨 앞부분으로 밀려납니다.
    // - last : 정의되지 않은 값은 목록의 끝으로 밀려납니다.
    // - -1 : 정의되지 않은 값은 오름차순으로 정렬 됩니다.
    // - 1 : 정의되지 않은 값은 내림차순으로 정렬 됩니다.

    // 더 다양한 옵션은 공식 문서로..
  }),
  columnHelper.accessor("image", {
    header: "ProfileImage",
    cell: (info) => {
      return <img src={info.getValue()} alt="thumbnail" />;
    },
    enableSorting: false,
  }),
  columnHelper.accessor("email", {
    header: "Email",
  }),
  columnHelper.accessor("location", {
    header: "Location",
  }),
];
```

### Table 설정

이제 `App.tsx`에서 `useTable`에 있는 설정을 다시 해 줍니다. 기본적인 정렬은 `getSortedRowModel` 에 `getSortedRowModel()` 메소드를 전달하고 `state`로 정렬에 대한 상태를 설정합니다.

마지막으로 각 테이블 헤더에 `Sort`의 상태애 따른 조건부 렌더링을 보여주도록 설정하면 아주 쉽게 정렬을 구현하실 수 있습니다.

```tsx
export default function App() {
  const [data, _setData] = useState(() => userToUserColumn(userData));
  const rerender = useReducer(() => ({}), {})[1];
  const { columns } = useColumn();
  const [sorting, setSorting] = useState<SortingState>([]);

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    onSortingChange: setSorting,
    state: {
      sorting,
    },
  });

  return (
    <div className="App">
      <h1>Hello CodeSandbox</h1>
      <table
        style={{
          width: "100%",
        }}
      >
        <thead>
          {table.getHeaderGroups().map((headerGroup) => (
            <tr key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <th key={header.id}>
                  {header.isPlaceholder ? null : (
                    <div
                      onClick={header.column.getToggleSortingHandler()}
                      title={
                        header.column.getCanSort()
                          ? header.column.getNextSortingOrder() === "asc"
                            ? "Sort ascending"
                            : header.column.getNextSortingOrder() === "desc"
                            ? "Sort descending"
                            : "Clear sort"
                          : undefined
                      }
                    >
                      {flexRender(
                        header.column.columnDef.header,
                        header.getContext()
                      )}
                      {{
                        asc: " 🔼",
                        desc: " 🔽",
                      }[header.column.getIsSorted() as string] ?? null}
                    </div>
                  )}
                </th>
              ))}
            </tr>
          ))}
        </thead>
      </table>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <tr key={row.id}>
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id}>
                {flexRender(cell.column.columnDef.cell, cell.getContext())}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </div>
  );
}
```

### 데이터 검색

테이블에서 데이터가 많을 경우 어떤 사람이 있는지 찾을려면 `Sort` 기능으로는 아직 부족합니다. 따라서 사용자가 `Colunm Data`를 검색하면서 원하는 데이터를 찾을 수 있도록 필터링 기능을 구현해 보겠습니다.

검색 기능을 구현하려면 우선 `globalFilter` 라는 `state`를 추가합니다. 그리고 `useTable`의 `params`을 추가합니다. `param`은 다음과 같습니다.

- `globalFilterFn` 에서 필터가 어떻게 적용되었으면 좋겠는지 타입을 설정합니다.
- `onGlobalFilterChange`에서 필터가 변경될 때마다 실행할 메소드를 설정합니다.

`App.tsx`를 다음과 같이 수정합니다. 각각의 기능들은 주석을 통해 확인하실 수 있습니다.

```tsx
export default function App() {
  const [data, _setData] = useState(() => userToUserColumn(userData));
  const rerender = useReducer(() => ({}), {})[1];
  const { columns } = useColumn();
  const [sorting, setSorting] = useState<SortingState>([]);

  // 글로벌 검색 상태 추가
  const [globalFilter, setGlobalFilter] = useState("");

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    onSortingChange: setSorting,
    state: {
      sorting,
      globalFilter,
    },
    globalFilterFn: "includesString", // string이 포함되어있을 경우 (더 다양한 옵션이 있음)
    onGlobalFilterChange: setGlobalFilter,
  });

  return (
    <div className="App">
      <h1>Hello CodeSandbox</h1>
      <div>
        <DebouncedInput
          value={globalFilter ?? ""}
          onChange={(value) => setGlobalFilter(String(value))}
          className="p-2 font-lg shadow border border-block"
          placeholder="컬럼에 포함되는 단어를 검색해 보세요!"
        />
      </div>
      <table
        style={{
          width: "100%",
        }}
      >
        <thead>
          {table.getHeaderGroups().map((headerGroup) => (
            <tr key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <th key={header.id}>
                  {header.isPlaceholder ? null : (
                    <div
                      onClick={header.column.getToggleSortingHandler()}
                      title={
                        header.column.getCanSort()
                          ? header.column.getNextSortingOrder() === "asc"
                            ? "Sort ascending"
                            : header.column.getNextSortingOrder() === "desc"
                            ? "Sort descending"
                            : "Clear sort"
                          : undefined
                      }
                    >
                      {flexRender(
                        header.column.columnDef.header,
                        header.getContext()
                      )}
                      {{
                        asc: " 🔼",
                        desc: " 🔽",
                      }[header.column.getIsSorted() as string] ?? null}
                    </div>
                  )}
                </th>
              ))}
            </tr>
          ))}
        </thead>
      </table>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <tr key={row.id}>
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id}>
                {flexRender(cell.column.columnDef.cell, cell.getContext())}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </div>
  );
}

// 인풋 컴포넌트
function DebouncedInput({
  value: initialValue,
  onChange,
  debounce = 500,
  ...props
}: {
  value: string | number;
  onChange: (value: string | number) => void;
  debounce?: number;
} & Omit<React.InputHTMLAttributes<HTMLInputElement>, "onChange">) {
  const [value, setValue] = useState(initialValue);

  useEffect(() => {
    setValue(initialValue);
  }, [initialValue]);

  useEffect(() => {
    const timeout = setTimeout(() => {
      onChange(value);
    }, debounce);

    return () => clearTimeout(timeout);
  }, [value]);

  return (
    <input
      style={{
        padding: "10px",
        float: "left",
        minWidth: "350px",
      }}
      {...props}
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

### 페이지 네이션

데이터 테이블은 주로 모바일 보다는 데스크탑에서 많이 사용하고 있기 때문에 데이터가 많을 경우 페이지네이션을 사용하여 구현합니다. `tanstack-table` 에서 페이지 네이션을 구현하려면 `App.tsx`에 `state`로 페이지네이션 상태를 구현합니다.

```tsx
const [pagination, setPagination] = useState<PaginationState>({
  pageIndex: 0,
  pageSize: 10,
});
```

그리고 `usetable`에 `pagination` 과 `getPaginationRowModel`, `onPaginationChange` 상태를 추가합니다.

```tsx
const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
  onSortingChange: setSorting,
  state: {
    sorting,
    globalFilter,
    pagination, // 추가
  },
  globalFilterFn: "auto",
  onGlobalFilterChange: setGlobalFilter,
  getPaginationRowModel: getPaginationRowModel(), // 추가
  onPaginationChange: setPagination, // 추가
});
```

다음으로 페이지네이션을 생성하는 함수를 만들어 줍니다. 페이지 네이션을 생성하는 함수는
`useTable`에서 제공하는 다양한 변수 및 메소드를 이용하여

```tsx
// 페이지 번호를 생성하는 함수
const pageNumbers = () => {
  const totalPageCount = table.getPageCount();
  const pageIndex = table.getState().pagination.pageIndex;
  const pageRange = 5;

  let startPage = Math.max(0, pageIndex - pageRange);
  let endPage = Math.min(totalPageCount - 1, pageIndex + pageRange);

  if (pageIndex < pageRange) {
    endPage = Math.min(totalPageCount - 1, startPage + 2 * pageRange);
  }

  if (pageIndex > totalPageCount - pageRange - 1) {
    startPage = Math.max(0, endPage - 2 * pageRange);
  }

  return Array.from(
    { length: endPage - startPage + 1 },
    (_, i) => startPage + i
  );
};

// 50개의 데이터가 10개씩 있으므로
// [0, 1, 2, 3, 4] 페이지번호가 생성됩니다.
```

마지막으로 테이블의 아래쪽에 페이지네이션 버튼을 추가해 줍니다. 페이지 네이션 버튼은 `useTable`을 활용하여 이동버튼을 아까 생성했었던 `pageNumbers` 의 변수를 활용하여 페이지 번호 버튼을 생성해 줍니다.

```tsx
// ....이 위는 테이블 압나다

<div className="flex items-center gap-2">
  <button
    className="border rounded p-1"
    onClick={() => table.setPageIndex(0)}
    disabled={!table.getCanPreviousPage()}
  >
    {"<<"}
  </button>
  <button
    className="border rounded p-1"
    onClick={() => table.previousPage()}
    disabled={!table.getCanPreviousPage()}
  >
    {"<"}
  </button>
  {/* 페이지 번호 버튼 추가 */}
  {pageNumbers().map((page) => (
    <button
      key={page}
      className={`border rounded p-1 ${
        page === table.getState().pagination.pageIndex ? "bg-gray-200" : ""
      }`}
      onClick={() => table.setPageIndex(page)}
    >
      {page + 1}
    </button>
  ))}
  <button
    className="border rounded p-1"
    onClick={() => table.nextPage()}
    disabled={!table.getCanNextPage()}
  >
    {">"}
  </button>
  <button
    className="border rounded p-1"
    onClick={() => table.setPageIndex(table.getPageCount() - 1)}
    disabled={!table.getCanNextPage()}
  >
    {">>"}
  </button>

  {/* 몇 페이지 인지 확인*/}
  <span className="flex items-center gap-1">
    <div>Page</div>
    <strong>
      {table.getState().pagination.pageIndex + 1} of {table.getPageCount()}
    </strong>
  </span>
</div>
```

## 전체 코드 확인하기

전체적인 코드는 아래의 `codesandbox`에서 확인이 가능합니다. 직접 데이터를 조작해 보고 콘솔을 확인하면서 어떤 값이 들어오는지 확인해 보세요

<iframe src="https://codesandbox.io/embed/7kykhq?view=preview&module=%2Fsrc%2FApp.tsx&expanddevtools=1"
     style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-tanstack-table-exam"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 결론

제가 개인 프로젝트에 적용해본 기능은 정렬, 필터링 검색, 페이지네이션입니다. 그러나 tanstack-table에서는 이보다 더 많은 기능과 예제들이 아주 친절하게 설명되어 있기 때문에, 이 예제를 보고 기본 기능을 구현한 후 추가적인 기능이 필요하다면 아래의 공식 문서를 참고하여 추가할 기능을 확인해 보시기 바랍니다.

[https://tanstack.com/table/v8](https://tanstack.com/table/v8)
