---
title: Custom Partial & Advanced Components
linkTitle: Custom & Advanced
weight: 3
type: docs
---

Hextra 테마를 확장한 커스텀 컴포넌트와 고급 기능들을 정리한 문서입니다.

## 빠른 참조

| 컴포넌트 | 용도 | 타입 |
|---------|------|------|
| [`recent-cards`](#recent-cards-최근-페이지-카드) | 최근 페이지 카드 표시 | 커스텀 |
| [`landing`](#landing-랜딩-페이지) | 랜딩 페이지 래퍼 | 커스텀 |
| [`include`](#include-콘텐츠-포함) | 다른 파일 포함 | Hextra |
| [`hero-container`](#hero-container-이미지-히어로) | 이미지 포함 히어로 | Hextra |
| [`hero-section`](#hero-section-히어로-섹션) | 히어로 섹션 헤딩 | Hextra |
| [`figure`](#figure-이미지--캡션) | 이미지 + 캡션 | Hugo |
| [`gist`](#gist-github-gist) | GitHub Gist 임베드 | Hugo |
| [Video Embeds](#비디오-임베드) | 비디오 임베드 | Hugo |

---

## 커스텀 컴포넌트

### Recent Cards (최근 페이지 카드)

최근 작성된 페이지들을 카드 그리드로 표시하는 커스텀 shortcode입니다.

**사용법:**

```markdown
{{</* recent-cards limit="5" cols="2" */>}}
```

**매개변수:**
- `limit` - 표시할 페이지 수 (기본값: 5)
- `cols` - 그리드 열 개수 (기본값: 2)

**실제 예시:**

{{< recent-cards limit="4" cols="2" >}}

**참고:**
- 현재 섹션의 페이지들만 표시됩니다
- 날짜 역순으로 정렬 (최신순)
- 각 카드에는 제목, 요약(100자), document-text 아이콘이 표시됩니다
- 페이지가 없으면 "아직 작성된 글이 없습니다" 메시지 표시

---

## Advanced Hextra 컴포넌트

### Include (콘텐츠 포함)

다른 페이지의 콘텐츠를 현재 페이지에 포함합니다.

**사용법:**

```markdown
{{%/* include "path/to/page" */%}}
```

{{< details title="예시" open=false >}}
  {{% include "memo/etc/hextra-guide/_example-callout" %}}
{{< /details >}}


**참고:**
- **반드시** `{{%/* */%}}` (퍼센트 기호) 사용
- 경로는 `content/` 디렉토리 기준 상대 경로
- `.md` 확장자는 생략
- 포함된 페이지의 shortcode도 렌더링됨
- `_`로 시작하는 파일은 Hugo가 독립 페이지로 생성하지 않음 (include 전용)

---

### Hero Container (이미지 히어로)

이미지와 콘텐츠를 그리드로 배치하는 히어로 컨테이너입니다.

**기본 사용법:**

```markdown
{{</* hextra/hero-container image="image.png" */>}}
  ## 제목

  설명 텍스트
{{</* /hextra/hero-container */>}}
```

**실제 예시:**

{{< hextra/hero-container
  image="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQmG0sTTujYLYv0fIIepY8CGiS6fY6rlvNlTA&s"
  imageTitle="Hextra Logo"
  imageCard="true"
  imageLink="https://themes.gohugo.io/themes/hextra/"
  imageWidth="400"
  imageHeight="400"
>}}

{{< hextra/hero-headline >}}
  Hextra로 시작하는&nbsp;<br class="hx:sm:block hx:hidden" />모던 문서 사이트
{{< /hextra/hero-headline >}}

{{< hextra/hero-subtitle >}}
  빠르고 강력한 Hugo 테마로&nbsp;<br class="hx:sm:block hx:hidden" />아름답고 반응형인 정적 웹사이트를 만들어보세요
{{< /hextra/hero-subtitle >}}

{{< hextra/hero-button text="시작하기" link="https://imfing.github.io/hextra/docs" >}}

{{< /hextra/hero-container >}}

**매개변수:**
- `image` - 이미지 경로 (필수)
- `imageLink` - 이미지 클릭 시 이동할 링크
- `imageTitle` - 이미지 alt 텍스트
- `imageCard` - 카드 스타일로 표시 (기본값: false)
- `imageWidth` - 이미지 너비 (기본값: 350)
- `imageHeight` - 이미지 높이 (기본값: 350)
- `cols` - 그리드 열 개수 (기본값: 2)
- `class` - 커스텀 CSS 클래스
- `style` - 인라인 스타일

**레이아웃:**
- 왼쪽: 콘텐츠
- 오른쪽: 이미지
- 반응형: 모바일에서는 세로 스택

---

### Hero Section (히어로 섹션)

그라디언트 스타일의 히어로 섹션 헤딩입니다.

**사용법:**

```markdown
{{</* hextra/hero-section heading="h2" */>}}
  섹션 제목
{{</* /hextra/hero-section */>}}
```

**실제 예시:**

{{< hextra/hero-section heading="h2" >}}
  Custom & Advanced Components
{{< /hextra/hero-section >}}

{{< hextra/hero-section heading="h3" >}}
  더 작은 헤딩
{{< /hextra/hero-section >}}

**매개변수:**
- `heading` - 헤딩 레벨 (h1-h6, 기본값: h2)
- `style` - 커스텀 인라인 스타일

**자동 스타일:**
- h1-h2: `text-4xl` (큰 사이즈)
- h3: `text-2xl` (중간 사이즈)
- h4-h6: `text-xl` (작은 사이즈)
- 그라디언트 텍스트 (라이트: gray-900→gray-600, 다크: gray-100→gray-400)
- Bold, tracking-tighter

---

## Hugo Built-in 컴포넌트

### Figure (이미지 + 캡션)

이미지에 캡션과 링크를 추가할 수 있는 Hugo 기본 shortcode입니다.

**기본 사용법:**

```markdown
{{</* figure src="/images/example.jpg" alt="예시 이미지" */>}}
```

**캡션 포함:**

```markdown
{{</* figure
  src="/images/example.jpg"
  alt="예시 이미지"
  caption="이것은 캡션입니다"
  attr="- 출처"
  attrlink="https://example.com"
*/>}}
```

**실제 예시:**

{{< figure
  src="https://via.placeholder.com/800x400"
  alt="Placeholder Image"
  caption="Figure shortcode로 삽입한 이미지"
  attr="- Placeholder.com"
  attrlink="https://placeholder.com"
>}}

**매개변수:**
- `src` - 이미지 경로 (필수)
- `alt` - 대체 텍스트
- `title` - 이미지 타이틀 (호버 시 표시)
- `caption` - 캡션 텍스트
- `attr` - 속성 텍스트 (출처 등)
- `attrlink` - 속성 텍스트 링크
- `link` - 이미지 클릭 시 이동할 링크
- `target` - 링크 target 속성
- `rel` - 링크 rel 속성
- `class` - 커스텀 CSS 클래스
- `height` - 이미지 높이
- `width` - 이미지 너비
- `loading` - 로딩 전략 (lazy, eager)

---

### Gist (GitHub Gist)

GitHub Gist를 페이지에 임베드합니다.

**사용법:**

```markdown
{{</* gist username gist-id */>}}
```

**특정 파일 지정:**

```markdown
{{</* gist username gist-id filename */>}}
```

**예시:**

{{< gist spf13 7896402 >}}
{{< gist spf13 7896402 "img.html" >}}


**참고:**
- `username`: GitHub 사용자명
- `gist-id`: Gist ID (URL에서 확인 가능)
- `filename`: (선택) 특정 파일만 표시
- JavaScript로 로드되므로 네트워크 연결 필요

---

### 비디오 임베드

Hugo에서 기본 제공하는 비디오 임베드 shortcode들입니다.

#### Vimeo

```markdown
{{</* vimeo 146022717 */>}}
```

**매개변수:**
- 첫 번째 인자: Vimeo 비디오 ID
- `class`: 커스텀 CSS 클래스
- `title`: 비디오 제목

#### Instagram

```markdown
{{</* instagram BWNjjyYFxVx */>}}
```

**매개변수:**
- 첫 번째 인자: Instagram 포스트 ID
- `hidecaption`: 캡션 숨김 (true/false)

#### Tweet (Twitter/X)

```markdown
{{</* tweet user="SanDiegoZoo" id="1453110110599868418" */>}}
```

**매개변수:**
- `user`: Twitter 사용자명
- `id`: 트윗 ID (URL에서 확인)

**참고:**
- 모든 비디오 임베드는 반응형으로 자동 렌더링
- 외부 서비스 사용으로 네트워크 연결 필요
- GDPR 준수를 위해 privacy-enhanced 모드 사용 고려

---

## Front Matter 변형

### Wide Layout (넓은 레이아웃)

사이드바 없이 넓은 레이아웃을 사용합니다.

```yaml
---
title: "넓은 페이지"
type: docs
layout: wide
---
```

**특징:**
- TOC와 Sidebar 모두 숨김
- 콘텐츠 영역이 전체 너비 사용
- 대시보드, 테이블, 이미지 갤러리 등에 적합

---

### No Sidebar (사이드바 숨김)

사이드바만 숨기고 TOC는 유지합니다.

```yaml
---
title: "사이드바 없는 페이지"
type: docs
sidebar:
  exclude: true
---
```

**또는 완전히 숨기기:**

```yaml
---
title: "사이드바 완전 숨김"
type: docs
sidebar:
  hide: true
---
```

**차이점:**
- `exclude: true` - 네비게이션에서만 숨김 (페이지는 존재)
- `hide: true` - 해당 페이지에서 사이드바 UI 자체를 숨김

---

### No TOC (목차 숨김)

오른쪽 TOC를 숨깁니다.

```yaml
---
title: "목차 없는 페이지"
type: docs
toc: false
---
```

**사용 예시:**
- 짧은 페이지
- 랜딩 페이지
- 헤딩이 거의 없는 페이지

---

### Combined Layout (조합 레이아웃)

여러 옵션을 조합할 수 있습니다.

```yaml
---
title: "커스텀 레이아웃"
type: docs
sidebar:
  hide: true
toc: false
breadcrumbs: false
---
```

**결과:**
- 사이드바 숨김
- TOC 숨김
- 브레드크럼 숨김
- 콘텐츠만 표시되는 깔끔한 레이아웃

---

## 사용 팁

### 커스텀 컴포넌트 활용

**recent-cards로 인덱스 페이지 만들기:**

```markdown
---
title: "블로그"
type: docs
---

최근 포스트를 확인하세요.

{{</* recent-cards limit="6" cols="3" */>}}
```

### Hero 컴포넌트 조합

**랜딩 페이지 구성:**

```markdown
{{< hextra/hero-section heading="h1" >}}
  Welcome to My Blog
{{< /hextra/hero-section >}}

{{< hextra/hero-container image="/logo.png" imageCard=true >}}
  ## About

  내용 작성
{{< /hextra/hero-container >}}

{{< hextra/feature-grid >}}
  {{< hextra/feature-card title="기능 1" icon="sparkles" >}}
  {{< hextra/feature-card title="기능 2" icon="cube" >}}
{{< /hextra/feature-grid >}}
```

### 콘텐츠 재사용

**공통 FAQ 포함:**

```markdown
## 자주 묻는 질문

{{%/* include "common/faq" */%}}
```

---

## 참고 자료

- [Hextra 공식 문서](https://imfing.github.io/hextra/)
- [Hugo Shortcodes](https://gohugo.io/content-management/shortcodes/)
- [Hugo Figure Shortcode](https://gohugo.io/content-management/shortcodes/#figure)

---

**마지막 업데이트:** 2025-11-11
