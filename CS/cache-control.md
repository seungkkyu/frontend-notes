# 📘 Cache-Control 정리

---

# 1️⃣ Cache-Control이란?

`Cache-Control`은 HTTP **응답 헤더(Response Header)** 이며,  
해당 리소스를 **어떻게 캐싱할지 서버가 브라우저나 CDN에게 지시하는 정책**이다.

👉 기본적으로 **서버가 설정하는 것이 원칙**

---

# 2️⃣ 주요 옵션 정리

## ✅ public

- 브라우저 + CDN + 프록시 캐시 가능

```
Cache-Control: public
```

정적 리소스에서 주로 사용

---

## ✅ private

- 사용자 브라우저에만 저장 가능
- CDN(공유 캐시) 저장 불가

```
Cache-Control: private
```

로그인 정보, 사용자 맞춤 데이터 등에 사용

---

## ✅ max-age

- 캐시 유효 시간 (초 단위)

```
Cache-Control: max-age=3600
```

→ 1시간 동안 서버에 재요청하지 않음

---

## ✅ must-revalidate

- 캐시가 만료되면 반드시 서버에 재검증

```
Cache-Control: max-age=0, must-revalidate
```

→ 항상 서버에 확인

---

## ✅ immutable

- 유효기간 동안 절대 변경되지 않음
- 재검증도 하지 않음

```
Cache-Control: max-age=31536000, immutable
```

보통 해시가 포함된 파일에 사용

```
main.aj39dk2.js
```

---

## ✅ no-store

- 절대 저장하지 말 것

```
Cache-Control: no-store
```

토큰, 민감 정보 등

---

## ✅ no-cache

- 저장은 가능
- 사용 전에 반드시 서버에 재검증

```
Cache-Control: no-cache
```

⚠️ "캐시하지 말라"는 의미가 아님

---

# 3️⃣ ETag

## ✅ 개념

리소스의 해시값을 기반으로 변경 여부를 판단하는 헤더

### 서버 응답

```
ETag: "abc123"
```

### 다음 요청 시

```
If-None-Match: "abc123"
```

같으면:

```
HTTP 304 Not Modified
```

→ body 없이 헤더만 응답  
→ 네트워크 비용 절약

---

# 4️⃣ 자주 쓰는 조합

## 🔵 항상 서버 확인

```
Cache-Control: max-age=0, must-revalidate
```

또는

```
Cache-Control: no-cache
```

API 응답에서 자주 사용

---

## 🔵 정적 리소스 (해시 포함 파일)

```
Cache-Control: public, max-age=31536000, immutable
```

JS 번들, 이미지, 폰트 등

---

## 🔵 로그인 / 민감 데이터

```
Cache-Control: no-store
```

---

# 5️⃣ Cache-Control은 누가 설정하나?

## ✅ 원칙: 서버가 설정

이유:

- 리소스 특성
- 보안 정책
- CDN 전략

은 서버가 결정하기 때문

---

## ❓ 클라이언트에서 설정 가능?

요청 시 이렇게 보낼 수는 있음:

```
Cache-Control: no-cache
```

→ "나는 캐시 사용하지 말고 새로 달라"는 요청

하지만  
👉 최종 정책은 서버 응답 헤더가 결정

---

# 6️⃣ 강한 캐시 vs 약한 캐시

## 🔵 강한 캐시 (Strong Cache)

- 유효기간 동안 서버에 요청하지 않음
- `max-age` 기반

## 🔵 약한 캐시 (Revalidation Cache)

- 서버에 변경 여부 확인 후 사용
- `ETag`, `If-None-Match`
- `Last-Modified`

---

# 7️⃣ 핵심 정리

| 옵션            | 의미                  |
| --------------- | --------------------- |
| public          | CDN 포함 캐시 가능    |
| private         | 브라우저만 캐시       |
| max-age         | 유효 기간             |
| no-cache        | 사용 전 서버 재검증   |
| no-store        | 저장 금지             |
| must-revalidate | 만료 후 반드시 재검증 |
| immutable       | 절대 변경 없음        |
| ETag            | 리소스 해시값         |

---

# 📦 결론

- Cache-Control은 서버가 설정하는 것이 원칙
- `immutable`은 해시 파일에서만 안전
- `no-cache`는 "저장 금지"가 아니라 "사용 전 검증"
- 304 응답은 body 없이 빠르게 응답

현대 웹 성능 최적화의 핵심 개념이다.
