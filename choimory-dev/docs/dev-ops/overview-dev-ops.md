# 서버 구성

- AWS EC2 t3a.medium 단일 인스턴스
- k3s
  - Ingress
  - Istio
  - App Pods (Front, API)
  - DB pods (Postgres, Redis)
  - ArgoCD Pods
  - Monitoring pods (Loki, Grafana, Prometheus)
- Docker
  - Jenkins
- 그 외
  - Docker hub

# refs

- https://x.com/pyrasis/status/1607169960585080832
 
# AWS IAM

- AWS IAM roles 생성
- EC2와 생성된 role을 연결

# AWS Security Group

- 22, 80, 443

# 도메인

- Route53 도메인 구매
- ALB 혹은 Elastic IP 혹은 EC2 퍼블릭 IP로 도메인 DNS 설정
    - ALB: AWS ALB 비용 발생
    - EIP: AWS EIP 비용 발생
    - EC2 퍼블릭 IP: EC2 인스턴스 시작시 동적할당

# 인증서

- nginx 무료 인증서

# k3s

- Ingress - nginx 혹은 ALB -> nginx
    - ALB: AWS ALB 리소스를 생성, 별도 비용 발생
    - nginx: 무료, 인증서 무료, 다양한 커스터마이징
- Istio - 오버 엔지니어링

# 배포 파이프라인

- Jenkins or Github actions 단일 혹은 Jenkins+Github actions 하이브리드 -> jenkins 단독
  - Jenkins: 무료, 플러그인, 커스터마이징, 서버 리소스 소모
  - Github actions: 일정의 무료 사용량 제공, 관리 편의, 보안 자동화

# Container registry

- Docker hub
  - Docker hub: 무료, 단순, Github actions와 쉬운 연동, 무료 Private repo 한개로 제한, Pull 제한, AWS와 네트워크 비용 발생 가능성
  - AWS ECR: AWS 네이티브, 무제한 Private repo, IAM 권한 연동, 유료, 복잡한 인증 
  - Github Container Registry: Github actions와 완벽 연동, 무료, 소스코드와 같은곳에서 관리

# k3s GitOps

- ArgoCD
- k3s pod으로 포함하여 관리
- choimory-project 내부에 작성하여 문서와 함께 집중시켜 관리

# 로그

- Ingress, Pods에서 로그 출력 및 파일 Output
- k3s pod에 포함하여 관리
- Loki+Grafana 혹은 ELK로 로그 관리 -> Loki+Grafana가 소규모에 더 적합 -> Loki+Grafana
  - Loki: 로그 저장소. 경량. Grafana labs 제품
  - Grafana: 로그 시각화
  - ELK: ES-로그 저장, Logstash-로그 수집, Kibana: 시각화. 무겁지만 강력

# 모니터링

- k3s pod에 포함하여 관리
- Prometheus
  - k8s 네이티브, 경량

# CI/CD flow

1. Github
   1. dev branch push
2. Jenkins (CI)
   1. Github webhook Jenkins trigger or jenkins scm pulling branch
   2. Jenkins build start
      1. Docker build
      2. Docker hub push
      3. k8s repository의 kustomization.yml 파일 이미지 tag update 후, commit & push
3. ArgoCD (CD)
   1. Recognize Git changed
   2. kustomize build
   3. k3s apply

---

# 검토 의견

## 장점
- 경량 k3s로 리소스 효율적
- GitOps 패턴으로 배포 자동화  
- 서비스 메시 고려로 확장성 대비

## 개선 제안
- **보안**: SSL/TLS, 방화벽, IAM 역할 명시
- **모니터링**: Prometheus/Grafana, 로그 수집 추가
- **백업**: DB 백업 전략 필요
- **네트워킹**: VPC, 서브넷 구성 세부사항
- **리소스**: t3a.small로 모든 워크로드 처리 가능한지 검증 필요

## nginx Ingress vs ALB Ingress 비교

### nginx Ingress
- **장점**: k3s 기본 포함, 비용 무료, 풍부한 기능
- **단점**: EC2 의존적, SSL 인증서 직접 관리

### ALB Ingress  
- **장점**: AWS 관리형, 고가용성, ACM 인증서 자동 관리
- **단점**: 월 $16-20 추가 비용, AWS 종속성

### 추천
- **현재**: nginx Ingress (비용 효율적)
- **확장 시**: ALB Ingress 고려

## IP 종류
- **EC2 퍼블릭 IP**: 재시작 시 변경, 무료, 동적 할당
- **Elastic IP**: 고정 IP, 월 $3-5 비용, 정적 할당
- **프로덕션**: EIP 권장 (도메인 안정성)

## GitOps YAML 관리 방법

### 프로젝트 구조
```
choimory-project/
├── src/                    # 애플리케이션 코드
├── k8s/                   # GitOps YAML 파일들
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── ingress.yaml
│   └── overlays/
│       ├── dev/
│       └── prod/
└── .github/workflows/     # CI 파이프라인
```

### ArgoCD 설정
- Repository URL: 같은 GitHub 리포지토리
- Path: `k8s/overlays/prod`
- Target Revision: `main` 브랜치

### 장점
- 코드와 배포 설정을 한 곳에서 관리
- 버전 동기화 쉬움
- 리포지토리 권한 관리 단순

### 주의사항
- `/k8s` 폴더 변경 시 자동 배포됨
- CI에서 이미지 태그만 업데이트하도록 설정
- main 브랜치 보호 규칙 설정 권장

## 추가 검토 내용

### Jenkins vs GitHub Actions 선택
- GitHub Actions 단독 사용 가능 (서버 배포까지 지원)
- Jenkins + GitHub Actions 하이브리드도 가능
- 소규모 프로젝트: GitHub Actions 권장

### ECR (Elastic Container Registry)
- AWS Docker 이미지 저장소
- 월 $1-5 비용, Docker Hub 대안
- IAM 권한 필요

### 모니터링 스택 최종 선택
- **Loki**: 로그 저장소 (경량, Grafana 네이티브)
- **Grafana**: 시각화 대시보드 
- **Prometheus**: 메트릭 수집 (k8s 네이티브)
- **설치 방법**: k3s Pod로 설치 (Helm 권장)
- **Docker 대신 k3s**: 클러스터 내부 메트릭 자동 수집

## Docker vs k3s Pod 비교 (PostgreSQL, Redis, Jenkins)

### PostgreSQL & Redis

**Docker 외부 운영 장점:**
- 데이터 안정성 (k3s 문제와 분리)
- 백업/복구 용이
- 성능 최적화 가능
- 볼륨 관리 단순

**k3s Pod 운영 장점:**
- 통합 관리 (모든 리소스를 k3s에서)
- 네트워크 단순화
- 스케일링/모니터링 통합
- GitOps로 설정 관리

### Jenkins

**Docker 외부 권장:**
- CI/CD는 k3s와 독립적으로 운영
- k3s 장애 시에도 빌드/배포 가능
- 플러그인/설정 관리 용이
- 보안 격리

### 추천

**소규모 프로젝트:**
- PostgreSQL/Redis: k3s Pod (통합 관리 편의)
- Jenkins: Docker 외부 (독립성)

**운영 환경:**
- PostgreSQL: 외부 RDS 또는 Docker (안정성)
- Redis: k3s Pod (캐시라서 휘발성 OK)
- Jenkins: Docker 외부 (필수)

## k3s Pod DB 환경별 분리

### 분리 방식
```
dev 환경: dev-postgres (독립 DB) - dev-choimory-dev 네임스페이스
prod 환경: prod-postgres (독립 DB) - prod-choimory-dev 네임스페이스
```

**분리 구조:**
1. **네임스페이스 분리**: `dev-choimory-dev` vs `prod-choimory-dev` (choimory-dev는 프로젝트명)
2. **PVC 분리**: 각 환경마다 별도 저장소
3. **Secret 분리**: 환경별 DB 패스워드
4. **Service 분리**: 각 환경의 앱이 자기 DB만 접근

### 장점
- 개발/운영 데이터 완전 분리
- 개발 환경에서 실험해도 운영 DB 안전
- 환경별 독립적인 스키마 변경 가능

### 단점
- 리소스 2배 사용 (dev DB + prod DB)
- 운영 데이터를 dev에서 테스트하기 어려움
- 각 환경마다 백업/관리 필요

### 대안
**공유 DB 방식:**
- 하나의 PostgreSQL에서 database 분리
- `dev_choimory-dev`, `prod_choimory-dev` 스키마 분리
- 리소스 절약하지만 격리 수준 낮음

**결론:** t3a.medium 리소스로는 환경별 분리가 더 안전하고 권장