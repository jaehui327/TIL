# JPA

## 🧾 1. JPA란?

- Java 애플리케이션에서 **데이터베이스와 객체(Entity)를 매핑**해주는 ORM(Object-Relational Mapping) 기술
- **자바 표준 ORM API**이며, 구현체로는 Hibernate, EclipseLink 등이 있음

---

## 🏗️ 2. 핵심 개념

### ✅ Entity

- DB 테이블과 매핑되는 자바 클래스
- `@Entity` 어노테이션 필수
- 기본키는 `@Id`로 명시

### ✅ Repository

- Entity에 대한 DB 접근 로직을 정의하는 인터페이스
- Spring Data JPA는 `JpaRepository<T, ID>` 등을 통해 구현 없이 자동 제공

### ✅ EntityManager

- JPA의 핵심 인터페이스
- 트랜잭션 내에서 엔티티의 **조회, 저장, 수정, 삭제** 등을 수행

---

## ⚙️ 3. 동작 방식 요약

1. 애플리케이션 → Repository 호출
2. Repository → JPQL 또는 메서드 이름 기반 쿼리 실행
3. EntityManager가 SQL로 변환 후 DB에 실행
4. 결과를 Entity 또는 DTO로 반환

---

## 💬 4. 주요 어노테이션

| 어노테이션                 | 설명                         |
| -------------------------- | ---------------------------- |
| `@Entity`                  | 클래스가 JPA 엔티티임을 명시 |
| `@Id`                      | 엔티티의 기본 키 필드        |
| `@GeneratedValue`          | 기본 키 자동 생성 전략 지정  |
| `@Table(name = "...")`     | 매핑될 테이블 이름 지정      |
| `@Column(name = "...")`    | 컬럼 매핑 (옵션 지정 가능)   |
| `@OneToMany`, `@ManyToOne` | 연관관계 매핑 지정           |

---

## 🧠 5. JPQL vs Native Query

| 항목        | JPQL            | Native Query                |
| ----------- | --------------- | --------------------------- |
| 작성 방식   | 엔티티 기준     | SQL 기준                    |
| 타입 안전성 | 높음            | 낮음                        |
| 동작 속도   | 추상화되어 있음 | 빠를 수 있음                |
| 사용 시점   | 대부분의 쿼리   | 복잡한 SQL, DB 함수 필요 시 |

---

## 🧩 6. 주요 기능

- CRUD 메서드 자동 생성 (`findById`, `save`, `delete`)
- JPQL / NativeQuery 쿼리 지원
- DTO 매핑 지원 (`SELECT new DTO(...)`)
- 페이징 & 정렬 지원 (`Pageable`, `Sort`)
- 연관관계 자동 관리 (`cascade`, `fetch`, `orphanRemoval`)

---

## 🛠️ 7. 장단점

### ✅ 장점

- 객체 중심의 개발 가능
- 코드 재사용성과 유지보수 용이
- 트랜잭션, 더티 체킹 등 자동 처리

### ❌ 단점

- 학습 비용 존재
- 복잡한 쿼리는 SQL보다 어렵게 느껴질 수 있음
- 퍼포먼스 튜닝 필요할 수 있음 (N+1, batch insert 등)

---

## 🔚 정리

JPA는 객체지향적인 방식으로 DB를 다룰 수 있게 해주는 자바 ORM 표준이다.  
Spring Data JPA를 함께 사용하면 생산성을 극대화할 수 있다.

---

# Spring Data JPA: @Query로 DTO 매핑하는 방법 정리

## 📌 1. 기본 개념

Spring Data JPA의 `@Query`에서 DTO로 직접 매핑하려면 **JPQL의 생성자 표현식**을 사용해야 한다.  
JPQL은 SQL과 달리 엔티티 객체 또는 DTO 객체를 대상으로 쿼리한다.

```java
@Query("SELECT new 패키지경로.DTO클래스(필드1, 필드2, ...) FROM Entity e")
List<DTO클래스> 메서드이름(...);
```

---

## ✅ 2. DTO 매핑 필수 조건

| 조건       | 설명                                                           |
| ---------- | -------------------------------------------------------------- |
| 생성자     | `public` 생성자 필요, 필드 개수·순서·타입이 JPQL과 일치해야 함 |
| 패키지명   | `new` 키워드 뒤에 **전체 패키지 경로 포함**                    |
| 반환타입   | `List<DTO>` 또는 `Page<DTO>` 가능                              |
| countQuery | `Page<DTO>`일 경우 countQuery는 엔티티 기준으로 작성           |

---

## 🛠 3. 실전 예제 (DTO + Pageable)

### 📁 DTO 클래스: `PowerTrdStlAtCmprInfo`

```java
@Getter
@AllArgsConstructor
public class PowerTrdStlAtCmprInfo {
    private String trdsh;
    private String geid;
    private String mgptid;
    private InitEntrListForm initEntrListForm;
    private LastEntrListForm lastEntrListForm;
    private EndEntrListForm endEntrListForm;
}
```

> ※ `@Getter`, `@AllArgsConstructor`는 Lombok 사용. 직접 작성해도 OK.

---

### 📁 Repository 인터페이스

```java
@Query("SELECT new kpx.mms.stl.apl.result.service.dto.PowerTrdStlAtCmprInfo(" +
       "e.id.trdsh, e.id.geid, e.mgptid, " +
       "e.initEntrListForm, e.lastEntrListForm, e.endEntrListForm) " +
       "FROM PowerTrdStlAtCmp e " +
       "WHERE (:strTrdTerm IS NULL OR e.strTradingTerm = :strTrdTerm) " +
       "AND (:endTrdTerm IS NULL OR e.endTradingTerm = :endTrdTerm) " +
       "AND (:mgptid IS NULL OR e.mgptid = :mgptid) " +
       "AND (:geid IS NULL OR e.geid = :geid) " +
       "AND (:strid IS NULL OR e.strid = :strid) " +
       "ORDER BY e.id.trdseq DESC, e.id.trdsh ASC, e.id.geid ASC")
Page<PowerTrdStlAtCmprInfo> searchPowerTrdStlAtCmprInfo(
    @Param("strTrdTerm") String strTrdTerm,
    @Param("endTrdTerm") String endTrdTerm,
    @Param("mgptid") String mgptid,
    @Param("geid") String geid,
    @Param("strid") String strid,
    Pageable pageable
);
```

---

## ❗ 발생했던 문제 및 해결

| 문제                            | 원인                                   | 해결                               |
| ------------------------------- | -------------------------------------- | ---------------------------------- |
| `No converter found`            | DTO로 반환하는데 `List<엔티티>`로 받음 | 반환타입을 `List<DTO>`로 변경      |
| `No serializer found for class` | Jackson이 DTO 필드를 읽지 못함         | DTO에 `getter` 또는 `@Getter` 추가 |

---

## 🔚 정리

- `@Query` + DTO 매핑은 JPQL에서만 가능
- DTO는 **public 생성자와 getter** 필수
- `Page<DTO>` 사용할 경우 `countQuery` 작성 권장
- Jackson 직렬화 오류는 대부분 getter 누락 문제

---
