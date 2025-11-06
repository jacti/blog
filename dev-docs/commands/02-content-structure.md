# 콘텐츠 구조 템플릿 스니펫

**출처**: Hextra 공식 문서 (2025년 최신)
**적용 대상**: `content/` 디렉토리 구조 및 `_index.md` 파일

---

## 1. 디렉토리 구조 개요

```
content/
├── _index.md                    # 홈페이지 (/)
├── learning-log/
│   ├── _index.md               # Learning Log 섹션 홈
│   ├── ai/
│   │   ├── _index.md
│   │   └── machine-learning.md
│   └── web/
│       ├── _index.md
│       └── react-hooks.md
├── debug-notes/
│   ├── _index.md               # Debug Notes 섹션 홈
│   └── troubleshooting-001.md
├── playground/
│   ├── _index.md               # Playground 섹션 홈
│   └── demo-project-1.md
├── about/
│   └── _index.md               # About 페이지
└── workbench/
    ├── _index.md               # Workbench (초안 보관)
    └── draft-post.md
```

---

## 2. Learning Log 섹션 템플릿

### `content/learning-log/_index.md`

```yaml
---
title: Learning Log
type: docs
sidebar:
  open: true
---

AI와 함께 정리한 학습 노트를 위키 형식으로 제공합니다.

왼쪽 사이드바에서 카테고리를 탐색할 수 있습니다.
```

### `content/learning-log/ai/_index.md`

```yaml
---
title: AI & Machine Learning
weight: 1
---

인공지능과 머신러닝 관련 학습 내용을 정리합니다.
```

### `content/learning-log/ai/machine-learning.md`

```yaml
---
title: 머신러닝 기초
date: 2025-01-15
weight: 1
---

## 머신러닝이란?

머신러닝의 기본 개념과 종류를 정리합니다.

### 지도 학습

...

### 비지도 학습

...
```

---

## 3. Debug Notes 섹션 템플릿

### `content/debug-notes/_index.md`

```yaml
---
title: Debug Notes
type: docs
sidebar:
  open: true
---

실제 겪었던 트러블슈팅 사례를 스토리 형식으로 기록합니다.

각 노트는 문제 상황, 해결 과정, 배운 점을 포함합니다.
```

### `content/debug-notes/troubleshooting-001.md`

```yaml
---
title: "Hugo 빌드 오류 해결기"
date: 2025-01-10
tags: ["hugo", "build", "troubleshooting"]
---

## 문제 상황

Hugo 빌드 시 `cards` 숏코드를 찾을 수 없다는 오류 발생

## 해결 과정

1. 테마 설치 상태 확인
2. Git Submodule 동기화
3. 캐시 정리

## 배운 점

...
```

---

## 4. Playground 섹션 템플릿

### `content/playground/_index.md`

```yaml
---
title: Playground
layout: hextra-home
---

{{< cards >}}
  {{< card link="demo-project-1" title="Demo Project 1" icon="code" subtitle="React 기반 인터랙티브 데모" >}}
  {{< card link="demo-project-2" title="Demo Project 2" icon="sparkles" subtitle="Three.js 3D 시각화" >}}
  {{< card link="demo-project-3" title="Demo Project 3" icon="chart-bar" subtitle="D3.js 데이터 차트" >}}
{{< /cards >}}
```

### `content/playground/demo-project-1.md`

```yaml
---
title: Demo Project 1
date: 2025-01-20
tags: ["react", "demo"]
---

## 프로젝트 설명

React로 만든 인터랙티브 데모입니다.

## 라이브 데모

<iframe src="/demos/project-1/index.html" width="100%" height="600px"></iframe>

## 소스 코드

GitHub: [링크](https://github.com/...)
```

---

## 5. About 페이지 템플릿

### `content/about/_index.md`

```yaml
---
title: About
breadcrumbs: true
toc: false
---

## 안녕하세요!

AI와 함께 개발하고 배우는 과정을 기록하는 블로그입니다.

## 경력

- 2023 ~ 현재: ...
- 2020 ~ 2023: ...

## 기술 스택

{{< cards >}}
  {{< card title="Frontend" icon="code" >}}
  React, Vue.js, TypeScript
  {{< /card >}}

  {{< card title="Backend" icon="server" >}}
  Node.js, Python, Go
  {{< /card >}}

  {{< card title="Tools" icon="tool" >}}
  Hugo, Git, Docker
  {{< /card >}}
{{< /cards >}}

## 연락처

- GitHub: [링크](https://github.com/...)
- Email: your@email.com
```

---

## 6. Workbench (초안) 섹션 템플릿

### `content/workbench/_index.md`

```yaml
---
title: Workbench
type: docs
draft: true
---

작성 중인 초안과 실험적 콘텐츠를 보관합니다.

이 섹션은 `hugo --buildDrafts` 옵션 사용 시에만 표시됩니다.
```

### `content/workbench/draft-post.md`

```yaml
---
title: 작성 중인 포스트
draft: true
date: 2025-01-25
---

이 글은 아직 작성 중입니다.
```

---

## 7. 홈페이지 템플릿

### `content/_index.md`

```yaml
---
title: jacti-log
layout: hextra-home
---

{{< hextra/hero-headline >}}
  AI와 함께하는&nbsp;<br class="sm:block hidden" />개발 여정
{{< /hextra/hero-headline >}}

{{< hextra/hero-subtitle >}}
  학습 노트, 트러블슈팅, 그리고 데모 프로젝트를 기록합니다.
{{< /hextra/hero-subtitle >}}

{{< hextra/hero-button text="시작하기" link="learning-log" >}}
{{< hextra/hero-button text="데모 보기" link="playground" style="secondary" >}}

## 주요 섹션

{{< cards >}}
  {{< card link="learning-log" title="Learning Log" icon="book-open" >}}
  AI와 함께 정리한 학습 노트
  {{< /card >}}

  {{< card link="debug-notes" title="Debug Notes" icon="bug" >}}
  실전 트러블슈팅 사례
  {{< /card >}}

  {{< card link="playground" title="Playground" icon="beaker" >}}
  AI 제작 데모 프로젝트
  {{< /card >}}
{{< /cards >}}
```

---

## 8. Front Matter 파라미터 설명

### 공통 파라미터

```yaml
---
title: "페이지 제목"           # 필수
date: 2025-01-15             # 날짜 (선택)
weight: 1                    # 사이드바 순서 (낮을수록 위)
draft: false                 # 초안 여부
toc: true                    # 목차 표시 (기본: true)
breadcrumbs: true            # 브레드크럼 표시
tags: ["tag1", "tag2"]       # 태그
---
```

### Docs 레이아웃 전용

```yaml
---
type: docs                   # Docs 레이아웃 사용
sidebar:
  open: true                 # 사이드바 기본 열림
  exclude: false             # 사이드바에서 제외 (기본: false)
cascade:
  type: docs                 # 하위 페이지에 type 상속
---
```

### 블로그/리스트 레이아웃 전용

```yaml
---
cascade:
  params:
    reversePagination: false  # 역순 페이지네이션 비활성화
---
```

---

## 9. 사용법 안내

### 섹션 생성 순서

1. 디렉토리 생성: `mkdir -p content/learning-log`
2. `_index.md` 작성 (섹션 홈)
3. 하위 카테고리 또는 포스트 작성
4. `weight` 값으로 순서 조정

### Docs vs Blog 레이아웃 선택

**Docs 레이아웃** (Learning Log, Debug Notes):
- `type: docs` 설정
- 좌측 사이드바 + 우측 TOC
- 위키형 콘텐츠에 적합

**일반 레이아웃** (Playground):
- `layout: hextra-home` 또는 기본 레이아웃
- 카드 그리드 형식
- 갤러리/포트폴리오에 적합

### 검증 방법

```bash
# 콘텐츠 생성
hugo new content/learning-log/new-post.md

# 로컬 서버 실행 (초안 포함)
hugo server -D

# 프로덕션 빌드 (초안 제외)
hugo --minify
```
