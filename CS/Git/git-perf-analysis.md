---
title: Git Performance Analysis – Git 구조별 성능 실험
date: 2025-05-07
tags: [git, performance, benchmark, blob, commit, tree, hash-object, cat-file, experiment]
category: git
description: 실제 Git 명령어를 통해 blob, commit, tree 구조의 저장 성능과 처리 속도를 실험하고, 구조적 효율성을 비교 분석한 문서
draft: false
---
# 🧪 Git Performance Analysis

> 이 문서는 실제 Git 명령어(`hash-object`, `cat-file` 등)를 사용해  
> 파일 크기, 커밋 속도, 구조별 처리 성능을 측정한 실험 결과를 정리한 문서입니다.

_Last updated: 2025-05-07_  