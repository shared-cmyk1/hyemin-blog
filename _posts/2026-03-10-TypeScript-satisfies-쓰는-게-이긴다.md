---
title: "2026.03.10 - TypeScript satisfies, 쓰는 게 이긴다"
description: "Record 타입이 리터럴을 string으로 넓히는 문제와 satisfies로 해결하는 법. 코드 리뷰에서 반복된 TypeScript 패턴 정리."
date: 2026-03-10 22:00:00 +0900
categories: [개발]
tags: [typescript, satisfies, 코드리뷰]
---

오늘 코드 리뷰 하다가, 같은 패턴으로 또 만났다. 짧게 정리.

## 흔한 코드

```ts
type Theme = "light" | "dark" | "system";

const themeColors: Record<Theme, string> = {
  light: "#ffffff",
  dark: "#000000",
  system: "#888888",
};

// 사용
themeColors.light.toUpperCase();
```

별 문제 없어 보인다. 그런데 한 가지 미묘한 점이 있다.

`themeColors.light`의 타입은 `string`이다. **그냥 `string`**. 원본 값이 `"#ffffff"`라는 리터럴이라는 정보를 잃어버린다.

이게 왜 문제냐면, 이런 케이스에서 드러난다:

```ts
type HexColor = `#${string}`;

const sendColor = (c: HexColor) => { /* ... */ };

sendColor(themeColors.light);
// ❌ Error: Argument of type 'string' is not assignable to parameter of type '`#${string}`'
```

`themeColors.light`가 `string`으로 *넓어졌기* 때문에, 더 구체적인 타입을 요구하는 곳에 못 넣는다.

## satisfies로 해결

`Record<Theme, string>`을 *타입 어노테이션*으로 쓰면, 변수 타입이 그걸로 *고정*된다. 그래서 리터럴 정보가 사라진다.

대신 `satisfies`를 쓰면, **타입 체크는 하되 변수의 실제 타입은 좁게 유지**된다.

```ts
const themeColors = {
  light: "#ffffff",
  dark: "#000000",
  system: "#888888",
} satisfies Record<Theme, string>;

themeColors.light;
// ^ type: "#ffffff"  ← 리터럴 그대로!
```

이제 위의 `sendColor` 호출이 통과된다.

## 정리하면

| 방식 | 타입 검증 | 리터럴 유지 |
| --- | --- | --- |
| `as Record<Theme, string>` | ❌ (강제 캐스팅) | ❌ |
| `: Record<Theme, string>` (어노테이션) | ✅ | ❌ |
| `satisfies Record<Theme, string>` | ✅ | ✅ |

> `satisfies`는 "이 값이 이 타입을 만족하는지 검증해줘. 단, 변수 타입은 내가 적은 그대로 두고." 라는 뜻이다.
{: .prompt-tip }

## 언제 쓰면 좋나

- **설정 객체** (config, theme, constants) — 키도 좁히고 값도 리터럴 유지하고 싶을 때
- **enum 대신 const object 패턴** — `keyof typeof` 조합으로 더 강력해진다
- **데이터-주도 컴포넌트의 prop 매핑 테이블** — 자동완성이 진짜 좋아진다

반대로 **타입을 *일부러* 넓히고 싶을 때**는 그냥 어노테이션을 써야 한다. 예를 들어 외부 라이브러리 함수에 넘기기 위해 `string`으로 통일하고 싶을 때 등.

## 마무리

TypeScript 4.9에서 들어온 기능인데(지금이 5.x 시대니까 한참 됐다), 아직도 모르는 사람이 많다. 한 번만 써보면 진짜 안 돌아가게 된다. 다음 PR부터 시도해보길.

```ts
// Before
const config: Config = { ... };

// After
const config = { ... } satisfies Config;
```

이거 한 줄 바꾸는 거다. 어렵지 않다.
