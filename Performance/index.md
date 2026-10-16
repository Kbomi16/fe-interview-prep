> Core Web Vitals는 LCP, INP, CLS다. 2024-03-12부터 INP가 FID를 대체했다. FID를 상호작용 지표로 답하면 현재 기준과 어긋난다.

### ✅ Core Web Vitals는 무엇을 재나요?

현장 사용자의 75퍼센타일로 보는 세 가지 지표입니다.

| 지표 | 보는 것 | 좋음 | 나쁨 |
| ---- | ------- | ---- | ---- |
| LCP (Largest Contentful Paint) | 가장 큰 콘텐츠가 그려지기까지 | 2.5초 이하 | 4초 초과 |
| INP (Interaction to Next Paint) | 상호작용 뒤 다음 페인트까지 | 200ms 이하 | 500ms 초과 |
| CLS (Cumulative Layout Shift) | 예기치 않은 레이아웃 이동 | 0.1 이하 | 0.25 초과 |

Lighthouse 점수는 실험실 측정입니다. 응답성 자리에는 Total Blocking Time이 들어가고, INP 자체를 대체하지는 않습니다. 출시 판단은 CrUX나 RUM 같은 필드 데이터로 합니다.

### ✅ 꼬리 질문) FCP와 TTFB는 Core Web Vital인가요?

둘 다 아닙니다. FCP는 첫 콘텐츠가 그려지는 시점이고, TTFB는 응답 첫 바이트까지입니다. LCP를 설명할 때 원인을 나누는 보조 지표로 말합니다. TTI도 Core Web Vital이 아닙니다.

### ✅ INP는 무엇이고 FID와 어떻게 다른가요?

INP는 방문 동안의 클릭, 탭, 키보드 입력 중 가장 느린 상호작용(이상치는 제외)이 다음 프레임을 그리기까지 걸린 시간입니다. 입력 지연, 이벤트 처리, 화면 반영이 모두 포함됩니다. FID는 첫 입력의 지연만 봤기 때문에, 처리와 페인트가 느린 페이지도 좋게 나올 수 있었습니다.

### ✅ 꼬리 질문) INP를 어떻게 줄이나요?

클릭 핸들러 안의 긴 동기 작업을 줄입니다. 급하지 않은 일은 다음 프레임 이후로 미루고, 큰 목록 렌더는 가상화하거나 입력이 끝난 뒤에 돌립니다. 서드파티 스크립트가 메인 스레드를 오래 잡으면 입력 지연이 그대로 INP에 더해집니다. `startTransition`은 긴급하지 않은 렌더의 우선순위를 낮추지만, 핸들러 자체의 긴 계산을 없애 주지는 않습니다.

### ✅ LCP는 무엇이고 어떻게 줄이나요?

LCP는 뷰포트에서 가장 크게 보이는 이미지나 텍스트 블록이 그려지는 시점입니다. 로딩 지표이고, 레이아웃 안정성 지표가 아닙니다.

개선은 후보 요소를 먼저 찾는 일부터입니다.

- 히어로 이미지는 `next/image`의 `priority`로 프리로드하고, `sizes`를 실제 표시 크기에 맞춥니다.
- 첫 화면에 필요한 CSS와 폰트만 막고, 나머지는 뒤로 보냅니다.
- 서버에서 HTML을 보내 LCP 텍스트가 JS 뒤에 오지 않게 합니다.
- TTFB가 크면 렌더를 최적화해도 LCP가 줄지 않습니다. 캐시와 데이터 조회를 같이 봅니다.

### ✅ 꼬리 질문) LCP 이미지에 `priority`를 여러 장 주면 어떻게 되나요?

브라우저의 초기 대역폭을 여러 프리로드가 나눕니다. LCP 후보는 보통 한 장입니다. 나머지 이미지는 기본 지연 로딩에 둡니다.

### ✅ CLS는 왜 생기고 어떻게 막나요?

자리 없는 요소가 나중에 밀고 들어올 때 생깁니다. 이미지와 영상은 너비와 높이를 미리 주고, 광고·임베드는 최소 높이를 비워 둡니다. 폰트는 폴백과 메트릭이 비슷한 글꼴로 바뀌게 하고, 기존 콘텐츠 위에 배너를 끼워 넣지 않습니다. 애니메이션은 `top`이나 `height` 대신 `transform`과 `opacity`를 씁니다.

### ✅ 꼬리 질문) 사용자가 연 아코디언도 CLS인가요?

CLS는 예기치 않은 이동만 봅니다. 입력 뒤 500ms 안의 이동은 사용자 기대로 보고 제외하는 규칙이 있습니다. 클릭과 무관하게 늦게 뜬 배너가 본문을 밀면 점수에 들어갑니다.

### ✅ `next/image`는 무엇을 최적화하나요?

빌드와 요청 시점에 크기를 줄이고, `avif`와 `webp`처럼 설정한 포맷으로 바꿉니다. `width`와 `height`, 또는 `fill`과 `sizes`가 있어야 자리가 정해져 CLS를 피합니다. 원격 이미지는 `images.remotePatterns`에 호스트를 적습니다. `images.domains`는 Next.js 16에서 deprecated입니다.

```tsx
import Image from 'next/image'

export function Hero() {
  return (
    <Image
      src="https://cdn.example.com/hero.jpg"
      alt="출시 행사 현장"
      width={1200}
      height={630}
      sizes="(max-width: 768px) 100vw, 720px"
      priority
    />
  )
}
```

### ✅ 꼬리 질문) 모든 이미지를 `next/image`로 바꿔야 하나요?

콘텐츠 이미지는 바꾸는 편이 맞습니다. 아이콘처럼 작고 색이 고정된 SVG는 인라인이나 SVGR이 요청 수를 더 줄입니다. 사용자가 올린 원본을 가공 없이 내려야 하는 다운로드 링크에는 최적화를 켜지 않습니다.

### ✅ `next/font`는 무엇을 해결하나요?

글꼴 파일을 빌드 때 받아 같은 출처에서 제공합니다. 런타임에 글꼴 CDN으로 가는 연결이 없고, 폴백 글꼴의 크기를 조정해 본문이 바뀌며 밀리는 일을 줄입니다. `display: 'swap'`이면 글꼴이 오기 전에 폴백으로 글을 보여 줍니다.

```tsx
import { Noto_Sans_KR } from 'next/font/google'

const noto = Noto_Sans_KR({
  subsets: ['latin'],
  weight: ['400', '700'],
  display: 'swap',
})

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body className={noto.className}>{children}</body>
    </html>
  )
}
```

`Noto_Sans_KR`은 서브셋 이름이 `latin`으로만 등록된 경우가 많아서, 그 값을 넣어도 한글 글리프가 포함됩니다. 굵기를 늘리면 받지 않아도 되는 파일까지 커지므로 쓰는 굵기만 지정합니다.

### ✅ 꼬리 질문) 링크한 CSS 글꼴과 무엇이 다른가요?

`<link>`로 받은 글꼴은 발견되기 전까지 요청이 시작되지 않고, 다른 출처면 연결 비용이 붙습니다. `next/font`는 사용할 파일을 미리 알고 preload와 폴백 메트릭을 같이 넣습니다.

### ✅ 코드 스플리팅은 어디서 일어나나요?

App Router는 라우트 세그먼트 단위로 클라이언트 번들을 나눕니다. 홈에만 쓰는 무거운 컴포넌트를 루트 레이아웃에 두면 모든 페이지가 그 코드를 받습니다. 특정 화면에서만 쓰는 차트나 에디터는 `next/dynamic`으로 분리합니다.

```tsx
'use client'

import dynamic from 'next/dynamic'

const Chart = dynamic(() => import('./chart'), {
  ssr: false,
  loading: () => <p>차트를 불러오는 중</p>,
})
```

`ssr: false`는 클라이언트 컴포넌트 안에서만 쓸 수 있습니다. 서버 컴포넌트에서 쓰면 빌드가 실패합니다. `ssr: false`는 그 덩어리를 HTML에 넣지 않으므로, 검색과 첫 페인트에 필요한 UI에는 쓰지 않습니다.

### ✅ 꼬리 질문) 잘게 나눌수록 항상 빠른가요?

청크마다 요청과 런타임 오버헤드가 있습니다. 라우트 단위와, 초기 화면에 없어도 되는 큰 라이브러리 단위가 보통의 경계입니다. 배럴 파일에서 라이브러리 전체를 다시 export하면 트리 셰이킹이 무너져 스플리팅을 해도 초기 번들이 줄지 않습니다. 번들 분석기로 초기 경로에 들어간 모듈을 확인합니다.
