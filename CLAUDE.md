# SKT 35기 월드컵응원단 — 프로젝트 가이드

회사 동기 8명이 2026 월드컵 한국 조별리그 3경기를 함께 보며 쓰는 **예측 게임 웹앱**.
오너: 이동엽 (UX 디자이너, 비개발자) — 설명은 비개발자 눈높이로, 한국어로.

## 아키텍처 (단순함 유지가 원칙)

- **index.html 단일 파일** — HTML+CSS+JS 전부. 프레임워크/빌드 없음. 이 구조를 유지할 것.
- **호스팅**: Vercel ← GitHub `leeyub/worldcup` repo 연동 (main에 커밋하면 자동 배포)
- **데이터**: Supabase (`https://iutmevindlhmupgzavci.supabase.co`)
  - 테이블: `participants`(직원: name/emp/phone/emoji), `predictions`(match_id+participant_id+data jsonb), `match_state`(locked, live jsonb), `settings`(admin_pin)
  - anon key는 index.html에 하드코딩 (공개되어도 되는 키, RLS 전체 허용 정책 — 친구용 게임이라 의도된 설계)
  - 뷰 `제출현황`: 이름+한국시간으로 제출 내역 조회용
- **동기화**: 5초 폴링 (`refresh()`), 실시간 채널 안 씀

## 핵심 도메인 로직

- 경기: m1 체코(6/12 11:00) · m2 멕시코(6/18 10:00) · m3 남아공(6/25 10:00) — `MATCHES`, 선수단 `TEAMS`
- 배점(105점 만점): 결과20 · 스코어25(승패만10/한팀만5) · 양팀 첫득점자 각15 · 첫골팀5 · 시간대10(인접5) · 전반5 · 카드5 · MVP5
- 채점: `getFacts()`(라이브 입력→확정 사실) + `scorePred()`(예측 채점). **predictions.data의 jsonb 구조를 바꾸면 기존 제출과 호환이 깨지니 주의.**
- 흐름: 예측 제출 → 어드민 마감(locked) → 라이브(골/카드/전반/종료 입력) → 자동 채점·순위
- 권한: 어드민은 `settings.admin_pin` 클라이언트 비교(세션 유지), 참여자 로그인은 이름+사번+전화 3종 일치

## 디자인 시스템

- 다크 테마, CSS 변수 토큰: `--bg --card --card2 --line --pill --txt --sub --red --blue --gold --green`
- 선택 상태는 **gold로 통일**. `color-scheme:dark` 선언됨(네이티브 컨트롤)
- 서체: Pretendard (폴백 Apple SD Gothic Neo, Noto Sans KR)
- Figma 시안: https://www.figma.com/design/rRpMgp4f0XUVpEkpcjIJZl (4개 프레임 + Theme 컬러 변수)

## ?demo 모드 (디자인 검토용)

`주소/?demo`로 열면 가짜 데이터 + 하단 골드 패널로 모든 상태 전환 가능 (역할 3종 × 경기상태 7종 × 모달 × 토스트/에러/빈상태). 실제 DB 안 건드림. **UI 수정 후엔 반드시 ?demo로 상태별 확인.** 상세 체크리스트는 `상태인벤토리.md`.

## 백로그 (오너 요청, 우선순위 순)

1. **라이트 테마** — 레퍼런스: Mobbin의 Fixtured 앱 같은 "뽀얀" 느낌(웜 그레이 배경, 흰 카드, 소프트 섀도). 토큰 구조가 있으니 `:root` 라이트 팔레트 + 테마 토글로 구현
2. **결과 리더보드 리디자인** — F1 리더보드 스타일(Fixtured Formula 1 화면처럼 순위·이름·점수 정렬이 강한 레이아웃)
3. **서체 교체 실험** — SUIT, Wanted Sans 등 후보 비교
4. **비교 보드 개선** — 다수파 픽 묶어서 표시(예: "🇰🇷 승 6명"), 단독 픽만 🦄 강조
5. (선택) 로그인 시각 기록

## 운영 메모

- 어드민 운영 순서: 킥오프 직전 **예측 마감** → **라이브 시작** → 골/카드/전반종료 입력 → **경기 종료→MVP 선택**. 실수는 ↩️ 취소
- 연습 후 리셋: 어드민 패널 → "예측·경기 데이터 초기화" (직원 명단 유지)
- 이모지 컬럼 SQL 미실행 시: `alter table participants add column if not exists emoji text;`
- **브랜치 규칙**: 모든 작업은 `dev` 브랜치에서 진행한다. `main`은 운영본이며, 오너가 **"배포해줘"라고 명시할 때만** dev → main 머지한다. (그 외에는 절대 main에 직접 커밋·머지하지 않는다)
- 배포: dev에서 수정 → 오너가 "배포해줘" 지시 → main에 머지·푸시 → Vercel 자동 배포 (1~2분). 접속자는 새로고침 필요

## 주의사항

- 단일 파일 유지, 외부 의존성은 supabase-js CDN 하나뿐 — 새 라이브러리 추가 지양
- 개인정보(이름·사번·전화)가 DB에 있으니 repo에 데이터 덤프 커밋 금지
- admin_pin을 코드나 문서에 적지 말 것
- 대회 종료 후 Supabase 프로젝트 삭제 예정
