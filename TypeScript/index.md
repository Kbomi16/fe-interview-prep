> 답은 프론트엔드에서 쓰는 TypeScript 5와 Next.js가 생성하는 `tsconfig`를 기준으로 했다.

### ✅ TypeScript를 왜 쓰나요?

TypeScript는 자바스크립트에 정적 타입을 얹은 언어입니다. 컴파일 전에 잘못된 인자, `null` 접근, 응답 필드 오타를 잡고, 에디터에서 자동완성과 리팩터링을 가능하게 합니다. 런타임에 타입이 남는 것은 아니고, 지워진 뒤의 결과는 자바스크립트입니다.

### ✅ 꼬리 질문) `any`와 `unknown`은 어떻게 다른가요?

`any`는 타입 검사를 끕니다. 어떤 값이든 넘기고 어떤 프로퍼티든 읽을 수 있어 오류가 호출하는 쪽으로 퍼집니다. `unknown`은 값이 아직 검증되지 않았다는 뜻이라, 타입을 좁히거나 타입 가드를 통과하기 전에는 프로퍼티 접근이 거절됩니다. 외부 JSON처럼 출처를 믿을 수 없는 값은 `unknown`으로 받고 좁힙니다.

### ✅ 인터페이스와 타입 별칭은 무엇이 다른가요?

둘 다 객체의 모양을 기술할 수 있습니다. `interface`는 같은 이름으로 다시 선언하면 병합되고, `extends`로 확장합니다. `type`은 유니온, 인터섹션, 튜플, 매핑 타입처럼 인터페이스가 표현하지 못하는 형태에 씁니다. 공개 API의 객체 계약은 인터페이스로, 유니온과 유틸리티 조합은 타입 별칭으로 두는 경우가 많습니다.

### ✅ 꼬리 질문) 유니온 타입은 어디에 쓰나요?

하나의 값이 여러 형태 중 하나일 때 씁니다. `type Status = 'idle' | 'loading' | 'success' | 'error'`처럼 허용 값을 제한하거나, `{ type: 'a'; a: string } | { type: 'b'; b: number }`처럼 판별 필드를 두면 `type`으로 좁힌 뒤 해당 필드만 안전하게 읽을 수 있습니다.

### ✅ 구조적 타이핑이란 무엇인가요?

TypeScript는 이름이 아니라 모양을 봅니다. 필요한 프로퍼티를 모두 가진 값이면, 선언한 클래스나 인터페이스가 달라도 그 타입에 할당할 수 있습니다. 덕 타이핑과 같은 방향입니다.

### ✅ 꼬리 질문) 명목적 타이핑과 어떻게 다른가요?

명목적 타입 체계는 이름이 같거나 명시적으로 상속한 경우만 호환됩니다. 자바와 C#이 이쪽에 가깝습니다. TypeScript에서 이름만으로 구분하고 싶으면 브랜드 필드를 겹쳐 겹치는 모양이 서로 대입되지 않게 만듭니다.

```ts
declare const userIdBrand: unique symbol
type UserId = string & { readonly [userIdBrand]: never }
```

### ✅ 제네릭은 왜 쓰나요?

함수나 컴포넌트가 타입을 호출하는 쪽에서 정하게 하려고 씁니다. `identity<T>(value: T): T`는 넣은 타입 그대로를 돌려주고, `useState<User | null>(null)`처럼 상태의 모양을 고정합니다. 제네릭이 없으면 내부를 `any`로 풀거나 호출마다 함수를 다시 써야 합니다.

### ✅ 꼬리 질문) `extends` 제약은 어떤 의미인가요?

`T extends { id: string }`은 T가 그 모양을 만족해야 한다는 상한입니다. 제약 안의 프로퍼티만 안전하게 읽을 수 있습니다. 상속 문법과 철자는 같지만, 제네릭에서는 "이 타입은 최소한 이 구조"라는 뜻입니다.

### ✅ 자주 쓰는 유틸리티 타입을 설명해 보세요.

- `Partial<T>`: 모든 필드를 선택적으로 만듭니다. 수정 요청 바디에 자주 씁니다.
- `Required<T>`: 선택 필드를 필수로 바꿉니다.
- `Pick<T, K>`: 지정한 키만 남깁니다.
- `Omit<T, K>`: 지정한 키를 뺍니다.
- `Record<K, V>`: 키 집합으로 값 맵을 만듭니다.
- `Readonly<T>`: 필드를 읽기 전용으로 만듭니다.

### ✅ 꼬리 질문) `ReturnType`과 `Parameters`는 언제 쓰나요?

이미 있는 함수의 타입을 복제할 때 씁니다. `ReturnType<typeof fetchUser>`는 반환 타입, `Parameters<typeof fetchUser>`는 인자 튜플입니다. 구현이 바뀌면 파생 타입도 같이 따라갑니다.

### ✅ 타입을 좁힌다는 것은 무엇인가요?

유니온이나 `unknown`을 더 구체적인 타입으로 줄이는 일입니다. `typeof`, `===`, `in`, `instanceof`, `Array.isArray`가 대표적입니다. 조건이 참인 분기 안에서 TypeScript는 그 타입만 남깁니다.

### ✅ 꼬리 질문) 사용자 정의 타입 가드는 어떻게 만드나요?

`value is User`를 반환 타입으로 적으면, 참을 반환한 분기의 `value`가 `User`로 좁혀집니다.

```ts
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    typeof value.id === 'string'
  )
}
```

가드 본문이 틀리면 컴파일러는 잡아 주지 않습니다. 런타임 검사와 반환 타입이 어긋나지 않게 작성합니다.

### ✅ `enum`과 문자열 유니온 중 무엇을 쓰나요?

프론트엔드에서는 `'asc' | 'desc'` 같은 문자열 유니온을 더 자주 씁니다. 지워진 뒤에도 그냥 문자열이라 번들에 객체가 늘지 않고, JSON과도 맞습니다. `enum`은 런타임 객체를 만들고, 숫자 enum은 역방향 매핑까지 생깁니다. 상수 묶음이 정말 필요할 때만 enum을 둡니다.

### ✅ 꼬리 질문) `as const`는 어떤 효과를 주나요?

리터럴을 가장 좁은 읽기 전용 타입으로 고정합니다. `['asc', 'desc'] as const`의 원소 타입은 `string`이 아니라 `'asc' | 'desc'`입니다. 라우트 목록이나 쿼리 키처럼 값이 곧 타입인 곳에 씁니다.

### ✅ `strict`를 켜면 무엇이 달라지나요?

`strict`는 여러 검사를 한 번에 켭니다. 면접에서 꼭 말할 항목은 `strictNullChecks`입니다. 켜면 `null`과 `undefined`를 값 타입에 자동으로 넣지 않아, 쓰기 전에 좁혀야 합니다. 같이 켜지는 검사에는 `strictFunctionTypes`, `noImplicitAny`, `strictBindCallApply` 등이 있습니다. Next.js가 만드는 `tsconfig`는 `strict: true`가 기본입니다.

### ✅ 꼬리 질문) `strictNullChecks`가 꺼져 있으면 어떤 버그가 새나요?

`User | null`을 `User`에 그냥 넣을 수 있습니다. 컴파일은 통과하고, 런타임에 `user.name`에서 터집니다. API가 빈 값을 돌려주는 코드에서 특히 위험합니다.

### ✅ 타입 단언 `as`와 `satisfies`의 차이는 무엇인가요?

`as`는 컴파일러에게 타입을 그대로 믿으라고 덮어씁니다. 틀린 단언도 통과할 수 있습니다. `satisfies`는 값이 어떤 계약을 만족하는지만 확인하고, 변수의 추론된 구체 타입은 유지합니다. 테마 객체나 라우트 맵처럼 "이 모양은 지켜야 하고, 키 자동완성도 살리고 싶을 때" `satisfies`가 맞습니다.

```ts
const routes = {
  home: '/',
  post: '/posts/[id]',
} satisfies Record<string, string>

routes.home // '/'
```

### ✅ 꼬리 질문) 단언이 위험한 경우는 언제인가요?

DOM 쿼리의 `as HTMLInputElement`, JSON의 `as User`처럼 런타임 근거 없이 단언하면 검사가 사라집니다. 없으면 `null`인 원소를 단언으로 지우는 코드는 프로덕션에서만 깨집니다. 외부 데이터는 가드나 스키마로 확인하고, 단언은 컴파일러가 맥락을 잃은 좁은 구간에만 둡니다.

### ✅ 프론트 `tsconfig`에서 무엇을 보나요?

- `jsx: "preserve"`: JSX 변환은 번들러와 React 런타임에 맡깁니다.
- `moduleResolution: "bundler"`: `exports` 필드를 번들러와 같게 해석합니다.
- `noEmit: true`: 타입 검사만 하고 파일은 Next나 번들러가 냅니다.
- `plugins: [{ "name": "next" }]`: 라우트와 Metadata 타입을 에디터에 연결합니다.
- `noUncheckedIndexedAccess`: 배열과 인덱싱 결과에 `undefined`를 포함합니다. Next 기본값은 아니고, 켠 프로젝트에서는 `arr[0]`을 바로 쓰면 안 됩니다.

### ✅ 꼬리 질문) `import type`은 왜 나누나요?

타입 전용 import는 런타임 코드에서 지워집니다. `verbatimModuleSyntax`를 켜면 값과 타입을 섞어 import할 때 오류가 나서, 번들에 타입만 있는 모듈이 남는 일을 막습니다. 사이클과 서버/클라이언트 경계를 나눌 때도 값 import인지부터 보면 됩니다.
