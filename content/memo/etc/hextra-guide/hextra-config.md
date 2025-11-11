---
title: "Hextra Front Matter 가이드"
linkTitle: "Hextra Front Matter"
weight: 2
type: docs
breadcrumbs: true
---


## 개요

Front Matter는 마크다운 파일 상단의 `---`로 구분된 YAML 헤더 영역입니다. 페이지의 메타데이터, 레이아웃, 네비게이션, SEO 등을 제어하는 설정을 정의합니다.

```yaml
---
title: "페이지 제목"
weight: 1
type: docs
---
```

{{< callout type="info" >}}
Front Matter는 반드시 파일의 **맨 첫 줄**부터 시작해야 하며, `---`로 시작하고 `---`로 끝나야 합니다.
{{< /callout >}}

---

## 기본 파라미터

### title (필수)

페이지 제목. 브라우저 탭, 네비게이션, 페이지 헤더에 표시됩니다.

```yaml
title: "시작하기"
```

### linkTitle

네비게이션과 브레드크럼에 사용되는 짧은 제목. 전체 제목이 너무 길 때 유용합니다.

```yaml
title: "Hextra 테마 전체 설치 가이드"
linkTitle: "설치 가이드"  # 네비게이션에는 짧게 표시
```

### weight

사이드바와 섹션 네비게이션의 정렬 순서를 제어합니다.

- **낮은 숫자 = 더 위에 표시** (우선순위 높음)
- weight가 같으면 알파벳순 정렬
- weight가 없으면 맨 뒤에 알파벳순 정렬

```yaml
weight: 1  # 첫 번째 항목
weight: 2  # 두 번째 항목
weight: 10 # 열 번째 항목
```

{{< callout type="warning" >}}
weight는 **같은 디렉토리 내**의 페이지들 간의 순서만 제어합니다.
{{< /callout >}}

### draft

`true`로 설정하면 프로덕션 빌드에서 제외됩니다.

```yaml
draft: true  # 개발 중인 페이지
```

### date

페이지 게시일. 블로그 포스트나 정렬에 사용됩니다.

```yaml
date: 2025-11-11
```

---

## 레이아웃 설정

### type

콘텐츠 타입. 레이아웃 동작을 결정합니다.

**옵션:**
- `docs` - 사이드바가 있는 문서 레이아웃
- `blog` - 블로그 포스트 레이아웃
- `default` - 사이드바 없는 단일 페이지

```yaml
type: docs
```

### layout

사용할 커스텀 레이아웃 템플릿.

```yaml
layout: hextra-home  # 랜딩 페이지용
```

### cascade

하위 페이지들에게 설정을 전파합니다. 주로 `_index.md`(섹션 인덱스)에서 사용합니다.

```yaml
cascade:
  type: docs  # 모든 하위 페이지가 docs 타입으로 설정됨
  params:
    breadcrumbs: true
```

---

## 네비게이션

### prev / next

페이지 하단의 이전/다음 링크를 설정합니다.

```yaml
prev: /docs/getting-started
next: /docs/guide/organize-files
```

{{< callout type="info" >}}
설정하지 않으면 Hugo가 자동으로 weight와 디렉토리 구조를 기반으로 결정합니다.
{{< /callout >}}

---

## 사이드바 제어

### sidebar.exclude

사이드바 네비게이션에서 페이지를 숨깁니다.

```yaml
sidebar:
  exclude: true  # 사이드바에 표시 안 함
```

**사용 예시:**
- 숨겨진 페이지
- 특정 URL로만 접근하는 페이지
- 네비게이션에 노출하고 싶지 않은 유틸리티 페이지

### sidebar.hide

해당 페이지에서 사이드바를 완전히 숨깁니다.

```yaml
sidebar:
  hide: true  # 사이드바 전체를 표시하지 않음
```

### sidebar.open

사이드바 섹션을 기본적으로 펼쳐진 상태로 표시합니다.

```yaml
sidebar:
  open: true  # 섹션이 펼쳐진 상태로 시작
```

---

## 목차(TOC) 설정

### toc

오른쪽 사이드바의 목차 표시 여부를 제어합니다.

```yaml
toc: false  # 이 페이지에는 목차 표시 안 함
```

**기본값:** `true`

**목차가 불필요한 경우:**
- 헤딩이 거의 없는 짧은 페이지
- 랜딩 페이지
- 커스텀 레이아웃 페이지

---

## 브레드크럼

### breadcrumbs

브레드크럼 네비게이션 표시 여부.

```yaml
breadcrumbs: false  # 브레드크럼 숨김
```

---

## 검색

### excludeSearch

FlexSearch 인덱스에서 페이지를 제외합니다.

```yaml
excludeSearch: true  # 검색 결과에 표시 안 함
```

**사용 예시:**
- 임시 페이지
- 내부 문서
- 중복 콘텐츠

---

## 특수 기능

### math

LaTeX/KaTeX 수식 렌더링을 활성화합니다.

```yaml
math: true
```

**사용 후:**
```markdown
인라인 수식: $E = mc^2$

블록 수식:
$$
\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
$$
```

### comments

댓글 시스템(예: giscus) 활성화 여부.

```yaml
comments: true  # 이 페이지에 댓글 활성화
```

{{< callout type="info" >}}
사이트 전체 설정(`hugo.yaml`)에서 댓글 시스템이 설정되어 있어야 작동합니다.
{{< /callout >}}

---

## 메타데이터 & SEO

### tags

페이지 태그. 설정에서 활성화된 경우 표시됩니다.

```yaml
tags:
  - Documentation
  - Guide
  - Tutorial
```

### editURL

이 페이지의 커스텀 편집 링크.

```yaml
editURL: "https://github.com/user/repo/edit/main/content/page.md"
```

### params.noindex

구글 검색 인덱싱을 차단합니다(`noindex` 메타 태그).

```yaml
params:
  noindex: true  # 검색 엔진에 노출 안 함
```

### params.images

Open Graph 이미지. 소셜 미디어 공유 시 표시됩니다.

```yaml
params:
  images:
    - "/images/og-image.jpg"
```

### params.audio / params.videos

Open Graph 오디오/비디오.

```yaml
params:
  audio: "/podcast.mp3"
  videos:
    - "/video.mp4"
```

---

## 실전 예제

### 최소 구성 (필수만)

대부분의 페이지에 필요한 최소 설정:

```yaml
---
title: "페이지 제목"
weight: 1
---
```

### 일반적인 문서 페이지

```yaml
---
title: "Hextra 컴포넌트 가이드"
linkTitle: "컴포넌트"
weight: 1
type: docs
toc: true
---
```

### 블로그 포스트

```yaml
---
title: "새로운 기능 출시"
date: 2025-11-11
type: blog
tags:
  - Release
  - Features
comments: true
---
```

### 랜딩 페이지

```yaml
---
title: "홈"
layout: hextra-home
type: default
sidebar:
  hide: true
toc: false
breadcrumbs: false
---
```

### 수학 수식이 포함된 기술 문서

```yaml
---
title: "선형 대수 개론"
weight: 3
type: docs
math: true
tags:
  - Mathematics
  - Linear Algebra
---
```

### 검색에서 제외되는 내부 문서

```yaml
---
title: "내부 개발 가이드"
draft: false
excludeSearch: true
sidebar:
  exclude: true
params:
  noindex: true
---
```

### 섹션 인덱스 (_index.md)

하위 페이지들에게 설정을 전파하는 섹션 인덱스:

```yaml
---
title: "개발 가이드"
weight: 2
type: docs
cascade:
  type: docs
  params:
    breadcrumbs: true
    editURL: "https://github.com/user/repo/edit/main/content/{path}"
---
```

### 전체 옵션 예제

가능한 모든 옵션을 포함한 예제:

```yaml
---
# 기본
title: "완전한 Front Matter 예제"
linkTitle: "Front Matter"
weight: 2
draft: false
date: 2025-11-11

# 레이아웃
type: docs
layout: default

# 네비게이션
prev: /docs/previous-page
next: /docs/next-page

# 사이드바
sidebar:
  exclude: false
  hide: false
  open: true

# 표시
toc: true
breadcrumbs: true

# 메타데이터
tags:
  - Documentation
  - Reference
  - YAML

# 특수 기능
math: true
comments: true

# SEO
editURL: "https://github.com/user/repo/edit/main/content/page.md"
excludeSearch: false

# Open Graph
params:
  noindex: false
  images:
    - "/images/page-cover.jpg"
  audio: "/podcast.mp3"
  videos:
    - "/demo-video.mp4"

# Cascade (섹션 인덱스용)
cascade:
  type: docs
  params:
    breadcrumbs: true
---
```

---

## 참고사항

{{< callout type="warning" >}}
**YAML 문법 주의사항:**
- 들여쓰기는 **2칸 스페이스** 사용 (탭 사용 금지)
- 문자열에 특수문자(`:`, `#`, `-` 등)가 있으면 따옴표로 감싸기
- Boolean 값은 `true`/`false` (대소문자 구분 없음)
- 날짜 형식: `YYYY-MM-DD` 또는 `YYYY-MM-DD HH:MM:SS`
{{< /callout >}}

{{< callout type="info" >}}
**Front Matter 디버깅:**
1. YAML 문법 오류는 빌드 시 에러 메시지로 표시됩니다
2. `hugo server` 명령으로 로컬에서 먼저 테스트하세요
3. YAML 유효성 검사 도구: [yamllint.com](https://www.yamllint.com/)
{{< /callout >}}

---

## 더 알아보기

- [Hugo Front Matter 공식 문서](https://gohugo.io/content-management/front-matter/)
- [Hextra 테마 설정 가이드](/docs/configuration)
- [페이지 구성 가이드](/docs/organize-files)
