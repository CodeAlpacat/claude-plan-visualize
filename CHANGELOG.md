# Changelog

[Keep a Changelog](https://keepachangelog.com/ko/1.1.0/) 형식.

## [0.1.0] — 2026-05-20

최초 공개 릴리스.

### 추가됨
- `SKILL.md` — 메인 스킬 정의 (모든 지침 + 색상 시스템 + 박스 스니펫 + Mermaid 패턴 통합). 3-fallback 입력 해석 (직접 경로 / `plan <name>` / `design <name>`).
- `template.html` — 다크 테마 HTML 셸 + Mermaid CDN + 박스 4종 (의사결정 / 리스크 / scope-out / invariant).
- `example.md` + `example.html` — 입력/출력 미리보기 (이메일 로그인 plan).
- MIT 라이선스.

### 설계 학습 — Lean 으로 가는 over-engineering 3 사이클

내부 dogfood 에서 3번의 over-engineering 사이클을 거친 후에야 lean 한 v0.1.0 에 도달. 미래 contributor 가 같은 함정 안 빠지게 박제:

**사이클 1 — Prescriptive 함정**: 첫 초안은 Plan 10 강제 섹션 + Design 12 강제 섹션. LLM 이 입력 무관 *모든* 섹션 채우려 함 → 빈 placeholder 텍스트로 가득 찬 비대한 HTML 산출. **Fix**: Hero 제외 모든 강제 섹션 제거. 케이스별 선택 가이드로 교체.

**사이클 2 — Feature creep 함정**: 두 번째 초안에 Executive Summary 카드, light/dark 토글, print CSS, mobile TOC 햄버거, 인터렉티브 와이어프레임 상태 탭, progressive disclosure 추가. 총 480 → 1100 줄. **Fix**: 다크 단일 테마 + 박스 4종 + Mermaid CDN 으로 축소. Print 는 OS 영역. 상태 탭은 진짜 데모일 때만 가치.

**사이클 3 — Structure 함정**: `templates/` `scripts/` `docs/` `examples/` 폴더 분리. *폴더 자체* 가 LLM 에 *"이 구조 강제"* 신호 + 13 파일 산만. **Fix**: 모든 파일 루트로. 7 파일 / 폴더 0개. `scripts/hash-source.sh` + `check-drift.sh` 는 SKILL.md 안에 한 줄 명령으로 박제 (별도 wrapper 불필요).

**원칙 증류**: 스킬 지침은 LLM 에게 **prescription** 이 아니라 **advice** 여야 함. 강제 섹션은 *"폼 채우기"* 행동을 유발하고, 결과가 자유 메뉴 선택보다 *측정상* 나쁨. 폴더 분리는 강제 신호. 단일 SoT 한 파일에 다 박제하는 게 LLM 활용에 최적. 같은 이유로 총 스킬 본체 ~500 줄 이내 유지.

기능 추가 유혹 받으면 자문: *"기존 패턴 메뉴 (박스 4종 / Mermaid 4종 / 와이어프레임 / 카드 / 표) 가 이미 커버하지 않나?"* 보통 yes.

### 의도적으로 안 넣은 것

- ❌ Light 모드 토글 — 다크 단일, 유지보수 ↓
- ❌ Print CSS — OS 가 처리
- ❌ 모바일 TOC 햄버거 — 1300px 이하 TOC 숨김, 스크롤 fallback
- ❌ 인터렉티브 와이어프레임 상태 탭 — 정적 1-2 프레임이면 충분
- ❌ 특정 plan-reviewer / PDCA 도구와의 tight 통합 — 내부에서만 의미, OSS 는 standalone
- ❌ 명령 후 auto-gate — host 환경에서 wire-up 할 영역
- ❌ 별도 폴더 (`templates/`, `scripts/`, `docs/`) — LLM 에 강제 신호. 모든 파일 루트로 평탄화
