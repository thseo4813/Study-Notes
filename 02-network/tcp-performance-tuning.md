# 🔧 TCP 성능 튜닝 & 트러블슈팅

> **이 문서의 목표:** TCP 연결 과정에서 발생하는 병목 현상(TIME_WAIT, 큐 오버플로우 등)의 원인을 커널 레벨에서 이해하고, 리눅스 커널 파라미터 튜닝을 통해 **안정적인 고성능 서버**를 구축한다.

---

## 0. 핵심 질문으로 시작하기

1. **TIME_WAIT 상태는 왜 발생하며, 왜 위험한가?** → 연결 종료 후 패킷 혼선을 막기 위한 대기 상태지만, 너무 많으면 포트 고갈을 유발함
2. **Connection Refused는 언제 발생하는가?** → 서버 포트가 닫혀 있거나, SYN/Accept 큐가 가득 찼을 때
3. **Keep-Alive 설정은 왜 중요한가?** → 비정상적으로 끊긴 연결(좀비 커넥션)을 감지하고 리소스를 회수하기 위해
4. **Recv-Q가 쌓인다는 것은 무엇을 의미하는가?** → 애플리케이션의 처리 속도가 네트워크 수신 속도를 따라가지 못함

---

## 1. ⚠️ 먼저 확인할 전제 (안전/환경)

이 문서의 커널 파라미터 예시는 대부분 **Linux 기준**입니다.

- **Windows**: `/proc`, `sysctl`, `ss`는 그대로 적용되지 않습니다. (Windows는 `netsh`, 성능 모니터, ETW 등으로 접근)
- **튜닝은 “증상-원인”이 맞을 때만**: 무작정 값을 올리면 메모리/FD/큐가 늘면서 **장애 형태만 바뀌는** 경우가 흔합니다.
- **`tcp_tw_reuse` 주의**: 일반적으로 **클라이언트(아웃바운드 연결을 많이 여는 쪽)**의 포트 고갈 완화에 도움이 됩니다. 서버(리스닝 소켓) 문제를 이것만으로 해결할 수 있다고 가정하면 오해가 생길 수 있습니다.

---

## 2. TIME_WAIT와 포트 고갈 문제

### 2.1 TIME_WAIT란?

TCP 연결을 **먼저 종료하는 쪽**이 반드시 거치는 상태다.  
약 **60초** 동안 유지된다.

```
클라이언트                              서버
    │                                    │
    │─── FIN (나 끊을게) ──────────────>│
    │                                    │
    │<── ACK (알겠어) ──────────────────│
    │<── FIN (나도 끊을게) ─────────────│
    │                                    │
    │─── ACK (알겠어) ──────────────────>│
    │                                    │
    │  ⏱️ TIME_WAIT (60초 대기)         │
    │                                    │
```

> **🤔 왜 60초나 기다릴까?**
> 1. 네트워크에 떠도는 이전 패킷이 새 연결에 섞이는 것 방지
> 2. 마지막 ACK 손실 시 상대방의 FIN 재전송에 응답하기 위해

### 2.2 🚨 포트 고갈 시나리오

```bash
# 문제 확인: TIME_WAIT가 5만개 이상이면 위험!
$ netstat -an | grep TIME_WAIT | wc -l
58234  # ⚠️ 위험 수준

# 사용 가능한 포트 확인
$ cat /proc/sys/net/ipv4/ip_local_port_range
32768   60999  # 약 28,000개뿐
```

**계산해보면:**
- 초당 1,000개 요청 × TIME_WAIT 60초 = **60,000개 포트 필요**
- 가용 포트 28,000개 → ❌ **포트 고갈!**

### 2.3 ✅ 해결책: 커널 파라미터 튜닝

```bash
# /etc/sysctl.d/99-tcp-tuning.conf

# ⭐ 가장 중요! TIME_WAIT 소켓 재사용
net.ipv4.tcp_tw_reuse = 1

# 포트 범위 확대 (28,000개 → 55,000개)
net.ipv4.ip_local_port_range = 10000 65535

# FIN_WAIT 타임아웃 단축 (60초 → 30초)
net.ipv4.tcp_fin_timeout = 30
```

```bash
# 적용
$ sudo sysctl -p
```

> ⚠️ **절대 사용 금지:** `tcp_tw_recycle = 1`  
> NAT 환경에서 연결 실패를 유발하며, Linux 4.12 이후 제거됨

### 2.4 ✅ 근본 해결책: 커넥션 풀

커널 튜닝은 **임시방편**이다. 근본 해결은 **연결 재사용**이다.

```java
// ❌ 나쁜 예: 매번 새 연결
conn.disconnect();  // → TIME_WAIT 발생!

// ✅ 좋은 예: 커넥션 풀 사용
CloseableHttpClient client = HttpClients.custom()
    .setMaxConnTotal(200)           // 전체 최대 연결
    .setMaxConnPerRoute(50)         // 호스트당 최대 연결
    .build();
// → 연결이 풀에 반환됨, TIME_WAIT 없음!
```

---

## 3. SYN Queue와 Accept Queue 관리

### 3.1 💡 두 개의 큐 이해하기

클라이언트가 `connect()`를 호출하면, 서버에서 **두 단계**를 거친다:

```
클라이언트                                    서버
    │                                          │
    │─── SYN ────────────> [ SYN Queue ]       │  ← Handshake 진행 중
    │                           │              │
    │<── SYN+ACK ───────────────┘              │
    │                                          │
    │─── ACK ────────────> [ Accept Queue ]    │  ← 연결 완료, accept() 대기
    │                           │              │
    │                     accept() 호출        │  ← 애플리케이션이 꺼내감
```

### 3.2 🚨 큐가 가득 차면?

| 큐 | 증상 | 원인 |
|----|------|------|
| **SYN Queue 가득** | 새 SYN 패킷 드롭 | SYN Flood 공격 |
| **Accept Queue 가득** | `Connection Reset` | accept()가 느림 |

```bash
# 현재 큐 상태 확인
$ ss -ltn
State   Recv-Q  Send-Q  Local:Port
LISTEN  0       128     0.0.0.0:80
        ↑       ↑
        │       └── backlog 설정값 (최대 크기)
        └── 현재 대기 중인 연결 수

# ⚠️ Recv-Q가 Send-Q의 80% 이상이면 위험!
```

### 3.3 ✅ 해결책: 큐 크기 튜닝

```bash
# /etc/sysctl.conf

# Accept Queue 최대값
net.core.somaxconn = 65535

# SYN Queue 최대값
net.ipv4.tcp_max_syn_backlog = 65535

# SYN Flood 방어
net.ipv4.tcp_syncookies = 1
```

**애플리케이션에서도 설정 필요:**

```python
# Python
server.listen(65535)  # 기본값 5는 너무 작음!
```

```nginx
# Nginx
listen 80 backlog=65535;
```

---

## 4. Keep-Alive와 좀비 커넥션

### 4.1 💡 좀비 커넥션(Half-open)이란?

클라이언트가 갑자기 죽으면, 서버는 **연결이 끊긴 줄 모른다**.

```
정상:     Client ←─── ESTABLISHED ───→ Server

문제 발생: Client (죽음) ←───?───→ Server (모름)
                                     └── 좀비 커넥션!
                                         메모리, FD 낭비
```

### 4.2 ✅ 해결책 1: TCP Keep-Alive (OS 레벨)

```bash
# 기본 2시간은 너무 느림! → 10분으로 단축
net.ipv4.tcp_keepalive_time = 600     # 600초 유휴 후 probe 시작
net.ipv4.tcp_keepalive_intvl = 60     # 60초 간격
net.ipv4.tcp_keepalive_probes = 3     # 3번 실패 시 종료

# 결과: 최대 780초(13분) 후 감지
```

```python
# 소켓에서 활성화
sock.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)
```

### 4.3 ✅ 해결책 2: 애플리케이션 Heartbeat (권장)

TCP Keep-Alive는 **너무 느리다**. 빠른 감지가 필요하면 **직접 구현**하라.

```python
class Connection:
    async def heartbeat_sender(self):
        """Send PING every 30 seconds."""
        while self.alive:
            await asyncio.sleep(30)
            self.send({"type": "PING"})
    
    async def heartbeat_checker(self):
        """Close if no PONG for 90 seconds."""
        while self.alive:
            await asyncio.sleep(10)
            if time.time() - self.last_pong > 90:
                print("⚠️ 좀비 커넥션 감지!")
                self.close()
```

---

## 5. 버퍼(Recv-Q/Send-Q) 관리

### 5.1 💡 Recv-Q가 쌓이는 이유

```bash
$ ss -tn
State   Recv-Q   Send-Q   Local:Port
ESTAB   0        0        10.0.0.1:8080    # ✅ 정상
ESTAB   152400   0        10.0.0.1:8080    # ⚠️ 문제!
        ↑
        └── 애플리케이션이 데이터를 안 읽고 있음
```

**원인:**
1. `recv()` 호출 안 함 → 다른 작업 중
2. 처리 속도 < 수신 속도 → 서버 과부하

### 5.2 ✅ 해결책: I/O와 Worker 분리

```python
# ❌ 나쁜 예: I/O와 비즈니스 로직이 같은 스레드
def handle(conn):
    data = conn.recv(1024)     # 블로킹
    heavy_computation(data)    # 이 동안 recv() 못 함!
```

```python
# ✅ 좋은 예: 비동기 I/O + Worker Pool
async def handle(reader, writer):
    data = await reader.read(1024)  # 논블로킹
    
    # CPU 작업은 별도 프로세스에서
    result = await loop.run_in_executor(executor, heavy_work, data)
    
    writer.write(result)
```

---

## 6. 트러블슈팅 체크리스트

### 6.1 🔍 "연결이 안 돼요!" 진단

```bash
# 1. TIME_WAIT 확인
$ ss -s | grep TIME-WAIT

# 2. Accept Queue 확인 (Recv-Q가 높으면 위험)
$ ss -ltn

# 3. 파일 디스크립터 확인
$ cat /proc/sys/fs/file-nr

# 4. 전체 연결 현황
$ ss -s
```

### 6.2 🔍 "연결은 됐는데 느려요!" 진단

```bash
# 1. 버퍼 쌓임 확인
$ ss -tn | awk '$2 > 0 {print "⚠️ Recv-Q:", $0}'

# 2. 재전송 확인 (많으면 네트워크 문제)
$ netstat -s | grep -i retrans

# 3. 혼잡 제어 알고리즘 (BBR 권장)
$ sysctl net.ipv4.tcp_congestion_control
```

---

## 7. 프로덕션 설정 템플릿

```bash
# /etc/sysctl.d/99-tcp-production.conf

#=== TIME_WAIT 관리 ===
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10000 65535
net.ipv4.tcp_fin_timeout = 30

#=== 연결 큐 ===
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_syncookies = 1

#=== Keep-Alive ===
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 60
net.ipv4.tcp_keepalive_probes = 3

#=== 버퍼 ===
net.ipv4.tcp_rmem = 4096 131072 6291456
net.ipv4.tcp_wmem = 4096 16384 4194304

#=== 혼잡 제어 (BBR) ===
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

#=== 파일 디스크립터 ===
fs.file-max = 2097152
```

---

## 8. 🎯 1분 요약

**TCP 문제 = 연결 수명 주기 이해 + 커널 파라미터 튜닝**

- **포트 고갈** → `tcp_tw_reuse=1` + 커넥션 풀
- **연결 거절** → `somaxconn` + `backlog` 증가
- **좀비 연결** → Keep-Alive + 애플리케이션 Heartbeat

> 💬 **기억할 것**  
> TCP 문제의 80%는 **TIME_WAIT, 큐 오버플로우, 좀비 커넥션** 3가지로 귀결된다.

---

## 9. 📝 자가 점검 질문

1. **TIME_WAIT 상태는 어떤 상황에서 발생하는가?**
   → TCP 연결 종료 시, 먼저 `close()`를 호출한(Active Close) 쪽에서 발생한다.
2. **`tcp_tw_reuse` 옵션이 하는 역할은?**
   → TIME_WAIT 상태의 소켓을 새로운 연결(Outbound)에 재사용할 수 있게 하여 포트 고갈을 방지한다.
3. **Accept Queue가 가득 찼을 때 클라이언트는 어떤 에러를 겪는가?**
   → `Connection refused` 또는 타임아웃.
4. **애플리케이션 레벨의 Heartbeat가 TCP Keep-Alive보다 선호되는 이유는?**
   → TCP Keep-Alive는 기본 주기가 길고(2시간), 애플리케이션 행(Hang) 상태를 감지하지 못할 수 있기 때문.
