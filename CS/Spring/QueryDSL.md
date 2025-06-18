# QueryDSL 이론 분석 정리

## 1. QueryDSL이란?

> ✅ QueryDSL은 자바 코드로 SQL과 유사한 **타입 안전한 쿼리**를 작성할 수 있게 해주는 **도메인 특화 언어(DSL)** 라이브러리이다.

- JPA Criteria API의 복잡함과 비직관성을 개선하기 위해 만들어짐
- 컴파일 시점에 쿼리 오류를 잡을 수 있음 (타입 안전성)

---

## 2. QueryDSL의 기본 개념

### 📌 Q타입 (QClass)
- 엔티티마다 생성되는 쿼리 전용 클래스 (예: `QMember`, `QOrder`)
- 예: `QMember.member.age.gt(30)` → SQL의 `WHERE age > 30`

### 📌 QueryFactory
- 쿼리를 생성하고 실행하는 핵심 객체
```java
JPAQueryFactory queryFactory = new JPAQueryFactory(em);
```

### 📌 주요 메서드
| 메서드 | 설명 |
|--------|------|
| `select()` | 조회할 컬럼 선택 |
| `from()` | 엔티티 지정 |
| `join()`, `leftJoin()` | 조인 처리 |
| `where()` | 조건절 |
| `orderBy()`, `groupBy()` | 정렬/그룹 |
| `fetch()`, `fetchOne()` | 결과 실행 |

---

## 3. QueryDSL 기본 예제

```java
QMember member = QMember.member;

List<Member> result = queryFactory
    .selectFrom(member)
    .where(member.age.gt(20))
    .orderBy(member.name.asc())
    .fetch();
```

---

## 4. DTO 매핑 방법

### ✅ 필드 접근 방식
```java
queryFactory
    .select(Projections.fields(MemberDto.class,
        member.name.as("name"),
        member.age
    ))
    .from(member)
    .fetch();
```

### ✅ 생성자 방식 (가장 안전)
```java
queryFactory
    .select(Projections.constructor(MemberDto.class,
        member.name,
        member.age
    ))
    .from(member)
    .fetch();
```

---

## 5. QueryDSL의 장점

| 장점 | 설명 |
|------|------|
| ✅ 타입 안전 | 컴파일 시점에 필드명/타입 오류 탐지 가능 |
| ✅ IDE 자동완성 | 쿼리 작성 속도 향상 |
| ✅ 복잡한 동적 쿼리 | 조건을 조립식으로 붙이는 게 쉬움 |
| ✅ 리팩터링에 강함 | 문자열이 아닌 코드 기반 쿼리라 변경에 강함 |

---

## 6. QueryDSL의 단점

| 단점 | 설명 |
|------|------|
| ❌ Q클래스 생성 필요 | Gradle or Maven으로 Q파일 생성 필요 |
| ❌ 복잡한 쿼리 구조 | join + group by 등은 직관성 낮음 |
| ❌ Native SQL 미지원 | JPA 기반이므로 DB 특화 기능은 제한 |

---

## 7. 실무 활용 팁

### ✅ 동적 where 조건 처리
```java
BooleanBuilder builder = new BooleanBuilder();
if (name != null) builder.and(member.name.eq(name));
if (age != null) builder.and(member.age.gt(age));

queryFactory
    .selectFrom(member)
    .where(builder)
    .fetch();
```

### ✅ 리스트 조건 동적 처리
```java
.where(member.id.in(ids)) // ids가 빈 리스트면 자동으로 false 처리됨
```

### ✅ 페이징 처리
```java
queryFactory
    .selectFrom(member)
    .offset(pageable.getOffset())
    .limit(pageable.getPageSize())
    .fetch();
```

---

## 8. Q파일 생성 방법 (Gradle)

```groovy
// build.gradle.kts
dependencies {
    implementation("com.querydsl:querydsl-jpa")
    annotationProcessor("com.querydsl:querydsl-apt")
}

tasks.withType<JavaCompile> {
    options.annotationProcessorPath = configurations.annotationProcessor.get()
}
```

> 생성 경로: `build/generated/querydsl`

---

## 9. QueryDSL vs JPQL

| 항목 | QueryDSL | JPQL |
|------|----------|------|
| 타입 안전성 | 높음 (컴파일 시점 체크) | 낮음 (런타임 오류) |
| 가독성 | 좋음 (IDE 지원) | 나쁨 (문자열 기반) |
| 동적 쿼리 | 용이 | 복잡 |
| 학습 곡선 | 다소 있음 | 낮음 |

---

## 10. 결론

QueryDSL은 JPA를 사용하는 Java 개발자에게 **타입 안전 + 동적 쿼리 + IDE 친화성**을 제공하는 매우 강력한 도구이다. 복잡한 도메인 조건 조립이나 정교한 DTO projection이 필요할 때 특히 유용하다. 다만 복잡한 조인, 집계 연산, native SQL이 필요한 경우는 한계가 있으므로 상황에 따라 적절히 병행해야 한다.

---

## ✅ 키워드 요약
`QueryDSL`, `JPAQueryFactory`, `BooleanBuilder`, `Q타입`, `Projections`, `fetch()`, `동적 조건`, `DTO 매핑`