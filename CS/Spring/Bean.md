# 🌱 Spring Bean

## 📌 1. Spring Bean이란?

Spring에서 **Bean**은 IoC 컨테이너가 관리하는 객체입니다.  
보통 Service, Repository, Controller, Config 등의 클래스들이 이에 해당하며, DI(의존성 주입) 대상이 됩니다.

---

## 📚 2. Spring Legacy vs Spring Boot: Bean 등록 방식 비교

| 항목           | Spring Legacy                   | Spring Boot                                          |
| -------------- | ------------------------------- | ---------------------------------------------------- |
| Bean 등록 방식 | 수동 (XML/JavaConfig)           | 자동 (`@ComponentScan` + `@EnableAutoConfiguration`) |
| 설정 위치      | XML 파일 또는 JavaConfig 클래스 | `@SpringBootApplication` 내부                        |
| 철학           | 명시적 구성                     | 관례 기반 자동 구성 (Convention over Configuration)  |
| 장점           | 명확한 제어 가능                | 빠른 개발, 설정 간소화                               |
| 단점           | 반복 설정, 비효율               | 자동 구성 파악 어려움 가능                           |

---

## 🧩 3. Spring Legacy 예제

### XML 기반

```xml
<bean id="myService" class="com.example.MyService"/>
```

### JavaConfig 기반

```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

---

## ⚡ 4. Spring Boot 예제 (자동 등록)

```java
@Service
public class GreetingService {
    public String greet() {
        return "Hello";
    }
}
```

```java
@RestController
public class GreetingController {
    private final GreetingService greetingService;
    public GreetingController(GreetingService greetingService) {
        this.greetingService = greetingService;
    }

    @GetMapping("/greet")
    public String greet() {
        return greetingService.greet();
    }
}
```

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

> ### 📌 @SpringBootApplication 내부 구조
>
> ```java
> @Target(TYPE)
> @Retention(RUNTIME)
> @Documented
> @Inherited
> @SpringBootConfiguration
> @EnableAutoConfiguration
> @ComponentScan
> public @interface SpringBootApplication {
> }
> ```
>
> - `@SpringBootConfiguration`: `@Configuration`과 같음
> - `@ComponentScan`: 자동으로 하위 패키지 스캔
> - `@EnableAutoConfiguration`: 자동 설정 클래스 활성화

---

## 🔍 5. `@ComponentScan`이란?

### ✅ 정의

- 특정 패키지 내의 클래스 중 `@Component`, `@Service`, `@Repository`, `@Controller` 등이 붙은 클래스를 **자동으로 스캔해 Bean으로 등록**하는 어노테이션

### ✅ 기본 동작 원리

- `@ComponentScan(basePackages = "com.example")`
  - 지정된 패키지 및 하위 패키지를 스캔
- 지정하지 않으면, 현재 클래스의 패키지부터 하위 패키지까지 스캔

### ✅ 함께 사용하는 어노테이션

| 어노테이션    | 역할         | 내부적으로 포함된 어노테이션 |
| ------------- | ------------ | ---------------------------- |
| `@Component`  | 기본 Bean    | 없음                         |
| `@Service`    | 서비스 계층  | `@Component`                 |
| `@Repository` | DAO 계층     | `@Component`                 |
| `@Controller` | MVC 컨트롤러 | `@Component`                 |

---

## 🧱 6. Spring Boot에서의 Bean 자동 등록 흐름

### ✅ @SpringBootApplication 구조

```java
@Target(TYPE)
@Retention(RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {}
```

### ✅ ComponentScan + EnableAutoConfiguration

- `@ComponentScan`: 현재 패키지 기준으로 Bean 탐색
- `@EnableAutoConfiguration`: classpath에 있는 설정 클래스들을 자동으로 구성 (ex: WebMvcAutoConfiguration, DataSourceAutoConfiguration 등)

### ✅ 내부 동작 원리

```plaintext
@EnableAutoConfiguration
      ↓
SpringFactoriesLoader.loadFactoryNames()
      ↓
META-INF/spring.factories에서 자동 설정 클래스 목록 로딩
      ↓
조건에 맞는 설정 클래스만 적용 (@ConditionalOnXxx)
```

---

## 🧠 7. `@ComponentScan` vs `@Bean` 비교

| 항목           | `@ComponentScan`               | `@Bean`                                    |
| -------------- | ------------------------------ | ------------------------------------------ |
| 사용 위치      | 클래스에 `@Component` 붙임     | `@Configuration` 클래스 내부 메서드에 붙임 |
| 자동 등록 여부 | O (스캔 기반)                  | X (명시적으로 등록)                        |
| 제어 수준      | 낮음 (관례 기반)               | 높음 (구체적 객체 생성 로직 가능)          |
| 예시 용도      | 일반적인 서비스, 레포지토리 등 | 외부 라이브러리 객체, 조건 기반 생성 등    |

---

## 🧾 8. 실무에서 주의할 점

### ✅ 애플리케이션 클래스 위치

- `@SpringBootApplication` 클래스는 항상 **최상위 패키지에 위치**
- 그렇지 않으면 ComponentScan이 하위 Bean을 인식 못할 수 있음

### ✅ 스캔 안 되는 상황

- `@ComponentScan`의 범위 밖에 클래스가 위치함
- `@Component` 계열 어노테이션이 누락됨
- Bean이 동적으로 생성되어 수동 등록 필요할 경우

---

## 🔧 9. 고급 설정: 필터링 예시

```java
@ComponentScan(
    basePackages = "com.example",
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = CustomComponent.class),
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ASSIGNABLE_TYPE,
        classes = LegacyService.class)
)
```

---

## 🧾 10. 정리 요약

- **Spring Legacy**: 명시적 설정 기반 (`@Bean`, XML)
- **Spring Boot**: 자동 설정 + 컴포넌트 스캔으로 개발 생산성 ↑
- **@ComponentScan**: 지정된 패키지 및 하위 패키지에서 자동 탐색
- **Auto Configuration**은 SpringFactoriesLoader를 통해 동적으로 설정 클래스 로딩

### Q1. Spring에서 Bean이란 무엇인가요?

**A1.** Spring에서 Bean은 **IoC 컨테이너에 의해 생성되고 관리되는 객체**를 말합니다. `@Component`, `@Service`, `@Repository`, `@Controller`, `@Bean` 등을 통해 등록되며, 주로 의존성 주입(DI) 대상이 됩니다.

### Q2. Spring Legacy와 Spring Boot의 Bean 등록 방식 차이는?

**A2.**

- **Spring Legacy**: XML 또는 Java Config로 명시적으로 Bean 등록
- **Spring Boot**: `@ComponentScan`과 `@EnableAutoConfiguration`에 의해 자동 등록  
  → **명시적 구성 vs 관례 기반 구성(Convention over Configuration)**이라는 철학 차이

### Q3. `@ComponentScan`이란 무엇인가요?

**A3.** `@ComponentScan`은 **지정된 패키지 내의 클래스 중 `@Component` 계열 어노테이션이 붙은 클래스를 자동으로 스캔**하여 Bean으로 등록하는 기능입니다.  
지정하지 않으면 현재 클래스의 패키지부터 하위 패키지를 스캔합니다.

### Q4. `@Component`와 `@Bean`의 차이는 무엇인가요?

**A4.**

| 항목      | `@Component` (자동 등록)     | `@Bean` (수동 등록)                |
| --------- | ---------------------------- | ---------------------------------- |
| 선언 위치 | 클래스에 어노테이션 부착     | `@Configuration` 내부 메서드       |
| 등록 방식 | 자동 등록 (`@ComponentScan`) | 명시적 등록                        |
| 제어 수준 | 낮음                         | 높음                               |
| 사용 예   | 일반적인 내부 클래스         | 외부 라이브러리, 조건 분기 필요 시 |

### Q5. `@SpringBootApplication` 어노테이션에 포함된 구성은?

**A5.** `@SpringBootApplication`은 다음 세 가지 어노테이션의 합성입니다:

- `@SpringBootConfiguration` (→ `@Configuration`)
- `@EnableAutoConfiguration` (→ 자동 설정)
- `@ComponentScan` (→ 컴포넌트 스캔 자동화)

### Q6. `@EnableAutoConfiguration`은 어떻게 동작하나요?

**A6.**

1. `@EnableAutoConfiguration` → `SpringFactoriesLoader.loadFactoryNames()` 호출
2. `META-INF/spring.factories`에서 자동 구성 클래스 목록 로딩
3. `@ConditionalOnXxx` 조건에 따라 필요한 설정 클래스만 실제 Bean으로 등록됨

---

## 📎 참고

- [Spring 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans)
- [Spring Boot Auto Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)
