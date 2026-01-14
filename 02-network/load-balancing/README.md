# ⚖️ 로드 밸런싱 완벽 이해: 트래픽 분배의 과학

> **이 문서의 목표:** 로드 밸런싱을 단순히 "트래픽 분배"로 알지 말고, **L4/L7의 근본적 차이**, **알고리즘 선택 기준**, **실무 문제 해결**을 이해한다.

---

## 0. 핵심 질문으로 시작하기

1. **왜 로드 밸런싱이 필요한가?** → 단일 서버의 한계
2. **L4와 L7의 차이는 무엇인가?** → 어느 계층에서 결정하느냐
3. **어떤 알고리즘을 선택해야 하는가?** → 상황별 트레이드오프
4. **세션 유지는 어떻게 하는가?** → Sticky Session의 원리

---

## 1. 왜 로드 밸런싱이 필요한가?

### 1.1 단일 서버의 한계

```
[단일 서버 구조]
모든 요청 → 서버 1대

문제점:
1. 용량 한계: 서버 1대가 처리할 수 있는 요청 수 제한
2. 단일 장애점: 서버 다운 = 서비스 전체 중단
3. 업그레이드 어려움: 스케일 업만 가능 (더 비싼 서버)
```

### 1.2 로드 밸런싱의 해결책

```
[로드 밸런싱 구조]
                    ┌─→ 서버 1
요청 → 로드밸런서 ─┼─→ 서버 2
                    └─→ 서버 3

해결되는 문제:
1. 확장성: 서버 추가로 용량 증가 (스케일 아웃)
2. 가용성: 서버 1대 죽어도 나머지가 처리
3. 유연성: 서버 무중단 교체 가능
```

### 1.3 로드 밸런서의 핵심 역할

```
[세 가지 핵심 기능]

1. 트래픽 분배 (Distribution)
   → 요청을 여러 서버에 나눠서 전달
   
2. 헬스 체크 (Health Check)
   → 서버 상태 감시, 죽은 서버 제외
   
3. 세션 관리 (Session Management)
   → 같은 사용자는 같은 서버로 (필요시)
```

---

## 2. L4 vs L7 로드 밸런싱: 근본적 차이

### 2.1 어느 계층에서 결정하느냐가 핵심

```
[L4 로드 밸런싱]
OSI 4계층(Transport)에서 결정
→ IP 주소 + 포트 번호만 봄
→ 패킷 내용(HTTP 헤더, URL 등)을 모름

[L7 로드 밸런싱]
OSI 7계층(Application)에서 결정
→ HTTP 헤더, URL, 쿠키 등을 분석
→ 내용 기반 지능적 라우팅 가능
```

### 2.2 동작 방식 비교

```
[L4 동작 방식]

Client → L4 LB → Server
         │
         └─ 패킷 헤더만 봄:
            - Source IP: 192.168.1.100
            - Dest IP: 10.0.0.1
            - Dest Port: 80
            → 서버 선택 후 패킷 그대로 전달

특징:
- 매우 빠름 (패킷 내용 분석 안 함)
- TCP 연결을 그대로 통과(Pass-through)
- SSL/TLS 암호화된 내용 볼 수 없음


[L7 동작 방식]

Client → L7 LB → Server
         │
         ├─ TCP 연결 종료
         ├─ HTTP 요청 파싱:
         │  - GET /api/users HTTP/1.1
         │  - Host: api.example.com
         │  - Cookie: session=abc123
         │
         └─ 새 TCP 연결로 서버에 전달

특징:
- 느림 (패킷 복호화 + 파싱)
- TCP 연결이 두 개 (Client-LB, LB-Server)
- 내용 기반 라우팅 가능
```

### 2.3 비유로 이해하기

```
[L4: 택배 분류 센터]
- 택배 상자의 "주소"만 봄
- 안에 뭐가 들었는지 모름
- 빠르게 분류

[L7: 고객 서비스 센터]
- 편지를 열어서 "내용"을 읽음
- 내용에 따라 담당 부서로 전달
- 느리지만 정확한 처리
```

### 2.4 L4 vs L7 상세 비교

| 측면 | L4 | L7 |
|-----|-----|-----|
| **보는 정보** | IP, Port | HTTP 헤더, URL, 쿠키 |
| **성능** | 매우 빠름 | 상대적으로 느림 |
| **SSL 처리** | 불가 (Pass-through) | SSL 종료(Termination) 가능 |
| **라우팅** | 단순 분배 | URL 기반 라우팅 가능 |
| **세션 유지** | IP 해시 방식 | 쿠키 기반 가능 |
| **가격** | 저렴 | 비쌈 |

---

## 3. L4/L7 선택 기준: 언제 무엇을 쓰는가?

### 3.1 L4를 쓰는 경우

```
[L4 적합한 상황]

1. 단순 TCP/UDP 서비스
   - 게임 서버 (UDP)
   - 데이터베이스 연결 분배
   - 메일 서버 (SMTP)

2. 성능이 최우선
   - 초당 수백만 연결 처리
   - 지연 시간이 중요한 서비스

3. SSL을 서버에서 처리
   - End-to-End 암호화 필요
   - 인증서를 각 서버에서 관리

[예시: 게임 서버]
UDP 기반 실시간 게임
→ 패킷 내용 분석 필요 없음
→ 지연 최소화 중요
→ L4 선택
```

### 3.2 L7을 쓰는 경우

```
[L7 적합한 상황]

1. URL 기반 라우팅
   /api/* → API 서버
   /static/* → CDN/정적 서버
   /admin/* → 관리 서버

2. 헤더 기반 분기
   Host: api.example.com → API 서버
   Host: www.example.com → Web 서버

3. SSL 오프로딩
   - 로드밸런서에서 암호화 종료
   - 내부 서버 부하 감소

4. 세션 쿠키 기반 유지
   - 쿠키 값으로 같은 서버 라우팅

[예시: 마이크로서비스]
하나의 도메인, 여러 백엔드 서비스
→ URL 기반 라우팅 필수
→ L7 선택
```

### 3.3 2-Tier 아키텍처 (현대적 표준)

```
[권장 아키텍처]

인터넷 → L4 LB → L7 LB → 서버

L4 역할:
- 입구에서 대량 트래픽 분산
- DDoS 방어 첫 번째 라인
- 빠른 헬스 체크

L7 역할:
- 내부에서 지능적 라우팅
- SSL 종료
- 세션 관리

[실제 구성 예]
AWS: NLB(L4) → ALB(L7) → EC2
GCP: TCP LB(L4) → HTTP LB(L7) → GCE
```

---

## 4. 로드 밸런싱 알고리즘: 어떤 기준으로 분배하는가?

### 4.1 알고리즘 종류와 원리

#### Round Robin (순차 분배)

```
[동작 원리]
요청 1 → 서버 1
요청 2 → 서버 2
요청 3 → 서버 3
요청 4 → 서버 1 (처음으로)
...

[장점]
- 구현 단순
- 균등한 분배

[단점]
- 서버 성능 차이 무시
- 요청 처리 시간 차이 무시

[적합한 경우]
- 서버 스펙이 동일
- 요청 처리 시간이 비슷
```

#### Weighted Round Robin (가중치 순차)

```
[동작 원리]
서버 1 (weight=3): 요청 1, 2, 3
서버 2 (weight=1): 요청 4
서버 1: 요청 5, 6, 7
서버 2: 요청 8
...

[장점]
- 서버 성능 차이 반영

[적합한 경우]
- 서버 스펙이 다름 (예: 구형/신형 혼재)
```

#### Least Connections (최소 연결)

```
[동작 원리]
현재 연결 수가 가장 적은 서버로 전달

서버 1: 10개 연결 중
서버 2: 5개 연결 중  ← 새 요청 전달
서버 3: 8개 연결 중

[장점]
- 실제 부하 반영
- 처리 시간이 긴 요청이 많을 때 효과적

[단점]
- 연결 수 추적 오버헤드

[적합한 경우]
- 요청마다 처리 시간 차이가 큼
- 장기 연결(WebSocket 등)
```

#### IP Hash (IP 해시)

```
[동작 원리]
hash(Client IP) % 서버 수 = 서버 번호

Client IP 192.168.1.100:
hash("192.168.1.100") = 12345
12345 % 3 = 0 → 서버 1

[장점]
- 같은 클라이언트는 항상 같은 서버
- 세션 유지 (Sticky Session)

[단점]
- 서버 추가/제거 시 재분배
- NAT 뒤 사용자들이 같은 서버로 몰릴 수 있음

[적합한 경우]
- 세션 상태를 서버에 저장하는 경우
- 캐시 효율을 높이고 싶은 경우
```

### 4.2 알고리즘 선택 가이드

```
[선택 기준]

1. 서버 스펙이 동일한가?
   Yes → Round Robin
   No  → Weighted Round Robin

2. 요청 처리 시간이 균일한가?
   Yes → Round Robin
   No  → Least Connections

3. 세션 유지가 필요한가?
   Yes → IP Hash 또는 Cookie 기반
   No  → Round Robin

4. 캐시 효율이 중요한가?
   Yes → Consistent Hash
```

---

## 5. 세션 유지(Session Affinity): 왜 필요하고 어떻게 하는가?

### 5.1 문제 상황

```
[세션 유지 없이]

요청 1 (로그인) → 서버 1 → 세션 저장 (session=abc)
요청 2 (대시보드) → 서버 2 → "세션 없음!" → 로그인 필요

사용자: "방금 로그인했는데 왜 또 로그인하래?"
```

### 5.2 해결 방법들

#### 방법 1: Sticky Session (서버 고정)

```
[IP Hash]
같은 IP → 항상 같은 서버

[Cookie 기반]
로드밸런서가 쿠키 삽입:
Set-Cookie: SERVERID=server1

이후 요청에서 쿠키 확인:
Cookie: SERVERID=server1 → 서버 1로
```

```nginx
# Nginx: IP Hash
upstream backend {
    ip_hash;
    server 10.0.0.1;
    server 10.0.0.2;
}

# Nginx: Cookie 기반
upstream backend {
    server 10.0.0.1;
    server 10.0.0.2;
    sticky cookie srv_id expires=1h;
}
```

#### 방법 2: 세션 공유 저장소

```
[Redis/Memcached로 세션 공유]

요청 → 서버 1 → 세션을 Redis에 저장
요청 → 서버 2 → Redis에서 세션 조회 → OK!

장점:
- 어느 서버로 가도 세션 유지
- 서버 장애 시에도 세션 유지

단점:
- Redis 의존성 추가
- 네트워크 지연
```

#### 방법 3: Stateless 설계 (JWT)

```
[JWT 토큰 사용]

로그인 → JWT 토큰 발급 (서버 상태 없음)
이후 요청 → 토큰 검증만 (어느 서버든 가능)

장점:
- 완전한 Stateless
- 확장성 최고

단점:
- 토큰 탈취 위험
- 토큰 무효화 어려움
```

### 5.3 방법 비교

| 방법 | 확장성 | 장애 대응 | 구현 복잡도 |
|-----|-------|----------|-----------|
| Sticky Session | 낮음 | 낮음 | 낮음 |
| 공유 저장소 | 높음 | 높음 | 중간 |
| JWT | 최고 | 최고 | 높음 |

---

## 6. 헬스 체크: 죽은 서버 감지

### 6.1 헬스 체크 종류

```
[Passive Health Check]
실제 트래픽 응답으로 판단
- 연속 N번 실패 → 서버 제외
- 장점: 추가 트래픽 없음
- 단점: 실패 후에야 감지

[Active Health Check]
주기적으로 상태 확인 요청
- 10초마다 /health 엔드포인트 호출
- 장점: 빠른 감지
- 단점: 추가 트래픽 발생
```

### 6.2 헬스 체크 구현

```nginx
# Nginx: Active Health Check
upstream backend {
    server 10.0.0.1;
    server 10.0.0.2;
    
    # 5초마다 /health 체크
    health_check interval=5s
                 uri=/health
                 match=healthy;
}

# 정상 응답 정의
match healthy {
    status 200;
    body ~ "OK";
}
```

```python
# 애플리케이션: 헬스 체크 엔드포인트
@app.get("/health")
def health_check():
    # DB 연결 확인
    try:
        db.execute("SELECT 1")
    except:
        return Response("DB Error", status_code=503)
    
    # Redis 연결 확인
    try:
        redis.ping()
    except:
        return Response("Cache Error", status_code=503)
    
    return Response("OK", status_code=200)
```

### 6.3 헬스 체크 설정 팁

```
[설정 권장값]

interval: 5-10초
- 너무 짧으면 불필요한 부하
- 너무 길면 장애 감지 지연

threshold: 2-3번
- 1번 실패로 제외하면 일시적 문제에도 제외됨
- 너무 많으면 장애 서버에 계속 요청

timeout: 3-5초
- 응답 대기 시간
- 너무 짧으면 느린 응답을 실패로 판단
```

---

## 7. SSL/TLS 처리: 어디서 암호화를 풀 것인가?

### 7.1 SSL Termination (종료)

```
[SSL Termination at LB]

Client ──HTTPS──→ LB ──HTTP──→ Server

장점:
- 서버 CPU 부하 감소
- 인증서 관리 일원화
- L7 라우팅 가능

단점:
- LB-Server 구간 평문
- 보안 규정 위반 가능성
```

### 7.2 SSL Pass-through (통과)

```
[SSL Pass-through]

Client ──HTTPS──→ LB ──HTTPS──→ Server

장점:
- End-to-End 암호화
- 보안 규정 준수

단점:
- L7 라우팅 불가
- 서버별 인증서 관리
```

### 7.3 SSL Re-encryption (재암호화)

```
[SSL Re-encryption]

Client ──HTTPS──→ LB ──HTTPS──→ Server
         (인증서1)      (인증서2)

장점:
- L7 라우팅 가능
- 내부도 암호화

단점:
- 가장 느림
- 인증서 두 개 관리
```

---

## 8. 실무 설정 예제

### 8.1 Nginx 기본 설정

```nginx
http {
    # 업스트림 정의
    upstream api_backend {
        least_conn;  # 최소 연결 알고리즘
        
        server 10.0.0.1:8080 weight=3;
        server 10.0.0.2:8080 weight=1;
        server 10.0.0.3:8080 backup;  # 백업 서버
    }
    
    server {
        listen 443 ssl;
        server_name api.example.com;
        
        # SSL 설정
        ssl_certificate /path/to/cert.pem;
        ssl_certificate_key /path/to/key.pem;
        
        location / {
            proxy_pass http://api_backend;
            
            # 원본 정보 전달
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # 타임아웃 설정
            proxy_connect_timeout 5s;
            proxy_read_timeout 60s;
        }
    }
}
```

### 8.2 URL 기반 라우팅

```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    # /api/* → API 서버
    location /api/ {
        proxy_pass http://api_backend;
    }
    
    # /static/* → 정적 파일 서버
    location /static/ {
        proxy_pass http://static_backend;
    }
    
    # 그 외 → Web 서버
    location / {
        proxy_pass http://web_backend;
    }
}
```

---

## 9. 자가 점검 질문

### 원리 이해

1. **L4와 L7 로드밸런서의 근본적 차이는?**
   → L4는 IP/Port만 보고 빠르게 분배, L7은 HTTP 내용을 분석해서 지능적 라우팅.

2. **Round Robin 대신 Least Connections를 쓰는 이유는?**
   → 요청 처리 시간이 다를 때, 실제 부하를 반영하기 위해.

3. **Sticky Session이 필요한 이유와 단점은?**
   → 서버에 세션을 저장할 때 필요. 단점은 서버 장애 시 세션 손실, 부하 불균형.

4. **SSL Termination의 장단점은?**
   → 장점: 서버 부하 감소, L7 라우팅 가능. 단점: 내부 구간 평문.

### 실무

5. **gRPC 서비스에서 L4 로드밸런서의 문제점은?**
   → gRPC는 하나의 TCP 연결을 유지하므로, L4는 연결 단위로만 분배. 요청이 한 서버에 몰릴 수 있음. L7 또는 클라이언트 사이드 로드밸런싱 필요.

---

**💡 핵심:** 로드 밸런싱은 단순 트래픽 분배가 아니라, **확장성**, **가용성**, **성능**의 균형을 맞추는 아키텍처 결정이다. L4/L7 선택, 알고리즘 선택, 세션 관리 방식을 상황에 맞게 조합하라.
