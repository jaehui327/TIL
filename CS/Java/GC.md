# Java Garbage Collection (GC) 정리

## 📌 1. GC란?

Garbage Collection(GC)은 더 이상 참조되지 않는 객체를 자동으로 메모리에서 해제하여 **메모리 누수 방지 및 자원 최적화**를 도와주는 Java의 핵심 기능이다.

- **목적**: Heap 메모리 누수 방지, 개발자 부담 감소
- **장점**: 수동 메모리 관리 불필요
- **단점**: Stop-the-world로 인한 성능 이슈 발생 가능

---

## 🧠 2. Java 메모리 구조

### 2.1 전체 메모리 구조

- **Heap**: 대부분의 객체가 저장되는 공간 (GC의 대상)
- **Stack**: 각 스레드별 호출 스택, 지역 변수 저장
- **Method Area(Metaspace)**: 클래스 메타 정보 저장
- **Program Counter**: 현재 실행 중인 명령어의 주소
- **Native Method Stack**: C/C++ 등 네이티브 메서드용 스택

### 2.2 Heap 내부 구조

- **Young Generation**: 새로 생성된 객체가 저장됨. GC가 자주 일어남
  - Eden: 객체 생성 시 최초 저장
  - Survivor 0/1: GC 이후 살아남은 객체가 임시 이동
- **Old Generation**: Young에서 오래 살아남은 객체 저장
- **Metaspace**: 클래스 메타정보 저장 (PermGen 대체, Java 8부터)

---

## 🔁 3. GC 동작 원리

### 3.1 Minor GC

- Young Generation에만 적용
- 비교적 빠름
- 대부분의 객체는 이 단계에서 소멸

### 3.2 Major GC (Old GC)

- Old Generation 대상으로 수행
- 상대적으로 느림
- 애플리케이션 일시 정지 발생 가능

### 3.3 Full GC

- Heap 전체 및 Metaspace까지 모두 청소
- 성능 저하 원인이 되기 쉬움

---

## ⚙️ 4. GC 알고리즘 종류 (상세 설명)

### 4.1 Serial GC

- **특징**: 단일 스레드로 GC 작업 수행 (Minor/Full GC 모두 단일 스레드)
- **장점**: 구현이 단순하여 오버헤드 적음
- **단점**: 멀티 코어 시스템에서 성능 미흡, 애플리케이션 정지 시간 길어짐
- **적용 예시**: 단일 스레드 환경 또는 소규모 테스트/개발 환경

### 4.2 Parallel GC (Throughput Collector)

- **특징**: 여러 스레드를 사용해 Young Generation과 Old Generation을 모두 병렬로 GC
- **장점**: Throughput(전체 처리량) 극대화
- **단점**: Stop-the-world 발생, 짧은 지연 시간 요구 환경에는 부적합
- **적용 예시**: 배치 처리, CPU 중심 애플리케이션, 백엔드 서버

### 4.3 CMS (Concurrent Mark-Sweep) GC

- **특징**: Old 영역에 대해 대부분의 작업을 애플리케이션 실행 중 백그라운드에서 수행
- **단계**:
  1. 초기 마킹 (STW)
  2. 마킹 (Concurrent)
  3. 리마킹 (STW)
  4. 스윕 (Concurrent)
- **장점**: 긴 정지 시간 방지
- **단점**: 메모리 단편화 발생, CPU 자원 소모 큼
- **Deprecated**: Java 9부터 G1 GC로 대체 권장

### 4.4 G1 GC (Garbage First)

- **특징**: Heap을 고정된 크기의 Region으로 나누고, GC 효율이 높은 영역부터 우선 수거
- **장점**:
  - 짧고 예측 가능한 pause time
  - Full GC 빈도 낮음
  - Region 단위로 메모리 할당 및 수거 가능
- **단점**: Throughput은 ParallelGC보다 낮을 수 있음
- **적용 예시**: 대규모 애플리케이션, Pause Time 중요시하는 서비스

### 4.5 ZGC (Java 11 이상)

- **특징**: 초저지연 GC, 최대 수백 GB Heap까지 지원
- **장점**:
  - pause time < 10ms (고정)
  - 대부분의 GC 작업을 동시에 수행
- **단점**: JVM 버전 제약(Java 11 이상), 초기 도입 비용
- **적용 예시**: 대규모 금융 시스템, 게임 서버, 실시간 처리 시스템

### 4.6 Shenandoah GC

- **특징**: RedHat이 주도 개발, ZGC처럼 낮은 pause time 지향
- **장점**:
  - pause time < 10ms
  - Concurrent Compaction 지원 (ZGC는 미지원)
- **단점**: 운영환경에서 안정성 고려 필요
- **적용 예시**: Java 12 이상, 실시간 응답 시스템, Linux 기반 서비스

### 4.1 Serial GC

- 단일 스레드로 GC 처리
- Client 모드 또는 소규모 애플리케이션에 적합

### 4.2 Parallel GC (Throughput Collector)

- 다중 스레드로 GC 수행
- Throughput 최적화

### 4.3 CMS (Concurrent Mark Sweep) GC

- Old Generation의 GC를 대부분 백그라운드로 수행
- 낮은 응답시간 목표
- Java 9부터 deprecated

### 4.4 G1 GC (Garbage First)

- Region 기반 Heap 분할
- Young/Old 영역 동적으로 관리
- Pause Time 예측 가능

### 4.5 ZGC

- 초저지연 GC (pause time < 10ms)
- Java 11 이상
- 수백 GB Heap에서도 효율적

### 4.6 Shenandoah

- ZGC와 유사, RedHat 제공
- Java 12 이상
- 낮은 지연 지향

---

## 🛠️ 5. GC 튜닝 주요 옵션

### 5.1 GC 타입 설정

```
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseG1GC
-XX:+UseZGC
```

### 5.2 Heap 사이즈 설정

```
-Xms512m        # 초기 힙 사이즈
-Xmx4g          # 최대 힙 사이즈
```

### 5.3 GC 로그 옵션

```
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:gc.log
```

---

## ✅ 6. 요약 정리

- Java는 GC를 통해 자동 메모리 관리
- Minor GC, Major GC, Full GC 개념 이해 중요
- G1 GC, ZGC 등 선택 가능한 GC 다양
- 애플리케이션 성격에 따라 GC 튜닝 필요

### Q1. Java에서 GC가 필요한 이유는 무엇인가요?

A1. Java는 객체를 동적으로 생성하고 소멸하는 언어이기 때문에, 수동으로 메모리를 해제해야 한다면 개발자의 부담이 커지고 메모리 누수 가능성도 높아집니다. GC는 이러한 부담을 줄이고, 더 이상 사용되지 않는 객체를 자동으로 해제해 메모리 자원을 효율적으로 관리합니다.

---

### Q2. Minor GC와 Major GC의 차이는?

A2. Minor GC는 Young Generation을 대상으로 하는 GC로, 짧은 주기로 자주 발생하며 속도가 빠릅니다. Major GC는 Old Generation을 대상으로 하며, 상대적으로 느리고 pause time이 길 수 있습니다. Major GC는 메모리 압박이 심해질 때 수행됩니다.

---

### Q3. G1 GC는 어떻게 동작하나요?

A3. G1 GC는 Heap을 Region이라는 고정 크기 블록으로 나누고, GC 효율이 높은 영역부터 우선 수거하는 방식입니다. Young/Old 영역 구분 없이 Region 단위로 관리하며, Pause Time을 예측 가능하게 조정할 수 있는 것이 특징입니다.

---

### Q4. GC 튜닝 시 어떤 지표를 참고하나요?

A4. 주요 지표는 다음과 같습니다:

- GC 빈도 (Minor/Full GC 횟수)
- GC 수행 시간 (Pause Time)
- Heap 사용률 및 증가 추세
- Throughput (총 실행 시간 중 GC를 제외한 비율)
- 메모리 단편화 여부 (CMS 사용 시 중요)

---

### Q5. ZGC와 G1 GC의 차이점은 무엇인가요?

A5. G1 GC는 pause time을 예측할 수 있으나 여전히 stop-the-world가 존재합니다. 반면 ZGC는 대부분의 GC 작업을 백그라운드에서 수행하며 pause time이 10ms 미만으로 고정되어 있습니다. 또한 ZGC는 더 큰 Heap (TB 단위)까지 효율적으로 관리할 수 있습니다.
