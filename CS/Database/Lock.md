# DB Lock

## 1. Lock이란?

- 여러 사용자가 동시에 데이터에 접근할 때 **데이터의 무결성과 일관성**을 보장하기 위한 **동기화 메커니즘**
- 충돌이나 이상 현상(Dirty Read, Lost Update 등)을 방지하기 위해 사용

---

## 2. Lock의 종류

### 🔒 2.1 DML Lock (Data Manipulation Lock)

| 종류                 | 설명                                                                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Row-level Lock**   | - 특정 **row 단위**로 잠금<br>- `SELECT ... FOR UPDATE`, `INSERT`, `UPDATE`, `DELETE` 시 발생                                         |
| **Table-level Lock** | - **테이블 전체**에 대한 잠금<br>- DML 작업 수행 시 함께 발생 가능<br>- 종류: ROW SHARE, ROW EXCLUSIVE, SHARE, SHARE ROW EXCLUSIVE 등 |

### 🔐 2.2 DDL Lock (Data Definition Lock)

| 종류                     | 설명                                                   |
| ------------------------ | ------------------------------------------------------ |
| **Breakable Parse Lock** | - 객체(테이블 등)를 사용하는 SQL 파싱 중 변경을 방지   |
| **Exclusive Lock**       | - `ALTER`, `DROP` 같은 명령어 수행 시 객체 변경을 막음 |

### 🚫 2.3 Internal Lock & Latch

| 종류      | 설명                                                                                 |
| --------- | ------------------------------------------------------------------------------------ |
| **Latch** | - 메모리 구조 보호를 위한 낮은 수준의 잠금<br>- 매우 빠르지만 충돌 시 성능 저하 가능 |
| **Mutex** | - Latch보다 더 정밀하고 가벼운 메커니즘                                              |

---

## 3. Lock 모드 (Table Lock 기준)

| Lock Mode                     | 설명                                | 예시                             |
| ----------------------------- | ----------------------------------- | -------------------------------- |
| **ROW SHARE (RS)**            | SELECT FOR UPDATE용, 가장 낮은 수준 | SELECT ... FOR UPDATE            |
| **ROW EXCLUSIVE (RX)**        | DML 작업 시 기본 설정               | INSERT, UPDATE, DELETE           |
| **SHARE (S)**                 | 읽기 공유는 가능하나 쓰기 불가      | LOCK TABLE ... IN SHARE MODE     |
| **SHARE ROW EXCLUSIVE (SRX)** | SH + RX, 드물게 사용됨              | 복잡한 병행성 제어               |
| **EXCLUSIVE (X)**             | 읽기/쓰기 모두 차단                 | LOCK TABLE ... IN EXCLUSIVE MODE |

---

## 4. Lock 관련 뷰

| 뷰 이름          | 설명                       |
| ---------------- | -------------------------- |
| `V$LOCK`         | 현재 걸려 있는 lock 정보   |
| `DBA_BLOCKERS`   | 다른 세션을 막고 있는 세션 |
| `DBA_WAITERS`    | 대기 중인 세션             |
| `V$SESSION`      | 세션 상태 확인용           |
| `V$SESSION_WAIT` | 세션의 대기 이벤트 정보    |

---

## 5. Lock 관련 이슈와 해결

### ⚠️ 5.1 Deadlock

- **서로 상대방의 리소스를 기다리며 무한 대기**
- Oracle은 Deadlock 발생 시 자동 감지하여 한 트랜잭션을 강제로 종료
- 해결: 트랜잭션 처리 순서 일관성 유지, Lock 점유 시간 최소화

### ⚠️ 5.2 Lock Wait

- 다른 세션의 트랜잭션이 commit되지 않아 대기 상태 지속
- 해결: 트랜잭션 분리, 쿼리 구조 개선, 필요시 `NOWAIT` 옵션 사용

```sql
SELECT * FROM employees FOR UPDATE NOWAIT;
```

---

## 6. 실무에서의 유의사항

- **트랜잭션 범위는 작게 유지**
- **오래 걸리는 작업 전에 SELECT FOR UPDATE 사용 주의**
- **자동 커밋 기능 OFF**
- Lock 이슈가 자주 발생하는 경우, **쿼리 튜닝 및 설계 변경 고려**

### Q1. Row-level Lock VS Table-level Lock

> Row-level은 특정 레코드에만 Lock을 걸어 병행처리에 유리하지만, Table-level은 전체 테이블에 Lock을 걸어 다른 작업과의 충돌 가능성을 줄인다.

### Q2. Deadlock이란?

> 서로가 상대의 자원을 기다리며 무한 대기하는 상태. 트랜잭션 순서 일관성 유지 및 Lock 최소화로 방지 가능.

### Q3. SELECT FOR UPDATE란?

> 데이터를 읽고 나중에 변경할 계획이 있을 때 다른 트랜잭션의 변경을 방지하기 위해 사용
