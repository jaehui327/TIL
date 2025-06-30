# 🧩 JPA 프로시저 호출

> Spring Boot 환경에서 JPA + Oracle 기반 프로시저 호출 방식 요약

---

## ✅ 1. 호출 방식 요약

| 방식            | 설명                        | 특징                      |
| --------------- | --------------------------- | ------------------------- |
| `@Procedure`    | Repository에서 선언적 호출  | Entity에 정의 필요        |
| `EntityManager` | 가장 일반적이고 유연한 방식 | IN/OUT 파라미터 처리 가능 |
| `JdbcTemplate`  | SQL 문장 직접 실행          | 빠르고 간단함 (JPA 아님)  |

---

## ✅ 2. EntityManager 사용 (권장 방식)

### 📌 기본 호출 (파라미터 없음)

```java
@Service
@RequiredArgsConstructor
public class MyService {
    private final EntityManager em;

    @Transactional
    public void callProcedure() {
        StoredProcedureQuery query = em.createStoredProcedureQuery("MY_PROCEDURE");
        query.execute();
    }
}
```

---

### 📌 IN 파라미터 있는 경우

```java
StoredProcedureQuery query = em.createStoredProcedureQuery("MY_PROCEDURE");

query.registerStoredProcedureParameter("PARAM1", String.class, ParameterMode.IN);
query.setParameter("PARAM1", "hello");

query.execute();
```

---

### 📌 OUT 파라미터 있는 경우

```java
query.registerStoredProcedureParameter("OUT_VAL", String.class, ParameterMode.OUT);
query.execute();

String result = (String) query.getOutputParameterValue("OUT_VAL");
```

---

### 📌 IN/OUT 혼합 예시

```java
StoredProcedureQuery query = em.createStoredProcedureQuery("MY_PROC");

query.registerStoredProcedureParameter("IN_VAL", Integer.class, ParameterMode.IN);
query.registerStoredProcedureParameter("OUT_MSG", String.class, ParameterMode.OUT);

query.setParameter("IN_VAL", 123);
query.execute();

String msg = (String) query.getOutputParameterValue("OUT_MSG");
```

---

## ✅ 3. @Procedure 방식 (선언적 호출)

### 📌 Entity 클래스에 정의

```java
@Entity
@NamedStoredProcedureQuery(
    name = "MyEntity.myProc",
    procedureName = "MY_PROCEDURE"
)
public class MyEntity {
    @Id
    private Long id;
}
```

### 📌 Repository 인터페이스에 정의

```java
public interface MyRepository extends JpaRepository<MyEntity, Long> {
    @Procedure(name = "MyEntity.myProc")
    void callMyProc();
}
```

> ⚠️ 복잡한 프로시저에는 비추천. 단순 호출만 적합

---

## 🛠 트러블슈팅

| 오류 메시지                        | 원인                                         |
| ---------------------------------- | -------------------------------------------- |
| `Parameter ... was not registered` | `registerStoredProcedureParameter()` 누락    |
| `No parameter binding found`       | 파라미터 이름 또는 순서 오류                 |
| 결과가 안 나옴                     | `@Transactional` 누락 가능                   |
| `NullPointerException`             | OUT 파라미터 반환값을 받기 전 예외 처리 없음 |

---

## ✅ 권장 구조

- Controller → Service → `EntityManager` 로 호출
- 트랜잭션이 필요한 경우 `@Transactional` 붙이기
- 반환값은 `query.getOutputParameterValue(...)` 로 명시적 처리

---

## 📎 참고

- [Spring Data JPA Reference - Stored Procedures](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.stored-procedures)
