---
title: 다양한 그리디 문제 해결 능력 강화
date: 2025-05-02
tags:
  - algorithm
  - greedy
  - dp
  - strategy
  - 반례
  - 회의실배정
  - 큰수만들기
draft: false
---

`목표` 다양한 그리디 문제 해결 능력 강화

## 📌 학습 목표

- 다양한 그리디 문제 유형 익히기
- 회의실 배정 및 대표 문제 풀이
- 동전 문제에서 그리디를 적용하지 못하는 경우
- 그리디 알고리즘의 한계<br>✅ DP와의 차이를 중점으로
## 학습 목록

| 개념                   | 체크  | 메모                                     |
| -------------------- | --- | -------------------------------------- |
| 그리디 알고리즘의 탐욕적 선택 속성  | ✅   | 현재 선택이 미래에 영향을 주지 않아야 함                |
| 동전 문제에서 그리디 한계 이해    | ✅   | 항상 최적해를 보장하지 않음 <br>→ 반례 찾기            |
| 회의실 배정 문제에서의 정렬 전략   | ✅   | 끝나는 시간 기준으로 **정렬**                     |
| 수학적 최적화 문제에서의 그리디 적용 | ✅   | 숫자 조작, 분배 등<br>다양한 문제 유형에 대응 가능        |
| DP와 그리디 차이점 정리       | ✅   | DP는 전체 경우 고려<br>그리디는 현 시점 *최선의 선택*에 집중 |

## 실습

| 문제      | 링크                                         |
| ------- | ------------------------------------------ |
| 회의실 배정  | 회의 시작/끝 시간 기반 정렬 후 선택 문제                   |
| 큰 수 만들기 | 가장 큰 수를 만들기 위한 그리디 로직<br>⚠️ 스택 활용 가능성 확인하기 |
##  자가 점검

- [x] 그리디의 반례 가능성을 설명할 수 있는가?
- [x] 정렬 기준은 `무엇`을 기준으로 설정해야 하는가?
- [x] 수학적 최소/최대 문제에서 그리디 알고리즘이 적용 가능한지 판단할 수 있는가?

### ✨ 마무리하며

-  그리디 알고리즘의 적용 조건을 판단하기
-  그리디의 단점과 반례를 꼭 인지할 것👀
-  정렬 기준이 그리디에 얼마나 큰 부분을 차지하는지 체감하기

> 오늘은 다양한 실전 예제를 통해  
> **그리디 알고리즘의 적용 가능성과 한계**를 체감하는 날🔥🔥
> 
> 상황에 따라 어떤 알고리즘을 선택해야 할지  
> *판단하는 능력*을 길러야한다👊👊

---

# 📌 오늘의 회고

`Today I Learned` 다양한 그리디 문제 해결 능력 강화

- [x] DP와 그리디 사용법을 비교할 수 있는가?
- [x] 수학적 최적화 문제에서 그리디를 적용할 수 있는가?
- [x] `회의실 배정`과 `큰 수 만들기` 문제 풀이

## 배운 점 

오늘은 **그리디랑 DP 사용 시점**을 구분하는 데 시간을 썼다!<br>사실, 지금까지도 좀 애매했던 게 `그리디로 풀어도 되는 문제인가?`였다🤯

> 지금의 선택이 미래에 영향을 주지 않아야 한다 ⚠️
>→ 이게 핵심.

<br>예를 들어, **동전 문제**에서는

- 동전이 1, 5, 10처럼 **배수 관계**일 땐 → *그리디 가능*
- 동전이 1, 3, 4이면 → *그리디 실패*
```javascript
// 반례 예시
coins = [1, 3, 4], target = 6
// 그리디: 4 + 1 + 1 → 3개 사용
// 최적해: 3 + 3 → 2개 사용 (DP 필요)
```

그래서 오늘의 핵심 문구는 <br>`내 선택이 이후 선택에 영향을 미치는가?` 이다🏄🏄

## 🧘 깨달은 점
### 문제 풀이 ① : 회의실 배정

- 회의실 문제는 심플하면서도 강력한 그리디 문제다.
- 처음엔 **조건문 2개 써서** 복잡하게 처리해야하나 했지만,<br>`[끝나는 시간, 시작 시간]`으로 구조분해 정렬을 이용했다👊🏻👊🏻

```javascript
meetings.sort((a, b) => {
  if (a[1] !== b[1]) return a[1] - b[1]; // 끝나는 시간 기준 정렬
  return a[0] - b[0];
});

let end = 0, count = 0;
for (let [start, finish] of meetings) {
  if (start >= end) {
    end = finish;
    count++;
  }
}
```

- **끝나는 시간이 빠른 회의부터 선택** <br>→ 이후 선택 가능성 높임
- 조건 1개로도 충분히 해결됨!

### 문제 풀이 ② : 큰 수 만들기

- 이 문제는 숫자 순서를 유지하되 가장 큰 수로 구성해야했다.
>`그리디와 스택`의 조합이 딱 맞아떨어졌다👀

- 처음엔 `slice, sort`로 하려다 *시간 초과 리스크*가 너무 커서  <br>**스택을 쌓고 작으면 pop, 크면 push**하는 방식으로 접근했다.

```javascript
function solution(number, k) {
  const stack = [];
  for (let digit of number) {
    while (k > 0 && stack[stack.length - 1] < digit) {
      stack.pop();
      k--;
    }
    stack.push(digit);
  }
  return stack.slice(0, stack.length - k).join('');
}
```

- 이 방식은 **앞자리를 크게 유지**하는 게 핵심이다.

### 수학적 최적화와 그리디

- 문제 중 **수식의 결과를 최소화하거나 최대화하는 문제**들도 있다.

 > `"55 - 50 + 40"` → **괄호 배치로 최소값 만들기**

```javascript
// 전략: 첫 '-' 이후 모든 수는 괄호로 묶어서 빼버리기
function minimize(expression) {
  const parts = expression.split('-').map(part =>
    part.split('+').reduce((a, b) => +a + +b, 0)
  );
  return parts.slice(1).reduce((acc, val) => acc - val, parts[0]);
}
```

- `-` 연산 이후 `+`가 붙은 항은 무조건 묶어서 빼야 최소값이 된다.
- 사칙연산 순서에 의해 현재의 선택이 미래에 영향을 안 미침 <br>→ ⚠️ 조건 만족


### DP와 Greedy, 언제 어떤 걸 쓸까?

- 그리디는 언제나 정답이 아니다🙅‍♀️  <br>→ 문제 조건이 만족될 때만 적용
>수학적 최적화 문제는 직관과 수식 구조 파악이 핵심**

## ✨ 마무리하며

오늘은 그리디 알고리즘의 다양한 형태를 배우고,  
`그리디가 정답일 조건`을 조금 더 이해한 하루였다.

> 그리디는 빠르지만 조심해서 써야 한다.  
> DP와 언제 갈라지는지를 아는 게 관건이다😮‍💨
