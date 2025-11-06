# Hextra 블로그 개발 계획 검토 결과 및 명령

**검토일**: 2025-11-06
**검토 대상**: 1차 개발 계획 (`dev-docs/1차_개발계획.md`)
**검토자**: AI 코드 리뷰어 (웹 검색 + Context7 기반)

---

## 🔴 중대한 오류 발견: "테마 모듈 복원" 전략

### 현재 상황 분석

프로젝트는 **이미 Git Submodule 방식으로 Hextra 테마가 정상 설치**되어 있습니다:

```
✅ themes/hextra/ 디렉토리에 완전한 테마 파일 존재
✅ Git submodule 상태: v0.11.1-7-g708358d (정상 연결)
✅ layouts/shortcodes/, assets/, i18n/ 등 모든 테마 자산 존재
✅ hugo.yaml에 theme: hextra 설정 정상
```

### ❌ 1차 개발 계획의 치명적 오류

**문제 항목 #1 (라인 28-29)**:
> 1. `hugo.yaml`에서 `module.imports`를 활성화하고 `hugo mod tidy`로 테마 자산 동기화.

**이 작업은 완전히 불필요하며 오히려 위험합니다:**

1. **테마는 이미 설치되어 있음**: Git Submodule로 정상 작동 중
2. **Hugo Modules와 Git Submodule은 상호 배타적**: 두 방식을 혼용하면 충돌 발생
3. **현재 설정과 모순**: `hugo.yaml:10`에 `theme: hextra`가 이미 명시됨 (Submodule 방식)
4. **go.mod가 비어있음**: Hugo Modules를 사용하려면 의존성이 명시되어야 하지만 현재 비어있음

**근거**:
- Hextra 공식 문서: 두 가지 설치 방법 제공 (Hugo Modules **또는** Git Submodule)
- Git Submodule 방식은 `theme: hextra` 설정 사용
- Hugo Modules 방식은 `module.imports` 설정 사용
- **둘 중 하나만 선택해야 함**

### 🔍 실제 문제의 정체

1차 개발 계획서 라인 5, 39에서 언급된 "빌드 오류"와 "cards 숏코드 미존재"는:

```
❌ 테마 파일 누락 문제가 아님
✅ 다른 원인일 가능성:
   - 빌드 명령어 오류
   - 캐시 문제
   - 로컬 개발 환경 설정 문제
   - content 파일 내 잘못된 shortcode 사용법
```

실제 확인 결과 `themes/hextra/layouts/shortcodes/` 디렉토리에 모든 숏코드가 존재합니다.

---

## 📋 수정된 작업 계획 (우선순위)

### ⚠️ 즉시 삭제할 항목

**❌ 삭제: 작업 #1 "테마 자산 동기화"**
- `hugo.yaml`의 `module.imports` 주석 처리된 라인(6-8)은 **절대 활성화하지 말 것**
- `hugo mod tidy` 실행 불필요 (Git Submodule 방식에서는 무의미)
- **현재 hugo.yaml:10 `theme: hextra` 설정 유지**

### ✅ 유지 및 개선할 항목

**작업 우선순위 (수정됨)**:

#### 1단계: 현재 상태 검증 (NEW - 최우선)
```bash
# 테마 정상 작동 확인
hugo server --buildDrafts --disableFastRender

# 만약 cards 숏코드 오류 발생 시:
# - content 파일에서 숏코드 사용법 재확인
# - Hugo 버전 확인 (minimum: 0.146.0 extended)
# - 캐시 정리: hugo --gc
```

#### 2단계: 다국어 설정 (우선순위 ⬆️)
- ✅ 유지: `defaultContentLanguage: ko`, `languages` 블록 추가
- ✅ 유지: `i18n/ko.yaml`, `i18n/en.yaml` 번역 추가
- **개선**: Hextra는 20개 언어 지원 - 필요한 언어만 선택적으로 추가

#### 3단계: 콘텐츠 구조 개편
- ✅ 유지: 모든 섹션 생성 계획 적절
- ✅ 유지: `Learning log`, `Debug notes`는 `type: docs` 레이아웃 활용
- ✅ 유지: `Playground` 카드 레이아웃
- ✅ 유지: `Workbench`에 `draft: true` 콘텐츠 보관

#### 4단계: 커스텀 레이아웃 (수정됨)
**기존 계획**: "커스텀 파셜 작성"
**수정 권고**:
- Hextra는 `layouts/_partials/custom/` 디렉토리를 커스텀 오버라이드 용도로 제공
- 새 파셜 작성 전, 기존 테마 컴포넌트 재사용 최대화
- `Playground` 카드 레이아웃은 Hextra 내장 `cards` 숏코드로 충분할 가능성

#### 5단계: 색상 팔레트 커스터마이징 (개선됨)
**기존 계획**: "SCSS 커스텀"
**최신 방법 (2025)**:
- Hextra는 **Tailwind CSS v4+** 사용 (PostCSS 기반)
- SCSS 대신 **CSS 변수 기반 커스터마이징 권장**
- 경로: `assets/css/custom.css` 생성
- 참고: `assets/css/styles.css`의 Tailwind 변수 오버라이드

```css
/* assets/css/custom.css 예시 */
:root {
  --color-primary: #your-color;
  --color-accent: #your-accent;
}
```

#### 6단계: 샘플 콘텐츠 및 배포
- ✅ 유지: 샘플 콘텐츠 작성 및 피드백 반영
- ✅ 유지: 배포 파이프라인 검증
- **주의**: `--buildDrafts`는 프로덕션 빌드에서 제거 필요

---

## 🚨 강력 권고 사항

### 1. Hugo Modules 전환하지 말 것
현재 Git Submodule 방식이 정상 작동 중이므로:
- ❌ Hugo Modules로 전환 **불필요**
- ❌ `go.mod`에 의존성 추가 **불필요**
- ❌ `module.imports` 활성화 **위험**

**전환이 필요한 경우**:
- Git Submodule을 완전히 제거한 후 Hugo Modules로 이전
- 둘을 혼용하면 빌드 오류 발생

### 2. 현재 문제의 실제 원인 파악 필요
"cards 숏코드 누락" 오류는 테마 파일 문제가 아닙니다:
1. **Hugo 버전 확인**: `hugo version` (0.146.0+ extended 필요)
2. **캐시 정리**: `hugo --gc` 또는 `hugo mod clean`
3. **숏코드 사용법 검증**: content 파일의 `{{< cards >}}` 문법 확인
4. **로컬 빌드 테스트**: `hugo server -D --disableFastRender`

### 3. 최신 Hextra 기능 활용
2025년 현재 Hextra v0.11.1+ 주요 기능:
- ✅ Tailwind CSS v4+ (PostCSS)
- ✅ 20개 언어 지원 (RTL 포함)
- ✅ FlexSearch 전문 검색
- ✅ 다크모드 자동 전환
- ✅ 풍부한 내장 숏코드: callout, cards, tabs, steps, filetree, jupyter

**불필요한 커스텀 개발 방지**:
- 대부분의 UI 컴포넌트는 이미 제공됨
- 커스텀 작업 전 테마 기능 먼저 확인

---

## 📝 Codex에게 전달할 명령어

> **⚠️ 중요**: 이 명령을 실행하기 전에 반드시 `dev-docs/commands/README.md`를 먼저 읽으세요!
>
> 모든 코드 스니펫은 `dev-docs/commands/` 폴더에 준비되어 있습니다.

---

## 🔴 긴급 수정 명령

### 삭제할 작업
- ❌ **작업 #1 삭제**: "hugo.yaml에서 module.imports 활성화" → 이 작업은 실행하지 마세요
- ❌ **hugo mod tidy 실행 금지**: 현재 Git Submodule 방식과 충돌합니다

---

## 📚 코드 스니펫 사용 가이드

**모든 작업에 필요한 최신 코드는 아래 파일들에 준비되어 있습니다:**

```bash
dev-docs/commands/
├── README.md                    # 👈 먼저 읽으세요! 전체 워크플로우
├── 01-multilingual-setup.md     # 다국어 설정 코드
├── 02-content-structure.md      # 콘텐츠 구조 템플릿
├── 03-color-customization.md    # 색상 커스터마이징 코드
└── 04-shortcodes-reference.md   # 숏코드 사용법
```

---

## 🚀 수정된 작업 순서 (스니펫 활용)

### **0단계: 전체 가이드 읽기** (필수!)

```bash
# 전체 워크플로우 및 주의사항 확인
cat dev-docs/commands/README.md
```

**핵심 확인 사항**:
- ❌ 금지 사항 리스트
- ✅ 5단계 작업 가이드
- ✅ 체크리스트
- ✅ 빠른 참조 명령어

---

### **1단계: 현재 상태 검증** (최우선)

```bash
# 현재 상태 검증
hugo version  # 0.146.0+ extended 확인
hugo --gc     # 캐시 정리
hugo server -D --disableFastRender  # 로컬 빌드 테스트
```

---

### **2단계: 다국어 설정** 📘 스니펫 파일: `01-multilingual-setup.md`

```bash
# 스니펫 파일 확인
cat dev-docs/commands/01-multilingual-setup.md
```

**실행 작업**:
- [ ] `hugo.yaml`에 다국어 설정 추가 (스니펫 섹션 1, 2 참조)
- [ ] 메뉴 identifier 방식 적용 (스니펫 섹션 2 참조)
- [ ] `i18n/ko.yaml` 생성 (스니펫 섹션 5 참조)
- [ ] `i18n/en.yaml` 생성 (선택, 스니펫 섹션 5 참조)
- [ ] 완전한 hugo.yaml 예시 복사 (스니펫 섹션 8 참조)

**바로 사용 가능한 코드**:
- 완전한 `hugo.yaml` 템플릿 (섹션 8)
- `i18n/ko.yaml` 전체 코드 (섹션 5)
- 메뉴 설정 전체 코드 (섹션 2)

---

### **3단계: 콘텐츠 구조 생성** 📘 스니펫 파일: `02-content-structure.md`

```bash
# 스니펫 파일 확인
cat dev-docs/commands/02-content-structure.md
```

**실행 작업**:
- [ ] `content/learning-log/_index.md` 생성 (스니펫 섹션 2 참조)
- [ ] `content/debug-notes/_index.md` 생성 (스니펫 섹션 3 참조)
- [ ] `content/playground/_index.md` 생성 (스니펫 섹션 4 참조)
- [ ] `content/about/_index.md` 생성 (스니펫 섹션 5 참조)
- [ ] `content/workbench/_index.md` 생성 (스니펫 섹션 6 참조)
- [ ] `content/_index.md` 홈페이지 생성 (스니펫 섹션 7 참조)

**바로 사용 가능한 코드**:
- 각 섹션별 완전한 `_index.md` 템플릿
- Front Matter 설정 예시 (섹션 8)
- 디렉토리 구조 가이드 (섹션 1)

---

### **4단계: 색상 커스터마이징** 📘 스니펫 파일: `03-color-customization.md`

```bash
# 스니펫 파일 확인
cat dev-docs/commands/03-color-customization.md
```

**실행 작업**:
- [ ] `assets/css/custom.css` 생성
- [ ] 기본 템플릿 복사 (스니펫 섹션 1 참조)
- [ ] 색상 팔레트 선택 (스니펫 섹션 2: Teal, Amber, Indigo, Blue)
- [ ] 다크 모드 설정 (스니펫 섹션 3 참조)
- [ ] 폰트 커스터마이징 (선택, 스니펫 섹션 6 참조)

**바로 사용 가능한 코드**:
- 완전한 `custom.css` 템플릿 (섹션 7)
- 4가지 색상 팔레트 코드 (섹션 2)
- 컴포넌트별 스타일링 (섹션 5)

**⚠️ 중요**:
- ❌ SCSS 파일 생성 금지
- ✅ CSS 변수 사용 (Tailwind CSS v4)
- ✅ `hx:` 접두어 사용 (이전: `hx-`)

---

### **5단계: 콘텐츠 작성 및 숏코드 활용** 📘 스니펫 파일: `04-shortcodes-reference.md`

```bash
# 스니펫 파일 확인
cat dev-docs/commands/04-shortcodes-reference.md
```

**활용 예시**:
- [ ] Playground: `cards` 숏코드로 데모 그리드 (스니펫 섹션 1, 9 참조)
- [ ] Learning Log: `callout`, `steps`, `tabs` 활용 (스니펫 섹션 2, 3, 4, 9 참조)
- [ ] Debug Notes: `callout`, `steps` 활용 (스니펫 섹션 2, 4, 9 참조)
- [ ] About: `cards`, `hextra/feature-card` 활용 (스니펫 섹션 1, 8 참조)

**바로 사용 가능한 코드**:
- Cards 숏코드 전체 예시 (섹션 1)
- Callout 타입별 예시 (섹션 2)
- Tabs 코드 블록 예시 (섹션 3)
- Steps 가이드 예시 (섹션 4)
- **실전 예시 모음** (섹션 9) 👈 가장 유용!

---

## ⚡ 빠른 시작 (한 줄 명령)

```bash
# 각 단계별로 스니펫을 바로 확인하고 복사-붙여넣기
# 1단계: 다국어
grep -A 50 "## 8. 완전한 hugo.yaml" dev-docs/commands/01-multilingual-setup.md

# 2단계: 콘텐츠
grep -A 30 "### \`content/learning-log/_index.md\`" dev-docs/commands/02-content-structure.md

# 3단계: 색상
grep -A 100 "## 7. 완전한 custom.css" dev-docs/commands/03-color-customization.md

# 4단계: 숏코드
grep -A 50 "### Playground 섹션" dev-docs/commands/04-shortcodes-reference.md
```

---

## 📋 작업 체크리스트

완료 시 체크:
- [ ] README.md 읽고 금지 사항 숙지
- [ ] 현재 상태 검증 완료 (`hugo server` 정상 작동)
- [ ] `01-multilingual-setup.md` 기반 hugo.yaml 수정
- [ ] `02-content-structure.md` 기반 콘텐츠 디렉토리 생성
- [ ] `03-color-customization.md` 기반 custom.css 생성
- [ ] `04-shortcodes-reference.md` 기반 샘플 콘텐츠 작성
- [ ] 다크/라이트 모드 확인
- [ ] 모든 숏코드 정상 작동 확인
- [ ] `hugo --minify` 프로덕션 빌드 성공

---

## 🎯 중요 원칙 (반복 강조)

1. **현재 hugo.yaml:10의 `theme: hextra` 설정 절대 변경 금지**
2. **hugo.yaml:6-8의 module.imports 주석 절대 활성화 금지**
3. **Git Submodule과 Hugo Modules 혼용 절대 금지**
4. **모든 코드는 `dev-docs/commands/` 스니펫 파일 참조**
5. **의문이 생기면 `README.md` 다시 읽기**

---

## 🔗 참고 자료

### 공식 문서
- [Hextra 설치 가이드](https://imfing.github.io/hextra/docs/getting-started/)
- [Hextra 설정 문서](https://imfing.github.io/hextra/docs/guide/configuration/)
- [Hugo Modules vs Git Submodule](https://gohugo.io/hugo-modules/)

### 검증된 정보
- **Context7 Library ID**: `/imfing/hextra` (Trust Score: 9.4)
- **최소 Hugo 버전**: 0.146.0 (extended)
- **현재 테마 버전**: v0.11.1-7 (Git Submodule)
- **CSS 프레임워크**: Tailwind CSS v4+ (PostCSS)

---

## ✅ 검토 결과 요약

| 항목 | 상태 | 조치 |
|------|------|------|
| 테마 설치 | ✅ 정상 | 추가 작업 불필요 |
| 작업 #1 (module.imports) | ❌ 오류 | **즉시 삭제** |
| 다국어 설정 | ✅ 적절 | 계획대로 진행 |
| 콘텐츠 구조 | ✅ 적절 | 계획대로 진행 |
| 색상 커스터마이징 | ⚠️ 수정 필요 | SCSS → CSS 변수 |
| 커스텀 레이아웃 | ⚠️ 과도함 | 테마 기능 우선 활용 |
| 배포 계획 | ✅ 적절 | 계획대로 진행 |

**최종 권고**: 1차 개발 계획의 80%는 적절하나, 작업 #1은 **즉시 삭제 필수**. 나머지는 CSS 커스터마이징 방법만 현대화하면 실행 가능.
