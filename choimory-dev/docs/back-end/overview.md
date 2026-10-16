# 공통

- facade pattern (controller-service-handler-repository)
- cqrs (command and query)
- id, created_at, modified_at, deleted_at은 모든 DB 공통
- 모든 id(PK)는 UUIDv7
- 모든 도메인은 api, grpc, queue를 기본적으로 가지고 있고, 필요에 따라 batch, socket 등을 추가함
    - api: 외부 프론트에서의 호출 담당
    - grpc: 내부 백엔드끼리의 호출 담당 (즉시 응답을 받아야하는 동기적 호출)
    - queue: 내부 백엔드끼리의 호출 담당 (즉시 응답을 받을 필요 없는 대신, 유실이 없어야하는 비동기적 호출)
    - batch: 스케줄링 기반의 정기적 작업이 필요한 도메인만 추가로 사용함
    - socket: 프론트와의 실시간 통신이 필요한 도메인만 추가로 사용함
- RDBMS는 Postgres 공통, NoSQL은 도메인에 적합한 스택을 선택

---

# 라이브러리

## 로깅 라이브러리

- 프로젝트별 공용 로그 관련 라이브러리
- choimory-jvm-logger
- choimory-node-logger

## 토큰 라이브러리

- 프로젝트별 공용 JWT 관련 라이브러리
- choimory-jvm-tokenizer
- choimory-node-tokenizer

---

# api-gateway

- 토큰처리
- 분산추적 로그(opentelemetry)
- 리버스 프록시 (내부 API 숨기기)
- BFF? (도입하면 API와 경계가 모호해질듯? 해도 Next.js Server로 BFF 다는게 나을수도)
- Spring cloud gateway
- Netty
- Webflux
- redis
- 선제적 도입 말고 API 부터 만들고 추후 도입

---

# noti

- 사이트 전반적 알림 담당
- nest.js
- postgres, redis-pubsub
- firebase

## api

- 외부 요청 처리

## gRPC

- 내부 동기 요청 처리

## queue

- 내부 요청 처리(kafka)
- db 싱크

## socket

- front와의 실시간 알림 소켓통신용

---

# sender

- 사이트 전반적 메일, sms, 모바일 푸시 발송 담당
- 비동기
- springboot webflux
- postgres, elasticsearch
- firebase

## api

- 외부 요청 처리

## gRPC

- 내부 동기 요청 처리

## queue

- 내부 비동기 요청 처리 (kafka)
- db 싱크

---

# member

- 사이트 전반적 회원 도메인
- Java 21
- postgres, elastic search, redis
- Spring security로 토큰 처리 → 이후 choimory-token-library로 덜어내기
- 회원가입
- 로그인
- 팔로잉
- 회원가입, 로그인만 빠르게 작성 후 배포 및 자동화 진행
- 배포 후 추가기능 하나씩 개발하여 반영

## api

- 외부 요청 처리

## gRPC

- 내부 동기 요청 처리

## queue

- 내부 비동기 요청 처리 (kafka)
- db 싱크

---

# blog

- 사이트의 블로그 도메인 게시글, 댓글 담당
- kotlin
- postgres, elastic search, redis

## api

- 외부 요청 처리

## gRPC

- 내부 동기 요청 처리

## queue

- 내부 비동기 요청 처리 (kafka)
- db 싱크

---

# memo

- 개인 메모 도메인
- nest.js
- postgres, mongo db

## api

- 외부 요청 처리

## gRPC

- 내부 동기 요청 처리

## queue

- 내부 비동기 요청 처리 (kafka)
- db 싱크

---

# feed

- 내가 관찰하고 싶은 스트리머의 활동 알림
- 내가 구매하고 싶은 물건의 가격변동 알림
- choimory.dev 사이트 + 디스코드 알림봇 연동
- 비동기
- ktor+coroutine
- postgres, elasticsearch

## api

- 외부 요청 처리

## gRPC

- 내부 동기 요청 처리

## queue

- 내부 비동기 요청 처리 (kafka)
- db 싱크

## socket

- front 혹은 디스코드와의 소켓통신용