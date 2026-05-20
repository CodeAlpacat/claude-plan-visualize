---
name: plan-visualize
description: plan/design 마크다운을 30초 안에 검토 가능한 1페이지 HTML로 변환합니다. Mermaid 다이어그램 + 의사결정/리스크/scope-out 박스 + 와이어프레임. MD가 SoT, HTML은 SHA-256 drift guard 박힌 파생 뷰. 풀스펙 문서 아님 — *읽기 어려운 MD를 빠르게 보기 위한 도구*. Use when 사용자가 "시각화", "html로", "plan/design 빠르게 검토", "한눈에", "visualize", "MD to HTML", "可视化", "プラン可視化", "/plan-visualize" 요청.
---

# plan-visualize

> 읽기 어려운 plan/design MD 를 30초 검토용 1페이지 HTML 로.

## 원칙 4가지

1. **MD = SoT.** 생성된 HTML 직접 수정 금지.
2. **자유 > 강제.** 필수 섹션 없음. MD 보고 케이스별 판단.
3. **멱등.** 같은 MD → 같은 HTML.
4. **비대해지지 마.** 단순 MD → ~400 줄, 복잡 → ~1200 줄 max.

## 사용법

```
/plan-visualize <md-path>           # 직접 경로
/plan-visualize plan <name>         # docs/01-plan/features/<name>.plan.md 등 자동 탐색
/plan-visualize design <name>       # docs/02-design/features/<name>.design.md 등
```

출력: MD 경로의 `.md` → `.html`.

## 워크플로우 3 단계

### 1. 입력 + drift 체크

- MD 경로 결정 (직접 / `plan <name>` 컨벤션 탐색 / `design <name>` 컨벤션 탐색)
- MD 없으면 *"먼저 MD 작성"* 안내 후 중단
- 기존 HTML 있으면 hash 비교:
  ```bash
  CURRENT=$(shasum -a 256 <md-path> | awk '{print $1}')
  RECORDED=$(grep -oE 'source-hash: sha256:[a-f0-9]+' <html-path> | sed 's/.*://')
  # 일치하면 재생성 불필요 (사용자 의사 확인), 불일치하면 재생성
  ```

### 2. MD 파싱 + 섹션 자유 선택

MD 전문 읽고 **케이스별로** 섹션 선택. 강제 매핑 X.

| MD 패턴 | 변환 |
|---|---|
| 옵션 A vs B / "결정 필요" | 의사결정 박스 (보라) |
| `## Risk` / 위험 / 함정 | 리스크 박스 (P0 빨강 / P1 노랑) |
| "안 함" / "제외" / "scope out" | Scope-out 박스 (회색) — *상단 노출* |
| "사용자 X → 시스템 Y" | Mermaid sequenceDiagram |
| `CREATE TABLE` / 컬럼 추가 | Mermaid erDiagram |
| 모듈 / 의존 구조 | Mermaid flowchart |
| "상태 X → Y" / 전이 | Mermaid stateDiagram-v2 |
| 페르소나 / 컴포넌트 목록 | 카드 그리드 |
| 비교 매트릭스 | 표 (`.styled`) |
| UI 변경 | 와이어프레임 (1-2 화면, 정적) |
| 결정론 영역 명제 | Invariant 박스 (초록) |

**언어 정제 의무**: 내부 코드명 (`paradigm-c-full-sot`, `cache priming gap`, 커밋 해시) → 풀어쓰기. 표준 기술 용어 (RPC, RLS, JWT, mutation, race condition) → 그대로.

### 3. HTML 생성 + 보고

- `template.html` 셸 로드
- placeholder 치환:
  - `{{SOURCE_PATH}}` — 상대 MD 경로
  - `{{SOURCE_HASH}}` — `shasum -a 256 <md>` 전체 64자
  - `{{SOURCE_HASH_SHORT}}` — 앞 12자
  - `{{GENERATED_AT}}` — `date -u +"%Y-%m-%dT%H:%M:%SZ"`
  - `{{TITLE}}` — MD 의 `# 헤더` 추출
  - `{{CONTENT}}` — LLM 이 구성한 섹션들 (Hero 는 항상, 나머지는 MD 따라)
- 저장 → 파일 경로 + 줄 수 + 섹션 개수 1줄 보고
- `open <html-path>` 명령 안내 (macOS) / `xdg-open` (Linux) / `start` (Windows)

## 색상 시스템 (template.html CSS 변수)

| 변수 | 값 | 용도 |
|---|---|---|
| `--accent` | `#60a5fa` blue-400 | 주 강조, badge |
| `--success` | `#34d399` emerald-400 | 신규 / 추가 / invariant |
| `--warning` | `#fbbf24` amber-400 | P1 위험, 변경 |
| `--danger` | `#f87171` red-400 | P0 위험, 삭제 |
| `--decision` | `#a78bfa` violet-400 | 의사결정 박스 |
| `--muted` | `#94a3b8` slate-400 | scope-out, secondary |

## 박스 4종 스니펫

### 의사결정 (보라)
```html
<section class="decision-box">
  <h2>❓ 결정 필요</h2>
  <p>{{질문}}</p>
  <table class="styled">
    <tr><th>옵션</th><th>장</th><th>단</th><th>권장</th></tr>
    <tr class="recommended"><td>B</td><td>...</td><td>...</td><td>✓</td></tr>
  </table>
</section>
```

### 리스크 (빨강/노랑/파랑)
```html
<div class="risk-box risk-p0">
  <h3>🚨 P0 — {{제목}}</h3>
  <p>{{설명}}</p>
  <p class="mit">대응: {{...}}</p>
</div>
```

### Scope-out (회색)
```html
<section class="scope-out">
  <h2>⛔ 이번 PR 안 함</h2>
  <ul><li>{{항목}} — {{이유}}</li></ul>
</section>
```

### Invariant (초록, 결정론 영역만)
```html
<div class="invariant">
  <span class="invariant-id">I1</span>
  <h3>{{명제}}</h3>
  <pre class="code-block"><code>{{검증 SQL}}</code></pre>
</div>
```

## Mermaid 패턴 4종

### sequenceDiagram (사용자 흐름)
```
sequenceDiagram
  actor U as 사용자
  participant UI as 화면
  U->>UI: {{액션}}
  UI-->>U: ✓ {{피드백}}
```
분기는 `alt / else / end`.

### erDiagram (DB 변경)
```
erDiagram
  USERS ||--o{ POSTS : "1:N"
  USERS { uuid id PK; text email }
```

### flowchart (아키텍처)
```
flowchart TD
  subgraph features[features/]
    F[login/ui]
  end
  F --> E[entities/user]
  classDef new fill:#34d399,color:#0f172a
  class F new
```

### stateDiagram-v2 (상태 머신)
```
stateDiagram-v2
  [*] --> idle
  idle --> loading: submit
  loading --> success
  loading --> error
```

## 분량 가이드

| 입력 MD | 권장 HTML | 섹션 |
|---|---|---|
| ~100 줄 (단순) | ~400 줄 | 3-4 |
| ~300 줄 (중간) | ~700 줄 | 5-7 |
| ~600 줄+ (복잡) | ~1200 줄 | 7-10 |

기준 초과 시 *"진짜 다 필요한가?"* 자문 후 압축.

## 안 만드는 케이스

- 단순 fix / typo / 단일 파일 < 100줄 → MD 자체가 가벼움
- bug fix → MD 짧음 → 가치 낮음
- **시각화 가치 hit**: 3+ 슬라이스 / 새 도메인 / paradigm 변경 / cross-team 공유

## 안 만드는 것 (의도적)

- ❌ Executive Summary 별도 카드 그리드 — Hero meta 로 충분
- ❌ Print CSS / Light mode 토글 / Mobile TOC 햄버거 — 비대화
- ❌ 와이어프레임 상태 토글 (loading/error tabs) — 정적 1-2 화면이면 OK
- ❌ Progressive disclosure `<details>` 남발 — 정말 클 때만
- ❌ 10+ 섹션 강제 매핑 — 케이스별 LLM 자유

## 거울 원칙

**HTML 은 MD 품질의 거울**. MD 에 비즈니스 임팩트 / KPI / 리스크가 없으면 HTML 에도 없음 — 환각 차단. MD 보강이 우선.

## Drift 마커 (frontmatter 필수)

생성 HTML 상단:
```html
<!--
  generated-by: plan-visualize
  source: {{md-path}}
  source-hash: sha256:{{hash}}
  generated-at: {{ISO-timestamp}}
  ⚠️ 자동 생성. 직접 수정 X. MD 수정 후 plan-visualize 재실행.
-->
```
