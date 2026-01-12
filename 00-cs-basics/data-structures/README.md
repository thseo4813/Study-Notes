# 🏗️ 자료구조: 코딩의 기초 체력

## 💻 실제 코딩에서 마주치는 상황들

### 개발자가 흔히 하는 고민들:

**"어떤 자료구조를 써야 빨라질까?"**
- **ArrayList vs LinkedList**: 중간 삽입이 많은가? 아니면 끝에 추가만?
- **HashMap vs TreeMap**: 빠른 검색 vs 정렬된 순서?
- **Queue vs Stack**: 먼저 들어온 거 먼저 처리 vs 나중에 들어온 거 먼저?

**"메모리 vs 속도 중 뭐가 중요할까?"**
- **게임**: 속도 우선, 메모리는 덜 중요
- **모바일 앱**: 메모리 절약 우선
- **빅데이터**: 둘 다 중요

## 🎯 1분 요약: 왜 자료구조를 알아야 하나?

**좋은 자료구조 = 좋은 성능 + 좋은 코드**

- **배열**: 가장 빠름, 가장 단순
- **해시**: 가장 빠른 검색
- **트리**: 정렬된 데이터에 강함
- **그래프**: 복잡한 관계 표현

> **결론:** 상황에 맞는 자료구조 선택이 성능을 좌우한다

---

## 2. 🔍 자료구조 선택 가이드 (실전 사용법)

### 2.1 ArrayList vs LinkedList: 어떤 걸 쓸까?

**실제 코딩 상황:**
```java
// 상황 1: 끝에만 추가, 앞에서만 꺼내기 (큐)
ArrayList<String> queue = new ArrayList<>();
queue.add("주문1");      // O(1)
queue.add("주문2");      // O(1)
queue.remove(0);         // O(n) - 느림!

// 상황 2: 중간에 자주 삽입/삭제
LinkedList<String> tasks = new LinkedList<>();
tasks.add("할일1");      // O(1)
tasks.add(1, "급한일");  // O(1) - 빠름!
```

**💡 선택 기준:**

| 상황 | 추천 | 이유 | 실제 사례 |
|------|-------|------|----------|
| **끝에 추가만** | ArrayList | 메모리 효율적 | 로그 저장 |
| **중간 삽입 많음** | LinkedList | 삽입/삭제 빠름 | 할일 목록 |
| **임의 접근 많음** | ArrayList | 인덱스로 바로 접근 | 게임 캐릭터 배열 |
| **메모리 제한적** | ArrayList | 포인터 오버헤드 없음 | 모바일 앱 |

### 2.2 HashMap vs TreeMap: 검색 속도 vs 정렬

**실제 코딩 상황:**

```java
// 상황 1: 빠른 검색만 필요 (로그인 세션)
HashMap<String, User> userCache = new HashMap<>();
userCache.put("user123", user);    // O(1)
User user = userCache.get("user123"); // O(1) - 엄청 빠름!

// 상황 2: 정렬된 순서가 필요 (점수판)
TreeMap<Integer, String> scoreBoard = new TreeMap<>();
scoreBoard.put(95, "김철수");
scoreBoard.put(87, "이영희");
// 자동 정렬: 87(이영희), 95(김철수)
```

**💡 선택 기준:**

| 상황 | 추천 | 성능 | 실제 사례 |
|------|-------|------|----------|
| **빠른 검색** | HashMap | O(1) | 사용자 세션, 캐시 |
| **정렬된 데이터** | TreeMap | O(log n) | 점수판, 사전 |
| **범위 검색** | TreeMap | O(log n) | 가격대별 상품 검색 |
| **메모리 적음** | HashMap | 더 효율적 | 대용량 캐시 |

---

## 3. ⚡ 시간 복잡도: 실제 성능 차이는 얼마나 날까?

**Big O가 뭐길래 중요할까?**

**비유로 이해하기:**
- **O(1)**: 사전에서 바로 찾기
- **O(log n)**: 전화번호부에서 찾기 (반씩 줄여가며)
- **O(n)**: 사람들 한 명씩 물어보기
- **O(n²)**: 모두가 모두에게 물어보기

### 3.1 실제 성능 비교 (n=100만개 데이터 기준)

| 연산 | ArrayList | LinkedList | HashMap | TreeMap | 실제 시간 |
|------|-----------|------------|---------|---------|----------|
| **인덱스로 접근** | O(1) | O(n) | O(1) | O(log n) | 0.001초 vs 1초 |
| **값으로 검색** | O(n) | O(n) | O(1) | O(log n) | 1초 vs 0.001초 |
| **끝에 추가** | O(1) | O(1) | O(1) | O(log n) | 모두 0.001초 |
| **중간 삽입** | O(n) | O(1) | O(1) | O(log n) | 1초 vs 0.001초 |

**💡 실무에서 자주 하는 실수:**
```java
// ❌ O(n) 검색을 반복 (100만개 × 1000번 = 10억번 연산!)
for (String name : names) {
    if (names.contains(target)) {  // O(n) 검색
        // 처리
    }
}

// ✅ HashSet으로 O(1)로 바꾸기
Set<String> nameSet = new HashSet<>(names);  // O(n) 한번만
if (nameSet.contains(target)) {  // O(1)
    // 처리
}
```

### 3.2 공간 복잡도

| 자료구조 | 공간 복잡도 | 비고 |
| --- | --- | --- |
| **배열** | O(n) | 연속 메모리 할당 |
| **연결 리스트** | O(n) | 포인터 오버헤드 포함 |
| **해시 테이블** | O(n) | 로드 팩터에 따라 동적 확장 |
| **트리** | O(n) | 균형에 따라 차이 |

---

## 4. 🔗 연결 리스트: 배열의 단점을 해결하는 방법

### 4.1 배열의 문제점 (실제 상황)

**배열의 치명적인 단점:**
```java
// ❌ 배열의 문제: 중간 삽입이 너무 느림
ArrayList<String> todos = new ArrayList<>(Arrays.asList("밥먹기", "공부하기", "운동하기"));
// "급한일"을 두 번째에 넣으려면?
todos.add(1, "급한일");  // 뒤에 있는 요소들 모두 한 칸씩 밀어야 함! O(n)
```

**연결 리스트의 해결:**
```java
// ✅ 연결 리스트: 중간 삽입이 빠름
LinkedList<String> todos = new LinkedList<>(Arrays.asList("밥먹기", "공부하기", "운동하기"));
todos.add(1, "급한일");  // 그냥 링크만 바꾸면 됨! O(1)
```

**결과:**
```
배열:     [밥먹기][급한일][공부하기][운동하기] (모두 이동)
연결리스트: 밥먹기 → 급한일 → 공부하기 → 운동하기 (링크만 변경)
```

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
