# choimory-dev

- 개인 공부용 -> 공부하고 싶거나 필요한 기능들 하나씩 추가
- 회원
- 블로그 -> 회원별 개인 블로그
- 게시판 -> 카테고리별 공용 게시판
- 메모 -> 개인 private 메모
- 알림

# 서버 구성

- AWS EC2 t3a.small -> 월 4천원~7천원
- EIP 고정 IP로 고정 진입점 유지, Route53으로 DNS 설정
- 단일 인스턴스 구성 -> 해당 인스턴스에 docker로 api, db 설치
- k3s로 싱글 노드 클러스터 구성 -> docker 관리하고 ingress로 포워딩함
- docker간의 내부 통신

# repositories

- [choimory-dev-architecture](https://github.com/choimory/choimory-architectures/tree/main/choimory-dev)
- [choimory-dev-front](https://github.com/choimory/choimory-dev-front)
- [choimory-dev-member-api](https://github.com/choimory/choimory-dev-member-api)
- choimory-dev-member-queue
- choimory-dev-board-api
- choimory-dev-board-queue
- choimory-dev-memo-api
- choimory-dev-memo-queue
- choimory-dev-noti-api
- choimory-dev-noti-queue
- choimory-dev-noti-socket

# refs

- https://x.com/pyrasis/status/1607169960585080832