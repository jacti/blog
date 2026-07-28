---
title: "ch23. 실행 컨텍스트 — 자바스크립트는 코드를 두 번 읽는다"
date: 2026-07-28
tags: [javascript, deep-dive, ch23, execution-context, lexical-environment, hoisting, tdz, closure]
weight: 12
---

> 호이스팅, TDZ, 스코프 체인, 클로저 — 전부 다른 현상처럼 보이지만, 사실 단 하나의 설계에서 나옵니다. **엔진은 코드를 실행하기 전에 한 번 먼저 읽는다**는 것.

13장에서 스코프를 "런타임에 실존하는 자료구조"로 봤다면, 23장은 그 자료구조가 **언제, 어떤 순서로 만들어지고 부서지는지**를 다루는 장이에요. 책을 읽으면서 저는 표와 화살표 그림을 계속 손으로 다시 그리게 됐는데요 — 그래서 이번 글은 아예 그 그림을 **스텝별로 움직이는 시뮬레이터**로 만들었습니다. pythontutor를 참고했지만 결정적인 차이가 하나 있어요. pythontutor는 "실행"만 보여주는데, 이 시뮬레이터는 **평가 단계와 실행 단계를 명확히 구분**해서 보여줍니다. 이 구분이 이 챕터의 전부라고 해도 과언이 아니거든요.

---

## 핵심 개념: 용어 지도부터

이 챕터는 용어가 많아서 길을 잃기 쉬워요. 딱 5개만 잡고 시작합니다.

| 용어 | 정체 | 비유 |
|---|---|---|
| **실행 컨텍스트** (Execution Context) | 코드 실행에 필요한 정보 묶음. 스택으로 관리된다 | 작업 중인 책상 |
| **렉시컬 환경** (Lexical Environment) | 식별자와 값을 묶어두는 자료구조 + 바깥 환경으로 가는 참조 | 책상 위의 서랍장 |
| **환경 레코드** (Environment Record) | 렉시컬 환경 안에서 실제로 바인딩을 저장하는 곳 | 서랍 칸 |
| **객체 환경 레코드 / 선언적 환경 레코드** | 전역에서 `var`·함수 선언문이 가는 곳 / `let`·`const`가 가는 곳 | 겉면이 유리인 서랍(globalThis로 보임) / 불투명한 서랍 |
| **[[Environment]]** | 함수 객체가 "자기가 태어난 환경"을 기억하는 내부 슬롯 | 함수의 출생신고서 |

그리고 대원칙 하나:

> **소스코드는 "평가"와 "실행" 두 단계로 처리된다.**
> 평가 단계: 선언문만 먼저 훑어서 환경 레코드에 **등록**한다.
> 실행 단계: 코드를 위에서 아래로 실행하며 값을 **할당하고 검색**한다.

{{< callout type="info" >}}
호이스팅이라는 단어를 들으면 "선언이 위로 끌어올려진다"는 이미지가 떠오르지만, 코드는 아무 데도 안 움직여요. 그냥 **등록이 실행보다 먼저 일어날 뿐**입니다. 이 비유를 버리는 것이 TDZ를 이해하는 첫걸음이에요.
{{< /callout >}}

---

## 왜 중요한가

- `undefined`가 나올 자리와 `ReferenceError`가 날 자리를 **추측이 아니라 계산**으로 알 수 있게 됩니다.
- "클로저가 뭐예요?"에 "함수가 자신의 렉시컬 환경을 기억하는 것"이라고 답하는 대신, **어느 시점에 어떤 참조가 생겨서 환경이 살아남는지** 그림으로 설명할 수 있게 됩니다.
- 에러 메시지 3종(`SyntaxError` / `ReferenceError` / `TypeError`)이 **각각 다른 단계에서** 발생한다는 걸 알면 디버깅 속도가 달라져요.

---

## 시뮬레이터 사용법

아래 임베드된 시뮬레이터는 책 23장의 그림처럼 **표들의 배치가 고정**되어 있어요. 스텝을 넘겨도 표가 사라지거나 재배치되지 않고, 오직 세 가지만 변합니다.

1. **값** — 등록/할당에 따라 표 안의 값이 바뀐다
2. **화살표** — LexicalEnvironment 포인터와 outer 참조가 이동한다
3. **테두리 강조** — 엔진이 "지금 보고 있는 곳"이 스텝마다 옮겨 다닌다. 식별자 검색은 ✗(없음) → ✓(발견)으로 강조가 흘러간다

상단 배지가 지금이 🔍**평가 단계**인지 ▶️**실행 단계**인지 항상 알려줍니다. 아직 안 만들어진 박스는 흐릿하게(생성 전), pop된 것도 흐릿하게(소멸) 표시돼요.

---

## 파트 A. 호이스팅 — 평가 단계에 일어나는 일

### 1. var는 왜 선언 위에서 써도 될까

```js
console.log(x); // undefined (에러가 아니다!)
x = 1;
console.log(x); // 1
var x = 2;
console.log(x); // 2
```

첫 줄에서 에러가 안 나는 이유: 실행이 시작되기 전 **평가 단계에서 이미 `x`가 객체 환경 레코드에 `undefined`로 등록**되어 있기 때문이에요. 그리고 `var`라서 등록되는 곳이 **객체 환경 레코드**(BindingObject → globalThis)라는 것도 중요합니다 — 전역 `var` 변수가 `globalThis.x`로 보이는 이유가 바로 이거예요.

### 2. const도 호이스팅된다 — 다만 &lt;uninitialized&gt;로

```js
try {
  console.log(x);
} catch (e) {
  console.log(`${e.name}: ${e.message}`);
  // ReferenceError: Cannot access 'x' before initialization
}
const x = 1;
```

에러 메시지를 잘 보세요. `x is not defined`가 아니라 **`Cannot access 'x' before initialization`**입니다. 엔진이 `x`의 존재를 이미 알고 있다는 뜻 — 즉 **호이스팅은 됐어요**. 차이는 두 가지뿐입니다. `const`는 ① **선언적 환경 레코드**에 등록되고 ② `undefined` 대신 `<uninitialized>` 상태로 남는다는 것. 등록부터 초기화 줄까지의 구간이 TDZ(일시적 사각지대)예요.

### 3 & 4. 함수 선언문 vs 함수 표현식

```js
sayHi(); // hi — 성공!
function sayHi() { console.log('hi'); }
```

```js
greet(); // TypeError: greet is not a function
var greet = function () { console.log('hello'); };
```

함수 **선언문**은 평가 단계에 **함수 객체가 통째로** 만들어져 바인딩됩니다. 반면 함수 **표현식**에서 평가 단계에 등록되는 건 `var greet: undefined`뿐이고, `= function () {...}`는 할당식이라 실행 단계의 일이에요. 그래서 `greet()`는 `undefined()`를 호출하는 셈이 되어 `TypeError`가 납니다. `ReferenceError`가 아닌 것에 주목하세요 — 식별자는 존재해요. 값이 함수가 아닐 뿐.

시나리오 3에서는 함수 호출 시 **실행 컨텍스트가 스택에 push/pop 되는 것**도 처음 등장합니다. 직접 넘겨보세요.

<iframe id="sim-a" src="/demos/ch23-execution-context/?scenario=var-hoisting" style="width:100%;border:none;border-radius:12px;min-height:900px;margin:1rem 0;" loading="lazy"></iframe>

---

## 파트 B. 식별자 검색과 블록

### 5. 검색은 현재 환경에서 시작해 outer를 따라간다

```js
const a = 'outer';
{
  const b = 'block';
  console.log(b); // block — 블록 환경에서 즉시 발견
  console.log(a); // outer — 블록에 없음 → outer 따라 전역에서 발견
  console.log(c); // ReferenceError: c is not defined — 체인 끝(null)까지 실패
}
```

시뮬레이터에서 이 시나리오를 보면 강조 테두리가 **블록 ✗ → 전역 ✓** 순서로 흘러가요. 그 경로가 곧 스코프 체인입니다. 그리고 `c`의 에러 메시지는 TDZ 때와 달리 `c is not defined` — 이번엔 **정말 어디에도 없기** 때문이에요.

### 6. 재선언 규칙 — 에러도 단계가 다르다

```js
// (A) SyntaxError: Identifier 'x' has already been declared
console.log('시작'); // 이 줄은 절대 출력되지 않는다!
let x = 1;
{
  var x = 2; // var는 블록을 뚫고 올라가 전역 소속 → let x와 충돌
}
```

```js
// (B) 성공
var x = 1;
{
  let x = 2; // 블록 전용 환경 레코드에 따로 등록 → 충돌 없음
  console.log(x); // 2
}
console.log(x); // 1
```

{{< callout type="danger" >}}
(A)를 node로 실행해보면 **첫 줄 `console.log('시작')`조차 출력되지 않습니다.** `SyntaxError`는 평가(파싱) 단계의 early error라서 실행이 시작조차 안 되기 때문이에요. 시나리오 2의 `ReferenceError`는 실행 중에 그 줄에 도달해야 나는 에러였죠. **에러가 나는 단계 자체가 다릅니다.**
{{< /callout >}}

### 7. 블록은 컨텍스트가 아니라 화살표를 바꾼다

```js
let x = 1;
{
  let x = 2;
  {
    let x = 3;
    console.log(x); // 3
  }
  console.log(x); // 2
}
console.log(x); // 1
```

블록에 들어갈 때마다 새 실행 컨텍스트가 생길 것 같지만, **스택은 끝까지 전역 컨텍스트 하나**예요. 대신 실행 컨텍스트의 **LexicalEnvironment 포인터(화살표)가 새 블록 환경으로 이동**하고, 블록을 나오면 복원됩니다. 시뮬레이터에서 화살표만 눈으로 따라가 보세요 — 이 시나리오의 핵심은 그 화살표 하나입니다.

<iframe id="sim-b" src="/demos/ch23-execution-context/?scenario=lookup" style="width:100%;border:none;border-radius:12px;min-height:900px;margin:1rem 0;" loading="lazy"></iframe>

---

## 파트 C. 함수와 [[Environment]]

### 8. 함수는 태어난 곳을 기억한다

```js
const x = 'global';

function outer() {
  const y = 'outer';
  function inner() {
    console.log(y); // outer
    console.log(x); // global
  }
  inner();
}

outer();
```

`inner`가 `y`와 `x`를 읽을 수 있는 메커니즘을 시점 순으로 쪼개면 이렇습니다.

1. **`inner` 함수 객체가 생성되는 순간** (outer 몸통의 평가 단계) — `inner.[[Environment]] ← 현재(outer의) 렉시컬 환경`이 저장된다. **어디서 호출될지와 무관하게 지금 결정된다.** 이것이 렉시컬 스코프의 실체.
2. **`inner()`가 호출되는 순간** — inner의 렉시컬 환경이 새로 만들어지면서, outer 참조 자리에 `inner.[[Environment]]`에 저장해둔 값이 꽂힌다. 체인이 inner → outer → 전역으로 **실시간 조립**된다.

### 9. 예고편: pop 됐는데 살아남는 환경

```js
function counter() {
  let n = 0;
  return function () {
    n += 1;
    return n;
  };
}

const inc = counter(); // counter 실행 컨텍스트는 여기서 pop
console.log(inc()); // 1
console.log(inc()); // 2 — n이 살아있다!
```

`counter`의 실행 컨텍스트는 스택에서 사라졌는데 `n`은 어떻게 살아있을까요? **반환된 익명 함수의 `[[Environment]]`가 counter의 렉시컬 환경을 참조**하고 있어서, 그 환경이 GC 대상이 되지 않기 때문입니다. 시뮬레이터에서 이 장면을 보면 counter 실행 컨텍스트 박스는 흐려지는데(소멸) counter **환경** 박스는 💚 생존 표시와 함께 멀쩡히 남아있어요. 그리고 `inc()`를 다시 호출하면 **아까 그 환경이** 체인에 다시 연결되어 `n`이 이어서 증가합니다. — 이것이 클로저입니다. 자세한 건 24장에서.

<iframe id="sim-c" src="/demos/ch23-execution-context/?scenario=closure" style="width:100%;border:none;border-radius:12px;min-height:1000px;margin:1rem 0;" loading="lazy"></iframe>

<script>
window.addEventListener('message', function (e) {
  if (!e.data || e.data.type !== 'demo-resize') return;
  ['sim-a', 'sim-b', 'sim-c'].forEach(function (id) {
    var f = document.getElementById(id);
    if (f && f.contentWindow === e.source) f.style.height = (e.data.height + 16) + 'px';
  });
});
</script>

---

## 흔한 오해 / 주의점

- **"호이스팅 = 선언이 위로 끌어올려진다"** — 코드는 이동하지 않아요. 평가 단계의 *등록*이 실행보다 먼저일 뿐. "끌어올려진다"는 비유가 오히려 TDZ를 설명하지 못하게 만듭니다.
- **"let/const는 호이스팅되지 않는다"** — 됩니다. `<uninitialized>` 상태로 등록되기 때문에 접근하면 `undefined` 대신 에러가 날 뿐이에요. 에러 메시지(`Cannot access ... before initialization`)가 그 증거.
- **"블록마다 실행 컨텍스트가 생긴다"** — 아니에요. 블록은 **렉시컬 환경만** 갈아끼웁니다. 실행 컨텍스트가 생기는 건 전역 코드, 함수 호출, eval, 모듈.
- **에러 3종은 단계가 다르다** — `SyntaxError`(평가/파싱, 실행 시작 전) / `ReferenceError`(실행 중 검색 실패 또는 TDZ) / `TypeError`(실행 중 잘못된 값 사용). 어느 단계의 에러인지 구분하는 것만으로 원인 범위가 절반으로 줄어요.

{{< callout type="warning" >}}
**"전역 var는 항상 globalThis의 프로퍼티다"?** — 이 챕터의 "전역 코드"는 `<script>`의 이야기입니다. ES 모듈이나 CommonJS의 최상위는 진짜 전역 코드가 아니라서 `var`가 `globalThis`에 붙지 않아요. node의 `vm.runInThisContext`로 스크립트 문맥을 재현하면 확인할 수 있습니다 — `var`로 선언한 것만 `globalThis`의 프로퍼티가 되고, `let`은 식별자로는 접근되지만 `globalThis`에는 없어요.
{{< /callout >}}

## 직접 실험해보기

이 글의 모든 시나리오는 [학습 레포의 `examples/ch23/`](https://github.com/jacti)에 실행 가능한 파일로 정리했습니다.

```bash
node examples/ch23/001-var-hoisting.js        # undefined → 1 → 2
node examples/ch23/002-const-tdz.js           # ReferenceError 메시지 확인
node examples/ch23/006a-let-then-var.js       # 첫 줄도 출력 안 되는 SyntaxError
node examples/ch23/009-closure-preview.js     # 1, 2, 3
node examples/ch23/010-global-binding-check.js # var만 globalThis에 붙는다
```

특히 006a는 꼭 직접 돌려보세요. "코드가 한 줄도 실행되지 않는 에러"를 눈으로 보면 평가/실행 분리가 체감됩니다.

## 정리

- 자바스크립트는 코드를 **두 번** 읽는다. 평가 단계에 선언을 등록하고, 실행 단계에 할당·검색한다. 호이스팅과 TDZ는 이 분리의 자연스러운 결과다.
- 전역에서 `var`·함수 선언문은 **객체 환경 레코드**(globalThis에 비침), `let`·`const`는 **선언적 환경 레코드**에 등록된다.
- 식별자 검색은 현재 LexicalEnvironment에서 시작해 **outer 참조를 따라** 올라가고, 발견 즉시 끝난다.
- 블록은 실행 컨텍스트를 만들지 않는다. **LexicalEnvironment 포인터만 교체**된다.
- 함수 객체는 생성 시점에 `[[Environment]]`에 자신이 태어난 환경을 저장하고, 호출 시점에 그것을 outer로 삼아 체인을 조립한다. 이 참조가 살아있는 한 환경은 pop 후에도 소멸하지 않는다 — 클로저(24장)의 발판.
