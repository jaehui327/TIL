# 🧱 Domain-Driven Design (DDD)

DDD는 복잡한 도메인 문제를 비즈니스 중심의 객체 모델링을 통해 해결하는 전략입니다. 실전에서는 MSA, 대규모 시스템에서 복잡한 비즈니스 로직을 명확히 하기 위해 사용됩니다.

---

## 📌 1. DDD 핵심 개념

### ✅ Entity
- ID를 기준으로 동일성 식별
- 상태 변경 가능
```java
public class Member {
    private Long id;
    private String name;
    private Address address; // VO
}
```

### ✅ Value Object
- 값 자체로 동일성 판단
- Immutable (불변)
```java
public class Address {
    private final String city;
    private final String street;

    // equals(), hashCode() override 필수
}
```

### ✅ Aggregate & Root
- 도메인의 일관성을 유지하는 트랜잭션 단위
- 외부는 Root만 접근
```java
public class Order { // Aggregate Root
    private Long id;
    private List<OrderItem> items;

    public void addItem(OrderItem item) {
        this.items.add(item);
    }
}
```

> 💡 `Order`만이 `OrderItem`을 관리하며, 외부에서 직접 접근 금지

---

## 📌 2. 계층형 아키텍처 (Layered Architecture)

```
┌────────────────────┐
│     Presentation   │ ← 사용자와 상호작용 (Controller)
├────────────────────┤
│    Application     │ ← 유스케이스 흐름 제어, 트랜잭션
├────────────────────┤
│      Domain        │ ← 핵심 로직 (Entity, VO, Service)
├────────────────────┤
│   Infrastructure   │ ← DB, 외부 API, Kafka 등
└────────────────────┘
```

- `Application`은 도메인을 조합하여 **비즈니스 유스케이스 처리**
- `Infrastructure`는 기술 세부사항 처리

---

## 📌 3. Repository Interface 예시

```java
public interface OrderRepository {
    Order findById(Long id);
    void save(Order order);
}
```

```java
@Repository
public class JpaOrderRepository implements OrderRepository {
    @PersistenceContext
    private EntityManager em;

    public Order findById(Long id) {
        return em.find(Order.class, id);
    }

    public void save(Order order) {
        em.persist(order);
    }
}
```

---

## 📌 4. Domain Service 예시

```java
public class DiscountService {
    public Money calculateDiscount(Order order) {
        if (order.isVipCustomer()) {
            return new Money(1000);
        }
        return new Money(0);
    }
}
```

---

## 📌 5. Bounded Context & Context Map

### 예시: 이커머스 도메인

```
+------------------+      +------------------+
|  Order Context   | ---> |  Payment Context |
|------------------|      |------------------|
| Order, Item      |      | Payment, Billing |
+------------------+      +------------------+

공통 모델 공유 불가 → ACL 또는 API 통합 필요
```

- Bounded Context는 독립적인 **도메인 모델, DB, 서비스**를 가질 수 있음

---

## 📌 6. Ubiquitous Language 예시

- 모든 팀원이 아래와 같은 단어를 동일 의미로 사용
  - `Customer` = 주문자
  - `Delivery` = 출고 처리
  - `Payment Completed` = 결제 승인 완료

> 보편 언어가 코드, 문서, 회의에 **동일하게 쓰여야 일관성 유지**

---

## 📌 7. 도입 시 유의사항

| 항목 | 설명 |
|------|------|
| 📘 도입 전제 | 복잡한 비즈니스 도메인에서 유리 |
| 👥 협업 필수 | 도메인 전문가와 긴밀한 협력 |
| 🔍 설계 비용 | CRUD만 있는 시스템엔 과한 설계가 될 수 있음 |

---

## 📌 8. 실전 도입 팁

- `도메인 설계 → 보편 언어 정의 → 아키텍처 분리 → 리팩토링`
- 단일 프로젝트 → 점진적 DDD 도입 가능
- 시작은 `도메인 계층 분리`부터

---

## 📚 참고자료

- Eric Evans - *Domain-Driven Design*
- Vaughn Vernon - *Implementing DDD*
- [https://dddcommunity.org](https://dddcommunity.org)

