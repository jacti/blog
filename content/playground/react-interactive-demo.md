---
title: React 인터랙션 데모
date: 2025-01-22
---

## 프로젝트 개요

React Hooks와 상태 머신을 조합해서 상태 전이를 시각적으로 확인할 수 있는 미니 앱입니다.

{{< callout type="info" >}}
실제 데모 HTML 파일은 `static/demos/react-interactive/index.html` 경로에 배치할 예정입니다.
{{< /callout >}}

## 특징

- 상태 변경 기록을 타임라인으로 표시
- 컴포넌트 간 이벤트 흐름을 시각적으로 표현
- Storybook으로 작성된 UI 스토리와 연동

## 체크리스트

{{< steps >}}
{{< step title="상태 차트 설계" >}}
XState로 상태와 이벤트를 다이어그램으로 정의합니다.
{{< /step >}}
{{< step title="UI 연결" >}}
React Hook으로 상태 전이를 UI 동작에 매핑합니다.
{{< /step >}}
{{< step title="문서화" >}}
데모 사용법을 README와 영상으로 정리합니다.
{{< /step >}}
{{< /steps >}}
