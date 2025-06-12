# Reflection

## 📚 Java 리플렉션(Reflection)이란?

### 🔹 정의

Java 리플렉션(Reflection)은 **실행 중(runtime)에 클래스, 인터페이스, 메서드, 필드 등에 대한 정보를 동적으로 조회하고 조작할 수 있는 기능**입니다.  
`java.lang.reflect` 패키지를 통해 제공됩니다.

### 🔹 주요 목적

- 클래스 이름만 알고 있을 때 동적으로 객체 생성
- 필드, 메서드, 생성자에 접근하여 값 읽기/쓰기
- 어노테이션 등 메타데이터 기반 프로그래밍
- 프레임워크(예: Spring, Jackson)에서 **동적 바인딩**, **객체 자동 주입**, **직렬화** 등에 활용

---

## 🧠 리플렉션의 핵심 클래스

| 클래스        | 역할                               |
| ------------- | ---------------------------------- |
| `Class<T>`    | 클래스 정보를 나타내는 핵심 클래스 |
| `Field`       | 클래스의 필드 정보 접근            |
| `Method`      | 메서드 정보 접근 및 실행           |
| `Constructor` | 생성자 접근 및 호출                |

---

## 🔍 예시 코드

### 클래스 정보 조회

```java
Class<?> clazz = Class.forName("com.example.MyClass");
System.out.println(clazz.getName());
```

### 필드 접근

```java
Field field = clazz.getDeclaredField("amount");
field.setAccessible(true);
Object value = field.get(obj);  // obj의 amount 필드 값을 읽음
```

### 메서드 실행

```java
Method method = clazz.getMethod("getAmount");
Object returnValue = method.invoke(obj);  // obj.getAmount() 호출
```

---

## ⚠️ 리플렉션의 단점

| 항목                    | 설명                                       |
| ----------------------- | ------------------------------------------ |
| 성능 저하               | 일반적인 호출보다 느림 (JVM 최적화 어려움) |
| 컴파일 타임 안정성 부족 | 문법 오류가 런타임 오류로 바뀔 수 있음     |
| 접근 제한 우회          | `setAccessible(true)`로 캡슐화 원칙 위배   |

---

## 🧾 리플렉션 활용 사례

- Spring Framework: 의존성 주입(DI), 프록시 기반 AOP
- Jackson/Gson: 객체 → JSON 직렬화/역직렬화
- JUnit/TestNG: 테스트 메서드 자동 실행
- Hibernate/JPA: 필드 및 관계 설정 기반 엔티티 매핑

---

## 활용 사례 : 넥사크로 연동 시 BigDecimal 소수점 손실 문제 분석

## 🧩 문제 상황 요약

넥사크로에서 백엔드로부터 받은 소수점 데이터(BigDecimal)가 다음과 같은 증상을 보임:

- `0.23423` → `0`으로 표시됨
- `123.45` → `123`으로 표시됨
- 넥사크로 `Dataset`에는 정상적으로 `BIGDECIMAL` 타입으로 설정되어 있음

> 📌 실제 문제는 **넥사크로가 아닌 BFF(Backend for Frontend) 단에서 발생**

---

## 🔍 원인 분석

### 1. 리플렉션 기반 데이터 변환

BFF에서 객체 → Map 변환 시 Java 리플렉션 사용:

```java
Map<String, Object> result = new HashMap<>();
for (Field field : obj.getClass().getDeclaredFields()) {
    field.setAccessible(true);
    result.put(field.getName(), field.get(obj));
}
```

이 때, `BigDecimal`도 `Object`로 들어가므로 내부 직렬화 시 타입 손실 발생 가능.

### 2. 직렬화 시 `BigDecimal` → `Long` 으로 오인

- `ObjectMapper` 또는 넥사크로 JSON 파서가 `BigDecimal`을 정수로 간주
- 특히 `new BigDecimal("0.23423")`도 내부에서 `longValue()` 등을 호출하면 `0`이 됨

### 3. 넥사크로 `Dataset`은 받은 값을 있는 그대로 렌더링

- 소수점 이하가 잘린 값이 들어오면 아무리 `BIGDECIMAL`, `displayformat`을 설정해도 무의미

---

## ✅ 해결 방법

### 방법 1. `BigDecimal`을 명시적으로 문자열로 직렬화

```java
if (value instanceof BigDecimal) {
    map.put(field.getName(), ((BigDecimal) value).toPlainString());
} else {
    map.put(field.getName(), value);
}
```

- `toPlainString()`을 쓰면 `"0.23423"`처럼 정확한 표현 유지
- 넥사크로에서 문자열로 받아도 `BIGDECIMAL` 타입이면 정상 렌더링 가능

### 방법 2. DTO를 사용하여 타입 보존

```java
public class AmountDto {
    public BigDecimal amount;
}
```

- DTO 클래스 기반으로 JSON을 생성하면 리플렉션으로 인한 타입 손실 최소화

### 방법 3. Jackson 설정을 통해 정밀도 보존

```java
mapper.configure(JsonGenerator.Feature.WRITE_BIGDECIMAL_AS_PLAIN, true);
```

- Spring Boot 환경에서는 `Jackson2ObjectMapperBuilderCustomizer`로 전역 설정 가능
