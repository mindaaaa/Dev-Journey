---
title: 3월 5주차 주간 계획 및 피드백 정리
date: 2025-03-31
tags:
  - project
  - plan
  - retrospective
  - feedback
  - react
  - virtual-scroll
  - frontend
project: hanzawa-kanji
type: weekly-log
draft: false
---
# 피드백 
##  ✨ FE 관련

- [x] 한자 카드 구성 정리:
  - [x] "정자체"는 음·훈 아래 위치하도록 수정
  - [x] **표시 순서: 훈 → 음** 으로 통일
- [x] API 주소는 `.env`로 분리 예정
  - 현재는 `constant.js` 파일에 상수로 추출해 관리
  - [x] `BASE_API_URL`, `QUIZ_API_URL` 등 이름 명확히해 `함수`로 분리

##  ✨ BE 관련

%% - [ ] 무한스크롤 대응:
  - virtual scroll 대비용으로 백엔드 API에 `cursor 기반 로직` 추가 %%
- [ ] 랜덤 퀴즈:
  - 퀴즈 ID가 `1`부터 시작하는 문제 → 로직 수정 필요 #랜덤성 

##  기술 성장 및 문서화 피드백 정리

- [ ] `useCallback` 등 훅 사용 이유 → 포스팅/정리 주제
  - [ ] "왜 굳이 써야 하지?" → 성능 최적화 관점 정리
- [x] 스프린트 여유 시간에 **기술 포스팅 or TIL 남기기**
  - [x] 글 쓰기 어렵다면 **포스팅 주제 키워드라도 메모하기**
- [x] 데일리 회고가 힘들면 `주간 회고`라도 꼭 블로그/TIL에 정리!
  - [ ] GitHub 블로그 또는 Notion/Velog 활용

## 스프린트 기준

`월~토` 
- [ ] 중간 점검일 하루 정해서 `피드백/리팩토링 상태 점검`
- [ ] 점검일에는 `전체 코드 흐름` 복기 및 간단 `이슈 정리`

## 🗂 포스팅 주제 모음 
#임시저장

- [ ] `useCallback`을 언제 왜 써야 하는가?
- [ ] 무한스크롤과 Virtual Scroll, 무엇이 다른가?
- [ ] `IntersectionObserver`의 모든 것 (disconnect와 최적화, 왜 통곡의 벽😂)
- [ ] `React` 상태 로직 설계 시 생각해야 할 조건
- [ ] `API cursor`와 `offset`의 차이점

---

# 주간 계획

`목표` Step 1. 피드백 수용 및 Virtual Scroll 학습

- [x] 기존 무한스크롤 구조에서 observer 예외 케이스 정리
- [x] Virtual Scroll 개념 재정리 (`itemSize`, `height`, `itemCount` 등)
- [x] `react-window` vs `react-virtuoso` 특징 비교
- [x] `FixedSizeList` 컴포넌트 샘플 코드 복습
- [x] 주차별 일지 작성

`목표` Step 2. Virtual Scroll 기능 도입

- [x] `StudyMode` 리스트를 `FixedSizeList`로 교체
- [x] 기존 `items.map()` 제거 → `react-window` 렌더 방식으로 변경
- [x] `style` prop 전달 확인 📌
- [x] 카드 높이 측정하여 `itemSize` 정확히 설정
- [x] 마지막 카드 도달 감지 → `fetchKanji()` 연동

`목표` Step 3. Virtual Scroll 최적화 및 디버깅

- [x] 스크롤 도달 이벤트 보완 `onItemsRendered`
- [x] 중복 fetch 방지 (`loading`, `cursor` 조건 체크)
- [ ] 성능 체크 `렌더링 수 제한이나 FPS 저하`
- [x] observer 제거 (`IntersectionObserver` 제거)
- [ ] 필요 시 fallback UI `로딩 카드, 빈 화면 처리`
---
`목표` 유한 모드 퀴즈 기본 기능 구현 

- [x] `/quiz?mode=limited` 페이지 진입 시 문제 출력
- [x] 문제 리스트 `mock 데이터` or `API`로 10개 불러오기
- [x] `KanjiCard` 컴포넌트 그대로 재사용
- [x] 정답 확인 버튼 → flip 처리 (`flipped` 상태 관리)
- [x] 다음 문제 버튼 (`quizIndex` 증가)
- [x] 현재 문제 번호 / 전체 문제 수 표시 `진행률`
- [ ] 정답 상태를 저장하는 구조 만들기 (`correctList`, `wrongList`)
- [ ] 퀴즈 종료 시 결과 요약 화면 or 메시지 출력
- [ ] 정답을 본 뒤에는 `Next`만 누를 수 있도록 버튼 조건 분기
- [ ] 빠르게 누르는 문제 방지 (`debounce`, 중복 방지 처리)
- [ ] 상태구조 간단 리팩토링 → 다음 단계 대비