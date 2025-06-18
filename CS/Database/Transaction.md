# 트랜잭션(Transaction)

## 🧩 트랜잭션이란?

- 데이터베이스에서 **작업의 논리적 단위**를 말함
- **ACID 원칙**을 만족해야 함:
  - **A**tomicity (원자성)
  - **C**onsistency (일관성)
  - **I**solation (격리성)
  - **D**urability (지속성)

---

## 🚦 Spring에서의 트랜잭션 관리

- `@Transactional` 어노테이션으로 선언적 트랜잭션 제어
- 트랜잭션 경계는 **프록시 기반 AOP**로 설정됨

```java
@Transactional
public void doSomething() {
    // 이 메서드는 트랜잭션 내에서 실행됨
}
```

---

## 🔄 Propagation(전파 속성)이란?

- 하나의 트랜잭션 안에서 **다른 트랜잭션과 어떻게 연관되는지** 결정하는 전략

| 속성                | 설명                                                  |
| ------------------- | ----------------------------------------------------- |
| `REQUIRED` (기본값) | 기존 트랜잭션이 있으면 참여, 없으면 새로 시작         |
| `REQUIRES_NEW`      | **기존 트랜잭션을 중단**하고 **새로운 트랜잭션 시작** |
| `NESTED`            | 기존 트랜잭션에 중첩되어 실행 (savepoint 사용)        |
| `SUPPORTS`          | 트랜잭션이 있으면 참여, 없으면 트랜잭션 없이 실행     |
| `NOT_SUPPORTED`     | 트랜잭션 없이 실행                                    |
| `MANDATORY`         | 반드시 트랜잭션 내에서 실행 (없으면 예외)             |
| `NEVER`             | 트랜잭션 있으면 예외 발생                             |

---

## 💡 `REQUIRES_NEW` 분석 및 활용

### 정의

- 현재 트랜잭션이 **중단(suspended)** 되고, 새로운 트랜잭션이 생성되어 **독립적으로 실행**됨
- 이전 트랜잭션의 rollback이나 commit 여부에 영향을 받지 않음

### 사용 시점

1. **로그, 알림 등 부수 작업**은 반드시 저장되어야 하는 경우
2. **실패해도 본 트랜잭션에 영향주지 말아야 하는 경우**
3. 비즈니스 로직 분리 시 **독립된 트랜잭션 단위**가 필요한 경우

### 실무 예시

```java
@Service
@RequiredArgsConstructor
public class SettlementService {

    private final LogService logService;

    @Transactional
    public void processSettlement() {
        // 메인 트랜잭션
        saveMainData();

        // 로그는 별도 트랜잭션으로 처리
        logService.writeLog("정산 처리 시작");
    }

    private void saveMainData() {
        // DB에 정산 정보 저장
    }
}

@Service
public class LogService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void writeLog(String message) {
        // 별도의 트랜잭션에서 로그 기록
        logRepository.save(new Log(message));
    }
}
```

### 동작 플로우

1. `processSettlement()` 실행 → 트랜잭션 A 시작
2. `saveMainData()` → A 안에서 실행
3. `writeLog()` 호출 → **트랜잭션 A 중단**, **트랜잭션 B 시작**
4. 로그 저장 후 → B 커밋
5. 다시 트랜잭션 A 복귀

### 실패 시 처리

- `saveMainData()`가 실패해도 `writeLog()`는 커밋됨
- 반대로 `writeLog()`가 실패해도 메인 트랜잭션은 롤백되지 않음 (단, 예외 전파 여부 설정 주의)

```java
try {
    logService.writeLog("처리 로그");
} catch (Exception e) {
    // 로그 실패는 무시, 메인 트랜잭션엔 영향 없음
}
```

---

## ⚠️ 주의사항

1. `REQUIRES_NEW`는 **별도 커넥션**을 사용하므로, 커넥션 풀에 부담이 될 수 있음
2. **내부 메서드 호출**로는 AOP 프록시가 적용되지 않음 (self-invocation 문제)
3. 예외 전파에 따라 상위 트랜잭션도 롤백될 수 있음 → **`try-catch`로 감싸야 안전**

---

## 🛠 관련 옵션들

| 옵션            | 설명                                        |
| --------------- | ------------------------------------------- |
| `rollbackFor`   | 어떤 예외 발생 시 롤백할지 지정             |
| `noRollbackFor` | 특정 예외는 롤백하지 않도록 지정            |
| `readOnly`      | 읽기 전용 트랜잭션 (쓰기 시 예외 발생 가능) |
| `timeout`       | 트랜잭션 제한 시간 설정                     |

---

## ✅ 결론

- `REQUIRES_NEW`는 **트랜잭션 분리**가 필요할 때 강력한 도구
- 하지만 남용 시 커넥션 낭비 및 성능 이슈 유발 가능
- 로깅, 이력 저장, 독립 실행 단위 등에 유용하게 활용 가능

---

> 참고 자료
>
> - Spring Framework Docs: [@Transactional](https://docs.spring.io/spring-framework/reference/data-access/transaction/annotation-based.html)
> - 김영한 - 스프링 DB 1편 강의
