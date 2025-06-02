---
title: Git Internals – Git 내부 구조와 핵심 객체 이해
date: 2025-06-02
tags:
  - git
  - internals
  - snapshot
  - blob
  - tree
  - commit
  - head
  - refs
  - version-control
category: git
description: Git의 내부 저장 구조와 핵심 객체(blob, tree, commit 등), HEAD, index, refs 등의 역할과 구성 원리를 정리한 문서
draft: false
---
# 📦 Git Internals

> 이 문서는 Git의 내부 저장 방식과 핵심 객체 구조(blob, tree, commit 등),  
> `.git/` 디렉토리의 구성 요소(HEAD, index, refs 등)를 정리한 문서입니다.

_Last updated: 2025-06-02_ 

---
# `Git`이란?

> 분산 버전 관리 시스템`DVCS, Distributed Version Control System`으로<br>소스 코드의 변경 이력을 기록하고 협업에서 충돌 없이 브랜치를 나눠 개발할 수 있도록 도와줌

- 변경 내용을 **스냅샷(snapshot)** 단위로 저장
- 파일이 아닌 `프로젝트 전체 구조(Tree)`를 기준으로 관리
- 저장소 내부는 `.git/` 디렉토리에 있는 `객체 기반 구조(Blob, Tree, Commit)`로 구성
- **SHA-1 해시 기반**으로 파일의 무결성(integrity)과 중복 제거를 자동 처리

# `Git` 내부 구조

> `git init` 명령어를 실행하면 현재 디렉토리에 `.git/`이라는 폴더가 생성<br>이 폴더는 Git이 **버전 관리**를 수행하는 데 필요한 다양한 구성요소를 포함

## `.git` 디렉토리 구조

```tree
.git/
├── HEAD
├── config
├── description
├── hooks/
├── info/
├── objects/
│   └── ...
├── refs/
│   ├── heads/
│   └── tags/
└── index
```

## 주요 파일 및 폴더

### 1. `HEAD`

- **현재 내가 작업 중인 브랜치나 커밋**을 가리킴<br> → 일반적으로 `ref: refs/heads/main`처럼 **브랜치 참조**<br> → 특정 커밋으로 `checkout`하면 **커밋 해시를 직접 가리킴** `Detached HEAD 상태`

#### 📌 예시

![HEAD](head_01.png)
→ 현재 `main` 브랜치를 가리키고 있음<br>→ 이때, `git checkout`으로 브랜치를 바꾸면 `HEAD` 포인터 이동
### 2. `.git/refs/`

- **Git이 어떤 커밋 해시를 참조하고 있는지**를 저장하는 구조
- 태그(tag)나 원격 브랜치(remote)도 포함

```tree
.git/
└── refs/
    ├── heads/           # 로컬 브랜치들의 HEAD 포인터
    │   ├── main         # main 브랜치가 가리키는 커밋 해시 저장
    │   └── feature-a
    ├── tags/            # 태그도 특정 커밋을 가리키는 포인터
    │   └── v1.0
    └── remotes/         # origin/main, origin/dev 같은 원격 브랜치
        └── origin/
            ├── main
            └── dev
```

| 폴더            | 설명                               |
| --------------- | ---------------------------------- |
| `refs/heads/`   | 로컬 브랜치들이 가리키는 커밋 해시 |
| `refs/tags/`    | 태그가 가리키는 커밋 해시          |
| `refs/remotes/` | 원격 저장소의 브랜치들 (읽기 전용) |

### `objects/`

> Git의 모든 데이터 저장소

- **blob, tree, commit, tag** 객체들이 **SHA-1** 해시 이름으로 저장

```bash
.git/objects/ad/cdef12... # 실제 커밋이나 파일 내용 저장
```

- Git은 변경사항이 아니라 **전체 파일 내용**을 `snapshot`처럼 저장
- 해시 기반으로 중복 내용은 저장하지 않음

---

# Git의 핵심 객체

> Git은 저장소의 내용을 **Blob → Tree → Commit** 구조로 구성하여  
> 파일, 폴더, 프로젝트 전체의 상태를 계층적으로 관리

```tree
Commit
 └── Tree (루트 디렉토리)
      ├── Tree (하위 디렉토리)
      │     └── Blob (파일 내용)
      └── Blob (파일 내용)
```

## 1. `Blob`

### 역할

> 파일의 실제 내용을 저장하는 Git 객체

- 오로지 `파일 내용`만 저장
- **파일 이름과 위치 정보**는 Tree가 기억

### 생성 방식

1. `git add <파일>` 실행` → 파일 내용을 읽음
2. `blob <바이트 수>\0<내용>` 형태로 포맷
3. SHA-1 해시 생성 후 `.git/objects/`에 저장

##### 🤔 데이터가 똑같으면 해시도 같나요?

![blob](blob_01.png)
디렉토리 구조와 관계 없이 `파일 내용`만 저장하기 때문에 <br>파일 내용이 동일하면 같은 해시가 반환된다.

## 2. `Tree`

### 역할

> Git의 **디렉토리 구조**를 표현하는 객체<br>
> 하위에 있는 `Blob`(파일)이나 또 다른 `Tree`(하위 폴더)를 **포함하는 목록**
> 파일명,<br>모드(권한), 해시 등 **위치 정보 + 연결 정보**를 담고 있습니다.

### 생성 방식

> 하위 디렉토리가 있으면, **Tree 안에 또 Tree가 중첩**되어 저장됩니다.

1. `git commit`시 스테이징된 Blob을 확인
2. 각 파일/디렉토리마다 `<파일 모드> <타입> <해시> <이름>` 작성
3. 전체 목록을 정렬해 `tree` 포맷 문자열 생성
4. **SHA-1** 해시 계산 후 `.git/objects/`에 저장

### 구조

![tree](tree_01.png)

```text
<파일 모드> <타입> <해시> <파일명>
```

```text
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    hello.txt
```

- `100644` → 일반 파일 권한
- `blob` → 저장된 객체 타입
- `e69de2...` → 해당 blob의 해시
- `hello.txt` → 실제 파일 이름

## 3. `Commit`

### 역할

> Git 저장소의 **하나의 스냅샷**을 의미<br>
> 해당 시점의 _Tree, 부모 커밋, 작성자, 메시지_ 등<br>모든 이력 정보(메타데이터 포함)를 담는 객체

### 생성 방식

1. `index`를 읽어 **Tree 객체** 생성
2. 메타데이터 수집
3. Tree 해시, parent 커밋 해시, 작성자를 포함한 포맷 구성
4. SHA-1 계산 후 `.git/objects/`에 저장
5. `HEAD`가 이 커밋을 가리키도록 업데이트

### 구조

![commit](commit_01.png)

- `tree`: 이 커밋이 가리키는 디렉토리 구조
- `parent`: 이전 커밋 → 최초 커밋이라 생략
- `author`, `committer`: 작성자 및 기록자
- 메시지: 커밋 설명

---
# Git의 포인터 시스템과 이력 탐색

Git은 단순히 객체를 저장하는 것만이 아니라,<br>그 객체들을 어떤 순서로 가리키고 이동할지를 **포인터 기반 구조**로 관리
```plaintext
HEAD → refs/heads/main → 커밋 해시
                       ↳ parent → parent ...
```

## 주요 구성 요소

| 구성 요소        | 역할                         |
| ------------ | -------------------------- |
| HEAD         | 현재 작업 위치 (브랜치 or 해시)       |
| refs/heads/* | 각 브랜치가 가리키는 커밋 해시          |
| parent       | 커밋 간 연결 고리 → Git 로그 순회에 사용 |

> 따라서 Git의 log는 단순한 출력이 아니라,  
> HEAD에서부터 parent를 따라가는 **역사 추적 알고리즘**

## 1. `HEAD`

### 역할

>현재 작업 중인 브랜치 혹은 커밋을 **가리키는 포인터**

### 생성 방식

| 상황              | 내용                                   |
| --------------- | ------------------------------------ |
| `init` 시        | `ref: refs/heads/main` 이라는 한 줄짜리 텍스트 |
| `checkout` 시    | `HEAD` 내용을 다른 브랜치로 변경                |
| `Detached HEAD` | 특정 커밋 해시를 직접 기록함                     |
```plaintext
ref: refs/heads/main      ← 브랜치 기반 HEAD
a1b2c3d4e5f6g7h8...        ← Detached HEAD (커밋 직접 참조)
```

## 2. `refs/heads/<브랜치명>`

### 역할

>브랜치 이름이 가리키는 **가장 최신 커밋 해시**

### 생성 방식

- 최초 `commit`시 HEAD가 가리키는 위치를 기준으로 만들어짐
- 이후 커밋이 발생할 때마다 **최신 해시**로 덮어써짐

### 디렉토리 구조

```bash
refs/heads/
├── main          # 커밋 해시: a1b2c3...
└── dev           # 커밋 해시: d4e5f6...
```

```bash
f9a0bc1a73ef10ed18a6293... # 파일 내부
```

## 3. `parent`

### 역할

> 해당 커밋이 **어떤 커밋의 후속**인지를 가리키는 포인터

#### 1. Git log에서 부모 → 부모 → 부모 식으로 따라가기
```plaintext
HEAD → refs/heads/main → 커밋(a1b2c3)
                          ↓
                     parent → 커밋(123456)
                                ↓
                           parent → ...
```

#### 2. merge시 다수의 parent를 가질 수 있음 `merge commit`
```plaintext
*───A───B───C       ← main
         \      
          D───E     ← feature
```

```plaintext
# main 브랜치에서 실행
$ git merge feature
```

```plaintext
*───A───B───C───────────M       ← main
         \           /
          D───E─────┘          ← feature
```

- 커밋 `M`은 `C`와 `E` 두 개의 커밋을 **부모로 가지게 됨**

### 생성 방식

- `commit` 명령 실행 시, HEAD가 가리키는 커밋이 있다면 `parent` 필드에 포함<br>→ 따라서 최초 커밋은 `parent` 필드가 없음

```plaintext
tree <트리해시>
parent a1b2c3d4e5f6
author minda <email> timestamp
committer ...
```

## 👀 mini-git에서는 어떻게 구현됐을까?

| 기능     | 구현 방식                                                    |
| ------ | -------------------------------------------------------- |
| HEAD   | - `HEAD` 파일 안에 `ref:`<br>- 커밋 해시 직접 기록 **detached**      |
| 브랜치    | - `refs/heads/<branch>` 파일 생성<br>- 커밋 해시 저장 **detached** |
| parent | `commitContent` 문자열 내부에 `parent ${parentHash}` 포함        |
| log    | 구현 중...🛠️                                               |

---

# Git의 흐름

## 📌 흐름 요약

```text
init  →  add  →  commit
         ↓         ↓
      Blob      Tree + Commit
```

| 단계       | domain 객체 | 역할                                                          |
| ---------- | ----------- | ------------------------------------------------------------- |
| **add**    | `Blob`      | 파일 내용을 해시화하여<br>`.mygit/objects/해시`로 저장        |
| **commit** | `Tree`      | 여러 파일(blob)을 디렉토리 구조로 묶음                        |
|            | `Commit`    | 트리 + 메시지 + 부모 커밋 + 시간 등을<br>포함하는 최종 스냅샷 |

## 디렉토리

```tree
Commit
 └── Tree (루트 디렉토리)
      ├── Tree (docs/)
      │    └── Blob (docs/readme.md)
      └── Blob (main.js)
```

> 디렉토리는 *하위 Tree*나 *Blob을 포함하는 Tree*로 구성

```text
.
├── app.js
└── src/
    └── util.js
```

```text
Commit
 └── Tree (.)
      ├── Blob (app.js)
      └── Tree (src)
           └── Blob (util.js)
```

- 루트 Tree는 `app.js`라는 Blob과 `src/`라는 Tree를 포함
- `src/` Tree는 `util.js`라는 Blob을 포함

## 명령어

### 1. `git init`

> 현재 디렉토리에 `.git/` 폴더를 생성하고,  
> Git이 동작하는 데 필요한 내부 구조를 초기화하는 명령어

#### 생성되는 `.git/` 디렉토리 구조

```tree
.git/
├── HEAD             ← 현재 브랜치를 가리키는 포인터
├── config           ← 저장소 설정 정보
├── description      ← 설명 (bare repo용, 일반 repo는 무시)
├── hooks/           ← 커밋/푸시 시 실행되는 스크립트
├── info/            ← 무시할 파일 목록 등 정보
├── objects/         ← Git 객체(blob, tree, commit 등) 저장소
└── refs/            ← 브랜치와 태그가 가리키는 커밋 참조
     └── heads/      ← 로컬 브랜치들
     └── tags/       ← 태그들
```

1. `.git/` 폴더를 생성
2. HEAD 파일 생성<br>→ 기본적으로 `ref: refs/heads/main`
3. `objects/`, 'refs/heads', 'refs/tags' 등 하위 디렉토리 생성
4. `config`, `description` 등의 기본 설정 파일 생성

  <br>

### 2. `git add`

> 파일의 내용을 읽어 Blob객체로 저장하고,<br>스테이징(index)에 파일명과 blob 해시를 기록

1. 파일 내용을 읽음 → `hello.txt`
2. `blob <크기>\0<내용>` 형식으로 포맷
3. **SHA-1** 해시 계산
4. `.git/obejects/ab/cd1234...` 형식으로 **zilb 압축** 저장
5. `index`에 파일명과 해시를 매핑해 기록

```json
// 바이너리 형태가 아닌 직관적인 표현
{
  "hello.txt": "e69de2..."
}
```

<br>

### 3. `git commit -m "메시지"`

> 스테이징된 파일 목록(index)을 기반으로
> **Tree 객체와 Commit 객체를 생성**하고
> HEAD가 이 커밋을 가리키도록 업데이트

1. `index`를 읽어 스테이징된 파일 해시 확인
2. 파일명과 `blob` 해시 계산<br>→ `tree` 객체 생성
3. 현재 시간과 작성자, 메시지를 포함<br>→ `commit` 객체 생성

#### 📌 Tree 생성

- index의 blob 목록을 정리하여 **디렉토리 구조 표현**
- `<파일 모드> <타입> <해시> <파일명>`
![tree](tree_01.png)
- 하위 폴더가 있다면 **Tree 안에 또 다른 Tree 포함**

#### 📌 Commit 생성

- 생성된 Tree 해시를 포함하여 Commit 객체 생성
- 커밋 메시지, 작성자, 부모 커밋 등 **메타데이터를 포함**

```text
tree <tree-id>
parent <commit-id>
author <이름> <이메일> <시간>
committer ...
<메시지>
```

![[commit_01.png]]
##### 👀 Commit 메타데이터란?

> *메타데이터*란, 데이터를 설명해주는 **정보에 대한 정보**를 가리킵니다.

<br>

커밋 객체는 내용`blob`보다 이 커밋이 <br>**어떤 스냅샷을 나타내며 누가 언제 만들었는지**에 대한 정보에 집중

| 항목        | 설명                              | 메타데이터 여부 |
| ----------- | --------------------------------- | --------------- |
| `tree`      | 이 커밋이 참조하는 Tree 객체 해시 | ✅              |
| `parent`    | 부모 커밋 ID (없으면 최초 커밋)   | ✅              |
| `author`    | 작성자 이름, 이메일, 시간         | ✅              |
| `committer` | 커밋 기록자 정보                  | ✅              |
| 메시지      | 커밋 메시지                       | ✅              |
| Blob 내용   | 파일의 실제 데이터                | ❌              |

따라서, **작성자나 메시지, 시간** 무엇 하나 달라져도 해시가 달라짐

> Git의 SHA-1 해시는 **내용과 메타데이터**를 기반으로 계산

<br>

#### 📌 전체 흐름

```plaintext
[Working Directory]
   ↓ git add
[Staging Area (Index)]
   ↓ git commit
[Git Object DB (objects/)]
   → blob, tree, commit 생성
   ↓
[HEAD] → 브랜치 → 커밋 가리킴
```

#### 👀 왜 VSCode에서 commit/blob이 안 열리나요?

```bash
# mini-git의 `blob` 파일이 들어갈 곳
```

> "The file is not displayed because it is either binary or uses an unsupported text encoding."

- blob 객체는 내부적으로 다음과 같이 저장
```plaintext
blob 1234\0파일내용
```

- `\0`은 **null 문자**기 때문에 텍스트 에디터는 `바이너리`라고 판단
- 실제 **Git**과 동일한 구조를 따름
	- Git의 blob 객체는 **zlib 압축된 바이너리**
	- Git의 blob 파일도 **에디터로 직접 열면 깨짐**

```bash
$ git cat-file -p <blob_hash> # 이렇게 열어볼 수 있다
```

### 4. `git branch`

> 현재 HEAD가 가리키는 커밋을 기준으로 새로운 브랜치를 생성하는 명령어

1. `HEAD` 파일을 열어 현재 브랜치가 참조하는 커밋 해시를 가져옴<br>→ `ref: refs/heads/main`<br><br>→ `.mini-git/refs/heads/main` 파일 열기
2. 해당 커밋 해시를 기준으로 새로운 브랜치 파일 생성<br><br>→ `.mini-git/refs/heads/feature/login`<br><br>→ 커밋 해시 저장

```bash
HEAD
└── ref: refs/heads/main

refs/heads/
├── main            # 기존 브랜치
└── feature/login   # 새로 만든 브랜치 (같은 커밋 해시)
```

#### ⚠️ fatal: HEAD가 가리키는 브랜치 refs/heads/main가 존재하지 않습니다.

```bash
# mini-git의 `fatal 에러` 이미지 들어갈 곳
```

>최초 커밋 전에 브랜치를 생성하면 오류 발생<br>→ `HEAD`가 가리키는 커밋이 없기 때문

##### 커밋 전에 브랜치를 만들면...
- `.mini-git/refs/heads/main`이 존재하지 않음<br>→ 현재 브랜치의 커밋 해시를 읽을 수 없음

```bash
# 실제 git의 `Not a valid object name` 이미지 들어갈 곳
```

##### 👀 mini-git에서는...
```plaintext
fatal: HEAD가 가리키는 브랜치 refs/heads/main가 존재하지 않습니다
fatal: 'HEAD'는 유효한 커밋이 아닙니다
```

1. HEAD는 `ref: refs/heads/main`을 가리킨다
2. `.mini-git/refs/heads/main` 파일이 없다 → 브랜치가 존재하지 않음
3. `getCurrentCommitHash()`가 `null` 반환

> `createBranch()`에서도 **커밋 해시가 없다고** 판단

##### 📌 해결법

>`init` 후 최초 `commit`을 실행한다.<br>→ **커밋**을 해야 실제 `.mini-git/refs/heads/main` 파일이 만들어짐

### 5. `git checkout`

>HEAD 포인터를 특정 브랜치로 전환하여 작업 위치를 변경

1. 인자로 받은 브랜치 이름으로 `refs/heads/<branch>` 경로 탐색
2. 해당 브랜치가 존재하면 `HEAD` 파일을 해당 브랜치를 참조하도록 변경<br>→ `ref: refs/heads/dev`

```bash
HEAD
└── ref: refs/heads/dev
```

#### 👀 mini-git에서는
- 존재하지 않는 브랜치 이름 → `fatal` 에러 출력
- HEAD가 직접 커밋 해시를 가리키는 경우 → `Detached HEAD` 상태 처리

### 6. `git log`

> HEAD에서 시작해 부모 커밋을 따라가며 커밋 이력을 출력

1. 현재 브랜치를 기준으로 HEAD가 가리키는 커밋 해시 조회
2. 해당 커밋 내용을 파싱하고 출력
3. `parent`가 존재하면, 계속 따라가며 커밋 로그 출력
```shell
commit 4f0a... (HEAD -> main)
Author: alice <alice@example.com>
Date:   Thu May 29 09:51:18 2025 +0900

    최초 커밋 메시지
```

#### 👀 mini-git에서는
```javascript
while (currentHash) {
  const raw = readObject(currentHash, gitDir);
  const parsed = parseCommitObject(raw);

  console.log(`commit ${currentHash}`);
  console.log(`Author: ${parsed.author}`);
  console.log(`Date:   ${formatGitDate(parsed.timestamp)}`);
  console.log(`\n    ${parsed.message}\n`);

  currentHash = parsed.parent;
}
```

- `.mini-git/refs/heads/<branch>` → HEAD 해시 추적
- 각 커밋 객체는 `parent` 필드를 통해 연결
- 커밋 시간은 `+0900` 형태로 출력 포맷 지정 (`formatGitDate`)

### 7. `git cat-file -p <해시>`

> Git 객체의 내용을 압축 해제해 출력 (blob/tree/commit)

- SHA-1 해시로 `.mini-git/objects/??/??????` 파일 탐색
- zlib 압축 해제 후 헤더(`type size`)와 본문 파싱
- 객체 유형에 따라 다르게 출력

#### 👀 mini-git에서는

```javascript
const GitObjectPrinter = {
blob: (body) => console.log(body.toString('utf-8')),
commit: (body) => console.log(body.toString('utf-8')),
tree: (body) => {
let i = 0;
while (i < body.length) {
const modeEnd = body.indexOf(32, i);
const mode = body.slice(i, modeEnd).toString();
i = modeEnd + 1;

const nameEnd = body.indexOf(0, i);
const name = body.slice(i, nameEnd).toString();
i = nameEnd + 1;

const hash = body.slice(i, i + 20).toString('hex');
i += 20;

console.log(`${mode} ${name} ${hash}`);
		}
	},
};
```

#### 출력 예시
![출력 예시](cat_01.png)
- Git과 동일한 `blob`, `tree`, `commit` 구조 재현
- 각 객체는 zlib 압축되어 바이너리 처리되었으므로 직접 확인 불가
- `catFile()` 함수는 Git처럼 압축을 해제하고 객체 유형에 맞게 출력 처리

---

# 해시와 무결성

```text
중세 유럽에서 한 기사가 왕의 명령서를 전달하러 갑니다.

양피지에 내용을 작성한 뒤,
이를 접고 밀랍(실링왁스)으로 봉인한 후,
왕실의 인장을 꾹 눌러 찍어 수신자에게 전달합니다.

편지를 받은 수신자는 먼저 실링이 온전히 유지되고 있는지를 확인함으로써
편지가 중간에 훼손되거나 위조되지 않았음을 신뢰하게 됩니다.
```

> SHA-1 해시는 **인장**과 같은 역할을 수행<br>
> 이를 통해 내용의 변경 여부를 검증하고 무결성을 보장

## 왜 Git은 해시를 사용하는가?

- `Git`은 모든 내용을 해시값으로 추적

```text
파일 내용 → SHA-1 해시 생성 → 저장
```

- 따라서 Git은 해시값만 비교해도 파일의 변경 여부를 검증할 수 있음

## `Git`과 `SHA-1` 해시

#### SHA-1이란?

> 입력 데이터로부터 **160비트 해시값**을 생성하는 단방향 해시 함수

```text
.git/objects/<해시 앞 2자리>/<해시 나머지 38자리>
```

> Git은 객체를 SHA-1 해시를 기준으로 식별하며
> 이 해시를 기반으로 `.git/objects/`에 저장

- 해시가 `57ba11cdd2e0cbcf1fe9939f4e8f7b214d20bb78`
- 저장 경로는 `.git/objects/57/ba11cdd2e0cbcf1fe9939f4e8f7b214d20bb78`

#### 🤔 왜 하필 `SHA-1`이었을까?

#### 장점

`Git`이 처음 만들어질 때 `SHA-1`은 **빠르면서 충돌 가능성이 매우 낮은 알고리즘**

- SHA-1의 공식 발표는 1995년
- `Git`의 초기 출시는 2005년
  당시 보안 요구보다 <br>**더 높은 수준의 무결성**을 제공했기에 채택됨

#### 🚀 SHA-256으로

- 십 년이 넘은 지금, `SHA-1`은 보안적으로 충돌이 가능해짐
- `Git`은 **SHA-256**기반의 **v2** 전환 중<br>→
  [SHA-256 전환 공식 문서](https://git-scm.com/docs/hash-function-transition)

---

## Git과 포인터

### Git의 포인터는 결국 `SHA-1` 해시를 가리킴

#### 📌 포인터

- 포인터`HEAD, 브랜치, 태그 등`는 **커밋을 가리키는 용도**
- 모든 커밋은 `SHA-1 해시값`

따라서 포인터는 **SHA-1**을 가리킴

![pointer](pointer_01.png)

```text
HEAD → branch → commit → tree → blob
```

---

# 스테이징 영역 `index`

> Git이 내부에 *실제 스냅샷*을 준비해두는 **중간 저장 공간**<br>
> 다음 커밋에 포함될 내용이 정리되어있다👀

- `git add`가 일어나면 그 파일은 **스테이징 영역**에 올라감
- `git commit`이 일어나면 **스테이징된 내용**만 커밋

> Git은 바로 커밋하지 않기 때문에 <br>`index` 파일에 **커밋할 것**(파일 해시, 경로, 권한)을 정리해둠

```bash
$ git add file.txt # file.txt가 index에 들어감
$ git commit # index 내용을 기반으로 커밋 생성
```

## `.git/index`

- 스테이징 영역은 `.git/index`에 **바이너리 형태**로 저장된다.
  ![index](index_01.png)

#### 바이너리란?

- `Git` 내부에서 해석할 수 있게 **인코딩된 이진 데이터 형식**
- 사람이 읽을 수 없기 때문에 `./git/index`를 확인하면 **깨진 글자**처럼 보인다.

## 📌 왜 필요할까?

- 커밋 전 **파일을 골라서 준비**하기 위해서<br>→ `index.js`만 커밋하고 `style.css`는 나중에 커밋하고 싶을 때
- **커밋할 범위**를 조절할 수 있게 만들어주는 중간 계층

---

## Git의 저장 형태

> Git은 내용과 **메타데이터**를 붙인 원본데이터를<br>**SHA-1 해시로 변환**해서 **zlib 압축된 바이너리**로 `.git/objects/`에 저장

```plaintext
Git 전용 형식 문자열 → 해싱 → 압축 후 저장
```

### 📌 흐름 예시

#### `hello.txt` 파일을 Git에 `add`하는 경우

##### 1. 파일 내용을 Git 객체로 포장

- Git의 내용에 **헤더**를 붙임

```text
blob 0\0
```

- **전체 문자열**을 기반으로 **SHA-1 해시**

```text
"blob 0\0" + ""
```

##### 2. SHA-1 해시 계산

- 이 예시에서 해시는 다음과 같이 저장되었다고 가정함

```text
e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
```

- 이 해시가 Git의 **ID이자 파일명**이 됨

##### 3. zilb 압축 후 저장

- `"blob 0\0` 포맷 데이터를 **zlib 압축**
- `.git/objects/e6/9de29b...` 형태로 저장
  - 폴더 → 해시 앞 2자리 `e6`
  - 파일명 → 나머지 해시 `9de29b...`

### 흐름 요약

```text
hello.txt 내용: ""
       ↓
Git 내부 포맷 생성: "blob 0\0"
       ↓
SHA-1 해시: e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
       ↓
zlib 압축
       ↓
.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
```

### ⚠️ `git hash-object`로 확인 가능

![hash](hash_01.png)

- 이 해시가 `.git/objects/e9/65047a...` 경로로 저장됨

> zlib 압축 결과값인 바이너리 상태로 `.git/objects` 디렉토리에 저장되지만<br> `hash-object`는 그 저장 대상의 **식별자**를 보여주기 때문에 사람이 읽을 수 있는 형태로 보임

# mini-git 구현 포인트

> `.git` 폴더 내부의 `HEAD`, `index`, `objects`, `refs`는 Git의 두뇌와 심장💓

## 구현

### 📁 디렉토리 요소

| 구성 요소    | 역할                                              |
| ------------ | ------------------------------------------------- |
| `objects/`   | `blob`, `tree`, `commit` 객체를 저장하는 디렉토리 |
| `index`      | 스테이징 영역 역할을 하는 JSON 파일               |
| `refs/heads` | 브랜치가 가리키는 커밋 ID를 저장                  |
| `HEAD`       | 현재 작업 중인 브랜치 정보를 저장                 |

### ⚙️ 명령어 내부 동작

| 명령어   | 동작                                                                                  |
| -------- | ------------------------------------------------------------------------------------- |
| `init`   | `.mini_git/` 디렉토리 및 내부 구조 초기화 <br>→ `HEAD`, `objects/`, `refs/`, `index`) |
| `add`    | 파일 내용을 읽어 blob으로 저장하고<br>`index`에 파일명-해시 매핑 추가                 |
| `commit` | index의 blob들을 Tree로 묶고<br>Commit 객체 생성 후 저장 <br>→HEAD 갱신               |

```tree
.mini_git/
├── HEAD                   ← 현재 브랜치 위치 기록
├── objects/               ← blob, tree, commit 저장
├── refs/
│   └── heads/             ← 브랜치 포인터 저장
└── index                  ← 스테이징 영역 (index.json)
```

### ⚙️ index 구조 예시

> 실제 Git은 바이너리로 저장하지만,<br>
> mini-git에서는 학습 목적으로 **JSON 기반의 단순 구조**를 사용

```json
// index.json 구조
{
  "hello.txt": "abcd1234...",
  "buy.md": "ef567890..."
}
```

### ⚠️ 구현시 고려 사항

#### Blob 생성

- 파일 내용 읽기 <br>→ SHA-1 해시 계산 <br>→ `objects/`에 저장
- **이미 존재하는 해시**라면 중복 저장하지 않음

#### Tree 생성

- index 기반으로 디렉토리 구조 정리
- 각 항목<br>→ `<파일 모드> <타입> <해시> <파일명>` 형태로 구성
- 하위 폴더는 Tree 안에 Tree로 표현

#### Commit 생성

- 현재 Tree 해시, 부모 커밋 해시, 메시지, 시간, 작성자 등을 포함
- 커밋 후 HEAD가 가리키는 브랜치가 이 커밋 해시로 업데이트

### ⚠️ 테스트 포인트

| 테스트 항목    | 검증 내용                                                    |
| -------------- | ------------------------------------------------------------ |
| Blob 생성      | 파일 내용을 읽어 객체가 `.mini_git/objects/`에 저장되는가?   |
| 중복 저장 방지 | 동일한 내용의 파일을 여러 번 add해도 해시가 동일한가?        |
| Tree 구조 생성 | commit 시 디렉토리 구조가 올바르게 tree 객체로 만들어지는가? |
| HEAD 갱신 확인 | commit 후 HEAD가 새 커밋을 가리키는가?                       |
| index 초기화   | 커밋 후 index가 비워지는? → `스테이징 영역 초기화`           |
