# 개요

- 개발서버: AWS on/off 혹은 홈서버
- 운영서버: 홈서버 24 hours
- CI: 홈서버 Jenkins or Github Actions
- CD: ArgoCD (bootstrapped by Helm)
- K8s: K8s or K3s GitOps, Kustomize
- Network: Tailscale or 공유기 Portforwarding
- 그 외
  - Dockerhub
  - DNS: Gabia
  - SSL: Let's encrypt