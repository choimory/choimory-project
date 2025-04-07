# 스펙

- Language
  - Java, Springboot, JPA
  - Kotlin, Springboot, JPA
  - TS, Nest.js, TypeORM
- DB
  - Postgresql
  - Redis

---

# 기능

- 회원 관련 API
- 회원 API를 개발하며 백엔드로 다룰수 있는 많은 내용들을 다뤄보기
- 다양한 언어로 작성해도 프론트에 문제가 없도록 작성하기

## 인증

- 회원 가입 및 로그인 패스워드 암호화
- CORS
- bcrypt
  - Spring security
  - bcrypt, @types/bcrypt
- JWT or Session
  - JJWT
  - Nestjs JWT
- OAuth2
  - Spring security
  - Nest.js Passport

## 권한

- 회원 권한별 접근처리
- Spring security
- Nest.js auth, guard

## 타입

- 타입을 DB화하여 관리할 수도 있고, 코드의 enum으로 관리할 수도 있다
- 회원 역할(DB), 이미지 타입(enum)
- Java Kotlin enum
- TypeScript enum, union type as const(권장)

## 파일 입출력, 스토리지, CDN

- 회원 프로필 사진, 게시물 첨부 이미지 파일 업로드/다운로드 구현
- Spring MultiPartFile
- Nestjs Multer
- 스토리지 저장
- 리사이징
- CDN

## 이메일 인증

- 회원가입 인증메일
- Java mail sender
- Nestjs mailer
- Redis
  - 인증번호를 TTL(time to live)걸어서 저장시키면 시간내에 안올시 세션 죽음

## 배치

- 이용정지회원 정지해제 배치처리
- 탈퇴 1년 회원 데이터 영구삭제 배치처리
- Spring batch

## 연관 관계

- 1:1 -
- 1:N, N:1 - 회원 정지내역, 게시물 이미지
- N:N - 회원 팔로우, 회원 역할(1:N 설계도 가능하긴 함), 역할 권한, 게시물 좋아요

## 계층형 테이블

- 게시물 댓글

## 복합키 엔티티

- 다대다 중간 테이블

## Redis

- 캐싱
- 세션
- 이메일, OTP 인증번호 일정시간 저장
- 로그인 실패횟수 IP 저장
- 활동이력 저장

## MongoDB

- TODO

## 웹소켓

- 채팅

---

# Entity

- id는 UUIDv7을 사용

## Member `회원`
  - id pk
  - email uk
  - password notnull
  - introduce text

## MemberImage `회원 이미지` `1:N`
  - id pk
  - member_id fk
  - type varchar(code enum - ICON, PROFILE...) notnull
  - name
  - path 
  - size 
  - resize_name
  - resize_path 
  - resize_size

## MemberSuspension `회원 정지내역` `1:N`
  - id pk
  - member_id fk
  - reason
  - suspended_to timestamp
  - suspended_at timestamp

## MemberPosition `회원 등급` `N:N`
  - id pk

## Position `등급`
  - id pk

## PositionPermission `등급별 권한` `1:N`
  - id pk

## Permission `권한`
  - id pk

## TODO (MongoDB)
  - id
  - member_id
  - 문서내용 json

## Memo (MongoDB)
  - id
  - member_id

---

# API

## 회원 목록조회

- GET /member

## 회원 상세조회

- GET /member/{id}

## 회원 가입

- POST /member/join

## 회원 가입확인 

- POST /member/join/confirm

## 회원 수정

- PUT /member