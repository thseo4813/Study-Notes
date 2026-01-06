# 통신의 7가지 관문: OSI 7 Layer & Network Architecture
(부제: 이론부터 SOTA 튜닝, 면접 대비까지)

이 문서는 네트워크의 뼈대인 **OSI 7계층**에서 시작하여, **TCP/UDP의 실무적 특성**, 그리고 대규모 트래픽을 처리하는 **고성능 아키텍처 전략**까지 엔지니어링 관점에서 통합 정리한 문서이다.

---

## 1. The Foundation: OSI 7 Layer vs Real World

네트워크 통신 과정을 7단계로 나눈 표준 모델이다. 실무에서는 이를 단순화한 **TCP/IP 4계층** 모델이 주로 사용되지만, **장애 격리(Troubleshooting Isolation)**를 위해 7계층의 개념을 이해하는 것이 필수적이다.

### 1.1. 계층별 핵심 요약 (Engineer's View)

| 계층 (Layer) | 이름 | 핵심 역할 | 다루는 단위 (PDU) | 실무 관련 장비/기술 |
| :--- | :--- | :--- | :--- | :--- |
| **L7** | **Application** | 사용자가 접하는 서비스, 비즈니스 로직 | Data | HTTP/1.1, HTTP/2, DNS, gRPC |
| **L6** | **Presentation** | 데이터의 변환, 암호화, 압축 (Format) | Data | TLS(SSL), JSON/Protobuf, JPEG |
| **L5** | **Session** | 통신 세션 구성 및 동기화 | Data | RPC, Socket Check |
| **L4** | **Transport** | **전송 방식 결정, 포트 번호 구분, 신뢰성** | Segment/Datagram | **TCP, UDP, L4 Load Balancer** |
| **L3** | **Network** | 경로 설정(Routing), 논리적 주소(IP) | Packet | Router, AWS VPC, IPSec |
| **L2** | **Data Link** | 물리적 주소(MAC), 에러 검출, 흐름 제어 | Frame | L2 Switch, Ethernet, WiFi |
| **L1** | **Physical** | 전기적 신호 전송 (0과 1) | Bit | Cable, NIC(랜카드) |



### 1.2. Why it matters? (실무적 의의)
* **캡슐화(Encapsulation):** 데이터가 내려갈수록 헤더가 붙고(L7→L1), 올라갈수록 벗겨진다(L1→L7).
* **장애의 분리(Isolation):**
    * *"핑(Ping)이 안 나간다"* → L3(IP/Routing) 혹은 L1(케이블) 문제인가?
    * *"웹사이트가 500 에러다"* → L7(애플리케이션 코드) 문제다.
    * *"Connection Timed out"* → L4(방화벽/포트) 문제일 가능성이 높다.

---

## 2. The Two Giants: TCP vs UDP (Core Comparison)

### 2.1. TCP (Transmission Control Protocol)
* **철학:** "속도보다 **신뢰성**이 먼저다."
* **특징:**
    * **연결 지향(Connection-oriented):** 3-way Handshake로 연결 수립 후 통신.
    * **순서 보장:** 패킷 순서가 뒤바뀌어도 재조립함.
    * **흐름/혼잡 제어:** 네트워크 상태에 따라 전송 속도를 조절함.
* **단점:** Handshake 비용 발생, 헤더가 큼(20~60 bytes), **HOL(Head of Line) Blocking** 발생 가능.

### 2.2. UDP (User Datagram Protocol)
* **철학:** "신뢰성보다 **속도**와 **실시간성**이 먼저다."
* **특징:**
    * **비연결형(Connection-less):** Handshake 없이 그냥 쏨 (Fire and Forget).
    * **단순함:** 패킷 헤더가 매우 작음(8 bytes). 순서 보장 안 함.
* **활용:** DNS, 스트리밍, 실시간 게임, **HTTP/3 (QUIC)**.

---

## 3. Real-world Implementation Strategies (실무 구현 전략)

### 3.1. TCP Connection Pool Management
3-way Handshake 비용을 줄이기 위해 커넥션을 재사용하는 핵심 기술. 잘못 관리하면 장애의 주원인이 된다.

| 관리 포인트 | 실무 권장 설정 및 주의사항 |
| :--- | :--- |
| **Pool Size** | 무조건 크다고 좋지 않음(Context Switching 비용). `(Core * 2) + Disk spindle` 공식을 참고하되 부하 테스트로 임계점 파악 필요. |
| **LifeCycle** | **MaxLifeTime**을 인프라(LB/Firewall)의 Idle Timeout보다 **짧게** 설정하여 `Stale Connection` 오류 방지. |
| **Timeout** | `ConnectTimeout`(짧게, 예: 1~3초)과 `ReadTimeout`(길게)을 명확히 분리하여 설정. |

### 3.2. gRPC와 전송 계층의 이슈 (Critical)
gRPC는 HTTP/2를 기반으로 하며, HTTP/2는 **Single Persistent TCP Connection**을 사용한다.
* **문제점:** L4 로드밸런서는 커넥션(IP:Port) 단위로 부하를 분산한다. gRPC는 연결을 끊지 않고 하나만 유지하므로, 트래픽이 최초 연결된 서버로만 쏠린다.
* **해결책:**
    1.  **L7 Load Balancer** (Envoy, Nginx) 도입.
    2.  **Client-side Load Balancing** 구현.
    3.  `MaxConnectionAge`를 설정하여 주기적으로 재연결 유도.

### 3.3. 서버 구현 패턴 (Blocking vs Non-Blocking)
* **Legacy:** Thread per Request (동시 접속 많으면 메모리 부족/스레드 경합).
* **Modern:** **Event-driven & Non-blocking I/O (I/O Multiplexing)**.
    * Linux `epoll`, Java Netty, Node.js Event Loop.
    * 소수의 스레드로 수만 개의 연결(C10K 문제)을 처리. `sysctl` 튜닝(`fs.file-max`, `somaxconn`) 필수.

---

## 4. High-Performance Architecture Deep Dive (SOTA 최적화)

### 4.1. TCP 혼잡 제어의 세대 교체 (CUBIC vs BBR)
* **CUBIC (Linux Default):** 패킷 손실(Loss) 기반. 망이 불안정하면 속도가 급락함.
* **BBR (By Google):** 대역폭(Bandwidth)과 RTT 기반. 패킷 손실이 있어도 물리적 대역폭만큼 전송 속도를 유지함.
    * *Action:* 글로벌 서비스나 모바일 환경 서버라면 `net.ipv4.tcp_congestion_control = bbr` 적용 시 처리량 30% 이상 향상 가능.

### 4.2. Latency Killer: Nagle vs Delayed ACK
* **현상:** Nagle 알고리즘(보낼 데이터 모으기)과 Delayed ACK(받을 때까지 기다리기)가 만나면 서로를 기다리는 **Deadlock** 유발 (최소 40ms 지연).
* **해결:** 실시간성이 중요한 API 서버는 반드시 `TCP_NODELAY` 옵션을 켜서 Nagle 알고리즘을 비활성화해야 한다.



### 4.3. Zero-Copy & Hardware Offloading
* **Zero-Copy:** `sendfile()` 시스템 콜을 사용하여 커널 영역에서 바로 NIC로 데이터 전송 (CPU Copy 제거). Kafka, Nginx의 성능 비결.
* **TSO/GRO (Offloading):** 패킷을 쪼개고 합치는 작업을 CPU 대신 **NIC 하드웨어**가 수행하도록 위임.

---

## 5. Load Balancing Strategies: L4 vs L7

### 5.1. L4 LB (Transport Layer)
* **기준:** IP + Port. (패킷 내용 안 봄)
* **특징:** Pass-through 방식. 매우 빠르고 처리량이 높음.
* **한계:** `/api`, `/img` 같은 URL 기반 라우팅 불가.

### 5.2. L7 LB (Application Layer)
* **기준:** HTTP Header, URL, Cookie, Payload.
* **특징:** 내용을 분석하여 지능적 라우팅 가능 (MSA, A/B Test). SSL Termination 수행.
* **한계:** 패킷 복호화 및 분석 비용으로 인한 부하 발생 (CPU Heavy).



> **Modern Architecture:** L4(입구, 트래픽 분산) → L7(내부, 라우팅/로직) 형태의 **2-Tier Load Balancing**이 표준이다.

---

## 6. Interview Deep Dive (면접 대비 & 시나리오)

**Q1. HTTPS 통신 시 L7 LB 뒤단의 백엔드 서버와는 암호화가 되어있나요?**
* **A:** 보통 성능을 위해 **SSL Offloading**을 하여 LB-서버 간은 HTTP(평문) 통신을 합니다. 단, 금융권 등 제로 트러스트(Zero Trust) 보안 규정이 필요한 경우 다시 암호화(Re-encryption)를 하기도 합니다.

**Q2. `TIME_WAIT` 소켓이 너무 많습니다. 장애인가요?**
* **A:** 당장 장애는 아니지만, 포트 고갈(Port Exhaustion)의 위험 신호입니다. `net.ipv4.tcp_tw_reuse = 1` 커널 파라미터를 활성화하여 안전한 상태의 소켓을 재사용하도록 설정해야 합니다.

**Q3. 신뢰성이 중요한 파일 전송에 UDP를 쓸 수 있나요?**
* **A:** 네, 가능합니다. UDP 자체는 신뢰성이 없지만, **QUIC(HTTP/3)**처럼 애플리케이션 레벨(코드)에서 순서 보장과 재전송 로직을 직접 구현하면, TCP의 HOL Blocking 문제 없이 신뢰성 있는 전송이 가능합니다.

**Q4. L4 LB 사용 시 특정 서버에만 트래픽이 몰립니다. 원인은?**
* **A:** 클라이언트가 **Persistent Connection**(Keep-Alive, gRPC 등)을 사용 중일 확률이 높습니다. L4는 연결 단위 분산이므로, 연결을 주기적으로 끊어주거나(MaxConnectionAge), 요청 단위 분산이 가능한 L7 LB로 교체해야 합니다.
