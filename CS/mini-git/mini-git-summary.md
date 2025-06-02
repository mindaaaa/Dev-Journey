---
title: Mini Git Summary – Git 구현 기반 학습 회고
date: 2025-06-02
tags:
  - git
  - mini-git
  - cli
  - nodejs
  - blob
  - tree
  - commit
  - vcs
  - git-internals
  - project-log
category: git
description: Git 내부 구조와 명령어 동작 원리를 직접 구현한 mini-git 프로젝트의 기획, 설계, 구현 과정을 정리한 회고 중심 문서
draft: false
---
# 🐙 Mini Git Summary

> 이 문서는 Git의 핵심 기능을 직접 구현한 mini-git 프로젝트의  
> 기획, 구현 과정, 회고 내용을 정리한 문서입니다.  
> Git 내부 구조와 명령의 작동 원리를 실제로 구현하며 학습한 과정을 담고 있습니다.

_Last updated: 2025-06-02_  

## `mini-git`은 무엇을 목표로 하는가?

> mini-git은 Git 내부 구조를 직접 구현하며 학습한 실습 기반 프로젝트입니다.

### 주요 목표

- `.git/` 디렉토리 내부 이해
- 커밋/브랜치/포인터의 흐름 시각화
- 해시 기반 저장 원리 체득

| 목표                | 설명                                                                           |
| ------------------- | ------------------------------------------------------------------------------ |
| Git의 구조 이해     | `.git/objects`, `HEAD`, `refs`, `index`가 <br>어떤 역할을 하는지 직접 다뤄보기 |
| 객체 저장 방식 체험 | Blob, Tree, Commit 객체가 어떻게 만들어지고 <br>연결되는지 직접 구성           |
| 무결성 구조 감 잡기 | SHA-1 해시 기반으로 파일 내용이<br>어떻게 추적되고 구별되는지 실습             |
| 기능 흐름 구현      | `init → add → commit` 흐름을 통해 Git의 핵심 프로세스를 재현                   |

### mini-git 특징

- zlib, SHA-1 사용 → Git 저장 방식 재현
- Git의 복잡한 **바이너리 포맷**을 생략화<br>→ **JSON과 텍스트 기반** 구조로 단순화
- `.git/`이 아닌 `.mini-git/`으로 실습 구현
- 내부 객체들을 **디렉토리와 파일**로 직접 저장하며 흐름 확인<br>→ 객체 저장은 `.mini-git/objects/`로 확인 가능
### 핵심 흐름 요약

```plaintext
init → add → commit
         ↓      ↓
      Blob   Tree + Commit
         ↘     ↙
        index (스테이징)
```

## 🛠️ 사용법
##### 명령어 시나리오 
- *2025_06_02 기준*
```bash
$ node src/index.js init
$ node src/index.js add hello.txt
$ node src/index.js commit "first commit"
$ node src/index.js branch dev
$ node src/index.js checkout dev
$ node src/index.js log
$ node src/index.js cat-file -p <SHA-1 해시>
```

> 내부적으로 어떤 구조가 만들어지는지는 [`git-internals.md`](git-internals.md)에서 확인할 수 있습니다.

## 문서 구성 가이드

| 문서명                                                        | 설명                                                                                                              |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| [`mini-git-summary.md`](mini-git-summary.md)               | 프로젝트 개요, 기능 구성, 구현 흐름, 전체 구조 요약 등<br>**전반적인 설명과 개요**를 담은 메인 문서<br>                                              |
| [`git-internals.md`](git-internals.md)                     | Git의 내부 저장 구조와 핵심 객체 (`blob`, `tree`, `commit`), `HEAD`, `index`, `refs` 등의 **구성 원리와 작동 방식**을 정리한 핵심 아키텍처 설명 문서 |
| [`git-design-notes.md`](git-design-notes.md)               | mini-git의 설계 기준, 단순화 결정 이유, 디렉토리 구조 고민, 전략 패턴 리팩토링 등 **설계 중심 회고 및 개발 의도**를 담은 문서                                |
| [`git-behavior-comparison.md`](git-behavior-comparison.md) | Git과 mini-git이 `add`, `commit`, `checkout` 등 명령어를 수행할 때 **어떤 저장 구조와 흐름 차이가 발생하는지 시각적으로 비교**한 실험 기반 분석 문서        |
| [`README.md`](https://github.com/mindaaaa/mini-git)        | 실제 명령어 사용법, CLI 구조, 옵션 설명 등 mini-git 프로젝트의 **사용 매뉴얼** 역할을 하는 문서                                                 |

## 주차별 진행 계획
### 1. *Week 1*
- 구조 이해 & 최소 동작 만들기

| 목표           | 상세 내용                                                                       |
| ------------ | --------------------------------------------------------------------------- |
| `git init`   | `.mygit/` 폴더, `objects/`, `refs/`, `HEAD` 초기화                               |
| `git add`    | 파일을 읽어 해시(blob) 생성 <br>→ `objects/`에 저장, `index` 역할 JSON에 기록                |
| `git commit` | staging 영역 <br>→ tree <br>→ commit 객체 생성 <br>→ `objects/` 저장, `HEAD` 포인터 갱신 |
| 학습           | Git object 구조 이해 <br>→ blob, tree, commit, HEAD 관계도 그리기                     |

> *Git이 파일을 어떻게 저장할까?* 
> **자료구조와 파일 시스템 감각**을 키우는 데 집중하는 주차

### 2. *Week 2* 
- 커밋 히스토리 & 브랜치 흐름

| 목표                  | 상세 내용                                   |
| ------------------- | --------------------------------------- |
| `git log`           | HEAD에서 부모 커밋 따라가며 로그 출력                 |
| `HEAD`              | 현재 커밋 또는 브랜치 정보 기록 및 이동 처리              |
| `branch` (optional) | `refs/heads/브랜치명` 파일 생성 <br>  → HEAD 연결 |
| 구조 학습               | Git에서 `포인터` 개념이 왜 중요한지 알아보자🏄           |

> Git은 결국 포인터의 집합이다.  
> 구조적 흐름 감각을 훈련하는 데 초점을 두는 주차

### 3. *Week 3*
- `checkout` & 확장

| 목표         | 상세 내용                                    |
| ---------- | ---------------------------------------- |
| `checkout` | HEAD를 브랜치로 전환하고, 작업 디렉토리 갱신              |
| 리팩토링       | - 클래스 분리<br>- 상태 패턴 적용<br>- 구조화된 커밋 시나리오 |
| 테스트        | 커밋 시나리오 테스트                              |

> 📌 커밋 시나리오
> init → add → commit → branch → commit → checkout → log  

### ✨ *Week 3.5* 

- 삽질 정리, 배운 개념 블로그 포스팅 
- `.mygit/` 폴더 전체 설명 문서 작성
- Git 개념 다이어그램(`tree`, `commit` `HEAD`) 시각화 

## 참고 도서

| 목적                     | 추천 도서           |
| ---------------------- | --------------- |
| 내부 구조 이해 및 mini-git 설계 | 깃: 분산 버전 관리 시스템 |
| 구조 및 실습 대비             | Pro Git         |
| 입문 / 정리용 스낵 도서         | 모두의 Git         |
| 핵심만 간단하게               | Git Internals   |

> 구조 이해를 위해 책을 읽고
> 실습은 `Git object`와 `JSON`구조로 *mini-git* 구현

> `tig`, `git log --graph`, `.git/objects/` 등을 직접 열어보면서 진행하기
> `.git/objects`, `cat-file`, `hash-object` 실습
