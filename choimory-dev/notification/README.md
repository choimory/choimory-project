# intro

- Nest.js
- TypeORM
- Postgresql
- MongoDB
- WebSocket
- Redis pub/sub

## choimory-io-noti-external-api

- CQRS
- 프론트와의 REST 통신

## choimory-io-noti-consumer

- Message Queue
- 내부 등록/수정/삭제용 DB와 조회용 DB간의 동기화 처리
- 외부 도메인에서의 DB 변경사항 반영
- 알림 발송

## choimory-io-noti-socket

- WebSocket
- 프론트와의 소켓 실시간 통신
- 이벤트 컨슈머가 Redis pub/sub을 통해 소켓에 알리면, 소켓이 프론트에 해당사항을 실시간 전달