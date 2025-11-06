---
title: React 훅스 총정리
date: 2025-01-20
weight: 1
---

## React 훅스가 필요한 이유

클래스 컴포넌트에서 상태와 생명주기를 관리하던 패턴을 함수형으로 단순하게 표현할 수 있습니다.

{{< callout type="tip" >}}
훅은 **상위 레벨에서만 호출**해야 하며 조건문이나 반복문 안에서 사용하면 안 됩니다.
{{< /callout >}}

### 필수 훅

- `useState`: 로컬 상태 관리
- `useEffect`: 부수 효과 처리
- `useMemo`: 계산 비용이 큰 값을 캐싱

### 커스텀 훅 패턴

```jsx {title="useFetch.ts"}
import { useEffect, useState } from "react";

export function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let mounted = true;

    fetch(url)
      .then((res) => res.json())
      .then((json) => mounted && setData(json))
      .finally(() => mounted && setLoading(false));

    return () => {
      mounted = false;
    };
  }, [url]);

  return { data, loading };
}
```

### 실무 체크리스트

{{% steps %}}

### 의존성 배열 점검

필수 값만 넣고 불필요한 재렌더를 방지합니다.

### 메모리 누수 방지

클린업 함수에서 타이머/구독을 정리합니다.

### 재사용성 확인

커스텀 훅으로 공통 로직을 모듈화합니다.

{{% /steps %}}
