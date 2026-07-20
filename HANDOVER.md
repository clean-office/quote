# 모두랩 관리 웹 — 인수인계 (다른 기기에서 이어서 개발용)

> 새 세션의 Claude에게: 이 문서가 곧 인수인계다. 다 읽으면 사용자에게 긴 설명 요구하지 말고 바로 이어서 작업할 것.
> 앱 소스는 이 저장소의 파일들이 전부다. 원문: https://raw.githubusercontent.com/clean-office/quote/main/

## 0. 지금 상태 (2026-07)
- 사용자(성원)는 인테리어 실측·영업·시공 관리를 폰으로 하는 **모두랩(moredolab)** 1인 운영자.
- 두 개의 배포된 웹앱 (repo `clean-office/quote`, GitHub Pages, 브라우저 로그인 운영 — 토큰 없음):
  - **견적서 추출기**: https://clean-office.github.io/quote/  (`index.html`)
  - **관리 웹 허브**: https://clean-office.github.io/quote/hub.html  (`hub.html`) — 홈·영업·현장·정산·고객·단가·설정 + 견적기 연결
- **가장 중요한 다음 작업 = Supabase 영구저장 실제 활성화.** 코드는 이미 다 들어가 있고(hub.html 설정 탭), 사용자가 ①Supabase SQL 1회 실행 ②URL·anon key 입력만 하면 켜진다.

## 1. 견적서 추출기 (index.html, ~740줄, base64 이미지 포함 ~127KB)
- 4단계 스텝형 UI + 브랜드 골드 테마. ①기본정보 ②공종·품목(카드형) ③간접비·단수정리 ④미리보기·내보내기(가로형 시트 3장 720×509).
- 계산: 품목=수량×재료+수량×노무 → 공종소계 → 직접공사비. 간접비=직접×(공사경비2.5%+공과잡비2.5%+기업이윤5%) 개별표시. 합계=직접+간접−단수정리(기본 만원미만 절사 / 목표총액 입력시 정확히 맞춤). **이 계산 로직·localStorage(mdl_draft,mdl_saves) 구조 변경 금지.**
- 시트: 갑지(一金 한글+총액+인장) / 집계표 / 내역서. 워드마크 텍스트("mo/do 골드·re 회색·lab 검정"), 문서 골드 #B07E1E.
- 내보내기: exportJPG() html2canvas scale:3 — **폰이면 navigator.share(공유시트→사진앱 저장), PC면 다운로드** 자동 분기. exportXLSX() ExcelJS 3시트+수식+로고/명함 base64(원본 유지). 파일명 `{공사명}_1_갑지`/`_2_집계표`/`_3_내부내역`.
- **단가사전 연동**: 로드시 localStorage `mdl_dict_pick` 있으면 그 품목들을 공종별 섹션으로 자동 추가 후 pick 비움(importDictPicks).

## 2. 관리 허브 (hub.html, ~52KB)
- 프리미엄 디자인: 에스프레소 슬림 사이드바(라인 아이콘) / 상단바 / 반응형(≤860px 하단 탭바+세로스택). 골드 포인트, dataviz 원칙(얇은 마크·상태색 분리·스파크라인·막대차트).
- 라우팅: 해시(#home,#sales,#sites,#quotes,#settle,#customers,#dict,#settings). render()가 view함수 디스패치.
- 데이터: 전역 `DB` = {sites,sales,customers,dict,tasks,_ts}. localStorage `mdl_hub_v1`(+_bak). seed()로 예시데이터. save()마다 _ts 스탬프+cloudPush().
- 모듈: 홈(KPI·월별수주차트·오늘할일·진행현장·수금 전부 DB에서 계산 calc()/monthChart()) / 영업 칸반(단계 이동, '계약'시 현장 자동생성) / 현장(추가·상태·진행률·수금 d/m/f 30·40·30) / 정산(현장 미수 표) / 고객 / 단가사전(공종별, '견적에 담기'→mdl_dict_pick) / 설정.
- **클라우드 동기화(Supabase, 단일행 JSON)**: 설정에서 URL+anon key 입력→cloudConnect(). 테이블 `moredo_state(id text pk, data jsonb, updated_at)`. 읽기=REST GET id=eq.main, 쓰기=upsert(Prefer merge-duplicates). save()→cloudPush() 디바운스, init→cloudPullMerge()(updated_at 최신 우선, last-write-wins). config는 localStorage `mdl_cloud`. 배너가 연결상태 표시. **anon key는 앱에 넣는 공개용 클라이언트 키(RLS로 보호). 지금 정책은 anon 전체허용=주소+키 아는 사람 열람가능(백로그: 로그인).**

## 3. 다음 할 일 (우선순위)
1. **Supabase 실제 연결 테스트** — 사용자가 SQL 실행+키 입력 후, 실제 프로젝트에서 읽기/쓰기/기기간 동기화 동작 확인. (샌드박스는 supabase.co 차단이라 stub으로만 검증됨.)
2. 로그인/접근제한 (지금 주소 아는 사람 열람 가능)
3. 현장 상세 페이지 강화(사진 업로드·공정 체크·메모), 견적↔현장/고객 자동 연결 강화
4. 완공 포트폴리오, 캘린더/알림

## 4. 배포 방법 (중요 — 이 세션에서 검증된 방법)
- github.com 브라우저 로그인(clean-office) → repo `quote` → `/upload/main` → **파일은 세션의 `/mnt/user-data/outputs/`에 둔 것만** GitHub 업로드 가능(임의 경로·uploads 폴더 거부됨. 데스크톱 브리지 불안정시 이 경로).
- **커밋 버튼 주의**: 파일 여러 개 올리면 폼이 아래로 밀려 좌표가 바뀐다. 업로드 후 **스크린샷으로 커밋 버튼 위치 확인 → 그 좌표 클릭**. 첫 시도 실패가 잦으니 commits/main에서 반영 확인, 안 됐으면 재업로드+재커밋. (이 세션에서 index/hub 배포 이렇게 성공.)
- Pages 재빌드 1~2분. github.io는 샌드박스에서 차단 → 확인은 브라우저(claude-in-chrome)로.

## 5. 사용자 작업 스타일 (중요)
- Claude가 할 수 있는 건 전부 직접 한다(파일·브라우저·배포). "해보세요" 금지 — 로그인·비번·Supabase 키 입력만 요청.
- 결과물이 웹페이지면 URL만 주지 말고 크롬으로 띄워 보여줄 것. 응답 간결하게. 완료 후 이 문서 갱신 커밋.

## 6. 변경 이력
- 07-16~17: 견적기 1단계(엑셀→웹앱).
- 07-19: 견적기 디자인 리뉴얼(스텝형+가로시트) / 폰 JPG=사진앱 저장.
- 07-21: **관리 허브 hub.html v1** (홈·영업·현장·정산·고객·단가) + **Supabase 클라우드 동기화 코드** + **단가사전→견적기 연동** + 예시데이터 비우기. (commit 9379425 외)
