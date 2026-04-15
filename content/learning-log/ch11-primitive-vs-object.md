---
title: "ch11. 원시 값과 객체의 비교 — 스펙이 그린 세계와 V8이 만든 현실"
date: 2026-04-15
tags: [javascript, deep-dive, ch11, v8, interning, immutability]
weight: 5
---

JavaScript에는 두 종류의 값이 있어요. **원시 값**(숫자, 문자열, 불리언, null, undefined, Symbol, BigInt)과 **객체**. 이 둘은 생성, 저장, 복사, 비교에서 전혀 다르게 동작합니다.

이 글은 두 파트로 나뉘어요:
1. **스펙이 정한 규칙** — JS가 "이렇게 동작해야 한다"고 약속한 것
2. **실제 구현도 그럴까?** — V8이 성능을 위해 어떻게 다르게 구현했는지

---

## Part 1. 스펙이 정한 규칙

### 원시 값은 불변(immutable)이다

원시 값은 한번 만들어지면 **그 값 자체를 수정할 수 없어요.** "변수에 다른 값을 넣는 것"과 "값 자체를 바꾸는 것"은 다릅니다.

```js
let str = "hello";
str[0] = "H";          // 값 자체를 바꾸려는 시도
console.log(str);       // "hello" — 바뀌지 않는다!

let arr = ["h","e","l","l","o"];
arr[0] = "H";           // 배열은 객체니까 가변
console.log(arr.join("")); // "Hello" — 바뀐다!
```

문자열의 개별 문자를 바꾸려고 해도 무시돼요 (strict mode에서는 TypeError). 반면 배열은 객체니까 자유롭게 수정됩니다.

{{< callout type="warning" >}}
"변수에 다른 값을 넣는 것"과 "값 자체를 바꾸는 것"을 혼동하지 마세요. `str = "world"`는 변수 재할당(가능), `str[0] = "H"`는 값 변이(불가).
{{< /callout >}}

### 변수 재할당 — "값이 바뀌는 게 아니라 바인딩이 바뀐다"

```js
let a = 5;   // 바인딩 a → 값 5
a = 10;      // 바인딩 a → 값 10 (5가 10으로 변한 게 아님!)
```

스펙에서 이 동작은 **SetMutableBinding**이라는 연산으로 정의돼요:

> "Change its bound value to V" — 바인딩의 값을 V로 변경하라

이 표현이 중요해요. "새 메모리를 할당하라"도 아니고, "기존 메모리를 덮어써라"도 아닙니다. **어떻게(HOW)**는 정하지 않고 **무엇이 관찰되어야 하는가(WHAT)**만 정의한 거예요.

### 값에 의한 전달 — 원시 값은 복사된다

```js
let a = 10;
let b = a;      // a의 값을 b에 복사
a = 99;

console.log(a); // 99
console.log(b); // 10 — a를 바꿔도 b는 그대로!
```

원시 값을 다른 변수에 할당하면 **값이 복사**돼요. 이후 두 변수는 완전히 독립적입니다. 한쪽을 바꿔도 다른 쪽에 영향이 없어요.

{{< callout type="info" >}}
Python에서는 이걸 **COW(Copy-on-Write)**와 비슷한 패턴으로 구현해요 — `b = a` 시점에 같은 정수 객체를 공유하다가, `a = 99`로 수정하면 그때 `a`만 새 객체를 가리키는 방식. JS 스펙은 이런 내부 메커니즘을 정하지 않고, "결과적으로 독립적이어야 한다"는 것만 보장합니다.
{{< /callout >}}

### 참조에 의한 전달 — 객체는 공유된다

```js
let obj1 = { name: "Kim" };
let obj2 = obj1;              // 참조 복사
obj2.name = "Lee";            // obj2를 통해 수정

console.log(obj1.name);       // "Lee" — obj1도 바뀌었다!
console.log(obj1 === obj2);   // true — 같은 객체
```

객체를 변수에 할당하면 **참조(메모리 주소)**가 복사돼요. 두 변수가 같은 객체를 가리키기 때문에 한쪽에서 수정하면 다른 쪽에도 반영됩니다.

### === 비교의 두 얼굴

이게 11장의 핵심이에요. `===`는 비교 대상이 원시 값이냐 객체냐에 따라 **완전히 다르게** 동작합니다.

```js
var person1 = { name: 'Lee' };
var person2 = { name: 'Lee' };

console.log(person1 === person2);           // false — 다른 객체
console.log(person1.name === person2.name); // true  — 같은 값
```

| 비교 대상 | `===` 가 보는 것 | 예시 |
|---|---|---|
| 원시 값 | **값** 자체 | `"Lee" === "Lee"` → `true` |
| 객체 | **참조**(주소) | `{} === {}` → `false` |

내용이 아무리 같아도, 서로 다른 객체면 `false`예요.

### 프로퍼티 키가 문자열/심볼만 되는 이유

```js
const obj = {};
obj[1] = "a";
obj["1"];        // "a" — 숫자 1이 문자열 "1"로 변환됨
```

객체 프로퍼티의 키는 문자열과 심볼만 허용돼요. 이유는:

1. **불변성** — 문자열/심볼은 불변이라 해시 키로 안전 (해시값이 바뀔 일 없음)
2. **동등성의 명확함** — 숫자를 직접 허용하면 `0`과 `-0`, `NaN`과 `NaN`, `1`과 `1.0` 같은 모호한 케이스가 생겨요

```js
// 가상의 세계: 숫자가 직접 키가 된다면
obj[0]   // vs obj[-0]   → 0 === -0 은 true인데?
obj[NaN] // vs obj[NaN]  → NaN === NaN 은 false인데?
```

문자열로 변환하면 이런 모호함이 전부 사라져요. `Map`은 이 문제를 **SameValueZero** 알고리즘으로 해결해서 숫자도 직접 키로 쓸 수 있습니다.

---

## Part 2. 실제 구현도 그럴까?

여기까지가 스펙이 정한 "관찰 가능한 동작"이에요. 그런데 의문이 들었어요:

> "변수를 재할당하면 정말 새 메모리를 할당하고 주소를 바꿔?"
> "매번 그러면 느리지 않아?"

V8의 실제 구현을 파보니, **스펙과 같은 결과를 내면서도 내부적으로는 꽤 다르게 동작**하고 있었어요.

### as-if rule — 관찰만 같으면 된다

ECMAScript 스펙에는 암묵적인 원칙이 있어요:

> **관찰 가능한 동작(observable behavior)이 스펙과 같으면, 내부 구현은 자유다.**

JS에는 변수의 메모리 주소를 관찰할 방법이 없어요. C의 `&a` 같은 연산자가 없습니다. 그래서 "새 메모리를 할당했느냐, 같은 메모리를 덮어썼느냐"를 코드로 구분할 수 없어요. V8은 이 자유를 최대한 활용합니다.

### V8의 숫자 저장: Smi vs HeapNumber

V8은 모든 값을 포인터 크기(64비트)의 **태그드 값**으로 표현해요. 최하위 비트로 종류를 구분합니다:

```
Smi (Small Integer):  [63비트 정수값][0]  ← 최하위 비트 0
HeapObject:           [63비트 힙 주소][1]  ← 최하위 비트 1
```

**Smi (작은 정수)**: 31비트 범위의 정수는 슬롯에 직접 인코딩돼요. 재할당하면 **같은 슬롯을 덮어씁니다.** 새 메모리 할당이 없어요. C의 `int x = 5; x = 10;`과 동일합니다.

**HeapNumber (큰 숫자, 소수점)**: 별도의 힙 객체로 할당돼요. 재할당하면 **새 HeapNumber를 생성하고 포인터를 변경**합니다. 교재의 "새 메모리 공간" 모델과 일치해요.

| 값 종류 | V8 동작 | 교재 모델과 일치? |
|---|---|---|
| Smi (`5`, `10`, `-3`) | 슬롯 덮어쓰기 | 불일치 (더 빠름) |
| HeapNumber (`1.5`, `2.7`) | 새 객체 할당 + 포인터 변경 | **일치** |
| 문자열 | 새 객체 할당 + 포인터 변경 | **일치** |

{{< callout type="info" >}}
V8은 2025년에 [MutableHeapNumber 최적화](https://v8.dev/blog/mutable-heap-number)를 도입해서 HeapNumber도 제자리 수정이 가능해졌어요. 즉 **최근까지 교재의 모델이 실제 동작**이었습니다.
{{< /callout >}}

### V8의 문자열 인터닝 — 불변성이 만든 최적화

문자열이 불변이기 때문에 V8은 **같은 내용의 문자열을 하나만 만들어서 공유**할 수 있어요. 이걸 **인터닝(interning)**이라고 합니다.

V8 내부에는 **String Table**이라는 전역 해시 테이블이 있어요:

```
문자열 리터럴 'Lee'를 만남
  → 해시 계산 (예: 0x3F2A)
  → String Table에서 해시로 탐색
  → 없으면: 새 문자열 생성 + 테이블에 등록
  → 있으면: 기존 문자열 재사용
```

그런데 **모든 문자열이 인터닝되는 건 아니에요.** V8의 `--allow-natives-syntax` 플래그로 직접 확인해봤습니다:

```js
// node --allow-natives-syntax examples/ch11/01-string-interning.js

const literal = "hello";
%IsInternalizedString(literal)          // true — 리터럴은 인터닝

const dynamic = "hel" + "lo";
%IsInternalizedString(dynamic)          // false — 동적 생성은 안 됨

literal === dynamic                      // true — 하지만 비교는 정확!
```

전체 실험 결과:

| 생성 방식 | 인터닝 여부 |
|---|---|
| 문자열 리터럴 `"hello"` | **됨** |
| 같은 리터럴 (다른 변수) | **됨** |
| 프로퍼티 키 | **됨** |
| `"hel" + "lo"` (동적 연결) | 안 됨 |
| 템플릿 리터럴 `` `hello ${name}` `` | 안 됨 |
| `slice(0, 5)` | 안 됨 |
| `String(42)` | 안 됨 |
| `JSON.parse('"hello"')` | **됨** (의외!) |
| `"a".repeat(3)` | 안 됨 |

리터럴과 프로퍼티 키는 컴파일 타임에 확정되니까 인터닝 대상이에요. 동적으로 만든 문자열은 매번 인터닝하면 String Table이 비대해지니까 엔진이 선택적으로 처리합니다.

{{< callout type="danger" >}}
**인터닝 안 되면 `===` 비교가 위험하다?** 전혀 아닙니다. `===`는 인터닝 여부와 무관하게 **항상 정확해요.** 인터닝된 문자열끼리는 포인터 비교로 더 빠를 뿐이지, 인터닝 안 된 문자열도 내용 비교를 통해 정확한 결과를 반환합니다. 인터닝은 **성능** 최적화이지, **정확성**에는 영향을 주지 않아요.
{{< /callout >}}

일반 JS 코드로는 인터닝 여부를 확인할 방법이 아예 없어요. `===`, `typeof`, `Object.is` 뭘 써도 리터럴과 동적 문자열을 구분할 수 없습니다. 이것이 **as-if rule**의 핵심 — 관찰할 수 없으면 다르게 구현해도 된다.

---

## 실제 실행 결과

위 내용을 `node`에서 직접 실행한 결과입니다.

<iframe id="ch11-lab" src="/demos/ch11-primitive-vs-object/" style="width:100%;border:none;border-radius:12px;min-height:520px;margin:1rem 0;" loading="lazy" title="Ch11 실행 결과" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

<script>window.addEventListener('message',function(e){if(e.data&&e.data.type==='demo-resize'){document.getElementById('ch11-lab').style.height=e.data.height+16+'px'}});</script>

---

## 정리

| 개념 | 스펙이 정한 규칙 | V8의 실제 구현 |
|---|---|---|
| 원시 값 불변성 | 값 자체를 수정할 수 없다 | 문자열: 실제로 차단 / Smi: 변이 연산 자체가 없음 |
| 변수 재할당 | "바인딩의 값을 변경" | Smi: 덮어쓰기 / HeapNumber, 문자열: 새 객체 |
| 문자열 인터닝 | 스펙에 없음 (구현 자유) | 리터럴, 키는 인터닝 / 동적 생성은 안 됨 |
| `===` 비교 | 원시값은 값 비교, 객체는 참조 비교 | 인터닝 여부와 무관하게 항상 정확 |

교재를 읽을 때 기억할 점: **스펙의 모델과 V8의 구현은 추상화 레벨이 달라요.** 교재는 스펙을 기반으로 설명하는데, 이건 "틀린 게 아니라 다른 레벨의 진실"이에요. 스펙은 모든 엔진을 포용하는 동작 정의이고, V8은 그 안에서 as-if rule을 활용해 성능을 극대화합니다.

---

### 참고 자료

- [Turbocharging V8 with mutable heap numbers](https://v8.dev/blog/mutable-heap-number) — HeapNumber 불변성과 MutableHeapNumber 최적화
- [The story of a V8 performance cliff in React](https://v8.dev/blog/react-cliff) — Smi와 HeapNumber 표현 차이
- [Pointer Compression in V8](https://v8.dev/blog/pointer-compression) — 태그드 포인터와 Smi 인코딩
