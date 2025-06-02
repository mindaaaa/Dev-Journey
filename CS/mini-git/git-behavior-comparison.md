---
title: Git Behavior Comparison – Git 내부 구조 비교 실험
date: 2025-06-02
tags:
  - git
  - internals
  - 구조비교
  - 학습프로젝트
category: git
description: Git과 mini-git의 add, commit, checkout 명령 수행 시 저장 방식과 참조 구조를 단계별로 비교하고 시각화한 학습 기반 실험 기록입니다.
draft: false
---
# 🧪 Git Behavior Comparison

> 이 문서는 Git과 mini-git이 유사한 명령어(`add`, `commit`, `checkout` 등)를 수행할 때  
> **내부적으로 어떤 구조를 공유하고, 어떤 부분이 단순화되었는지**를 비교하는 실험 기반 기록이다.  
> 각 명령어의 흐름을 따라가며 저장 구조, 포맷, 참조 방식 등을 시각적으로 정리하여  
> Git의 구조를 **직접 구현을 통해 체화**하는 데 초점을 둔다.

> ⚠️ 본 문서의 내용은 `Pro Git`, Git 공식 문서 등에서 소개된 내부 구조를 기반으로  
> 학습용으로 구현한 mini-git 프로젝트에 대한 **단순화된 실험 기록**이다.  
> 실제 Git의 바이너리 포맷 및 zlib 구현 사양과는 차이가 있을 수 있다.

---
## 1. 압축 방식 비교 

- mini-git은 Git과 동일하게 **zlib 압축**을 사용하지만,  내부 객체 포맷은 Git의 복잡한 바이너리 구조 대신  `blob 12\0<내용>` 형식의 **문자열 기반 단순 포맷**으로 구성한다.
- 단, `\0` (null 문자)로 인해 일부 텍스트 에디터에서는 해당 객체를 **바이너리로 인식**할 수 있다.
### 실험 목적

>Git과 mini-git이 동일한 파일(`objects.txt`)을 저장할 때,  
>압축 방식과 해시 결과가 일치하는지 확인한다.


![객체 디렉토리](blob_02.png)

#### 실험 ①: 동일 파일 Git에 저장
![Git 해시 출력](hash_02.png)

- Git은 빈 파일에도 `"blob 0\0"` 형태의 헤더를 붙여 SHA-1 해시를 계산한다.
- `.git/objects/e6/9de2...` 경로에 **zlib 압축된 바이너리 객체**가 저장된다.

#### 실험 ②:  mini-git으로 동일 파일 저장

![mini-Git 해시 저장](hash_03.png)

```bash
$ mini-git add objects.txt
# 내부적으로 createBlobObject → writeGitObject → addFileToIndex 실행
```

```js
// 내부 포맷 예시
const header = `${type} ${buffer.length}\0`;
const store = Buffer.concat([headerBuffer, buffer]);
```

- SHA-1 해시: `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391` → Git과 동일
- 저장 경로: `.mini-git/objects/e6/9de2...` → Git과 동일

#### 압축 및 저장 방식 비교

| 항목         | Git                  | mini-git                     | 비고                      |
| ---------- | -------------------- | ---------------------------- | ----------------------- |
| 헤더 형식      | `blob 0\0`           | `blob 0\0`                   | 동일                      |
| SHA-1 해시   | 동일                   | 동일                           | 해시 충돌 없음                |
| 압축 방식      | `zlib`               | `zlib`                       | 동일                      |
| 저장 디렉토리 구조 | `objects/ab/cdef...` | `objects/ab/cdef...`         | 동일                      |
| 파일 형식      | 바이너리 (직접 열면 깨짐)      | 문자열 기반이나 `\0`로 인해 바이너리처럼 인식됨 | ⚠️ null 문자로 에디터에서 깨짐 가능 |
### 💡 왜 mini-git 객체는 깨져 보일까?

> mini-git은 내부적으로 **문자열 포맷을 zlib로 압축**한 구조이지만,  
> Git과 동일하게 `"blob 0\0"` 형식의 **null 문자**를 포함한다.  
> 이로 인해 VSCode나 `cat` 명령어 등에서는 해당 파일을 **바이너리로 인식**하게 된다.  
> 실질적으로는 문자열 기반 구조지만, **시스템 환경에서는 바이너리로 처리될 수 있다.**

### 객체 저장 위치 예시

![Git 저장 경로](path_01.png)
![mini-git 저장 경로](path_02.png)
- 동일한 해시로 Git과 mini-git 모두 `objects/e6/9de2...` 위치에 객체를 저장한다.

---
## 2. Index 포맷 비교

- Git은 스테이징 영역(index)을 **바이너리 포맷**으로 저장하며, 파일의 메타데이터(경로, 해시, 권한, 변경 시간 등)를 포함한다.
- 반면 mini-git은 학습 목적에 맞춰 **JSON 기반의 단순한 키-값 구조**로 index를 구성한다.

### 실험 목적

>동일한 파일을 `add`했을 때,  
>Git과 mini-git이 index에 어떤 정보를 저장하는지 비교한다.

#### 실험 ①: Git에서 `add` 후 index 내용 확인

![Git index](index_02.png)
- Git은 `.git/index`에 바이너리 포맷으로 다음 정보를 저장한다.
    - 권한 (100644 등)
    - 스테이징 순서
    - blob 해시
    - 파일 경로
- index는 바이너리 형식이므로 직접 열면 깨지며, 전용 파서(`cat-file -p`)로 확인해야 한다.
![git index 바이너리](index_03.png)
#### 실험 ②: mini-git에서 동일 파일 `add`
##### index 구조 비교

```bash
$ mini-git add file.txt
# 내부적으로 createBlobObject → writeGitObject → addFileToIndex 실행
```

![mini-git index 구조](index_04.png)

| 키     | 값              |
| ----- | -------------- |
| 파일 경로 | 해당 파일의 blob 해시 |

- mini-git은 index를 **JSON 포맷**으로 저장한다.
- 사람도 직접 읽고 수정 가능한 구조로, 학습 목적에 적합하다.

##### **index 접근 방식 비교**

| 항목    | Git                  | mini-git               | 비고                           |
| ----- | -------------------- | ---------------------- | ---------------------------- |
| 저장 형식 | 바이너리                 | JSON                   | Git은 내부 최적화, mini-git은 학습 목적 |
| 파일 위치 | `.git/index`         | `.mini-git/index.json` | 구조 유사                        |
| 접근 방법 | `ls-files`, 내부 파서 필요 | 직접 열람 가능               | mini-git이 가시성 높음             |
| 저장 정보 | 해시, 권한, 경로, 타임스탬프 등  | 파일명 - 해시 (단순 구조)       | mini-git은 핵심 정보만 저장          |
| 가시성   | 직접 열면 깨짐             | 텍스트 기반으로 쉽게 확인 가능      | 학습용 설계                       |
#### 💡 왜 Git은 바이너리 index를 사용할까?

>Git은 성능 최적화를 위해 index 파일을 바이너리 포맷으로 설계한다.  
>이를 통해 빠른 탐색, 권한/타임스탬프 추적, 충돌 탐지 등을 효율적으로 수행할 수 있다.  
>반면 mini-git은 **핵심 동작 원리를 학습하기 위해** JSON 기반의 단순 구조를 채택한다.

### `index` 저장 예시

```json
{
"objects.txt": "e69de29bb2d1d6434b8b29ae775ad8c2e48c5391"
}
```

- 이 구조는 이후 `commit` 시 tree 객체 생성에 직접 활용된다.
- mini-git은 Git처럼 "index → tree" 흐름을 유지하되, 구현은 직관적으로 단순화되어 있다.

### 📌 핵심
- Git의 index는 commit 전의 스냅샷을 고성능으로 관리하기 위해  정밀한 메타데이터와 바이너리 구조를 사용한다.
- mini-git은 핵심 흐름(파일 → blob → index → commit)을 단순하게 체험하기 위해  **파일명-해시만 저장하는 JSON 포맷**을 채택한다.
- 결과적으로 두 시스템 모두 index를 거쳐 commit 객체를 구성하지만,  **Git은 실무 최적화**, mini-git은 **학습 최적화**를 목표로 설계된다.

---

## 3. 포인터 시스템 (HEAD, refs)

- Git은 `HEAD` 포인터와 `refs` 디렉토리를 통해 브랜치와 커밋 간의 참조 구조를 유지한다.
- mini-git은 동일한 개념을 따르되, 저장 형식과 오류 처리 방식에서 차이를 보인다.

### 실험 목적

Git과 mini-git의 `HEAD`, `refs/heads` 구조를 비교하고  
동일 브랜치를 생성했을 때 어떤 방식으로 참조가 구성되는지 확인한다.

### 실험 ①: Git에서 `branch` 명령 실행

```bash
$ git branch dev
```
- `HEAD` 파일에는 기본적으로 `ref: refs/heads/main` 텍스트가 저장된다.
- `refs/heads/dev` 파일이 생성되며, 현재 커밋 해시가 기록된다.

```plaintext
.git/
├── HEAD                # ref: refs/heads/main
└── refs/
     └── heads/
         ├── main      # abc123...
         └── dev       # abc123... ← 새로 생성된 브랜치
```

- `HEAD`는 여전히 `main` 브랜치를 참조한다.
- 새로운 포인터 파일(`dev`)만 추가로 생성된다.

### 실험 ②: mini-git에서 동일한 흐름 실행

```bash
$ mini-git branch dev
# 내부적으로 getCurrentCommitHash() → createBranch() 호출
```
- `.mini-git/HEAD` 파일에는 `ref: refs/heads/main`이 저장된다.
- `.mini-git/refs/heads/dev` 파일이 생성되며, 현재 커밋 해시가 기록된다.
![branch 동작](branch_01.png)

- Git과 동일한 브랜치 참조 구조를 따르며,
- 최초 커밋이 없는 경우에는 `fatal: HEAD가 가리키는 브랜치가 존재하지 않습니다`와 같은 오류를 출력한다.
![branch_error](error_01.png)
### 포인터 구성 비교
|항목|Git|mini-git|비고|
|---|---|---|---|
|HEAD 형식|`ref: refs/heads/main`|동일|동일한 텍스트 포맷|
|브랜치 참조 파일|`refs/heads/<branch>`|동일|동일한 디렉토리 구조 사용|
|커밋 해시 저장 방식|커밋 해시 단일 라인|동일|형식 동일|
|오류 처리|없음 (초기 커밋 안 해도 가능)|최초 커밋 전 에러 발생|학습 목적 에러 출력 포함|
### 💡 HEAD와 refs의 관계

> Git은 `HEAD`를 **현재 작업 중인 위치를 가리키는 포인터**로 사용하고,  
> `refs/heads/<branch>`는 **각 브랜치가 가리키는 커밋 해시를 저장하는 참조 파일**로 활용한다.  
> 이를 통해 브랜치 간 이동이나 커밋 이력 추적이 가능하다.

> mini-git은 Git의 포인터 시스템을 동일하게 모사하면서도,  
> 학습 목적에 맞춰 구조를 단순화하고 주요 흐름을 직관적으로 구현한다.

### 저장 예시
#### Git HEAD
![Git_HEAD](head_02.png)

#### mini-git HEAD
![mini-git HEAD](head_03.png)
---

## 4. Commit 포맷 비교

- Git은 `commit` 시점에 `tree`, `parent`, `author`, `committer`, `message` 등의 정보를 포함한 **텍스트 기반 포맷**을 구성한 뒤, zlib 압축하여 저장한다.
- mini-git은 이 구조를 동일하게 따르되, 학습 목적에 따라 **직접 문자열을 조합하여 저장**하는 방식을 사용한다.

### 실험 목적

> Git과 mini-git이 `commit` 명령어를 실행했을 때,  
> 어떤 형식으로 커밋 객체를 구성하고 저장하는지 비교한다.

### 실험 ①: Git 커밋 객체 포맷 확인

![Git commit](commit_02.png)
커밋 객체 내부에는 다음과 같은 요소가 포함된다:

- **tree**: 현재 상태를 나타내는 트리 객체 해시
- **parent**: 이전 커밋 해시 (최초 커밋은 생략)
- **author / committer**: 작성자 및 시간 정보
- **message**: 커밋 메시지

### 실험 ②: mini-git 커밋 객체 포맷 확인

![mini-git commit](commit_03.png)
```javascript
const commitContent = `tree ${treeHash}
${parent ? `parent ${parent}\n` : ''}
author ${author}
committer ${timestamp}
${message}

`;
```
- 커밋 객체는 문자열 템플릿으로 생성되며, Git과 유사한 포맷을 유지한다.
- SHA-1 해시를 계산한 후 `.mini-git/objects/<dir>/<hash>`에 저장한다.

#### 커밋 포맷 비교표
| 항목       | Git                                                | mini-git      | 비고          |
| -------- | -------------------------------------------------- | ------------- | ----------- |
| 저장 포맷    | 텍스트 + zlib 압축                                      | 텍스트 + zlib 압축 | 구조 동일       |
| 필드 구성    | `tree`, `parent`, `author`, `committer`, `message` | 동일            | 필드명/순서 동일   |
| 시간 포맷    | Unix 타임스탬프                                         | ISO 문자열       | 일부 형식 차이 존재 |
| 포맷 생성 방식 | 내부 구현 (C 기반)                                       | 문자열 조립        |             |
| 헤더 형식    | `commit <길이>\0...`                                 | 동일            | 저장 방식 동일    |
### 💡 커밋 객체는 어떻게 구성되는가?

> Git은 커밋 객체를 통해 **스냅샷 간의 연결고리**를 형성한다.  
> 각 커밋은 이전 커밋(parent)을 가리키고, 실제 파일 구조(tree)를 참조함으로써  
> 버전 히스토리를 선형 혹은 병렬로 구성할 수 있게 된다.

> mini-git은 이 흐름을 동일하게 따르되,  
> 텍스트 템플릿 조합으로 커밋 객체를 직접 생성함으로써  
> 구조의 핵심을 **직접 눈으로 확인하고 이해**할 수 있도록 단순화했다.

---

## 5. Checkout 동작 비교

- Git은 `checkout` 명령을 통해 **HEAD 포인터를 이동**시키고, 해당 브랜치의 커밋에 연결된 파일 상태를 워킹 디렉토리에 반영한다.
- mini-git은 HEAD 파일만 수정하고, 워킹 디렉토리를 직접 업데이트하지는 않는다.  

>학습 목적에 따라 실제 파일 변경은 생략되었지만,
>포인터 구조는 동일하게 구현되어 있다.

### 실험 목적

> Git과 mini-git에서 `checkout` 명령어가 어떤 파일을 갱신하고,  
> 내부적으로 어떤 참조를 수정하는지 비교한다.

### 실험 ①: Git에서 브랜치 전환

![checkout](checkout_01.png)
- `.git/HEAD` 내용이 `ref: refs/heads/dev`로 변경된다.
- 워킹 디렉토리의 파일이 해당 브랜치의 최종 커밋 상태로 변경된다.
- `.git/index`도 해당 커밋 시점의 index로 업데이트된다.

```plaintext
.git/
├── HEAD → refs/heads/dev
└── refs/
     └── heads/
         ├── main
         └── dev
```

> `checkout`은 HEAD가 가리키는 커밋을 기준으로 **파일, index, 포인터** 모두를 갱신하는 명령이다.

### 실험 ②: mini-git에서 `checkout`
```bash
$ mini-git checkout dev
# 내부적으로 getBranchPath → setHeadRef 실행
```
![mini-git checkout](checkout_02.png)
- `.mini-git/HEAD`가 `ref: refs/heads/dev`로 변경된다.
- `.mini-git/refs/heads/dev`에서 커밋 해시를 읽는다.
- 워킹 디렉토리는 변경되지 않는다.

```plaintext
.mini-git/
├── HEAD → refs/heads/dev
└── refs/
     └── heads/
         ├── main
         └── dev
```

>포인터는 정상적으로 이동하지만,  
>실제 파일 상태는 유지되며, **Git처럼 파일을 덮어쓰지는 않는다.**

### checkout 동작 비교표

| 항목         | Git                   | mini-git      | 비고                 |
| ---------- | --------------------- | ------------- | ------------------ |
| HEAD 이동    | `refs/heads/<branch>` | 동일            | 포인터 구조 동일          |
| 커밋 해시 조회   | `.git/refs/heads/*`   | 동일            | 파일에서 직접 읽음         |
| 워킹 디렉토리 반영 | 자동 변경                 | 변경 없음         | mini-git은 생략       |
| index 갱신   | 자동 갱신                 | 갱신 없음         | mini-git은 별도 처리 안함 |
| 목적         | 실사용 기준 상태 복원          | 구조 학습용 포인터 전환 | 학습 목적 단순화          |

### 💡 왜 mini-git은 파일을 덮어쓰지 않을까?

> mini-git은 Git 내부 구조 학습에 집중하기 위해,  
> 파일 시스템 동기화보다는 **객체 간 참조 흐름과 포인터 시스템 구현**에 초점을 맞췄다.  
> 실제 파일을 갱신하려면 tree → blob 추적 후 파일 쓰기 로직이 필요하지만,  
> 해당 기능은 구현 범위에서 제외되어 있다.

### 포인터 이동 흐름도 (mini-git)

```plaintext
HEAD → refs/heads/dev → 커밋 해시
```

> 브랜치 전환은 단순히 HEAD 포인터를 다른 브랜치로 바꾸는 작업이다.

_Last updated: 2025-06-02_  