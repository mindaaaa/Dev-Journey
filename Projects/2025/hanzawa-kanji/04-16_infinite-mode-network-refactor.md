---
title: 무한 모드 개선 및 공통 네트워크 로직 리팩토링
date: 2025-04-16
tags: [react, refactor, fetch, infinite-mode, network, ui, frontend]
project: hanzawa-kanji
type: log
draft: false
---
`목표` 무한모드 디벨롭 및 코드 리팩토링 

## 구현 목록

- 무한모드 기능 개선
- 세 가지 모드에서 사용되는 공통 네트워크 로직을 리팩토링

## 📌 작업 항목

### ♾️ 무한모드 개선

- [x] **무한모드 중간 종료 기능 추가**  
  → 사용자가 버튼을 눌러 언제든지 무한모드에서 빠져나올 수 있도록 구현  
  → `종료하기`, `그만 풀기`, `돌아가기` 등의 UI 제공 고려

- [x] **중간 종료 시 결과 요약 표시**  
  → 사용자가 종료 버튼을 누르면 지금까지의 정답 수/오답 수/정답률 등 요약 정보를 출력  
  → 기존의 `ResultSummary` 컴포넌트 재활용 or 확장

### 🛠 try-catch 분리 및 네트워크 로직 개선

- [x] **공통 네트워크 fetch 분리**  
  → `fetchQuizList`, `fetchKanjiData` 등 공통 로직을 `hooks` 또는 `api` 유틸로 분리  
  → 세 모드(LimitedMode, StudyMode, InfiniteMode)에서는 내부에 try-catch를 두지 않고 함수만 호출
  ```jsx
// 예시: utils/api.js
export async function fetchQuizData({ quizId, mode, limit, cursor }) {
  const url = buildApiUrl({ quizId, mode, limit, cursor });
  const res = await fetch(url);
  
  if (!res.ok) throw new Error('데이터 요청 실패');
  return res.json();
}
```

- [ ] **에러 메시지와 로깅 통일화**  
  → 에러 발생 시 사용자에게 보여줄 메시지와 콘솔 에러 메시지를 정리하여 관리



