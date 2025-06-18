
# 📘 CI/CD 구성요소 이론 정리

이 문서에서는 CI/CD 파이프라인을 구성하는 주요 도구 및 기술 요소들의 **개념과 역할**에 대해 설명합니다.

---

## 🐳 Kubernetes (K8s)

**정의:**  
오픈소스 컨테이너 오케스트레이션 플랫폼으로, 수천 개의 컨테이너를 자동으로 배포, 확장, 관리할 수 있게 해줍니다.

**주요 기능:**
- 컨테이너 자동 배포 및 롤백
- 서비스 디스커버리 및 로드밸런싱
- 스토리지 오케스트레이션
- 자동화된 Rollout / Rollback
- 자가 복구 (Self-healing)
- 수평 확장 (Horizontal Scaling)

**구성 요소:**
- **Master Node**: 클러스터 제어 (API Server, Scheduler, Controller 등)
- **Worker Node**: 실제 애플리케이션이 실행되는 Pod를 호스팅

---

## 🚀 ArgoCD

**정의:**  
GitOps 방식의 Kubernetes 애플리케이션 배포 도구. Git 저장소를 단일 진실 소스로 삼아 자동으로 K8s에 애플리케이션을 배포/동기화합니다.

**주요 기능:**
- Git 저장소 변경 감지 → 자동 배포
- 실시간 상태 모니터링 및 Sync 기능
- Rollback 지원
- Web UI, CLI, API 지원
- RBAC 기반 권한 관리

**GitOps란?**
Git을 단일 소스로 삼아 애플리케이션 상태를 정의하고, 이 상태를 기준으로 클러스터를 자동으로 일치시키는 방식

---

## 🧪 SonarQube

**정의:**  
코드의 품질을 정적 분석(Static Analysis)을 통해 평가하는 도구. 중복 코드, 복잡도, 보안 이슈 등을 탐지합니다.

**주요 기능:**
- 코드 스멜, 버그, 보안 취약점 탐지
- 커버리지, 중복도, 복잡도 리포트 제공
- 다양한 언어 지원 (Java, JS, Python 등)
- GitLab, GitHub 연동 가능

---

## 📦 Nexus

**정의:**  
사설 저장소(Private Repository)로 Maven, npm, Docker 등의 아티팩트 저장소 역할을 수행합니다.

**주요 기능:**
- Maven 중앙 저장소 캐싱
- 아티팩트 저장 및 버전 관리
- 보안 및 접근 제어
- 내부에서만 사용할 수 있는 패키지 저장소 구성 가능

---

## 🏗️ Kaniko

**정의:**  
Docker 데몬 없이 컨테이너 이미지를 빌드할 수 있게 해주는 빌드 도구. Kubernetes 환경에서 안전하게 동작함.

**주요 기능:**
- Dockerfile을 기반으로 이미지 생성
- Docker-in-Docker 없이 작동 → 보안 향상
- Kubernetes와 잘 통합됨

---

## 🗂️ Harbor

**정의:**  
Cloud Native Computing Foundation(CNCF)에서 관리하는 오픈소스 이미지 저장소. 보안과 사용자 관리를 강화한 Docker Registry입니다.

**주요 기능:**
- Role 기반 접근 제어 (RBAC)
- 이미지 서명 및 취약점 스캔
- LDAP, OIDC 인증 연동
- Replication 지원
- Web UI 제공

---

## 🔧 Maven

**정의:**  
Java 프로젝트의 빌드 및 의존성 관리를 위한 도구. XML 기반의 POM 파일로 설정

**주요 기능:**
- 라이프사이클 기반 빌드
- 플러그인 확장성
- 의존성 자동 다운로드 (Nexus와 연동 가능)
- 테스트, 패키징, 배포 등 자동화

---

## ✅ 정리

| 구성 요소 | 핵심 역할 |
|-----------|------------|
| Kubernetes | 컨테이너 실행, 오케스트레이션 |
| ArgoCD     | GitOps 기반 자동 배포 |
| SonarQube  | 코드 품질 분석 및 리포팅 |
| Nexus      | 사설 아티팩트 저장소 |
| Kaniko     | 이미지 빌드 (Dockerless) |
| Harbor     | 이미지 저장소 및 보안 관리 |
| Maven      | 빌드 및 의존성 관리 |

