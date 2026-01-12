# 자료구조와 알고리즘의 본질

자료구조와 알고리즘의 핵심 개념을 정리하는 공간입니다. 효율적인 데이터 저장과 처리를 위한 기본 개념들을 다룹니다.

## 1. 핵심 요약 (Executive Summary)

데이터를 효과적으로 저장하고 조작하기 위한 구조를 **자료구조(Data Structure)**라고 하며, 이를 처리하는 절차를 **알고리즘(Algorithm)**이라고 한다. 같은 문제를 해결하더라도 자료구조 선택에 따라 성능이 천차만별로 달라진다.

> **결론:**
> 1. **메모리 vs 속도:** 배열(Array)은 빠르지만 크기 고정, 연결 리스트(Linked List)는 유연하지만 느림
> 2. **검색 vs 삽입:** 해시 테이블(Hash Table)은 빠른 검색, 트리(Tree)는 정렬된 데이터에 유리
> 3. **실무 선택:** 문제의 특성(데이터 크기, 접근 패턴)에 따라 최적의 자료구조를 선택하는 것이 핵심

---

## 2. 주요 자료구조 비교

### 2.1 선형 구조 (Linear Structures)

| 자료구조 | 개념 | 장점 | 단점 | 주요 사용 사례 |
| --- | --- | --- | --- | --- |
| **배열(Array)** | 메모리에 연속적으로 저장된 동일한 타입의 데이터 모음 | 인덱스로 O(1) 접근, 메모리 효율적 | 크기 고정, 중간 삽입/삭제 시 O(n) | 정적 데이터 저장, 행렬 연산 |
| **연결 리스트(Linked List)** | 노드들이 링크로 연결된 구조 | 동적 크기 조정, 중간 삽입/삭제 O(1) | 임의 접근 O(n), 추가 메모리 사용 | 큐 구현, 빈번한 삽입/삭제 |
| **스택(Stack)** | LIFO(Last In, First Out) 방식의 자료구조 | 간단한 구현, 함수 호출 관리에 적합 | 제한된 접근 | 함수 호출 스택, 실행 취소(Undo) |
| **큐(Queue)** | FIFO(First In, First Out) 방식의 자료구조 | 공정한 처리, 순서 보장 | 제한된 접근 | 작업 대기열, BFS 알고리즘 |

### 2.2 비선형 구조 (Non-linear Structures)

| 자료구조 | 개념 | 장점 | 단점 | 주요 사용 사례 |
| --- | --- | --- | --- | --- |
| **해시 테이블(Hash Table)** | 키-값 쌍으로 데이터를 저장, 해시 함수로 빠른 접근 | 평균 O(1) 검색/삽입/삭제 | 해시 충돌, 순서 보장 안됨 | 캐시, 데이터베이스 인덱스 |
| **이진 트리(Binary Tree)** | 각 노드가 최대 2개의 자식을 가지는 트리 | 정렬된 데이터 유지, 탐색 O(log n) | 균형 유지가 어려움 | 검색 트리, 힙 구현 |
| **힙(Heap)** | 완전 이진 트리 기반, 우선순위 큐 구현 | 최솟값/최댓값 빠른 접근 | 임의 값 검색 O(n) | 우선순위 큐, 스케줄링 |
| **그래프(Graph)** | 노드와 엣지로 구성된 복잡한 관계 표현 | 유연한 관계 모델링 | 구현 복잡성, 공간 사용 많음 | 소셜 네트워크, 경로 찾기 |

---

## 3. 시간 복잡도(Big O) 비교표

알고리즘 성능을 평가하는 핵심 지표입니다. n은 데이터 크기를 의미합니다.

### 3.1 기본 연산 비교

| 연산 | 배열 | 연결 리스트 | 해시 테이블 | 이진 탐색 트리 |
| --- | --- | --- | --- | --- |
| **접근(Access)** | O(1) | O(n) | O(1)* | O(log n) |
| **검색(Search)** | O(n) | O(n) | O(1)* | O(log n) |
| **삽입(Insert)** | O(n) | O(1) | O(1)* | O(log n) |
| **삭제(Delete)** | O(n) | O(1) | O(1)* | O(log n) |

*해시 충돌이 적은 경우

### 3.2 공간 복잡도

| 자료구조 | 공간 복잡도 | 비고 |
| --- | --- | --- |
| **배열** | O(n) | 연속 메모리 할당 |
| **연결 리스트** | O(n) | 포인터 오버헤드 포함 |
| **해시 테이블** | O(n) | 로드 팩터에 따라 동적 확장 |
| **트리** | O(n) | 균형에 따라 차이 |

---

## 4. 자료구조 선택 가이드

### 4.1 문제 유형별 추천

| 상황 | 추천 자료구조 | 이유 |
| --- | --- | --- |
| **빠른 임의 접근** | 배열(Array) | 인덱스로 O(1) 접근 |
| **빈번한 삽입/삭제** | 연결 리스트 | 중간 연산 O(1) |
| **키-값 쌍 빠른 검색** | 해시 테이블 | 평균 O(1) 검색 |
| **정렬된 데이터 관리** | 이진 탐색 트리 | 자동 정렬, O(log n) |
| **우선순위 처리** | 힙(Heap) | 최솟값/최댓값 O(1) |
| **LIFO 작업** | 스택(Stack) | 자연스러운 후입선출 |
| **FIFO 작업** | 큐(Queue) | 공정한 순서 처리 |

### 4.2 프로그래밍 언어별 특징

* **Java:** ArrayList(동적 배열), LinkedList(연결 리스트), HashMap(해시 테이블)
* **Python:** list(동적 배열), dict(해시 테이블), heapq(힙)
* **JavaScript:** Array(동적 배열), Map(해시 테이블), Set(중복 없는 집합)

---

## 5. 실무 적용 예시

### 5.1 캐시 구현 (LRU Cache)

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        # 최근 사용으로 이동
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value

        if len(self.cache) > self.capacity:
            # 가장 오래된 항목 제거
            self.cache.popitem(last=False)
```

### 5.2 우선순위 큐 구현 (Python heapq)

```python
import heapq

# 최소 힙으로 최대 우선순위 큐 구현
class PriorityQueue:
    def __init__(self):
        self.heap = []

    def push(self, item, priority):
        # 음수로 변환하여 최대 힙처럼 동작
        heapq.heappush(self.heap, (-priority, item))

    def pop(self):
        return heapq.heappop(self.heap)[1]

    def is_empty(self):
        return len(self.heap) == 0
```

---

## 6. 전문가적 조언 (Pro Tip)

### 6.1 자료구조의 트레이드오프 이해
* **메모리 vs 속도:** 더 빠른 접근을 위해 더 많은 메모리를 사용하는 경우가 많음
* **단순성 vs 효율성:** 간단한 자료구조로도 대부분의 문제를 해결할 수 있음

### 6.2 프로파일링의 중요성
* 이론적 복잡도와 실제 성능은 다를 수 있음
* 실제 애플리케이션에서 프로파일링하여 최적의 자료구조를 선택해야 함

### 6.3 현대 언어의 발전
* 고수준 언어들은 내부적으로 최적화된 자료구조를 제공
* 직접 구현하기보다는 표준 라이브러리를 활용하는 것이 안전하고 효율적

---

## 7. 실무 문제 해결 사례

### 7.1 소셜 미디어 피드 최적화 (Twitter/LinkedIn 사례)

**문제 상황:**
- 수백만 사용자의 피드를 실시간으로 생성
- 각 사용자의 타임라인에 수십~수백 개의 포스트 표시
- 팔로워 수에 따른 성능 저하 (N+1 쿼리 문제)

**기존 아키텍처:**
```python
# 비효율적인 구현
def get_user_feed(user_id):
    followers = get_followers(user_id)  # 수십~수백명
    posts = []
    for follower in followers:
        posts.extend(get_recent_posts(follower, limit=10))  # N번 DB 쿼리!
    return sorted(posts, key=lambda x: x.created_at, reverse=True)[:50]
```

**해결책 적용:**
1. **데이터 구조 변경:** Redis Sorted Set 사용
2. **Fan-out on Write 전략:** 포스트 작성 시점에 모든 팔로워의 피드에 추가
3. **메모리 최적화:** LRU 캐시로 오래된 포스트 자동 제거

```python
import redis

class FeedManager:
    def __init__(self):
        self.redis = redis.Redis()

    def post_created(self, user_id, post_id, timestamp):
        """포스트 작성 시 모든 팔로워의 피드에 추가"""
        followers = self.get_followers(user_id)

        for follower_id in followers:
            feed_key = f"feed:{follower_id}"
            # Sorted Set에 추가 (timestamp를 score로 사용)
            self.redis.zadd(feed_key, {post_id: timestamp})

            # 피드 크기 제한 (최근 1000개만 유지)
            self.redis.zremrangebyrank(feed_key, 0, -1001)

    def get_feed(self, user_id, page=0, size=50):
        """O(log N)으로 빠른 조회"""
        feed_key = f"feed:{user_id}"
        start = page * size
        end = start + size - 1

        # 최신순으로 조회
        post_ids = self.redis.zrevrange(feed_key, start, end)
        return self.get_posts_by_ids(post_ids)
```

**결과:**
- 응답 시간: 500ms → 50ms (10배 개선)
- DB 부하: 95% 감소
- 확장성: 서버 수평 확장 가능

### 7.2 게임 리더보드 구현 (실시간 랭킹 시스템)

**문제 상황:**
- 수백만 플레이어의 점수 실시간 업데이트
- 순위 변동 시 즉각 반영
- 동시성 문제 (Race Condition)

**해결책 적용:**
1. **Redis Sorted Set 활용**
2. **Atomic 연산으로 동시성 해결**
3. **메모리 효율적 설계**

```python
class Leaderboard:
    def __init__(self):
        self.redis = redis.Redis()
        self.key = "game_leaderboard"

    def update_score(self, player_id, score_change):
        """원자적 점수 업데이트"""
        with self.redis.pipeline() as pipe:
            pipe.zincrby(self.key, score_change, player_id)
            pipe.zrevrank(self.key, player_id)  # 새 순위 계산
            result = pipe.execute()
            return result[1]  # 새 순위 반환

    def get_top_players(self, count=10):
        """상위 플레이어 조회 (O(log N))"""
        return self.redis.zrevrange(self.key, 0, count-1, withscores=True)

    def get_player_rank(self, player_id):
        """특정 플레이어 순위 조회 (O(log N))"""
        return self.redis.zrevrank(self.key, player_id)

    def get_rank_range(self, start_rank, end_rank):
        """순위 범위 조회"""
        return self.redis.zrevrange(self.key, start_rank, end_rank, withscores=True)
```

**성능 비교:**

| 연산 | MySQL 기반 | Redis Sorted Set | 개선율 |
| --- | --- | --- | --- |
| 점수 업데이트 | O(log N) | O(log N) | 유사 |
| 순위 조회 | O(N) | O(log N) | 1000배+ |
| 범위 조회 | O(N) | O(log N) | 1000배+ |
| 메모리 사용 | 높음 | 최적화 | 50% 절약 |

### 7.3 URL 단축 서비스 최적화 (bit.ly 스타일)

**문제 상황:**
- 긴 URL을 짧은 키로 변환
- 수십억 개의 매핑 관리
- 빠른 조회와 높은 동시성 요구

**해결책 적용:**
1. **Hash Table + 데이터베이스 조합**
2. **Base62 인코딩으로 키 생성**
3. **분산 캐시로 성능 최적화**

```python
import hashlib
import redis

class URLShortener:
    def __init__(self):
        self.redis = redis.Redis()
        self.counter_key = "url_counter"
        self.url_prefix = "https://short.ly/"

    def shorten_url(self, long_url):
        """URL 단축"""
        # 중복 URL 처리
        existing_key = self.redis.get(f"url:{long_url}")
        if existing_key:
            return self.url_prefix + existing_key.decode()

        # 새 키 생성
        counter = self.redis.incr(self.counter_key)
        short_key = self.encode_base62(counter)

        # 저장 (양방향 매핑)
        self.redis.set(f"url:{long_url}", short_key)
        self.redis.set(f"key:{short_key}", long_url)

        # TTL 설정 (선택적)
        self.redis.expire(f"url:{long_url}", 86400 * 365)  # 1년
        self.redis.expire(f"key:{short_key}", 86400 * 365)

        return self.url_prefix + short_key

    def expand_url(self, short_key):
        """원래 URL 조회"""
        return self.redis.get(f"key:{short_key}")

    def encode_base62(self, num):
        """숫자를 Base62 문자열로 변환"""
        chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
        if num == 0:
            return chars[0]

        result = []
        while num > 0:
            result.append(chars[num % 62])
            num //= 62
        return ''.join(reversed(result))
```

**설계 고려사항:**
- **충돌 처리:** 동일 URL에 대한 재사용
- **만료 정책:** 사용하지 않는 URL 자동 삭제
- **분산 처리:** 여러 서버에서 카운터 동기화
- **보안:** 예측 불가능한 키 생성

---

*"올바른 자료구조 선택이 알고리즘 최적화보다 중요하다."*

> 실무에서는 이론적 복잡도보다 메모리 사용량, 캐시 효율성, 동시성 처리가 더 중요하다. 프로파일링과 벤치마킹을 통해 실제 환경에 최적화하라.
