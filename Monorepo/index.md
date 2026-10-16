### ✅ 프론트엔드에서 모노레포를 언제 쓰나요?

앱 여러 개와 공유 패키지를 한 저장소에서 같이 버전 관리할 때 씁니다. 디자인 시스템, 설정, 타입, SDK를 `packages`에 두고 `apps`의 웹과 관리자 화면이 같은 커밋에서 그 패키지를 소비합니다. 저장소가 하나라 원자적 변경이 됩니다. 배포 단위까지 하나라는 뜻은 아닙니다.

### ✅ 꼬리 질문) 저장소를 나누는 편이 나은 경우는 언제인가요?

릴리스 주기와 권한이 완전히 다르고, 공유 코드가 거의 없으면 저장소를 나누는 편이 단순합니다. 모노레포는 태스크 러너와 설치 도구가 없으면 설치와 CI가 전체로 퍼집니다.

### ✅ pnpm workspace는 어떻게 구성하나요?

루트의 `pnpm-workspace.yaml`이 패키지 글롭을 정합니다.

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

패키지 의존은 `workspace:*`로 잇습니다.

```json
{
  "dependencies": {
    "@repo/ui": "workspace:*"
  }
}
```

`pnpm install`은 워크스페이스 패키지를 레지스트리가 아니라 로컬 경로에 연결합니다. 버전 범위는 퍼블리시할 때의 규칙이고, 로컬에서는 현재 소스입니다.

### ✅ 꼬리 질문) 팬텀 의존성이 왜 막히나요?

pnpm은 의존성을 패키지별로 격리합니다. `package.json`에 없는 패키지를 import하면 해석에 실패합니다. npm의 평탄한 `node_modules`에서는 다른 패키지가 끌어온 모듈이 우연히 보여서, 그 패키지를 지우면 갑자기 깨집니다. 이 격리가 워크스페이스에서 패키지 경계를 유지하는 이유입니다. 도구가 호이스트를 요구하면 그 패키지에만 `public-hoist-pattern`을 열고, 저장소 전체를 `shamefully-hoist`로 풀지는 않습니다.

### ✅ 꼬리 질문) catalog는 무엇인가요?

워크스페이스가 같은 라이브러리 버전을 한곳에서 고르게 할 때 씁니다. `pnpm-workspace.yaml`의 `catalog`에 `react`, `next` 버전을 적고 패키지에서는 `"react": "catalog:"`로 참조합니다. 앱마다 버전을 따로 올리다가 타입이 어긋나는 일을 줄입니다.

### ✅ Turborepo는 무엇을 담당하나요?

패키지 스크립트의 실행 순서와 캐시입니다. 현재 설정 키는 `tasks`입니다. 예전 키 `pipeline`은 Turbo 2에서 `tasks`로 바뀌었습니다.

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "lint": {},
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

`^build`는 의존 패키지의 `build`가 먼저 끝나야 한다는 뜻입니다. `dev`는 끝나지 않는 프로세스라 캐시하지 않고 `persistent`로 표시합니다.

### ✅ 꼬리 질문) 캐시는 무엇을 보고 히트하나요?

입력 파일, 의존 패키지 해시, 태스크에 넘긴 환경 변수, `outputs`입니다. 출력을 빠뜨리면 캐시 히트 뒤 산출물이 없습니다. `.env`의 비밀이 해시와 로그에 들어가지 않게 `globalEnv`와 `env`에 필요한 키만 적습니다. 원격 캐시를 연결하면 CI와 로컬이 같은 빌드 산출물을 재사용합니다.

### ✅ 꼬리 질문) 바뀐 패키지만 CI에서 어떻게 돌리나요?

`turbo run build --filter=web...`은 web과 web이 의존하는 패키지입니다. `...web`은 web에 의존하는 패키지입니다. 변경분만 빌드하려면 `--filter=...[origin/main]`으로 기준 커밋 이후 바뀐 패키지와, 그 패키지에 의존하는 쪽을 고릅니다.

### ✅ 패키지 공개 표면은 어떻게 자르나요?

공유 패키지는 `exports`에 진입점만 적습니다. 앱은 그 진입점만 import하고, 패키지 내부 파일을 깊은 경로로 가져오지 않습니다. Next 앱이 TypeScript 소스를 직접 쓰면 `transpilePackages`에 패키지 이름을 넣습니다. 패키지가 이미 자바스크립트로 빌드되면 그 설정 없이 `exports`의 빌드 결과를 씁니다.

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  transpilePackages: ['@repo/ui'],
}

module.exports = nextConfig
```

의존 방향은 앱에서 패키지로만 갑니다. 패키지가 특정 앱을 import하면 그 패키지는 더 이상 공유 모듈이 아닙니다.

### ✅ 꼬리 질문) 내부 패키지 버전은 어떻게 맞추나요?

사설 패키지는 `"private": true`로 두고 워크스페이스 프로토콜로 연결하면 레지스트리 버전과 어긋나지 않습니다. 외부에 배포하는 패키지만 Changesets 같은 도구로 버전과 체인지로그를 올립니다. 앱 배포는 패키지 버전과 분리해, 해당 앱의 파이프라인만 태우면 됩니다.
