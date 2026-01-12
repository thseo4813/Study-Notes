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
| **연결 리스트(Linked List)** | 노드들이 링크로 연결된 구조 | **장점:** 동적 크기 조정, 중간 삽입/삭제 O(1), 메모리 단편화 저항<br>**단점:** 임의 접근 O(n), 추가 메모리(포인터) 사용, CPU 캐시 비효율 | **사용례:** 큐/스택 구현, 빈번한 중간 삽입/삭제, 메모리 단편화가 문제되는 환경<br>**주의:** n번째 요소 접근 시 처음부터 순차 탐색해야 함 |
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

## 3. 연결 리스트의 본질 이해

### 3.1 연결 리스트가 왜 필요할까? (배열의 한계)

연결 리스트를 이해하려면 먼저 **배열(Array)**의 문제를 파악해야 합니다.

**배열의 메모리 구조:**
```
메모리 주소: 1000  1004  1008  1012  1016  1020
데이터:     [10]  [20]  [30]  [40]  [50]  [  ]
인덱스:       0     1     2     3     4     5
```

**배열의 문제점들:**
1. **크기가 고정적:** 배열을 만들 때 크기를 미리 정해야 함
2. **중간 삽입/삭제의 비효율성:** 중간에 데이터를 넣으려면 모든 요소를 이동시켜야 함

```java
// 배열에서 중간에 삽입하는 문제
int[] arr = {10, 20, 30, 40, 50};  // 크기 5의 배열
// 인덱스 2에 25를 삽입하려면?
// 30, 40, 50을 모두 한 칸씩 뒤로 밀어야 함!
```

**연결 리스트의 해결책:**
연결 리스트는 "데이터"와 "다음 데이터의 주소"를 함께 저장하는 방식입니다.

### 3.2 연결 리스트의 기본 구성 요소

**노드(Node)의 구조:**
```
[데이터] → [다음 노드 주소]
```

**단순한 연결 리스트의 모습:**
```
노드1: [10] → 주소(노드2)
노드2: [20] → 주소(노드3)
노드3: [30] → 주소(null, 끝 표시)
```

**Java로 표현하면:**
```java
class Node {
    int data;      // 실제 데이터
    Node next;     // 다음 노드를 가리키는 참조

    Node(int data) {
        this.data = data;
        this.next = null;
    }
}
```

### 3.3 연결 리스트의 메모리 레이아웃

**배열 vs 연결 리스트 메모리 비교:**

**배열의 메모리 (연속 할당):**
```
메모리 주소: 0x1000    0x1004    0x1008    0x100C
데이터:       [10]      [20]      [30]      [40]
인덱스 계산: arr[0] = 시작주소 + (0 × 4)
             arr[1] = 시작주소 + (1 × 4)
```

**연결 리스트의 메모리 (흩어진 할당):**
```
메모리 주소: 0x1000          0x2000          0x1500
노드:       [10|0x2000]     [20|0x1500]     [30|null]
```

**왜 이런 구조가 유연할까?**
- 각 노드가 독립적으로 메모리에 존재
- 새로운 노드를 추가할 때 다른 노드들을 이동시킬 필요 없음
- 메모리가 연속되지 않아도 됨 (단편화에 강함)

### 3.4 연결 리스트의 종류

**1. 단방향 연결 리스트 (Singly Linked List):**
```
Head → [10] → [20] → [30] → null
```

**2. 양방향 연결 리스트 (Doubly Linked List):**
```
Head ↔ [10] ↔ [20] ↔ [30] ↔ null
       ↑       ↑       ↑
     prev    prev    prev
```

**3. 환형 연결 리스트 (Circular Linked List):**
```
Head → [10] → [20] → [30] → (다시 Head로)
```

### 3.5 기본 연산들의 동작 원리

**삽입 연산 (중간 삽입):**
```java
// [10] → [30] 사이에 [20]을 삽입
// 1. 새로운 노드 생성: newNode = [20]
// 2. newNode.next = node10.next  (즉, [30])
// 3. node10.next = newNode

// 결과: [10] → [20] → [30]
```

**삭제 연산:**
```java
// [20] 노드를 삭제
// 1. node10.next = node20.next  ([30])
// 2. node20는 이제 메모리에서 분리됨

// 결과: [10] → [30]
```

**검색 연산:**
```java
// 값 30을 찾기
Node current = head;
while (current != null) {
    if (current.data == 30) {
        return current;  // 찾음!
    }
    current = current.next;
}
// 끝까지 못 찾으면 null 반환
```

### 3.6 연결 리스트의 장단점

**장점:**
1. **동적 크기:** 필요할 때마다 크기 조정 가능
2. **효율적인 삽입/삭제:** 중간 위치에서도 O(1) 가능
3. **메모리 단편화 저항:** 연속 메모리가 필요 없음

**단점:**
1. **느린 임의 접근:** n번째 요소를 찾으려면 처음부터 따라가야 함
2. **추가 메모리 사용:** 각 노드마다 "다음 주소" 저장 공간 필요
3. **CPU 캐시 비효율:** 메모리가 흩어져 있어 캐시 미스 발생

### 3.7 실전 구현 예제

```java
public class LinkedList {
    private Node head;
    private int size;

    // 노드 클래스
    private static class Node {
        int data;
        Node next;

        Node(int data) {
            this.data = data;
            this.next = null;
        }
    }

    // 맨 앞에 추가
    public void addFirst(int data) {
        Node newNode = new Node(data);
        newNode.next = head;
        head = newNode;
        size++;
    }

    // 특정 위치에 추가
    public void add(int index, int data) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException();
        }

        if (index == 0) {
            addFirst(data);
            return;
        }

        Node newNode = new Node(data);
        Node current = head;

        // 삽입할 위치의 이전 노드 찾기
        for (int i = 0; i < index - 1; i++) {
            current = current.next;
        }

        newNode.next = current.next;
        current.next = newNode;
        size++;
    }

    // 값 찾기
    public boolean contains(int data) {
        Node current = head;
        while (current != null) {
            if (current.data == data) {
                return true;
            }
            current = current.next;
        }
        return false;
    }

    // 값 삭제
    public boolean remove(int data) {
        if (head == null) return false;

        // 헤드가 삭제 대상인 경우
        if (head.data == data) {
            head = head.next;
            size--;
            return true;
        }

        Node current = head;
        while (current.next != null) {
            if (current.next.data == data) {
                current.next = current.next.next;  // 연결 끊기
                size--;
                return true;
            }
            current = current.next;
        }
        return false;
    }
}
```

### 3.8 연결 리스트의 실무 활용

**언제 연결 리스트를 사용할까?**
1. **빈번한 삽입/삭제:** 음악 플레이리스트, 브라우저 히스토리
2. **크기가 동적으로 변함:** 사용자 목록, 작업 큐
3. **메모리가 제한적:** 임베디드 시스템에서 메모리 단편화 방지

**언제 배열을 사용할까?**
1. **빠른 임의 접근 필요:** 게임 캐릭터 배열, 행렬 연산
2. **크기가 고정적:** 달력, 색상 팔레트
3. **CPU 캐시 효율 중요:** 실시간 그래픽스 처리

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

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        // accessOrder=true로 설정하여 접근 순서 유지
        super(capacity + 1, 1.0f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 용량 초과 시 가장 오래된 항목 제거
        return size() > capacity;
    }

    // 사용 예시
    public static void main(String[] args) {
        LRUCache<Integer, String> cache = new LRUCache<>(3);

        cache.put(1, "A");  // [1:A]
        cache.put(2, "B");  // [1:A, 2:B]
        cache.put(3, "C");  // [1:A, 2:B, 3:C]

        cache.get(1);       // 1번 키 접근, 최근으로 이동
        cache.put(4, "D");  // 용량 초과, 가장 오래된 2:B 제거
                            // 결과: [3:C, 1:A, 4:D]
    }
}
```

### 5.2 우선순위 큐 구현 (Java PriorityQueue)

```java
import java.util.PriorityQueue;
import java.util.Comparator;

public class MaxPriorityQueue<T> {
    private PriorityQueue<Element<T>> queue;

    // 요소와 우선순위를 함께 저장하는 클래스
    private static class Element<T> implements Comparable<Element<T>> {
        T item;
        int priority;

        Element(T item, int priority) {
            this.item = item;
            this.priority = priority;
        }

        @Override
        public int compareTo(Element<T> other) {
            // 내림차순 정렬 (최대 힙)
            return Integer.compare(other.priority, this.priority);
        }
    }

    public MaxPriorityQueue() {
        this.queue = new PriorityQueue<>();
    }

    public void push(T item, int priority) {
        queue.offer(new Element<>(item, priority));
    }

    public T pop() {
        if (queue.isEmpty()) {
            throw new IllegalStateException("Queue is empty");
        }
        return queue.poll().item;
    }

    public boolean isEmpty() {
        return queue.isEmpty();
    }

    // 사용 예시
    public static void main(String[] args) {
        MaxPriorityQueue<String> pq = new MaxPriorityQueue<>();

        pq.push("Task A", 3);  // 우선순위 3
        pq.push("Task B", 1);  // 우선순위 1
        pq.push("Task C", 5);  // 우선순위 5

        System.out.println(pq.pop());  // "Task C" (우선순위 5)
        System.out.println(pq.pop());  // "Task A" (우선순위 3)
        System.out.println(pq.pop());  // "Task B" (우선순위 1)
    }
}
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
```java
// 비효율적인 구현
public List<Post> getUserFeed(int userId) {
    List<Integer> followers = getFollowers(userId);  // 수십~수백명
    List<Post> posts = new ArrayList<>();

    for (Integer followerId : followers) {
        posts.addAll(getRecentPosts(followerId, 10));  // N번 DB 쿼리!
    }

    // 최신순 정렬 후 상위 50개 반환
    return posts.stream()
                .sorted((a, b) -> b.getCreatedAt().compareTo(a.getCreatedAt()))
                .limit(50)
                .collect(Collectors.toList());
}
```

**해결책 적용:**
1. **데이터 구조 변경:** Redis Sorted Set 사용
2. **Fan-out on Write 전략:** 포스트 작성 시점에 모든 팔로워의 피드에 추가
3. **메모리 최적화:** LRU 캐시로 오래된 포스트 자동 제거

```java
import redis.clients.jedis.Jedis;
import java.util.List;
import java.util.Set;

public class FeedManager {
    private final Jedis redis;

    public FeedManager() {
        this.redis = new Jedis("localhost", 6379);
    }

    public void postCreated(int userId, int postId, long timestamp) {
        // 포스트 작성 시 모든 팔로워의 피드에 추가
        List<Integer> followers = getFollowers(userId);

        for (Integer followerId : followers) {
            String feedKey = "feed:" + followerId;

            // Sorted Set에 추가 (timestamp를 score로 사용)
            redis.zadd(feedKey, timestamp, String.valueOf(postId));

            // 피드 크기 제한 (최근 1000개만 유지)
            redis.zremrangeByRank(feedKey, 0, -1001);
        }
    }

    public List<Post> getFeed(int userId, int page, int size) {
        // O(log N)으로 빠른 조회
        String feedKey = "feed:" + userId;
        int start = page * size;
        int end = start + size - 1;

        // 최신순으로 조회 (ZREVRANGE 사용)
        Set<String> postIds = redis.zrevrange(feedKey, start, end);

        return getPostsByIds(postIds);
    }

    // 헬퍼 메소드들
    private List<Integer> getFollowers(int userId) {
        // 팔로워 목록 조회 로직
        return List.of(); // 구현 생략
    }

    private List<Post> getPostsByIds(Set<String> postIds) {
        // 포스트 ID들로 실제 포스트 조회
        return List.of(); // 구현 생략
    }
}
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

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;
import java.util.List;
import java.util.Map;

public class Leaderboard {
    private final Jedis redis;
    private final String key;

    public Leaderboard() {
        this.redis = new Jedis("localhost", 6379);
        this.key = "game_leaderboard";
    }

    public Long updateScore(String playerId, double scoreChange) {
        // 원자적 점수 업데이트 및 순위 반환
        Transaction tx = redis.multi();
        tx.zincrby(key, scoreChange, playerId);
        tx.zrevrank(key, playerId);  // 새 순위 계산
        List<Object> result = tx.exec();

        return (Long) result.get(1);  // 새 순위 반환
    }

    public Map<String, Double> getTopPlayers(int count) {
        // 상위 플레이어 조회 (O(log N))
        return redis.zrevrangeWithScores(key, 0, count - 1);
    }

    public Long getPlayerRank(String playerId) {
        // 특정 플레이어 순위 조회 (O(log N))
        return redis.zrevrank(key, playerId);
    }

    public Map<String, Double> getRankRange(long startRank, long endRank) {
        // 순위 범위 조회
        return redis.zrevrangeWithScores(key, startRank, endRank);
    }
}
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

```java
import redis.clients.jedis.Jedis;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.nio.charset.StandardCharsets;

public class URLShortener {
    private final Jedis redis;
    private final String counterKey;
    private final String urlPrefix;
    private final String charset;

    public URLShortener() {
        this.redis = new Jedis("localhost", 6379);
        this.counterKey = "url_counter";
        this.urlPrefix = "https://short.ly/";
        this.charset = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    }

    public String shortenUrl(String longUrl) {
        // 중복 URL 처리
        String existingKey = redis.get("url:" + longUrl);
        if (existingKey != null) {
            return urlPrefix + existingKey;
        }

        // 새 키 생성
        long counter = redis.incr(counterKey);
        String shortKey = encodeBase62(counter);

        // 저장 (양방향 매핑)
        redis.set("url:" + longUrl, shortKey);
        redis.set("key:" + shortKey, longUrl);

        // TTL 설정 (선택적) - 1년
        int oneYearSeconds = 86400 * 365;
        redis.expire("url:" + longUrl, oneYearSeconds);
        redis.expire("key:" + shortKey, oneYearSeconds);

        return urlPrefix + shortKey;
    }

    public String expandUrl(String shortKey) {
        // 원래 URL 조회
        return redis.get("key:" + shortKey);
    }

    private String encodeBase62(long num) {
        // 숫자를 Base62 문자열로 변환
        if (num == 0) {
            return String.valueOf(charset.charAt(0));
        }

        StringBuilder result = new StringBuilder();
        while (num > 0) {
            result.append(charset.charAt((int)(num % 62)));
            num /= 62;
        }
        return result.reverse().toString();
    }

    // MD5 해시를 사용한 추가 검증 메소드 (선택적)
    private String generateHash(String input) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] hash = md.digest(input.getBytes(StandardCharsets.UTF_8));
            StringBuilder sb = new StringBuilder();
            for (byte b : hash) {
                sb.append(String.format("%02x", b));
            }
            return sb.toString();
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**설계 고려사항:**
- **충돌 처리:** 동일 URL에 대한 재사용
- **만료 정책:** 사용하지 않는 URL 자동 삭제
- **분산 처리:** 여러 서버에서 카운터 동기화
- **보안:** 예측 불가능한 키 생성

---

*"올바른 자료구조 선택이 알고리즘 최적화보다 중요하다."*

> 실무에서는 이론적 복잡도보다 메모리 사용량, 캐시 효율성, 동시성 처리가 더 중요하다. 프로파일링과 벤치마킹을 통해 실제 환경에 최적화하라.
