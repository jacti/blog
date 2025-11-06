# jacti-log Hugo 템플릿 안내

## 프로젝트 개요
- Hugo와 Hextra 테마를 바탕으로 하는 초기 템플릿입니다.
- `content/` 아래의 문서와 `hugo.yaml` 설정을 조정해 원하는 블로그나 문서 사이트를 빠르게 만들 수 있습니다.
- 기본 구조를 이해하고 필요한 지점만 교체하면 테마 전체를 건드리지 않고도 커스터마이징이 가능합니다.

## 준비 사항
- Hugo Extended 0.120+ 버전, Go, Git이 설치되어 있어야 합니다.
- 버전 확인 예시: `hugo version`, `go version`, `git --version`.
- 외부 모듈을 동기화하기 위해 한 번씩 `hugo mod tidy`를 실행해주세요.

## 개발 서버와 빌드
```bash
# 의존성 정리
hugo mod tidy

# 로컬 미리보기 (http://localhost:1313)
hugo server --disableFastRender -p 1313

# 프로덕션 빌드 (public/ 폴더 생성)
hugo --minify

# 초안/예약 게시물까지 포함해 검수할 때
hugo --buildDrafts --buildFuture --minify
```

## 주요 폴더 구조
| 경로 | 설명 | 자주 하는 변경 |
| --- | --- | --- |
| `content/` | 공개될 페이지와 문서의 마크다운 파일 위치 | 새 글 작성, front matter 수정, 섹션 구조 재정비 |
| `archetypes/` | `hugo new` 실행 시 사용할 기본 front matter 템플릿 | 새 문서 타입이 필요할 때 파일 추가/수정 |
| `layouts/` | 기본 테마를 오버라이드할 Hugo 템플릿 레이아웃 | 특정 페이지/부분 UI 커스터마이징 |
| `assets/` | SCSS, JS 등 Hugo Pipes로 처리되는 리소스 | 커스텀 스타일, 스크립트 추가 |
| `data/` | Hugo 템플릿에서 읽어오는 YAML/JSON/TOML 데이터 | 네비게이션, 반복 데이터 구성 |
| `i18n/` | 다국어 번역 문자열 | 다국어 지원 문구 수정 |
| `static/` | 정적 파일이 그대로 배포되는 위치 | 이미지, 파비콘, robots.txt 등 배치 |
| `themes/hextra/` | 가져온 Hextra 테마 소스 | 직접 수정은 최소화, 필요 시 자체 포크 권장 |
| `hugo.yaml` | 사이트 전역 설정(메뉴, 언어, 파라미터) | 사이트 제목, 메뉴, 모듈 설정 변경 |
| `netlify.toml` | Netlify 배포 설정 예시 | Netlify 사용 시 빌드 명령/환경 변수 조정 |

## 커스터마이징 가이드
- **콘텐츠 작성**: `hugo new content/docs/<경로>/index.md` 명령으로 초안(`draft: true`) 페이지를 생성하고, front matter의 `title`, `description`, `weight` 등을 채워주세요.
- **메뉴 및 네비게이션**: `hugo.yaml`의 `menu` 섹션을 수정하거나 `data/` 디렉터리에 데이터 파일을 추가해 항목을 제어할 수 있습니다.
- **스타일 변경**: `assets/css/`에 SCSS 파일을 추가한 뒤 `hugo.yaml`의 `params.inject_stylesheets` 옵션으로 불러오면 전역 스타일을 확장할 수 있습니다.
- **컴포넌트 오버라이드**: 테마의 특정 레이아웃을 수정하고 싶다면 동일한 경로를 `layouts/` 밑에 생성해 파일을 복사한 뒤 필요한 부분만 변경합니다.
- **정적 자산 관리**: 페이지에서 사용하는 이미지는 `static/images/<주제>/`에 두면 최종 빌드 시 `/images/<주제>/` 경로로 배포됩니다.
- **모듈 동기화**: 테마 버전을 업데이트하거나 모듈 구성을 바꿀 때는 `go.mod` / `go.sum`과 함께 `hugo mod tidy`를 다시 실행해 잠금 파일을 정리합니다.

## Hextra 테마 활용법
- **주요 특징**: Tailwind CSS 기반의 현대적인 디자인, 다크 모드, 반응형 레이아웃, FlexSearch 기반 오프라인 검색, 다국어 및 SEO 지원 등 대부분의 기능이 기본값으로 포함되어 있습니다.
- **빠르게 시작하기**: [Hextra Starter Template](https://github.com/imfing/hextra-starter-template)를 복제하거나 현재 저장소처럼 모듈로 추가해 바로 사용하세요. 테마 업데이트가 필요하면 `hugo mod get -u` 후 `hugo mod tidy`로 잠금 파일을 정리합니다.
- **공식 문서 살펴보기**: [Hextra 문서 사이트](https://imfing.github.io/hextra/docs)에서 레이아웃, 사이드바, 검색, 다국어 설정을 단계별로 안내합니다. 특히 `docs/configuration` 섹션은 `hugo.yaml` 파라미터 설명이 잘 정리되어 있습니다.
- **컴포넌트 & 숏코드**: Markdown에서 사용할 수 있는 `callout`, `tabs`, `steps` 등 내장 컴포넌트가 많으니 문서의 `docs/content` 챕터를 참고해 활용 범위를 확인하세요.
- **스타일 확장**: Tailwind 유틸리티를 그대로 사용할 수 있으며, 필요 시 `assets/css/`에 SCSS 파일을 작성해 `hugo.yaml`의 `params.inject_stylesheets`로 등록하면 전역 스타일을 확장할 수 있습니다.

## 작업 흐름 팁
- 큰 변경 전에는 `public/` 디렉터리를 삭제하고 `hugo --minify`를 실행해 깨끗한 빌드 산출물을 확인하세요.
- 커밋 메시지는 `feat:`, `fix:`처럼 명확한 접두사를 사용하면 기록 관리가 편합니다.
- 시각적 변경을 PR로 공유할 때는 `hugo server` 미리보기 화면 스크린샷이나 `public/` 에서 확인한 경로를 첨부하면 리뷰가 수월합니다.

필요한 기본 구조는 모두 포함되어 있으니, 위 안내에 따라 필요한 부분을 하나씩 수정해 나가면 됩니다. 문제가 생기면 `hugo server --logLevel debug` 옵션으로 상세 로그를 확인하세요.
