> 답은 TanStack Query v5(`@tanstack/react-query`) 기준이다. v3/v4의 `cacheTime`, `keepPreviousData`, `status: 'loading'`, `useQuery`의 `onSuccess`는 현재 API가 아니다.

### ✅ TanStack Query는 어떤 상태를 다루나요?

서버 상태를 다룹니다. 서버에 원본이 있고, 클라이언트는 잠시 빌려 보여주는 데이터입니다. 목록 조회, 캐시, 백그라운드 갱신, 재시도, 뮤테이션 후 무효화가 이 라이브러리의 일입니다. 모달 열림이나 입력 중인 폼 값 같은 클라이언트 상태는 `useState`나 다른 스토어에 둡니다.

### ✅ 꼬리 질문) Redux 같은 스토어와 역할이 어떻게 다른가요?

Redux는 앱이 소유한 상태를 액션으로 바꾸는 저장소입니다. 서버 목록을 넣으면 로딩, 에러, 재조회, 캐시 폐기를 직접 짜게 됩니다. TanStack Query는 요청을 쿼리 키로 식별하고 그 수명 주기를 대신 관리합니다. 둘을 같이 쓸 수는 있지만, 서버 응답을 전역 스토어에 다시 복사하면 캐시가 두 곳이 됩니다.

### ✅ 쿼리 키는 어떻게 잡나요?

쿼리 키는 캐시 주소입니다. 결과가 달라지는 모든 입력이 키에 들어갑니다.

```ts
['posts', { status: 'open', page }]
['posts', postId]
```

계층으로 잡으면 `['posts']` 무효화로 목록과 상세를 함께 갱신할 수 있습니다. 키 배열의 객체는 깊게 해시되므로 키 순서가 바뀌어도 같은 객체면 같은 캐시입니다. 함수나 JSX를 키에 넣지 않습니다.

### ✅ 꼬리 질문) 의존 쿼리는 어떻게 쓰나요?

앞 요청의 결과가 있어야 다음 요청이 가능하면 `enabled`로 막습니다.

```ts
useQuery({
  queryKey: ['author', post?.authorId],
  queryFn: () => fetchAuthor(post!.authorId),
  enabled: Boolean(post?.authorId),
})
```

`enabled`가 거짓이면 요청하지 않고 상태는 `pending`으로 남을 수 있습니다. 화면은 "아직 조건이 안 됨"과 "요청 중"을 구분해서 그립니다.

### ✅ `staleTime`과 `gcTime`은 무엇이 다른가요?

- `staleTime`: 데이터가 신선한 시간. 기본값은 `0`이라 받자마자 stale이 되고, 마운트·창 포커스·재연결 때 다시 가져옵니다.
- `gcTime`: 쿼리를 아무도 구독하지 않은 뒤 캐시를 메모리에서 지울 때까지의 시간. 기본값은 5분입니다. v4까지의 이름 `cacheTime`이 이 옵션입니다. 신선한지와는 관계가 없습니다.

SSR에서의 기본 `gcTime`은 `Infinity`입니다. 요청이 끝나기 전에 캐시가 버려지지 않게 하려는 값입니다.

### ✅ 꼬리 질문) 새로고침할 때마다 바로 다시 요청되는 이유는 무엇인가요?

`staleTime`이 0이면 캐시에 데이터가 있어도 stale입니다. 하이드레이션 직후 클라이언트가 한 번 더 요청하는 패턴이 여기서 나옵니다. 자주 안 바뀌는 데이터는 `staleTime`을 60초처럼 0보다 크게 둡니다. `gcTime`을 늘리는 것과는 다른 문제입니다. `gcTime`은 언마운트된 뒤의 보관 시간입니다.

### ✅ `isPending`, `isLoading`, `isFetching`은 어떻게 다른가요?

v5의 상태 이름은 `pending | error | success`입니다. v4의 `status: 'loading'`은 `pending`으로 바뀌었습니다.

- `isPending`: 아직 보여줄 데이터가 없습니다.
- `isFetching`: 요청이 진행 중입니다. 백그라운드 재조회도 포함합니다.
- `isLoading`: `isPending && isFetching`입니다. 데이터가 없고 첫 요청이 날아가는 중일 때만 참입니다.
- `isFetched` 이후에 다시 가져오면 `isPending`은 거짓이고 `isFetching`만 참입니다. 이때 스피너로 목록을 갈아끼우면 화면이 깜빡입니다.

### ✅ 꼬리 질문) `fetchStatus`는 왜 따로 있나요?

`status`는 데이터의 유무와 성공 여부이고, `fetchStatus`는 네트워크입니다. `fetching`, `paused`, `idle`이 있습니다. 오프라인에서 요청이 멈추면 `paused`로 구분할 수 있습니다.

### ✅ 페이지를 넘겨도 이전 목록을 유지하려면 어떻게 하나요?

v4의 `keepPreviousData: true`와 `isPreviousData`는 제거되었습니다. `placeholderData: keepPreviousData`를 쓰고, 자리 표시 데이터인지는 `isPlaceholderData`로 봅니다.

```tsx
import { keepPreviousData, useQuery } from '@tanstack/react-query'

const { data, isPending, isPlaceholderData } = useQuery({
  queryKey: ['projects', page],
  queryFn: () => fetchProjects(page),
  placeholderData: keepPreviousData,
})
```

키가 바뀌는 동안 이전 데이터를 보여주고, 새 데이터가 오면 교체합니다. 다음 페이지 버튼은 `isPlaceholderData`일 때 잠그면 연속 클릭으로 요청이 겹치지 않습니다.

### ✅ 꼬리 질문) `initialData`와 `placeholderData`는 어떻게 다른가요?

`initialData`는 캐시에 실제로 들어갑니다. 신선한 데이터로 취급될 수 있어 `staleTime` 계산에도 포함됩니다. `placeholderData`는 관찰하는 동안만 보여주는 값이고 캐시에 기록되지 않습니다. 서버에서 받아 온 진짜 초기값이 있으면 `initialData`, 페이지 전환의 이전 화면을 붙잡는 용도면 `placeholderData`입니다.

### ✅ `useQuery`의 `onSuccess`는 어디로 갔나요?

v5의 `useQuery`에는 `onSuccess`, `onError`, `onSettled`가 없습니다. 렌더마다 콜백을 거는 방식이 시점 보장이 어려워 빠졌습니다. 성공 데이터로 파생 값을 만들 때는 렌더 중에 계산하거나 `select`를 씁니다. 요청 성공 후 다른 시스템을 건드리려면 그 효과가 정말 필요할 때만 `useEffect`로 분리합니다.

`useMutation`의 `onSuccess`, `onError`, `onSettled`는 남아 있습니다. 캐시 전역 부수 효과는 `QueryClient`의 `QueryCache`에 둡니다.

### ✅ 꼬리 질문) 에러를 에러 바운더리로 보내려면 어떻게 하나요?

v4의 `useErrorBoundary`는 `throwOnError`입니다. 참이면 쿼리 에러가 렌더 중으로 던져져 가장 가까운 에러 바운더리가 받습니다. 토스트로 충분한 뮤테이션 에러는 바운더리까지 보내지 않고 `onError`에서 처리합니다.

### ✅ 뮤테이션 뒤에 목록을 어떻게 맞추나요?

서버가 원본이므로, 성공 후 관련 쿼리를 무효화해 다시 받게 합니다.

```tsx
useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['posts'] })
  },
})
```

`invalidateQueries`는 활성 쿼리를 다시 가져오고, 비활성 쿼리는 stale로만 표시했다가 다음에 구독되면 가져옵니다. 부분 키가 맞으면 상세 쿼리도 함께 무효화됩니다.

### ✅ 꼬리 질문) 낙관적 업데이트는 어떻게 구성하나요?

`onMutate`에서 진행 중인 재조회를 취소하고, 이전 캐시를 컨텍스트로 반환한 뒤 `setQueryData`로 화면을 먼저 바꿉니다. `onError`에서 컨텍스트의 이전 값으로 되돌리고, `onSettled`에서 무효화해 서버 값으로 수렴시킵니다. 롤백 없는 낙관적 업데이트는 실패 시 화면과 서버가 어긋납니다.

### ✅ App Router에서 QueryClient는 어디에 두나요?

`QueryClientProvider`는 클라이언트 컴포넌트입니다. 브라우저에서는 클라이언트를 한 번만 만들고, 서버에서는 요청마다 새로 만듭니다. 모듈 스코프의 싱글턴을 서버에서 공유하면 사용자 A의 캐시가 사용자 B 렌더에 섞입니다.

```tsx
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useState } from 'react'

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [client] = useState(() => new QueryClient())
  return <QueryClientProvider client={client}>{children}</QueryClientProvider>
}
```

서버 컴포넌트에서 `prefetchQuery`한 뒤 `dehydrate`하고, 클라이언트는 `HydrationBoundary`로 그 상태를 받습니다. 이렇게 해야 첫 페인트에 데이터가 있고, 클라이언트가 같은 키로 즉시 한 번 더 치지 않게 `staleTime`을 맞출 수 있습니다.

### ✅ 꼬리 질문) 무한 스크롤 API는 v4와 무엇이 다른가요?

`useInfiniteQuery`는 `initialPageParam`이 필수입니다. `getNextPageParam`은 마지막 페이지와 전체 페이지 배열, 마지막 페이지 파라미터를 받습니다. 다음 커서가 없으면 `undefined`를 반환해 `hasNextPage`를 거짓으로 만듭니다. `pageParams`와 `pages`는 캐시에 같이 들어 있으므로, 낙관적 업데이트 때 둘의 길이를 같이 맞춰야 합니다.
