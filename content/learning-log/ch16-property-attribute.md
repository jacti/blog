---
title: "ch16. 이 값을 다른 놈들이 변경 불가능하게 해주세요!! 예..? 제가요..?"
date: 2026-06-02
tags: [javascript, deep-dive, ch16, v8, hidden-class, property-attribute]
weight: 9
---

> 프로퍼티 어트리뷰트 챕터 전체는 한 문장으로 요약된다. **프로퍼티에도 우리가 설정할 수 있는 여러 속성이 숨겨져 있다.**

사람들은 자기가 저장한 값을 남이 — 악의로든 실수로든 — 함부로 바꾸지 않기를 바랐어요.

이걸 구현하는 방법은 여러 가지가 있겠죠. 코드 단에서라면 값을 저장한 위치는 숨기고 읽어오는 함수만 바깥에 드러내는 식으로 **캡슐화**할 수도 있을 거예요. 실제로 우리는 그렇게 많이 짜왔고요.

그런데 사람들의 욕심은 끝이 없었어요. "그걸 매번 직접 짜기 귀찮으니, **언어가 직접** 각 프로퍼티(변수) 단위로 이 기능을 제공해줘." 그리고 이건 시작에 불과했죠. "이 프로퍼티는 목록에 나오지 않게 숨겨줘", "이건 아예 못 지우게 잠가줘", "값 대신 읽을 때마다 계산해서 줘"... 끝없이 밀려드는 요구사항을 마주한 **작고 조그마한 js**는, 이걸 대체 어떻게 풀었을까요?

이 글은 그 풀이를 **V8 엔진의 입장**에서, "나라면 이걸 어떻게 메모리에 구현하지?"를 따라가며 정리한 기록이에요.

---

## 1. 꼬리표를 달다

js는 가장 쉽고 단순한 방법을 택했어요. 객체의 각 프로퍼티마다 일종의 **꼬리표(tag)**를 달아둔 거죠.

*"나는 변경 가능해."*
*"나는 목록에 나와도 돼."*
*"내 이 설정들은 절대 못 바꿔."*

이렇게 프로퍼티 하나하나가 자기 꼬리표를 들고 다니면, "이건 읽기 전용, 저건 숨김" 같은 개별 규칙을 값과 따로 관리할 수 있게 돼요. 우리가 평소에 `obj.name`이라고 점 하나 찍어 읽을 때, 그 값 뒤에는 이런 꼬리표들이 조용히 매달려 있던 거예요.

이 꼬리표들에는 진짜 이름이 있어요. js는 프로퍼티 하나에 **네 개의 꼬리표**를 붙여요.

- `value` — 값 그 자체 ("나는 `'기래'`야")
- `writable` — 변경 가능 여부 ("내 값 바꿔도 돼?")
- `enumerable` — 목록 노출 여부 ("나 목록에 나와도 돼?")
- `configurable` — 설정 잠금 여부 ("내 꼬리표들 바꾸거나 날 지워도 돼?")

이 네 꼬리표를 한데 묶어서 **프로퍼티 어트리뷰트(property attribute)**라고 불러요. "어트리뷰트 = 속성", 그러니까 **프로퍼티가 가진 속성**인 거죠. 제목이 외친 "이 값을 변경 불가능하게 해줘"라는 요구는, 결국 `writable` 꼬리표를 끄는 것 하나로 풀려요.

<iframe id="ch16-fig1" src="/demos/ch16-property-tags/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" loading="lazy" title="값에 매달린 꼬리표" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

ECMAScript 스펙은 이 꼬리표들을 `[[Value]]`, `[[Writable]]`, `[[Enumerable]]`, `[[Configurable]]`처럼 이중 대괄호로 적어요. **"엔진 내부에만 존재하는, 우리가 직접 못 만지는 것"**이라는 표식이에요.

---

## 2. 값 말고 동작을 달라 — 또 다른 종류의 꼬리표

요구사항은 또 들어와요. "값을 그냥 저장하지 말고, 읽을 때마다 **계산**해서 주거나, 쓸 때마다 **검증**하게 해줘."

js는 꼬리표 구성을 한 벌 더 만들어요. `value`·`writable` 자리를 **`get`·`set` 함수**로 바꾼 거죠.

- `value` / `writable` 을 가진 프로퍼티 → **데이터 프로퍼티** (값을 직접 저장한다)
- `get` / `set` 을 가진 프로퍼티 → **접근자 프로퍼티** (값을 함수로 가로챈다)

<iframe id="ch16-fig2" src="/demos/ch16-data-vs-accessor/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" loading="lazy" title="데이터 vs 접근자 프로퍼티" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

{{< callout type="info" >}}
**왜 데이터와 접근자는 합쳐질 수 없을까?**
같은 프로퍼티가 "저장된 값"이면서 동시에 "계산된 값"일 수는 없기 때문이에요. `obj.x`를 읽을 때 저장된 값을 줄지, `get()` 함수를 실행할지는 둘 중 하나여야 하죠. 그래서 js는 둘을 **상호 배타적**으로 못박았어요. 한 칸에 저장과 가로채기를 같이 담을 수는 없으니까요.
{{< /callout >}}

뒤쪽 두 꼬리표(`enumerable`·`configurable`)가 양쪽에 공통인 이유도 여기서 나와요. 그건 "값을 어떻게 다루나"와는 무관한, **"이 프로퍼티가 목록에 나오나 / 잠겼나"**라는 별개의 성질이거든요. 값의 정체가 무엇이든 똑같이 필요한 꼬리표라 양쪽이 공유해요.

---

## 3. 그 꼬리표, 어디에 저장하지

여기서부터가 진짜 구현 이야기예요. 꼬리표라는 발상은 좋은데, **이걸 메모리에 어떻게 둘 것이냐**가 남았어요.

순진하게 가면, 프로퍼티마다 값 옆에 꼬리표 4개를 그대로 붙여두면 돼요. 하지만 V8은 두 번 영리하게 굴어요.

**첫째, 값과 꼬리표를 분리한다.**
값(`'기래'`)은 객체 본체에 그대로 두지만, 꼬리표들은 **객체 바깥의 별도 공간**에 모아둬요. 왜 분리하는지는 다음 장에서 드러나요 — 일단은 "값과 꼬리표가 다른 곳에 산다"만 기억하면 돼요.

<iframe id="ch16-fig3" src="/demos/ch16-value-tag-split/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" loading="lazy" title="값과 꼬리표의 분리 저장" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

**둘째, 꼬리표를 통째로 압축한다.**
`writable`, `enumerable`, `configurable`은 결국 켜짐/꺼짐(true/false)이에요. 이런 걸 일일이 별도의 boolean 변수로 두면 자리만 차지하죠. 그래서 V8은 이 꼬리표들을 **정수 하나 안의 몇 개의 비트**로 욱여넣어요. 검사도 비트 연산 한 번이면 끝나고, 메모리도 거의 안 먹어요. 이 압축된 꼬리표 묶음을 V8은 `PropertyDetails`라고 불러요.

그런데 이 압축에는 한 가지 **변태적인 디테일**이 숨어 있어요. "정상 상태"를 비트로 표시할 때, 보통이라면 "정상이면 1"로 둘 것 같죠? V8은 정반대로 갔어요. **정상이 `000`**이에요. 왜 굳이 그랬는지는, 이 글의 마지막에서 이야기할게요. (지금은 떡밥만.)

---

## 4. 그럼 모든 프로퍼티가 꼬리표를 따로 들고 있어야 할까?

잠깐, 여기까지 오니 당연한 의문이 생겨요. 다시 우리 입장으로 돌아가 봅시다.

이 꼬리표들도 결국 **객체마다 메모리 어딘가에 저장**돼야 하잖아요? 그런데 우리는 보통 **같은 모양의 객체를 엄청나게 많이** 만들어요.

```js
{ x: 1, y: 2 }
{ x: 9, y: 8 }
{ x: 3, y: 7 }
// ... 이런 point 객체를 1000개
```

이 1000개는 전부 `x`, `y`라는 똑같은 프로퍼티 구성을 가져요. 그러면 `x`·`y`의 꼬리표(`writable`·`enumerable`·`configurable`)도 1000개 객체에서 **전부 동일**하죠. 그런데 객체를 만들 때마다 이 똑같은 꼬리표 세트를 새로 만들어 붙인다면? **같은 메모를 1000번 베껴 쓰는 엄청난 낭비**예요.

그래서 js는 생각해요.

> **"모양이 같으면 꼬리표도 똑같을 텐데... 그럼 꼬리표를 한 벌만 만들어두고, 같은 모양 객체들이 다 같이 쓰면 되잖아?"**

이 아이디어가 바로 **히든클래스(hidden class)**예요. V8 내부에서는 `Map`이라고 불러요(우리가 아는 `Map` 자료구조랑은 다른, 동음이의어예요).

3장에서 "값과 꼬리표를 왜 분리하지?"라고 떡밥을 던졌었죠. 바로 이걸 위해서예요. 꼬리표 묶음을 객체 본체가 아니라 **히든클래스에 모아두고, 같은 모양의 객체들이 그 히든클래스 하나를 공유**하는 거예요. 그래야 꼬리표가 한 벌만 존재할 수 있으니까요.

<iframe id="ch16-fig5" src="/demos/ch16-hidden-class/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" loading="lazy" title="히든클래스 공유" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

### 모양이 바뀌면?

객체에 프로퍼티를 새로 추가하면 "모양"이 바뀌죠. 그러면 V8은 **새 히든클래스로 갈아타요(transition).**

```js
{}  →  { x }  →  { x, y }
// 빈 모양  x만 있는 모양  x,y 있는 모양
```

이 전이 경로는 캐싱돼서, **같은 순서로 같은 프로퍼티를 추가하는 객체들은 같은 경로를 타고 같은 히든클래스에 도착**해요. 그래서 `{ x, y }`를 만드는 모든 객체가 결국 히든클래스 하나를 공유하게 되는 거예요.

### 그래서 어떻게 쓰면 빨라지나

여기서 실전 교훈이 나와요. **객체의 모양을 일정하게 유지하면** 히든클래스 공유가 잘 일어나서 빠른 길(V8 용어로 *fast properties*)을 타요. 반대로 모양을 흐트러뜨리면 공유가 깨지고, 객체가 자기만의 꼬리표 사전을 들고 다니는 **느린 길(*dictionary mode*)**로 떨어져요.

{{< callout type="warning" >}}
**객체의 모양을 깨뜨리는 대표적인 행동**

- 생성자나 객체마다 프로퍼티 추가 **순서가 제각각**일 때
- 중간 프로퍼티를 `delete`로 지울 때 (전이 경로가 끊어진다)
- `Object.defineProperty`로 꼬리표를 비표준적으로 만질 때

정리하면, **같은 형태의 객체는 같은 순서로 똑같이 빚어내라.** 그게 히든클래스 공유의 입장권이다.
{{< /callout >}}

---

## 5. 왜 `111`이 아니라 `000`이었나 — V8의 집착

이제 3장에서 던진 떡밥을 회수할 차례예요. 압축된 꼬리표에서 "정상 상태"를 V8은 왜 `1`이 아니라 `000`으로 뒀을까요?

만약 "정상이면 1"로 했다면, 평범한 프로퍼티 하나를 만들 때마다 `writable=1`, `enumerable=1`, `configurable=1` 비트를 일일이 **켜주는 작업**을 해야 해요.

그런데 V8은 거꾸로, **정상을 `000`**으로 뒀어요. 예외일 때만 비트를 켜는 거죠. 내부 이름도 부정형이에요 — `READ_ONLY`(쓰기 금지), `DONT_ENUM`(열거 금지), `DONT_DELETE`(삭제 금지). 전부 "~하지 마"라는 금지 플래그예요.

{{< callout type="info" >}}
**왜 `000`이 영리한가**
메모리는 새로 할당하면 어차피 `0`으로 깔려요. `0`이 곧 정상 상태라면, 평범한 프로퍼티를 만들 때 꼬리표를 **쓸 필요조차 없어요.** 메모리를 0으로 미는 순간 이미 "변경 가능 · 목록에 나옴 · 잠기지 않음"이 완성돼 있는 거예요. 그리고 세상 대부분의 프로퍼티는 이 평범한 상태죠. **압도적 다수가 공짜로 처리되는** 거예요.
{{< /callout >}}

<iframe id="ch16-fig4" src="/demos/ch16-bitfield/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" loading="lazy" title="정상이 000인 이유" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

> "예외에만 표식을 단다." 다수를 위한 비용을 0으로 만드는, 구현자의 집착이 담긴 선택이다.

이 정도로 V8은 흔한 경우 하나하나의 비용까지 깎아내요. 프로퍼티 어트리뷰트라는 작은 개념 뒤에, 이런 변태적인 최적화가 깔려 있던 거죠.

### 마지막으로, V8이 객체를 담는 그릇들

지금까지 등장한 조각들을 한자리에 모아볼게요. 우리가 `const o = { x: 1 }` 한 줄을 쓸 때, V8 안에서는 이 그릇들이 조용히 조립돼요.

- **JSObject** — 실제 인스턴스. 값을 인라인으로 품고, 히든클래스·프로퍼티 저장소·elements를 가리킨다.
- **Map(히든클래스)** — 모양 정보. 꼬리표 묶음을 가리키고, 모양이 바뀌면 transition으로 다른 Map과 이어진다. 같은 모양 객체들이 공유한다.
- **DescriptorArray** — 꼬리표 묶음. 프로퍼티별로 키·위치·`PropertyDetails`를 담는다.
- **PropertyDetails** — 그 압축된 비트필드. "데이터냐 접근자냐 + 어트리뷰트 3비트"가 들어 있다.
- **AccessorPair** — 접근자 프로퍼티일 때, 값 자리에 들어앉는 `get`/`set` 함수 쌍.
- **elements** — `arr[0]` 같은 인덱스(숫자 키) 프로퍼티는 여기 따로 저장된다.

<iframe id="ch16-fig6" src="/demos/ch16-object-map/" style="width:100%;border:none;border-radius:12px;min-height:420px;margin:1rem 0;" loading="lazy" title="V8 객체 전체 지도" onload="this.style.height=this.contentWindow.document.body.scrollHeight+16+'px'"></iframe>

---

## 정리

작고 조그마한 js가 끝없는 요구사항을 어떻게 풀었는지, 그 형태를 거꾸로 따라왔어요. 요구사항 → 구현 형태로 한 줄씩 매핑하면 이래요.

| 사람들의 요구 | js의 꼬리표 | V8의 구현 |
|---|---|---|
| "값을 못 바꾸게 해줘" | `writable` 끄기 | `PropertyDetails`의 `READ_ONLY` 비트 |
| "목록에 안 나오게 숨겨줘" | `enumerable` 끄기 | `DONT_ENUM` 비트 |
| "지우거나 재설정 못 하게 잠가줘" | `configurable` 끄기 | `DONT_DELETE` 비트 |
| "값 대신 계산해서 줘" | 접근자 프로퍼티(`get`/`set`) | 값 자리에 `AccessorPair` |
| "이걸 수백만 개 객체에 효율적으로" | (사용자는 모르는 일) | 히든클래스 공유 + 비트필드 압축 |

핵심을 다시 짚으면:

- **프로퍼티는 값만 가진 게 아니다.** 값 뒤에 `writable`·`enumerable`·`configurable`이라는 꼬리표가 함께 매달려 있다.
- **데이터 프로퍼티와 접근자 프로퍼티는 합쳐질 수 없다.** 저장이냐 가로채기냐는 한 칸에 못 담는다.
- **V8은 값과 꼬리표를 분리하고, 꼬리표를 비트로 압축한다.** 그리고 같은 모양 객체끼리 **히든클래스**로 꼬리표를 공유한다.
- **그래서 객체의 모양을 일정하게 유지하면 빨라진다.** 모양을 깨는 `delete`·순서 변경·무분별한 `defineProperty`는 느린 길로 떨어뜨린다.
- **정상을 `000`으로 둔 부정형 인코딩**까지, V8은 흔한 경우의 비용을 0으로 깎아내는 집착을 보인다.

우리가 무심코 쓰는 `obj.x = 1` 한 줄 뒤에는, 이렇게 촘촘하게 짜인 엔진의 설계가 숨어 있었다.

<script>
window.addEventListener('message', function(e){
  if (e.data && e.data.type === 'demo-resize') {
    var frames = document.querySelectorAll('iframe[id^="ch16-fig"]');
    for (var i = 0; i < frames.length; i++) {
      if (frames[i].contentWindow === e.source) {
        frames[i].style.height = e.data.height + 16 + 'px';
      }
    }
  }
});
</script>
