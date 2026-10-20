# 오늘의 진짜 정보판 (T04) — 서울 현재 기온

공개 기상 자료(Open-Meteo, 키 없음)에서 서울의 현재 기온을 조회해 Asia/Seoul 날짜별로 한 건씩 저장하고,
외부 원천 실패 5종을 합성 시험값으로 재생하는 프로젝트입니다.

## 구조
- `public/index.html` — 화면(값·단위·출처·출처 시각·조회 시각·기준 시간대, 원자료·저장값·화면값 대조, 일별 기록, 실패 재생)
- `api/reading.js` — GET 저장값 조회 / POST 공개 원천 조회·저장
- `api/replay.js` — 합성 fixture 재생(서버 파일만 읽음)
- `api/_lib.js` — 정규화·검증·저장·실패 분류 (live와 replay가 같은 함수 사용)
- `fixtures/` — 공개 꾸러미의 합성 fixture 9개(원본 그대로)
- `supabase.sql` — 테이블 2개(`t04_daily` PK = signal_id+record_date, `t04_status`)
- `test/run.js` — `node test/run.js`로 fixture 전이·live 경로 검증 (ALL PASS 확인함)

## 배포 순서
1. Supabase 새 프로젝트 → SQL Editor에 `supabase.sql` 실행
2. GitHub에 이 폴더를 올림 (`.env`는 `.gitignore`로 제외됨)
3. Vercel에서 저장소 가져오기 → Environment Variables에 `SUPABASE_URL`, `SUPABASE_SERVICE_KEY` 입력 (서버에서만 사용, 화면 코드에는 없음)
4. 배포 주소를 **시크릿 창**에서 열어 로그인 없이 보이는지 확인
5. 화면 상단 "실제 조회"에서 첫날 값을 저장 → **다음 날(KST)** 다시 열어 두 번째 기록 생성

## 비밀값 검색 (C11)
```
git log -p --all | grep -iE "service_key|eyJ[A-Za-z0-9_-]{20,}|sb_secret"
grep -rniE "service_key|eyJ[A-Za-z0-9_-]{20,}" public api fixtures
```
브라우저 개발자 도구 Network 탭에서 `/api/reading` 응답에 키가 없는지도 확인하세요.

## 실패 상태 매핑 (live)
| 상황 | error_code |
|---|---|
| 4초 안에 응답 없음 | timeout |
| 401/403 | auth |
| 429 | rate_limit |
| 연결 실패, 5xx | offline |
| JSON 아님·필드 누락·검증 실패 | schema_error |

실패해도 일별 기록은 지우지 않고 상태만 `stale`로 바꿉니다.

## 제출문 초안 (본인 확인 후 수정)
**짧은 확인 방법**
1. 어디로 가나요: 제출한 결과물 주소를 시크릿 창에서 엽니다.
2. 3단계 이내 무엇을 하나요: ① 화면 맨 위 실제 조회 값을 확인 → ② "실패 재생"에서 초기화 → 정상 D1-A, D1-B 순서로 누름 → ③ "느림" 등 실패 버튼을 누르고 "다시 시도"를 누릅니다.
3. 무엇이 보이면 통과인가요: 값·단위·출처·출처 시각·조회 시각·Asia/Seoul이 보이고, 실패 뒤에도 105가 "오래된 값"으로 남고, 복구 후 fresh/none·값 120·기록 2건이 됩니다.
4. 안 될 때 무엇이 보이나요: "서버에 연결하지 못했습니다" 또는 실패 종류별 안내 문구와 "다시 시도" 버튼이 보입니다.

**AI와 나의 판단**
1. AI에게 맡긴 일: 코드 초안(조회·저장·재생·화면)과 테스트 작성, 제출 항목 정리
2. 내가 직접 판단한 일: (직접 작성) 예: 공개 원천 선택, 배포 환경, 실제 조회 결과가 맞는지 대조
3. AI 제안을 따르지 않은 일: (직접 작성, 없으면 "없음 — 이유: …")
