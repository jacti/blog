---
title: "ch12. 함수 (2) — 강제하지 않는 언어"
date: 2026-05-12
tags: [javascript, deep-dive, ch12, immutability, hoisting, callback, pure-function]
weight: 7
---

C로 처음 함수를 배웠을 때, 컴파일러가 잡아준 것들이 꽤 많았어요. 인자에 `const`를 붙이면 내부에서 못 바꿨고, 함수 시그니처와 다른 개수를 넘기면 컴파일이 안 됐고, 변수 선언 위치를 어기면 그 자리에서 에러가 났죠. **약속을 어기면 컴파일러가 막아준다** — 그게 그 언어의 안전망이었어요.

JS로 넘어오면서 가장 어색했던 게 이 안전망이 거의 다 풀려있다는 점이었어요. 매개변수에 `const`도 못 붙이고, 인자 개수가 안 맞아도 잘만 돌아가고, 함수 선언이 어디 있든 호이스팅으로 다 끌어올려지고. 처음엔 "어 이래도 돼?" 였는데, 어느 순간 깨달았어요 — **JS는 강제하지 않는 대신 자유를 줬구나.** 그리고 자유는 항상 책임과 짝이에요.

이 글은 함수가 외부와 맺는 네 가지 약속 — *인자를 안 건드린다, 정해진 자리에서만 보인다, 정해진 인자만 받는다, 외부를 건드리지 않는다* — 이 JS에서 어떻게 풀려있는지, 그리고 풀린 자리를 **어떤 패턴으로 메우면 실수 없이 쓸 수 있는지** 정리하는 글입니다.

---

## 약속 1 — 인자를 안 건드린다

함수에 객체를 넘기는 순간 함수는 그 객체를 손에 쥐어요. 그리고 그 손은 자유롭습니다. 안에서 마음대로 바꿔도 호출자는 알 수 없어요.

```js
function applyDiscount(cart) {
  cart.total *= 0.9;   // 호출자의 객체가 그냥 바뀜
}

const myCart = { total: 10000 };
applyDiscount(myCart);
console.log(myCart.total);  // 9000 — 원본이 변함
```

C++이라면 `void applyDiscount(const Cart& cart)`라고 적어서 "이 함수는 cart를 안 바꿔요"를 컴파일러에게 약속할 수 있어요. Rust는 `&Cart`와 `&mut Cart`로 권한 자체를 타입에 박아 넣고요. **JS에는 이런 문법이 없어요.** 매개변수 자리에서 readonly를 강제할 방법이 *언어 차원으로는* 존재하지 않습니다.

대신 우회로 세 가지가 있어요. 첫째는 `Object.freeze`예요. 객체 자체를 얼려서 mutate가 시도되면 strict 모드에선 `TypeError`를 던지게 해요. 다만 한 단계만 얼리는 얕은 동결이라, 중첩된 객체까지 막으려면 재귀로 `deepFreeze`를 직접 짜야 합니다.

```js
function deepFreeze(o) {
  Object.values(o).forEach(v => {
    if (v && typeof v === 'object' && !Object.isFrozen(v)) deepFreeze(v);
  });
  return Object.freeze(o);
}

const user = deepFreeze({ name: '기래', profile: { age: 30 } });
user.profile.age = 99;  // strict 모드면 throw
```

둘째는 `Proxy`로 readonly view를 씌우는 방법이에요. 원본은 mutable로 두되 함수에 넘길 때만 "쓰기 차단" 래퍼를 통과시키면 돼요. 셋째는 TypeScript의 `Readonly<T>` / `DeepReadonly<T>`예요. 이게 실무에서는 가장 흔한 답인데, 한 가지 분명히 알아둘 게 있어요.

{{< callout type="warning" >}}
**TypeScript의 `Readonly<T>`는 컴파일이 끝나면 사라져요.** 런타임에는 그냥 mutable 객체예요. 외부에서 들어오는 데이터(API 응답, 사용자 입력)는 타입을 거짓말할 수 있어서, 진짜 안전하게 가려면 *경계에서* `Object.freeze`로 한 번 더 받쳐야 해요.
{{< /callout >}}

그런데 JS 커뮤니티가 실제로 이 문제를 푸는 방식은 freeze도 Proxy도 아니에요. **"애초에 안 바꾸는 패턴"** 으로 풀어요. 객체를 mutate하지 않고, 바꿔야 하면 spread로 새 객체를 만들어 반환합니다. React, Redux, Zustand가 다 이 패턴이에요.

```js
function applyDiscount(cart) {
  return { ...cart, total: cart.total * 0.9 };   // 새 객체
}

const myCart = { total: 10000 };
const discounted = applyDiscount(myCart);
console.log(myCart.total);      // 10000 — 원본 보존
console.log(discounted.total);  // 9000
```

"강제할 수단이 없는 대신, 강제할 필요 없는 패턴을 쓴다." 이게 JS의 첫 번째 작법이에요.

---

## 약속 2 — 정해진 자리에서만 보인다

C에서는 함수 선언이 *어디서 보이는지* 가 위치로 정해져요. 파일 상단에 선언하면 그 아래 전부, 함수 안에 선언하면 그 함수 안만. 단순합니다.

JS의 함수 선언문은 **호이스팅** 때문에 조금 더 복잡해요. 함수 선언문은 자기가 속한 스코프의 최상단으로 끌어올려져요. 그래서 코드 흐름상 아래에 적힌 함수도 위에서 호출할 수 있어요. 여기까지는 익숙한 이야기.

문제는 **블록 안에 함수 선언문**을 쓸 때예요.

```js
if (condition) {
  function helper() { return 'A'; }
} else {
  function helper() { return 'B'; }
}
helper();
```

이 코드, 어떻게 동작할까요? 답이 **"환경과 모드에 따라 다르다"** 입니다. ES5 이전엔 V8과 Firefox가 서로 다른 답을 냈고, 지금도 strict 모드와 sloppy 모드의 동작이 갈려요.

**strict 모드(또는 ES Module)** 에서는 깨끗하게 정리됐어요. 블록 안 함수 선언문은 **블록 스코프** 에 갇혀요. `let`처럼 블록 밖에선 안 보입니다. 다만 `let`과 다른 점이 하나 있는데, 블록 안에서는 **최상단으로 호이스팅** 돼요. 선언문 위에서 호출해도 동작해요. TDZ(temporal dead zone)가 없거든요. 함수 선언문은 호이스팅되는 시점에 값까지 같이 들어가니까요.

{{< callout type="danger" >}}
**sloppy 모드의 함정**: 블록 안 함수 선언문이 **두 개의 바인딩**으로 생성돼요 — 블록 스코프 바인딩 하나, 바깥 함수/스크립트의 var 바인딩 하나. 같은 식별자가 코드 흐름의 어디서 보느냐에 따라 `undefined`였다가 함수였다가 합니다. Annex B.3.3이라는 이름의, 웹 호환성을 위해 살려둔 규칙이에요. 한마디로 *함정* 입니다.
{{< /callout >}}

세 가지 동작을 직접 돌려보면 감이 와요. 아래 실행기에서 코드를 수정해가며 결과를 확인해보세요.

<iframe id="demo-block-hoist" src="/demos/ch12-block-hoist/" style="width:100%;border:none;border-radius:12px;min-height:520px;margin:1rem 0;" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

<script>window.addEventListener('message',function(e){if(e.data&&e.data.type==='demo-resize'){var el=document.getElementById('demo-block-hoist');if(el)el.style.height=e.data.height+16+'px'}});</script>

### 그래서 실수 안 하려면

세 가지 패턴이면 충분해요.

**첫째, strict 모드를 강제하세요.** ES Module(`"type": "module"` 또는 `.mjs`)을 쓰면 자동으로 strict예요. 그러면 Annex B.3.3의 이중 바인딩 함정이 사라지고, 블록 함수 선언문은 깔끔한 블록 스코프가 됩니다.

**둘째, 조건부로 함수를 정의하고 싶으면 함수 선언문 대신 `const` + 함수 표현식**으로 가세요.

```js
const helper = condition
  ? function () { return 'A'; }
  : function () { return 'B'; };
```

이렇게 하면 호이스팅도, 모드별 차이도 다 사라져요. 변수만 호이스팅되고 값은 할당 시점에야 들어오니까, 의도한 분기의 함수만 살아남습니다.

**셋째, 함수 선언문은 자기 스코프 최상단에만 두세요.** if/for/while 같은 블록 안에 함수 선언문을 박는 건 — 동작은 하지만 — 가독성 측면에서 손해예요. 읽는 사람이 "이게 블록 밖에서도 보이나?" 를 매번 의심하게 됩니다.

강제할 수단이 없는 자리는 **습관으로 메워요**. JS의 두 번째 작법이에요.

---

## 약속 3 — 정해진 인자만 받는다

함수에 콜백을 넘기는 패턴은 JS의 핵심이에요. `map`, `forEach`, `addEventListener`, `setTimeout`, `fs.readFile` — 거의 모든 비동기·이벤트 API가 콜백 기반이죠. 그런데 콜백을 받는 함수(=caller)와 콜백 자체(=callee)는 시그니처에 대해 어떤 약속도 강제하지 않아요.

C의 함수 포인터를 떠올려봐요. `void (*cb)(int, char*)`라고 선언하면 정확히 그 시그니처에 맞는 함수만 받을 수 있어요. 인수 개수도, 타입도 맞아야 컴파일러를 통과합니다. JS는?

```js
function add(a, b) { return a + b; }
add(1, 2, 3, 4);   // 3   — 초과 인수 무시
add(1);            // NaN — 부족하면 undefined
add();             // NaN — 전부 undefined
```

**개수 불일치가 에러가 아니에요.** 이게 콜백 설계의 모든 자유를 만들어내는 동시에, 모든 함정을 만들어요.

자유부터 보면, JS의 콜백 계약은 **비대칭** 이에요. caller는 자기가 가진 모든 정보를 *최대치* 로 넘기고, callback은 자기가 필요한 것만 매개변수로 받아요. 남는 건 그냥 버려져요. 예를 들어 `Array.prototype.map`의 계약은 "`callback(value, index, array)` 형태로 항상 3개를 넘긴다" 인데, callback은 1개만 받든 3개를 다 받든 자유예요.

```js
[10, 20, 30].map(x => x * 2);              // value만 필요
[10, 20, 30].map((x, i) => `${i}:${x}`);   // index도 필요
[10, 20, 30].map((x, i, arr) => arr.length); // array까지 필요
```

caller(`map`) 입장에선 항상 3개를 넘기지만, callee 입장에선 자기 사정에 맞게 골라 받아요. 이 비대칭이 JS의 콜백 API를 그토록 유연하게 만들어요. 새 정보를 시그니처에 추가해도(예: ES5에서 `map`의 세 번째 인자가 추가됐어요) 기존 callback이 안 깨져요. 무시되니까요.

### 그래서 실수 안 하려면

자유의 대가는 시그니처의 **암묵성** 이에요. 코드만 보면 "이 함수에 어떤 모양의 콜백을 넘겨야 하는지" 가 안 보여요. 문서를 읽거나 구현을 까봐야 알 수 있죠. 이 빈자리를 메우는 패턴 세 가지.

**첫째, 인수가 3개를 넘어가면 객체 한 덩어리로 묶어 넘기세요.** 위치 인수는 2~3개까지가 인간이 기억할 수 있는 한계예요. 그 이상은 객체로 묶고, callback은 destructuring으로 필요한 것만 꺼내 쓰면 됩니다.

```js
// nope — 위치 인수 6개. 순서를 기억해야 함
onEvent('click', x, y, target, timestamp, modifiers);

// yes — 객체로 묶기. 순서 무관, 확장에 강함
onEvent({ type: 'click', x, y, target, timestamp, modifiers });

// callback은 필요한 것만 destructuring
onEvent(({ type, target }) => console.log(type, target));
onEvent(({ x, y }) => moveTo(x, y));
```

새 필드가 추가돼도 기존 callback이 안 깨지고, IDE 자동완성이 잡아주고, 의미가 이름으로 드러납니다. React props, GraphQL resolver, Express middleware 등이 다 이 패턴을 따르는 이유예요.

**둘째, TypeScript로 시그니처를 강제하세요.** JS가 런타임에 안 강제하니까 컴파일 타임에 강제하면 됩니다.

```ts
type Listener = (event: { type: string; x: number; y: number }) => void;

function onEvent(cb: Listener) {
  cb({ type: 'click', x: 0, y: 0 });
}

onEvent(({ type }) => console.log(type));    // ✅
onEvent((e, extra) => console.log(extra));   // ❌ extra가 타입에 없음
```

위치 인수든 객체 인수든, 타입이 있으면 IDE가 시그니처를 보여줘요. 콜백 기반 코드의 가독성이 한 단계 올라갑니다.

**셋째, 기본값 매개변수를 적극적으로 쓰세요.** callee 쪽에서 "안 들어오면 이렇게 동작한다"를 명시하면, caller가 인자 누락해도 안전합니다.

```js
function fetch(url, { method = 'GET', headers = {}, body = null } = {}) {
  // ...
}
```

caller가 옵션 객체를 통째로 빼먹어도, 빈 객체 디폴트가 받아주고 그 안에서 다시 키별 디폴트가 받아줘요. 두 겹의 안전망이에요.

자유로운 시그니처를 **명시적인 구조** 로 메우는 것 — JS의 세 번째 작법입니다.

---

## 약속 4 — 외부를 건드리지 않는다

마지막 약속이 가장 까다로워요. **"이 함수는 외부를 건드리지 않는다"** 는 약속, 즉 *순수함수* 의 약속이요.

순수함수의 정의는 두 가지 조건이에요. **참조 투명성** — 같은 입력에 항상 같은 출력. 호출을 결과값으로 그냥 치환해도 프로그램이 동일하게 동작해야 함. 그리고 **부수효과 없음** — 함수 반환값 외에 외부에서 관찰 가능한 어떤 변화도 일으키지 않음. 두 조건 모두 만족해야 순수합니다.

이 정의를 JS에 적용해보면 묘하게 모호한 자리들이 생겨요. JS는 mutable한 세계이고, 객체는 참조로 공유되고, 함수가 외부 어디까지 닿느냐의 경계가 흐릿하거든요. 그래서 "이 함수 순수해?" 라는 질문에 즉답이 어려운 경우가 많아요.

JS는 순수성을 *언어 차원에서 표시하지 않아요*. Haskell은 부수효과를 `IO` 타입으로 시그니처에 박아 넣어서 한눈에 구분되지만, JS는 함수 시그니처만 봐서는 그게 순수한지 비순수한지 알 길이 없어요. 결국 **사람이 판단해야** 합니다.

판단의 기준은 단순해요. 세 가지를 묻습니다.

1. **인자 외부의 가변 상태를 읽나?** (글로벌, 클로저 변수, `Date.now()`, `Math.random()`, DOM, fetch 응답…)
2. **인자나 외부 상태를 mutate하나?** (`obj.x = ...`, `arr.push`, global 할당)
3. **관찰 가능한 IO를 일으키나?** (console, 파일, 네트워크, throw)

셋 다 No면 순수. 하나라도 Yes면 비순수예요.

{{< callout type="info" >}}
**"순수에 가까운"** 같은 비공식 표현은 학술적 용어가 아니에요. 콘솔 출력이 한 줄이라도 있으면 엄밀히는 *비순수* 입니다. 다만 실무에서 그 비순수성이 코드 로직에 영향을 주느냐 안 주느냐는 별개 문제고, 그 구분은 사람이 맥락으로 봐야 해요.
{{< /callout >}}

### 🎯 인터랙티브 퀴즈

이론을 들었으니 직접 판단해볼 차례예요. 아래 두 함수를 보고 "순수한가" 를 답해보세요. 정답과 함께 이유까지 풀어드릴게요.

<iframe id="demo-purity-quiz" src="/demos/ch12-purity-quiz/" style="width:100%;border:none;border-radius:12px;min-height:560px;margin:1rem 0;" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

<script>window.addEventListener('message',function(e){if(e.data&&e.data.type==='demo-resize'){var el=document.getElementById('demo-purity-quiz');if(el)el.style.height=e.data.height+16+'px'}});</script>

---

## 마무리 — 강제 없는 세계의 작법

네 가지 약속을 돌아보면 패턴이 보여요. JS는 약속을 *언어 차원으로 강제하지 않습니다*. 매개변수 readonly도, 함수 선언 위치도, 콜백 시그니처도, 순수성도. 그래서 처음에는 위태롭게 느껴져요. C나 Rust라면 컴파일러가 잡아줬을 실수들이 다 런타임에 흩어져 있으니까요.

그런데 강제가 사라진 자리에는 그만큼의 자유가 들어와요. 객체 하나로 모든 옵션을 넘기는 콜백 패턴, 환경별로 함수를 갈아끼우는 유연성, 함수를 값처럼 자유롭게 옮기고 재할당하는 일급성. C에서는 한 줄로 못 쓰는 것들이 JS에선 자연스러워요. 그 자유가 JS를 브라우저 스크립트에서 웹 전반의 기반 언어로 키운 동력이에요.

그래서 강제하지 않는 언어를 쓸 때 필요한 건 **습관** 입니다. 강제할 수 없는 자리를 *문법 대신 작법으로* 메우는 거예요. 객체를 mutate하지 않는 spread 패턴, strict 모드와 `const` + 함수 표현식, 옵션 객체 + TypeScript 시그니처, 그리고 함수가 외부를 건드리는지 항상 자문하는 태도. 이게 모이면 컴파일러 없이도 안전한 함수를 쓸 수 있어요.

자유는 책임과 짝이라는 말, JS 함수에서 가장 또렷하게 와닿는 것 같아요.
