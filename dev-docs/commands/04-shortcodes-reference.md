# Hextra 내장 숏코드 레퍼런스

**출처**: Hextra 공식 문서 (2025년 최신)
**적용 대상**: Markdown 콘텐츠 파일 (`.md`)

---

## 1. Cards (카드 그리드)

### 기본 사용법

```markdown
{{< cards >}}
  {{< card link="learning-log" title="Learning Log" icon="book-open" >}}
  {{< card link="debug-notes" title="Debug Notes" icon="bug" >}}
  {{< card link="playground" title="Playground" icon="beaker" >}}
{{< /cards >}}
```

### 카드 옵션

```markdown
{{< cards >}}
  {{< card
    link="/path"
    title="제목"
    icon="icon-name"
    subtitle="부제목"
    tag="베타"
    tagColor="blue"
  >}}
{{< /cards >}}
```

### 이미지 카드

```markdown
{{< cards >}}
  {{< card
    link="/"
    title="이미지 카드"
    image="https://example.com/image.jpg"
    subtitle="인터넷 이미지"
  >}}

  {{< card
    link="/"
    title="로컬 이미지"
    image="/images/card-image.jpg"
    subtitle="static 디렉토리 이미지"
  >}}

  {{< card
    link="/"
    title="처리된 이미지"
    image="images/photo.jpg"
    subtitle="assets 디렉토리 (Hugo 처리)"
    method="Resize"
    options="600x q80 webp"
  >}}
{{< /cards >}}
```

### 컬럼 수 조정

```markdown
{{< cards cols="1" >}}
  {{< card title="상단 카드" >}}
  {{< card title="하단 카드" >}}
{{< /cards >}}

{{< cards cols="2" >}}
  {{< card title="왼쪽 카드" >}}
  {{< card title="오른쪽 카드" >}}
{{< /cards >}}

{{< cards cols="3" >}}
  {{< card title="카드 1" >}}
  {{< card title="카드 2" >}}
  {{< card title="카드 3" >}}
{{< /cards >}}
```

---

## 2. Callout (강조 박스)

### 타입별 사용법

```markdown
{{< callout type="info" >}}
정보 메시지입니다.
{{< /callout >}}

{{< callout type="warning" >}}
경고 메시지입니다.
{{< /callout >}}

{{< callout type="error" >}}
오류 메시지입니다.
{{< /callout >}}

{{< callout type="important" >}}
중요 메시지입니다.
{{< /callout >}}

{{< callout >}}
기본 메시지입니다 (전구 아이콘).
{{< /callout >}}
```

### 커스텀 아이콘

```markdown
{{< callout icon="star" >}}
커스텀 아이콘을 사용한 메시지
{{< /callout >}}

{{< callout emoji="🚀" >}}
이모지를 사용한 메시지
{{< /callout >}}
```

---

## 3. Tabs (탭)

### 기본 사용법

```markdown
{{< tabs items="macOS,Linux,Windows" >}}

  {{< tab >}}
  **macOS**: Apple이 개발한 데스크톱 운영체제입니다.
  {{< /tab >}}

  {{< tab >}}
  **Linux**: 오픈소스 운영체제입니다.
  {{< /tab >}}

  {{< tab >}}
  **Windows**: Microsoft가 개발한 운영체제입니다.
  {{< /tab >}}

{{< /tabs >}}
```

### 코드 블록과 함께 사용

```markdown
{{< tabs items="JSON,YAML,TOML" >}}

  {{< tab >}}
  ```json
  {
    "hello": "world"
  }
  ```
  {{< /tab >}}

  {{< tab >}}
  ```yaml
  hello: world
  ```
  {{< /tab >}}

  {{< tab >}}
  ```toml
  hello = "world"
  ```
  {{< /tab >}}

{{< /tabs >}}
```

### 기본 탭 설정

```markdown
{{< tabs items="Tab1,Tab2,Tab3" defaultIndex="1" >}}
  {{< tab >}}첫 번째 탭{{< /tab >}}
  {{< tab >}}두 번째 탭 (기본 활성){{< /tab >}}
  {{< tab >}}세 번째 탭{{< /tab >}}
{{< /tabs >}}
```

---

## 4. Steps (단계별 가이드)

### 기본 사용법

```markdown
{{% steps %}}

### Step 1

첫 번째 단계입니다.

### Step 2

두 번째 단계입니다.

### Step 3

세 번째 단계입니다.

{{% /steps %}}
```

### 상세 내용 포함

```markdown
{{% steps %}}

### 프로젝트 초기화

```bash
hugo new site my-site
cd my-site
```

### 테마 설치

Git Submodule 방식으로 설치합니다:

```bash
git submodule add https://github.com/imfing/hextra.git themes/hextra
```

### 설정 파일 작성

`hugo.yaml` 파일을 생성합니다.

{{% /steps %}}
```

---

## 5. Details (접기/펼치기)

### 기본 사용법

```markdown
{{< details title="더 보기" >}}
숨겨진 내용입니다.

- 리스트 아이템 1
- 리스트 아이템 2
{{< /details >}}
```

### 기본 열림 상태

```markdown
{{< details title="기본 열림" open=true >}}
이 내용은 기본적으로 표시됩니다.
{{< /details >}}
```

---

## 6. FileTree (파일 트리)

### 기본 사용법

```markdown
{{< filetree/container >}}
  {{< filetree/folder name="content" >}}
    {{< filetree/file name="_index.md" >}}
    {{< filetree/folder name="docs" >}}
      {{< filetree/file name="_index.md" >}}
      {{< filetree/file name="getting-started.md" >}}
    {{< /filetree/folder >}}
  {{< /filetree/folder >}}
  {{< filetree/file name="hugo.yaml" >}}
{{< /filetree/container >}}
```

### 프로젝트 구조 예시

```markdown
{{< filetree/container >}}
  {{< filetree/folder name="jacti-log" >}}
    {{< filetree/folder name="content" >}}
      {{< filetree/file name="_index.md" >}}
      {{< filetree/folder name="learning-log" >}}
        {{< filetree/file name="_index.md" >}}
      {{< /filetree/folder >}}
      {{< filetree/folder name="debug-notes" >}}
        {{< filetree/file name="_index.md" >}}
      {{< /filetree/folder >}}
    {{< /filetree/folder >}}
    {{< filetree/folder name="assets" >}}
      {{< filetree/folder name="css" >}}
        {{< filetree/file name="custom.css" >}}
      {{< /filetree/folder >}}
    {{< /filetree/folder >}}
    {{< filetree/file name="hugo.yaml" >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}
```

---

## 7. Icon (아이콘)

### 기본 아이콘 사용

```markdown
{{< icon "star" >}}
{{< icon "heart" >}}
{{< icon "code" >}}
{{< icon "book-open" >}}
```

### 카드와 함께 사용

```markdown
{{< cards >}}
  {{< card icon="star" title="인기" >}}
  {{< card icon="fire" title="트렌딩" >}}
  {{< card icon="sparkles" title="신규" >}}
{{< /cards >}}
```

### 사용 가능한 주요 아이콘

- `book-open`: 책
- `bug`: 버그
- `beaker`: 실험
- `code`: 코드
- `star`: 별
- `fire`: 불
- `sparkles`: 반짝임
- `chart-bar`: 차트
- `terminal`: 터미널
- `folder-tree`: 폴더 트리

### 커스텀 아이콘 추가

`data/icons.yaml` 파일 생성:

```yaml
your-icon: |
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
    <path d="..." />
  </svg>
```

사용:

```markdown
{{< icon "your-icon" >}}
{{< card icon="your-icon" title="제목" >}}
```

---

## 8. Hextra Feature Cards (홈페이지 전용)

### Feature Card

```markdown
{{< hextra/feature-card
  title="빠른 속도"
  subtitle="Hugo 기반의 초고속 정적 사이트 생성"
  image="/images/feature-speed.jpg"
  imageClass="hx:rounded-lg"
  style="background: linear-gradient(to bottom right, #00f, #0ff);"
>}}
```

### Feature Grid

```markdown
{{< hextra/feature-grid cols="3" >}}
  {{< hextra/feature-card
    title="반응형 디자인"
    subtitle="모든 기기에서 완벽하게 작동"
    icon="device-mobile"
  >}}

  {{< hextra/feature-card
    title="다크 모드"
    subtitle="라이트/다크 모드 자동 전환"
    icon="moon"
  >}}

  {{< hextra/feature-card
    title="전문 검색"
    subtitle="FlexSearch 기반 오프라인 검색"
    icon="search"
  >}}
{{< /hextra/feature-grid >}}
```

---

## 9. 실전 예시 모음

### Playground 섹션 (카드 그리드)

```markdown
---
title: Playground
---

AI가 제작한 데모 프로젝트를 체험해보세요.

{{< cards cols="2" >}}
  {{< card
    link="demo-1"
    title="React 인터랙티브 데모"
    icon="code"
    image="/images/demo-1.png"
    subtitle="React Hooks를 활용한 인터랙티브 UI"
  >}}

  {{< card
    link="demo-2"
    title="Three.js 3D 시각화"
    icon="cube"
    image="/images/demo-2.png"
    subtitle="WebGL 기반 3D 그래픽"
  >}}

  {{< card
    link="demo-3"
    title="D3.js 데이터 차트"
    icon="chart-bar"
    image="/images/demo-3.png"
    subtitle="인터랙티브 데이터 시각화"
  >}}

  {{< card
    link="demo-4"
    title="AI 챗봇 데모"
    icon="chat"
    image="/images/demo-4.png"
    tag="새로운"
    tagColor="green"
    subtitle="자연어 처리 기반 챗봇"
  >}}
{{< /cards >}}
```

### Learning Log 포스트

```markdown
---
title: React Hooks 완벽 가이드
---

React Hooks의 개념과 사용법을 정리합니다.

{{< callout type="info" >}}
이 글은 React 16.8+ 버전을 기준으로 작성되었습니다.
{{< /callout >}}

## useState

{{% steps %}}

### 기본 사용법

```jsx
const [count, setCount] = useState(0);
```

### 함수형 업데이트

```jsx
setCount(prev => prev + 1);
```

{{% /steps %}}

## 주의사항

{{< callout type="warning" >}}
Hooks는 컴포넌트 최상위에서만 호출해야 합니다.
{{< /callout >}}
```

### Debug Notes 포스트

```markdown
---
title: Hugo 빌드 오류 해결기
---

Hugo 빌드 시 발생한 오류를 해결한 과정을 기록합니다.

## 문제 상황

{{< callout type="error" >}}
Error: shortcode "cards" not found
{{< /callout >}}

## 해결 과정

{{% steps %}}

### 1단계: 테마 확인

테마 설치 상태를 확인합니다:

```bash
ls -la themes/hextra
```

### 2단계: Git Submodule 동기화

```bash
git submodule update --init --recursive
```

### 3단계: 캐시 정리

```bash
hugo --gc
```

{{% /steps %}}

## 배운 점

{{< callout icon="light-bulb" >}}
Git Submodule은 정기적으로 동기화해야 합니다.
{{< /callout >}}
```

---

## 10. 사용 시 주의사항

### Shortcode vs Markdown

- **`{{< >}}`**: HTML 렌더링 (숏코드)
- **`{{% %}}`**: Markdown 렌더링 (steps, details 등)

### 중첩 사용

```markdown
✅ 가능:
{{< cards >}}
  {{< card >}}
    {{< callout >}}내용{{< /callout >}}
  {{< /card >}}
{{< /cards >}}

❌ 불가능:
{{< tabs >}}
  {{< tab >}}
    {{% steps %}}  <!-- 중첩 오류 -->
  {{< /tab >}}
{{< /tabs >}}
```

### 디버깅

```bash
# 숏코드 렌더링 오류 확인
hugo server -D --logLevel=debug

# 특정 페이지만 빌드
hugo --renderToMemory --source=content/debug-notes/
```
