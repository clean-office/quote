# CLAUDE.md — 모두랩 프로젝트 작업 규칙 (세션 시작 시 자동 참고)

> 이 폴더로 작업하는 Claude 세션은 이 파일을 먼저 읽고 규칙을 지킬 것.
> 상세 인수인계는 GitHub repo의 `HANDOVER.md` 참고: https://github.com/clean-office/quote/blob/main/HANDOVER.md

## 프로젝트가 뭔가
- 운영자 **성원(모두랩/moredolab)** — 1인 인테리어 실측·영업·시공 관리를 폰으로 함.
- 배포된 웹앱 2개 (repo `clean-office/quote`, GitHub Pages):
  - **견적서 추출기**: https://clean-office.github.io/quote/  (`index.html`)
  - **관리 허브**: https://clean-office.github.io/quote/hub.html  (`hub.html`)
- 소스는 이 repo의 `index.html`·`hub.html` 단일 파일 두 개가 전부. base64 이미지 포함.

## 절대 지킬 것 (변경 금지)
- **견적 계산 로직 바꾸지 말 것**: 품목=수량×재료+수량×노무 → 공종소계 → 직접공사비. 간접비=직접×(공사경비2.5%+공과잡비2.5%+기업이윤5%). 합계=직접+간접−단수정리(기본 만원미만 절사 / 목표총액 입력시 그 금액에 맞춤).
- **저장 키·구조 유지**: localStorage `mdl_draft`(작성중), `mdl_saves`(저장목록), `mdl_hub_v1`(관리앱), `mdl_cloud`(클라우드 설정).
- **견적서 3장 디자인**(갑지·집계표·내역서, 가로 720×509, 문서 골드 #B07E1E, 텍스트 워드마크)·엑셀(ExcelJS 수식·로고/명함 base64) 원본 유지.
- 사용자 확인 없이 양식·요율·계산 방식 바꾸지 말 것.

## 데이터 저장 = Supabase (영구, 이미 연결됨)
- 클라우드: Supabase 단일행 JSON. 테이블 `moredo_state(id text pk, data jsonb, updated_at)`, RLS anon 전체허용.
  - id `main` = 관리앱 데이터, id `estimates` = 견적 저장목록(합집합 병합, 안 날아감).
- **Supabase URL·anon key 등 키는 절대 이 공개 repo에 하드코딩하지 말 것.** 앱은 사용자 브라우저 localStorage(`mdl_cloud`)에서 읽음. 키 원본은 사용자 PC `클린 오피스/00_프로젝트_기록/키모음_비공개.md`.
- 새 기기는 hub 설정 탭에서 URL+anon key 1회 입력하면 연결됨(견적기도 같은 origin이라 자동 공유).

## 배포 방법 (이 방식으로 검증됨)
- github.com 브라우저 로그인(clean-office) → repo `quote` → `/upload/main`.
- **파일은 세션의 `/mnt/user-data/outputs/`에 둔 것만** GitHub 업로드 가능(임의 경로 거부).
- **커밋 버튼 주의**: 파일 수·스크롤에 따라 폼 위치가 바뀜. 업로드 후 스크린샷으로 커밋 버튼 위치 확인 → 그 좌표 클릭. 첫 시도 실패 잦으니 `commits/main`에서 반영 확인, 안 됐으면 재업로드·재커밋.
- Pages 재빌드 1~2분. github.io는 세션 샌드박스에서 차단되니 확인은 브라우저(claude-in-chrome)로.

## 사용자(성원) 작업 스타일 — 매우 중요
- **Claude가 할 수 있는 건 전부 직접 한다** (파일·브라우저·배포·Supabase 설정 포함). "해보세요" 금지.
- 사용자에게 요청해도 되는 것: **로그인·비밀번호·키 입력·폴더 연결** 정도(권한상 사용자만 가능한 것).
- 결과물이 웹페이지면 URL만 주지 말고 **크롬으로 띄워 보여줄 것.**
- **응답 간결하게.** 한국어.
- 작업 완료 후 **HANDOVER.md 갱신해서 커밋**할 것.

## 요청받은 견적서 처리 규칙 (성원이 직접 안 쓰고 "만들어줘" 할 때)
- 요청받아 만든 견적서는 **반드시 견적기 앱 저장목록에 넣어** 직접 만든 것과 한 곳에 모이게 한다. URL만 주고 끝내지 말 것.
- 저장 이름 = **현장(프로젝트)명** 으로 통일 → 이름순으로 현장별 정리됨. 같은 현장 다른 버전이면 `현장명 v2`, 날짜 구분 필요하면 `현장명 (YYYY-MM-DD)`.
- 넣는 방법: Supabase `moredo_state`의 id `estimates` 행을 **읽어서 병합**한 뒤 upsert(기존 저장분 덮어쓰지 말 것) — `data.saves['현장명'] = {t: 시각(ms), data: 견적state}`. 그러면 성원이 앱 저장목록을 열면(어느 기기든) 자동으로 뜬다.
- 견적 state 형식: `{title:현장명, client:제출처, date, terms, r1:2.5, r2:2.5, r3:5, target:"", sections:[{name:공종, items:[{name,spec,unit,qty,mat,lab}]}]}`. 계산·요율은 위 "절대 지킬 것"의 표준을 따른다.
- 다 만든 뒤 성원에게 "앱 저장목록 → 현장명 열어서 확인/수정하세요"라고 안내.

## 다음 할 일 (백로그)
1. 로그인/접근 제한(지금은 주소+키 아는 사람 열람 가능).
2. 현장 상세 강화(사진 업로드·공정 체크), 견적↔현장/고객 자동 연결.
3. 완공 포트폴리오, 캘린더·알림.
