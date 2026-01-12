# 웹의 교통 경찰: 로드 밸런싱(Load Balancing)과 Nginx

## 1. 핵심 요약 (Executive Summary)

서비스가 커지면 서버 한 대(Single Instance)로는 트래픽을 감당할 수 없다. 여러 대의 서버를 두고 트래픽을 분산시켜야 하는데, 이 역할을 하는 것이 **로드 밸런서(Load Balancer)**다. 실무에서는 **Nginx**를 사용하여 **리버스 프록시(Reverse Proxy)** 형태로 구현하는 것이 표준이다.

> **결론:**
> 1. **확장성(Scalability):** 서버를 수평으로 계속 늘릴 수 있다(Scale-out).
> 2. **고가용성(High Availability):** 서버 하나가 터져도 로드 밸런서가 알아서 살아있는 다른 서버로 요청을 보낸다. (무중단 서비스)
> 3. **보안:** 실제 서버의 IP를 숨기고, 앞단에서 공격(DDoS 등)을 막아낸다.
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

### 3.1 비교표

| 구분 | L4 로드 밸런서 (Transport Layer) | L7 로드 밸런서 (Application Layer) |
| --- | --- | --- |
| **기준** | **IP 주소 + 포트 번호** (단순 패킷 레벨) | **URL, HTTP 헤더, 쿠키** (콘텐츠 레벨) |
| **특징** | 패킷 내용을 안 보고 토스만 함. **빠르고 단순함.** | 내용을 뜯어보고 판단함. **똑똑하지만 부하가 있음.** |
| **기능** | 단순 부하 분산 | `/img`는 이미지 서버로, `/api`는 API 서버로 분기 가능. |
| **대표 도구** | AWS Network Load Balancer (NLB) | AWS Application Load Balancer (ALB), Nginx |

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
