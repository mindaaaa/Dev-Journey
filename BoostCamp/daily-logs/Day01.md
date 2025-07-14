# Git과 개발환경에 뒤통수 맞고 일어선 챌린지

> 예시를 위한 코드를 작성했으며, 실제 미션과 무관한 경우가 많습니다.
## tl;dr

> 데이터 정렬 기준 구현 중 `index` 계산 오류와 리베이스 충돌 등  
> 도구(Git, 콘솔, 포맷팅)와 구조 설계 모두가 중요한 요소임을 체감했다.

---
## 1. 이번에 알게 된 내용은? 

### 1.1 로직에서 데이터 원본 순서를 유지하는 법

```javascript
const entries = Array.from(sourceMap); // 원본 복사
entries.sort((a, b) => customCompare(a, b, entries));
```

- `Map`을 `sort()` 도중 참조할 경우 **in-place 정렬**로 index 참조가 깨질 수 있음
- 외부에 `entries` 배열을 복사한 뒤 **클로저 방식으로 정렬 기준 함수에 주입**하여 해결

### 1.2 콘솔 포맷팅도 UX다

- 고정폭 글꼴 기반 표를 만들기 위해 `.padEnd()`, `.padStart()` 활용
- 강조 텍스트는 `chalk` 또는 ANSI 코드 활용

### 1.3 상태 관리를 객체 내부에 위임하는 방식

- 외부 관리자는 단순히 `.open(id)` 등 요청만 보내고
- 실제 변경은 내부 인스턴스가 담당 `obj.opend = true` 
- **역할 분리와 단일 책임 원칙을 반영한 구조 설계**

---

## 2. 삽질 로그 

### 2.1 의도와 다른 함수 이름과 역할 혼란

- `close()`라는 함수명이 **무언가를 출력하거나 반환할 것처럼 느껴짐**
- 실제로는 단순히 종료 상태를 `true`로 바꾸는 함수여야 하는데, 혼란이 있었음

> 출력은 별도의 함수에서 담당하도록 **책임 분리**

### 2.2 정렬 중 배열 참조 문제

```javascript
const index = entries.findIndex(([time]) => time === currentTime);
```
- 정렬 중 참조 대상 배열이 `in-place`로 변경되며 **순서가 꼬임**
- 정렬 대상 배열을 별도 복사하여 전달하는 방식으로 구조 변경

### 2.3 Git 리베이스 충돌 → 파일 증발 

- 디렉토리 구조를 변형하는 과정에서 Gist에서의 경로 충돌 발생
- 해결 중 `git rebase`, `--continue`, `--abort` 등 사용 시 **충돌 해결 실패**
- 결국 파일이 증발하고, 직접 복구하거나 다시 작성해야 했음
- 추후 주말에 에러 로그 분석해 학습정리 업데이트 예정🔥

---

## 3. 이번 미션이 남긴 것

| 항목  | 내용                                          |
| --- | ------------------------------------------- |
| 정렬  | 단일 기준이 아닌 **다단계 비교** 구현 시, 기준 유지와 순서 보존이 중요 |
| Git | `rebase`, `reset`, `stash` 사용 시 사전 백업은 필수   |
| 설계  | 상태와 역할을 분리하는 설계는 **예측 가능성과 테스트 용이성**을 높임    |
| 포맷팅 | 콘솔 출력도 설계의 일부, 가독성은 디버깅 효율과도 직결됨            |

---

## 참고 자료

- [Git Rebase (ko)](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)
- [MDN - Array.sort() 정렬 방식](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
- [MDN - Map 순서 보장](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [chalk - 콘솔 컬러링](https://github.com/chalk/chalk)