# 🎙️ VO:DA - 음성 감정 일기 서비스

[🔗 GitHub Repository 바로가기](https://github.com/jaehui327/VODA)

---

## 🧾 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **프로젝트명** | VO:DA |
| **참여 인원** | 6인 |
| **기간** | 2023.02.20 ~ 2023.04.07 |
| **개요** | 사용자의 음성 기반으로 감정을 분석하고 감정 일기를 작성하는 웹 애플리케이션 |
| **기술 스택** | - **Frontend**: Vue.js<br>- **Backend**: Spring Boot, JPA, Swagger<br>- **Database**: MySQL |

---

## 🧑‍💻 담당 역할

- **Frontend**
  - 감정 곡선을 시각화하는 Chart 및 DatePicker 구현
- **Backend**
  - 데이터베이스 테이블 설계
  - 캘린더 및 감정 차트 관련 API 개발

---

## 🔧 주요 기능 및 구현 과정

### 1️⃣ Java Stream 활용 (함수형 프로그래밍)

- 주별 및 월별 감정 통계 API 구현 시 DB 조회 수를 줄이기 위해 Stream 도입
- Swift에서의 함수형 프로그래밍 경험을 바탕으로 자바에서도 적용하여 성능 향상 및 코드 가독성 개선

### 2️⃣ Chart 및 DatePicker 구현

- `Chart.js`를 이용한 시각화로 감정 변화를 직관적으로 표시
- 문장별/주별/월별로 감정 곡선을 표시
- Vuex를 활용해 상태를 효율적으로 관리하며 사용자 경험 개선

### 3️⃣ JPA Entity 매핑

- 테이블 간 연관관계를 명확히 정의하여 복잡한 쿼리 없이 원하는 데이터를 쉽게 조회
- 유지보수와 확장성 높은 설계를 구현

---

## 🌱 성과 및 배운 점

### ✅ 배운 점

- 감정 분석을 위한 데이터셋 구성 및 분류 모델 적용
- MyBatis와 JPA의 장단점 비교 후 JPA 선택 및 활용
- Java Stream을 이용한 함수형 프로그래밍 적용 경험

### 🔄 개선 및 보완점

- 사용자 맞춤형 감정 분석 모델 추가 필요
- 음성의 높낮이를 활용한 감정 판별 AI 고도화
- 음성 일기 작성 시 인식 속도 개선 필요

---

## 📎 참고 링크

- GitHub: [https://github.com/jaehui327/VODA](https://github.com/jaehui327/VODA)
