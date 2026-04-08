---
title: "ch10. 객체 리터럴 — 목판과 복사기, 두 세계의 객체 만들기"
date: 2026-04-08
tags: [javascript, deep-dive, ch10, object, hidden-class, v8]
weight: 4
---

Java에서는 클래스를 먼저 정의해야 객체를 만들 수 있어요. JavaScript에서는 `{}`만 쓰면 바로 객체가 생겨요. 이건 단순한 문법 차이가 아니라, 객체를 바라보는 **철학 자체가 다르기 때문**이에요. 이번 글에서는 객체 리터럴의 문법을 넘어서, "JS는 왜 이렇게 동작하는가"를 엔진 내부까지 파고들어 정리했어요.

---

## 1. 목판 인쇄 vs 복사기 — 두 세계의 객체 만들기

프로그래밍 언어에서 객체를 만드는 방식은 크게 두 갈래로 나뉘어요.

### 목판 인쇄 방식 (클래스 기반 — Java, C++)

팔만대장경을 떠올려 보세요. 경전을 찍으려면 먼저 **목판(경판)에 글자를 하나하나 새겨야** 해요. 목판 자체는 경전이 아니에요. 목판에 먹칠을 하고 종이에 찍으면 그제서야 경전이 나오죠.

새로운 경전을 만들고 싶으면? 같은 목판에 넣고 같은 방식으로 찍으면 돼요. 하지만 **내용을 수정하려면 새 목판을 깎아야** 해요. 이미 찍힌 종이에 글자를 추가하는 것도 불가능하고요.

```java
// Java — 목판(클래스)을 먼저 새겨야 한다
class Dog {
    String name;
    int age;
    void bark() { System.out.println("Woof!"); }
}

Dog myDog = new Dog();   // 목판으로 찍어냄
Dog yourDog = new Dog(); // 같은 목판으로 또 찍어냄
// myDog.color = "brown"; // ❌ 컴파일 에러! 목판에 없는 건 추가 불가
```

```
[목판 = 클래스 Dog]              [찍어낸 경전 = 인스턴스 myDog]
  name: String              →     name: null
  age: int                  →     age: 0
  bark()                    →     bark()

목판(추상) → 경전(구체). 목판 없이는 경전을 찍을 수 없다.
내용을 바꾸려면 새 목판을 깎아야 한다.
```

### 복사기 방식 (프로토타입 기반 — JavaScript)

복사기는 다르죠. **이미 있는 문서를 넣고 복사**해요. 원본도 문서고, 복사본도 문서예요. 복사 후에 펜으로 내용을 수정하거나 페이지를 추가할 수도 있어요.

```js
// JavaScript — 원본 문서(객체)를 복사해서 새 문서(객체)를 만든다
const dog = {
  bark() { console.log("Woof!"); }
};

const myDog = Object.create(dog); // dog을 원본으로 복사
myDog.name = "멍이";              // 복사본에 자유롭게 추가
myDog.age = 3;                    // 이것도 OK
myDog.bark();                     // "Woof!" — 원본에서 찾아온다
```

```
[원본 문서 = dog]             [복사본 = myDog]
  bark()              ←────   __proto__ 연결
                               name: "멍이"
                               age: 3

구체 → 구체. 원본도 객체, 복사본도 객체. 설계도가 필요 없다.
```

핵심 차이를 정리하면:

| | 목판 인쇄 (클래스 기반) | 복사기 (프로토타입 기반) |
|---|---|---|
| 필요한 것 | 목판(클래스) = 추상적 설계도 | 원본 문서(객체) = 이미 존재하는 실체 |
| 제품 | 목판에서 찍어낸 경전(인스턴스) | 원본을 복사한 문서(객체) |
| 내용 수정 | 새 목판을 깎아야 함 | 복사본에 펜으로 자유롭게 수정 |
| 구조 추가 | 불가능 — 목판에 새긴 대로만 | 언제든 페이지 추가/삭제 가능 |
| 상속 | 목판 → 목판 (클래스 → 클래스) | 문서 → 문서 (객체 → 객체) |

{{< callout type="info" >}}
JS의 `class` 문법(ES6)은 **목판 인쇄처럼 포장한 복사기**예요. 겉으로는 클래스처럼 보이지만, 내부는 여전히 프로토타입 기반으로 동작해요. 이 부분은 19장(프로토타입)에서 자세히 다뤄요.
{{< /callout >}}

---

## 2. 객체 리터럴 — `{}`의 두 얼굴

JS에서 객체를 가장 간단하게 만드는 방법은 **객체 리터럴**이에요.

```js
const dog = {
  name: "멍이",
  age: 3
};
```

여기서 하나 짚을 게 있어요. 같은 중괄호 `{}`인데, 어디서는 세미콜론을 붙이고 어디서는 안 붙여요:

```js
// 객체 리터럴 — 값으로 평가되는 표현식이라 세미콜론을 붙인다
const dog = { name: "멍이" };

// 코드 블록 — 문(statement)이라 세미콜론을 안 붙인다
if (true) { console.log("hi") }
```

{{< callout type="warning" >}}
객체 리터럴의 `{}`는 **값을 만들어내는 표현식**이에요. `const dog = 42;`에서 `42`와 같은 위치에 있는 거죠. 반면 `if`의 `{}`는 코드를 묶는 블록이라 값이 아니에요.
{{< /callout >}}

---

## 3. 프로퍼티 — 키와 값의 규칙

객체는 **프로퍼티(키-값 쌍)의 집합**이에요.

### 키의 타입 — 문자열이 기본, Symbol은 예외

`[]` 안에 무엇을 넣든 **표현식으로 평가**돼요. 그 결과값이 키로 사용되는데, 대부분은 문자열로 변환돼요:

```js
const obj = {};

obj[1] = "숫자";        // 1 → "1"로 변환
obj[true] = "불리언";   // true → "true"로 변환
obj[null] = "널";       // null → "null"로 변환

console.log(Object.keys(obj));
// ['1', 'true', 'null'] — 전부 문자열이 되었다

obj['1'] === obj[1];   // true — 같은 키다
```

그런데 **Symbol만은 문자열로 변환되지 않고 그대로 키가 돼요**:

```js
const sym = Symbol("id");
const obj = { [sym]: "심볼 값" };

// Symbol은 문자열로 변환되지 않는다
obj["Symbol(id)"];    // undefined — 문자열 키 "Symbol(id)"는 없다
obj[sym];             // "심볼 값" — Symbol 값 자체가 키다
```

정리하면, `[]`는 "문자열로 변환하는 문법"이 아니라 "표현식을 평가하는 문법"이에요. 평가 결과의 타입에 따라 동작이 달라져요:

```
[]에 넣으면
├── 문자열 → 그대로 문자열 키
├── Symbol → 그대로 Symbol 키 (변환 없음, 유일한 예외)
└── 그 외   → 문자열로 변환 → 문자열 키
```

### 마침표(`.`) vs 대괄호(`[]`) 접근

```js
const obj = { name: "멍이" };
const key = "name";

obj.name     // "멍이" — . 뒤는 항상 문자열 리터럴 취급
obj.key      // undefined — 문자열 "key"로 찾음 (변수가 아님!)
obj[key]     // "멍이" — 변수 key의 값 "name"으로 찾음
```

**`[]` 안은 표현식으로 평가돼요.** 따옴표가 없으면 변수로, 있으면 문자열 리터럴로 해석해요:

```js
obj["name"]           // 문자열 리터럴 → "name"
obj[name]             // 변수 name의 값을 평가
obj["na" + "me"]      // 표현식 → "name"
obj[1 + 1]            // 표현식 → 2 → 문자열 "2"
```

| 상황 | `.` | `[]` |
|------|-----|------|
| 일반적인 프로퍼티 | `obj.name` | `obj["name"]` |
| 특수문자 포함 키 | ❌ | `obj["first-name"]` |
| 동적 키 (변수) | ❌ | `obj[key]` |
| Symbol 키 | ❌ | `obj[sym]` |

---

## 4. 찰흙인데 어떻게 빠르지? — V8 Hidden Class

여기서 자연스러운 의문이 생겨요. JS 객체는 런타임에 프로퍼티를 자유롭게 추가/삭제할 수 있는데, **어떻게 C++에 가까운 속도로 프로퍼티에 접근할 수 있을까?**

C++에서 클래스를 미리 정의하는 이유 중 하나는, 컴파일 타임에 **메모리 레이아웃을 확정**하기 위해서예요:

```
[C++ Dog 인스턴스 — 컴파일 시 크기 확정]
┌─────────────────────────┐
│ name (32바이트, 오프셋 0) │
│ age  (4바이트, 오프셋 32) │
└─────────────────────────┘
  → 필드 접근 = 단순 오프셋 계산 (초고속)
```

JS에는 이런 사전 정보가 없잖아요. 그래서 V8 엔진(Chrome, Node.js)은 **Hidden Class**라는 전략을 씁니다.

### Hidden Class — 엔진이 알아서 만드는 설계도

```js
const dog = {};      // Hidden Class H0: {}
dog.name = "멍이";   // Hidden Class H1: { name }
dog.age = 3;         // Hidden Class H2: { name, age }
```

```
프로퍼티를 추가할 때마다 Hidden Class가 전이(transition)된다:

  H0 {}  ──name──→  H1 {name}  ──age──→  H2 {name, age}
                     오프셋 0              오프셋 0, 1
```

같은 순서로 같은 프로퍼티를 추가하면 **같은 Hidden Class를 공유**해요:

```js
// 이 두 객체는 같은 Hidden Class H2를 공유한다
const dog1 = {}; dog1.name = "멍이"; dog1.age = 3;
const dog2 = {}; dog2.name = "바둑"; dog2.age = 5;
// → 둘 다 H2 → 오프셋 기반 빠른 접근 (C++과 유사한 속도)
```

"클래스가 없다"가 아니라 **"엔진이 런타임에 알아서 클래스를 만든다"** 에 가까운 거예요.

### 왜 빨라질까? — Hidden Class + Inline Cache

"잠깐, Hidden Class 안에서 'name이 오프셋 몇 번인지' 찾는 것도 결국 해시맵 조회 아냐?" — 맞아요. Hidden Class 자체의 내부 구조도 해시맵과 유사해요. **Hidden Class만으로는 특별히 빠르지 않아요.**

진짜 빨라지는 이유는 **Inline Cache(IC)** 에 있어요. Hidden Class는 IC가 작동할 수 있는 토대를 제공하는 거예요.

```
[1단계] 첫 접근 — Hidden Class 조회 (해시맵과 비슷한 비용)

  obj.name → Hidden Class에서 "name" 검색 → 오프셋 0 발견
             → IC 캐시에 저장: "H2일 때 name = 오프셋 0"

[2단계] 반복 접근 — IC 캐시 히트 (해시맵 조회 없음!)

  obj.name → Hidden Class가 H2인지? → YES → 바로 Properties[0] 읽기
             (정수 비교 한 번으로 끝)
```

같은 구조의 객체가 반복적으로 접근되면, IC 덕분에 해시 조회를 완전히 건너뛰어요:

```js
function getName(obj) {
  return obj.name;  // 이 코드가 반복 실행된다면?
}

getName({ name: "멍이", age: 3 });  // Hidden Class H2에서 name = 오프셋 0 → IC에 캐싱
getName({ name: "바둑", age: 5 });  // 같은 H2 → IC 히트! → 바로 오프셋 0
getName({ name: "초코", age: 2 });  // 같은 H2 → IC 히트!
```

C의 심볼 테이블과 비교하면 역할이 명확해져요:

| | C | V8 |
|---|---|---|
| 매핑 정보 보관 | 심볼 테이블 (컴파일 타임) | Hidden Class (런타임) |
| 실제 빠른 접근 | 컴파일된 오프셋 (바이너리에 구워짐) | Inline Cache (런타임에 학습) |
| 비용 | 없음 | Hidden Class + IC 유지 비용 |

{{< callout type="info" >}}
"그러면 해시맵에서도 결과를 캐싱하면 되는 거 아냐?"라는 의문이 들 수 있어요. 핵심은 **캐시 히트를 판단하는 비용**이에요.

IC가 캐싱한 결과를 재사용하려면 "이 객체가 전에 본 것과 같은 구조인지"를 판단해야 해요. Hidden Class가 있으면 **포인터(정수) 비교 한 번**으로 끝나요. Hidden Class가 없다면? "같은 구조"라는 개념 자체가 없어서, 객체의 모든 키를 비교해야 하고 — 그건 매번 해시맵을 조회하는 것과 다를 바 없어요.

Hidden Class의 진짜 역할은 검색이 아니라, **구조에 신분증을 부여하는 것**이에요. 같은 신분증 = 같은 구조 = 같은 오프셋.
{{< /callout >}}

결국 V8은 **C 컴파일러가 컴파일 타임에 하는 일을, 런타임에 동적으로 재현**하고 있는 셈이에요.

### Hidden Class가 깨지는 경우

```js
// 순서가 다르면 다른 Hidden Class
const dog1 = {}; dog1.name = "멍이"; dog1.age = 3;
const dog2 = {}; dog2.age = 5; dog2.name = "바둑";  // 순서가 다르다!
// → dog1과 dog2는 다른 Hidden Class → 최적화 실패
```

### `delete`는 Hidden Class를 파괴한다

`delete` 연산자는 메모리를 직접 해제하는 게 아니라 **프로퍼티와 객체의 연결을 끊는 것**이에요. C++의 `delete`와는 완전히 달라요.

```js
const obj = { name: "멍이", age: 3 };
delete obj.name;
// name에 연결된 "멍이"는 GC가 나중에 회수한다
```

문제는 성능이에요. V8은 프로퍼티 추가에 대해서는 Hidden Class 전이를 캐싱하지만, **삭제에 대해서는 포기하고 딕셔너리 모드(해시맵)로 전환**해요:

```
[Fast 모드 — Hidden Class]           [Slow 모드 — Dictionary]
  오프셋 기반 접근 (빠름)     delete     해시맵 조회 (느림)
                           ────────→
                          한 번 빠지면 복구 안 됨
```

{{< callout type="danger" >}}
`delete`를 사용하면 Hidden Class가 파괴되어 **같은 프로퍼티 접근이 수 배 느려질 수 있어요.** 값을 제거하고 싶다면 `obj.prop = undefined`로 Hidden Class를 유지하는 것이 성능에 유리해요. 단, `"prop" in obj`의 결과가 달라지니까(`undefined` 할당은 `true`, `delete`는 `false`) 상황에 맞게 선택하세요.
{{< /callout >}}

### 직접 체험: delete의 성능 영향

말로만 들으면 와닿지 않으니, 직접 측정해 보세요. 객체 수 슬라이더를 조절하며 차이를 확인할 수 있어요.

<iframe id="demo-hidden-class" src="/demos/ch10-hidden-class-bench/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

<script>window.addEventListener('message',function(e){if(e.data&&e.data.type==='demo-resize'){document.getElementById('demo-hidden-class').style.height=e.data.height+16+'px'}});</script>

---

## 5. ES6 확장 — 단순 편의 문법이 아닌 것들

### 프로퍼티 축약 표현

변수명과 키가 같으면 생략할 수 있어요:

```js
const name = "멍이";
const age = 3;

// 축약 전
const dog = { name: name, age: age };
// 축약 후
const dog = { name, age };
```

### 계산된 프로퍼티명

`[]` 안에 표현식을 넣어 키를 동적으로 결정해요:

```js
const prefix = "dog";
const obj = {
  [prefix + "Name"]: "멍이",      // dogName: "멍이"
  [prefix + "Age"]: 3             // dogAge: 3
};
```

### 메서드 축약 표현 — 단순 축약이 아니다

```js
const obj = {
  // ES5 — 프로퍼티에 함수 할당
  sayHi: function() { return "Hi"; },

  // ES6 — 메서드 축약 표현
  sayHello() { return "Hello"; }
};
```

이 둘은 겉보기에만 같아요. 내부적으로 메서드 축약 표현만 가지는 특별한 것이 있어요: **`[[HomeObject]]`** 내부 슬롯이에요. 이게 있어야 `super` 키워드를 쓸 수 있어요:

```js
const parent = {
  greet() { return "부모입니다"; }
};

const child = {
  // 메서드 축약 — [[HomeObject]]가 있어서 super 사용 가능
  greet() {
    return super.greet() + " → 자식입니다";  // ✅
  }
};

Object.setPrototypeOf(child, parent);
console.log(child.greet());  // "부모입니다 → 자식입니다"
```

```js
const child2 = {
  // 함수 할당 — [[HomeObject]]가 없어서 super 사용 불가
  greet: function() {
    return super.greet();  // ❌ SyntaxError!
  }
};
```

| | 함수 할당 | 메서드 축약 | 화살표 함수 |
|---|---|---|---|
| `this` | 호출한 객체 | 호출한 객체 | 외부 스코프 |
| `super` | ❌ | ✅ | ❌ |
| `new` 가능 | ✅ | ❌ | ❌ |

메서드 축약 표현은 **"진짜 메서드"를 위해 설계된 문법** — 필요한 기능(`super`)은 추가하고, 불필요한 기능(`constructor`)은 뺀 최적화된 형태예요.

---

## 정리

| 주제 | 핵심 |
|------|------|
| 목판 인쇄 vs 복사기 | 클래스 기반 = 설계도(추상) → 제품(구체), 프로토타입 기반 = 실물(구체) → 복사본(구체) |
| `{}`의 두 얼굴 | 객체 리터럴 = 값(표현식, 세미콜론 O), 코드 블록 = 문(세미콜론 X) |
| 프로퍼티 키 | 문자열 또는 Symbol만 가능, 나머지는 암묵적 문자열 변환 |
| `.` vs `[]` | `.`은 리터럴 키, `[]`는 표현식 평가 — 동적 접근은 `[]`만 가능 |
| V8 Hidden Class | 구조에 신분증을 부여 → Inline Cache가 포인터 비교만으로 캐시 히트 판단 |
| `delete`의 비용 | 프로퍼티 연결만 끊음, GC가 회수. Hidden Class를 파괴해서 성능 저하 |
| 메서드 축약 | `[[HomeObject]]`를 가져 `super` 사용 가능, 단순 축약이 아닌 별개 문법 |

JS의 객체는 자유롭지만, 엔진은 그 자유 속에서 질서를 만들어요. 프로퍼티를 같은 순서로 초기화하고, `delete`를 피하는 것만으로도 V8이 만든 질서를 유지할 수 있어요. "자유롭게 쓸 수 있다"와 "자유롭게 써도 된다"는 다른 이야기니까요.
