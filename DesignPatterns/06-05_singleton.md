# 싱글톤(Singleton) 패턴이 필요한 이유
## ① 싱글톤이란?
**싱글톤(Singleton) 패턴**은 어떤 클래스의 인스턴스를<br>**애플리케이션 전체에서 단 하나만** 생성해서 사용하는 디자인 패턴이다.

- 전역적으로 하나의 객체를 공유해야 할 때 사용
- 인스턴스 생성을 통제하여 **중복 생성, 리소스 낭비, 상태 충돌을 방지**한다

## ② `하나만 있어야 하는 경우`가 **왜** 있을까?
### 대표적인 상황

| 상황      | 설명                               |
| ------- | -------------------------------- |
| DB 연결   | DB 커넥션은 비용이 크고, 동시에 여러 개 열면 위험함  |
| 네트워크 소켓 | 같은 서버에 여러 연결을 열면 충돌 가능성 존재       |
| 설정 객체   | 앱 전역에서 공통으로 쓰는 설정은 하나만 있어야 함     |
| 상태 관리자  | 리액트 등에서 글로벌 상태를 한 곳에서 관리할 필요가 있음 |
| 로깅 시스템  | 로그는 중앙 집중으로 남겨야 의미가 있음           |

## ③ 예시

### 📌 로거(Logger)

```js
const logger = Logger.getInstance();
logger.log(`[INFO] ${msg}`);
```

- 로그를 콘솔, 파일, 서버 등에 전송할 수 있음
- 인스턴스가 여러 개면 로그 위치가 분산되고 관리 어려움

### 📌 설정(Config)

```javascript
const config = Config.getInstance();
console.log(config.apiKey);
```

- 앱 전반에 걸쳐 공유되는 설정 객체는 **하나로 유지되어야 함**
- `config.json` 파일을 여러 번 읽는 것도 비효율적

## ④ 싱글톤의 핵심 목적

- **중복 생성 방지**
- **공통 리소스의 통합 관리**
- **전역 상태의 예측 가능성 확보**
- **비용이 큰 객체의 재사용 최적화**

> 무조건 싱글톤을 쓰기보단,  
> *정말 하나여야만 하는가*를 먼저 고민해야 한다.

- 상태를 공유해야 하는가?
- 생성 비용이 큰가?
- 앱 전체에서 단 하나로 동작해야 하는가?

>그럴 경우에만 싱글톤을 도입하는 것이 **좋은 설계 습관**이다.

---

# 자바스크립트에서 싱글톤 구현 
> static 필드 / getInstance 구조

## ① 핵심 개념

싱글톤을 구현할 때는 보통 아래처럼 구현한다.

```javascript
class Singleton {
  static #instance = null; // 클래스 자체에 붙는 정적(private) 필드

  constructor() {
    if (Singleton.#instance) {
      throw new Error("getInstance()를 사용하세요!");
    }
    console.log("싱글톤 인스턴스 생성");
  }

  static getInstance() {
    if (!this.#instance) {
      this.#instance = new Singleton();
    }
    return this.#instance;
  }

  sayHi() {
    console.log("Hello, World!");
  }
}
```

### 👀 왜 static이어야 할까?

| 항목              | 이유                                                     |
| --------------- | ------------------------------------------------------ |
| `#instance`     | **클래스 전체에서 공유되는 인스턴스 저장소** 역할<br>→ 개별 인스턴스마다 갖는 속성이 아님 |
| `getInstance()` | **인스턴스를 생성하고 반환하는 정적 메서드**<br>→ 클래스에서 바로 호출해야 함        |

>따라서 싱글톤에서는 `new Singleton().getInstance()`처럼  
>인스턴스를 먼저 만들고 메서드를 호출하는 흐름이 **말이 안 된다.**

## ② 사용 예시

```javascript
const s1 = Singleton.getInstance();
const s2 = Singleton.getInstance();
console.log(s1 === s2); // true → 같은 인스턴스
```

## ③ 만약 static이 아니면?

```javascript
class BadSingleton {
  #value = Math.random();

  getInstance() {
    return new BadSingleton(); // ⚠️ 항상 새로 만들어진다.
  }
}

const b1 = new BadSingleton().getInstance();
const b2 = new BadSingleton().getInstance();
console.log(b1 === b2); // false 🤯
```

📌 이렇게 되면 **싱글톤이 아니라 그냥 공장(factory)**이 된다.

>싱글톤의 핵심은 **클래스 레벨에서 인스턴스를 통제**하는 것

>  **전체에서 하나로 유지해야 하는 데이터나 기능**은
>  무조건 static으로 빼두는 게 좋다.

---

# 모듈도 싱글톤이다

## 📌 ① 개념 요약

>자바스크립트의 ESModule 시스템(`import/export`)은  한 번 로딩된 모듈을 메모리에 **캐시**한다.  
>그 후 동일한 모듈을 다시 `import`하면, **같은 객체(참조)를 반환**한다.

즉, **모듈 전체가 싱글톤처럼 동작한다.**

## ② 예제

>같은 모듈이 공유되는 상황
#### config.js

```js
export const config = {
  value: Math.random(),
};
```
#### 👀 fileA.js

```javascript
import { config } from './config.js';
console.log("A:", config.value);
```
#### 👀 fileB.js
```javascript
import { config } from './config.js';
console.log("B:", config.value);
```
#### main.js

```javascript
import './fileA.js';
import './fileB.js';
```

#### 📌 출력 결과
```file
A: 0.4281...
B: 0.4281...  → 같은 값
```

> `config.js`는 실제로 **딱 한 번만 실행**되며,  
> 이후엔 동일한 객체를 **모든 import 지점에서 공유**한다.

## ③ 왜 이럴까? 

> ESModule의 캐싱 메커니즘

1. ESModule 시스템은 모듈을 처음 import할 때 **1번 실행**된다.
2. 실행된 결과는 메모리에 **캐시**된다.
3. 이후 동일한 경로로 import하면 **실행 없이 캐시된 객체 반환**된다.

이러한 흐름 때문에 모듈은 **싱글톤처럼 작동**한다고 말할 수 있다.

## ④ 장단점

### 📌 장점

| 항목              | 설명                                       |
| --------------- | ---------------------------------------- |
| 상태 공유가 쉬움       | 설정, 유틸, 상수 등을 한 번 만들어서<br>**전역에서 사용** 가능 |
| 자연스러운 전역 싱글톤 구현 | 별도 클래스 없이도 전역 인스턴스처럼 활용 가능               |
| 모듈 간 상태 동기화 가능  | import한 객체에서 상태를 수정하면 알아서 전파됨            |

### 📌 단점 

| 항목                   | 설명                                              |
| -------------------- | ----------------------------------------------- |
| 상태가 숨겨져 있다           | 명시적으로 인스턴스를 생성하지 않기 때문에<br>**어디서 변경됐는지 추적 어려움** |
| 테스트 격리 어려움           | import된 모듈은 캐시되므로,<br>**테스트 간 상태가 공유됨**         |
| 모듈이 *은근히* 전역 상태처럼 작동 | 무의식적으로 상태 공유가 일어나면 버그 유발 가능🤦‍♀️                |

#### 👀 테스트에서 문제가 되는 예

##### store.js

```javascript
export const state = { count: 0 };
```

```javascript
import { state } from './store.js';
state.count++; // 변경
```

```javascript
import { state } from './store.js';
console.log(state.count); // 📌 이미 1로 되어있음
```

> 테스트 간 상태 공유 → **테스트 신뢰성 저하**

#### 🙌 해결 방법

| 방법               | 설명                                   |
| ---------------- | ------------------------------------ |
| 상태를 함수로 감싸기      | `createStore()` 같은 팩토리 함수로 새로운 객체 반환 |
| 테스트 전에 모듈 리셋     | Jest의 `jest.resetModules()` 등 사용     |
| 명시적으로 클래스 싱글톤 사용 | 필요한 경우 class / static 방식 사용          |
## 정리

>ESModule은 *자동으로 싱글톤*처럼 동작한다.
    
>상태가 공유되는 걸 **의도하고 쓰면 유용**할 수 있지만,
>그렇지 않으면 **버그, 테스트 오류**를 유발할 수 있다.
    
>복잡한 상태나 테스트가 필요한 경우,  
>**class 기반 싱글톤이나 팩토리 함수 구조가 더 적절**하다.

---
# 모듈 싱글톤과 테스트 문제, 그리고 `reset()` 해법

## ① 문제 요약

자바스크립트에서 ESModule은 자동으로 **싱글톤처럼 동작**하기 때문에, <br> 
모듈에서 export된 객체는 테스트 간에도 **같은 인스턴스가 공유**된다.

>테스트 상태 공유로 인한 **테스트 격리 실패** 문제가 발생할 수 있다.

## ② 예제

>모듈 상태 공유로 테스트 실패

### logger.js

```javascript
export class Logger {
  logs = [];

  log(msg) {
    this.logs.push(msg);
  }

  getLogs() {
    return this.logs;
  }
}

export const logger = new Logger(); // 사실상 싱글톤
```

### 👀 test1.js

```javascript
import { logger } from './logger.js';
logger.log("TEST 1");
```

### 👀 test2.js

```javascript
import { logger } from './logger.js';
console.log(logger.getLogs()); 
// ["TEST 1"] ← 이전 테스트의 상태가 남아 있다.
```


## ③ 상황에 맞는 `reset` 해법
### 해법 ❶ reset() 메서드로 상태 초기화

```javascript
export class Logger {
  logs = [];

  log(msg) {
    this.logs.push(msg);
  }

  getLogs() {
    return this.logs;
  }

  reset() {
    this.logs = []; // 테스트용 메서드
  }
}
```
#### 테스트 코드 예시

```javascript
import { logger } from './logger.js';

beforeEach(() => {
  logger.reset(); // 상태 초기화
});
```

### 해법 ❷ 클래스 싱글톤 구조 및 static reset()

```javascript
class Logger {
  static #instance = null;
  logs = [];

  static getInstance() {
    if (!this.#instance) {
      this.#instance = new Logger();
    }
    return this.#instance;
  }

  static reset() {
    this.#instance = null;
  }

  log(msg) {
    this.logs.push(msg);
  }

  getLogs() {
    return this.logs;
  }
}
```
#### 테스트 코드

```javascript
Logger.reset(); // 인스턴스 자체를 초기화
const logger = Logger.getInstance();
```

### 해법 ❸ 테스트 프레임워크에서 모듈 리셋 (Jest)

```javascript
beforeEach(() => {
  jest.resetModules(); // 모듈 캐시 날림
  const { logger } = require('./logger.js'); // 다시 import
  logger.reset(); // 상태 초기화
});
```

> **주의**
> Jest에서는 ESM 대신 CommonJS(`require`) 방식으로 모듈을 재로드해야 동작

## ④ 언제 어떤 reset 해법을 쓸까?

| 상황                 | 해법                       |
| ------------------ | ------------------------ |
| 상태가 간단하고 객체 하나만 관리 | `reset()` 메서드 추가         |
| 클래스 기반 싱글톤으로 설계    | `static reset()` 메서드     |
| 모듈 자체를 초기화         | `jest.resetModules()` 사용 |

## 정리

>모듈은 **캐스**되므로 **테스트 간 상태가 공유**된다.

>이를 해결하기 위해 **reset 메서드**나 **모듈 초기화 기법**을 사용한다.
>테스트 코드의 신뢰성을 위해 반드시 **상태 격리 구조**를 고려해야 한다.

---

# 멀티톤(Multiton)

## ① 멀티톤 패턴이란?

> 여러 개의 싱글톤을 `각각의 키(Key)`에 따라 관리하는 패턴

- 싱글톤: 전체 앱에서 `하나의 인스턴스`
- 멀티톤: 특정 기준(식별자)에 따라 **하나씩만 존재**하는 인스턴스 모음

```plaintext
🙋‍♀️ 싱글톤: 하나의 관리자 (ex. 시스템 전체 로그 관리자)
👩‍❤️‍👨멀티톤: ID별 관리자 (ex. 사용자별, 페이지별, 캔버스별 객체)
```

```javascript
class Manager {
  static #instances = new Map();

  static getInstance(key) {
    if (!this.#instances.has(key)) {
      this.#instances.set(key, new Manager(key));
    }
    return this.#instances.get(key);
  }

  constructor(key) {
    this.key = key;
    console.log(`${key} 인스턴스가 생성됐습니다.`);
  }
}
```
## ② 사용하는 경우

- 싱글톤이 너무 전역적이라 하나로 묶기에 **과한 경우**
- 모든 인스턴스를 **계속 새로 만들기에 낭비**인 경우

> **각각의 상황에 대응하는 고유 인스턴스를 단 하나만 유지하고 싶을 때** 멀티톤 사용

>중복 생성 방지 / 리소스 낭비 방지 / 상태 충돌 방지

## ③ 현실적인 예시

| 사례              | 설명                   |
| --------------- | -------------------- |
| Canvas ID별 컨트롤러 | 각각의 DOM 요소를 관리       |
| 사용자 ID별 세션      | 각 유저마다 하나의 세션 객체 유지  |
| 채팅방 ID별 메시지 관리자 | 채팅방마다 고유한 데이터 캐시     |
| 지역 코드별 로더       | 국가/지역에 따른 API 데이터 로딩 |
| 프로젝트 ID별 서비스    | 프로젝트별 로직 캡슐화         |

>멀티톤은 **멀티 사용자, 멀티 영역, 멀티 리소스** 등에서 진가를 발휘한다.


### ④ 주의할 점

- 키 충돌 방지 필수 (string, number, Symbol 등)
- 인스턴스를 **명시적으로 제거하거나 초기화**할 수 있는 API 필요할 수 있다.
- 너무 많은 키를 관리하면... <br>→ **메모리 누수 가능** → `WeakMap` 활용 고려

## 정리

> 멀티톤은 싱글톤의 확장 버전이다. 
> **범위는 제한하되, 하나로 유지되는 객체가 필요할 때** 사용하면 된다.

---

# 강결합(Tight Coupling) vs. 간결합(Loose Coupling)

## ① 📌 개념 요약

- **강결합(Tight Coupling)**  
  → 하나의 클래스나 모듈이 다른 것에 **직접적으로 의존**하고 있어, **변경이 어렵고 테스트가 힘든 상태**

- **간결합(Loose Coupling)**  
  → 필요한 기능만 **인터페이스를 통해 의존**,  
  내부 구현에는 신경 쓰지 않아 **교체와 확장이 쉬운 상태**

## ② 예시

### ❶ 강결합 예시

```js
class UserService {
  constructor() {
    this.logger = new Logger(); // 직접 생성 → 강결합
  }

  createUser(name) {
    this.logger.log(`Created: ${name}`);
  }
}
```
- Logger를 꼭 `new` 해야 하고, 다른 로거로 교체 불가
- 테스트 시에도 MockLogger를 넣을 수 없음

### ❷ 간결합 예시 (의존성 주입)
```javascript
class UserService {
  constructor(logger) {
    this.logger = logger; // 외부에서 주입 → 간결합
  }

  createUser(name) {
    this.logger.log(`Created: ${name}`);
  }
}
```

- Logger를 외부에서 주입 가능 → 테스트 시 MockLogger 주입 가능
- 다른 구현체 (`FileLogger`, `SilentLogger`)로 교체도 쉬움

## ③ 강결합이 **왜** 문제일까?

| 문제 상황    | 설명                                       |
| -------- | ---------------------------------------- |
| 테스트가 어려움 | 의존 객체를 내부에서 직접 생성하면,<br>Mock 주입 불가       |
| 유연하지 않음  | 기능 확장을 위해 기존 클래스를 자주 수정해야 함 <br>→ OCP 위반 |
| 유지보수가 힘듦 | 한 클래스 수정이 여러 클래스에 연쇄적으로 영향을 줌            |

## 정리 

|항목|강결합 (Tight Coupling)|간결합 (Loose Coupling)|
|---|---|---|
|의존성 생성|내부에서 직접 생성 (`new`)|외부에서 주입 (DI)|
|테스트|어려움 (Mock 불가)|쉬움 (Mock 주입 가능)|
|유연성|낮음|높음|
|유지보수|수정 시 영향이 큼|교체와 확장이 쉬움|

> **간결합은 좋은 아키텍처의 핵심**  
> 간결합은 변경에 강하고, 테스트가 쉬운 구조를 만드는 첫걸음이다.

---

# 의존성 주입

>`DI` Dependaency Injection

## 📌 ① 개념 요약

>의존성 주입(DI)이란,
>어떤 객체가 필요한 의존성을 **직접 생성하지 않고**
>**외부에서 주입받도록** 설계하는 구조다.

- 클래스가 필요한 객체(의존성)를 직접 만들면 **강결합 상태**가 됨
- 테스트나 교체, 확장이 어려워짐
```js
class UserService {
  constructor() {
    this.logger = new Logger(); // 강결합
  }
}
```
- Logger가 바뀌면 UserService도 고쳐야 함
- 테스트에서 MockLogger 같은 걸 주입 불가

## ② 예시

### ❶ 강결합 싱글톤 예시

```javascript
class DrawingBoard {
  static getInstance() {
    if (!this.instance) {
      const logger = new Logger(); // 내부에서 의존성 생성
      this.instance = new DrawingBoard(logger);
    }
    return this.instance;
  }
}
```
## ❷ 간결합으로 전환

```javascript
class DrawingBoard {
  constructor(logger) {
    this.logger = logger;
  }

  static getInstance(logger) {
    if (!this.instance) {
      this.instance = new DrawingBoard(logger); // 외부 주입
    }
    return this.instance;
  }
}
```

- 외부에서 어떤 Logger든 주입 가능
- 테스트에서는 가짜 Logger(mock)로 주입 가능

## ③ DI가 주는 테스트 유연성

```javascript
class MockLogger {
  log(msg) {
    console.log(`[MOCK] ${msg}`);
  }
}
// 테스트 코드
const mockLogger = new MockLogger();
const service = new UserService(mockLogger);
service.createUser("테스트 유저"); // ✅ 테스트 가능
```
## ④ DI는 어떤 패턴에서도 활용 가능

| 패턴     | DI 가능 여부 | 설명                              |
| ------ | -------- | ------------------------------- |
| 싱글톤    | 가능       | getInstance에 인자로 주입             |
| 멀티톤    | 가능       | getInstance(key, dependency) 구조 |
| 팩토리 패턴 | 매우 자연스러움 | 생성자를 감싸는 함수에서 주입함               |
## ⑤ DI는 어떻게 구현될까?

|방식|설명|예시|
|---|---|---|
|생성자 주입|인스턴스 생성 시 함께 전달|`new A(dependency)`|
|메서드 주입|메서드 호출 시 의존성 전달|`a.setLogger(logger)`|
|프로퍼티 주입|객체 속성으로 직접 할당|`a.logger = logger`|

## 정리

|항목|강결합|DI 적용 (간결합)|
|---|---|---|
|유연성|낮음|높음|
|테스트 가능성|낮음|높음|
|변경 대응력|약함|강함|
|OCP, SRP|위반 가능성 있음|유지 쉬움|
>DI는 설계를 `open`하기 위한 기본 기법이자,
>**유지보수성과 테스트성을 동시에 확보하는 필수 요소**이다.

- 테스트 가능하게 만들고 싶을 때
- 의존 대상을 교체할 가능성이 있을 때
- 클래스 내에서 `new`가 자주 등장할 때
- 환경(운영/개발 등)에 따라 다른 구현을 써야 할 때

>DI를 이용해 설계하면 된다🔥
