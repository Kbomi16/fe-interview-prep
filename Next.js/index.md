> Pages Router API는 그대로 남아 있다. App Router의 캐시 기본값은 Next.js 15에서 바뀌었고, 16의 Cache Components(`cacheComponents`)는 켜야만 적용된다. 라우트 앞단 파일은 16부터 `proxy.ts`다. `middleware.ts`는 Edge가 필요할 때만 남기는 하위 호환이다.

### ✅ App Router와 Pages Router는 어떻게 다른가요?

| 구분 | Pages Router | App Router |
| ---- | ------------ | ---------- |
| 위치 | `pages/` | `app/` |
| UI 기본값 | 페이지 컴포넌트가 곧 라우트 | Server Component가 기본. 브라우저 상태는 `'use client'` |
| 데이터 | `getStaticProps`, `getServerSideProps` | 서버 컴포넌트에서 직접 `fetch`하거나 서버 함수 호출 |
| 레이아웃 | `_app`, `_document`가 전역 | 중첩 `layout.tsx`. 바뀌지 않은 레이아웃은 유지 |
| 로딩과 에러 | 페이지가 직접 처리 | `loading.tsx`, `error.tsx`, `not-found.tsx` |
| API | `pages/api` | `route.ts` Route Handler |
| i18n | `next.config`의 `i18n` | `app/[lang]`. 설정 파일의 `i18n`은 App 경로에서 지원하지 않음 |
| 요청 정보 | `context.req` | `cookies()`, `headers()`는 15부터 비동기 |

한 프로젝트에 `app`과 `pages`를 같이 둘 수 있습니다. 같은 경로를 양쪽에 정의하면 충돌합니다. 신규 화면은 App Router에 두고, 기존 페이지만 Pages에 남기는 이관이 가능합니다.

### ✅ 꼬리 질문) `getInitialProps`를 새 페이지에 써도 되나요?

Pages Router에서도 자동 정적 최적화를 끄기 때문에 신규 코드의 선택지가 아닙니다. App Router에는 해당 함수가 없습니다. 데이터가 필요하면 서버 컴포넌트에서 가져옵니다.

### ✅ App Router의 파일 컨벤션을 설명해 보세요.

- `page.tsx`: 그 경로에서 유니크하게 렌더되는 UI. 라우트를 외부에 공개합니다.
- `layout.tsx`: 자식이 바뀌어도 상태를 유지하는 껍데기. 루트 레이아웃은 `html`과 `body`를 가집니다.
- `template.tsx`: 레이아웃과 달리 내비게이션마다 새 인스턴스입니다.
- `loading.tsx`: 해당 세그먼트의 Suspense 폴백입니다.
- `error.tsx`: 클라이언트 에러 바운더리입니다. 루트 레이아웃의 에러는 `global-error.tsx`가 받습니다.
- `route.ts`: UI 없이 HTTP 핸들러입니다. 같은 세그먼트에 `page.tsx`와 함께 두지 않습니다.
- `not-found.tsx`: `notFound()`가 호출됐을 때의 UI입니다.

동적 세그먼트는 `[id]`, 캐치는 `[...slug]`입니다. 15부터 `params`와 `searchParams`는 Promise입니다.

```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  return <article>{id}</article>
}
```

### ✅ 꼬리 질문) 레이아웃은 언제 다시 렌더되나요?

같은 레이아웃 아래의 페이지만 바뀌면 레이아웃은 유지되고 자식만 바뀝니다. 레이아웃이 읽는 `params`가 바뀌거나, 그 레이아웃 파일이 의존하는 쿠키와 캐시가 무효화되면 다시 그립니다. 모든 페이지의 데이터를 루트 레이아웃에서 기다리면 하위 로딩 UI가 늦어집니다. 느린 조회는 그 세그먼트 쪽으로 내립니다.

### ✅ Server Component와 Client Component는 무엇이 다른가요?

Server Component는 서버에서만 실행됩니다. 데이터베이스와 비밀 키에 닿을 수 있고, 그 결과만 클라이언트로 보냅니다. `useState`, 이벤트 핸들러, 브라우저 API는 쓸 수 없습니다.

Client Component는 `'use client'`가 있는 모듈과 그 모듈이 import하는 클라이언트 그래프입니다. 상태와 이벤트를 쓸 수 있습니다. 이 지시어는 "브라우저에서만 실행"이 아닙니다. 특별한 설정을 하지 않으면 서버에서 HTML로 한 번 그리고, 브라우저에서 하이드레이트합니다.

서버에서 클라이언트로 넘기는 props는 직렬화되어야 합니다. 함수와 클래스 인스턴스는 넘기지 못합니다. 서버 전용 모듈을 클라이언트 파일이 import하면 번들 경계가 깨지므로 `server-only` 패키지로 그 실수를 빌드에서 막습니다.

### ✅ 꼬리 질문) 클라이언트 컴포넌트 안에 서버 컴포넌트를 import하면 어떻게 되나요?

그 서버 컴포넌트도 클라이언트 그래프에 들어갑니다. 서버 컴포넌트를 자식으로 두고 싶으면 서버 부모에서 만든 노드를 `children`으로 넘겨야 합니다. import 문장이 클라이언트 파일에 있으면 이미 늦은 것입니다.

### ✅ SSR, SSG, ISR은 어떻게 다른가요?

- **SSR**: 요청마다 서버가 HTML을 만듭니다. 사용자와 시각에 따라 내용이 달라질 때 씁니다.
- **SSG**: 빌드 때 HTML을 만들어 두고 요청에는 그 파일을 줍니다. 내용이 요청과 무관할 때 씁니다.
- **ISR**: SSG에 재생성 주기를 더한 것입니다. 주기가 지나면 다음 요청이 갱신을 트리거하고, 그 사이에는 만들어 둔 HTML을 줍니다. 글 수정처럼 시점을 알고 있으면 경로나 태그 무효화로 바로 갱신합니다.

App Router는 여기에 스트리밍을 더합니다. `loading.tsx`나 `<Suspense>` 경계 밖은 먼저 보내고, 느린 부분만 나중에 붙입니다.

### ✅ 꼬리 질문) Pages와 App에서 각각 어떻게 표현하나요?

Pages Router:

```js
export async function getStaticProps() {
  const posts = await getPosts()
  return { props: { posts }, revalidate: 60 }
}

export async function getServerSideProps() {
  const posts = await getPosts()
  return { props: { posts } }
}
```

동적 SSG의 경로는 `getStaticPaths`가 만듭니다.

App Router에서 캐시를 켜지 않은 조회는 요청마다 실행되어 SSR에 가깝습니다. 빌드에 고정할 동적 경로는 `generateStaticParams`가 만듭니다. 시간 기반 ISR은 세그먼트의 `export const revalidate = 60`이거나, 캐시하는 `fetch`의 `next.revalidate`입니다. `cookies()`나 `headers()`를 읽는 세그먼트는 그 순간 동적이 됩니다.

### ✅ Next.js 캐시는 어떤 계층인가요?

Cache Components를 켜지 않은 15와 16은 네 계층으로 설명합니다.

1. **Request Memoization**: 한 번의 서버 렌더 안에서 URL과 옵션이 같은 `fetch`를 한 번만 보냅니다. 렌더가 끝나면 사라집니다. 캐시 옵션과 별개로 동작합니다.
2. **Data Cache**: `fetch` 결과를 요청을 넘어 저장합니다. 배포 사이에도 남을 수 있습니다.
3. **Full Route Cache**: 정적 라우트의 HTML과 RSC 페이로드입니다. 빌드 때 또는 첫 방문 때 만들어집니다. 동적 라우트는 여기에 들어가지 않습니다.
4. **Router Cache**: 브라우저가 방문한 세그먼트의 RSC 페이로드를 기억합니다.

### ✅ 꼬리 질문) 13, 14와 15의 기본값은 어떻게 다른가요?

Next.js 13과 14 App Router는 `fetch`가 기본으로 Data Cache에 들어갔습니다(`force-cache`). GET Route Handler도 동적 함수가 없으면 캐시됐습니다. 14의 클라이언트 Router Cache는 동적 페이지를 약 30초 재사용했습니다.

Next.js 15부터는 다음이 기본입니다.

- `fetch`의 `cache`를 생략하면 Data Cache에 넣지 않습니다. `no-store`와 같습니다.
- GET Route Handler도 기본은 비캐시입니다. `sitemap`, `opengraph-image`, `icon` 같은 메타데이터 파일은 동적 함수가 없으면 정적으로 남습니다.
- 클라이언트에서 페이지 세그먼트의 `staleTime`은 0입니다. 이동할 때마다 그 페이지 데이터를 서버에서 다시 받습니다.
- 공유 레이아웃은 부분 렌더를 위해 다시 받지 않습니다. 뒤/앞으로 가기는 스크롤 복원을 위해 캐시에서 복원합니다. `loading.js`는 약 5분 유지됩니다.

15에서 이전처럼 페이지를 잠시 재사용하려면 `experimental.staleTimes.dynamic`을 줍니다. 데이터를 캐시하려면 `cache: 'force-cache'`나 `next: { revalidate, tags }`로 명시합니다.

### ✅ Cache Components는 무엇인가요?

Next.js 16에서 `cacheComponents: true`로 켜는 명시적 캐시입니다. 켜기 전에는 위의 15 모델이 유지됩니다. 켜면 캐시에 들어가는 코드는 `'use cache'`가 있는 함수와 컴포넌트뿐이고, 그 밖의 동적 코드는 요청 때 실행됩니다. 세그먼트 설정의 `dynamic`, `revalidate`, `fetchCache`는 이 모드에서 빌드 오류이므로 캐시 경계로 옮깁니다.

```tsx
import { cacheLife, cacheTag } from 'next/cache'

export async function getPosts() {
  'use cache'
  cacheLife('hours')
  cacheTag('posts')
  return fetch('https://example.com/posts').then((res) => res.json())
}
```

정적 셸을 먼저 보내고 캐시하지 않은 구멍을 Suspense로 채우는 Partial Prerendering도 이 설정으로 선택합니다. 15 카나리의 `experimental.ppr`과는 동작이 다릅니다.

### ✅ 꼬리 질문) `revalidateTag`와 `updateTag`는 어떻게 다른가요?

16에서 `revalidateTag('posts', 'max')`처럼 두 번째 인자로 `cacheLife` 프로필이 필요합니다. 한 인자 호출은 더 이상 쓰지 않습니다. 프로필을 주면 stale-while-revalidate라서, 잠시 낡은 응답을 보여주고 뒤에서 갱신합니다. 글을 저장한 사용자가 바로 새 글을 봐야 하면 Server Action에서 `updateTag('posts')`를 씁니다. 캐시를 건드리지 않고 캐시되지 않은 데이터만 다시 그리려면 `refresh()`를 씁니다. `revalidatePath`는 태그 대신 경로 단위로 같은 종류의 무효화를 겁니다.

### ✅ 꼬리 질문) `Cache-Control`과 Next 캐시는 같은 것인가요?

둘은 다른 계층입니다. `Cache-Control`은 브라우저와 CDN이 보는 HTTP 캐시입니다. Data Cache와 Full Route Cache는 Next 서버가 `fetch`와 렌더 결과를 저장하는 계층입니다. CDN의 `s-maxage`만 늘리고 Data Cache를 비우지 않으면, 서버는 새 값을 만들었는데 엣지가 옛 HTML을 줄 수 있습니다. 무효화할 계층을 답할 때 둘을 나눠 말합니다.

### ✅ Metadata API로 SEO를 어떻게 넣나요?

정적 메타는 `metadata` 객체, 데이터에 따라 달라지면 `generateMetadata`입니다. 제목 템플릿은 부모 레이아웃에 두고 자식은 앞부분만 바꿉니다. 15부터 `params`는 비동기로 받습니다.

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  metadataBase: new URL('https://example.com'),
  title: { default: '서비스', template: '%s | 서비스' },
  description: '서비스 소개',
  openGraph: { type: 'website', locale: 'ko_KR' },
}

export async function generateMetadata({
  params,
}: {
  params: Promise<{ slug: string }>
}): Promise<Metadata> {
  const { slug } = await params
  const post = await getPost(slug)
  return {
    title: post.title,
    description: post.excerpt,
    alternates: { canonical: `/posts/${slug}` },
  }
}
```

페이지마다 다른 설명과 canonical이 없으면 검색 결과가 한 문장으로 뭉칩니다. `metadataBase`가 있어야 OG 이미지의 상대 경로가 절대 URL이 됩니다.

### ✅ 꼬리 질문) `generateMetadata`가 느리면 무엇이 늦어지나요?

헤더에 넣을 제목을 기다리느라 그 세그먼트의 응답 시작이 늦어질 수 있습니다. 메타에 쓰는 조회는 본문과 같은 캐시 키를 타게 해서 두 번 치지 않게 합니다. 본문을 메타데이터 함수 안에서 조립하지는 않습니다.

### ✅ JSON-LD는 어디에 넣나요?

검색 엔진용 구조화 데이터는 `application/ld+json` 스크립트로 본문에 넣습니다. Metadata API의 title과 역할이 다릅니다. 메타는 탭과 소셜 카드, JSON-LD는 기사나 상품 같은 스키마입니다.

```tsx
export function ArticleJsonLd({
  title,
  publishedAt,
}: {
  title: string
  publishedAt: string
}) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: title,
    datePublished: publishedAt,
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
      }}
    />
  )
}
```

본문에 `</script>`가 섞이면 HTML이 끊깁니다. `<`를 유니코드 이스케이프로 바꿉니다. 스키마 타입은 페이지의 실제 내용과 같아야 하고, 보이지 않는 문장을 JSON-LD에만 넣지 않습니다.

### ✅ 꼬리 질문) 리스트 페이지에는 어떤 타입을 쓰나요?

목록은 `ItemList`, 글은 `Article`이나 `BlogPosting`, 상품은 `Product`처럼 보이는 정보에 해당하는 타입을 고릅니다. 필수에 가까운 필드를 지어내지 않습니다. 리치 결과 대상이 아니면 타입을 넣어도 검색 노출이 보장되지 않습니다.

### ✅ App Router의 i18n은 Pages와 어떻게 다른가요?

Pages Router는 `next.config`의 `i18n.locales`와 `defaultLocale`로 로케일 라우팅을 내장합니다. App Router 경로에는 이 설정을 쓰지 않습니다. 라우트는 `app/[lang]` 아래에 두고, `generateStaticParams`로 지원 로케일을 만듭니다. `Accept-Language`를 보고 접두사를 붙이는 일은 `proxy.ts`가 맡습니다. 번역 사전은 서버에서 로케일별로 불러오고, 클라이언트에는 그 화면에 필요한 메시지만 provider로 넘깁니다. `next-intl`이 이 구성을 많이 감쌉니다. 루트 레이아웃의 `<html lang>`은 현재 로케일과 같아야 합니다.

```tsx
export async function generateStaticParams() {
  return [{ lang: 'ko' }, { lang: 'en' }]
}
```

### ✅ 꼬리 질문) 로케일을 깊은 컴포넌트에서 어떻게 읽나요?

서버 컴포넌트는 세그먼트 `params`로 받습니다. Next.js 16.3부터는 루트 레이아웃 위의 동적 세그먼트를 `next/root-params`로 읽을 수 있습니다. 이 모듈은 클라이언트 컴포넌트, Server Action, Route Handler에서는 쓸 수 없습니다. 사전 전체를 클라이언트로 넘기면 쓰지 않는 언어의 문자열이 번들에 들어갑니다.

### ✅ 권한은 RBAC로 어떻게 나누나요?

사용자는 역할을 가지고, 역할은 `post:write`, `post:delete` 같은 권한을 가집니다. 업무 코드는 역할 이름 대신 권한을 검사하면 역할이 늘어도 분기가 늘지 않습니다. "작성자 본인"처럼 리소스 속성이 조건이면 역할 검사 옆에 소유자 검사를 둡니다. 권한의 원본은 서버입니다. 화면에서 버튼을 숨기는 것은 그 권한의 결과일 뿐, 요청을 거절하는 지점이 아닙니다.

### ✅ 꼬리 질문) 역할만 쿠키에 넣으면 무엇이 문제인가요?

쿠키의 역할은 사용자가 고칠 수 있습니다. 서명이 없는 역할로 관리자 화면을 열면 위조가 됩니다. 세션 식별자만 쿠키에 두고, 역할과 권한은 서버가 세션에서 읽습니다. 권한을 클라이언트에 실어 보낼 때는 화면 분기에만 쓰고, 같은 검사를 Server Action과 Route Handler에서 다시 합니다.

### ✅ 라우트 보호는 어디에 두나요?

Next.js 16의 요청 경계는 프로젝트 루트의 `proxy.ts`입니다. 함수 이름도 `proxy`이고 런타임은 Node.js입니다. Edge가 꼭 필요하면 deprecated된 `middleware.ts`를 유지합니다. 이름만 바꾸고 Edge 코드를 그대로 두면 런타임이 달라져 깨집니다.

```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  const hasSession = Boolean(request.cookies.get('session')?.value)
  const isPrivate = request.nextUrl.pathname.startsWith('/app')

  if (isPrivate && !hasSession) {
    const url = request.nextUrl.clone()
    url.pathname = '/login'
    url.searchParams.set('next', request.nextUrl.pathname)
    return NextResponse.redirect(url)
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/app/:path*'],
}
```

이 단계는 쿠키가 있는지만 보는 거친 분기입니다. 세션이 유효한지, 그 사용자가 그 리소스를 볼 수 있는지는 데이터를 읽는 서버 함수에서 다시 확인합니다. matcher를 너무 넓히면 정적 파일과 이미지 요청까지 이 함수를 탑니다.

### ✅ 꼬리 질문) 레이아웃에서 사용자를 기다리면 충분하지 않나요?

레이아웃은 경로 이동 때 유지될 수 있어, 한 번 읽은 사용자 정보가 오래 보일 수 있습니다. 인가가 필요한 조회는 페이지나 그 아래 데이터 함수에서 합니다. 서버 액션은 화면을 거치지 않고 호출할 수 있으므로 액션 첫줄에서 같은 권한 함수를 호출합니다. 프록시는 로그인이 필요한 경로의 조기 리다이렉트이고, 권한의 기준점은 아닙니다.
