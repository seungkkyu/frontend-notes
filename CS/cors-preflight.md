# CORS Preflight 정리

## 1. Preflight란?

Preflight는 **CORS 과정 중 하나**로,  
브라우저가 실제 요청을 보내기 전에 서버에 먼저 확인하는 절차입니다.

이때 브라우저는 `OPTIONS` 메서드로 요청을 먼저 보냅니다.

---

## 2. 언제 Preflight가 발생할까?

Preflight는 아래 조건을 모두 만족할 때 발생합니다:

> **Cross-Origin 요청이면서 + 단순 요청(Simple Request)이 아닐 때**

---

## 3. 왜 Preflight가 필요할까?

### 1️⃣ 배경: SOP만으로는 부족함

브라우저에는 **SOP(Same-Origin Policy)** 가 있어서  
다른 출처의 응답을 JS에서 읽지 못하게 막고 있습니다.

하지만 중요한 점은:

> 응답을 읽지 못하더라도 **요청 자체는 서버에 도달할 수 있습니다.**

즉, 공격자는 피해자의 브라우저를 이용해  
다른 사이트에 **의도치 않은 요청을 보낼 수 있습니다.**

---

### 2️⃣ 위험한 요청을 그냥 보내면 생기는 문제

예를 들어:

```js
fetch('https://bank.com/transfer', {
  method: 'DELETE',
  headers: {
    Authorization: 'Bearer victim-token'
  }
});
```

이 요청이 아무 검증 없이 전송된다면:

사용자의 쿠키/토큰이 자동으로 포함될 수 있음

서버 상태 변경 가능

CSRF / 권한 오남용 위험 증가

그래서 브라우저가 먼저 서버에 묻습니다:

"이런 메서드와 헤더로 요청을 보내도 되나요?"

### 3️⃣ Preflight의 역할

Preflight는 일종의 사전 허가 절차입니다.

브라우저는 먼저 OPTIONS 요청을 보내 다음을 확인합니다:

이 Origin을 허용하는지?

이 HTTP 메서드를 허용하는지?

이 헤더를 허용하는지?

서버가 명시적으로 허용해야만 실제 요청을 전송합니다.

---

## 4. 단순 요청(Simple Request)이란?

아래 조건을 모두 만족하면 단순 요청이며,
이 경우 Preflight는 발생하지 않습니다.

### ① HTTP 메서드

GET

HEAD

POST

### ② Content-Type (POST일 경우)

아래 중 하나만 허용됩니다:

application/x-www-form-urlencoded

multipart/form-data

text/plain

### ③ 커스텀 헤더 없음

다음과 같은 헤더가 있으면 단순 요청이 아닙니다:

Authorization

X-Custom-\*

Content-Type: application/json

기타 임의 헤더

---

## 5. 예시

### ✔ Preflight 없음 (단순 요청)

```js
fetch('https://api.example.com/data', {
  method: 'GET'
});
```

### ❌ Preflight 발생

```js
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ a: 1 })
});
```

→ application/json 때문에 브라우저가 먼저 OPTIONS 요청을 보냅니다.

---

## 6. Preflight 요청 흐름

### ① 브라우저가 먼저 OPTIONS 요청 전송

OPTIONS /api

Origin: https://frontend.com

Access-Control-Request-Method: POST

Access-Control-Request-Headers: Content-Type

### ② 서버가 허용 여부 응답

Access-Control-Allow-Origin: https://frontend.com

Access-Control-Allow-Methods: POST

Access-Control-Allow-Headers: Content-Type

### ③ 허용되면 실제 요청 전송

---

## 7. 주의사항

Same-Origin이면 CORS 자체가 적용되지 않음

포트가 다르면 Cross-Origin

localhost:3000 ↔ localhost:8080

application/json만 사용해도 Preflight 발생

Authorization 헤더 사용 시 Preflight 발생

## 한 줄 요약

단순 요청이면 Preflight는 발생하지 않는다.
단, Cross-Origin 요청일 때만 CORS / Preflight 개념이 적용된다.
