---
title: "Callout 예시"
sidebar:
  exclude: true
_build:
  render: never
  list: never
---

## Callout (알림 박스)

강조하거나 주의를 끌어야 하는 내용을 표시합니다.

**기본 사용법:**

```markdown
{{</* callout type="info" */>}}
  여기에 내용을 작성합니다.
{{</* /callout */>}}
```

**실제 예시:**

{{< callout type="info" >}}
  여기에 내용을 작성합니다.
{{< /callout >}}

**사용 가능한 타입:**
- `info` - 정보 (파란색)
- `warning` - 경고 (노란색)
- `error` - 오류 (빨간색)
- `important` - 중요 (보라색)
