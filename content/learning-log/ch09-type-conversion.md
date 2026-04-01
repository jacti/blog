---
title: "ch09. 타입 변환과 단축 평가 — JS는 왜 에러 대신 변환을 선택했나"
date: 2026-04-01
tags: [javascript, deep-dive, ch09, type-coercion, short-circuit]
weight: 3
---

`"5" - 1`이 `4`가 되고, `"5" + 1`이 `"51"`이 되는 언어. 다른 언어에서는 에러를 던질 법한 코드가 JavaScript에서는 "어떤 값"을 반환해요. 이번 장에서는 **왜 JS가 이런 선택을 했는지**, 그리고 그 규칙을 정확히 아는 것이 왜 중요한지를 정리했어요.

---

## 1. JS는 왜 에러 대신 변환을 선택했나

언어마다 타입이 맞지 않는 연산을 만났을 때의 전략이 완전히 달라요.

```js
// JavaScript — 변환해서라도 동작시킨다
"5" - 1     // 4
true + 1    // 2
```

```python
# Python — 즉시 에러
"5" - 1     # TypeError: unsupported operand type(s)
```

```rust
// Rust — 컴파일 자체가 안 된다
// let x: i32 = 1 + 1.0;  // error: expected `i32`, found `{float}`
```

타입 변환 전략의 스펙트럼은 이렇게 나뉘어요:

```
엄격 ◄──────────────────────────────► 관대
Rust   Go   Python   Java   PHP   JavaScript
```

1995년 Brendan Eich가 JS를 만들 때, 웹 폼에서 들어오는 값은 항상 문자열이었어요. `<input>` 태그의 `value`는 항상 `string`이거든요. 비개발자도 쉽게 쓸 수 있도록 `"5" - 1`이 그냥 동작하게 만든 거예요. **편의성 > 안전성**이라는 설계 철학이었죠.

이 선택 덕분에 JS는 진입장벽이 낮아졌지만, 동시에 타입 변환 규칙을 정확히 모르면 **예측 불가능한 버그**의 원인이 되기도 해요.

---

## 2. 암묵적 타입 변환 — 엔진이 몰래 하는 일

개발자가 의도하지 않았는데 JS 엔진이 자동으로 타입을 변환하는 것을 **암묵적 타입 변환(implicit coercion)** 이라고 해요. ECMAScript 스펙에 정의된 추상 연산(`ToString`, `ToNumber`, `ToBoolean`)이 내부에서 호출돼요.

### 문자열 변환 — `+` 연산자의 이중성

`+` 연산자는 **피연산자 중 하나라도 문자열이면** 나머지를 문자열로 변환해요:

```js
1 + '2'        // "12"   — 숫자가 문자열로
true + ''      // "true"
null + ''      // "null"
undefined + '' // "undefined"
```

`-`, `*`, `/`에서는 반대로 문자열이 숫자로 변환돼요. `+`만 문자열 연결을 겸하기 때문에 생기는 비대칭이에요:

```js
'5' - 1    // 4    — 문자열이 숫자로
'5' * 2    // 10
'5' / 1    // 5
'5' + 1    // "51" — 여기만 문자열 연결!
```

### 숫자 변환 — 비교와 산술 연산

비교 연산자와 산술 연산자(`-`, `*`, `/`)는 피연산자를 숫자로 변환해요:

```js
// ToNumber 변환 테이블
Number('')          // 0
Number('  ')        // 0    — 공백 문자열도 0
Number('10')        // 10
Number('10px')      // NaN  — 숫자가 아닌 문자 포함
Number(true)        // 1
Number(false)       // 0
Number(null)        // 0
Number(undefined)   // NaN  ← null과 다르다!
```

{{< callout type="info" >}}
**`null → 0`인데 `undefined → NaN`인 이유는?** `null`은 "의도적으로 비어있음"이라 0으로 다뤄지고, `undefined`는 "정의되지 않음"이라 숫자로 변환할 근거가 없기 때문이에요. 이것은 비트 처리의 우연이 아니라 의미론적 설계 의도예요.
{{< /callout >}}

### 불리언 변환 — 조건문의 핵심

조건문(`if`, `for`, `while`), 삼항 연산자, 논리 연산자는 조건식의 결과를 불리언으로 평가해야 해요. 이때 엔진이 호출하는 것이 `ToBoolean` 추상 연산이에요. 이 연산의 규칙은 단순해요 — **falsy 값 7개를 제외하면 전부 `true`**:

#### falsy 값 7가지

ToBoolean에서 `false`로 변환되는 값은 딱 7개뿐이다. **나머지는 전부 truthy**:

```js
Boolean(false)      // false
Boolean(0)          // false
Boolean(-0)         // false
Boolean(0n)         // false (BigInt의 0)
Boolean('')         // false
Boolean(null)       // false
Boolean(undefined)  // false
Boolean(NaN)        // false
```

{{< callout type="warning" >}}
**빈 객체 `{}`와 빈 배열 `[]`은 truthy예요.** 빈 문자열 `''`은 falsy인데 빈 배열 `[]`은 truthy라서 자주 혼동돼요.

```js
Boolean({})    // true
Boolean([])    // true
Boolean('')    // false
```
{{< /callout >}}

#### 다른 언어는 조건문에서 bool을 강제하나?

언어에 따라 조건문의 처리가 완전히 달라요:

| 언어 | `if (0)` | `if ("")` | 규칙 |
|------|----------|-----------|------|
| **Rust / Go** | 컴파일 에러 | 컴파일 에러 | bool 타입만 허용 |
| **Java** | 컴파일 에러 | 컴파일 에러 | bool 타입만 허용 |
| **Python** | false | false | `__bool__()` 프로토콜로 변환 |
| **Ruby** | **true** ⚠️ | **true** ⚠️ | nil과 false만 falsy |
| **JavaScript** | false | false | ToBoolean 추상 연산 |

Ruby에서 `0`이 truthy인 것은 "0도 유효한 값"이라는 철학이고, Rust/Go에서 에러를 내는 것은 "실수를 컴파일 타임에 잡겠다"는 철학이에요. JS의 truthy/falsy는 그 중간 — 편의를 주되, 규칙은 있어요.

---

## 3. 명시적 타입 변환 — 개발자가 직접 하는 일

의도를 코드에 명확히 드러내는 변환이에요. 협업 시에는 암묵적 변환보다 명시적 변환을 쓰는 것이 코드의 의도를 읽기 쉽게 만들어요.

### 문자열로 변환

```js
String(1)          // "1"
(1).toString()     // "1"
1 + ''             // "1" — 암묵적이지만 관례적으로 사용
```

### 숫자로 변환

```js
Number('10')       // 10
parseInt('10px')   // 10   — 앞에서부터 파싱, 숫자 아닌 문자에서 멈춤
parseFloat('3.14') // 3.14
+'10'              // 10   — 단항 + 연산자
'10' * 1           // 10
```

`Number()`는 값을 있는 그대로 숫자로 변환하려고 시도해요. 반면 `parseInt()`는 문자열에서 숫자를 **뽑아내는** 것에 가까워요:

```js
Number('10px')     // NaN  — 전체가 온전한 숫자가 아니므로 실패
parseInt('10px')   // 10   — 앞에서부터 숫자를 추출, 'px'에서 멈춤

Number('')         // 0    — 빈 문자열은 0으로 변환
parseInt('')       // NaN  — 추출할 숫자가 없음
```

### 불리언으로 변환

```js
Boolean('hello')   // true
!!'hello'          // true — !! 이중 부정 (관례적 방법)
```

---

## 4. `toString()`, 그리고 왜 `toBoolean()`은 없을까

### `toString()`으로도 변환할 수 있다

객체와 원시값 모두 `.toString()` 메서드를 호출해서 문자열로 변환할 수 있어요:

```js
(123).toString()       // "123"
true.toString()        // "true"
[1, 2, 3].toString()   // "1,2,3"
new Date().toString()  // "Tue Apr 01 2026 12:00:00 GMT+0900"
```

### 그럼 `toNumber()`나 `toBoolean()`도 있을까?

없어요. `toString()`은 있는데 `toNumber()`나 `toBoolean()`은 존재하지 않아요:

```js
const obj = {};
obj.toString()    // "[object Object]" ✅
obj.toNumber()    // ❌ TypeError: not a function
obj.toBoolean()   // ❌ TypeError: not a function
```

### 왜 `toString()`만 필요했을까?

`toString()`은 **객체의 메서드**예요. 메서드가 필요한 이유는 **객체마다 답이 달라야 하기 때문**이에요:

```js
[1, 2, 3].toString()    // "1,2,3"          — 배열만의 표현
new Date().toString()   // "Tue Apr 01 ..."  — Date만의 표현
/regex/.toString()      // "/regex/"         — 정규식만의 표현
```

"자기소개 해봐"라고 물으면 객체마다 대답이 달라요. 그래서 각 객체가 자기만의 `toString()`을 가져야 해요.

반면 불리언 변환은? **객체는 종류와 상태에 관계없이 항상 truthy**예요:

```js
Boolean({})                  // true
Boolean([])                  // true
Boolean(new Boolean(false))  // true — false를 감싼 객체조차!
```

"너 truthy야?"라고 물으면 모든 객체가 똑같이 "네"라고 대답해요. 물어볼 필요가 없으니 메서드도 필요 없는 거예요. 그래서 `ToBoolean`은 엔진 내부의 추상 연산으로만 존재하고, 개발자가 호출하는 메서드로는 만들지 않았어요.

{{< callout type="info" >}}
**메서드는 "객체마다 답이 다를 때"만 필요해요.** `toString()` → 객체마다 다른 답 → 메서드 필요. `toBoolean()` → 항상 같은 답(true) → 메서드 불필요.
{{< /callout >}}

### 사실 `toString()`과 `String()`도 동작이 다르다

`toString()`이 객체의 메서드라는 점에서 힌트를 얻을 수 있어요. **메서드는 객체에 속한 것**이기 때문에, 객체가 아닌 값에서는 한계가 있어요:

```js
String(null)          // "null"      ✅
String(undefined)     // "undefined" ✅
null.toString()       // ❌ TypeError — null은 객체가 아니라 메서드가 없다
undefined.toString()  // ❌ TypeError — undefined도 마찬가지
```

`String()`은 엔진의 추상 연산 `ToString`을 호출하는 것이라 어떤 값이든 처리할 수 있어요. 반면 `toString()`은 객체(또는 오토박싱된 래퍼)에서 호출되는 메서드라, null/undefined처럼 객체가 아닌 값에서는 호출 자체가 불가능한 거예요.

또한 `toString()`은 객체의 메서드이기 때문에, `String()`에 없는 **객체별 고유 기능**을 제공하기도 해요:

```js
(255).toString(16)    // "ff"       — 16진법 (Number.prototype.toString의 고유 기능)
(255).toString(2)     // "11111111" — 2진법
String(255)           // "255"      — 항상 10진법, 커스텀 불가
```

---

## 5. 단축 평가 — 논리 연산자의 진짜 동작

`&&`와 `||`는 **불리언을 반환하는 것이 아니라, 판단이 결정된 순간의 피연산자를 그대로 반환**해요. 이것을 **단축 평가(short-circuit evaluation)** 라고 해요.

### `||` — 첫 번째 truthy를 찾는다

```js
'hello' || 'world'     // 'hello' — 이미 truthy, 끝
0 || 'fallback'        // 'fallback' — 0은 falsy, 다음으로
0 || '' || 'default'   // 'default' — 0 falsy, '' falsy, 마지막 도달
```

### `&&` — 첫 번째 falsy를 찾는다

```js
'cat' && 'dog'    // 'dog' — 'cat'이 truthy이므로 끝까지 평가
0 && 'dog'        // 0 — 이미 falsy, 끝
```

### 실용 패턴

```js
// 기본값 할당
const name = userInput || '익명';

// 조건부 함수 실행
isLoggedIn && showDashboard();

// null/undefined 체크
const length = str && str.length;
```

---

## 6. 모던 JS의 진화: `?.`과 `??`

`||`로 기본값을 처리하는 패턴에는 치명적인 문제가 있었어요.

### `||`의 한계

{{< callout type="danger" >}}
```js
const count = userCount || 10;
// userCount가 0이면? → 0은 falsy → 10이 할당됨!
// 의도: "값이 없을 때만 10"
// 현실: "0이어도 10"
```
{{< /callout >}}

### `??` — null 병합 연산자 (ES2020)

`null`과 `undefined`**만** 걸러내요. 다른 falsy 값(0, '', false)은 그대로 통과해요:

```js
0 || 10      // 10  — 0을 falsy로 판정
0 ?? 10      // 0   — 0은 null/undefined가 아니니 통과

'' || '기본값'  // '기본값'
'' ?? '기본값'  // ''

false || true   // true
false ?? true   // false
```

### `?.` — 옵셔널 체이닝 (ES2020)

중첩된 객체의 프로퍼티에 안전하게 접근하기 위한 연산자예요. `null`이나 `undefined`를 만나면 에러 대신 `undefined`를 반환해요:

```js
const user = { address: { city: 'Seoul' } };

// 옵셔널 체이닝 없이
const city = user && user.address && user.address.city;

// 옵셔널 체이닝으로
const city = user?.address?.city;  // 'Seoul'

const user2 = null;
user2?.address?.city;  // undefined (에러 안 남)
```

다양한 형태가 있어요:

```js
obj?.prop       // 프로퍼티 접근
obj?.[expr]     // 계산된 프로퍼티 접근
func?.()        // 함수 호출 (func가 null이면 undefined)
```

### `??`와 `?.`의 조합

```js
const city = user?.address?.city ?? '미지정';
// user가 없거나 address가 없거나 city가 없으면 → '미지정'
// city가 빈 문자열 ''이면 → '' (빈 문자열 유지)
```

이 조합이 `||`보다 의도를 정확하게 표현하기 때문에, 모던 코드에서는 기본값 처리에 `??`를 쓰는 것이 권장돼요.

---

## 정리

| 주제 | 핵심 |
|------|------|
| JS의 설계 철학 | 에러보다 변환을 택함 — 편의성 우선, 대신 규칙을 알아야 한다 |
| 암묵적 변환 | `ToString`, `ToNumber`, `ToBoolean` 추상 연산이 자동 호출 |
| `+` 연산자 | 문자열이 하나라도 있으면 문자열 연결, 나머지 산술 연산자는 숫자 변환 |
| falsy 값 | 7개뿐: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN` |
| `toString()` | 객체마다 답이 달라야 해서 메서드로 존재, `toBoolean()`은 항상 같아서 불필요 |
| 단축 평가 | `\|\|`는 첫 truthy, `&&`는 첫 falsy를 반환 — 불리언이 아닌 값 자체를 반환 |
| `??` vs `\|\|` | `??`는 null/undefined만 걸러냄, `\|\|`는 모든 falsy를 걸러냄 |
| `?.` | null/undefined 참조 시 에러 대신 undefined 반환 |

타입 변환은 JS에서 가장 자주 버그를 만드는 영역이에요. 규칙 자체는 많지 않지만, **"이 맥락에서 어떤 변환이 일어나는지"를 의식하는 습관**이 중요해요. 확실하지 않으면 명시적으로 변환하세요 — 코드의 의도가 명확해지고, 동료도 (미래의 나도) 감사할 거예요.

---

## 연산자 퀴즈 — 타입 변환 마스터 도전!

위에서 배운 내용을 직접 실험해보세요. 주어진 요소들을 **전부** 사용해서 목표 값을 만들어야 해요. 괄호 `()`는 무제한이에요.

<div id="oq-root"></div>

<style>
.oq-box{max-width:520px;margin:1.5rem 0;padding:1.25rem 1.5rem;border-radius:12px;border:1px solid #d1d5db;font-family:inherit;background:#f8fafc}
html.dark .oq-box{background:#1e293b;border-color:#334155}
.oq-target{margin:.75rem 0;padding:.75rem 1rem;border-radius:8px;background:#e0f2fe;text-align:center;font-size:1.1rem}
html.dark .oq-target{background:#0c4a6e}
.oq-target code{font-size:1.25rem;font-weight:700;color:#0369a1}
html.dark .oq-target code{color:#7dd3fc}
.oq-target .oq-type{font-size:.85rem;opacity:.7;margin-left:.35rem}
.oq-tokens{display:flex;flex-wrap:wrap;gap:.4rem;margin:.75rem 0}
.oq-tok{display:inline-flex;align-items:center;gap:.3rem;padding:.25rem .6rem;border-radius:6px;font-family:'SF Mono',Monaco,monospace;font-size:.85rem;font-weight:500;border:1px solid #a7f3d0;background:#ecfdf5;color:#065f46;transition:all .2s}
html.dark .oq-tok{border-color:#064e3b;background:#064e3b;color:#6ee7b7}
.oq-tok.used{border-color:#d1d5db;background:#f1f5f9;color:#94a3b8;text-decoration:line-through}
html.dark .oq-tok.used{border-color:#334155;background:#1e293b;color:#475569}
.oq-tok.over{border-color:#fca5a5;background:#fef2f2;color:#dc2626}
html.dark .oq-tok.over{border-color:#7f1d1d;background:#450a0a;color:#fca5a5}
.oq-tok .oq-cnt{font-size:.7rem;opacity:.65}
.oq-paren{font-size:.8rem;color:#64748b;margin-left:.25rem}
.oq-input-wrap{margin:.75rem 0}
.oq-input{width:100%;padding:.6rem .8rem;border-radius:8px;border:1.5px solid #cbd5e1;font-family:'SF Mono',Monaco,monospace;font-size:1rem;background:#fff;color:#0f172a;outline:none;transition:border-color .15s}
html.dark .oq-input{background:#0f172a;border-color:#475569;color:#e2e8f0}
.oq-input:focus{border-color:#0ea5e9}
.oq-result{padding:.6rem .8rem;border-radius:8px;font-family:'SF Mono',Monaco,monospace;font-size:.9rem;min-height:2.4rem;background:#f1f5f9;color:#475569;transition:all .2s}
html.dark .oq-result{background:#0f172a;color:#94a3b8}
.oq-result.error{color:#ef4444}
.oq-result.correct{background:#ecfdf5;color:#059669;font-weight:600}
html.dark .oq-result.correct{background:#064e3b;color:#6ee7b7}
.oq-label{font-size:.8rem;font-weight:600;color:#64748b;margin-bottom:.3rem}
html.dark .oq-label{color:#94a3b8}
.oq-success{margin-top:.75rem;padding:.75rem 1rem;border-radius:8px;background:#ecfdf5;border:1px solid #a7f3d0;color:#065f46;display:none}
html.dark .oq-success{background:#064e3b;border-color:#065f46;color:#a7f3d0}
.oq-success.show{display:block;animation:oq-pop .3s ease}
@keyframes oq-pop{0%{transform:scale(.95);opacity:0}100%{transform:scale(1);opacity:1}}
.oq-hint-btn{margin-top:.5rem;padding:.3rem .7rem;border-radius:6px;border:1px solid #cbd5e1;background:transparent;color:#64748b;font-size:.8rem;cursor:pointer}
html.dark .oq-hint-btn{border-color:#475569;color:#94a3b8}
.oq-hint{margin-top:.4rem;font-size:.85rem;color:#0369a1;display:none}
html.dark .oq-hint{color:#7dd3fc}
.oq-explain{margin-top:.5rem;font-size:.85rem;line-height:1.5;color:#065f46;display:none}
html.dark .oq-explain{color:#a7f3d0}
</style>

<script>
(function(){
  var PUZZLE = {
    target: 0,
    targetDisplay: '0',
    targetType: 'number',
    tokens: {'true':1, '&&':1, '""':1, 'null':1, '||':1, '+':1},
    hint: '<code>&&</code>는 truthy를 만나면 다음으로, <code>||</code>는 falsy를 만나면 다음으로 넘어가요. 둘을 체이닝하면?',
    explain: '정답 예시: <code>+((true && null) || "")</code><br>① <code>true && null</code> → <code>null</code> (true는 truthy, &&가 끝까지 평가해 null 반환)<br>② <code>null || ""</code> → <code>""</code> (null은 falsy, ||가 다음으로 넘어가 "" 반환)<br>③ <code>+""</code> → <code>0</code> (단항 +로 숫자 변환)<br><br>다른 풀이: <code>+((true && "") || null)</code><br>① <code>true && ""</code> → <code>""</code> → ② <code>"" || null</code> → <code>null</code> → ③ <code>+null</code> → <code>0</code>'
  };

  var TOKEN_RE = /"(?:[^"\\]|\\.)*"|'(?:[^'\\]|\\.)*'|undefined|typeof|true|false|null|NaN|===|!==|==|!=|&&|\|\||\?\?|\d+(?:\.\d+)?|[+\-*\/!~%]/g;

  function tokenize(expr){
    var m, out=[];
    TOKEN_RE.lastIndex=0;
    while((m=TOKEN_RE.exec(expr))!==null) out.push(m[0].replace(/'/g,'"'));
    return out;
  }

  function countUsed(expr){
    var toks=tokenize(expr), counts={};
    for(var i=0;i<toks.length;i++) counts[toks[i]]=(counts[toks[i]]||0)+1;
    return counts;
  }

  function safeEval(expr){
    if(!expr.trim()) return {ok:false, msg:'입력을 시작하세요'};
    // 보안: 할당문, 함수 호출 등 차단 (괄호 안 연산만 허용)
    if(/[;{}]|=>|function|eval|new |delete |void /.test(expr))
      return {ok:false, msg:'허용되지 않는 구문이에요'};
    try{
      var r = (0,eval)(expr);
      return {ok:true, value:r, type:typeof r};
    }catch(e){
      return {ok:false, msg:'구문 오류'};
    }
  }

  function render(){
    var root=document.getElementById('oq-root');
    if(!root) return;
    var html='<div class="oq-box">';
    html+='<div class="oq-label">목표값</div>';
    html+='<div class="oq-target"><code>'+PUZZLE.targetDisplay+'</code><span class="oq-type">'+PUZZLE.targetType+'</span></div>';
    html+='<div class="oq-label">사용할 요소 (전부 사용해야 해요!)</div>';
    html+='<div class="oq-tokens" id="oq-toks">';
    var keys=Object.keys(PUZZLE.tokens);
    for(var i=0;i<keys.length;i++){
      var k=keys[i], n=PUZZLE.tokens[k];
      html+='<span class="oq-tok" data-tok="'+k+'"><code>'+k+'</code><span class="oq-cnt">&times;'+n+'</span></span>';
    }
    html+='<span class="oq-paren">( ) 무제한</span>';
    html+='</div>';
    html+='<div class="oq-input-wrap"><input class="oq-input" id="oq-in" placeholder="식을 입력하세요 (예: 1 + 1)" autocomplete="off" spellcheck="false"></div>';
    html+='<div class="oq-label">현재 평가 결과</div>';
    html+='<div class="oq-result" id="oq-res">입력을 시작하세요</div>';
    html+='<div class="oq-success" id="oq-ok"><strong>정답!</strong> 타입 변환의 연쇄를 꿰뚫었어요.<span class="oq-explain" id="oq-exp"></span></div>';
    html+='<button class="oq-hint-btn" id="oq-hint-btn">힌트 보기</button>';
    html+='<div class="oq-hint" id="oq-hint">'+PUZZLE.hint+'</div>';
    html+='</div>';
    root.innerHTML=html;

    var input=document.getElementById('oq-in');
    var res=document.getElementById('oq-res');
    var toks=document.getElementById('oq-toks').children;
    var ok=document.getElementById('oq-ok');
    var exp=document.getElementById('oq-exp');
    var hintBtn=document.getElementById('oq-hint-btn');
    var hintEl=document.getElementById('oq-hint');
    var solved=false;

    hintBtn.addEventListener('click',function(){
      hintEl.style.display=hintEl.style.display==='block'?'none':'block';
      hintBtn.textContent=hintEl.style.display==='block'?'힌트 숨기기':'힌트 보기';
    });

    input.addEventListener('input',function(){
      if(solved) return;
      var expr=input.value;
      var used=countUsed(expr);
      var keys=Object.keys(PUZZLE.tokens);
      var allValid=true;

      // 토큰 뱃지 업데이트
      var allUsed=true;
      for(var i=0;i<keys.length;i++){
        var k=keys[i], allowed=PUZZLE.tokens[k], u=used[k]||0;
        var el=toks[i];
        var remaining=allowed-u;
        el.className='oq-tok'+(remaining<=0?(remaining<0?' over':' used'):'');
        el.querySelector('.oq-cnt').textContent=(remaining>0?'\u00d7'+remaining:remaining<0?'\u00d7'+remaining:'\u2713');
        if(remaining<0) allValid=false;
        if(remaining>0) allUsed=false;
      }
      // 허용되지 않은 토큰 체크
      for(var t in used){
        if(!(t in PUZZLE.tokens)){allValid=false;break;}
      }
      // 모든 토큰을 전부 사용해야 함
      if(!allUsed) allValid=false;

      var ev=safeEval(expr);
      if(!ev.ok){
        res.textContent=ev.msg;
        res.className='oq-result'+(ev.msg==='입력을 시작하세요'?'':' error');
        return;
      }

      var display=typeof ev.value==='string'?'"'+ev.value+'"':String(ev.value);
      res.textContent=display+' ('+ev.type+')';

      var match=Number.isNaN(PUZZLE.target)?Number.isNaN(ev.value):ev.value===PUZZLE.target;
      if(match && allValid){
        res.className='oq-result correct';
        ok.className='oq-success show';
        exp.innerHTML=PUZZLE.explain;
        exp.style.display='block';
        solved=true;
        input.style.borderColor='#059669';
      }else{
        res.className='oq-result';
      }
    });
  }

  if(document.readyState==='loading') document.addEventListener('DOMContentLoaded',render);
  else render();
})();
</script>
