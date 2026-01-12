# 📚 Study Notes: 컴퓨터 과학 핵심 개념 정리

> "컴퓨터 과학의 본질을 이해하고, 실무에서 적용할 수 있는 능력을 기르자"

이 저장소는 **컴퓨터 과학의 기초부터 시스템 설계까지** 핵심 개념들을 실무 관점에서 정리한 학습 노트입니다. 단순한 이론 나열이 아닌, **왜**와 **어떻게**를 강조하며 프로덕션 환경에서의 적용 방법을 다룹니다.

## 🎯 학습 목표

- ✅ **개념의 본질 이해:** 표면적인 지식이 아닌 깊이 있는 원리 파악
- ✅ **실무 적용 능력:** 이론을 실제 코드와 아키텍처에 적용하는 방법
- ✅ **트러블슈팅 역량:** 문제가 발생했을 때 근본 원인을 분석하는 사고력
- ✅ **성능 최적화:** 효율적인 시스템 설계를 위한 전략적 사고

## 📖 주요 특징

### 💡 **실무 중심 접근**
- 각 개념마다 **Production-Ready Code Example** 포함
- 실제 서비스에서 발생하는 문제와 해결책 제시
- 업계 표준 도구와 프레임워크 활용

### 🔍 **심층 분석**
- 단순 암기가 아닌 **왜 이렇게 동작하는가**에 초점
- **Trade-off** 분석과 상황별 최적 선택 가이드
- **Pro Tip**으로 전문가 수준의 인사이트 제공

### 📊 **시각화 지원**
- **Mermaid 다이어그램**으로 복잡한 개념 명확화
- **비교 테이블**로 장단점 한눈에 파악
- **시퀀스 다이어그램**으로 동작 흐름 이해

---

## 📂 상세 목차

### 💻 [00-cs-basics: CS 기초](./00-cs-basics)
컴퓨터 과학의 근본적인 개념들을 다룹니다. "왜 컴퓨터는 이렇게 동작하는가"에 대한 답을 찾는 공간입니다.

- [**알고리즘의 세계: 문제 해결의 핵심 도구**](./00-cs-basics/algorithms.md)
  - 정렬 알고리즘 (퀵, 병합, 힙 정렬)
  - 검색 알고리즘 (이진 검색, 해시 검색)
  - 동적 프로그래밍과 그리디 알고리즘
  - 그래프 알고리즘 (DFS/BFS, 최단 경로)
  - 시간/공간 복잡도 분석

- [**컴퓨터 아키텍처 기초: 하드웨어와 소프트웨어의 연결**](./00-cs-basics/computer-architecture.md)
  - CPU 아키텍처와 명령어 처리 파이프라인
  - 메모리 계층 구조와 캐시 원리
  - 병렬 처리 (SIMD, MIMD)와 현대 프로세서
  - I/O 시스템과 인터럽트/DMA

- [**프로그래밍 언어 기초: 언어 설계와 컴파일러 원리**](./00-cs-basics/programming-languages.md)
  - 컴파일 vs 인터프리터의 차이점
  - 정적 vs 동적 타이핑의 트레이드오프
  - 프로그래밍 패러다임 (명령형, 객체지향, 함수형)
  - 메모리 관리 전략과 가비지 컬렉션

- [**자료구조와 알고리즘의 본질**](./00-cs-basics/data-structures)
  - 선형/비선형 구조의 특징과 선택 기준
  - 시간 복잡도(Big O) 분석
  - 실무 적용 예시 (LRU Cache, Priority Queue)

- [**컴퓨터의 실수 표현과 부동 소수점**](./00-cs-basics/number-representation)
  - IEEE 754 표준과 정밀도 손실 메커니즘
  - 금융/과학 연산에서 주의해야 할 함정
  - Decimal 타입과 정수 기반 연산 전략

- [**유니코드와 UTF-8**](./00-cs-basics/unicode-utf8)
  - 문자 집합 vs 인코딩의 구분
  - UTF-8의 가변 길이 인코딩 원리
  - Mojibake 문제와 NFC/NFD 정규화

### 🖥️ [01-os: 운영체제](./01-os)
운영체제가 제공하는 자원 관리와 실행 모델을 이해합니다.

- [**프로세스 vs 스레드**](./01-os/process-vs-thread)
  - 메모리 구조와 문맥 교환 비용 비교
  - 경쟁 상태(Race Condition)와 교착 상태(Deadlock)
  - Mutex/Semaphore와 동기화 전략

- [**스택과 힙**](./01-os/memory-stack-heap)
  - 메모리 레이아웃과 할당 전략
  - Stack Overflow vs Memory Leak
  - Call by Value/Reference의 메모리 관점 분석

- [**유닉스 타임스탬프**](./01-os/time-unix-timestamp)
  - 2038년 문제와 64비트 전환
  - UTC Sandwich 전략으로 타임존 문제 해결
  - ISO 8601 표준과 시간대 처리

- [**가상 메모리**](./01-os/virtual-memory.md)
  - 페이지와 세그먼테이션
  - MMU와 주소 변환
  - 스와핑과 페이지 폴트

### 🌐 [02-network: 네트워크](./02-network)
네트워크 통신의 원리와 최적화 기법을 다룹니다.

- [**OSI 7 Layer**](./02-network/osi-7-layers)
  - 각 계층의 역할과 트러블슈팅 가이드
  - TCP vs UDP의 실무적 특성 비교
  - L4/L7 로드 밸런싱 전략

- [**TCP vs UDP**](./02-network/tcp-vs-udp)
  - 3-Way Handshake와 HOL Blocking
  - HTTP/3와 QUIC의 혁신적 접근
  - 타임아웃, 커넥션 풀, gRPC 적용

- [**DNS 원리**](./02-network/dns-domain)
  - 계층적 구조와 재귀적 질의
  - TTL과 캐싱 전략
  - CNAME vs A Record의 차이점

- [**HTTP vs HTTPS**](./02-network/http-https)
  - 하이브리드 암호화 시스템 (비대칭+대칭키)
  - SSL/TLS Handshake 과정
  - Mixed Content와 HSTS 보안

- [**HTTP/3와 QUIC**](./02-network/http-3.md)
  - UDP 기반의 새로운 전송 프로토콜
  - 0-RTT 핸드셰이크와 연결 마이그레이션
  - 스트림 멀티플렉싱과 헤더 압축

- [**로드 밸런싱**](./02-network/load-balancing)
  - Reverse Proxy vs Forward Proxy
  - Nginx 설정과 헬스 체크 전략
  - SSL Termination과 X-Forwarded-For

### 🗄️ [03-database: 데이터베이스](./03-database)
데이터 저장과 검색의 효율성을 극대화하는 기술입니다.

- [**확장 전략 (Replication/Sharding)**](./03-database/replication-sharding)
  - Master-Slave Replication과 Lag 문제
  - Range/Hash Sharding 전략
  - CAP 이론과 실무적 선택 기준

- [**인덱스와 트랜잭션**](./03-database/index-transaction)
  - B-Tree vs Hash 인덱스 구조
  - ACID 속성과 격리 수준(Isolation Level)
  - 잠금 메커니즘과 데드락 방지

- [**캐싱 전략과 Redis**](./03-database/caching-redis)
  - Look-aside vs Write-through 패턴
  - Thundering Herd 문제와 해결책
  - 메모리 관리와 Eviction Policy

### 🏗️ [04-system-design: 시스템 설계](./04-system-design)
대규모 분산 시스템을 설계하는 아키텍처 패턴입니다.

- [**대규모 데이터 시스템 설계**](./04-system-design/large-scale-system)
  - 수직 vs 수평 확장 전략
  - 마이크로서비스와 API Gateway
  - 모니터링과 장애 복구 패턴

- [**메시지 큐 (Kafka, RabbitMQ)**](./04-system-design/message-queue)
  - 동기 vs 비동기 처리의 트레이드오프
  - RabbitMQ vs Kafka 선택 기준
  - 멱등성과 데드 레터 큐

- [**인증 vs 인가**](./04-system-design/auth-oauth)
  - Session vs JWT 비교
  - OAuth 2.0 플로우
  - Access/Refresh Token 전략

