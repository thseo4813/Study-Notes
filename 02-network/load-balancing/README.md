# ⚖️ 로드 밸런싱: 트래픽을 어떻게 분배할까?

## 🚨 실제로 겪어본 로드밸런싱 문제들

### 서비스 확장 시 흔히 마주치는 고민:

**"서버 한 대로는 트래픽을 못 감당해!"**
- 동시에 1000명 이상 접속 시 서버가 죽음
- 이벤트 기간에 트래픽 폭증으로 서비스 다운
- 데이터베이스 연결이 한계에 도달

**"로드 밸런서 설정이 잘못됐어"**
- 특정 서버에만 트래픽 몰림 (Sticky session 문제)
- 헬스체크 실패로 서버가 죽었는데도 요청 감
- SSL 터미네이션 설정으로 성능 저하

**"어떤 로드 밸런싱 알고리즘이 좋을까?"**
- Round Robin vs Least Connections vs IP Hash
- 세션 유지(Session Affinity) 때문에 골치
- 캐시 무효화 시 모든 서버에 적용하기 어려움

## 🎯 1분 요약: 로드 밸런싱의 핵심

**로드 밸런서 = 스마트한 트래픽 분배기**

- **역할**: 들어오는 요청을 여러 서버로 골고루 분배
- **종류**: L4 (IP/포트 기반) vs L7 (콘텐츠 기반)
- **도구**: Nginx, HAProxy, AWS ALB/ELB

> **결론:**
> 1. **확장성**: 서버 대수만큼 용량 증가
> 2. **가용성**: 서버 하나 죽어도 서비스 유지
> 3. **성능**: 캐싱, 압축으로 응답 속도 개선
> 
> 

---

## 2. 프록시(Proxy)의 두 얼굴: Forward vs Reverse

'프록시'라는 용어는 맥락에 따라 정반대의 의미로 쓰이므로 명확히 구분해야 한다.

| 구분 | 포워드 프록시 (Forward Proxy) | 리버스 프록시 (Reverse Proxy) |
| --- | --- | --- |
| **위치** | **클라이언트(User)** 앞단 | **서버(Server)** 앞단 |
| **목적** | 검열 우회(VPN), 캐싱, 사내 보안 | **로드 밸런싱**, 보안, SSL 처리 |
| **누가 숨나?** | **사용자**가 숨는다. (서버는 누가 요청했는지 모름) | **서버**가 숨는다. (사용자는 최종 서버 IP를 모름) |
| **예시** | 사내망 인터넷 제한, 개인용 VPN | Nginx, AWS ELB, HAProxy |

> **실무:** 백엔드 개발자가 다루는 것은 99% **리버스 프록시**다.

---

## 3. 로드 밸런싱 계층: L4 vs L7

OSI 7계층 중 어디서 데이터를 나누느냐에 따라 성능과 기능이 달라진다.

### 3.1 실제 적용 예시

**💡 선택 기준:**

| 상황 | L4 선택 | L7 선택 |
|------|---------|---------|
| **단순 부하 분산** | 게임 서버 (TCP/UDP) | 웹 애플리케이션 (HTTP) |
| **높은 성능 요구** | 비디오 스트리밍 | API 게이트웨이 |
| **콘텐츠 기반 라우팅** | X | 마이크로서비스 |

**🚨 실제 문제 사례:**

**문제 1: 세션 유지가 안 되는 Sticky Session 문제**
```nginx
# ❌ 잘못된 설정: 모든 요청이 다른 서버로 감
upstream backend {
    server app1.example.com;
    server app2.example.com;
    server app3.example.com;
}

# ✅ 해결: IP 해시로 같은 사용자 같은 서버로
upstream backend {
    ip_hash;  # 같은 IP는 같은 서버로
    server app1.example.com;
    server app2.example.com;
    server app3.example.com;
}
```

**문제 2: 헬스체크 실패로 인한 다운타임**
```nginx
# ❌ 헬스체크 없음: 죽은 서버에도 요청 감
upstream backend {
    server app1.example.com;
    server app2.example.com down;  # 수동으로만 설정
}

# ✅ 자동 헬스체크
upstream backend {
    server app1.example.com;
    server app2.example.com;
    server app3.example.com;

    # 5초마다 /health 엔드포인트 체크
    health_check interval=5s uri=/health;
}
```

---

## 4. 아키텍처 다이어그램 (Reverse Proxy Flow)

Nginx가 중간에서 요청을 받아 뒤쪽의 애플리케이션 서버들(Upstream)로 뿌려주는 구조다.

```mermaid
graph LR
    User[👤 Client] -- "1. Request (https://api.com)" --> Nginx[🚦 Nginx (Reverse Proxy)]
    
    subgraph "Internal Network (Private Subnet)"
        Nginx -- "2. Round Robin" --> App1[WAS 1 (Node/Java)]
        Nginx -- "2. Round Robin" --> App2[WAS 2 (Node/Java)]
        Nginx -- "2. Round Robin" --> App3[WAS 3 (Node/Java)]
    end
    
    App1 -- "3. Response" --> Nginx
    Nginx -- "4. Response" --> User
    
    style Nginx fill:#ffecb3,stroke:#ff8f00,stroke-width:2px
    style App1 fill:#e1f5fe,stroke:#0277bd
    style App2 fill:#e1f5fe,stroke:#0277bd
    style App3 fill:#e1f5fe,stroke:#0277bd

```

---

## 5. Production-Ready Code Example (Nginx)

실무에서 바로 사용할 수 있는 Nginx 설정 파일(`nginx.conf`) 예시다. 단순 전달뿐만 아니라 **헤더 포워딩**이 핵심이다.

```nginx
# nginx.conf

http {
    # 1. 업스트림 정의 (부하 분산 대상 서버들)
    upstream my_backend_app {
        # 로드 밸런싱 알고리즘 (기본값: Round Robin)
        # least_conn; # 연결 적은 놈한테 우선 보냄
        # ip_hash;    # 같은 IP는 계속 같은 서버로 보냄 (Sticky Session)
        
        server 10.0.0.1:8080 weight=3; # 가중치 3 (더 많이 받음)
        server 10.0.0.2:8080;
        server 10.0.0.3:8080 backup;   # 다른 애들 다 죽으면 투입
    }

    server {
        listen 80;
        server_name api.example.com;

        location / {
            # 2. 리버스 프록시 설정
            proxy_pass http://my_backend_app;

            # [중요] 헤더 포워딩 (Client IP 보존)
            # 이걸 안 하면 WAS는 모든 요청이 Nginx IP(로컬)에서 온 줄 안다.
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            
            # 타임아웃 설정 (504 Gateway Time-out 방지)
            proxy_connect_timeout 60s;
            proxy_read_timeout 60s;
        }
    }
}

```

---

### 6.1 클라이언트 진짜 IP 찾기 (`X-Forwarded-For`)

리버스 프록시 뒤에 있는 서버(Spring, Node.js 등)에서 `request.getRemoteAddr()`를 찍으면 사용자가 아닌 **Nginx의 내부 IP**가 나온다.

* **해결:** 반드시 `X-Forwarded-For` 헤더를 파싱해서 원본 IP를 가져와야 한다. (로그 분석이나 어뷰징 차단 시 필수)

### 6.2 SSL Termination (SSL 종료)

HTTPS 암호화/복호화는 CPU를 많이 먹는 작업이다.

* **전략:** Nginx(로드 밸런서)까지만 HTTPS로 받고, **Nginx  내부 서버** 구간은 HTTP(평문)로 통신한다.
* **이점:** 비싼 암호화 작업을 Nginx가 전담하고, WAS는 비즈니스 로직에만 집중하여 전체 성능을 높인다.

### 6.3 헬스 체크 (Health Check)

로드 밸런서의 가장 중요한 덕목은 **"아픈 놈에겐 일을 주지 않는다"**는 것이다.

* Nginx나 AWS ELB는 주기적으로 서버에 핑(Ping)을 보내거나 특정 API(`/health`)를 찔러본다. 응답이 없거나 200 OK가 아니면 해당 서버를 트래픽 목록에서 즉시 제외(Circuit Breaker)한다.

---

* **내용:** Replication(복제) vs Sharding(샤딩), CAP 이론(일관성 vs 가용성).
