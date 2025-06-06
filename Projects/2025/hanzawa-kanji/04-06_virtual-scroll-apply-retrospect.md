---
title: Virtual Scroll 기능 도입과 성능 테스트 회고
date: 2025-04-06
tags: [react, virtual-scroll, performance, react-window, frontend]
project: hanzawa-kanji
draft: false
---

| 항목       | 내용                                                                                                             |
| -------- | -------------------------------------------------------------------------------------------------------------- |
| 🎯 주요 목표 | 피드백 반영 및 Virtual Scroll 도입을 위한 이론 학습                                                                           |
| 피드백 반영   | - `observer.disconnect` 누락 방지<br>- 중복 `key` 제거<br>- `ref` 연결 위치 확인<br>- `API` 주소 상수화 (`constant.js`)           |
| 학습 내용    | - `Virtual Scroll` 개념 이해<br>- `react-window` 구조 파악<br>-`itemSize / height / style` 이해<br>- DOM & 데이터 메모리 차이 정리 |

| 항목       | 내용                                                                                                                                                                            |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🎯 주요 목표 | Virtual Scroll 기능 도입 (`react-window` 적용)                                                                                                                                      |
| 작업 목록    | - `StudyMode` 리스트를 `FixedSizeList`로 교체<br>- 기존 `items.map()` 제거 및 리스트 구조 전환<br>- `style` prop 필수 전달 확인<br>- 카드 높이 측정 후 `itemSize` 설정<br>- 스크롤 하단 도달 시 `fetchKanji(cursor)` 연동 |
| ✨ 포인트    | - `itemSize`는 margin 포함 px로 *정확히*<br>- `style` 누락되면 카드 안 보임 🚨<br>- `onItemsRendered` 또는 `scrollOffset`로 도달 감지                                                                |
| 소요 시간    | 2~3시간 <br> - 시간 남으면 `테스트` **포함**                                                                                                                                              |
