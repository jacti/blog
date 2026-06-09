---
title: "ch17. 생성자 함수는 결국 그냥 함수였다 — new 와 생성자 함수에 대한 6문 6답"
date: 2026-06-09
tags: [javascript, deep-dive, ch17, constructor, prototype, new-target]
weight: 10
---

> 17장을 한 문장으로 줄이면 이렇다. **생성자 함수라는 특별한 존재는 없다. 평범한 함수에 `new`를 붙였을 때 추가되는 암묵적 행동들이 있을 뿐이다.**

이 글은 17장을 읽고 떠오른 질문 6개를 하나씩 파고들며 정리한 기록이에요. 책을 그대로 옮기기보다 "왜 이렇게 설계했지?"를 따라가며 직접 코드로 확인한 흔적에 가깝습니다.

먼저 출발점만 깔고 가죠. 함수 객체는 내부에 `[[Call]]`과 `[[Construct]]`라는 메서드를 가져요. 일반 호출 `f()`는 `[[Call]]`을, `new f()`는 `[[Construct]]`를 부릅니다. 그리고 `new`로 호출되면 함수에 두 가지 암묵적 행동이 끼어들어요 — **this 바인딩**과 **암묵적 return**. 이 글의 질문 대부분은 결국 이 두 가지를 둘러싼 이야기예요.

---

## Q1. 왜 생성자 함수에 "명시적 return"이 가능하게 만들었을까?

처음 든 의문이에요. `Number(123)`을 `new` 없이 부르면 원시값이 나오잖아요. 그래서 "생성자가 원시값을 return하면 무시하고, 객체면 그걸 반환하는 규칙으로 빌트인을 구현한 게 아닐까?"라고 추측했는데 — **이 추측은 틀렸어요.**

`Number`류 빌트인의 `new` 유무 분기는 명시적 return 규칙과 무관합니다. 빌트인은 `[[Call]]`과 `[[Construct]]`를 스펙에서 아예 따로 정의하고, `new.target`을 직접 검사해서 분기해요.

```js
// Number의 동작을 의사코드로 그리면
// Number(value):
//   value를 숫자로 변환
//   new.target이 undefined면 (= 일반 호출) → 원시 숫자 그대로 반환
//   new.target이 있으면 (= new 호출)      → 래퍼 객체 반환
```

그럼 우리가 17장에서 배운 **명시적 return 규칙의 본질**은 뭘까요? `[[Construct]]`의 마지막은 이렇게 생겼어요.

```
result = 함수 본문 실행 결과
if Type(result) is Object → return result   // 객체면 그걸 반환
else                      → return this     // 아니면 새로 만든 this
```

핵심은 *"원시값을 특별히 무시한다"가 아니라 "객체만 통과시킨다"*예요. `return`을 안 쓴 평범한 생성자는 `undefined`를 반환하는데, `undefined`도 원시값이라 똑같이 this로 떨어져요. 즉 **return을 안 쓴 생성자와 `return 0`을 쓴 생성자가 동일하게 동작**하도록 만든 안전장치인 거죠.

```js
function Foo() {
  this.x = 1;
  return 100;        // 원시값 → 무시됨
}
console.log(new Foo()); // Foo { x: 1 }  (100이 아니다)

function Bar() {
  this.x = 1;
  return { y: 2 };   // 객체 → 이게 채택됨, this는 버려짐
}
console.log(new Bar()); // { y: 2 }  (Bar 인스턴스가 아니다!)
```

**직접 실행해보기** — `return 100`을 객체로 바꾸면 Foo도 그 객체를 돌려줘요.

<iframe src="/demos/ch17-constructor/?only=q1" style="width:100%;border:none;border-radius:12px;min-height:330px;margin:1rem 0;" loading="lazy" title="Q1 명시적 return 실험" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

{{< callout type="info" >}}
"객체면 통과, 아니면 this"라는 한 가지 규칙이 두 가지를 동시에 해결한다. ① `return`을 생략한 평범한 생성자(`undefined` 반환)도 자연스럽게 this를 돌려주고, ② 실수로 원시값을 return해도 인스턴스가 깨지지 않는다. 일관성과 안전성을 한 규칙으로 묶은 설계다.
{{< /callout >}}

---

## Q2. 그래서 "객체 return"을 허용한 게 실제로 쓸모가 있나?

있어요. 생성자가 "방금 만든 this 말고 다른 객체"를 돌려줄 수 있다는 점을 이용하는 패턴들이에요.

**① 인스턴스 캐싱 (같은 인스턴스 재사용)**

```js
function User(id) {
  if (User.cache[id]) return User.cache[id]; // 객체 return → 채택
  this.id = id;
  User.cache[id] = this;
}
User.cache = {};

console.log(new User(1) === new User(1)); // true — new 두 번인데 같은 객체
```

**② jQuery의 `$()`** — 가장 유명한 사례예요. 내부적으로 항상 객체를 명시적 return 합니다.

```js
// jQuery = function(selector, context) {
//   return new jQuery.fn.init(selector, context); // 항상 객체 return
// };
// 덕분에 $()든 jQuery()든 new 없이 써도 늘 jQuery 객체를 받는다.
```

**③ 싱글톤 & Proxy 래핑** — 같은 원리로 "인스턴스를 하나만 유지"하거나 "생성자가 감시용 Proxy를 돌려주게" 만들 수 있어요. 아래에서 **코드를 직접 고쳐 실행**해보세요. 왼쪽 코드를 바꾸고 Run을 누르면 오른쪽에 결과가 바로 나옵니다.

<iframe src="/demos/ch17-constructor/?only=singleton,proxy" style="width:100%;border:none;border-radius:12px;min-height:560px;margin:1rem 0;" loading="lazy" title="Q2 싱글톤·Proxy 실험" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

- **싱글톤**: `if (Config.instance) return Config.instance` — 이미 만든 인스턴스를 객체로 return하니, `new`를 몇 번 해도 항상 같은 객체가 나온다.
- **Proxy 래핑**: 생성자가 `this` 대신 `new Proxy(this, ...)`를 return해서, 그 뒤의 모든 프로퍼티 변경을 가로챈다(반응형 데이터·ORM의 토대).

---

## Q3. 왜 "생성자"를 별도 타입이 아니라 그냥 함수로 만들었을까?

JavaScript는 1995년에 두 가지 상충하는 요구 사이에서 태어났어요.

- **내부 설계**: Self 언어 영향을 받은 **프로토타입 기반**. 원래 "클래스"라는 개념이 없다.
- **외부 압박**: "Java처럼 보이게" 만들라 → Java 개발자에게 친숙한 `new Foo()` 문법을 줘야 했다.

이 둘을 절충한 결과가 **"함수를 생성자로 겸용"**이에요. 함수는 이미 일급 객체이고 `prototype` 프로퍼티를 가질 수 있으니, `new F()` 한 가지 연산만 정의하면 (1) 새 객체 생성 (2) `F.prototype`을 부모로 연결 (3) this 바인딩 후 실행까지 한 번에 처리할 수 있었죠. **"클래스"라는 새 개념을 언어에 추가하지 않고** OOP 골격을 만든 거예요.

그 증거로, **`class`조차 사실은 함수**예요.

```js
class Bar {}
console.log(typeof Bar); // "function"

Bar(); // TypeError: Class constructor Bar cannot be invoked without 'new'
```

`class`는 새로운 종류의 객체가 아니라, "`[[Construct]]`는 가지되 `[[Call]]`은 막아둔(= new 없이 부르면 throw)" 특수한 함수일 뿐이에요.

그리고 모든 함수가 `[[Construct]]`를 갖는 건 아니에요. 정의 방식에 따라 갈려요.

| 정의 방식 | `[[Construct]]` | `new` 가능? |
|---|---|---|
| 함수 선언 / 표현식 | O | O |
| `class` | O (단 `[[Call]]`은 throw) | O |
| 화살표 함수 | **X** | X |
| 메서드 축약 `{ foo() {} }` | **X** | X |
| async / generator | **X** | X |

```js
const arrow = () => {};
new arrow(); // TypeError: arrow is not a constructor
```

**직접 실행해보기** — `class`는 `new` 없이 부르면, 화살표는 `new`로 부르면 각각 어떤 에러가 나는지 확인해보세요.

<iframe src="/demos/ch17-constructor/?only=q3" style="width:100%;border:none;border-radius:12px;min-height:330px;margin:1rem 0;" loading="lazy" title="Q3 new 분기 실험" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

---

## Q4. `prototype`은 `[[Construct]]`가 실행될 때 생기는 건가?

아니에요. 여기를 뒤집어야 해요. **`prototype` 프로퍼티는 함수가 정의되는 순간 이미 만들어집니다.** `new`와 무관해요.

```js
function Foo() {}
console.log(Foo.prototype);                     // { constructor: Foo } — new 안 했는데 이미 존재
console.log(Foo.prototype.constructor === Foo); // true (서로 가리키는 백참조)
```

`[[Construct]]`는 그저 **이미 존재하는 `Foo.prototype`을 참조해서 연결**할 뿐이에요. `new`가 하는 일을 한 줄로 줄이면 **"새 객체의 `[[Prototype]]` ← 생성자의 `prototype`"**이에요.

그럼 "`prototype`은 생성자를 위해 있는 거냐?"는 직관 — **이건 맞아요.** 증거는 생성자로 못 쓰는 함수엔 `prototype`이 아예 없다는 거예요.

```js
const arrow = () => {};
console.log(arrow.prototype); // undefined — 생성자로 못 쓰니 만들 이유가 없다
```

{{< callout type="warning" >}}
이름이 비슷한 두 가지를 꼭 구분하자. `함수.prototype`(prototype **프로퍼티**)은 "생성자가 만들 인스턴스의 부모가 될 객체"로 **constructor 함수만** 갖는다. `객체.[[Prototype]]`(내부 슬롯, `Object.getPrototypeOf`로 조회)은 "그 객체 자신의 부모"로 **모든 객체**가 갖는다. `new`는 전자를 후자에 연결하는 일이다.
{{< /callout >}}

| 이름 | 정체 | 누가 갖나 |
|---|---|---|
| `함수.prototype` (프로퍼티) | 생성자가 만들 인스턴스의 부모가 될 객체 | constructor 함수만 |
| `객체.[[Prototype]]` (내부 슬롯) | 그 객체 자신의 부모 | 모든 객체 |

**직접 실행해보기** — `new`를 한 번도 안 했는데 `Foo.prototype`이 이미 있는지, 화살표엔 없는지 확인해보세요.

<iframe src="/demos/ch17-constructor/?only=q4" style="width:100%;border:none;border-radius:12px;min-height:330px;margin:1rem 0;" loading="lazy" title="Q4 prototype 실험" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

---

## Q5. "프로토타입 기반 언어"가 뭐지? 객체를 복사해서 만드는 건가?

절반만 맞아요. **본질은 "복사"가 아니라 "위임(delegation)"**이고, **특히 JS는 복사를 안 해요.**

- **클래스 기반**(Java): 클래스라는 청사진이 먼저 있고, 그걸로 인스턴스를 찍어낸다.
- **프로토타입 기반**(JS, Self): 클래스가 없다. 객체가 다른 **객체**를 직접 부모로 삼는다. 부모도 자식도 똑같이 그냥 객체다.

"복제(cloning)"는 Self·Io 같은 *일부* 프로토타입 언어의 생성 방식일 뿐이에요. JS는 부모를 복사하지 않고 **링크(참조)로 연결**하고, 프로퍼티를 못 찾으면 런타임에 체인을 타고 올라가 찾아요. 이게 위임이에요. 코드로 보면 분명해져요.

```js
function Animal() {}
const a = new Animal();   // a를 먼저 만든다 (이 순간 speak는 어디에도 없음)

Animal.prototype.speak = function () { return "소리!"; }; // 만든 '후에' 부모에 추가

console.log(a.speak());   // "소리!" — 복사였다면 알 리 없는데, 호출 시점에 체인에서 찾아온다
```

만약 복사 모델이라면 `a`는 생성 시점에 없던 `speak`를 알 수 없어야 해요. 그런데 동작하죠. 부모를 나중에 바꿔도 **살아있는 모든 자식에 즉시 반영**되는 것도 같은 이유예요.

그래서 프로토타입의 역할은 결국 하나로 수렴해요. **프로퍼티 검색을 위임받는 대상.** 덕분에 메서드를 인스턴스마다 복사하지 않고 `prototype`에 하나만 둬도 모두가 공유해요(메모리 효율 + 동적 확장 + 상속).

**직접 실행해보기** — 인스턴스를 만든 '후에' 추가한 메서드가 정말 동작하는지 확인해보세요.

<iframe src="/demos/ch17-constructor/?only=q5" style="width:100%;border:none;border-radius:12px;min-height:330px;margin:1rem 0;" loading="lazy" title="Q5 위임 실험" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

---

## Q6. 스코프 세이프 패턴에서 `instanceof` 뒤에 엉뚱한 타입을 넣으면 무한루프 돌지 않나?

맞아요. **무한 재귀에 빠져 `RangeError`로 터집니다.** 직접 돌려서 확인했어요.

{{< callout type="danger" >}}
아래 코드는 `RangeError: Maximum call stack size exceeded`로 죽는다. 스코프 세이프 가드의 `instanceof` 뒤에는 **자기 자신(생성자)**을 적어야 하며, 엉뚱한 타입을 적으면 출구 없는 무한 재귀가 된다.
{{< /callout >}}

```js
function Person(name) {
  if (!(this instanceof Date)) {  // Person이 아니라 Date를 넣어버림
    return new Person(name);      // new로 만들어도 Date 인스턴스가 아니니 또 가드에 걸림
  }
  this.name = name;
}
new Person("kim"); // RangeError: Maximum call stack size exceeded
```

추적해보면 `new Person()`이 만드는 객체는 **항상 Person 인스턴스(Date 아님)**라서 `this instanceof Date`는 영원히 `false`예요. 가드를 빠져나갈 출구가 없으니 자기 자신을 끝없이 재호출하죠. (`while(true)`와 달리 호출마다 스택이 쌓여서, 한계에 닿는 순간 즉시 throw 돼요.)

올바른 버전은 `Date`를 `Person`으로 바꾸면 돼요. `new`로 만든 객체가 가드 조건을 **만족**하게 되니 두 번째 호출에서 빠져나가죠.

```js
function Person(name) {
  if (!(this instanceof Person)) return new Person(name);
  this.name = name;
}
console.log(Person("kim").name); // "kim" — new 없이 불러도 안전
```

근본 원인은 `instanceof` 뒤에 **타입을 사람이 직접 적어야** 한다는 거예요. 그래서 ES6 이후엔 **`new.target`**을 써요. "new로 호출됐냐"만 보니까 타입을 적을 자리가 없고, 이런 실수가 **구조적으로 불가능**해져요.

```js
function Person(name) {
  if (!new.target) return new Person(name); // 비교할 타입이 없다 → 실수 여지 없음
  this.name = name;
}
```

**직접 실행해보기** — 위쪽 빨간 박스는 페이지를 열면 자동으로 `RangeError`를 토해내고, 아래 박스는 올바른 두 가드(`instanceof` 자기 자신 / `new.target`)가 안전하게 동작하는 걸 보여줘요.

<iframe src="/demos/ch17-constructor/?only=q6,q6b" style="width:100%;border:none;border-radius:12px;min-height:520px;margin:1rem 0;" loading="lazy" title="Q6 무한재귀와 올바른 가드 실험" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

> 덧붙여, `this`는 **재할당이 불가능**해요(`this = ...`는 SyntaxError). 그래서 스코프 세이프 패턴은 "this를 바꾸는" 게 아니라, **가드에 걸리면 `new`로 다시 호출해 그 객체를 return**하는 거예요. 가드가 보통 첫 줄에 오는 이유는 **this를 사용(오염)하기 전에** 분기를 끝내야 하기 때문이고요.

---

## 흔한 오해 / 주의점

- **"생성자 함수는 특별한 함수다"** → 아니다. 그냥 함수다. `new`가 붙었을 때만 `[[Construct]]`가 작동할 뿐. `class`조차 함수다.
- **"생성자가 원시값을 return하면 무시된다"** → 정확히는 "객체만 통과, 나머지(원시값·undefined)는 모두 this". `Number()`의 `new` 분기와는 별개 메커니즘.
- **"`prototype`은 `new` 할 때 생긴다"** → 함수 정의 시점에 이미 있다.
- **"프로토타입 기반 = 객체 복사"** → JS는 복사가 아니라 위임(링크). 부모 수정이 자식에 즉시 반영되는 게 그 증거.
- **"`instanceof` 가드면 안전하다"** → 타입을 잘못 적으면 무한 재귀. `new.target`이 더 안전하고 정확하다.

## 정리

1. 생성자 함수는 별도 타입이 아니라 **`[[Construct]]`를 가진 평범한 함수**다. `new`가 this 바인딩과 암묵적 return을 끼워넣을 뿐.
2. 명시적 return 규칙의 본질은 **"객체만 통과, 나머지는 this"** — 안전장치이자, 캐싱·싱글톤·래핑의 토대.
3. `prototype` 프로퍼티는 **정의 시점에 생기고**, `new`는 그걸 인스턴스의 `[[Prototype]]`으로 연결할 뿐.
4. JS는 **복사가 아니라 위임** 기반이라, 프로토타입은 "프로퍼티 검색을 위임받는 대상"이다.
5. 옛 스코프 세이프 패턴(`instanceof` 가드)은 타입 오타에 취약하다 → 지금은 **`new.target`**.
