---
title: "ch19. __proto__·빌트인·V8 내부 슬롯"
linkTitle: "ch19 프로토타입 메모"
weight: 19
type: docs
---

> 프로토타입을 공부하다 갈래로 새어나온 글감 메모예요. 다듬은 본문이 아니라 "이걸로 글을 쓰겠다"는 포인트 모음입니다.

## `__proto__`도 결국 일개 프로퍼티일 뿐이다

- 핵심 명제: `__proto__`는 언어의 특별한 문법(키워드)이 아니라 `Object.prototype`에 정의된 **접근자 프로퍼티**일 뿐이다.
  - 즉 모든 객체가 프로토타입 체인을 통해 상속받아 쓰는 평범한 프로퍼티.
  - 직접 객체에 `__proto__`라는 데이터 프로퍼티를 만들면 그놈과는 별개다 (`Object.getOwnPropertyDescriptor(Object.prototype, '__proto__')`로 증명).

- **왜 데이터 프로퍼티가 아니라 접근자 프로퍼티여야 했나**
  - `[[Prototype]]`은 엔진이 관리하는 내부 슬롯이라 값을 그냥 읽고 쓰면 안 된다.
  - getter/setter로 감싸야 `set __proto__`에서 순환 참조 검사, 객체 여부 검사 등 엔진 차원의 동작을 끼워넣을 수 있다.
  - 데이터 프로퍼티였다면 그냥 슬롯을 덮어써서 프로토타입 체인을 깨뜨릴 수 있었을 것.
  - → 접근자 프로퍼티 = "프로퍼티 접근을 함수 호출로 가로채는" 메커니즘. (16장 프로퍼티 어트리뷰트와 연결)

- **JS에서 "언어가 공식 지원하는 함수처럼 보이는 것"을 직접 만드는 방법**
  - `__proto__`, `length`, `size` 등은 마법이 아니라 전부 접근자 프로퍼티 / 빌트인으로 구현된 것.
  - `Object.defineProperty`로 get/set을 정의하면 나도 똑같이 "값처럼 접근하지만 실제론 함수가 도는" API를 만들 수 있다.
  - 예: 클래스의 getter/setter, 라이브러리들이 `obj.foo`처럼 자연스러운 인터페이스를 제공하는 원리.
  - 결론: 언어 내장 기능과 사용자 정의 기능 사이의 경계가 생각보다 얇다 — 같은 도구(접근자 프로퍼티)로 만들어진다.

- (인사이트 연결) `[[Prototype]]`이 왜 접근자로 감싸여야 했는지는 **엔진 메모리 구조**를 보면 더 명확해짐 → 아래 V8 섹션 참고.

## 빌트인 객체도 결국 전역 객체의 프로퍼티다

- `__proto__`가 일개 프로퍼티였던 것과 같은 결의 이야기: 표준 빌트인 객체(`Object`, `Number`, `Array`, `Function`, `Math`, `JSON` 등)도 마법 키워드가 아니라 **전역 객체(`globalThis`)의 프로퍼티**로 등록되어 있을 뿐이다.
  - `globalThis.Object === Object`, `globalThis.Math === Math` → true 로 증명.
  - 그래서 선언 없이 어디서든 쓸 수 있다 — 전역 객체가 전역 스코프의 최상위라서 식별자 해석이 전역 객체 프로퍼티로 귀결되기 때문.
- 포인트: "언어가 기본 제공하는 것"처럼 보이는 이름들이 실은 **평범한 객체의 평범한 프로퍼티**로 얹혀 있다. `__proto__`(접근자 프로퍼티) → 빌트인 객체(전역 객체의 프로퍼티)로 같은 메시지가 반복됨.
- 환경별 전역 객체: 브라우저 `window`, Node.js `global`, 표준화된 통합 이름 `globalThis`(ES2020).
- `Math`/`JSON`은 생성자가 아닌 정적 객체라는 차이도 함께 짚기 (new 못 함).
- (TMI) 이건 엔진이 그냥 해주는 게 아니라 스펙에 절차로 박혀 있다 — Realm 생성 시 `CreateIntrinsics`(표준 객체 생성) → `SetDefaultGlobalBindings`(전역 객체 프로퍼티로 바인딩). V8에선 `Genesis`(bootstrapper.cc)가 이 일을 하고, 매번 새로 설치하면 느리니 부트스트랩 끝난 힙을 스냅샷으로 직렬화해두고 복원함.

## V8 내부 — `[[Prototype]]` 등 내부 슬롯은 어디에 저장되나

- `[[Prototype]]`은 객체 인스턴스의 일반 프로퍼티들과 **같은 곳에 저장되지 않는다.**
- V8 객체 레이아웃: `[Map 포인터][properties 백킹스토어][elements][in-object 프로퍼티들...]`
  - 일반 named 프로퍼티 → in-object 슬롯 or properties 배열(빠름) / dictionary 모드(느림)에 **인스턴스별**로 저장.
  - `[[Prototype]]`은 인스턴스가 아니라 그 객체의 **Map(Hidden Class)** 안의 `prototype` 필드에 저장됨.
- 함의:
  - 같은 Hidden Class를 공유하는 객체들은 프로토타입도 공유 (shape 메타데이터의 일부).
  - `Object.setPrototypeOf` / `__proto__` 변경 → **Hidden Class 전이(transition)** 발생 → 비싸고 최적화(IC) 깨짐. "프로토타입 바꾸지 마라"의 엔진 레벨 근거.
  - `[[Extensible]]` 등 다른 내부 슬롯도 Map의 bit field에 저장 — 역시 인스턴스 데이터와 분리.
- → 10장 글의 Hidden Class 다이어그램(Map 포인터 / Properties / Prototype 포인터)과 그대로 이어짐.
