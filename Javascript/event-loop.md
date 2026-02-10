# Event Loop

## 소개

자바스크립트는 기본적으로 **싱글 스레드(single-thread)** 언어이다.  
즉, 한 번에 하나의 작업만 처리할 수 있다.

그럼에도 불구하고 실제 브라우저 환경에서는 비동기 요청, 타이머, 사용자 이벤트 등이 발생해도
화면이 멈추지 않고 자연스럽게 동작한다.  
이러한 비동기 처리를 가능하게 해주는 핵심 메커니즘이 바로 **이벤트 루프(Event Loop)**이다.

---

## 싱글 스레드란?

- Call Stack은 한 번에 하나의 실행 컨텍스트만 처리
- 오래 걸리는 작업이 Stack을 점유하면 UI가 멈춤
- 따라서 비동기 작업은 **브라우저(Web APIs)** 또는 **런타임 환경**에 위임됨

---

## 이벤트 루프의 역할

이벤트 루프는 다음과 같은 역할을 한다.

1. **Call Stack이 비어있는지 확인**
2. 비동기 작업이 완료되어 대기 중인 Queue가 있는지 확인
3. Queue에 있는 작업을 Call Stack으로 이동시켜 실행

> 즉, 이벤트 루프는  
> **“언제 어떤 비동기 작업을 실행할지 결정하는 관리자”** 역할을 한다.

---

## Task Queue의 종류

이벤트 루프가 관리하는 Queue는 크게 두 가지로 나뉜다.

### 1️⃣ Microtask Queue

- 현재 실행 중인 작업이 끝난 직후 **가장 먼저 처리**
- Call Stack이 비워지면 즉시 실행

#### 대표 예시

- `Promise.then / catch / finally`
- `async / await`
- `queueMicrotask`

---

### 2️⃣ Macrotask Queue (Task Queue)

- Microtask Queue가 모두 비워진 후 실행

#### 대표 예시

- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- `requestAnimationFrame`

---

## 실행 우선순위

- Microtask는 **Macrotask보다 항상 먼저 실행**
- Microtask가 계속 추가되면 Macrotask가 실행되지 않을 수도 있음

---

## 실행 순서 예제

다음 코드를 실행하면 어떤 순서로 출력될까?

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

console.log("4");
```

## 실행 결과 설명

**결과**

- 1 → 4 → 3 → 2

**실행 과정**

1. `1`, `4`는 동기 코드로 Call Stack에서 즉시 실행된다.
2. `Promise.then`은 Microtask Queue에 등록된다.
3. `setTimeout`은 Macrotask Queue에 등록된다.
4. Call Stack이 비워진 후  
   → Microtask Queue가 먼저 실행되고  
   → 이후 Macrotask Queue가 실행된다.

---
