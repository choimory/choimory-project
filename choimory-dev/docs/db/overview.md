# 개요

- Command는 Postgres 공통
- Query는 도메인별로 적합한 NoSQL을 선택
- id는 UUIDv7 통일
- created_at, modified_at(미수정시 null), deleted_at(미삭제시 null) 통일
- 날짜는 Postgres timestamptz 통일