# 인터넷의 전화번호부: DNS와 도메인 작동 원리

## 1. 핵심 요약 (Executive Summary)

컴퓨터는 `google.com` 같은 문자열을 이해하지 못하며, 오직 `142.250.217.78` 같은 IP 주소만으로 통신한다. **DNS**는 인간이 읽을 수 있는 도메인 이름을 컴퓨터가 이해하는 IP 주소로 변환(**Resolving**)해 주는 거대한 분산 데이터베이스 시스템이다.

> **결론:** DNS는 단순한 1:1 매핑 테이블이 아니라, **계층적(Hierarchical)이고 분산된(Distributed)** 구조다. 따라서 도메인 변경 시 전 세계에 전파되는 데 시간(TTL)이 걸리며, 캐싱 전략이 성능에 지대한 영향을 미친다.

---

## 2. DNS의 계층 구조 (The Hierarchy)

DNS는 `.`(Root)을 최상위로 하는 역트리(Inverted Tree) 구조를 가진다. 우리가 보는 주소는 사실 뒤에서부터 해석된다.
예: `www.google.com.` (마지막의 점은 보통 생략됨)

| 계층 | 명칭 | 역할 | 예시 |
| --- | --- | --- | --- |
| **Root** | **Root DNS Server** | 전 세계에 13개(논리적)만 존재. TLD 서버의 위치를 알려줌. | `.` |
| **1단계** | **TLD (Top-Level Domain)** | 국가(.kr)나 일반(.com) 도메인을 관리하는 서버. | `.com`, `.net`, `.org` |
| **2단계** | **Authoritative (권한 있는) DNS** | 실제 도메인 소유자가 관리하는 서버. 최종 IP를 알고 있음. | AWS Route53, Gabia, GoDaddy |

---

## 3. 작동 원리: `google.com`을 찾아서

사용자가 브라우저에 주소를 입력하면 다음과 같은 **재귀적 질의(Recursive Query)**가 발생한다.

### 3.1 시퀀스 다이어그램 (The Flow)

```mermaid
sequenceDiagram
    participant User as 👤 User (Browser)
    participant OS as 💻 OS / Resolver (ISP)
    participant Root as 🌍 Root DNS (.)
    participant TLD as 🏢 TLD DNS (.com)
    participant Auth as 🔒 Auth DNS (google.com)

    User->>OS: "google.com IP가 뭐야?"
    
    Note over OS: 1. 캐시 확인 (Browser -> OS -> Router)
    
    alt 캐시에 없음
        OS->>Root: "google.com 어디로 가야 해?"
        Root->>OS: ".com 관리하는 TLD 서버 주소 줄게."
        
        OS->>TLD: "google.com IP 알려줘."
        TLD->>OS: "google.com 관리하는 Auth 서버 주소 줄게."
        
        OS->>Auth: "google.com의 진짜 IP 내놔."
        Auth->>OS: "여기 있어: 142.250.217.78 (TTL: 300s)"
    end
    
    OS->>User: IP 전달 (142.250.217.78)
    User->>User: 해당 IP로 접속 시도 (HTTP Handshake)

```

### 3.2 상세 단계

1. **로컬 캐시 확인:** 브라우저 캐시  OS 캐시(`hosts` 파일)  라우터 캐시  ISP DNS 캐시 순으로 뒤진다. (여기서 90%는 해결됨)
2. **Root DNS:** 모르면 전 세계의 뿌리인 Root 서버에게 묻는다. Root는 `.com`을 관리하는 서버 목록을 던져준다.
3. **TLD DNS:** `.com` 서버에게 다시 묻는다. `.com` 서버는 `google.com`의 네임서버(Auth Server) 정보를 던져준다.
4. **Authoritative DNS:** 드디어 구글이 관리하는 네임서버에 도달한다. 여기서 최종적으로 `A 레코드`에 적힌 IP 주소를 받아온다.
5. **접속:** 받아온 IP로 서버에 연결한다.

---

## 4. 주요 DNS 레코드 타입 (Record Types)

개발자가 도메인을 설정할 때 반드시 알아야 할 레코드들이다.

| 타입 | 설명 | 용도 | 예시 |
| --- | --- | --- | --- |
| **A** | Address | 도메인을 **IPv4** 주소로 매핑 | `google.com` -> `1.2.3.4` |
| **AAAA** | Quad-A | 도메인을 **IPv6** 주소로 매핑 | `google.com` -> `2001:0db8...` |
| **CNAME** | Canonical Name | 도메인의 **별명** (도메인 -> 도메인) | `www.google.com` -> `google.com` |
| **MX** | Mail Exchanger | **이메일** 수신 서버 지정 | `google.com` -> `smtp.gmail.com` |
| **TXT** | Text | 텍스트 정보 (주로 소유권 확인, SPF 스팸 방지용) | `v=spf1 include:_spf.google.com ~all` |
| **NS** | Name Server | 해당 도메인을 관리하는 **네임서버** 지정 | `ns-123.awsdns.com` |

---

## 5. 전문가적 조언 (Pro Tip)

### 5.1 TTL (Time To Live)과 전파 시간

도메인 설정을 바꿨는데 "왜 적용이 안 되나요?"라고 묻는다면 100% **TTL** 때문이다.

* **상황:** 기존 IP가 `1.1.1.1`이었고 TTL이 `86400`(24시간)이었다면, 지금 IP를 `2.2.2.2`로 바꿔도 전 세계의 캐시 서버(ISP 등)는 최대 24시간 동안 옛날 주소를 알려준다.
* **Tip:** 중요한 마이그레이션(서버 이전) 전에는 미리 TTL을 `60`(1분)이나 `300`(5분)으로 극단적으로 줄여놓고 작업을 시작해야 한다.

### 5.2 CNAME vs A Alias (Root Domain 문제)

DNS 표준상 도메인의 뿌리(`example.com`, Naked Domain)에는 **CNAME을 설정할 수 없다.** (MX 레코드 등과 충돌하기 때문).

* **문제:** AWS ELB 같은 로드밸런서는 IP가 수시로 바뀌어서 도메인 주소만 제공한다. 근데 `example.com`에는 CNAME을 못 쓴다.
* **해결:** AWS Route53 등 최신 DNS 서비스는 **Alias(별칭) A 레코드**라는 독자 기술을 제공해, Root 도메인도 CNAME처럼 동작하게 해 준다.

### 5.3 개발자의 비밀 무기: `/etc/hosts`

도메인을 정식으로 구입하거나 설정하기 전에, 내 컴퓨터에서만 특정 도메인이 특정 IP로 가도록 속일 수 있다.

* **경로:** (Mac/Linux) `/etc/hosts`, (Windows) `C:\Windows\System32\drivers\etc\hosts`
* **활용:** 개발 서버(`dev.myapp.com`)를 내 로컬(`127.0.0.1`)로 띄워 놓고 테스트할 때 필수적이다.

```text
# /etc/hosts file example
127.0.0.1       dev.myapp.com

```
