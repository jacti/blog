---
title: "ch08. 제어문 — 코드 작성의 가치는 어떻게 바뀌었는가"
date: 2026-03-25
tags: [javascript, deep-dive, ch08, control-flow]
weight: 2
---

프로그램은 위에서 아래로 흐릅니다. 제어문은 이 흐름을 **꺾는** 도구예요. `if`로 갈래를 나누고, `for`로 되돌리고, `break`로 끊어요.

그런데 "흐름을 꺾는 방법"을 고르는 기준은 시대에 따라 달랐어요. 예전에는 **CPU 사이클을 줄이는 것**이 좋은 코드의 조건이었고, 지금은 **읽는 사람이 빠르게 이해하는 것**이 좋은 코드의 조건이에요. 제어문의 설계를 따라가다 보면 이 가치 변화가 그대로 보여요.

---

## "이게 되네?" — 제어문에 숨은 의외의 것들

제어문을 공부하다 보면 이상한 것들이 눈에 띄어요.

**switch에서 break를 빼면?** — 다음 case로 그냥 떨어져요.

```js
switch ('A') {
  case 'A': console.log('우수');
  case 'B': console.log('양호');
  case 'C': console.log('보통');
}
// 출력: 우수, 양호, 보통 — 전부 출력된다!
```

**break에 인자를 넣을 수 있다고?** — 레이블을 붙이면 바깥 루프까지 탈출해요.

```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i + j === 3) break outer;  // 2중 루프를 한 번에 탈출
  }
}
```

**while이랑 do...while이 왜 따로 있어?** — 결과는 같은데 존재 이유가 다르다고?

이런 질문들을 파고들면, 전부 **같은 이유**로 연결돼요 — "좋은 코드"의 기준이 지금과 달랐던 시절의 유산이에요.

---

## 1. 성능이 우선이던 시절 — 왜 이런 문법이 생겼는가

### do...while의 탄생 이유: 명령어 하나의 무게

1970년대, CPU 사이클 하나가 지금의 서버비 수십 달러에 해당하던 시절이 있었어요. 이 시대에 `while`과 `do...while`은 성능 차이가 있었어요.

아래 인터랙티브 데모에서 직접 확인해보세요. **"실행" 버튼**을 누르면 각 반복문이 어떤 순서로 명령어를 실행하는지 애니메이션으로 보여줘요.

<iframe id="loop-race-demo" src="/demos/ch08-loop-race/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

<script>window.addEventListener('message',function(e){if(e.data&&e.data.type==='demo-resize'){document.getElementById('loop-race-demo').style.height=e.data.height+16+'px'}});</script>

### 왜 while이 점프를 더 많이 할까?

어셈블리 수준에서 보면 명확해요:

```
// while — 조건이 위에 있으니 매번 올라가야 한다
CHECK:                    ← 여기로 점프해야 함
  CMP n, 0               // 조건 검사
  JLE END                 // 거짓이면 종료
  ... 본문 ...
  JMP CHECK               // 조건으로 점프 (이게 추가 비용!)
END:

// do...while — 조건이 아래에 있어서 점프가 줄어든다
LOOP:
  ... 본문 ...
  CMP n, 0               // 조건 검사
  JG LOOP                 // 참이면 본문으로 점프
                          // 거짓이면 그냥 다음 줄 (점프 불필요)
```

`while`은 매 반복마다 **"조건으로 올라가는 점프"**가 한 번 더 필요해요. `do...while`은 조건이 끝에 있어서, 거짓일 때 그냥 다음 줄로 떨어지면 돼요. **이 점프 하나가 1970년대에는 진짜 비용**이었어요.

현대 CPU와 JIT 컴파일러는 이 차이를 최적화해서 사라지게 만들어요. 하지만 **do...while이라는 문법이 존재하는 이유**를 이해하는 데는 이 역사가 핵심이에요.

### 성능을 위해 가독성을 포기한 다른 사례들

| 문법 | 성능상 이점 | 가독성 비용 |
|------|------------|------------|
| switch fall-through | 점프 테이블로 O(1) 분기 | break 빠뜨리면 버그, 97%가 실수 |
| 레이블 + break | 함수 호출 오버헤드 없이 중첩 루프 탈출 | goto처럼 시선이 위아래로 점프 |
| do...while | 점프 명령어 1개 절약 | while보다 의도 파악이 어려움 |

공통점이 있어요: **모두 "함수 호출이 비싸던 시대"의 산물**이에요. 함수로 추출하면 깔끔하지만 성능이 떨어지니까, 문법 수준에서 해결한 거예요.

---

## 2. "빠르게 구현"에서 "잘 읽히게 구현"으로

### goto가 사라진 이유 — 읽는 비용의 발견

초기 프로그래밍에서 `goto`는 가장 자연스러운 흐름 제어였어요. 원하는 곳으로 바로 점프하면 되니까 **구현은 빨랐어요.**

```
10: x = 1
20: IF x > 5 GOTO 50
30: x = x + 1
40: GOTO 20
50: PRINT x
60: GOTO 30          ← 여기서 어디로 가지?
```

문제는 코드를 **읽을 때** 드러나요. `GOTO 30`을 만나면 30번 줄로 시선이 올라가고, 거기서 다시 40번의 `GOTO 20`을 따라가고... 코드가 길어지면 시선이 스파게티처럼 엉키어요.

1968년, 다익스트라(Dijkstra)가 "Go To Statement Considered Harmful"이라는 논문을 썼어요. 핵심 주장은 간단했어요:

> **"코드를 읽을 때 시선이 위아래로 점프하는 횟수가 곧 이해 비용이다."**

이전까지는 "**빠르게 실행되는** 코드가 좋은 코드"였어요. 다익스트라는 "**빠르게 읽히는** 코드가 좋은 코드"라는 새로운 기준을 제시한 거예요. 이게 **구조적 프로그래밍**의 시작이에요 — `if`, `for`, `while` 같은 구조만으로 모든 프로그램을 표현하자는 움직임이요.

### 그래서 지금은 어떻게 쓸까?

그 이후 50년간, "잘 읽히는 코드"를 향한 패턴들이 계속 발전했어요. 1장에서 다뤘던 것들의 현대적 대안을 볼게요.

**fall-through switch → 객체 매핑**

```js
// fall-through를 활용한 switch — 여러 case를 묶을 수 있다
function getDaysInMonth(month) {
  switch (month) {
    case 2: return 28;
    case 4: case 6: case 9: case 11:  // fall-through로 묶기
      return 30;
    default: return 31;
  }
}

// 객체 매핑 — fall-through 자체가 필요 없어진다
const DAYS = { 2: 28, 4: 30, 6: 30, 9: 30, 11: 30 };
const getDaysInMonth = (month) => DAYS[month] ?? 31;
```

fall-through는 "여러 case를 하나로 묶는" 용도로 쓰이지만, 객체 매핑을 쓰면 그 필요 자체가 사라져요.

**레이블 break → 함수 추출 + return**

```js
// 레이블 break — "outer가 어디지?" 하고 시선이 위로 올라간다
let found = null;
outer: for (const row of matrix) {
  for (const val of row) {
    if (val === target) {
      found = val;
      break outer;
    }
  }
}

// 함수 추출 — return이 곧 탈출이다
function findInMatrix(matrix, target) {
  for (const row of matrix)
    for (const val of row)
      if (val === target) return val;
  return null;
}
```

`break outer`를 만나면 "outer가 어디지?" 하고 위로 시선이 올라가요. `return`은 "이 함수가 끝난다"는 보편적 의미라 시선 이동이 없어요. 다익스트라가 지적한 바로 그 문제의 해결이에요.

**for + 인덱스 → 고차 함수**

```js
// for — "어떻게" 하는지를 읽어야 "무엇"을 하는지 알 수 있다
const results = [];
for (let i = 0; i < users.length; i++) {
  if (users[i].active) {
    results.push(users[i].name);
  }
}

// 고차 함수 — "무엇"을 하는지가 이름에 드러난다
const results = users
  .filter(user => user.active)
  .map(user => user.name);
```

`filter`, `map` 같은 고차 함수는 **의도를 이름으로 표현**해요. for문은 "어떻게 반복하는지"를 읽어야 "무엇을 하는지" 알 수 있지만, 고차 함수는 이름만 보면 돼요.

---

## 3. break와 continue

### break — 반복문/switch를 탈출

```js
for (let i = 0; i < 10; i++) {
  if (i === 5) break;  // i가 5이면 루프 종료
  console.log(i);
}
// 0, 1, 2, 3, 4
```

### continue — 현재 반복을 건너뛰기

```js
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) continue;  // 짝수면 건너뛴다
  console.log(i);
}
// 1, 3, 5, 7, 9
```

### 레이블 — 바깥 루프까지 제어

```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i + j === 3) break outer;
    console.log(i, j);
  }
}
// 0 0 / 0 1 / 0 2 / 1 0 / 1 1 — i=1,j=2에서 break outer
```

레이블 break는 **자신을 감싸는 바깥 레이블만** 지정할 수 있어요. 형제나 바깥으로 점프하는 건 불가능해요 — goto와의 결정적 차이예요.

하지만 앞서 봤듯이, 현대 코드에서는 **함수 추출 + return**으로 대체하는 게 관례예요. ESLint의 `no-labels` 규칙이 기본적으로 경고하는 것도 이 때문이에요.

---

## 정리

| 주제 | 핵심 |
|------|------|
| if...else | Truthy/Falsy 변환이 핵심. `[]`과 `{}`는 truthy |
| switch | `===` 비교, fall-through가 기본 (점프 테이블 유산) |
| for | 초기화→조건→본문→증감 순서 |
| while vs do...while | 조건 위치의 차이. do...while은 점프 절약 목적으로 탄생 |
| break / continue | 레이블로 바깥 루프 제어 가능, 하지만 함수 추출이 현대적 |

### 50년간의 교훈

> **"어떻게 빠르게 실행할까"** — 점프 하나를 줄이기 위해 do...while을, 함수 호출을 피하기 위해 레이블 break를 만들었다.
>
> **"어떻게 빠르게 읽힐까"** — goto를 없애고, 구조적 프로그래밍을 만들고, 고차 함수로 의도를 이름에 담기 시작했다.

좋은 코드의 기준이 바뀌었어요. 제어문도 이 기준으로 고르면 돼요:

- fall-through로 case를 묶고 있는가? → **객체 매핑**을 검토하라
- 레이블 break가 시선을 끌어올리는가? → **함수로 추출**하라
- 루프 안에서 if로 필터링하고 있는가? → **고차 함수**를 검토하라
