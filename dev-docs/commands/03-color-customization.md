# 색상 커스터마이징 스니펫 (Tailwind CSS v4+)

**출처**: Hextra 공식 문서 (2025년 최신)
**적용 대상**: `assets/css/custom.css` (Tailwind CSS v4 기반)

---

## ⚠️ 중요: Hextra v0.10.0+ 변경 사항

Hextra는 **Tailwind CSS v4+**로 마이그레이션되었습니다:

- ❌ **SCSS/SASS 사용 불가**
- ✅ **CSS 변수 기반 커스터마이징** (PostCSS)
- ✅ **Tailwind 유틸리티 클래스 접두어**: `hx:` (이전: `hx-`)

---

## 1. 기본 설정: custom.css 생성

### 파일 경로
```
assets/
└── css/
    └── custom.css  # 이 파일을 생성
```

### 기본 템플릿

```css
/* assets/css/custom.css */

/* ===========================
   Tailwind CSS v4 Theme Layer
   =========================== */

@layer theme {
  :root {
    /* 기본 변수 커스터마이징 */
    --hx-default-mono-font-family: "JetBrains Mono", monospace;
  }
}

/* ===========================
   Primary Color Customization
   =========================== */

:root {
  /* HSL 색상 체계 (Hue, Saturation, Lightness) */
  --primary-hue: 210deg;         /* 색상 (0-360deg) */
  --primary-saturation: 90%;     /* 채도 (0-100%) */
  --primary-lightness: 50%;      /* 명도 (0-100%) */
}

/* ===========================
   Element Customization
   =========================== */

/* 푸터 커스터마이징 */
.hextra-footer {
  background-color: #f9fafb;
  border-top: 1px solid #e5e7eb;
}

.hextra-footer:is(html[class~="dark"] *) {
  background-color: #1f2937;
  border-top: 1px solid #374151;
}

/* 인라인 코드 색상 */
.content code:not(.code-block code) {
  color: #c97c2e;
}
```

---

## 2. 색상 팔레트 예시

### 슬레이트 + 틸 (Slate + Teal)

```css
:root {
  /* Primary: Teal */
  --primary-hue: 173deg;
  --primary-saturation: 80%;
  --primary-lightness: 40%;
}
```

### 웜그레이 + 앰버 (Warm Gray + Amber)

```css
:root {
  /* Primary: Amber */
  --primary-hue: 38deg;
  --primary-saturation: 92%;
  --primary-lightness: 50%;
}
```

### 인디고 + 시아 (Indigo + Cyan)

```css
:root {
  /* Primary: Indigo */
  --primary-hue: 239deg;
  --primary-saturation: 84%;
  --primary-lightness: 67%;
}
```

### 진한 남청 (Hextra 기본)

```css
:root {
  /* Primary: Blue */
  --primary-hue: 210deg;
  --primary-saturation: 90%;
  --primary-lightness: 50%;
}
```

---

## 3. 다크 모드 커스터마이징

```css
/* 라이트 모드 */
:root {
  --color-background: #ffffff;
  --color-text: #1f2937;
  --color-border: #e5e7eb;
}

/* 다크 모드 */
:root:is(html[class~="dark"] *) {
  --color-background: #111827;
  --color-text: #f9fafb;
  --color-border: #374151;
}

/* 적용 예시 */
body {
  background-color: var(--color-background);
  color: var(--color-text);
}
```

---

## 4. Tailwind 유틸리티 클래스 사용 (v4 변경사항)

### ❌ 이전 방식 (Tailwind v3)

```css
.hx-flex {
  display: flex;
}
```

### ✅ 새로운 방식 (Tailwind v4)

```css
.hx\:flex {
  display: flex;
}
```

### HTML에서 사용

```html
<!-- Tailwind 유틸리티 클래스 사용 시 -->
<div class="hx:flex hx:items-center hx:gap-4">
  ...
</div>
```

---

## 5. 컴포넌트별 커스터마이징

### 네비게이션 바

```css
.hextra-navbar {
  background-color: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
}

.hextra-navbar:is(html[class~="dark"] *) {
  background-color: rgba(17, 24, 39, 0.95);
}
```

### 사이드바

```css
.hextra-sidebar {
  border-right: 1px solid #e5e7eb;
}

.hextra-sidebar:is(html[class~="dark"] *) {
  border-right: 1px solid #374151;
}
```

### 목차 (TOC)

```css
.hextra-toc a {
  color: #6b7280;
}

.hextra-toc a:hover {
  color: var(--primary-color);
}
```

### 코드 블록

```css
.code-block {
  background-color: #f9fafb;
  border-radius: 0.5rem;
  padding: 1rem;
}

.code-block:is(html[class~="dark"] *) {
  background-color: #1f2937;
}
```

---

## 6. 폰트 커스터마이징

### Google Fonts 사용

```css
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&family=JetBrains+Mono:wght@400;700&display=swap');

@layer theme {
  :root {
    /* 본문 폰트 */
    --hx-default-font-family: "Noto Sans KR", -apple-system, BlinkMacSystemFont, sans-serif;

    /* 코드 폰트 */
    --hx-default-mono-font-family: "JetBrains Mono", "Courier New", monospace;
  }
}
```

### 로컬 폰트 사용

```css
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/CustomFont.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
}

@layer theme {
  :root {
    --hx-default-font-family: "CustomFont", sans-serif;
  }
}
```

---

## 7. 완전한 custom.css 예시 (jacti-log 프로젝트)

```css
/* assets/css/custom.css */

/* 폰트 임포트 */
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&family=JetBrains+Mono:wght@400;700&display=swap');

/* ===========================
   Tailwind Theme Variables
   =========================== */

@layer theme {
  :root {
    /* 폰트 설정 */
    --hx-default-font-family: "Noto Sans KR", -apple-system, sans-serif;
    --hx-default-mono-font-family: "JetBrains Mono", monospace;
  }
}

/* ===========================
   Primary Color (Teal)
   =========================== */

:root {
  --primary-hue: 173deg;
  --primary-saturation: 80%;
  --primary-lightness: 40%;
}

/* ===========================
   Light/Dark Mode Colors
   =========================== */

:root {
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f9fafb;
  --color-text-primary: #1f2937;
  --color-text-secondary: #6b7280;
  --color-border: #e5e7eb;
}

:root:is(html[class~="dark"] *) {
  --color-bg-primary: #111827;
  --color-bg-secondary: #1f2937;
  --color-text-primary: #f9fafb;
  --color-text-secondary: #9ca3af;
  --color-border: #374151;
}

/* ===========================
   Component Customization
   =========================== */

/* 네비게이션 바 */
.hextra-navbar {
  background-color: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid var(--color-border);
}

.hextra-navbar:is(html[class~="dark"] *) {
  background-color: rgba(17, 24, 39, 0.95);
}

/* 푸터 */
.hextra-footer {
  background-color: var(--color-bg-secondary);
  border-top: 1px solid var(--color-border);
  padding: 2rem 0;
}

/* 인라인 코드 */
.content code:not(.code-block code) {
  color: #c97c2e;
  background-color: rgba(201, 124, 46, 0.1);
  padding: 0.2em 0.4em;
  border-radius: 0.25rem;
  font-size: 0.9em;
}

/* 코드 블록 */
.code-block {
  background-color: var(--color-bg-secondary);
  border: 1px solid var(--color-border);
  border-radius: 0.5rem;
}

/* 링크 호버 효과 */
.content a:hover {
  text-decoration: underline;
  color: hsl(var(--primary-hue), var(--primary-saturation), var(--primary-lightness));
}

/* 사이드바 액티브 항목 */
.hextra-sidebar .active {
  color: hsl(var(--primary-hue), var(--primary-saturation), var(--primary-lightness));
  font-weight: 600;
}
```

---

## 8. 사용법 안내

### 적용 순서

1. `assets/css/custom.css` 파일 생성
2. 위 예시 코드 복사 및 수정
3. `hugo server -D` 로컬 테스트
4. 브라우저에서 색상 및 스타일 확인
5. 다크/라이트 모드 토글 후 재확인

### 주의사항

- ❌ **SCSS 파일 사용 금지** (`.scss` → `.css`)
- ✅ **CSS 변수** 사용 권장
- ✅ **Tailwind 유틸리티 클래스**는 `hx:` 접두어 사용
- ✅ **다크 모드**는 `:is(html[class~="dark"] *)` 선택자 사용

### 검증 방법

```bash
# 로컬 서버 실행
hugo server -D --disableFastRender

# 브라우저 개발자 도구에서 확인:
# 1. Elements 탭에서 CSS 변수 값 확인
# 2. Network 탭에서 custom.css 로드 확인
# 3. 다크 모드 토글 후 스타일 변경 확인
```

### 디버깅

```bash
# Hugo가 CSS를 인식하는지 확인
hugo --verbose

# CSS 컴파일 오류 확인
hugo server -D --logLevel=debug
```

---

## 9. 색상 선택 가이드

### HSL 색상 체계 이해

- **Hue (색상)**: 0-360도 (0=빨강, 120=초록, 240=파랑)
- **Saturation (채도)**: 0-100% (0=회색, 100=순색)
- **Lightness (명도)**: 0-100% (0=검정, 50=기본, 100=흰색)

### 색상 조합 추천

| 팔레트 | Hue | Saturation | Lightness | 특징 |
|--------|-----|------------|-----------|------|
| Teal | 173deg | 80% | 40% | 전문적, 차분함 |
| Amber | 38deg | 92% | 50% | 따뜻함, 활기참 |
| Indigo | 239deg | 84% | 67% | 모던함, 기술적 |
| Blue | 210deg | 90% | 50% | 신뢰감, 기본 |

### 온라인 도구

- [Coolors](https://coolors.co/): 색상 팔레트 생성기
- [Tailwind Color Generator](https://uicolors.app/): Tailwind 팔레트 생성
- [HSL Color Picker](https://hslpicker.com/): HSL 값 확인
