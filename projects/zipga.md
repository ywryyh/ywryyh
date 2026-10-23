# 집가(ZipGa) — 술자리 미니게임 앱의 Supabase 백엔드

> 2026.08.25 – 08.31 · 6인 팀 · **백엔드 담당**
> [서비스](https://zip-ga-frontend.vercel.app) · [백엔드 레포](https://github.com/1DAN-2ZO/ZipGa_Backend) · [프론트엔드 레포](https://github.com/1DAN-2ZO/ZipGa_Frontend)

술자리 인원이 QR로 한 방에 모여 각자 폰으로 미니게임 3판을 합니다. 3판 평균이 40점 미만이면 카카오T가 뜨고 방에서 나가게 됩니다.

## 맡은 일

팀이 작성한 백엔드 명세를 받아 Supabase에 올렸습니다. 명세에는 스키마·RLS·RPC SQL이 들어 있었습니다([`mdfile/백엔드_Supabase명세.md`](https://github.com/1DAN-2ZO/ZipGa_Backend/blob/main/mdfile/%EB%B0%B1%EC%97%94%EB%93%9C_Supabase%EB%AA%85%EC%84%B8.md)). 그다음 실제 호출에서 드러난 문제를 고치고, 화면 쪽 요구에 맞춰 함수를 추가했습니다. 서버 코드 없이 Postgres(SQL·PL/pgSQL)만으로 동작합니다.

| 구분 | 내용 | 근거 |
|---|---|---|
| 명세 반영 | 테이블 4개, RLS 정책, RPC·헬퍼 함수 13개, Realtime publication | 마이그레이션 1~4 ([8bf0f87](https://github.com/1DAN-2ZO/ZipGa_Backend/commit/8bf0f87)) |
| 명세 버그 수정 | 함수 실행 권한, `search_path`, 42702 컬럼 모호성(2차례) | 마이그레이션 5~8 ([8bf0f87](https://github.com/1DAN-2ZO/ZipGa_Backend/commit/8bf0f87)) |
| 요청 반영 | 폰마다 다른 시계를 맞추는 `server_now()` | 마이그레이션 9 ([7ed3c74](https://github.com/1DAN-2ZO/ZipGa_Backend/commit/7ed3c74)) |
| 명세 이후 추가 | `check_room`(닉네임 받기 전에 방만 확인), 방장이 벌칙으로 나가면 세션 1등이 승계, 세션 시작 리드타임 5초 → 9초 | 마이그레이션 10~14 — Supabase에 배포했으나 **레포에는 미반영** |
| 운영 | Supabase 프로젝트 생성·연결, Supabase CLI로 마이그레이션 배포 | — |

## 구조 (팀 설계)

- **게임 중에는 통신하지 않음**: 각 폰이 서버가 준 시드로 같은 문제를 독립 실행하고, 판이 끝날 때 점수만 올립니다.
- **읽기는 RLS, 쓰기는 RPC**: 클라이언트는 자기가 속한 방만 읽을 수 있습니다. 모든 변경은 `security definer` 함수를 거칩니다.
- **서버에 타이머가 없음**: `start_session`이 미래 시각을 `starts_at`으로 내려 주고, 시간 판정은 클라이언트가 합니다. 만료된 방은 다음 입장 시도 때 정리합니다.
- **재제출 차단**: `scores`의 PK가 `(session_id, player_id, round_index)`라서 같은 판에 두 번 제출할 수 없습니다.

## 트러블슈팅

### 1. 권한을 회수했는데도 내부 함수가 밖에서 호출됨

- **증상**: `revoke execute ... from public` 뒤에도 클라이언트가 내부 헬퍼(`_ensure_host` 등)를 부를 수 있었습니다.
- **원인**: 이 프로젝트는 default privileges로 `anon`·`authenticated` 롤에 EXECUTE를 직접 부여합니다. 그래서 `public`에서 회수해도 롤별 권한이 남아 있었습니다.
- **해결**: 롤을 지정해서 회수했습니다. 내부 헬퍼는 모두 막고, 공개 RPC는 `authenticated`에게만 허용했습니다.

### 2. 호출할 때만 나는 42702 (ambiguous column)

- **증상**: `CREATE FUNCTION`은 성공하는데, 호출하면 42702가 났습니다.
- **원인**: `returns table (room_id, player_id ...)`로 선언한 컬럼은 함수 본문 안에서 같은 이름의 변수가 됩니다. 본문에서 `where room_id = ...`처럼 별칭 없이 쓰면 변수인지 컬럼인지 모호해집니다.
- **해결**: `join_room`·`rejoin_room`·`end_session`에 테이블 별칭을 붙였습니다. `join_room`에 남은 한 곳은 2차 마이그레이션으로 고쳤습니다.
- **이후**: 다음 날 방장 승계 규칙을 넣을 때 같은 원인으로 두 번 더 깨졌습니다. 그래서 "`RETURNS TABLE` 함수를 고칠 때 가장 먼저 의심할 곳"으로 문서에 남겼습니다.

### 3. `security definer` 함수의 `search_path` 미고정

- **증상**: Supabase Security Advisor 경고가 떴습니다.
- **원인**: `search_path`를 고정하지 않으면, 권한이 높은 함수가 호출자의 경로를 따라 다른 스키마의 객체를 참조할 수 있습니다.
- **해결**: 모든 함수에 `set search_path = public`을 지정했습니다.

## 사용 기술

`PostgreSQL` `PL/pgSQL` `Supabase(RLS · Realtime · 익명 Auth)` `Supabase CLI` · Claude Code + Supabase MCP로 작업 (커밋에 공동작성 표기)
