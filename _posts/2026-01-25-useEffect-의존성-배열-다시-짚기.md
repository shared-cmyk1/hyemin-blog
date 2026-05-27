---
title: "2026.01.25 - useEffect 의존성 배열 다시 짚기"
description: "useEffect 의존성 배열을 빈 배열로 두면 안 되는 이유. PR 리뷰에서 반복된 패턴으로 정리한 의존성 배열의 진짜 의미."
date: 2026-01-25 20:30:00 +0900
categories: [개발]
tags: [react, useEffect, javascript, 코드리뷰]
---

이번 주에 후배 PR을 리뷰하다가 같은 패턴으로 세 번 코멘트를 달았다. 그래서 한 번 정리해 두기로 했다.

## 문제의 코드

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then(setUser);
  }, []); // ← 빈 배열

  return <div>{user?.name}</div>;
}
```

리뷰 코멘트는 한 줄이었다.

> 의존성 배열에 `userId` 넣어주세요.

후배가 답했다.

> 컴포넌트 처음 마운트될 때만 호출되면 되는 거 아닌가요?

여기서부터 길어졌다.

## 의존성 배열의 진짜 의미

많은 사람들이 의존성 배열을 *"언제 다시 실행할지"* 정하는 옵션으로 이해하는데, 그건 절반만 맞는 설명이다. 정확히는 이렇다.

> 이 effect 안에서 **참조하는 모든 외부 값**을 적어야 한다. React가 그 값이 바뀌었는지 비교해서 실행 여부를 정한다.
{: .prompt-info }

위 코드에서 `userId`를 참조하면서 배열에 안 넣으면, 부모가 `userId`를 바꿔도 effect가 다시 안 실행된다. 그러면 화면에는 *이전 유저의 데이터*가 그대로 남는다. 이게 흔히 말하는 "stale closure" 문제다.

## 고친 버전

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then((data) => {
        if (!cancelled) setUser(data);
      });
    return () => {
      cancelled = true;
    };
  }, [userId]);

  return <div>{user?.name}</div>;
}
```

두 가지를 바꿨다:

1. **`userId`를 의존성 배열에 추가** — 유저가 바뀌면 다시 fetch
2. **cancelled 플래그** — 이전 요청이 늦게 도착했을 때 이전 유저 데이터로 덮어쓰지 않게

2번을 빼먹으면 race condition이 생긴다. `userId=1`로 요청 보낸 직후 `userId=2`로 바뀌었는데, 1번 응답이 2번보다 늦게 도착하면 화면에 1번 유저가 찍힌다. 흔하다.

## 더 나은 길

사실 요즘은 직접 `useEffect`로 fetch 안 쓴다. React Query(TanStack Query)나 SWR을 쓴다.

```jsx
function UserProfile({ userId }) {
  const { data: user } = useQuery({
    queryKey: ["user", userId],
    queryFn: () => fetch(`/api/users/${userId}`).then((r) => r.json()),
  });

  return <div>{user?.name}</div>;
}
```

이게 훨씬 낫다. `queryKey`가 의존성 배열 역할을 하고, 캐싱·재시도·race condition 처리까지 다 알아서 해준다. 의존성 배열 실수할 일도 줄어든다.

| 방식 | 코드 양 | race 처리 | 캐싱 |
| --- | --- | --- | --- |
| 생 useEffect | 짧지만 함정 많음 | 직접 | ❌ |
| useEffect + cancel | 길어짐 | 직접 | ❌ |
| React Query | 가장 짧음 | 자동 | ✅ |

> 새 프로젝트면 무조건 React Query. 레거시면 cancel flag라도 챙기자.
{: .prompt-tip }

## 마무리

후배한테 이 글 링크 보낼 거다. 그게 블로그 쓰는 이유 중 하나.
