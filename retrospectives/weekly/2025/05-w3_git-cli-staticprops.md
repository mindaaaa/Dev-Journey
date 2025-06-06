---
title: mini-git CLI 구현과 mindapresso 정적 페이지 설계
date: 2025-05-22
tags:
  - weekly
  - retrospective
  - cli
  - git
  - nextjs
  - getStaticProps
  - gray-matter
  - jest
  - fs
  - path
  - project-design
summary: CLI 환경에서 Git 원리를 구현하고, Next.js 기반 블로그 프로젝트에서 정적 데이터 흐름을 이해하며 구조 중심 사고를 강화한 한 주
draft: false
---

`기간` `2025-05-12` ~ `2025-05-20`

## 이번 주에 한 일

| 구분       | 내용                                                             |
| -------- | -------------------------------------------------------------- |
| 주요 작업    | - `mindapresso` 글 목록 조회 기능 구현<br>- `mini-git` 내부 동작 이해 및 구현 완료 |
| 테스트 학습   | - 제로초 Jest 강의 수강 `1장` Jest 배워보기<br>- 테스트 도구와 CLI 환경 흐름 분석      |
| 알고리즘 루틴  | - 프로그래머스 문제 풀이                                                 |
| 학습 계획 조정 | - `모던 자바스크립트 딥다이브` 완독 연기<br>- 알고리즘 학습을 위한 LeetCode 진입 전략 수립    |
| 기타       | - 현대차 부트캠프 소프티어 정보 탐색<br>- 향후 코딩 테스트 준비 방향 재설정                 |

## 주요 작업

### `mindapresso` 글 목록 조회 기능 구현

- `gray-matter`를 활용해 `.md` 파일에서 메타데이터(작성일, 제목, 태그 등)를 추출
- `getStaticProps`로 정적 데이터 패칭 → `PostList` UI에 props로 전달
- Next.js의 SSR 구조와 `pages/` vs `app/`의 차이 이해

### 배운 점

- **Next.js에서의 구조적 개념들**
  - `pages/` 디렉토리는 기존 라우팅 중심 구조
  - `app/` 디렉토리는 13버전 이후 도입된 새로운 App Router 방식으로,<br> **레이아웃 중첩**, **서버 컴포넌트**, **로딩 상태** 등 현대적인 기능을 기본 지원함
- **`@`와 `slug` 사용**
  - `@`: 경로 별칭 (alias) 기능<br>`tsconfig.json`에 등록하면 `../../../` 지옥 탈출 가능📌
  - `slug`: 글 URL 식별자, 글 제목이나 ID를 URL-safe하게 표현한 것 (e.g., hello-world)

### 📌 구현 흐름도

```plaintext
posts/hello.md         ← 마크다운 원본
↓ (gray-matter)
lib/posts.ts           ← 글 목록 데이터 추출
↓
getStaticProps         ← 빌드 시점에 props로 넘김
↓
pages/index.tsx        ← 목록 렌더링
↓
PostList + PostCard    ← UI 구성
```

---

### `mini-git` Git 내부 원리 구현

- Git의 핵심 개념 (`init`, `add`, `commit`)을 직접 구현
- Git 내부 구조를 이해하기 위해 `.mini-git/` 디렉토리 내 객체 저장 로직 직접 구현

#### 🙅 막혔던 부분

- Git의 자료구조 (`blob`, `tree`, `commit`)와 해시 기반 저장 방식의 개념 이해에만 4~5일 소요
- CLI 특성상 **도메인 중심의 설계**가 익숙하지 않아 구조화에 애를 먹음
- `fs`, `path` 모듈로 파일 시스템을 다루는 것도 처음이었고, 테스트 환경에 따라 **경로 처리 이슈** 발생

| 디렉토리명     | 설명                                    |
| --------- | ------------------------------------- |
| `core/`   | Git 핵심 기능 모듈 (`add`, `commit`, `log`) |
| `domain/` | Git 객체 추상화 (Blob, Tree 등)             |
| `utils/`  | 파일 읽기/쓰기, 해시 생성 등 유틸 함수               |
| `cli/`    | 명령어 실행 진입점, `argv` 파싱 등 사용자 인터페이스     |

> Git의 해시 기반 객체 모델과 포인터 시스템을 이해하는 데 상당한 시간이 걸렸지만,<br>전체 흐름을 구조화하고 나니 각 기능 구현은 비교적 간결하게 마무리할 수 있었다.<br>
> 복잡한 구현보다 중요한 건 **구조적 이해**였고, <br>이번 스프린트를 통해 Git의 내부 철학과 설계 원리에 더 가까워질 수 있었다.

> Git은 단순한 파일 저장 구조가 아니라 해시값을 포인터처럼 사용해 객체를 추적하고 조합하는 시스템이다.

### 배운 점

#### Jest와 CLI 테스트 환경 충돌

- CLI 프로젝트는 `resolve()`와 `path.join()`으로 `.mini-git` 경로를 하드코딩하면, <br>Jest는 Node.js의 현재 작업 디렉토리(`process.cwd()`)와 충돌할 수 있음
- Jest는 테스트 환경에서 루트 디렉토리를 달리 인식함<br>
  → **`basePath`를 옵션으로 분리**하여 전달

```javascript
function add(filename, options = {}) {
  const basePath = options.basePath || '';
  const filePath = path.join(process.cwd(), basePath, filename);
  ...
}
```

> 테스트 환경과 CLI 환경을 **분리된 경로 전략**으로 통합 관리하면서 확장성과 테스트 가능성 모두 개선했다.

---

## 이번 주의 가장 어려웠던 점

### `mini-git`의 내부 동작 원리 파악

- Git에서 `add` → `commit` → `log`로 이어지는 흐름에서 파일이 어떻게 **스테이징되고, 해시화되며, 커밋 객체로 변환**되는지 이해하는 데 며칠 소요됨
- 특히 CLI 프로젝트는 UI가 없고 로직 중심이기 때문에 설계를 시각화하기 어려웠음🤯
- 디버깅 도중, <br>경로 오류, `fs.readFileSync()` 에러, 절대/상대 경로 충돌 등 예상 외의 이슈 다수 발생🧨

---

## 이번 주를 돌아보며

### 자아성찰

- `기분`: 😆
- `컨디션`: 너무 좋아
- `한줄 피드백`: 이렇게만 매주 디벨롭해가면서 방향을 잡아나가자

### 한 주 요약

> 이번 주는 **개념 중심 사고와 구조적 이해를 바탕으로 실전 구현에 반영한 주**였다.  
> 단순히 돌아가는 코드가 아닌, 왜 그렇게 되는지까지 고민하며 성장했다.  
> 특히 **Next.js 구조 이해**, **CLI 기반 Git 구현**, **테스트 환경 분리 설계**까지 모두 의미 있는 학습이었다.

### 다음 주 목표

- [ ] `mindapresso` 상세 페이지 렌더링 구현
- [ ] `mini-git` `log` `HEAD` `branch` 구현
- [ ] `mini-git` 테스트 코드 개선 및 Jest 설정 안정화
- [ ] 프로그래머스 1단계 마무리
- [ ] 알고리즘 공부 2단계 루틴 설계
- [ ] 모던 자바스크립트 딥다이브 `비동기 및 이벤트처리` 이해하기
