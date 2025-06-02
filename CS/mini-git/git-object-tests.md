---
title: Git Object Test & Comparison – 내부 구조 실험 및 실제 Git 비교
date: 2025-05-07
tags: [git, internals, test, hash-object, cat-file, blob, commit, experiment, git-vs-minigit]
category: git
description: 직접 구현한 mini-git과 실제 Git 명령어(hash-object, cat-file)를 비교하여, 객체 저장 방식과 내부 구조 차이를 실험하고 정리한 문서
draft: false
---
# 🧪 Git Object Test & Comparison

> 이 문서는 직접 구현한 mini-git과 실제 Git 명령어(`hash-object`, `cat-file`)를 비교해보며,  
> Git 객체가 저장되고 참조되는 방식의 차이점을 실험하고 정리한 문서입니다.

_Last updated: 2025-05-07_  