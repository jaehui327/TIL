# 🚀 Ta-Da World - 실시간 GPS 기반 보물찾기 게임

[🔗 GitHub Repository 바로가기](https://github.com/jaehui327/Ta-Da-World)

---

## 🧾 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **프로젝트명** | Ta-Da World |
| **참여 인원** | 6인 |
| **기간** | 2023.04.10 ~ 2023.05.19 |
| **개요** | 실시간 GPS 기반 보물찾기 게임으로, 사용자의 위치를 기반으로 보물 탐색 및 사진 인식 기능을 제공 |
| **기술 스택** | - **Frontend**: React<br>- **Backend**: Spring Boot, JPA, Swagger<br>- **Database**: MySQL |

---

## 🧑‍💻 담당 역할

- **Backend**
  - FastAPI 모델과의 연동을 위한 RestTemplate 구현
- **CI/CD**
  - 전체 CI/CD 환경 설계 및 구축
  - Jenkins 기반 Pipeline 구성
  - nginx 서버 설정 및 Docker 기반 배포 자동화

---

## 🔧 주요 기능 및 구현 과정

### 1️⃣ CI/CD 구축

- **Jenkins Pipeline 구현**
  - GitLab과 연동하여 `develop` 브랜치에 푸시 발생 시 자동으로 Jenkins Pipeline 실행
  - Pipeline은 프로젝트를 빌드하고 Docker Hub에 이미지 업로드
- **CD 서버 설정**
  - nginx 서버에 SSH로 원격 접속
  - Docker Hub로부터 이미지를 pull 받아 컨테이너 실행
  - 웹페이지는 index.html을 복사 방식으로 빠르게 반영
- **배포 시각화**
  - Jenkins 아이콘을 통해 실시간으로 배포 상태 확인 가능

### 2️⃣ FastAPI 연동 (RestTemplate)

- Python 기반 FastAPI 모델 서버와 연동
- Spring에서 RestTemplate을 활용하여 사진 인식 API 호출
- Front-End는 Spring 서버 하나에만 요청 → 내부적으로 Python 모델과 통신 처리

---

## 🌱 성과 및 배운 점

### ✅ 배운 점

- CI/CD 환경 전반에 대한 설계 및 실습 (Jenkins + Docker + nginx)
- RestTemplate을 활용한 이기종 서버 간 통신 구현

### 🔄 개선 및 보완점

- **무중단 배포**를 위한 Blue-Green Deployment 전략 도입 필요
- 실시간 위치 공유 기능을 고도화 (Socket 기반 로직 개선 예정)

---

## 📎 참고 링크

- GitHub: [https://github.com/jaehui327/Ta-Da-World](https://github.com/jaehui327/Ta-Da-World)
