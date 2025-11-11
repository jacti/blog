# 다국어 설정 스니펫 (Multilingual Setup)

**출처**: Hextra 공식 문서 (2025년 최신)
**적용 대상**: `hugo.yaml` 설정 및 `i18n/` 번역 파일

---

## 1. hugo.yaml 다국어 기본 설정

```yaml
# 기본 언어 설정
defaultContentLanguage: ko

# 지원 언어 정의
languages:
  ko:
    languageName: 한국어
    weight: 1
  en:
    languageName: English
    weight: 2
```

---

## 2. 메뉴 다국어 지원 (identifier 방식)

### hugo.yaml 메뉴 설정

```yaml
menu:
  main:
    - identifier: learning-log
      name: Learning Log
      pageRef: /learning-log
      weight: 1
    - identifier: debug-notes
      name: Debug Notes
      pageRef: /debug-notes
      weight: 2
    - identifier: playground
      name: Playground
      pageRef: /playground
      weight: 3
    - identifier: about
      name: About
      pageRef: /about
      weight: 4
    - identifier: search
      name: Search
      weight: 5
      params:
        type: search
    - identifier: github
      name: GitHub
      weight: 6
      url: "https://github.com/YOUR_USERNAME"
      params:
        icon: github
```

### i18n/ko.yaml (한국어 번역)

```yaml
learning-log: "학습 노트"
debug-notes: "디버그 노트"
playground: "놀이터"
about: "소개"
search: "검색"
github: "GitHub"
```

### i18n/en.yaml (영어 번역)

```yaml
learning-log: "Learning Log"
debug-notes: "Debug Notes"
playground: "Playground"
about: "About"
search: "Search"
github: "GitHub"
```

---

## 3. 언어 전환 버튼 추가

```yaml
menu:
  main:
    - name: Language Switcher
      weight: 7
      params:
        type: language-switch
        label: true  # 언어 이름 표시
        icon: "globe-alt"  # 아이콘 (기본값: "translate")
```

---

## 4. 테마 토글 버튼 추가

```yaml
menu:
  main:
    - name: Theme Toggle
      weight: 8
      params:
        type: theme-toggle
        label: false  # 라벨 숨김 (아이콘만 표시)

params:
  theme:
    default: system  # light | dark | system
    displayToggle: true  # 테마 토글 버튼 표시
```

---

## 5. 추가 i18n 번역 항목 (공통)

### i18n/ko.yaml

```yaml
# 기본 레이블
copyright: "© 2025 jacti-log."
changeLanguage: "언어 변경"

# 네비게이션
readMore: "더 보기"
editThisPage: "이 페이지 수정"
lastUpdated: "마지막 업데이트"

# 검색
searchPlaceholder: "검색..."
noResults: "결과 없음"

# 페이지네이션
next: "다음"
previous: "이전"
```

### i18n/en.yaml

```yaml
# Basic labels
copyright: "© 2025 jacti-log."
changeLanguage: "Change language"

# Navigation
readMore: "Read more"
editThisPage: "Edit this page"
lastUpdated: "Last updated"

# Search
searchPlaceholder: "Search..."
noResults: "No results"

# Pagination
next: "Next"
previous: "Previous"
```

---

## 6. 드롭다운 메뉴 (계층적 네비게이션)

```yaml
menu:
  main:
    - identifier: products
      name: "Products"
      weight: 9
    - name: "Product A"
      parent: products
      url: "/product-a"
      weight: 1
    - name: "Product B"
      parent: products
      url: "/product-b"
      weight: 2
```

---

## 7. 푸터 설정

```yaml
params:
  footer:
    displayCopyright: false   # 저작권 표시
    displayPoweredBy: false   # "Powered by Hextra" 표시
```

---

## 8. 완전한 hugo.yaml 예시 (jacti-log 프로젝트)

```yaml
title: jacti-log
baseURL: "https://blog.girae.me/"
theme: hextra

# 다국어 설정
defaultContentLanguage: ko
languages:
  ko:
    languageName: 한국어
    weight: 1
  en:
    languageName: English
    weight: 2

# Markdown 설정
markup:
  goldmark:
    renderer:
      unsafe: true
  highlight:
    noClasses: false

# 메뉴
menu:
  main:
    - identifier: learning-log
      name: Learning Log
      pageRef: /learning-log
      weight: 1
    - identifier: debug-notes
      name: Debug Notes
      pageRef: /debug-notes
      weight: 2
    - identifier: playground
      name: Playground
      pageRef: /playground
      weight: 3
    - identifier: about
      name: About
      pageRef: /about
      weight: 4
    - name: Search
      weight: 5
      params:
        type: search
    - name: GitHub
      weight: 6
      url: "https://github.com/YOUR_USERNAME"
      params:
        icon: github
    - name: Language
      weight: 7
      params:
        type: language-switch
        label: false
        icon: "globe-alt"

# 파라미터
params:
  navbar:
    displayTitle: true
    displayLogo: false

  theme:
    default: system
    displayToggle: true

  footer:
    displayCopyright: true
    displayPoweredBy: true

  editURL:
    enable: true
    base: "https://github.com/YOUR_USERNAME/jacti-log/edit/main/content"
```

---

## 9. 사용법 안내

### 적용 순서
1. `hugo.yaml` 파일 업데이트
2. `i18n/ko.yaml` 파일 생성
3. `i18n/en.yaml` 파일 생성 (선택)
4. `hugo server -D` 명령으로 로컬 테스트
5. 언어 전환 버튼 및 번역 확인

### 주의사항
- `identifier` 사용 시 메뉴 항목명이 자동으로 i18n 파일에서 번역됨
- `name` 직접 지정 시 번역되지 않으므로 identifier 방식 권장
- `weight` 값이 낮을수록 메뉴 순서가 앞에 표시됨
- `pageRef`는 내부 링크, `url`은 외부 링크에 사용

### 검증 방법
```bash
# 로컬 서버 실행
hugo server -D --disableFastRender

# 브라우저에서 확인:
# - 언어 전환 버튼 작동 여부
# - 메뉴 항목 번역 표시 확인
# - 다크/라이트 모드 토글 확인
```
