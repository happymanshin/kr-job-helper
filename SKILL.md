---
name: kr-job-helper
description: 국내채용도우미 — 사람인·잡코리아·인크루트·원티드·캐치·링크드인·워크넷 등 국내 채용사이트를 한 번에 훑어 내 이력·조건에 맞는 공고를 고르고, 추천 이유와 링크가 든 보고서를 메일로 보낸다. "채용공고 찾아줘", "국내 채용사이트 다 뒤져봐", "나한테 맞는 공고", "/kr-job-helper"라고 하면 사용.
---

# 국내채용도우미 (kr-job-helper)

여러 국내 채용사이트에서 공고를 모아 사용자 프로필과 대조하고, 상위 후보를 추천 이유와 함께 보고한다. 지원서 제출은 하지 않는다. 후보 탐색과 보고까지만 한다.

## 0. 시작 전: 프로필 확인
**기본 방식: 사람인에 로그인된 사용자의 이력서를 읽어 프로필로 쓴다.** ego의 고정 공간 `saramin`에서 `https://www.saramin.co.kr/zf_user/resume/resume-manage`(이력서 관리)를 열어 대표 이력서의 경력·학력·희망조건·연봉을 `innerText`로 읽고, 아래 3종 파일로 정리한다(읽기 전용, 수정 금지). 로그인이 안 돼 있으면 로그인 화면을 띄우고 `handOff()` 후 안내한다. 사람인 이력서가 없거나 희망조건이 비어 있으면 질문으로 채운다.
프로필 파일 3종이 작업 폴더에 있어야 한다. 이미 있으면 그대로 쓰되, 날짜가 오래됐으면 사람인에서 다시 읽을지 묻는다.
- `profile/resume.txt` — 이력서 전문(사실의 근거는 이것만 쓴다)
- `profile/criteria.md` — 희망 직무·직급, 연차 범위, 최소 연봉, 선호 지역·통근 한도, 제외 기업·업종
- `profile/seen.md` — 이미 보고한 공고 ID(중복 방지, 한 줄에 하나)

질문할 것: 직무 키워드, 희망 직급, 지역/통근 한도, 연봉 하한, 이미 지원했거나 제외할 기업. 이력서에 없는 사실은 지어내지 않는다.

## 1. 브라우저 규칙 (ego-browser 스킬 필수 선행)
- 작업 전 `ego-browser skill`로 가이드를 읽는다. PowerShell에서는 스크립트를 `.mjs` 파일로 쓰고 `ego-browser nodejs -e "await import('file:///…/round-01.mjs')"`로 실행한다.
- 공간 이름은 `jobs-multi`로 고정, 새 공간·탭을 만들지 않는다. 먼저 `listTaskSpaces()`로 확인하고 재사용한다. 끝나도 닫지 않는다.
- 위치·알림 권한 팝업이 뜨면 스크립트가 멈춘다(task.handOff 상태). 사용자에게 "차단"을 눌러 달라고 요청하고, 확인 후 `takeOverTaskSpace(id)`로 이어간다.
- 로그인 벽·봇 차단은 우회하지 않는다. 사용자에게 로그인 화면을 띄워 주고 `handOff()`한다.
- 한 번에 한 스크립트만 실행한다(ego 병렬 금지).

## 2. 사이트별 검색 URL (키워드 `K`는 `encodeURIComponent`)
| 사이트 | 검색 URL | 메모 |
|---|---|---|
| 사람인 | `https://www.saramin.co.kr/zf_user/search/recruit?searchword=K` | 상세는 `/zf_user/jobs/relay/view?rec_idx=ID`. 상세가 이미지뿐인 공고는 '미확인' |
| 잡코리아 | `https://www.jobkorea.co.kr/Search/?stext=K&careerType=2&careerMin=15` | 상세 `/Recruit/GI_Read/ID`. 상세 하단에 유사 공고가 많아 후보 발굴에 유용 |
| 인크루트 | `https://search.incruit.com/list/search.asp?col=job&kw=K` | 상세 `job.incruit.com/jobdb_info/jobpost.asp?job=ID`. 본문이 iframe일 수 있어 iframe 텍스트도 읽는다 |
| 원티드 | `https://www.wanted.co.kr/search?query=K&tab=position` | 대부분 연차 상한이 낮아 고경력자는 걸러진다. 상세 `/wd/ID` |
| 캐치 | `https://www.catch.co.kr/Search/SearchList?Keyword=K` | 대기업·공채 위주, 결과가 적다 |
| 링크드인 | `https://www.linkedin.com/jobs/search-results/?keywords=K&geoId=105149562&f_E=4%2C5` | 로그인 필요(사용자가 먼저 로그인). `105149562`=대한민국, 잘못된 geoId는 해외로 나온다. 외국계는 영어 요건 확인 |
| 워크넷(고용24) | `https://www.work24.go.kr/wk/a/b/1200/retriveDtlEmpSrchList.do?empTpGbcd=1&resultCnt=50&keyword=K&siteClcd=all&sortField=DATE&sortOrderBy=DESC&srcKeyword=K&careerTypes=E&pageIndex=1` | 위 형태의 URL이어야 키워드가 적용된다 |
| 잡플래닛 | `https://www.jobplanet.co.kr/search/job?query=K` | 링크가 거의 안 잡힌다. 기업 평판 확인용 |
| 블라인드 | 채용 메뉴가 공개 URL로 열리지 않음 | 못 읽으면 '미확인'으로 보고 |
| 당근 | `https://jobs.daangn.com/s?query=K` | 사용자 동네 기준 알바·기사 위주라 사무직 팀장급은 거의 없다. 위치 권한 팝업 주의 |

추출 요령: `page.evaluate`로 `a[href]` 텍스트·링크를 모으거나 `document.body.innerText`를 파일(`.json`)에 저장한 뒤, 한글 깨짐을 피하려고 **UTF-8 파일로 쓰고 Read로 읽는다**(Windows 콘솔 cp949 출력 금지).

추출 시 주의(시험 실행에서 확인): 목록 링크를 자를 때 `rec_idx`·`GI_Read/ID`·`job=`·`/wd/`·`wantedAuthNo=` 번호를 지우지 않는다. 사람인 상세는 로그인 상태에서 'AI 서류 합격률'과 지원자 통계(연령·경력 분포)가 보여 경쟁 강도 판단에 쓴다.

## 3. 절차
1. 프로필 읽기 → 키워드 3~6개 선정(예: 영업관리 팀장, 영업지원 팀장, 채권관리, 영업기획 팀장).
2. 사이트별로 키워드를 돌려 목록 수집 → 신입·인턴·계약직·알바·추심직 제외, `seen.md`의 ID 제외.
3. 후보 8~12건을 골라 상세를 읽는다. 대조 항목: 경력 상한(사용자 연차가 상한을 넘으면 감점), 직급, 연봉, 근무지/통근, 직무 일치, 마감일, 기업 위험 신호, 필수 언어·자격.
4. 상위 3~5건 선정. 억지로 채우지 않는다. 좋은 공고가 없으면 없다고 쓴다.
5. 못 읽은 사이트·키워드·항목은 '미확인'으로 명시한다.

## 4. 보고서 형식 (메일 HTML)
- 우선 추천: 공고명/기업/링크/조건/근무지/마감/**나에게 추천하는 이유**(이력 어디와 맞는지 구체적으로)/리스크
- 참고 후보, 제외한 공고와 사유, 지원 순서 제안(마감 임박 순)
- 맨 끝: 확인하지 못한 항목(급여, 기업 재무, 통근 실측 등), 지원서는 제출하지 않았다는 문구

## 5. 발송
- 받는 사람은 사용자 본인 주소만. 사용자가 요청했을 때 Gmail `send_message`로 보낸다. 다른 수신자 금지.
- 보낸 뒤 `seen.md`에 보고한 공고 ID를 추가한다.

## 금지
입사지원 제출, 로그인 대행, 계정 생성, 이력서 수정, 봇 차단 우회, 이력에 없는 내용으로 지원자를 유리하게 서술하는 것.
