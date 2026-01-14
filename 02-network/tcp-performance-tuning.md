# 🔧 TCP 성능 튜닝 & 트러블슈팅: 서버가 터지는 상황 해결하기

## 🚨 실무에서 마주치는 TCP 문제들

### 서버 운영 시 흔히 겪는 상황:

**"갑자기 새 연결이 안 맺어져요!"**
- `netstat`으로 보니 `TIME_WAIT` 상태가 수만 개
- 포트 고갈로 인한 연결 거부 (Connection Refused)
- 트래픽은 정상인데 서버 응답이 없음

**"서버가 연결을 받지 않아요!"**
- `connection refused` 에러 폭주
- SYN Flood 공격도 아닌데 SYN Queue가 가득 참
- `backlog` 설정 문제로 연결 드롭

**"연결은 됐는데 데이터가 안 와요!"**
- `Recv-Q`가 점점 쌓여서 애플리케이션이 멈춤
- 좀비 커넥션(Half-open)으로 리소스 낭비
- 네트워크 지연인지 서버 문제인지 판단 어려움

---

## 🎯 1분 요약: TCP 튜닝의 핵심

**TCP 문제 = 연결 수명 주기(Lifecycle) 이해 + 커널 파라미터 튜닝**

- **연결 수립**: SYN Queue, Accept Queue, `backlog` 설정
- **연결 유지**: Keep-Alive, Heartbeat, 버퍼 관리
- **연결 종료**: `TIME_WAIT` 상태, 포트 재사용 정책

> **결론:**
> 1. **포트 고갈**: `tcp_tw_reuse` + 커넥션 풀로 해결
> 2. **연결 거절**: `somaxconn` + `backlog` 증가로 해결
> 3. **좀비 연결**: Keep-Alive + 애플리케이션 Heartbeat로 해결

---

## 2. TIME_WAIT와 포트 고갈 문제

### 2.1 TIME_WAIT란?

TCP 연결을 **먼저 종료하는 쪽(Active Close)**이 반드시 거치는 상태다. 약 **60초(2MSL)** 동안 유지된다.

```
[TCP 4-Way Handshake - 연결 종료]

Client (Active Close)              Server (Passive Close)
        |                                  |
        |------- FIN (seq=100) ----------->|
        | 상태: FIN_WAIT_1                 | 상태: CLOSE_WAIT
        |                                  |
        |<------ ACK (ack=101) ------------|
        | 상태: FIN_WAIT_2                 |
        |                                  |
        |<------ FIN (seq=300) ------------|
        | 상태: TIME_WAIT ⚠️              | 상태: LAST_ACK
        |                                  |
        |------- ACK (ack=301) ----------->|
        |                                  | 상태: CLOSED
        |                                  |
        | (60초 대기 후)                   |
        | 상태: CLOSED                     |
```

**왜 60초나 기다릴까?**
1. **지연 패킷 처리**: 네트워크에 떠도는 이전 연결의 패킷이 새 연결에 섞이는 것 방지
2. **마지막 ACK 손실 대비**: 상대방이 FIN 재전송 시 응답할 수 있어야 함

### 2.2 포트 고갈(Port Exhaustion) 현상

```bash
# 문제 상황: TIME_WAIT 폭주
$ netstat -an | grep TIME_WAIT | wc -l
58234

# 사용 가능한 임시 포트 범위 확인
$ cat /proc/sys/net/ipv4/ip_local_port_range
32768   60999
# 약 28,000개의 포트만 사용 가능
```

**포트 고갈 시나리오:**
```
[서버 → 외부 API 호출 시나리오]

서버(10.0.0.1) ----HTTP 요청----> API서버(api.example.com:443)
         |
         └── 출발 포트: 32768 ~ 60999 중 랜덤 선택
         
초당 1000개 요청 시:
- 1초에 1000개 포트 사용
- TIME_WAIT 60초 유지 → 60,000개 포트 필요
- 가용 포트 28,000개 → ❌ 포트 고갈!
```

### 2.3 해결책: 커널 파라미터 튜닝

```bash
# /etc/sysctl.conf 또는 /etc/sysctl.d/99-tcp-tuning.conf

# 1. TIME_WAIT 소켓 재사용 (가장 중요!)
# 안전하게 TIME_WAIT 소켓을 새 연결에 재사용
net.ipv4.tcp_tw_reuse = 1

# 2. 임시 포트 범위 확대
net.ipv4.ip_local_port_range = 10000 65535

# 3. FIN_WAIT 상태 타임아웃 단축 (기본 60초 → 30초)
net.ipv4.tcp_fin_timeout = 30

# 4. TCP 연결 추적 테이블 크기 증가 (NAT/방화벽 서버)
net.netfilter.nf_conntrack_max = 1048576

# 적용
$ sysctl -p
```

**⚠️ 주의: `tcp_tw_recycle`은 사용 금지!**
```bash
# ❌ 절대 사용하지 말 것 (NAT 환경에서 연결 실패 유발)
# net.ipv4.tcp_tw_recycle = 1  # Linux 4.12 이후 제거됨
```

### 2.4 근본 해결책: 커넥션 풀(Connection Pool)

커널 튜닝은 임시방편이다. 근본 해결은 **연결을 재사용**하는 것이다.

```java
// ❌ Bad: 매번 새 연결 생성
public String callApi(String endpoint) {
    HttpURLConnection conn = (HttpURLConnection) new URL(endpoint).openConnection();
    // ... 사용 후 close
    conn.disconnect();  // 이 순간 TIME_WAIT 발생
    return response;
}

// ✅ Good: 커넥션 풀 사용 (Apache HttpClient)
public class ApiClient {
    private static final CloseableHttpClient httpClient = HttpClients.custom()
        .setConnectionManager(new PoolingHttpClientConnectionManager())
        .setMaxConnTotal(200)        // 전체 최대 연결 수
        .setMaxConnPerRoute(50)      // 호스트당 최대 연결 수
        .setConnectionTimeToLive(5, TimeUnit.MINUTES)  // 연결 최대 수명
        .build();
    
    public String callApi(String endpoint) {
        HttpGet request = new HttpGet(endpoint);
        try (CloseableHttpResponse response = httpClient.execute(request)) {
            // ... 응답 처리
            // 연결은 풀에 반환됨 (TIME_WAIT 발생 안 함)
        }
        return result;
    }
}
```

---

## 3. SYN Queue와 Accept Queue 관리

### 3.1 TCP 연결 수립의 두 단계 큐

클라이언트가 `connect()`를 호출하면, 서버에서는 두 개의 큐를 거친다.

```
[TCP 연결 수립 흐름]

Client                                    Server
   |                                        |
   |-------- SYN --------> [SYN Queue]      |  <-- 3-way Handshake 진행 중
   |                           |            |
   |<---- SYN+ACK -------------|            |
   |                           |            |
   |-------- ACK --------> [Accept Queue]   |  <-- 연결 완료, accept() 대기
   |                           |            |
   |               <-- accept() 호출 --     |  <-- 애플리케이션이 꺼내감
   |                           |            |
   |        [연결 확립 - ESTABLISHED]       |
```

### 3.2 큐가 가득 차면 어떻게 될까?

**SYN Queue Overflow:**
- 새 SYN 패킷 드롭 (클라이언트는 재시도)
- SYN Flood 공격 시 주로 발생

**Accept Queue Overflow:**
- 완료된 연결이 드롭됨 (클라이언트는 `connection reset`)
- 애플리케이션이 `accept()`를 늦게 호출할 때 발생

```bash
# 현재 큐 상태 확인
$ ss -ltn
State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port
LISTEN  0       128     0.0.0.0:80          0.0.0.0:*
        ^       ^
        |       └── backlog 설정값 (Accept Queue 최대 크기)
        └── 현재 Accept Queue에 대기 중인 연결 수

# Recv-Q가 Send-Q에 근접하면 위험 신호!
$ ss -ltn | awk '{if($2>$3*0.8) print "⚠️ Queue almost full:", $0}'
```

### 3.3 해결책: 큐 크기 튜닝

```bash
# /etc/sysctl.conf

# 1. 시스템 전역 Accept Queue 최대값
# listen(fd, backlog)의 backlog가 이 값을 초과할 수 없음
net.core.somaxconn = 65535

# 2. SYN Queue 최대 크기
net.ipv4.tcp_max_syn_backlog = 65535

# 3. SYN Cookie 활성화 (SYN Flood 방어)
net.ipv4.tcp_syncookies = 1

# 적용
$ sysctl -p
```

**애플리케이션에서 backlog 설정:**
```python
# Python 소켓 서버
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(('0.0.0.0', 8080))

# backlog 값 설정 (기본값 5는 너무 작음!)
server.listen(65535)  # Accept Queue 최대 크기
```

```java
// Java ServerSocket
ServerSocket serverSocket = new ServerSocket();
serverSocket.setReuseAddress(true);
serverSocket.bind(new InetSocketAddress(8080), 65535);  // backlog=65535
```

```nginx
# Nginx 설정
server {
    listen 80 backlog=65535;
    # ...
}
```

---

## 4. Keep-Alive와 좀비 커넥션 감지

### 4.1 Half-Open 연결(좀비 커넥션)이란?

네트워크 장애나 클라이언트 크래시로 **한쪽만 연결이 끊어진 상태**다.

```
[Half-Open 연결 시나리오]

정상 상태:
Client ←────── ESTABLISHED ──────→ Server

문제 발생 (클라이언트 갑자기 종료/네트워크 끊김):
Client (죽음) ←─────?─────→ Server (여전히 ESTABLISHED)
                                   └── 좀비 커넥션!
                                       리소스(메모리, FD) 낭비
```

**문제점:**
- 서버는 클라이언트가 죽은 줄 모르고 연결 유지
- 파일 디스크립터(FD) 낭비 → `Too many open files` 에러
- 커넥션 풀 고갈 → 새 연결 불가

### 4.2 TCP Keep-Alive (OS 레벨)

```bash
# TCP Keep-Alive 커널 설정

# 1. Keep-Alive 시작 시간 (연결 유휴 후 몇 초 뒤 probe 시작)
net.ipv4.tcp_keepalive_time = 600    # 기본 7200초(2시간) → 600초로 단축

# 2. Keep-Alive Probe 간격
net.ipv4.tcp_keepalive_intvl = 60    # 60초마다 probe 전송

# 3. Keep-Alive Probe 횟수 (이 횟수 실패 시 연결 종료)
net.ipv4.tcp_keepalive_probes = 3    # 3번 실패 → 연결 끊음

# 결과: 600초 유휴 → 60초 간격 × 3회 시도 → 최대 780초 후 감지
```

**소켓에서 Keep-Alive 활성화:**
```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Keep-Alive 활성화
sock.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)

# (Linux 전용) 세부 설정
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPIDLE, 600)   # 유휴 시간
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPINTVL, 60)   # probe 간격
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPCNT, 3)      # probe 횟수
```

### 4.3 애플리케이션 레벨 Heartbeat (권장)

TCP Keep-Alive는 너무 느리다(기본 2시간). **빠른 감지가 필요하면 직접 구현**해야 한다.

```python
# WebSocket 스타일 Ping/Pong Heartbeat
import asyncio
import json

class Connection:
    def __init__(self, reader, writer):
        self.reader = reader
        self.writer = writer
        self.last_pong = time.time()
        self.alive = True
    
    async def heartbeat_sender(self):
        """30초마다 PING 전송"""
        while self.alive:
            await asyncio.sleep(30)
            ping_msg = json.dumps({"type": "PING", "ts": time.time()})
            self.writer.write(ping_msg.encode() + b'\n')
            await self.writer.drain()
    
    async def heartbeat_checker(self):
        """PONG 응답 확인 (90초 이상 무응답 시 연결 종료)"""
        while self.alive:
            await asyncio.sleep(10)
            if time.time() - self.last_pong > 90:
                print("⚠️ 좀비 커넥션 감지! 연결 종료")
                self.alive = False
                self.writer.close()
                break
    
    async def handle_message(self, msg):
        data = json.loads(msg)
        if data.get("type") == "PONG":
            self.last_pong = time.time()
        # ... 다른 메시지 처리
```

---

## 5. 버퍼(Buffer)와 흐름 제어

### 5.1 Recv-Q와 Send-Q 이해하기

```bash
$ ss -tn
State   Recv-Q   Send-Q   Local Address:Port    Peer Address:Port
ESTAB   0        0        10.0.0.1:8080         10.0.0.2:54321    # 정상
ESTAB   152400   0        10.0.0.1:8080         10.0.0.2:54322    # ⚠️ 문제!
        ^
        └── 커널이 받아서 애플리케이션에 전달하지 못한 데이터
```

**Recv-Q가 쌓이는 원인:**
1. **애플리케이션이 `recv()` 호출을 안 함** → 블로킹 I/O에서 다른 작업 중
2. **처리 속도 < 수신 속도** → 서버 과부하

### 5.2 I/O 모델과 Worker 분리

**문제 상황:**
```python
# ❌ Bad: I/O와 비즈니스 로직이 같은 스레드
def handle_client(conn):
    while True:
        data = conn.recv(1024)  # 여기서 블로킹
        result = heavy_computation(data)  # CPU 작업 동안 recv() 못 함!
        conn.send(result)
```

**해결책: I/O Worker와 Business Worker 분리**
```python
# ✅ Good: 비동기 I/O + Worker Pool
import asyncio
from concurrent.futures import ProcessPoolExecutor

executor = ProcessPoolExecutor(max_workers=4)  # CPU 작업용 프로세스 풀

async def handle_client(reader, writer):
    while True:
        data = await reader.read(1024)  # 논블로킹 I/O
        
        # CPU 작업은 별도 프로세스에서 실행
        loop = asyncio.get_event_loop()
        result = await loop.run_in_executor(executor, heavy_computation, data)
        
        writer.write(result)
        await writer.drain()  # 버퍼 비우기
```

### 5.3 버퍼 크기 튜닝

```bash
# TCP 버퍼 크기 조정 (min, default, max 순서)

# 읽기 버퍼 (Recv-Q 최대 크기에 영향)
net.ipv4.tcp_rmem = 4096 131072 6291456
#                   4KB  128KB  6MB

# 쓰기 버퍼 (Send-Q 최대 크기에 영향)
net.ipv4.tcp_wmem = 4096 16384 4194304
#                   4KB  16KB  4MB

# 자동 튜닝 활성화 (기본 ON)
net.ipv4.tcp_moderate_rcvbuf = 1
```

---

## 6. 트러블슈팅 체크리스트

### 6.1 "연결이 안 돼요!" 진단 순서

```bash
# 1️⃣ TIME_WAIT 확인
$ ss -s
# TIME-WAIT이 너무 많으면 → tcp_tw_reuse 설정

# 2️⃣ Accept Queue 확인
$ ss -ltn | awk '$2 > $3 * 0.7 {print "⚠️ Queue 위험:", $0}'
# Recv-Q가 높으면 → backlog 증가, 애플리케이션 accept() 속도 확인

# 3️⃣ 파일 디스크립터 확인
$ cat /proc/sys/fs/file-nr
# 현재 열린 FD / 할당된 FD / 최대 FD

# 4️⃣ 커넥션 수 확인
$ ss -s
# total, TCP, UDP 등 전체 현황
```

### 6.2 "연결은 됐는데 느려요!" 진단 순서

```bash
# 1️⃣ Recv-Q/Send-Q 확인
$ ss -tn | awk '$2 > 0 {print "Recv-Q 쌓임:", $0}'
$ ss -tn | awk '$3 > 0 {print "Send-Q 쌓임:", $0}'

# 2️⃣ 재전송 확인
$ netstat -s | grep -i retrans
# 재전송이 많으면 → 네트워크 문제 또는 수신 측 처리 지연

# 3️⃣ 혼잡 제어 알고리즘 확인
$ sysctl net.ipv4.tcp_congestion_control
# BBR 사용 권장: net.ipv4.tcp_congestion_control = bbr
```

### 6.3 실시간 모니터링 명령어

```bash
# TCP 연결 상태별 개수 실시간 확인
$ watch -n 1 'ss -s'

# TIME_WAIT 실시간 카운트
$ watch -n 1 'ss -ant | awk "/TIME-WAIT/{c++} END{print c}"'

# 특정 포트의 연결 상태 확인
$ watch -n 1 'ss -tn state established "( dport = :443 or sport = :443 )"'
```

---

## 7. 커널 파라미터 종합 설정 템플릿

```bash
# /etc/sysctl.d/99-tcp-production.conf
# 프로덕션 서버용 TCP 튜닝 설정

#==========================================
# TIME_WAIT 및 포트 관리
#==========================================
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10000 65535
net.ipv4.tcp_fin_timeout = 30

#==========================================
# 연결 큐 (SYN Queue, Accept Queue)
#==========================================
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_syncookies = 1

#==========================================
# Keep-Alive 설정
#==========================================
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 60
net.ipv4.tcp_keepalive_probes = 3

#==========================================
# 버퍼 크기 (min, default, max)
#==========================================
net.ipv4.tcp_rmem = 4096 131072 6291456
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216

#==========================================
# 혼잡 제어 (BBR 사용 권장)
#==========================================
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

#==========================================
# 파일 디스크립터 및 리소스
#==========================================
fs.file-max = 2097152
fs.nr_open = 2097152
```

```bash
# 적용 방법
$ sudo sysctl -p /etc/sysctl.d/99-tcp-production.conf

# 영구 적용 확인
$ sysctl net.ipv4.tcp_tw_reuse
net.ipv4.tcp_tw_reuse = 1
```

---

## 8. 정리: 상황별 해결 가이드

| 증상 | 원인 | 해결책 |
|------|------|--------|
| `Cannot assign requested address` | 포트 고갈 (TIME_WAIT 폭주) | `tcp_tw_reuse=1` + 커넥션 풀 |
| `Connection refused` 폭주 | Accept Queue Overflow | `somaxconn` + `backlog` 증가 |
| 연결 후 응답 없음 | 좀비 커넥션 (Half-open) | Keep-Alive + 애플리케이션 Heartbeat |
| `Recv-Q` 계속 증가 | 애플리케이션 처리 지연 | I/O/Worker 분리, 비동기 처리 |
| `Too many open files` | FD 제한 | `ulimit -n` + `fs.file-max` 증가 |
| 재전송 많음 | 네트워크 혼잡 또는 상대방 지연 | BBR 혼잡 제어, 버퍼 크기 조정 |

---

*"TCP 문제의 80%는 TIME_WAIT, 큐 오버플로우, 좀비 커넥션 3가지로 귀결된다. 이 3가지만 제대로 이해하면 대부분의 TCP 트러블슈팅이 가능하다."*

> **다음 학습 추천:**
> - `tcp-vs-udp/README.md`: TCP 소켓 프로그래밍의 실무 패턴 (Framing, Heartbeat)
> - `../01-os/process-vs-thread/README.md`: File Descriptor와 동시 접속 처리
