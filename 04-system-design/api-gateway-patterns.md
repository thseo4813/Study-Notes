# 🌐 API 통신 패턴: 프로토콜 선택과 로드밸런싱 전략

## 🤔 실제 설계 시 고민들

### API 설계 시 흔히 하는 고민:

**"REST vs gRPC, 뭘 써야 하죠?"**
- 내부 MSA 통신에 REST가 느린 것 같음
- gRPC 도입하려는데 팀에서 반대
- 모바일 앱이 데이터 사용량 민감해서 고민

**"로드밸런서 설정을 어떻게?"**
- L4 vs L7 차이를 모르겠음
- Sticky Session이 필요한데 분산이 안 됨
- WebSocket 쓰니까 로드밸런싱이 이상해짐

**"API Gateway가 병목이 되는 것 같아요"**
- 모든 트래픽이 Gateway를 통과하니 지연 발생
- Gateway 장애 시 전체 서비스 마비
- Rate Limiting, 인증을 어디서 해야 할지 모르겠음

---

## 🎯 1분 요약: API 통신의 핵심

**프로토콜 = 외부는 REST, 내부는 gRPC**
**로드밸런싱 = 단순하면 L4, 스마트하면 L7**

- **REST:** 범용성, 디버깅 용이, 브라우저 친화적
- **gRPC:** 성능, 타입 안전성, 양방향 스트리밍
- **L4 LB:** 빠르고 단순 (TCP/UDP 레벨)
- **L7 LB:** 지능적 라우팅 (HTTP 헤더 기반)

> **결론:**
> 1. **외부 API:** REST/JSON (호환성 우선)
> 2. **내부 MSA:** gRPC (성능 우선)
> 3. **LB 선택:** 요청 기반 라우팅 필요하면 L7, 아니면 L4

---

## 2. 프로토콜 선택 가이드

### 2.1 HTTP/REST vs gRPC 비교

| 측면 | HTTP/REST | gRPC |
|------|-----------|------|
| **데이터 포맷** | JSON (텍스트) | Protocol Buffers (바이너리) |
| **전송 프로토콜** | HTTP/1.1, HTTP/2 | HTTP/2 |
| **크기** | 크다 (verbose) | 작다 (10x 압축) |
| **속도** | 상대적으로 느림 | 빠름 (직렬화/역직렬화) |
| **타입 체크** | 런타임 (에러 발생 가능) | 컴파일 타임 (안전) |
| **스트리밍** | 단방향 (SSE) | 양방향 지원 |
| **브라우저 지원** | 완벽 | 제한적 (gRPC-Web 필요) |
| **디버깅** | 쉬움 (curl, Postman) | 어려움 (전용 툴 필요) |
| **도구 생태계** | 매우 풍부 | 성장 중 |

### 2.2 언제 REST를 선택하는가?

**✅ REST가 적합한 경우:**

```
[REST 적합 시나리오]

1. 외부 공개 API (Third-party 개발자용)
   - 진입 장벽 낮음 (curl로 테스트 가능)
   - 문서화 도구 풍부 (Swagger, OpenAPI)

2. 웹 브라우저가 직접 호출하는 API
   - CORS, Cookie, 인증 처리 표준화
   - 프론트엔드 개발자 친화적

3. CRUD 중심의 단순한 API
   - 리소스 기반 URL 설계 자연스러움
   - HTTP 메서드가 의미를 가짐 (GET, POST, PUT, DELETE)

4. 다양한 클라이언트 (iOS, Android, Web, IoT)
   - 모든 플랫폼에서 HTTP 클라이언트 지원
```

**REST API 예시:**

```yaml
# OpenAPI 3.0 스펙
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /users/{id}:
    get:
      summary: Get user by ID
      responses:
        200:
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: integer
                  name:
                    type: string
                  email:
                    type: string
```

### 2.3 언제 gRPC를 선택하는가?

**✅ gRPC가 적합한 경우:**

```
[gRPC 적합 시나리오]

1. MSA 내부 서비스 간 통신
   - 네트워크 오버헤드 최소화 (바이너리)
   - 서비스 간 계약(Contract) 명확 (Proto 파일)

2. 실시간/양방향 스트리밍
   - 채팅, 알림, 실시간 데이터 피드
   - 단일 연결로 양방향 통신

3. 모바일 앱-서버 통신 (데이터 절약 필요)
   - 페이로드 크기 10x 감소
   - 배터리 소모 감소 (적은 전송량)

4. 엄격한 타입 체크가 필요한 경우
   - 컴파일 타임에 API 불일치 발견
   - 대규모 팀에서 실수 방지
```

**gRPC 예시:**

```protobuf
// user.proto
syntax = "proto3";

package user;

service UserService {
  // 단일 요청-응답
  rpc GetUser(GetUserRequest) returns (User);
  
  // 서버 스트리밍 (실시간 피드)
  rpc WatchUsers(WatchRequest) returns (stream User);
  
  // 양방향 스트리밍 (채팅)
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

message GetUserRequest {
  int64 id = 1;
}

message User {
  int64 id = 1;
  string name = 2;
  string email = 3;
  int64 created_at = 4;
}
```

```go
// Go gRPC 서버 구현
type userServer struct {
    pb.UnimplementedUserServiceServer
}

func (s *userServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.User, error) {
    user, err := s.db.FindByID(req.Id)
    if err != nil {
        return nil, status.Errorf(codes.NotFound, "user not found: %v", err)
    }
    return &pb.User{
        Id:    user.ID,
        Name:  user.Name,
        Email: user.Email,
    }, nil
}

// 서버 스트리밍 예시
func (s *userServer) WatchUsers(req *pb.WatchRequest, stream pb.UserService_WatchUsersServer) error {
    for {
        select {
        case user := <-s.userUpdates:
            if err := stream.Send(user); err != nil {
                return err
            }
        case <-stream.Context().Done():
            return nil
        }
    }
}
```

### 2.4 하이브리드 접근: REST + gRPC

**실무에서 가장 많이 사용하는 패턴:**

```
[하이브리드 아키텍처]

                    ┌─────────────────────────────────┐
                    │          API Gateway            │
                    │   - REST 엔드포인트 노출        │
                    │   - 인증/인가                   │
                    │   - Rate Limiting               │
                    └─────────────────┬───────────────┘
                                      │
                         ┌────────────┼────────────┐
                         │            │            │
                         ▼            ▼            ▼
                    ┌─────────┐ ┌─────────┐ ┌─────────┐
                    │ User    │ │ Order   │ │ Product │
                    │ Service │ │ Service │ │ Service │
                    └────┬────┘ └────┬────┘ └────┬────┘
                         │           │           │
                         └─────── gRPC ──────────┘
                              (내부 통신)

외부 클라이언트 ──REST──> API Gateway ──gRPC──> 내부 서비스들
```

**gRPC-Gateway를 이용한 REST → gRPC 변환:**

```protobuf
// user.proto with HTTP annotations
syntax = "proto3";

import "google/api/annotations.proto";

service UserService {
  rpc GetUser(GetUserRequest) returns (User) {
    option (google.api.http) = {
      get: "/v1/users/{id}"  // REST 엔드포인트 자동 생성
    };
  }
  
  rpc CreateUser(CreateUserRequest) returns (User) {
    option (google.api.http) = {
      post: "/v1/users"
      body: "*"
    };
  }
}
```

---

## 3. 로드밸런싱 전략

### 3.1 L4 vs L7 로드밸런싱

```
[OSI 레이어와 로드밸런싱]

Layer 7 (Application) ─── HTTP 헤더, URL, Cookie 기반 분산
Layer 6 (Presentation)
Layer 5 (Session)
Layer 4 (Transport) ───── IP + 포트 기반 분산
Layer 3 (Network)
Layer 2 (Data Link)
Layer 1 (Physical)
```

**L4 로드밸런싱 (Transport Layer):**

```
[L4 LB 동작 방식]

클라이언트 ──TCP 연결──> L4 LB ──TCP 연결──> 서버 A
                           │
                           ├──TCP 연결──> 서버 B
                           │
                           └──TCP 연결──> 서버 C

특징:
- TCP/UDP 레벨에서 동작
- 패킷 내용을 보지 않음 (빠름)
- IP + 포트만으로 분산 결정
- NAT 또는 DSR(Direct Server Return) 방식
```

**L7 로드밸런싱 (Application Layer):**

```
[L7 LB 동작 방식]

클라이언트 ──HTTP 요청──> L7 LB ──분석후 라우팅──> 서버 A
                           │
                           │ [분석 대상]
                           │ - URL 경로 (/api/users)
                           │ - HTTP 헤더 (Authorization)
                           │ - Cookie (session_id)
                           │ - Query 파라미터
                           │
                           ├─ /api/users ──> 서버 A (User Service)
                           ├─ /api/orders ─> 서버 B (Order Service)
                           └─ /static/* ───> 서버 C (CDN)
```

### 3.2 언제 어떤 LB를 선택?

| 상황 | L4 LB | L7 LB |
|------|-------|-------|
| **단순 트래픽 분산** | ✅ 최적 | 오버킬 |
| **URL 기반 라우팅** | ❌ 불가 | ✅ 필수 |
| **SSL 종료 (Termination)** | ❌ 불가 | ✅ 가능 |
| **헤더 기반 라우팅** | ❌ 불가 | ✅ 가능 |
| **WebSocket** | ✅ 가능 | ⚠️ 설정 필요 |
| **gRPC** | ✅ 가능 | ✅ gRPC-aware 필요 |
| **성능** | 매우 빠름 | 상대적 느림 |
| **비용** | 저렴 | 비쌈 |

### 3.3 로드밸런싱 알고리즘

```nginx
# Nginx 로드밸런싱 설정 예시

# 1. Round Robin (기본값)
# 순차적으로 서버에 분배
upstream backend {
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
    server 10.0.0.3:8080;
}

# 2. Least Connections
# 연결 수가 가장 적은 서버로
upstream backend {
    least_conn;
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
}

# 3. IP Hash (Sticky Session)
# 같은 클라이언트는 같은 서버로
upstream backend {
    ip_hash;
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
}

# 4. Weighted Round Robin
# 가중치 기반 분배 (성능 좋은 서버에 더 많이)
upstream backend {
    server 10.0.0.1:8080 weight=5;  # 50%
    server 10.0.0.2:8080 weight=3;  # 30%
    server 10.0.0.3:8080 weight=2;  # 20%
}

# 5. Health Check + Failover
upstream backend {
    server 10.0.0.1:8080 max_fails=3 fail_timeout=30s;
    server 10.0.0.2:8080 max_fails=3 fail_timeout=30s;
    server 10.0.0.3:8080 backup;  # 다른 서버 다 죽으면 사용
}
```

### 3.4 Sticky Session 문제와 해결

**문제: 세션 불일치**

```
[Sticky Session이 필요한 이유]

요청 1: Client → LB → Server A (세션 생성, 장바구니 저장)
요청 2: Client → LB → Server B (세션 없음! 장바구니 비어있음)
```

**해결책 1: Sticky Session (ip_hash, cookie)**

```nginx
# 방법 1: IP 기반 (간단하지만 NAT 환경에서 편중 발생)
upstream backend {
    ip_hash;
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
}

# 방법 2: Cookie 기반 (더 정확함)
upstream backend {
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
    
    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

**⚠️ Sticky Session의 문제점:**

```
[Sticky Session의 단점]

1. 부하 불균형
   - 특정 서버에 "무거운" 사용자가 몰릴 수 있음
   - 자동 스케일링과 충돌

2. 장애 시 세션 유실
   - Server A 죽으면 → 해당 사용자들 세션 모두 날아감

3. 수평 확장 방해
   - 서버 추가해도 기존 사용자는 기존 서버에 고정
```

**해결책 2: 외부 세션 저장소 (권장)**

```
[Stateless 서버 + 외부 세션 저장소]

        ┌──────────────────────────────────┐
        │            Redis Cluster          │
        │         (세션 데이터 저장)         │
        └──────────────────────────────────┘
                         ↑
              세션 조회/저장
                         │
   ┌──────────────────────┼──────────────────────┐
   │                      │                      │
┌──┴───┐              ┌───┴──┐              ┌───┴──┐
│Server│              │Server│              │Server│
│  A   │              │  B   │              │  C   │
│(무상태)              │(무상태)              │(무상태)
└──────┘              └──────┘              └──────┘
   ↑                      ↑                      ↑
   └──────────────────────┼──────────────────────┘
                          │
                    ┌─────┴─────┐
                    │    LB     │ (Round Robin OK)
                    └───────────┘

장점:
- 어느 서버로 가도 세션 접근 가능
- 서버 추가/제거 자유로움
- 서버 장애 시에도 세션 유지
```

```java
// Spring Session + Redis 설정
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 3600)
public class SessionConfig {
    
    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory(
            new RedisStandaloneConfiguration("redis.example.com", 6379)
        );
    }
}
```

---

## 4. gRPC 로드밸런싱의 특수성

### 4.1 gRPC + L4 LB의 문제

gRPC는 HTTP/2 기반이므로 **하나의 TCP 연결로 여러 요청을 처리**한다(Multiplexing).

```
[문제: L4 LB + gRPC = 부하 불균형]

gRPC Client ══════════════════════════════> Server A
              (단일 TCP 연결, 모든 요청)
                                           Server B (유휴)
                                           Server C (유휴)

L4 LB는 "연결" 단위로 분산 → 모든 요청이 한 서버로!
```

### 4.2 해결책: gRPC-aware L7 LB

```
[gRPC-aware L7 LB]

gRPC Client ────요청1────> L7 LB ────> Server A
           ────요청2────>      ────> Server B
           ────요청3────>      ────> Server C
           ────요청4────>      ────> Server A
           ...

L7 LB가 HTTP/2 스트림을 이해하고 요청 단위로 분산
```

**Envoy Proxy 설정 (gRPC용):**

```yaml
# envoy.yaml
static_resources:
  listeners:
    - name: listener_0
      address:
        socket_address:
          address: 0.0.0.0
          port_value: 8080
      filter_chains:
        - filters:
            - name: envoy.filters.network.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                codec_type: AUTO  # HTTP/2 자동 감지
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  virtual_hosts:
                    - name: backend
                      domains: ["*"]
                      routes:
                        - match:
                            prefix: "/"
                            grpc: {}  # gRPC 요청만 매칭
                          route:
                            cluster: grpc_backend
                http_filters:
                  - name: envoy.filters.http.router

  clusters:
    - name: grpc_backend
      type: STRICT_DNS
      lb_policy: ROUND_ROBIN
      http2_protocol_options: {}  # HTTP/2 강제 (gRPC용)
      load_assignment:
        cluster_name: grpc_backend
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: server1.example.com
                      port_value: 50051
              - endpoint:
                  address:
                    socket_address:
                      address: server2.example.com
                      port_value: 50051
```

### 4.3 클라이언트 사이드 로드밸런싱

서비스 메쉬(Service Mesh)나 특수 환경에서는 **클라이언트가 직접 LB 역할**을 한다.

```go
// Go gRPC 클라이언트 사이드 LB
import (
    "google.golang.org/grpc"
    "google.golang.org/grpc/balancer/roundrobin"
)

conn, err := grpc.Dial(
    "dns:///my-service.example.com:50051",  // DNS 기반 서비스 디스커버리
    grpc.WithDefaultServiceConfig(`{
        "loadBalancingPolicy": "round_robin"
    }`),
    grpc.WithInsecure(),
)
```

---

## 5. API Gateway 패턴

### 5.1 API Gateway의 역할

```
[API Gateway가 처리하는 것들]

                    ┌────────────────────────────────┐
                    │         API Gateway            │
                    │                                │
                    │  1. 라우팅 (Path → Service)    │
                    │  2. 인증/인가 (JWT 검증)       │
                    │  3. Rate Limiting (DDoS 방어)  │
                    │  4. 요청/응답 변환             │
                    │  5. 로깅 & 모니터링            │
                    │  6. 캐싱                       │
                    │  7. SSL Termination            │
                    │                                │
                    └────────────────────────────────┘
```

### 5.2 Gateway 병목 방지

**문제: 단일 Gateway = SPOF**

```
[안티패턴: 단일 Gateway]

모든 트래픽 ───> Gateway ───> 서비스들
                   ↑
              단일 장애점
              성능 병목
```

**해결책: Gateway 클러스터 + DNS LB**

```
[고가용성 Gateway 구성]

                    DNS Round Robin
                         │
           ┌─────────────┼─────────────┐
           │             │             │
       ┌───┴───┐     ┌───┴───┐     ┌───┴───┐
       │Gateway│     │Gateway│     │Gateway│
       │   1   │     │   2   │     │   3   │
       └───┬───┘     └───┬───┘     └───┬───┘
           │             │             │
           └─────────────┼─────────────┘
                         │
                    내부 서비스들
```

### 5.3 Gateway 설정 예시 (Kong)

```yaml
# kong.yml
_format_version: "2.1"

services:
  - name: user-service
    url: http://user-service:8080
    routes:
      - name: user-route
        paths:
          - /api/users
        strip_path: false
    plugins:
      - name: rate-limiting
        config:
          minute: 100
          policy: local
      - name: jwt
        config:
          secret_is_base64: false
          
  - name: order-service
    url: http://order-service:8080
    routes:
      - name: order-route
        paths:
          - /api/orders
    plugins:
      - name: rate-limiting
        config:
          minute: 50
```

---

## 6. 정리: 프로토콜 & LB 선택 가이드

### 6.1 프로토콜 선택 플로우차트

```
브라우저가 직접 호출?
  │
  ├─ Yes ──> REST/JSON
  │
  └─ No
       │
       └─ 내부 MSA 통신?
            │
            ├─ Yes ──> gRPC
            │
            └─ No
                 │
                 └─ 모바일 앱 + 데이터 절약 필요?
                      │
                      ├─ Yes ──> gRPC (또는 GraphQL)
                      │
                      └─ No ──> REST/JSON
```

### 6.2 LB 선택 플로우차트

```
URL/헤더 기반 라우팅 필요?
  │
  ├─ Yes ──> L7 LB
  │
  └─ No
       │
       └─ SSL Termination 필요?
            │
            ├─ Yes ──> L7 LB
            │
            └─ No
                 │
                 └─ gRPC 사용?
                      │
                      ├─ Yes ──> L7 LB (gRPC-aware)
                      │
                      └─ No ──> L4 LB (성능 우선)
```

### 6.3 핵심 요약표

| 상황 | 프로토콜 | LB | 세션 처리 |
|------|----------|----|----|
| **웹 + 모바일 공개 API** | REST | L7 | Redis 세션 |
| **MSA 내부 통신** | gRPC | L7 (gRPC-aware) | Stateless (JWT) |
| **실시간 채팅/게임** | gRPC Streaming 또는 WebSocket | L4 | Sticky (필요 시) |
| **파일 업로드/다운로드** | REST | L7 | Stateless |
| **IoT 디바이스** | MQTT 또는 경량 HTTP | L4 | 토큰 기반 |

---

*"API 설계는 기술 선택이 아니라 트레이드오프 관리다. 완벽한 솔루션은 없고, 상황에 맞는 최선의 선택만 있다."*

> **관련 문서:**
> - `large-scale-system/README.md`: 타임아웃과 서킷 브레이커
> - `../02-network/tcp-vs-udp/README.md`: gRPC의 TCP 기반 통신
> - `../02-network/load-balancing/README.md`: 로드밸런싱 심화