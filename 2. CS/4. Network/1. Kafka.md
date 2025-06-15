# Apache Kafka

## 📌 1. Kafka 기본 이론

### ✅ Kafka란?

Apache Kafka는 **대용량 실시간 데이터 스트리밍 플랫폼**으로, 대규모 이벤트 데이터를 **발행-구독(Pub/Sub)** 구조로 처리할 수 있게 해주는 메시징 시스템입니다.

- 분산 시스템으로 확장성과 내결함성이 우수함
- 마이크로서비스 간 비동기 통신, 실시간 로그 수집, 이벤트 처리 등에서 사용

---

### 🧱 Kafka 구성 요소

| 구성 요소          | 설명                                        |
| ------------------ | ------------------------------------------- |
| **Producer**       | 데이터를 Kafka에 전송하는 주체              |
| **Consumer**       | Kafka로부터 데이터를 읽는 주체              |
| **Topic**          | 메시지를 구분하는 논리 단위                 |
| **Partition**      | Topic 내 데이터를 병렬로 처리하기 위한 단위 |
| **Broker**         | Kafka 서버, 메시지를 저장하고 분산          |
| **Consumer Group** | 메시지를 분산 소비하기 위한 Consumer 집합   |

---

### 🔁 메시지 처리 흐름

```plaintext
Producer → [Kafka Topic (n개의 Partition)] → Consumer Group
```

- 같은 Partition 내에서는 메시지 순서 보장
- Consumer Group 내에서 Partition은 Consumer에게 분산 할당됨
- 각 Consumer는 오프셋(offset)을 관리하여 메시지 소비 위치를 기억

---

## 📦 2. Avro 스키마 관리

### ✅ Avro란?

**Apache Avro**는 Kafka 메시지 직렬화를 위한 **스키마 기반 이진 포맷**입니다. JSON보다 작고 빠르며, **Schema Registry**와 함께 사용하여 **버전 관리와 데이터 호환성**을 보장합니다.

---

### 🧩 Avro 스키마 예시 (`.avsc`)

```json
{
  "type": "record",
  "name": "User",
  "fields": [
    { "name": "name", "type": "string" },
    { "name": "age", "type": "int" }
  ]
}
```

---

### 🛠 Spring Boot + Kafka + Avro 설정 흐름

1. **의존성 추가**

```groovy
implementation 'io.confluent:kafka-avro-serializer:7.0.1'
```

2. **Schema Registry 설정**

```yaml
spring.kafka.properties.schema.registry.url=http://localhost:8081
spring.kafka.producer.value-serializer=io.confluent.kafka.serializers.KafkaAvroSerializer
spring.kafka.consumer.value-deserializer=io.confluent.kafka.serializers.KafkaAvroDeserializer
```

3. **스키마 클래스 생성**
   - Maven/Gradle 플러그인으로 `.avsc` → Java 클래스로 변환

---

### 📌 장점 요약

| 항목              | 설명                                          |
| ----------------- | --------------------------------------------- |
| 이진 포맷         | JSON보다 빠르고 용량 작음                     |
| 스키마 기반       | 데이터 구조 고정 → 호환성 보장                |
| 스키마 레지스트리 | 버전 관리 가능, 생산자와 소비자의 일관성 확보 |

---

## 🔄 3. Kafka Streams

### ✅ Kafka Streams란?

Kafka 내부에서 제공하는 **스트림 처리 라이브러리**로, Kafka 토픽 간의 데이터를 실시간으로 변환, 집계, 조인하는 로직을 구현할 수 있습니다.

---

### 🧩 구성 요소

| 개념           | 설명                               |
| -------------- | ---------------------------------- |
| `KStream`      | 일반적인 이벤트 흐름 (레코드 기반) |
| `KTable`       | 상태 기반 스트림 (업데이트 중심)   |
| `GlobalKTable` | 모든 인스턴스에 복제되는 테이블    |

---

### 🛠 Streams 예시 (Spring Kafka Streams)

```java
@Bean
public KStream<String, String> process(StreamsBuilder builder) {
    KStream<String, String> stream = builder.stream("input-topic");
    stream.mapValues(val -> val.toUpperCase())
          .to("output-topic");
    return stream;
}
```

---

### ✅ Kafka Streams 특징

| 항목              | 설명                                        |
| ----------------- | ------------------------------------------- |
| 완전한 라이브러리 | 별도 클러스터 없이 애플리케이션 내에서 동작 |
| 상태 저장 가능    | RocksDB + Kafka changelog topic 활용        |
| Exactly-once 처리 | 트랜잭션 기반 메시지 처리 보장 가능         |

---

## 🧠 Kafka 실무 요약

| 기술    | 설명                                 |
| ------- | ------------------------------------ |
| Kafka   | 대용량 이벤트 기반 메시징 플랫폼     |
| Avro    | 이진 직렬화 포맷 + 스키마 검증       |
| Streams | Kafka 스트림 변환 및 실시간 집계 API |

---

## 🔚 확장 팁

- ✅ Kafka + Avro + Streams 조합은 **대규모 실시간 분석/전송 처리**에 매우 적합
- ✅ `KafkaTemplate`, `@KafkaListener`는 Spring Kafka에서 기본 제공
- ✅ Schema Registry, Avro, Topic 설계는 **프로젝트 초기에 표준화**하는 것이 중요
