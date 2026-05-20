# Plan: 이메일 + 비밀번호 로그인

> 복잡도: **Simple**

## Why

현재 소셜 로그인만 지원 (Google / GitHub). 내부 분석상 **가입 시도의 15% 가 소셜 제공자 화면에서 이탈** — 3rd-party 권한 부여 원치 않는 유저나 해당 계정 없는 유저. 이메일/비밀번호 추가 시 이 이탈 대부분 회복 가능.

## Success Metrics

| 지표 | Before | After 목표 | 측정 방법 |
|---|---|---|---|
| 가입 완료율 | 62% | 75%+ | 퍼널 분석, 주간 평균 |
| 첫 로그인 도달 시간 | 42초 (소셜) | <30초 (이메일) | 페이지 timing 이벤트 |

## Decision

| 옵션 | 장 | 단 | 권장 |
|---|---|---|---|
| A. bcrypt cost = 12 | 더 안전 | 로그인 지연 ~300ms | |
| B. bcrypt cost = 10 | 지연 <100ms | 약간 약함 | ✓ (2024+ 업계 default) |

## DB

```sql
CREATE TABLE auth_passwords (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  hash TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_auth_passwords_user ON auth_passwords(user_id);
```

## API

### POST /api/auth/signup
입력: `{ email, password }`
출력: `{ user, session }` (201) 또는 `{ error }` (409 — 이메일 중복)

### POST /api/auth/login
입력: `{ email, password }`
출력: `{ user, session }` (200) 또는 `{ error }` (401)

## 사용자 흐름

1. 유저가 /login 진입 → 이메일/비밀번호 입력란 + "Google 로 로그인" 버튼
2. 이메일/비밀번호 입력 → "로그인" 클릭
3. 프론트 zod 검증 (이메일 형식, 비밀번호 8자+)
4. POST /api/auth/login
5. 백엔드가 auth_passwords 에서 user_id 로 조회, bcrypt.compare
6. 성공: 세션 쿠키 set, /dashboard 리다이렉트
7. 실패: "이메일 또는 비밀번호가 일치하지 않습니다" 인라인 에러

## UI

로그인 화면 (default):
- 이메일 입력
- 비밀번호 입력 (눈 아이콘으로 표시/숨김 토글)
- "로그인" 주 버튼
- "비밀번호 찾기" 링크 하단
- "or" 구분선
- "Google 로 로그인" 버튼 (기존)
- 하단 "계정 없으신가요? 가입하기" 링크

로딩 상태: 버튼 스피너, 입력란 disabled.
에러 상태: 입력란 위 빨강 배너 + 메시지.

## Risk

- **P0**: bcrypt cost 너무 높음 → 로그인 지연 → 유저가 앱 느리게 인지. **대응**: cost=10, staging 에서 p95 지연 측정 후 출시.
- **P1**: 비밀번호 재설정 흐름 이번 PR 미포함 → 비밀번호 까먹은 유저 막힘. **대응**: "비밀번호 찾기? 고객센터 문의" 임시 링크 노출.

## 이번 PR 안 함

- 2FA / TOTP — 다음 PR
- 매직 링크 — backlog
- 비밀번호 재설정 — 다음 PR (임시: 고객센터 이메일)
- Rate limiting — 별도 보안 PR

## 변경 파일

| 파일 | 변경 |
|---|---|
| `supabase/migrations/{ts}_add_auth_passwords.sql` | NEW — 테이블 + 인덱스 |
| `src/app/api/auth/signup/route.ts` | NEW — POST 핸들러 |
| `src/app/api/auth/login/route.ts` | NEW — POST 핸들러 |
| `src/features/auth/login/ui/LoginForm.tsx` | NEW — 폼 컴포넌트 |
| `src/features/auth/login/model/login-schema.ts` | NEW — zod 스키마 |
| `src/app/(auth)/login/page.tsx` | EDIT — LoginForm 추가 |
