# 견적 대시보드 — 인수인계 (다른 기기에서 이어서 개발용)

> 새 세션의 Claude에게: 이 문서가 곧 인수인계다. 다 읽으면 사용자에게 긴 설명 요구하지 말고 바로 이어서 작업할 것.
> 앱 소스는 이 저장소의 `index.html` 하나가 전부다. 원문: https://raw.githubusercontent.com/clean-office/quote/main/index.html

## 1. 이게 뭔가
- 사용자(성원)는 인테리어 실측·영업을 하는 **모두랩(moredolab)** 운영자. 밖에서 폰으로 견적서를 만들어 고객에게 카톡으로 보내는 용도.
- 회사 엑셀 견적 양식("템플릿 1")을 웹앱으로 옮긴 것. 시트 3장 구성: **갑지**(표지: 로고+명함+금액) / **집계표**(공종별 합계+간접비) / **내부내역**(품목 상세).
- 배포됨: https://clean-office.github.io/quote/ (GitHub Pages, repo `clean-office/quote`, 브라우저 로그인으로 운영 — 별도 토큰 없음)

## 2. 양식 규칙 (변경 금지, 사용자 확인 없이 바꾸지 말 것)
- 계산: 품목금액 = 수량×재료단가 + 수량×노무단가 → 공종 소계 → 직접공사비
- 간접비 = 직접공사비 × (공사경비 2.5% + 공과잡비 2.5% + 기업이윤 5%) — 각각 개별 계산·표시, 요율은 UI에서 조정 가능
- 합계 = 직접 + 간접 − 단수정리. 단수정리는 기본 만원 미만 절사, "목표 총액" 입력 시 그 금액에 정확히 맞춤
- 갑지 금액 표기: 一金 ○○○원整 (한글 변환 함수 hangulNum 내장)
- 내보내기 JPG 파일명: `{공사명}_1_갑지.jpg` / `_2_집계표` / `_3_내부내역`, 엑셀은 `{공사명} 견적서.xlsx`
- V.A.T 별도가 기본 견적조건

## 3. index.html 구조 (단일 파일, ~726줄 / base64 이미지 포함 ~125KB)
- **에디터 UI = 4단계 스텝형** (2026-07 디자인 리뉴얼 반영):
  - 상단 진행바(4칸)+라벨(클릭 이동), 하단 고정 바(1~2단계 직접공사비 / 3~4단계 합계 + 이전·다음 버튼)
  - ①기본정보(+텍스트 임포트 접이식) ②공종·품목(카드형: 품명·규격 1행 + 단위/수량/재료/노무 2×2 그리드 + 금액 자동) ③간접비 요율·단수정리·계산 요약 ④미리보기·내보내기(시트 3장)
  - 모바일 max-width 440px, 390px 가로 스크롤 없음
- 상태: 전역 `state` {title, client, date, terms, r1, r2, r3, target, sections:[{name, items:[{name,spec,unit,qty,mat,lab}]}]} — **구조 이전과 동일(호환)**
- 스텝 이동: `goStep/nextStep/prevStep/updateStepUI`. 전역 `curStep`
- 주요 함수: calc() / hangulNum() / renderSections()(스텝2 카드 렌더) / buildPreview()(시트3장 720×509 HTML) / exportJPG()(html2canvas scale:3) / exportXLSX()(ExcelJS, 수식+이미지 3시트 — **리뉴얼에서 손대지 않고 원본 유지**) / importText() / autosave·saveNamed·openSaves (localStorage: mdl_draft, mdl_saves — **키·구조 유지**)
- 로고·명함: index.html 안에 base64 상수 `LOGO_B64`(png) / `CARD_B64`(jpeg) — 엑셀 갑지에 삽입됨. 웹 시트 3장은 **텍스트 워드마크**("mo/do 골드 · re 회색 · lab 검정") 사용
- 외부 라이브러리: cdnjs html2canvas 1.4.1, ExcelJS 4.4.0 / 폰트 Pretendard(jsdelivr CDN)

### 견적서 시트 디자인 토큰 (가로형 720×509, 고정 — 앱 테마와 무관)
- 골드 액센트 #B07E1E (명함 색) / 본문 검정 #1c1c1c / 소계 배경 #faf6ec / 인장 원 #c33 / 푸터 "INTERIOR STUDIO · SEOUL / WE DO MORE"
- 갑지=수신·공사명·유효기간+一金 한글+총액 크게+인장 / 집계표=공종합계+A직접·간접요율·B간접·단수정리+합계바(품목없음) / 내역서=품목상세+공종소계→"직접공사비 계"(간접비·합계는 집계표 참조)

### 앱 골드 테마 토큰 "1b" (CSS :root 변수)
- 페이지 #f8f5ee / 카드 #fffdf8(테두리 #e6dfcd) / 본문 #2b2417 / 액센트 #8B6914 / 하단바 #2b2417 / 카드 6px·입력 4px

## 4. 다음 할 일 (2단계 백로그 — 사용자와 우선순위 확인)
1. Supabase 연동: 견적 저장·불러오기 기기 간 동기화 (프로젝트 있음 — URL·anon 키는 사용자 PC `클린 오피스/00_프로젝트_기록/키모음_비공개.md`. **키를 이 공개 repo에 절대 넣지 말 것.**)
2. 로그인(비밀번호 게이트 또는 Supabase Auth) — 지금은 주소 아는 사람 누구나 열람 가능
3. 단가·품목 사전(자주 쓰는 품목 탭 한 번에 추가)
4. 폰 실사용 피드백 — JPG/엑셀 다운로드가 폰 브라우저에서 잘 되는지 실사용 확인

## 5. 배포 방법 (수정 후)
- github.com 브라우저 로그인(계정 clean-office) → repo `quote` → Upload files(`/upload/main`)로 index.html 덮어쓰기 → Commit → 1~2분 뒤 Pages 반영
- 또는 index.html 연필(Edit)로 내용 교체. Pages 설정 이미 켜짐(main / root, .nojekyll 있음)
- **클라우드 세션 주의**: Claude가 크롬으로 커밋할 때 파일 업로드는 세션의 `/mnt/user-data/outputs/` 에 둔 파일만 GitHub Upload에 올릴 수 있음(임의 경로·uploads 폴더는 거부됨). 데스크톱 브리지 불안정 시 이 경로 사용.

## 6. 사용자 작업 스타일 (중요)
- Claude가 할 수 있는 건 전부 Claude가 직접 한다 (파일 처리·브라우저 조작·배포 포함). 사용자에게 "해보세요" 금지 — 로그인·비밀번호 입력만 요청
- 결과물이 웹페이지면 URL만 주지 말고 크롬으로 띄워서 보여줄 것
- 응답은 간결하게. 완료 후 이 문서(HANDOVER.md)도 최신 상태로 갱신해서 repo에 같이 커밋할 것

## 7. 변경 이력
- 2026-07-16~17: 1단계 완료·배포 (엑셀 양식 → 웹앱, 시트3장·엑셀·JPG·텍스트임포트·localStorage)
- 2026-07-19: **디자인 리뉴얼 배포** (commit 95adc9e) — 입력 UI를 4단계 스텝형+골드테마로, 견적서 3장을 가로형 확정 디자인으로 교체. 계산·단수정리·엑셀 생성·localStorage·로고/명함 base64·JPG 파일명 전부 원본 유지. 헤드리스 검증: 직접 927,500 / 간접 92,751 / 단수 −251 / 합계 1,020,000 정확, 390px 가로스크롤 없음.
