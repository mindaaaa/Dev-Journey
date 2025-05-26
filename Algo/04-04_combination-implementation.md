---
title: 조합(Combination) 직접 구현
date: 2025-04-04
tags:
  - algorithm
  - combination
  - 구현
  - 완전탐색
  - til
draft: false
description: 조합 개념과 구현 방식(nCr)에 대해 학습하고, 재귀 기반 조합 함수를 직접 구현하며 완전탐색 감각을 익힌 날
---
`목표` 완전탐색의 개념을 이해
## 📌  목표

- 완전탐색의 개념을 이해하고, for 중첩 구조로 `모든 경우`를 만들어낼 수 있다.
- 조건문을 활용해 *정답 필터링 로직*을 자연스럽게 작성한다.
- 프로그래머스 `모의고사` 문제를 완전탐색으로 해결하며 실전 감각을 익힌다.

## 학습 흐름

| Step   | 내용                                          | 체크  |
| ------ | ------------------------------------------- | --- |
| Step 1 | 완전탐색 개념 학습 *모든 경우를 직접 탐색하는 방식*              | ✅   |
| Step 2 | `중첩 for`문으로 조합 생성 연습<br> - `서로 다른 3자리 수 조합` | ✅   |
| Step 3 | `모의고사` 문제 이해 → 완전탐색 설계                      | ✅   |
| Step 4 | 문제 풀이 및 로직 점검 *패턴 vs 정답 비교*                 |     |

#예제 #서로다른3자리수

```js
for (let i = 1; i <= 5; i++) {
  for (let j = 1; j <= 5; j++) {
    for (let k = 1; k <= 5; k++) {
      if (i !== j && j !== k && i !== k) {
        console.log(i, j, k); 
      }
    }
  }
}
```

## 학습 키워드

- 완전탐색 (Brute-force)
- 중첩 for문
- 조건문 필터링
- 경우의 수 생성 → 정답 비교

## 자가 점검

-  완전탐색이 필요한 상황을 설명할 수 있는가?
-  중첩 for문으로 모든 조합을 생성할 수 있는가?
-  조건문을 통해 유효한 경우만 필터링했는가?
-  문제에 맞는 정답 비교 로직을 작성했는가?

## ✨ 마무리

-  완전탐색은 언제 효과적인가? 
-  중첩 반복문은 어떻게 구성되는가?
-  모의고사 문제에서 가장 어려웠던 점은?

---

# 📌 오늘의 회고

`Today I Learned` 완전탐색 & 모의고사 회고

- [x] `완전탐색` 문제를 처음부터 끝까지 혼자 힘으로 분석해 보았는가?
- [x] `정답 패턴을 순환`시키는 방법을 정확히 구현했는가?
- [x] `map → filter(Boolean)`을 통해 조건부 배열 처리 기법을 체득했는가?
- [x] 다른 사람의 코드를 보며 **시야를 넓히는 경험**을 했는가?

## 오늘의 배운 점 #TIL

- `완전탐색`은 정답을 찾기 위해 **모든 경우의 수를 탐색**하는 방식이며,  
  단순한 구현도 **정확하고 반복적인 비교가 핵심**이다.
- `answers.length`만큼 반복되는 정답 비교는  
  `idx % pattern.length`로 패턴을 순환하며 간단하게 구현할 수 있었다.
- 정답자 점수를 비교할 때, 기존에는 이중 `map`을 이용한 연쇄 체이닝 방식으로 구현하였으나
  `map + filter(Boolean)`을 활용한 더 깔끔한 방식으로 수정했다✍️

## 🧘 깨달은 점

- `map()`은 항상 원본 배열과 동일한 길이를 반환하지만,  
  조건에 따라 `null` 또는 `undefined`를 넣고  
  `filter(Boolean)`으로 *falsy 값 제거*하면 결과 배열의 크기를 유동적으로 조절할 수 있다. 

- 다른 사람의 정답 풀이 중 **filter + 조건문**으로 매우 간결하게 푼 코드를 보며  
> 완전탐색 문제에서는 무조건 for문이 정답이 아닐 수 있으며
> 목적을 가장 깔끔하게 달성하는 게 코테의 가장 큰 핵심이라는 **사고 전환**을 경험했다.

- 특히 아래 코드를 구현하는 과정에서 새로운 인사이트를 획득했다.
```javascript
return count
  .map((score, idx) => (score === maxScore ? idx + 1 : null))
  .filter(Boolean)
  .sort((a, b) => a - b);
```

- map은 **항상 원본 배열과 같은 길이로 반환**하기 때문에  
    조건에 맞는 요소만 남기려면 `filter`가 필요했으며,
    `filter(Boolean)`을 이용한 매우 간결한 *truthy 필터링 방식*을 체득했다.
- 이 방식은 `null`, `false`, `undefined` 등을 걸러주는 유용한 패턴이라는 것도 함께 배웠다.
    
- 또 다른 사람의 코드에서 `result 배열`을 쓰는 것이 더 직관적일 수도 있었겠다는  
    **시야 확장의 경험**도 함께 얻어갔다.  
```javascript
const result = [];
for (let i = 0; i < 3; i++) {
  if (count[i] === maxScore) result.push(i + 1);
}
return result;
```
        
- 무조건 함수 체이닝으로 해결했는데 **문제 구조와 상황에 따라 더 읽기 쉬운 코드**에 대해 고민하게 되었다🤔

## ⚙️ 구현 코드

```javascript
function solution(answers) {
  const correctArray = [
    [1, 2, 3, 4, 5],
    [2, 1, 2, 3, 2, 4, 2, 5],
    [3, 3, 1, 1, 2, 2, 4, 4, 5, 5],
  ];

  const count = new Array(3).fill(0);
  for (let i = 0; i < correctArray.length; i++) {
    answers.forEach((answer, idx) => {
      if (answer === correctArray[i][idx % correctArray[i].length]) {
        count[i]++;
      }
    });
  }

  const maxScore = Math.max(...count);
  return count
    .map((score, idx) => (score === maxScore ? idx + 1 : null))
    .filter(Boolean)
    .sort((a, b) => a - b);
}
```


## 🎯 내일의 목표

|주제|목표|
|---|---|
|순열(permutation) 구현 연습|**외우지 않고 직접 구현**하며 완전탐색 감각 기르기|
|조합(combination) 구현 연습|**nCr의 구조를 직접 구현**하며 조합 원리 체득|

> 오늘은 완전탐색 사고방식을 손에 익히고,  
> **조건 비교 → 점수 계산 → 정답 반환**의 흐름을 정제했다.  
> 다음엔 완전탐색의 꽃인 `순열과 조합`을 **내 손으로 직접** 구현해보겠다 🔥🔥🔥