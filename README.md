# plan-visualize

plan-visualize는 [Claude Code](https://claude.com/claude-code) 스킬입니다. plan/design 마크다운 문서를 HTML 한 페이지로 변환하여 검토 시간을 줄입니다.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 설치

프로젝트에 추가:

```bash
git clone https://github.com/CodeAlpacat/claude-plan-visualize \
  .claude/skills/plan-visualize
```

모든 프로젝트에서 사용하려면 `~/.claude/skills/plan-visualize` 경로에 설치합니다.

## 사용

Claude Code 내에서:

```
/plan-visualize plan login
```

`docs/01-plan/features/login.plan.md` 등의 경로를 자동으로 탐색하여 옆에 `login.plan.html` 을 생성합니다.

경로를 직접 지정할 수도 있습니다.

```
/plan-visualize docs/임의경로/내-plan.md
```

design 문서는 `/plan-visualize design <이름>`.

## 예시

`example.md` 가 입력, `example.html` 이 결과입니다. 브라우저로 열어 확인할 수 있습니다.

## 변환 규칙

마크다운에서 다음 패턴을 감지하여 시각 요소로 변환합니다.

| 마크다운 패턴 | 변환 결과 |
|---|---|
| `## Risk` 섹션 | 색상 박스 (P0 빨강 / P1 노랑) |
| "A vs B" 비교표 | 의사결정 박스 (보라) |
| "이번에 안 함" 섹션 | scope-out 박스 (회색) |
| `CREATE TABLE` SQL | Mermaid ER 다이어그램 |
| 사용자 흐름 (액션 → 결과) | Mermaid sequence 다이어그램 |
| 상태 전이 | Mermaid state 다이어그램 |
| UI 변경 명세 | HTML/CSS 와이어프레임 |

마크다운에 없는 정보는 생성하지 않습니다.

## 재생성

마크다운 수정 후 동일 명령을 재실행합니다. 생성된 HTML 상단에 source-hash 가 기록되어 있어 변경을 자동으로 감지합니다.

직접 수정한 HTML 은 다음 실행 시 덮어쓰여집니다.

## 구성

```
plan-visualize/
├── SKILL.md
├── template.html
├── example.md
├── example.html
├── README.md
├── CHANGELOG.md
└── LICENSE
```

## 요구사항

- Claude Code v2 이상
- 브라우저 (Mermaid 가 CDN 에서 로드되므로 첫 실행 시 네트워크 필요)

## 기여

이슈 / PR 환영합니다. `CHANGELOG.md` 에 의도적으로 추가하지 않은 기능 목록이 있으니 참고 바랍니다.

## 라이선스

[MIT](LICENSE)
