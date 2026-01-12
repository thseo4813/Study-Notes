# 데이터베이스 성능의 양대 축: 인덱스(Index)와 트랜잭션(Transaction)

인덱스와 트랜잭션은 데이터베이스 성능과 신뢰성을 좌우하는 핵심 개념입니다. 이 두 가지를 제대로 이해하지 못하면 데이터베이스 튜닝은 불가능합니다.

## 1. 핵심 요약 (Executive Summary)

**인덱스(Index)**는 책의 목차처럼 데이터를 빠르게 찾기 위한 구조이고, **트랜잭션(Transaction)**은 데이터베이스 작업의 신뢰성과 일관성을 보장하는 실행 단위다. 이 둘은 데이터베이스 설계의 양대 축으로, 서로 다른 목적을 가지지만 함께 작동하여 데이터베이스의 성능과 안정성을 결정한다.

> **결론:**
> 1. **인덱스:** 쿼리 속도를 위해 필수지만, 삽입/수정 시 오버헤드가 발생하므로 전략적으로 사용해야 함
> 2. **트랜잭션:** ACID 속성을 통해 데이터 정합성을 보장하지만, 성능과 트레이드오프 관계임
> 3. **실무:** 인덱스와 트랜잭션 격리 수준을 상황에 맞게 튜닝하는 것이 핵심

---

## 2. 인덱스(Index): 데이터 검색의 핵심

인덱스는 데이터베이스의 성능을 좌우하는 가장 중요한 요소 중 하나다. "인덱스가 없으면 테이블 전체를 스캔하는 Full Table Scan이 발생한다."

### 2.1 인덱스의 구조와 종류

#### B-Tree 인덱스 (가장 일반적)
* **구조:** 균형 잡힌 트리 구조로, 루트-브랜치-리프 노드로 구성됨
* **특징:** 정렬된 상태 유지, 범위 검색에 강함
* **사용:** 기본 키, 유니크 제약조건, 외래 키에 자동 생성

```text
[B-Tree 구조 예시]
        [50]
       /    \
    [20]    [80]
   /  \    /  \
[10] [30] [60] [90]
```

#### Hash 인덱스
* **구조:** 해시 함수로 키를 해시값으로 변환하여 저장
* **특징:** 정확한 값 검색에 O(1), 범위 검색 불가
* **사용:** 메모리 기반 DB(Redis)나 정확한 키 검색이 필요한 경우

#### 클러스터형 vs 비클러스터형 인덱스
| 구분 | 클러스터형 인덱스 | 비클러스터형 인덱스 |
| --- | --- | --- |
| **데이터 저장** | 실제 데이터와 함께 저장 | 인덱스만 별도 저장 |
| **개수** | 테이블당 1개만 가능 | 여러 개 가능 |
| **장점** | 범위 검색에 매우 빠름 | 추가 공간 적게 사용 |
| **단점** | 업데이트 비용 높음 | 두 번의 조회 필요 |

### 2.2 인덱스 생성 전략

#### 카디널리티(Cardinality) 고려
* **높은 카디널리티:** 성별(남/여)보다는 이름, 이메일처럼 다양한 값이 있는 컬럼
* **낮은 카디널리티:** 성별, 국가 코드처럼 값의 종류가 적은 컬럼은 비효율적

#### 복합 인덱스(Composite Index)
```sql
-- 잘못된 예: 개별 컬럼에 인덱스 생성
CREATE INDEX idx_user_name ON users(name);
CREATE INDEX idx_user_age ON users(age);

-- 올바른 예: 자주 함께 사용하는 컬럼으로 복합 인덱스
CREATE INDEX idx_user_name_age ON users(name, age);

-- WHERE name = '김철수' AND age > 20 인 쿼리에 최적화됨
```

#### 인덱스 사용 Tip
* **선두 컬럼 우선:** 복합 인덱스에서 왼쪽 컬럼부터 사용해야 함
* **함수 사용 금지:** `WHERE UPPER(name) = 'KIM'`은 인덱스 사용 불가
* **LIKE 패턴:** `LIKE '김%'`은 사용 가능, `LIKE '%김'`은 불가

---

## 3. 트랜잭션(Transaction): 데이터 일관성의 수호자

트랜잭션은 "All or Nothing"의 원칙으로 데이터베이스의 신뢰성을 보장한다. 하나의 작업 단위로 취급되어 모두 성공하거나 모두 실패한다.

### 3.1 ACID 속성

트랜잭션이 가져야 할 4가지 핵심 속성이다.

| 속성 | 설명 | 예시 |
| --- | --- | --- |
| **Atomicity (원자성)** | 트랜잭션의 모든 연산이 완전히 실행되거나 전혀 실행되지 않음 | 계좌 이체: 출금과 입금이 함께 성공하거나 실패 |
| **Consistency (일관성)** | 트랜잭션 실행 전후에 데이터베이스의 일관된 상태 유지 | 잔액이 음수가 되지 않도록 제약조건 유지 |
| **Isolation (격리성)** | 동시에 실행되는 트랜잭션들이 서로 영향을 주지 않음 | A의 조회가 B의 진행 중인 변경을 보지 않음 |
| **Durability (지속성)** | 성공한 트랜잭션의 결과는 영구적으로 저장됨 | 시스템 장애 후에도 데이터 유지 |

### 3.2 격리 수준(Isolation Level)

트랜잭션 격리 수준은 성능과 데이터 정합성 사이의 트레이드오프를 조절한다.

| 격리 수준 | Dirty Read | Non-repeatable Read | Phantom Read | 성능 |
| --- | :---: | :---: | :---: | :---: |
| **Read Uncommitted** | ❌ | ❌ | ❌ | ⭐⭐⭐⭐⭐ |
| **Read Committed** | ✅ | ❌ | ❌ | ⭐⭐⭐⭐ |
| **Repeatable Read** | ✅ | ✅ | ❌ | ⭐⭐⭐ |
| **Serializable** | ✅ | ✅ | ✅ | ⭐⭐ |

#### 각 격리 수준의 문제점

**Dirty Read (더티 리드):**
```sql
-- 트랜잭션 A: UPDATE users SET balance = 0 WHERE id = 1;
-- 트랜잭션 B: SELECT balance FROM users WHERE id = 1; -- 0을 읽음
-- 트랜잭션 A: ROLLBACK; -- 취소됨
-- 결과: 트랜잭션 B는 실제로 존재하지 않았던 데이터를 읽음
```

**Non-repeatable Read (반복 불가능 읽기):**
```sql
-- 트랜잭션 A: SELECT balance FROM users WHERE id = 1; -- 100
-- 트랜잭션 B: UPDATE users SET balance = 200 WHERE id = 1; COMMIT;
-- 트랜잭션 A: SELECT balance FROM users WHERE id = 1; -- 200 (변경됨)
-- 결과: 같은 트랜잭션 내에서 같은 쿼리가 다른 결과를 반환
```

**Phantom Read (팬텀 리드):**
```sql
-- 트랜잭션 A: SELECT COUNT(*) FROM users WHERE age > 20; -- 10명
-- 트랜잭션 B: INSERT INTO users VALUES (21); COMMIT;
-- 트랜잭션 A: SELECT COUNT(*) FROM users WHERE age > 20; -- 11명
-- 결과: 없는 행이 갑자기 나타남
```

### 3.3 잠금(Locking) 메커니즘

격리 수준을 구현하기 위한 핵심 메커니즘이다.

#### 공유 락(Shared Lock, S-Lock) vs 배타 락(Exclusive Lock, X-Lock)
* **공유 락:** 읽기 작업 시 사용, 여러 트랜잭션이 동시에 획득 가능
* **배타 락:** 쓰기 작업 시 사용, 한 트랜잭션만 독점

#### 데드락(Deadlock) 방지
```sql
-- 데드락 발생 예시
-- 트랜잭션 A: UPDATE table1 SET col1 = 1 WHERE id = 1;
-- 트랜잭션 B: UPDATE table2 SET col2 = 1 WHERE id = 2;
-- 트랜잭션 A: UPDATE table2 SET col2 = 2 WHERE id = 2; -- 대기
-- 트랜잭션 B: UPDATE table1 SET col1 = 2 WHERE id = 1; -- 데드락!

-- 해결: 잠금 순서를 일관되게 유지하거나 타임아웃 설정
```

---

## 4. 인덱스와 트랜잭션의 상호작용

### 4.1 인덱스가 트랜잭션에 미치는 영향

**잠금 범위 감소:**
```sql
-- 인덱스 없는 경우: 테이블 전체 잠금
UPDATE users SET status = 'inactive' WHERE age > 60;

-- 인덱스 있는 경우: 필요한 행만 잠금
UPDATE users SET status = 'inactive' WHERE age > 60;
```

**하지만 주의:** 인덱스가 많을수록 트랜잭션 동안 더 많은 잠금을 관리해야 함

### 4.2 트랜잭션이 인덱스에 미치는 영향

**격리 수준에 따른 인덱스 효율:**
* 낮은 격리 수준: 더티 리드가 가능해 빠르지만 데이터 정합성 문제
* 높은 격리 수준: 느리지만 데이터 정합성 보장

---

## 5. Production-Ready 구현 예시

### 5.1 MySQL 인덱스 생성 및 모니터링

```sql
-- 1. 인덱스 생성
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_post_user_created ON posts(user_id, created_at);

-- 2. 인덱스 사용 확인
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 3. 인덱스 통계 확인
SHOW INDEX FROM users;
ANALYZE TABLE users;

-- 4. 트랜잭션 격리 수준 설정
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
-- 작업 수행
COMMIT;
```

### 5.2 PostgreSQL에서 트랜잭션과 인덱스

```sql
-- PostgreSQL은 기본적으로 Read Committed
BEGIN;
-- 작업 수행
SAVEPOINT my_savepoint; -- 롤백 포인트
-- 추가 작업
ROLLBACK TO my_savepoint; -- 부분 롤백
COMMIT;

-- 인덱스 생성 (CONCURRENTLY로 블로킹 방지)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
```

---

## 6. 성능 튜닝 전략

### 6.1 인덱스 튜닝
* **Slow Query Log** 분석으로 비효율적 쿼리 찾기
* **Composite Index**로 여러 조건을 하나의 인덱스로 처리
* **Partial Index**로 필요한 데이터만 인덱싱 (PostgreSQL)

### 6.2 트랜잭션 튜닝
* **가능한 짧게:** 트랜잭션 시간을 최소화
* **적절한 격리 수준:** 대부분의 경우 Read Committed로 충분
* **Connection Pool:** 연결 재사용으로 오버헤드 감소

### 6.3 모니터링 지표
* **Lock Wait Time:** 잠금 대기 시간
* **Deadlock Count:** 데드락 발생 빈도
* **Index Usage:** 인덱스 히트율

---

## 7. 전문가적 조언 (Pro Tip)

### 7.1 인덱스 설계 원칙
* **규칙 1:** WHERE, JOIN, ORDER BY에서 자주 사용되는 컬럼에 인덱스 생성
* **규칙 2:** 카디널리티가 높은 컬럼 우선
* **규칙 3:** 쓰기 작업이 많은 테이블은 인덱스 수를 최소화

### 7.2 트랜잭션 설계 원칙
* **규칙 1:** 필요한 최소 범위만 잠금
* **규칙 2:** 데드락을 피하기 위해 리소스 접근 순서 표준화
* **규칙 3:** 롤백이 쉬운 작은 트랜잭션으로 분할

### 7.3 데이터베이스별 특징
* **MySQL InnoDB:** 클러스터형 인덱스 기본, 갭 락으로 팬텀 리드 방지
* **PostgreSQL:** MVCC로 높은 동시성 지원, Partial Index 강력
* **Oracle:** UNDO 세그먼트로 읽기 일관성 유지
