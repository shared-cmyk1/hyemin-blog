---
title: 마크다운 작성 예시
date: 2026-05-18 14:00:00 +0900
categories: [가이드]
tags: [markdown, chirpy, 예시]
pin: true
---

블로그에 글 쓸 때 자주 쓰는 마크다운 문법 모음.

## 1. 헤딩

```markdown
## 큰 제목
### 중간 제목
#### 작은 제목
```

## 2. 강조

**굵게**, *기울임*, ~~취소선~~, `인라인 코드`

## 3. 리스트

- 사과
- 바나나
  - 노란 바나나
  - 초록 바나나
- 포도

1. 첫째
2. 둘째
3. 셋째

- [x] 완료된 항목
- [ ] 안 끝낸 항목

## 4. 인용

> 인용문은 이렇게 씁니다.
> 두 줄로 이어 쓸 수도 있어요.

## 5. 코드 블록

```python
def hello(name):
    return f"안녕, {name}!"

print(hello("혜민"))
```

```javascript
const greet = (name) => `안녕, ${name}!`;
console.log(greet("혜민"));
```

## 6. 표

| 이름   | 역할     | MBTI |
| ------ | -------- | ---- |
| 혜민   | 디자이너 | INFP |
| 민정   | 개발자   | ENTP |

## 7. 링크 & 이미지

[GitHub](https://github.com) 으로 가는 링크.

이미지는 이렇게:

```markdown
![대체 텍스트](이미지경로.png)
_캡션은 이렇게_
```

## 8. Chirpy 전용 프롬프트 박스

> 팁! 이런 식으로 강조할 수 있어요.
{: .prompt-tip }

> 알아두면 좋은 정보입니다.
{: .prompt-info }

> 주의해서 보세요.
{: .prompt-warning }

> 위험! 절대 하지 마세요.
{: .prompt-danger }

## 9. 각주

본문에 각주[^1]를 달 수 있고, 두 번째 각주[^둘째]도 가능합니다.

[^1]: 첫 번째 각주 내용.
[^둘째]: 두 번째 각주 내용.

## 10. 구분선

---

## 마무리

이 정도면 일상 블로깅에 필요한 건 거의 다 됩니다. 더 궁금하면 [Chirpy 공식 데모](https://chirpy.cotes.page/posts/text-and-typography/)에서 더 많은 예시를 볼 수 있어요.
