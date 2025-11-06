---
title: "Hugo 빌드 오류 해결기"
date: 2025-01-10
tags: ["hugo", "build", "shortcode"]
weight: 1
---

## 문제 상황

배포 파이프라인에서 `cards` 숏코드를 찾지 못한다는 오류가 발생했습니다.

{{< callout type="danger" title="에러 로그" >}}
Error: failed to extract shortcode: template for shortcode "cards" not found
{{< /callout >}}

## 해결 과정

{{% steps %}}

### 테마 상태 확인

Git Submodule로 포함된 Hextra 테마가 최신 커밋인지 확인했습니다.

### 서브모듈 초기화

`git submodule update --init --recursive` 명령으로 테마 파일을 다시 내려받았습니다.

### 캐시 정리

`hugo --gc`로 캐시를 정리한 후 다시 빌드하니 문제가 해결되었습니다.

{{% /steps %}}

## 배운 점

- 테마와 모듈 방식은 혼용하지 않는다.
- 빌드 오류가 발생하면 서브모듈, 캐시, 버전 요구사항을 순서대로 점검한다.
- 에러 로그를 빠르게 공유할 수 있도록 스니펫을 정리해 두면 재발 방지에 도움이 된다.
