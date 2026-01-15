# 📨 메시지 큐 완벽 이해: 비동기 아키텍처의 핵심

> **이 문서의 목표:** 메시지 큐를 단순 도구가 아니라, **왜 필요한지**, **어떤 문제를 해결하는지** 분산 시스템 관점에서 이해한다.

---

## 0. 핵심 질문으로 시작하기

1. **동기 vs 비동기의 본질적 차이는?** → 대기 vs 비대기
2. **메시지 큐가 해결하는 문제는?** → 결합도, 장애 격리, 부하 평활화
3. **정확히 한 번(Exactly Once) 처리가 왜 어려운가?** → 분산 시스템의 본질적 문제
4. **RabbitMQ vs Kafka 선택 기준은?** → 설계 철학의 차이

---

## 1. 왜 메시지 큐가 필요한가?

### 1.1 동기 방식의 문제

```mermaid
graph TD
    User((User)) -->|1. 회원가입 요청| Signup[회원가입 서비스]
    Signup -->|2. 이메일 발송 요청| Email[이메일 서비스]
    Signup -->|3. 포인트 적립 요청| Point[포인트 서비스]

    style Email fill:#ffcdd2,stroke:#c62828
    style Point fill:#e0e0e0,stroke:#9e9e9e,stroke-dasharray: 5 5

    linkStyle 1 stroke:#c62828,stroke-width:2px,color:red;
    linkStyle 2 stroke:#9e9e9e,stroke-width:2px,stroke-dasharray: 5 5;

    Note_Email[❌ 장애 발생!] --- Email
    Note_Point[❌ 실행 안 됨] --- Point
```

### 1.2 메시지 큐의 해결책

```mermaid
graph TD
    User((User)) -->|1. 회원가입 요청| Signup[회원가입 서비스]
    
    subgraph Async_Layer [비동기 처리 구간]
        MQ[("Message Queue")]
    end
    
    Signup -->|2. 이벤트 발행| MQ
    Signup -.->|3. 즉시 응답 OK| User
    
    MQ -.->|4. 나중에 처리| Email[이메일 서비스]
    MQ -.->|4. 나중에 처리| Point[포인트 서비스]

    style Signup fill:#e3f2fd,stroke:#1565c0
    style MQ fill:#fff9c4,stroke:#fbc02d
    style Email fill:#ffcc80,stroke:#ef6c00
    style Point fill:#ffcc80,stroke:#ef6c00
    
    Note_Fail[❌ 장애 발생해도<br/>큐에 안전하게 저장됨] -.-> Email
```

### 1.3 메시지 큐의 세 가지 이점

```
1. 결합도 감소
   Producer는 Consumer를 모름
   → 독립적 개발/배포

2. 장애 격리
   하나 죽어도 다른 것 영향 없음
   → 시스템 안정성

3. 부하 평활화 (Load Leveling)
   트래픽 폭증 시 큐가 완충
   → 백엔드 보호
```

---

## 2. 분산 시스템의 핵심 문제: 메시지 전달 보장

### 2.1 세 가지 보장 수준

```
At-Most-Once (최대 한 번):
  보내고 끝, 확인 안 함
  → 손실 가능, 중복 없음
  → 로그, 메트릭 (손실 괜찮음)

At-Least-Once (최소 한 번):
  ACK 받을 때까지 재전송
  → 손실 없음, 중복 가능
  → 대부분의 서비스 (멱등성 필요)

Exactly-Once (정확히 한 번):
  손실도 중복도 없음
  → 가장 어려움, 성능 비용 높음
  → 금융, 결제 (반드시 필요)
```

### 2.2 Exactly-Once가 어려운 이유

```
[네트워크 문제 시나리오]
1. Producer → Broker: 메시지 전송
2. Broker: 저장
3. Broker → Producer: ACK 전송
4. (네트워크 끊김! ACK 못 받음)
5. Producer: "실패한 줄 알고" 재전송
6. 결과: 메시지 중복

[해결책]
멱등성(Idempotency) 보장:
같은 메시지 여러 번 처리해도 결과 동일
예: 메시지 ID로 중복 체크
```

---

## ⚡ 실제 비동기 처리 문제들

### 시스템 통합 시 흔히 하는 고민:

**"API 호출이 너무 느려서 사용자 응답이 지연돼!"**
- 이메일 발송 때문에 회원가입이 5초 걸림
- 결제 완료 후 포인트 적립이 느려서 사용자 불만
- 이미지 업로드 후 리사이징이 오래 걸려서 화면 멈춤

**"외부 API가 장애나면 우리 서비스도 죽어!"**
- 결제 대행사 다운으로 주문 불가
- 이메일 서비스 장애로 알림 못 보냄
- 외부 API 타임아웃으로 전체 서비스 블락

**"트래픽이 갑자기 몰리면 시스템이 터져!"**
- 이벤트 때 주문량 폭증으로 DB 과부하
- 피크타임에 메시지 처리 지연
- 큐가 꽉 차서 메시지 유실

## 🎯 1분 요약: 메시지 큐의 핵심

**메시지 큐 = 시스템 간 완충 장치**

- **동기**: 요청 → 기다림 → 응답 (블로킹)
- **비동기**: 요청 → 큐에 넣음 → 바로 응답 (논블로킹)
- **장점**: 결합도 낮춤, 장애 격리, 트래픽 평활화

> **결론:**
> 1. **RabbitMQ**: 복잡한 라우팅, 신뢰성 우선
> 2. **Kafka**: 대용량 스트리밍, 재생 기능 필요
> 3. **Redis Queue**: 단순한 작업 큐, 빠른 처리
> 
> 
> 
> 

---

## 2. 동기 vs 비동기: 실제 적용 사례

**💡 서비스별 적용 예시:**

| 서비스 | 동기/비동기 | 이유 |
|--------|-------------|------|
| **쿠팡 주문** | 비동기 (메시지 큐) | 결제 후 재고 차감, 배송 처리 |
| **카카오톡 메시지** | 비동기 (WebSocket + 큐) | 실시간이지만 큐로 안정성 확보 |
| **은행 송금** | 동기 (2PC) | 즉시 확인 필요, 롤백 가능해야 |

**🚨 실제 문제 사례:**

**문제 1: 동기로 인한 사용자 응답 지연**
```java
// ❌ 동기 처리로 인한 느린 응답
@PostMapping("/signup")
public User signup(UserRequest request) {
    User user = userService.createUser(request);

    // 이메일 발송까지 기다림 (5초 소요!)
    emailService.sendWelcomeEmail(user.getEmail());

    // 포인트 적립까지 기다림 (2초 소요!)
    pointService.giveWelcomePoints(user.getId());

    return user;  // 총 7초 후 응답
}
```

```java
// ✅ 비동기로 빠른 응답
@PostMapping("/signup")
public User signup(UserRequest request) {
    User user = userService.createUser(request);

    // 메시지 큐에 이벤트 발행 (즉시 완료!)
    eventPublisher.publish("user.created", user);

    return user;  // 즉시 응답 (0.1초)
}

// 별도 Consumer에서 처리
@RabbitListener(queues = "user-events")
public void handleUserCreated(User user) {
    emailService.sendWelcomeEmail(user.getEmail());
    pointService.giveWelcomePoints(user.getId());
}
```

**문제 2: 외부 API 장애로 인한 연쇄 실패**
```java
// ❌ 외부 API 직접 호출
public void processPayment(Order order) {
    try {
        paymentGateway.charge(order);  // PG사 다운 시 실패!
        inventoryService.reduceStock(order);
        shippingService.createLabel(order);
    } catch (Exception e) {
        // 결제 실패 시 모든 작업 취소
        throw new PaymentFailedException();
    }
}
```

```java
// ✅ 메시지 큐로 장애 격리
public void processPayment(Order order) {
    paymentGateway.charge(order);  // 동기 (중요하므로)

    // 나머지는 큐로 비동기 처리
    eventPublisher.publish("payment.completed", order);
}

// Consumer에서 안전하게 처리
@KafkaListener(topics = "payment-completed")
public void handlePaymentCompleted(Order order) {
    try {
        inventoryService.reduceStock(order);
        shippingService.createLabel(order);
    } catch (Exception e) {
        // 재시도 로직
        retry(order);
    }
}
```

**문제 3: 큐가 꽉 차서 메시지 유실**
```java
// ❌ 큐 용량 제한 없이 메시지 발행
for (int i = 0; i < 1000000; i++) {
    rabbitTemplate.convertAndSend("order-queue", order);
    // 큐가 꽉 차면 메시지 유실!
}
```

```java
// ✅ Backpressure 적용
@RabbitListener(queues = "order-queue")
public void processOrder(Order order) {
    // 처리 속도를 조절하여 큐 과부하 방지
    processSlowly(order);
}

// 또는 Circuit Breaker
if (queueSize > threshold) {
    // 큐가 가득 차면 잠시 요청 거부
    throw new TooManyRequestsException();
}
```



### 2.2 비동기 방식 (Message Queue)

* **시나리오:** 회원가입(A)  "가입했음" 메시지를 큐에 던짐(Publish)  A 즉시 완료 응답. (사용자는 빠름)  이메일 서버(B)가 나중에 큐에서 메시지를 꺼내서(Consume) 발송.
* **장점:**
1. **빠른 응답:** 사용자는 이메일 발송을 기다리지 않음.
2. **장애 격리:** 이메일 서버가 죽어도 큐에 메시지가 쌓여 있을 뿐, 회원가입은 정상 동작함. 나중에 서버가 살아나면 처리하면 됨.



---

## 3. 아키텍처 다이어그램 (Event-Driven Architecture)

가장 대표적인 사용 사례인 **"회원가입 후 후처리"** 프로세스입니다.

```mermaid
graph LR
    User[👤 User] -- "Sign Up" --> AuthService[Auth Service]
    
    subgraph Broker [Message Broker (Kafka/RabbitMQ)]
        direction TB
        Queue[("✉️ Message Queue <br/> (Topic: user.created)")]
    end
    
    AuthService -- "1. Publish Event" --> Queue
    AuthService -- "2. Response OK" --> User
    
    Queue -- "3. Consume" --> EmailService[📧 Email Service]
    Queue -- "3. Consume" --> CouponService[🎟️ Coupon Service]
    Queue -- "3. Consume" --> LogService[📊 Log Service]
    
    style Queue fill:#ffcc80,stroke:#ef6c00
    style AuthService fill:#e1f5fe,stroke:#0277bd
    style Broker fill:#fff3e0,stroke:#ff9800
    style EmailService fill:#f5f5f5,stroke:#333
    style CouponService fill:#f5f5f5,stroke:#333
    style LogService fill:#f5f5f5,stroke:#333
```

---

## 4. RabbitMQ vs Kafka: 무엇을 써야 하는가?

둘 다 메시지를 주고받지만, 설계 철학과 용도가 완전히 다르다.

| 구분 | RabbitMQ | Apache Kafka |
| --- | --- | --- |
| **기본 철학** | **"똑똑한 브로커, 멍청한 컨슈머"** <br>

<br> 브로커가 메시지 전달 상태를 관리함. | **"멍청한 브로커, 똑똑한 컨슈머"** <br>

<br> 브로커는 파일시스템에 저장만 하고, 관리는 컨슈머가 함. |
| **메시지 보존** | 소비(Ack)되면 **삭제됨**. (휘발성) | 소비되어도 디스크에 **남아있음**. (설정 기간 동안) |
| **처리량(Throughput)** | 초당 수만 건 (안정성 중시) | 초당 수백만 건 (속도 중시) |
| **주요 용도** | 복잡한 라우팅(1:1, 1:N), 작업 큐(Task Queue), 즉시 처리 | 대용량 로그 수집, 실시간 스트리밍, 이벤트 소싱 |
| **프로토콜** | AMQP (표준 프로토콜) | TCP 기반 바이너리 프로토콜 (자체) |

> **Pro Tip:**
> * 단순히 "이메일 보내기", "푸시 알림 보내기" 같은 **작업(Task)** 위주라면 **RabbitMQ** (또는 AWS SQS)가 관리하기 훨씬 편합니다.
> * "사용자 클릭 로그 전수 수집", "CDC(DB 변경 감지)" 같은 **데이터 파이프라인**이라면 **Kafka**가 표준입니다.
> 
> 

---

## 5. Production-Ready Code Example (Python + RabbitMQ)

RabbitMQ의 Python 클라이언트인 `pika`를 사용한 생산자(Producer)와 소비자(Consumer) 패턴입니다.

### 5.1 Producer (회원가입 서버)

```python
import pika
import json

# 연결 설정 (실무에선 커넥션 풀 사용 권장)
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# 큐 선언 (durable=True: 브로커가 재시작되어도 큐 유지)
channel.queue_declare(queue='email_task_queue', durable=True)

def sign_up_user(user_data):
    # 1. DB 저장 로직 (생략)
    print(f"DB Saved: {user_data['id']}")
    
    # 2. 메시지 발행 (Fire & Forget)
    message = json.dumps(user_data)
    channel.basic_publish(
        exchange='',
        routing_key='email_task_queue',
        body=message,
        properties=pika.BasicProperties(
            delivery_mode=2,  # 메시지를 디스크에 영구 저장 (Persistent)
        ))
    print(" [x] Sent Email Task")

sign_up_user({'id': 1, 'email': 'user@example.com'})
connection.close()

```

### 5.2 Consumer (이메일 발송 서버)

```python
import pika
import time
import json

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='email_task_queue', durable=True)

def callback(ch, method, properties, body):
    user_data = json.loads(body)
    print(f" [>] Received task for: {user_data['email']}")
    
    try:
        # 3. 실제 이메일 발송 로직 (SMTP 등)
        time.sleep(2) # 모의 지연 시간
        print(" [v] Email Sent")
        
        # 4. [중요] 처리 완료 신호 (Ack)
        # 이걸 안 보내면 큐는 처리가 안 된 줄 알고 다른 워커에게 다시 줌
        ch.basic_ack(delivery_tag=method.delivery_tag)
        
    except Exception as e:
        # 실패 시 로직 (재시도 혹은 Dead Letter Queue로 이동)
        print(f" [!] Error: {e}")
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=False)

# 공평 분배 (워커가 바쁘면 일 주지 마라)
channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='email_task_queue', on_message_callback=callback)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()

```

---

## 6. 전문가적 조언 (Pro Tip)

### 6.1 멱등성 (Idempotency) 보장 필수

메시지 큐는 네트워크 문제로 인해 **"적어도 한 번(At-least-once)"** 전송을 보장하는 경우가 많습니다. 즉, **같은 메시지가 두 번 올 수 있습니다.**

* **문제:** 이메일 발송 서버가 같은 메시지를 두 번 받으면, 유저에게 가입 환영 메일이 두 통 날아갑니다.
* **해결:** Consumer는 반드시 멱등성을 가져야 합니다.
* 메시지 ID를 Redis에 저장하여 `if exists(msg_id): skip` 처리를 하거나,
* 로직 자체가 여러 번 수행되어도 결과가 같도록 설계해야 합니다.



### 6.2 Dead Letter Queue (DLQ)

코드 버그나 데이터 문제로 인해 소비자가 **영원히 처리할 수 없는 메시지**가 들어올 수 있습니다.

* 이 메시지를 계속 재시도(Retry)하면 큐가 막혀버립니다(Head-of-line blocking).
* **전략:** 3~5회 재시도 후에도 실패하면, 해당 메시지를 별도의 **"죽은 편지함(DLQ)"**으로 옮기고 `Ack` 처리합니다. 이후 개발자가 DLQ를 모니터링하여 원인을 분석하고 수동 처리합니다.

### 6.3 메시지 순서 보장 (Ordering)

큐를 쓰면서 순서를 100% 보장하는 것은 매우 어렵고 성능 비용이 큽니다.

* 특히 여러 개의 Consumer(Worker)가 동시에 큐를 파먹을 때 순서가 뒤섞일 수 있습니다.
* 순서가 중요하다면(예: 결제 생성  결제 완료), Kafka의 파티셔닝(Partitioning) 키를 유저 ID로 설정하여 **"특정 유저의 메시지는 항상 같은 파티션(같은 순서)"**으로 가도록 설계해야 합니다.
