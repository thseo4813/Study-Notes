# 통신의 7가지 관문: OSI 7 Layer & Network Architecture
(부제: 이론부터 SOTA 튜닝, 면접 대비까지)

# 📡 OSI 7계층: 네트워크 문제 디버깅의 지도

## 🔧 실제 네트워크 문제 해결 사례들

### 네트워크 엔지니어들이 흔히 하는 고민:

**"왜 인터넷이 안 되지? 어느 계층 문제일까?"**
- "사이트 접속 안 돼" → DNS 문제? (L7)
- "핑은 되는데 웹은 안 돼" → 포트/방화벽 문제? (L4)
- "와이파이는 연결됐는데 인터넷 안 돼" → IP 할당 문제? (L3)

**"어느 계층에서 성능 튜닝을 해야 할까?"**
- 응답이 느림 → CDN/L7 최적화?
- 패킷 손실 → TCP/L4 튜닝?
- 물리적 연결 불안 → 케이블/L1 교체?

**"보안 취약점은 어느 계층에 있을까?"**
- SQL 인젝션 → 애플리케이션/L7
- 패킷 스니핑 → 암호화/L6
- IP 스푸핑 → 라우팅/L3

## 🎯 1분 요약: OSI 7계층의 핵심

**OSI 7계층 = 네트워크 문제 해결 지도**

- **L1-L4**: 하드웨어/네트워크 인프라 (속도, 연결성)
- **L5-L7**: 소프트웨어/데이터 (보안, 프로토콜)
- **실무**: 문제 발생 시 계층별로 순차적 디버깅

> **결론:** 계층별 문제 격리로 빠른 장애 해결 가능

### 1.1. 계층별 핵심 요약 (Engineer's View)

**💡 계층별 문제 해결 예시:**

| 계층 | 전형적인 문제 | 진단 방법 | 해결 방법 |
|------|-------------|----------|----------|
| **L7** | "웹사이트 500 에러" | 브라우저 개발자 도구 | 애플리케이션 코드 수정 |
| **L6** | "HTTPS 인증서 에러" | `openssl s_client` | SSL 인증서 갱신 |
| **L5** | "세션 타임아웃" | `netstat` 연결 상태 확인 | 세션 관리 설정 조정 |
| **L4** | "포트 연결 거부" | `telnet IP PORT` | 방화벽 규칙 확인 |
| **L3** | "핑은 되는데 웹 안 돼" | `traceroute` | 라우팅 테이블 확인 |
| **L2** | "WiFi 연결 불안정" | `iwconfig` | WiFi 신호 강도 확인 |
| **L1** | "케이블 연결 끊김" | `ethtool` | 물리적 연결 상태 확인 |

**🚨 실제 문제 해결 사례:**

**사례 1: "인터넷은 되는데 특정 사이트만 안 들어가지네?"**
```
1. ping google.com → 응답 O (L3 정상)
2. nslookup google.com → IP 변환 O (L7 DNS 정상)  
3. telnet google.com 80 → 연결 거부 (L4 문제!)
4. 원인: 기업 방화벽이 80번 포트 차단
5. 해결: HTTPS(443)로 접속 또는 VPN 사용
```

**사례 2: "와이파이는 연결됐는데 인터넷이 안 돼!"**
```
1. ping 8.8.8.8 → 응답 O (L3 정상)
2. ping google.com → 실패 (L7 DNS 문제!)
3. nslookup google.com → 타임아웃
4. 원인: DNS 서버 설정 문제 (192.168.1.1 대신 8.8.8.8로 변경)
5. 해결: DNS 서버를 8.8.8.8로 변경
```

**사례 3: "사이트가 갑자기 엄청 느려졌어!"**
```
1. ping 사이트 → 정상 (L3)
2. traceroute 사이트 → 특정 홉에서 지연 (L3 라우팅 문제)
3. mtr 사이트 → 패킷 손실 확인 (L2/L3 문제)
4. 원인: 네트워크 경로 변경 또는 ISP 문제
5. 해결: CDN 사용 또는 다른 ISP 고려
```



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
