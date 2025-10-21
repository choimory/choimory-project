# k3s 배포 구성

## 디렉토리 구조

```
k3s/
├── base/                   # 기본 YAML 템플릿 (모든 환경에서 공유하는 공통 설정)
│   ├── namespace.yaml      # choimory-dev (프로젝트 기본), monitoring, argocd 네임스페이스
│   ├── deployment.yaml     # Front, API 애플리케이션 배포
│   ├── service.yaml        # 서비스 정의
│   ├── ingress.yaml        # nginx Ingress (SSL/TLS 포함)
│   ├── postgres.yaml       # PostgreSQL 데이터베이스
│   ├── redis.yaml          # Redis 캐시
│   ├── monitoring.yaml     # Prometheus, Grafana 모니터링
│   └── kustomization.yaml  # base 설정
└── overlays/               # 환경별 설정 (base 위에 덮어씌울 환경별 차이점)
    ├── dev/                # 개발 환경 overlay
    │   ├── kustomization.yaml   # dev 환경 kustomize 설정
    │   └── replica-patch.yaml   # dev 환경 리소스 패치
    └── prod/               # 운영 환경 overlay
        ├── kustomization.yaml   # prod 환경 kustomize 설정
        └── replica-patch.yaml   # prod 환경 리소스 패치
```

## 주요 구성 요소

### 애플리케이션
- **choimory-front**: React 프론트엔드 (포트 3000)
- **choimory-api**: Spring Boot API 서버 (포트 8080)

### 데이터베이스
- **PostgreSQL**: 메인 데이터베이스 (포트 5432)
- **Redis**: 캐시 서버 (포트 6379)

### 모니터링
- **Prometheus**: 메트릭 수집 (포트 9090)
- **Grafana**: 시각화 대시보드 (포트 3000)

### 네트워킹
- **nginx Ingress**: SSL/TLS 자동 인증서 관리
- **도메인**: choimory.dev (프론트), api.choimory.dev (API)

## 환경별 설정

### Dev 환경 (dev-choimory-dev)
- Front/API 각 1개 replica
- 낮은 리소스 할당 (개발용)
- 개발용 이미지 태그 (dev-latest)
- Spring Profile: dev
- namePrefix: dev- 적용

### Prod 환경 (prod-choimory-dev)
- Front/API 각 2개 replica (고가용성)
- 높은 리소스 할당 (운영용)
- 버전 태그 (v1.0.0)
- Spring Profile: prod
- namePrefix: prod- 적용

## 배포 명령어

### Dev 환경 배포
```bash
kubectl apply -k k3s/overlays/dev
```

### Prod 환경 배포
```bash
kubectl apply -k k3s/overlays/prod
```

### 상태 확인
```bash
# Dev 환경
kubectl get pods -n dev-choimory-dev
kubectl get svc -n dev-choimory-dev
kubectl get ingress -n dev-choimory-dev

# Prod 환경
kubectl get pods -n prod-choimory-dev
kubectl get svc -n prod-choimory-dev
kubectl get ingress -n prod-choimory-dev
```

## 보안 설정

### Secrets
- `postgres-secret`: PostgreSQL 패스워드
- `redis-secret`: Redis 패스워드
- `choimory-tls`: SSL/TLS 인증서

### 인증서 관리
- cert-manager를 통한 Let's Encrypt 자동 인증서
- nginx.ingress.kubernetes.io/ssl-redirect: "true"

## 리소스 할당

### Dev 환경
- Front: 64Mi/50m (요청) → 128Mi/100m (제한)
- API: 128Mi/100m (요청) → 256Mi/200m (제한)

### Prod 환경
- Front: 128Mi/100m (요청) → 256Mi/200m (제한)
- API: 256Mi/200m (요청) → 512Mi/400m (제한)
- PostgreSQL: 512Mi/300m (요청) → 1Gi/500m (제한)

## GitOps 연동

ArgoCD 설정:
- Repository: choimory-project
- Path: `k3s/overlays/prod`
- Target Revision: `main`
- Auto-sync: enabled

## Kustomize

Kubernetes YAML 파일을 관리하는 도구

### 주요 기능
- YAML 파일을 수정하지 않고 커스터마이징
- 환경별 설정 관리 (dev, staging, prod)
- 템플릿 없이 순수 YAML 기반

### 동작 방식
1. 기본 YAML 파일들 (base)
2. 환경별 수정사항 (overlays)
3. kustomization.yaml로 어떻게 합칠지 정의
4. `kubectl apply -k` 명령으로 최종 YAML 생성 후 배포

### 장점
- Helm보다 단순함 (템플릿 문법 없음)
- Kubernetes 네이티브 (kubectl에 내장)
- 원본 YAML 파일 보존
- 환경별 차이점만 관리

### 사용 예시
```bash
# base YAML + dev overlay 적용하여 배포
kubectl apply -k overlays/dev

# base YAML + prod overlay 적용하여 배포  
kubectl apply -k overlays/prod
```

> 즉, "원본은 건드리지 않고 환경별로 필요한 부분만 수정해서 배포하는 도구"

## Overlays란?

Kustomize에서 사용하는 환경별 설정 관리 방식입니다.

### 기본 구조
- `base/`: 공통 YAML 템플릿 (모든 환경에서 공유하는 기본 설정)
- `overlays/`: 환경별 커스터마이징 (dev, prod 등 환경별 차이점)

### 동작 방식
1. `base/`의 기본 설정을 가져옴
2. `overlays/dev/` 또는 `overlays/prod/`에서 환경별 수정사항 적용
3. 최종 YAML 생성하여 배포

### 예시
- **base**: replica 1개, 기본 리소스, 공통 설정
- **dev overlay**: 개발용 설정 (낮은 리소스, dev 이미지 태그)
- **prod overlay**: 운영용 설정 (높은 리소스, 2개 replica, prod 이미지 태그)

### 장점
- 코드 중복 없이 환경별 설정 관리
- base 수정하면 모든 환경에 자동 반영
- 환경별 차이점만 overlay에서 관리
- 버전 관리 및 추적 용이

> overlay = "덮어씌우는 층"이라는 뜻으로, 기본 설정 위에 환경별 설정을 덮어씌워서 최종 배포 설정을 만드는 방식

## 참고사항

1. **네임스페이스 분리**: 애플리케이션(choimory-dev), 모니터링(monitoring), GitOps(argocd)
2. **Kustomize 활용**: base/overlays 패턴으로 환경별 설정 관리
3. **리소스 제한**: t3a.medium 인스턴스에 적합한 리소스 할당
4. **모니터링**: Prometheus/Grafana로 클러스터 상태 모니터링
5. **저장소**: PVC를 통한 PostgreSQL 데이터 영속성