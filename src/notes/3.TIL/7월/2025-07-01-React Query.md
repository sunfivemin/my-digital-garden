---
tags:
  - TIL
  - React
created: 2025-07-01
---
# 📘 2025-07-01 TIL

## 📌 오늘 배운 핵심 요약
- 기존 useEffect + useState 기반의 도서 목록 패칭 로직을 React Query로 전환
- 쿼리스트링을 기반으로 동적 요청 및 자동 캐싱 적용
- 로딩 상태는 BooksLoading 컴포넌트로 Skeleton UI 구성
- 상태 관리와 UI 렌더링의 관심사를 깔끔하게 분리


## 🧠 상세 학습 내용
## 📍 주제 1: React Query를 활용한 도서 목록 관리 리팩토링

### ✅ 기존 구조: useEffect + useState + fetchBooks

도서 목록 페이지에서는 다음과 같이 useEffect, useState를 사용해 API 요청을 처리하고 있었습니다.
```tsx
useEffect(() => {
  fetchBooks({ ...params }).then(res => {
    setBooks(res.books);
    setPagination(res.pagination);
  });
}, [location.search]);
```

이 방식은 단순하지만 다음과 같은 단점이 있습니다:
- 로딩, 에러 상태를 직접 관리해야 함
- 동일 요청에 대해 캐싱이 불가능하여 UX 낭비 발생
- 중복 요청 제어가 어려움
- 데이터 패칭과 UI 렌더링 로직이 훅 내부에 섞여 구조가 지저분해짐

---

### 💡  React Query로 리팩토링한 이유

React Query는 위의 문제들을 해결해주는 매우 강력한 라이브러리입니다. 

|**장점**|**설명**|
|---|---|
|자동 캐싱|같은 요청이면 서버에 가지 않고 캐시에서 가져옴|
|로딩/에러 관리 내장|isLoading, isError 등으로 쉽게 상태 표현 가능|
|쿼리 키 기반 요청 제어|queryKey만 변경되면 자동으로 refetch|
|관심사 분리|데이터 패칭 로직과 UI 렌더링 분리 가능|


---

### 📦 구현: useBooks.ts 훅
```tsx
// src/hooks/useBooks.ts
import { useLocation } from 'react-router-dom';
import { useQuery } from '@tanstack/react-query';
import { fetchBooks } from '@/api/books.api';
import type { FetchBooksResponse } from '@/api/books.api';
import { QUERYSTRING } from '@/constants/querystring';
import { LIMIT } from '@/constants/pagination';

export const useBooks = () => {
  const location = useLocation();
  const params = new URLSearchParams(location.search);

  const category_id = params.get(QUERYSTRING.CATEGORY_ID)
    ? Number(params.get(QUERYSTRING.CATEGORY_ID))
    : undefined;

  const news = params.get(QUERYSTRING.NEWS) === 'true' ? true : undefined;

  const currentPage = params.get(QUERYSTRING.PAGE)
    ? Number(params.get(QUERYSTRING.PAGE))
    : 1;

  const {
    data: booksData,
    isLoading: isBooksLoading,
    isError,
  } = useQuery<FetchBooksResponse>({
    queryKey: ['books', location.search],
    queryFn: () =>
      fetchBooks({
        category_id,
        news,
        currentPage,
        limit: LIMIT,
      }),
    staleTime: 1000 * 60 * 5, // 5분 동안 캐시 유지
  });

  return {
    books: booksData?.books || [],
    pagination: booksData?.pagination || { totalCount: 0, currentPage: 1 },
    isEmpty: booksData?.books.length === 0,
    isBooksLoading,
    isError,
  };
};
```

---

### 🖼 로딩 상태 처리: BooksLoading

로딩 중일 때 보여줄 Skeleton UI도 따로 컴포넌트로 분리했습니다.
```tsx
// src/components/books/BooksLoading.tsx
import { Skeleton } from '@/components/ui/Skeleton';

const BooksLoading = () => {
  return (
    <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4 w-full">
      {Array.from({ length: 10 }).map((_, i) => (
        <div key={i} className="space-y-2">
          <Skeleton className="h-[160px] w-full rounded-md" />
          <Skeleton className="h-4 w-3/4" />
          <Skeleton className="h-4 w-1/2" />
        </div>
      ))}
    </div>
  );
};

export default BooksLoading;
```

---

### 📄 최종 적용 예시: Books.tsx 페이지
```tsx
function Books() {
  const { books, pagination, isBooksLoading, isEmpty } = useBooks();

  return (
    <section className={mainContainer}>
      <div className="flex justify-between items-center w-full">
        <Title size="lg" color="primary">도서 검색 결과</Title>
        <BooksViewSwitcher />
      </div>

      <div className="mt-6">
        <BooksFilter />
      </div>

      <div className="mt-8 min-h-[600px] flex items-center justify-center">
        {isBooksLoading ? (
          <BooksLoading />
        ) : isEmpty ? (
          <BooksEmpty />
        ) : (
          <BooksList books={books} />
        )}
      </div>

      <div className="mt-8">
        <Pagination pagination={pagination} />
      </div>
    </section>
  );
}
```

| **항목** | **기존 방식**         | **React Query 도입 후**          |
| ------ | ----------------- | ----------------------------- |
| 상태 관리  | 직접 상태 관리          | 자동 상태 관리 (isLoading, isError) |
| 요청 방식  | useEffect + fetch | useQuery로 간결화                 |
| 중복 요청  | 직접 제어 필요          | 자동 캐싱 및 제어                    |
| UI 분리  | 데이터 로직과 섞임        | 관심사 분리 가능                     |

![BooksList](https://seonohblog.netlify.app/assets/BooksList.png)

---

## 💭 회고
- **새롭게 알게 된 점**
   - React Query를 사용하면 API 요청과 상태 관리를 매우 효율적으로 처리할 수 있으며, 코드가 훨씬 간결해진다.
   - 특히 queryKey, staleTime 등의 옵션을 잘 활용하면 UX도 좋아지고 불필요한 재요청을 막을 수 있다.

- **어렵게 느껴졌던 부분**
   - 쿼리 키 구성에서 객체를 그대로 넣었을 때 캐싱이 안 되는 경우가 있어 queryKey를 어떻게 설계해야 할지 고민이 필- 쿼리 키 설정이나 타입 에러가 처음에는 헷갈렸다.
   - 특히 queryKey에 location.search나 객체를 넘길 때 캐싱이 의도대로 작동하는지 확인이 필요하다.

- **다음에 학습할 주제**
  - 모킹 서버, 작성리뷰, 다양한 UI

### 🔗 참고자료
- [React Query 공식 문서](https://tanstack.com/query/latest/docs/framework/react/overview)