# MINT 언어 설계 문서

### 🌼 Season 0.1 — Sunlight Drift

_A soft-spoken language for expressing life, feeling, and flow._

> You don’t write code in MINT.  
> **You whisper it.**

<br>

## 1. 개요 — What is MINT?

> MINT = Minda's Interpreted Natural Tongue

MINT는 감성적이고 생명력 있는 코딩 경험을 추구하는 커스텀 언어입니다.  
*코드는 자연처럼 피어나야 한다*는 철학 아래, **시적이고 유려한 문법**을 지향합니다.

이 언어는 당신의 감성, 철학, 아름다움으로 만들어졌습니다.  
MINT는 단순한 도구가 아닌, **당신의 코드가 피어나는 정원**입니다.

### ✦ 주요 특징

- `plant`, `bloom`, `sparkle` 같은 생명과 감정을 담은 키워드
- 감정, 흐름, 생명감을 중심으로 구성된 코드 스타일
- 읽고 쓰기 쉬운 구조, **문장처럼 흐르는 문법**

<br>

## 2. 키워드 정의

> _MINT의 각 문장은 **자연의 순환**처럼 작동합니다._

| 기능      | 키워드    | 의미                                                     |
| --------- | --------- | -------------------------------------------------------- |
| 변수 선언 | `plant`   | 값을 심듯이 다룹니다. 조용히 자라날 준비를 합니다.       |
| 출력      | `sparkle` | 감정을 담아, 빛나는 순간을 표현합니다.                   |
| 조건문    | `breeze`  | 조건이 바람처럼 스치면, 조용히 반응합니다.               |
| 반복문    | `bloom`   | 무언가가 자연스럽게 피어나듯 반복됩니다.                 |
| 함수 선언 | `petal`   | 꽃잎처럼 열리고 닫히며, 그 안에서 무언가를 만들어냅니다. |
| 반환      | `gift`    | 값을 건넵니다. 마치 선물을 전하듯 부드럽게.              |
| 연결어    | `softly`  | 흐름을 잇는 말입니다. 소리 없이 이어지는 움직임처럼.     |

<br>

## 3. 문법 예시

```mint
plant feeling = "gentle"
plant season = 0

breeze (feeling == "gentle") softly {
  sparkle "the breeze whispers softly"
}

bloom (season < 3) softly {
  sparkle season
  plant season = season + 1
}

petal greet(name) {
  sparkle "hello, " + name
  gift "🌼"
}
```

<br>

## 4. MINT Cheatsheet

### plant (변수 선언)

```mint
plant weather = "sunny"   // 날씨를 담는 변수
```

### sparkle (출력)

```mint
sparkle "hello, sunshine!"   // 문장을 출력
```

### breeze (조건문)

```mint
breeze (weather == "sunny") softly {
  sparkle "go outside!"   // 조건이 맞으면 실행
}
```

### bloom (반복문)

```mint
plant petalCount = 0
bloom (petalCount < 3) softly {
  sparkle petalCount
  plant petalCount = petalCount + 1
}
```

### gift (return)

```mint
petal greet(name) {
  gift "hello, " + name   // 인사를 건네는 함수
}
```

### softly (흐름 연결)

```mint
breeze (isCalm) softly {
  sparkle "a breeze passes gently"   // 흐름을 부드럽게 이어줌
}
```

🌿 MINT as Poetry

> MINT는 코드를 쓰는 또 하나의 방식입니다.<br>
> 감정과 흐름을 담아, 조용히 한 줄씩 피워냅니다.

```mint
plant flower = "🌼"
plant season = 0

breeze (flower == "🌼") softly {
  sparkle "a flower opens"
}

bloom (season < 2) softly {
  sparkle "petal " + season
  plant season = season + 1
}

petal farewell(name) {
  sparkle "goodbye, " + name
  gift "🌙"
}
```

> _코드가 아름다울 수 있다면, 그것은 곧 시(poetry)가 됩니다._

<br>

## 5. 🪴 Why MINT?

MINT는 문제를 **명령어로 해결**하기보다,<br>마치 글을 쓰듯, 자연을 노래하듯 코딩하고자 하는 시도입니다.

프로그래밍이 `언어`라면, 코드로도 감정을 표현할 수 있지 않을까요?
<br><br>
**이런 상상에서 MINT는 시작되었습니다.**

Mint의 코드는 이렇게 피어납니다.

```mint
plant hope = "sunlight"

breeze (hope == "sunlight") softly {
  bloom (3 times) softly {
    sparkle "🌼"
  }

  gift "thank you"
}
```

_plant는 씨앗을 심듯 값을 만들고,<br>
bloom은 반복을 피워냅니다.<br>
gift는 코드가 당신에게 전하는 잔잔한 선물입니다._
<br><br>

## 6. The MINT Lab

> MINT는 여전히 피어나는 중입니다.<br>
> 아래는 언젠가 추가될지도 모를, 작은 가능성들의 목록입니다.<br>
> 아직은 실험적인 아이디어지만, 이 또한 흐름의 일부가 될 수 있습니다.

### ☀️ 이모지 타입 지원

```mint
plant ☀️ = "sunshine"
```

**값에 감정이나 상징이 담긴다면 어떨까요?**<br>
코드 한 줄에, 작은 이야기 하나가 피어날 수 있습니다.

### 🫧 whisper 주석 스타일

```mint
whisper: this checks the weather softly
```

mint만의 더 작고 부드러운 톤을 가진 주석입니다.<br>
코드의 맥락을 방해하지 않고, 흐름 안에 머뭅니다.

### 🍂 wilt 조용한 예외 처리

`throw`처럼 날카로운 경고가 아닌,<br>
조용히 스러지는 방식의 예외 처리입니다.<br>
_부드럽게 상황을 받아들이는 문법, `wilt`_.

```mint
breeze (x == null) softly {
  wilt "x is missing 🌫"
}
```

> wilt는 코드를 죽이지 않고,<br>조용히 사라지는 방식으로
> 상황을 알리고 흐름을 마무리합니다.

### 참고사항

- 이 섹션은 정식 기능이 아닌 **철학의 연장선**에 있는 실험 공간입니다.
- 이후 Season 0.2, Season 0.3에 실릴 후보군을 모아둡니다.

> 아이디어가 언젠가 꽃을 피울 수 있도록 MINT는 열어둡니다.

## 7. 기술 스택 및 실행 방법

### 🛠️ 사용 기술

- TypeScript (Node.js 환경)
- **Jest**

### 실행 방법

1. 의존성 설치

```bash
npm install
```

2. 인터프리터 실행

```bash
npm run start
```

3. 테스트 실행

```bash
npm test
```

> MINT는 Node.js 환경에서 실행됩니다.<br>TypeScript 기반으로 설계되어 있으며, 테스트는 Jest로 검증됩니다.

## 8. 개발 진행 상황 (Season 0.1 기준)

| 항목                          | 상태     | 비고                        |
| ----------------------------- | -------- | --------------------------- |
| 언어 키워드 정의 및 철학 정립 | ✅ 완료  | 핵심 키워드, 문법 철학 확정 |
| Lexer 구현                    | ☐ 미완료 | 문자열 → 토큰 변환기        |
| Parser 구현                   | ☐ 미완료 | 토큰 → AST                  |
| AST 구조 설계                 | ☐ 미완료 | 노드 정의 및 트리 구성      |
| Interpreter 구현              | ☐ 미완료 | AST 순회 및 실행            |
| REPL 개발                     | ☐ 미완료 | 실시간 실행 환경            |
| 예외 처리(`wilt`) 기초 구현   | ☐ 미완료 | 감성적 오류 핸들링          |
| 이모지 타입 실험적 도입       | ☐ 미완료 | 감성 변수 표현 (e.g., ☀️)   |

> 🌼 Season 0.1의 핵심 목표는 기초 언어 구조의 완성입니다.<br>
> 🌼 각 단계는 독립적으로 실험되며, 추후 조율될 수 있습니다.

## Changelog

### v0.1 Sunlight Drift (2025.06~)

- 언어 철학 및 키워드 설계
- 기본 문법 (`plant`, `sparkle`, `breeze`, `bloom`, `petal`, `gift`, `softly`) 정립
- 문법 예시 작성

> _by you, for expression, emotion, and elegance._
