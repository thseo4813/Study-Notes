# 💡 알고리즘: 코딩 인터뷰 생존 가이드

## 🥺 실제로 겪어본 고민들

### 코딩 인터뷰에서 흔히 마주치는 상황들:

**"이 알고리즘 시간 복잡도가 O(n²)인데... 너무 느린데?"**
- 100개 데이터로 테스트 통과했는데, 10만개 넣으니 10초 걸림
- 면접관: "더 효율적인 방법은 없나요?"

**"메모리 제한 때문에 고민돼"**
- "Two Sum" 문제에서 O(n) 공간 대신 O(1)로 풀라고 함
- 해시맵 쓰면 쉽지만, 메모리 제한 때문에 못 씀

**"정렬 알고리즘 중 뭐 써야 할까?"**
- 퀵소트 vs 머지소트 vs 힙소트, 각각 언제 써야 하나?
- 자바스크립트의 sort()는 어떤 알고리즘 쓸까?

## 🎯 1분 요약: 왜 알고리즘이 중요한가?

**알고리즘 = 문제 해결의 레시피**

- **좋은 알고리즘**: 100만개 데이터, 1초 안에 처리
- **나쁜 알고리즘**: 100개 데이터, 10초 걸림
- **인터뷰**: "이게 최선인가?"라는 질문의 답

> **결론:**
> 1. **시간 복잡도**: 코드 실행 속도 예측
> 2. **공간 복잡도**: 메모리 사용량 예측
> 3. **트레이드오프**: 속도 vs 메모리 vs 구현 복잡도
> 4. **실전 선택**: 문제 크기에 맞는 최적 알고리즘

---

## 2. 알고리즘의 기초: 개념과 중요성

### 2.1 알고리즘이란 무엇인가?

**알고리즘의 정의:**
- 유한한 단계의 명확한 명령어 집합
- 입력을 받아 출력으로 변환하는 과정
- 반드시 종료되는(endliche) 특성

**좋은 알고리즘의 조건:**
- **정확성 (Correctness):** 올바른 결과를 생성
- **효율성 (Efficiency):** 적은 자원 사용
- **명확성 (Clarity):** 이해하기 쉬운 구조
- **완전성 (Completeness):** 모든 경우 처리
- **최적성 (Optimality):** 가능한 최고의 효율

### 2.2 알고리즘 vs 프로그램

| 측면 | 알고리즘 | 프로그램 |
| --- | --- | --- |
| **추상성** | 문제 해결 방법 | 구체적 구현 |
| **언어** | 자연어/의사코드 | 프로그래밍 언어 |
| **실행** | 사람의 두뇌 | 컴퓨터 하드웨어 |
| **검증** | 수학적 증명 | 테스트와 디버깅 |

### 2.3 알고리즘의 발전史

- **고대:** 유클리드 호제법 (기원전 300년)
- **중세:** 알콰리즈미 (780-850, 알고리즘이라는 용어 유래)
- **현대:** 튜링 기계, 컴퓨터 알고리즘
- **현재:** 머신러닝 알고리즘, 양자 알고리즘

---

## 3. ⚡ 빅오 표기법: 코드 속도 측정하기

### 3.1 빅오는 왜 필요할까? (알고리즘의 속도계)

**🚗 비유로 이해하기: 자동차 연비 측정**

- **빅오는 "연료(연산)가 얼마나 들까?"를 측정하는 방법**
- 데이터 양이 늘어날 때 연산 횟수가 어떻게 변하는지 예측

**🏃‍♂️ 일상생활 예시:**
```
📚 책 찾기 시나리오

도서관에서 책 찾기:
- 책이 10권: 1분 걸림
- 책이 100권: 10분 걸림 → O(n) - 책 개수만큼 시간 걸림

전화번호부에서 이름 찾기:
- 10명: 1분 걸림
- 100명: 2분 걸림 → O(log n) - 데이터 10배 늘어도 시간 2배만 늘어남

무작위로 섞인 카드 뒤집기:
- 10장: 1분 걸림
- 100장: 100분 걸림 → O(n²) - 데이터 10배 늘면 시간 100배 늘어남
```

### 3.2 빅오 기호별 의미

**O(1) - 상수 시간: "항상 똑같은 시간"**
```
⚡ 특징: 데이터 개수와 상관없이 항상 같은 시간
💡 예시: TV 채널 돌리기, 전등 스위치 켜기
🚫 절대 변하지 않는 연산
```

**O(log n) - 로그 시간: "점점 느려지지만 아주 천천히"**
```
📈 특징: 데이터가 2배 늘 때 연산은 1번만 늘어남
💡 예시: 전화번호부에서 이름 찾기, 사전에서 단어 찾기
📊 그래프: 천천히 올라가는 곡선
```

**O(n) - 선형 시간: "데이터만큼 일한다"**
```
📈 특징: 데이터 개수만큼 연산 횟수 증가
💡 예시: 줄서서 표 사기, 식당에서 음식 나열하기
📊 그래프: 직선으로 올라가는 그래프
```

**O(n²) - 제곱 시간: "데이터가 늘수록 급격히 느려짐"**
```
📈 특징: 데이터 제곱만큼 연산 횟수 증가
💡 예시: 모두가 서로 악수하기, 버블 정렬
📊 그래프: 급격히 올라가는 곡선 (나쁨!)
```

### 3.3 실제 성능 비교 (숫자로 보기)

| 빅오 | 10개 데이터 | 100개 데이터 | 1,000개 데이터 | 10,000개 데이터 |
|------|-------------|--------------|----------------|------------------|
| O(1) | 1초 | 1초 | 1초 | 1초 |
| O(log n) | 1초 | 2초 | 3초 | 4초 |
| O(n) | 1초 | 10초 | 100초 | 1,000초 |
| O(n²) | 1초 | 100초 | 10,000초 | 1,000,000초 |

**🔥 현실적인 의미:**
- **O(n²) 알고리즘**: 10만개 데이터 → 10초 (아직 OK)
- **O(n²) 알고리즘**: 100만개 데이터 → 100초 (느림)
- **O(n²) 알고리즘**: 1000만개 데이터 → 10,000초 = 3시간 (불가능!)

### 3.2 각 복잡도의 실제 의미

**O(1) - 상수 시간: "항상 1번만에 끝"**
```java
// 배열에서 특정 인덱스 값 가져오기
int getUserName(String[] users, int index) {
    return users[index];  // 몇 백만개든 1번만에 끝!
}
```

**O(log n) - 로그 시간: "데이터가 2배 늘어도 연산은 1번만 늘어남"**
```java
// 전화번호부에서 이름 찾기
int findUserIndex(String[] sortedUsers, String target) {
    int left = 0, right = sortedUsers.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (sortedUsers[mid].equals(target)) return mid;
        if (sortedUsers[mid].compareTo(target) < 0) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
// 10억개 데이터도 30번 비교만에 찾음!
```

**O(n) - 선형 시간:**
```python
def linear_search(arr, target):
    for i in range(len(arr)):
        if arr[i] == target:
            return i
    return -1
```

**O(n log n) - 선형 로그 시간:**
```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)
```

**O(n²) - 이차 시간:**
```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
```

### 3.2 공간 복잡도 (Space Complexity)

알고리즘이 사용하는 메모리 공간의 함수.

**예시:**
```python
# O(1) 공간 복잡도
def sum_array(arr):
    total = 0  # 상수 공간만 사용
    for num in arr:
        total += num
    return total

# O(n) 공간 복잡도
def copy_array(arr):
    result = []  # 입력 크기에 비례하는 공간 사용
    for item in arr:
        result.append(item)
    return result
```

### 3.3 복잡도 분석의 실제 적용

**문제 크기에 따른 실행 시간 비교:**

| 복잡도 | n=10 | n=100 | n=1,000 | n=10,000 |
| --- | --- | --- | --- | --- |
| O(1) | 1초 | 1초 | 1초 | 1초 |
| O(log n) | 1초 | 2초 | 3초 | 4초 |
| O(n) | 1초 | 10초 | 100초 | 1,000초 |
| O(n log n) | 3초 | 200초 | 3,000초 | 40,000초 |
| O(n²) | 10초 | 1,000초 | 100,000초 | 10,000,000초 |

---

## 4. 정렬 알고리즘 (Sorting Algorithms)

### 4.1 비교 기반 정렬

#### 4.1.1 버블 정렬 (Bubble Sort) - O(n²)

**🫧 비유: 거품이 위로 올라가는 것처럼 큰 값이 뒤로 이동**

**단계별 작동 방식:**
```
초기 배열: [5, 3, 8, 2, 1]

1회차 (큰 값들이 뒤로 "거품처럼" 올라감):
   [5, 3, 8, 2, 1] → 비교 5>3 → [3, 5, 8, 2, 1]
   [3, 5, 8, 2, 1] → 비교 5<8 → 그대로
   [3, 5, 8, 2, 1] → 비교 8>2 → [3, 5, 2, 8, 1]
   [3, 5, 2, 8, 1] → 비교 8>1 → [3, 5, 2, 1, 8] ✓ 가장 큰 8이 끝으로

2회차:
   [3, 5, 2, 1, 8] → 비교 3<5 → 그대로
   [3, 5, 2, 1, 8] → 비교 5>2 → [3, 2, 5, 1, 8]
   [3, 2, 5, 1, 8] → 비교 5>1 → [3, 2, 1, 5, 8] ✓ 5가 뒤로

3회차:
   [3, 2, 1, 5, 8] → 비교 3>2 → [2, 3, 1, 5, 8]
   [2, 3, 1, 5, 8] → 비교 3>1 → [2, 1, 3, 5, 8] ✓ 3이 뒤로

4회차:
   [2, 1, 3, 5, 8] → 비교 2>1 → [1, 2, 3, 5, 8] ✓ 완료!

최종 결과: [1, 2, 3, 5, 8]
```

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):  # 전체 배열을 n번 순회
        # 각 순회마다 가장 큰 값이 맨 뒤로 이동
        for j in range(0, n - i - 1):
            # 인접한 두 요소 비교
            if arr[j] > arr[j + 1]:
                # 큰 값이 뒤로 이동 (swap)
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr
```

**💡 특징 정리:**
- ✅ **장점**: 코드가 매우 간단함, 이해하기 쉬움
- ❌ **단점**: 느림! 데이터가 많아지면 기하급수적으로 느려짐
- 🎯 **사용처**: 교육용, 아주 작은 데이터(10개 미만)
- 🔒 **안정 정렬**: 같은 값의 순서가 유지됨

#### 4.1.2 선택 정렬 (Selection Sort) - O(n²)

**🎯 비유: 매번 가장 작은 값을 "선택"해서 제자리로 옮기는 것**

**단계별 작동 방식:**
```
초기 배열: [5, 3, 8, 2, 1]

1단계: 0번 위치에 가장 작은 값 찾기
   배열에서 최솟값은 1 (인덱스 4)
   [5, 3, 8, 2, 1] → [1, 3, 8, 2, 5] ✓ 1을 맨 앞으로

2단계: 1번 위치에 남은 값 중 최솟값 찾기
   [1, 3, 8, 2, 5]에서 1 이후 최솟값은 2 (인덱스 3)
   [1, 3, 8, 2, 5] → [1, 2, 8, 3, 5] ✓ 2를 두 번째로

3단계: 2번 위치에 남은 값 중 최솟값 찾기
   [1, 2, 8, 3, 5]에서 2 이후 최솟값은 3 (인덱스 3)
   [1, 2, 8, 3, 5] → [1, 2, 3, 8, 5] ✓ 3을 세 번째로

4단계: 3번 위치에 남은 값 중 최솟값 찾기
   [1, 2, 3, 8, 5]에서 3 이후 최솟값은 5 (인덱스 4)
   [1, 2, 3, 8, 5] → [1, 2, 3, 5, 8] ✓ 5를 네 번째로

최종 결과: [1, 2, 3, 5, 8]
```

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):  # 각 위치에 대해
        # 현재 위치부터 끝까지 최솟값 찾기
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j

        # 찾은 최솟값을 현재 위치로 이동
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
    return arr
```

**💡 특징 정리:**
- ✅ **장점**: 구현이 직관적, 버블 정렬보다 조금 빠름
- ❌ **단점**: 여전히 O(n²), 데이터가 많으면 느림
- 🎯 **사용처**: 아주 작은 데이터, 메모리가 제한적인 환경
- 🔄 **불안정 정렬**: 같은 값의 상대적 순서가 바뀔 수 있음

#### 4.1.3 삽입 정렬 (Insertion Sort) - O(n²), 최선 O(n)

**🃏 비유: 카드 게임처럼 새로운 카드를 제자리에 "끼워넣는" 것**

**단계별 작동 방식:**
```
초기 배열: [5, 3, 8, 2, 1]
정렬된 부분: [5],   아직 정렬 안 된 부분: [3, 8, 2, 1]

1단계: 3을 정렬된 부분에 끼워넣기
   [5, 3] → 3이 5보다 작으므로 교환 → [3, 5, 8, 2, 1]
   정렬된 부분: [3, 5],   나머지: [8, 2, 1]

2단계: 8을 정렬된 부분에 끼워넣기
   [3, 5, 8] → 8이 제일 크므로 그대로 → [3, 5, 8, 2, 1]
   정렬된 부분: [3, 5, 8],   나머지: [2, 1]

3단계: 2를 정렬된 부분에 끼워넣기
   [3, 5, 8, 2] → 2는 3보다 작으므로 왼쪽으로 이동
   [3, 5, 2, 8] → 2는 3보다 작으므로 계속 왼쪽으로
   [3, 2, 5, 8] → 2는 3보다 작으므로 계속 왼쪽으로
   [2, 3, 5, 8] ✓ 제자리 찾음
   정렬된 부분: [2, 3, 5, 8],   나머지: [1]

4단계: 1을 정렬된 부분에 끼워넣기
   [2, 3, 5, 8, 1] → 1은 모두보다 작으므로 맨 앞으로
   [1, 2, 3, 5, 8] ✓ 완료!

최종 결과: [1, 2, 3, 5, 8]
```

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):  # 두 번째 요소부터 시작
        key = arr[i]  # 현재 끼워넣을 값
        j = i - 1

        # key보다 큰 요소들을 오른쪽으로 밀면서 공간 만들기
        while j >= 0 and key < arr[j]:
            arr[j + 1] = arr[j]  # 요소를 오른쪽으로 이동
            j -= 1

        # 찾은 위치에 key 삽입
        arr[j + 1] = key
    return arr
```

**💡 특징 정리:**
- ✅ **장점**: 거의 정렬된 데이터에서는 O(n)으로 매우 빠름
- ✅ **장점**: 이해하기 쉽고, 구현이 직관적
- ❌ **단점**: 최악의 경우 O(n²), 데이터가 많으면 느림
- 🎯 **사용처**: 실시간 데이터(온라인 알고리즘), 거의 정렬된 데이터
- 🔒 **안정 정렬**: 같은 값의 순서가 유지됨

### 4.2 분할 정복 정렬

#### 4.2.1 퀵 정렬 (Quick Sort) - 평균 O(n log n), 최악 O(n²)

**⚡ 비유: "분할 정복" - 큰 문제를 작은 문제로 나누어 해결**

**핵심 개념: 피벗(pivot)을 기준으로 좌우로 나누기**
```
피벗: 기준점 (보통 가운데 값 선택)
- 피벗보다 작은 값 → 왼쪽
- 피벗과 같은 값 → 가운데
- 피벗보다 큰 값 → 오른쪽
```

**단계별 작동 방식:**
```
초기 배열: [5, 3, 8, 2, 1, 9, 4, 7]

1단계: 피벗 선택 (가운데 값 2 선택)
   피벗: 2
   왼쪽: [1] (2보다 작은 값)
   가운데: [2] (2와 같은 값)
   오른쪽: [5, 3, 8, 9, 4, 7] (2보다 큰 값)

   결과: [1] + [2] + [5, 3, 8, 9, 4, 7]

2단계: 오른쪽 부분 [5, 3, 8, 9, 4, 7] 정렬
   피벗: 9 (가운데 값)
   왼쪽: [5, 3, 8, 4, 7]
   가운데: [9]
   오른쪽: []

   결과: [1, 2] + [5, 3, 8, 4, 7] + [9]

3단계: [5, 3, 8, 4, 7] 다시 정렬
   피벗: 8 (가운데 값)
   왼쪽: [5, 3, 4, 7]
   가운데: [8]
   오른쪽: []

   계속 반복... 최종적으로 모든 부분이 정렬됨
```

```python
def quick_sort(arr):
    if len(arr) <= 1:  # 기저 조건: 1개 이하면 이미 정렬됨
        return arr

    pivot = arr[len(arr) // 2]  # 피벗 선택 (가운데 값)

    # 피벗을 기준으로 3등분
    left = [x for x in arr if x < pivot]     # 피벗보다 작은 값들
    middle = [x for x in arr if x == pivot]  # 피벗과 같은 값들
    right = [x for x in arr if x > pivot]    # 피벗보다 큰 값들

    # 재귀적으로 정렬 후 합치기
    return quick_sort(left) + middle + quick_sort(right)
```

**💡 특징 정리:**
- ✅ **장점**: 평균적으로 가장 빠른 정렬 알고리즘
- ✅ **장점**: 추가 메모리 사용 적음 (제자리 정렬 가능)
- ❌ **단점**: 최악의 경우 O(n²) - 이미 정렬된 데이터에서 발생
- 🎯 **사용처**: 일반적인 정렬, 큰 데이터셋
- 🔄 **불안정 정렬**: 같은 값의 순서가 바뀔 수 있음

**🎲 피벗 선택 전략:**
- **좋은 피벗**: 중앙값에 가까운 값 → 균형 잡힌 분할
- **나쁜 피벗**: 최솟값이나 최댓값 → 한 쪽으로 치우친 분할

#### 4.2.2 병합 정렬 (Merge Sort) - O(n log n)

**🔀 비유: "분할 후 병합" - 큰 책을 작은 장으로 나누어 각각 정렬한 후 합치기**

**핵심 개념: "분할(Divide) → 정복(Conquer) → 결합(Combine)"**
```
1. 큰 배열을 작은 단위로 계속 나누기
2. 나눠진 작은 배열들을 각각 정렬하기
3. 정렬된 작은 배열들을 큰 배열로 병합하기
```

**단계별 작동 방식:**
```
초기 배열: [5, 3, 8, 2, 1, 9, 4, 7]

1단계: 반으로 나누기
   왼쪽: [5, 3, 8, 2]     오른쪽: [1, 9, 4, 7]

2단계: 다시 반으로 나누기
   왼쪽: [5, 3] [8, 2]    오른쪽: [1, 9] [4, 7]

3단계: 계속 나누기
   [5] [3] [8] [2] [1] [9] [4] [7]  (더 이상 나눌 수 없음)

4단계: 작은 배열들부터 병합하기
   [5] + [3] → [3, 5]     [8] + [2] → [2, 8]
   [1] + [9] → [1, 9]     [4] + [7] → [4, 7]

5단계: 더 큰 배열로 병합
   [3, 5] + [2, 8] → [2, 3, 5, 8]
   [1, 9] + [4, 7] → [1, 4, 7, 9]

6단계: 최종 병합
   [2, 3, 5, 8] + [1, 4, 7, 9] → [1, 2, 3, 4, 5, 7, 8, 9]

최종 결과: [1, 2, 3, 4, 5, 7, 8, 9]
```

**병합(Merge) 과정 상세:**
```python
def merge(left, right):
    result = []
    i = j = 0  # 각 배열의 시작 인덱스

    # 두 배열을 비교하면서 작은 값부터 result에 추가
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:  # 왼쪽이 작거나 같으면
            result.append(left[i])
            i += 1
        else:                    # 오른쪽이 작으면
            result.append(right[j])
            j += 1

    # 남은 요소들 모두 추가
    result.extend(left[i:])  # 왼쪽 배열 남은 부분
    result.extend(right[j:]) # 오른쪽 배열 남은 부분

    return result
```

```python
def merge_sort(arr):
    if len(arr) <= 1:  # 기저 조건
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])   # 왼쪽 반 정렬
    right = merge_sort(arr[mid:])  # 오른쪽 반 정렬

    return merge(left, right)      # 병합
```

**💡 특징 정리:**
- ✅ **장점**: 항상 O(n log n), 최악의 경우에도 안정적
- ✅ **장점**: 안정 정렬, 외부 정렬에 적합
- ❌ **단점**: 추가 메모리 공간 필요 (O(n))
- 🎯 **사용처**: 큰 데이터 정렬, 안정성이 중요한 경우
- 🔒 **안정 정렬**: 같은 값의 순서가 유지됨

**📊 분할 정복 시각화:**
```
레벨 0: [5,3,8,2,1,9,4,7] (8개)
레벨 1: [5,3,8,2] [1,9,4,7] (각 4개)
레벨 2: [5,3] [8,2] [1,9] [4,7] (각 2개)
레벨 3: [5][3] [8][2] [1][9] [4][7] (각 1개)
병합 →: [3,5] [2,8] [1,9] [4,7] (각 2개)
병합 →: [2,3,5,8] [1,4,7,9] (각 4개)
병합 →: [1,2,3,4,5,7,8,9] (8개)
```

### 4.3 특수 목적 정렬

#### 4.3.1 힙 정렬 (Heap Sort) - O(n log n)
```python
def heap_sort(arr):
    def heapify(arr, n, i):
        largest = i
        left = 2 * i + 1
        right = 2 * i + 2

        # 왼쪽 자식이 더 큰 경우
        if left < n and arr[left] > arr[largest]:
            largest = left

        # 오른쪽 자식이 더 큰 경우
        if right < n and arr[right] > arr[largest]:
            largest = right

        # largest가 root가 아닌 경우 교환 후 재귀
        if largest != i:
            arr[i], arr[largest] = arr[largest], arr[i]
            heapify(arr, n, largest)

    n = len(arr)

    # 최대 힙 구성
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)

    # 힙에서 요소를 하나씩 추출
    for i in range(n - 1, 0, -1):
        arr[i], arr[0] = arr[0], arr[i]  # 최대 요소를 끝으로 이동
        heapify(arr, i, 0)

    return arr
```

#### 4.3.2 기수 정렬 (Radix Sort) - O(n)
```python
def radix_sort(arr):
    # 최대 자릿수 찾기
    max_num = max(arr)
    max_digits = 0
    while max_num > 0:
        max_num //= 10
        max_digits += 1

    # 각 자릿수에 대해 카운팅 정렬
    for digit in range(max_digits):
        buckets = [[] for _ in range(10)]

        for num in arr:
            digit_value = (num // (10 ** digit)) % 10
            buckets[digit_value].append(num)

        # 버킷에서 요소들을 다시 arr로
        arr = []
        for bucket in buckets:
            arr.extend(bucket)

    return arr
```

**비교 기반 vs 비비교 기반 정렬:**

| 분류 | 시간 복잡도 | 안정성 | 장점 | 단점 |
| --- | --- | --- | --- | --- |
| **비교 기반** | Ω(n log n) | 다양함 | 범용성 | 최선 복잡도 제한 |
| **비비교 기반** | O(n) | 가능 | 빠름 | 특정 데이터에 한정 |

---

## 5. 검색 알고리즘 (Search Algorithms)

### 5.1 선형 검색 (Linear Search) - O(n)

**🔍 비유: 방 안에서 잃어버린 열쇠를 한 곳씩 뒤지는 것**

**단계별 작동 방식:**
```
배열: [5, 3, 8, 2, 1, 9, 4, 7]
찾을 값: 2

1단계: 첫 번째 요소 확인
   인덱스 0: 5 → 5 ≠ 2 → 다음으로

2단계: 두 번째 요소 확인
   인덱스 1: 3 → 3 ≠ 2 → 다음으로

3단계: 세 번째 요소 확인
   인덱스 2: 8 → 8 ≠ 2 → 다음으로

4단계: 네 번째 요소 확인
   인덱스 3: 2 → 2 = 2 → 찾음! 인덱스 3 반환

최악의 경우: 마지막 요소까지 확인해야 함
```

```python
def linear_search(arr, target):
    for i in range(len(arr)):  # 처음부터 끝까지 하나씩 확인
        if arr[i] == target:    # 찾는 값과 같으면
            return i            # 인덱스 반환
    return -1                   # 못 찾으면 -1 반환
```

**💡 특징 정리:**
- ✅ **장점**: 구현이 매우 간단함, 정렬되지 않은 데이터에서도 사용 가능
- ❌ **단점**: 느림! 데이터가 많아지면 선형으로 느려짐
- 🎯 **사용처**: 작은 데이터, 정렬되지 않은 데이터

### 5.2 이진 검색 (Binary Search) - O(log n)

**📖 비유: 사전에서 단어를 찾는 것 (중간 페이지를 펼쳐서 앞/뒤 결정)**

**⚠️ 필수 조건: 배열이 정렬되어 있어야 함!**

**단계별 작동 방식:**
```
정렬된 배열: [1, 2, 3, 4, 5, 7, 8, 9]
찾을 값: 7

1단계: 중간값 확인
   left=0, right=7, mid=3
   arr[3] = 4
   4 < 7 → 왼쪽 범위 버림, left = 4

   범위: [1, 2, 3, 4, 5, 7, 8, 9]
          ↑         ↑
         left      right

2단계: 새로운 범위에서 중간값 확인
   left=4, right=7, mid=5
   arr[5] = 7
   7 = 7 → 찾음! 인덱스 5 반환

총 비교 횟수: 2번 (8개 데이터 중)
```

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1  # 검색 범위 초기화

    while left <= right:  # 범위가 유효할 때까지
        mid = (left + right) // 2  # 중간 인덱스 계산

        if arr[mid] == target:     # 찾았으면
            return mid             # 인덱스 반환

        elif arr[mid] < target:    # 중간값보다 크면
            left = mid + 1         # 왼쪽 범위 늘림

        else:                      # 중간값보다 작으면
            right = mid - 1        # 오른쪽 범위 줄임

    return -1  # 못 찾으면 -1 반환
```

**💡 특징 정리:**
- ✅ **장점**: 매우 빠름! 데이터 2배 늘어도 연산 1번만 늘어남
- ❌ **단점**: 반드시 정렬된 데이터에서만 사용 가능
- 🎯 **사용처**: 정렬된 큰 데이터셋 검색

**📊 검색 범위 시각화:**
```
찾을 값: 7
초기: [1, 2, 3, 4, 5, 7, 8, 9]  ← 전체 범위

1회: 중간값 4 < 7 → 오른쪽만 검색
      [1, 2, 3, 4, 5, 7, 8, 9]  ← 왼쪽 버림

2회: 중간값 7 = 7 → 찾음!
      [5, 7, 8, 9]               ← 최종 범위
```

**재귀 버전:**
```python
def binary_search_recursive(arr, target, left=0, right=None):
    if right is None:
        right = len(arr) - 1

    if left > right:
        return -1

    mid = (left + right) // 2

    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search_recursive(arr, target, mid + 1, right)
    else:
        return binary_search_recursive(arr, target, left, mid - 1)
```

### 5.3 해시 기반 검색 (Hash Search) - O(1) 평균
```python
class HashTable:
    def __init__(self, size=100):
        self.size = size
        self.table = [[] for _ in range(size)]  # 충돌 처리를 위한 리스트

    def _hash(self, key):
        return hash(key) % self.size

    def insert(self, key, value):
        index = self._hash(key)
        # 충돌 발생 시 리스트에 추가
        for pair in self.table[index]:
            if pair[0] == key:
                pair[1] = value
                return
        self.table[index].append([key, value])

    def search(self, key):
        index = self._hash(key)
        for pair in self.table[index]:
            if pair[0] == key:
                return pair[1]
        return None
```

---

## 6. 재귀와 분할 정복 (Recursion & Divide and Conquer)

### 6.1 재귀의 기초

**재귀 함수의 구성 요소:**
1. **기본 케이스 (Base Case):** 재귀 호출을 멈추는 조건
2. **재귀 케이스 (Recursive Case):** 자신을 호출하는 부분
3. **진행 방향:** 문제를 더 작은 문제로 분해

```python
def factorial(n):
    # 기본 케이스
    if n <= 1:
        return 1
    # 재귀 케이스
    return n * factorial(n - 1)
```

### 6.2 분할 정복 알고리즘

#### 6.2.1 최대 부분합 문제 (Maximum Subarray Sum)
```python
def max_subarray_sum(arr):
    def divide_and_conquer(left, right):
        if left == right:
            return arr[left]

        mid = (left + right) // 2

        # 왼쪽 부분의 최대 부분합
        left_max = float('-inf')
        current_sum = 0
        for i in range(mid, left - 1, -1):
            current_sum += arr[i]
            left_max = max(left_max, current_sum)

        # 오른쪽 부분의 최대 부분합
        right_max = float('-inf')
        current_sum = 0
        for i in range(mid + 1, right + 1):
            current_sum += arr[i]
            right_max = max(right_max, current_sum)

        # 가운데를 포함한 최대 부분합
        cross_max = left_max + right_max

        # 세 가지 경우 중 최대값 반환
        return max(
            divide_and_conquer(left, mid),
            divide_and_conquer(mid + 1, right),
            cross_max
        )

    if not arr:
        return 0
    return divide_and_conquer(0, len(arr) - 1)
```

### 6.3 재귀의 장단점

**장점:**
- 코드가 간결하고 이해하기 쉬움
- 복잡한 문제를 단순하게 표현

**단점:**
- 스택 오버플로우 위험
- 메모리 사용량 증가
- 디버깅이 어려움

**꼬리 재귀 최적화:**
```python
# 꼬리 재귀 (Tail Recursion)
def factorial_tail(n, accumulator=1):
    if n <= 1:
        return accumulator
    return factorial_tail(n - 1, n * accumulator)  # 마지막 연산이 재귀 호출
```

---

## 7. 동적 프로그래밍 (Dynamic Programming)

### 7.1 동적 프로그래밍의 원칙

**중복되는 부분 문제 (Overlapping Subproblems):**
- 큰 문제를 작은 문제로 분해할 때 동일한 부분 문제가 반복

**최적 부분 구조 (Optimal Substructure):**
- 부분 문제의 최적해가 전체 문제의 최적해를 구성

### 7.2 동적 프로그래밍의 마법: 피보나치 수열로 이해하기

**🐰 문제: 토끼 번식 수열 계산**
```
피보나치 수열: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...
규칙: F(n) = F(n-1) + F(n-2)
```

**❌ 방법 1: 순진한 재귀 (최악! O(2^n))**
```python
def fibonacci_bad(n):
    if n <= 1:
        return n
    return fibonacci_bad(n - 1) + fibonacci_bad(n - 2)

# F(5)를 계산하기 위해 F(4)와 F(3)을 각각 계산
# F(4)는 F(3)과 F(2)를, F(3)은 F(2)와 F(1)을...
# 같은 계산을 수십 번 반복! 💥
```

**계산 트리 시각화 (F(5) 계산):**
```
F(5) = F(4) + F(3)
     ├─ F(4) = F(3) + F(2)
     │    ├─ F(3) = F(2) + F(1)
     │    │    ├─ F(2) = F(1) + F(0) = 1 + 0 = 1
     │    │    └─ F(1) = 1
     │    └─ F(2) = F(1) + F(0) = 1 + 0 = 1  ← 중복 계산!
     └─ F(3) = F(2) + F(1)
          ├─ F(2) = F(1) + F(0) = 1 + 0 = 1  ← 또 중복!
          └─ F(1) = 1
```

**✅ 방법 2: 메모이제이션 (Memoization) - O(n)**
```python
def fibonacci_dp(n, memo=None):
    if memo is None:
        memo = {}  # 계산 결과를 저장할 메모장

    if n in memo:          # 이미 계산했으면
        return memo[n]      # 메모장에서 꺼내기

    if n <= 1:            # 기저 조건
        return n

    # 계산해서 메모장에 저장
    memo[n] = fibonacci_dp(n - 1, memo) + fibonacci_dp(n - 2, memo)
    return memo[n]

# F(5) 계산 시:
# F(2), F(3), F(4)는 한 번만 계산되고 메모장에 저장됨!
```

**✅ 방법 3: 상향식 동적 프로그래밍 (Tabulation) - O(n)**
```python
def fibonacci_iterative(n):
    if n <= 1:
        return n

    dp = [0] * (n + 1)  # DP 테이블 (메모장)
    dp[1] = 1           # 초기값 설정

    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]  # 작은 문제부터 차례로 해결

    return dp[n]

# 계산 과정:
# dp[2] = dp[1] + dp[0] = 1 + 0 = 1
# dp[3] = dp[2] + dp[1] = 1 + 1 = 2
# dp[4] = dp[3] + dp[2] = 2 + 1 = 3
# dp[5] = dp[4] + dp[3] = 3 + 2 = 5 ✓
```

**📊 성능 비교:**
| 방법 | F(10) | F(20) | F(30) | F(40) |
|------|-------|-------|-------|-------|
| 재귀 | 0.0001초 | 0.1초 | 1분 | 불가능 |
| DP | 0.00001초 | 0.0001초 | 0.0001초 | 0.0001초 |

**💡 동적 프로그래밍의 핵심 원칙:**
1. **중복 부분 문제**: 같은 계산을 반복하지 말자
2. **최적 부분 구조**: 큰 문제의 답 = 작은 문제들의 답 조합
3. **메모이제이션**: 계산 결과를 저장하고 재사용하자

### 7.3 대표적인 동적 프로그래밍 문제

#### 7.3.1 0/1 배낭 문제 (0/1 Knapsack)
```python
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0 for _ in range(capacity + 1)] for _ in range(n + 1)]

    for i in range(1, n + 1):
        for w in range(1, capacity + 1):
            if weights[i-1] <= w:
                # 물건을 넣는 경우 vs 넣지 않는 경우
                dp[i][w] = max(
                    values[i-1] + dp[i-1][w - weights[i-1]],  # 넣음
                    dp[i-1][w]  # 넣지 않음
                )
            else:
                dp[i][w] = dp[i-1][w]

    return dp[n][capacity]
```

#### 7.3.2 최장 공통 부분 수열 (LCS)
```python
def longest_common_subsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    return dp[m][n]
```

---

## 8. 그리디 알고리즘 (Greedy Algorithms)

### 8.1 그리디 알고리즘의 특징

**국소 최적화 선택:**
- 각 단계에서 가장 좋은 선택을 하는 알고리즘
- 미래를 고려하지 않고 현재 최적의 선택만 함

### 8.2 대표 예시: 거스름돈 문제

```python
def greedy_coin_change(amount, coins):
    coins.sort(reverse=True)  # 큰 동전부터 사용
    result = []

    for coin in coins:
        while amount >= coin:
            amount -= coin
            result.append(coin)

    return result

# 예시
print(greedy_coin_change(63, [1, 5, 10, 25]))  # [25, 25, 10, 1, 1, 1]
```

**그리디가 최적해를 보장하는 경우:**
- 미국 동전 시스템 (1, 5, 10, 25 센트)
- 작업 스케줄링 문제

### 8.3 그리디 vs 동적 프로그래밍

| 측면 | 그리디 알고리즘 | 동적 프로그래밍 |
| --- | --- | --- |
| **접근 방식** | 국소 최적화 | 전역 최적화 |
| **시간 복잡도** | 일반적으로 빠름 | 상대적으로 느림 |
| **정확성** | 일부 문제에서만 최적 | 대부분 문제에서 최적 |
| **메모리 사용** | 적음 | 많음 |

---

## 9. 그래프 알고리즘 (Graph Algorithms)

### 9.1 그래프 표현 방식

**인접 리스트:**
```python
# 방향 그래프
graph = {
    'A': ['B', 'C'],
    'B': ['D', 'E'],
    'C': ['F'],
    'D': [],
    'E': ['F'],
    'F': []
}
```

**인접 행렬:**
```python
# 6개 노드 그래프
adj_matrix = [
    [0, 1, 1, 0, 0, 0],  # A
    [0, 0, 0, 1, 1, 0],  # B
    [0, 0, 0, 0, 0, 1],  # C
    [0, 0, 0, 0, 0, 0],  # D
    [0, 0, 0, 0, 0, 1],  # E
    [0, 0, 0, 0, 0, 0]   # F
]
```

### 9.2 그래프 탐색 알고리즘

#### 9.2.1 깊이 우선 탐색 (DFS)
```python
def dfs(graph, start, visited=None):
    if visited is None:
        visited = set()

    visited.add(start)
    print(start, end=' ')  # 방문한 노드 출력

    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)

# 사용 예시
graph = {
    'A': ['B', 'C'],
    'B': ['D', 'E'],
    'C': ['F'],
    'D': [],
    'E': ['F'],
    'F': []
}

dfs(graph, 'A')  # A B D E F C
```

#### 9.2.2 너비 우선 탐색 (BFS)
```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue = deque([start])
    visited.add(start)

    while queue:
        node = queue.popleft()
        print(node, end=' ')  # 방문한 노드 출력

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

# 사용 예시 (동일 그래프)
bfs(graph, 'A')  # A B C D E F
```

### 9.3 최단 경로 알고리즘

#### 9.3.1 다익스트라 알고리즘 (Dijkstra's Algorithm)
```python
import heapq

def dijkstra(graph, start):
    # 거리 초기화
    distances = {node: float('inf') for node in graph}
    distances[start] = 0

    # 우선순위 큐: (거리, 노드)
    pq = [(0, start)]

    while pq:
        current_distance, current_node = heapq.heappop(pq)

        # 이미 더 좋은 경로를 찾았으면 건너뜀
        if current_distance > distances[current_node]:
            continue

        for neighbor, weight in graph[current_node].items():
            distance = current_distance + weight

            # 더 좋은 경로를 찾았으면 업데이트
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))

    return distances

# 가중 그래프 예시
weighted_graph = {
    'A': {'B': 4, 'C': 2},
    'B': {'D': 5, 'E': 10},
    'C': {'B': 1, 'F': 3},
    'D': {},
    'E': {'F': 2},
    'F': {}
}

print(dijkstra(weighted_graph, 'A'))
# {'A': 0, 'B': 3, 'C': 2, 'D': 8, 'E': 13, 'F': 5}
```

---

## 10. 알고리즘 설계 패러다임 비교

| 패러다임 | 접근 방식 | 시간 복잡도 | 예시 알고리즘 |
| --- | --- | --- | --- |
| **분할 정복** | 문제를 작은 부분으로 분해 | 종종 O(n log n) | 퀵 정렬, 병합 정렬 |
| **동적 프로그래밍** | 부분 문제의 해를 재사용 | 종종 O(n²) | 피보나치, LCS |
| **그리디** | 각 단계에서 최적 선택 | 종종 O(n) | 허프만 코딩, MST |
| **백트래킹** | 모든 가능한 경우 탐색 | 지수 시간 | N-퀸, 서브셋 |
| **분기 한정** | 탐색 공간 가지치기 | 다양함 | TSP, 배낭 문제 |

---

## 11. 실무 적용 사례

### 11.1 구글 검색 엔진의 PageRank 알고리즘

**개념:**
- 웹 페이지의 중요도를 계산하는 알고리즘
- 링크 구조를 그래프로 모델링

**기본 아이디어:**
```python
def pagerank_basic(graph, damping_factor=0.85, iterations=100):
    # 초기 PageRank 값 설정
    num_pages = len(graph)
    pagerank = {page: 1/num_pages for page in graph}

    for _ in range(iterations):
        new_pagerank = {}
        for page in graph:
            # 랜덤 서핑 + 링크 기반 점수
            rank_sum = sum(pagerank[linking_page] / len(graph[linking_page])
                          for linking_page in graph
                          if page in graph[linking_page])

            new_pagerank[page] = (1 - damping_factor) / num_pages + damping_factor * rank_sum

        pagerank = new_pagerank

    return pagerank
```

### 11.2 유튜브 추천 알고리즘의 기초

**협업 필터링 기반 추천:**
```python
import numpy as np

def collaborative_filtering(user_item_matrix):
    # 사용자-아이템 행렬 분해
    U, sigma, Vt = np.linalg.svd(user_item_matrix)

    # 잠재 요인 추출
    k = 50  # 잠재 요인 개수
    U_k = U[:, :k]
    sigma_k = np.diag(sigma[:k])
    Vt_k = Vt[:k, :]

    # 예측 평점 계산
    predicted_ratings = np.dot(np.dot(U_k, sigma_k), Vt_k)

    return predicted_ratings
```

### 11.3 네비게이션 앱의 경로 최적화

**A* 알고리즘 적용:**
```python
import heapq

def a_star(graph, start, goal, heuristic):
    open_set = []
    heapq.heappush(open_set, (0, start))

    came_from = {}
    g_score = {node: float('inf') for node in graph}
    g_score[start] = 0

    f_score = {node: float('inf') for node in graph}
    f_score[start] = heuristic(start, goal)

    while open_set:
        current = heapq.heappop(open_set)[1]

        if current == goal:
            return reconstruct_path(came_from, current)

        for neighbor in graph[current]:
            tentative_g_score = g_score[current] + graph[current][neighbor]

            if tentative_g_score < g_score[neighbor]:
                came_from[neighbor] = current
                g_score[neighbor] = tentative_g_score
                f_score[neighbor] = g_score[neighbor] + heuristic(neighbor, goal)

                if neighbor not in [item[1] for item in open_set]:
                    heapq.heappush(open_set, (f_score[neighbor], neighbor))

    return None
```

---

## 12. 알고리즘 최적화 기법

### 12.1 캐시 친화적 알고리즘 설계

**공간 지역성 활용:**
```python
# 캐시 친화적인 행렬 곱셈 (블록 단위 처리)
def matrix_multiply_cache_friendly(A, B, block_size=64):
    n = len(A)
    C = [[0 for _ in range(n)] for _ in range(n)]

    for ii in range(0, n, block_size):
        for jj in range(0, n, block_size):
            for kk in range(0, n, block_size):
                # 블록 단위로 처리하여 캐시 히트율 향상
                for i in range(ii, min(ii + block_size, n)):
                    for j in range(jj, min(jj + block_size, n)):
                        for k in range(kk, min(kk + block_size, n)):
                            C[i][j] += A[i][k] * B[k][j]

    return C
```

### 12.2 병렬 알고리즘

**맵리듀스 패턴:**
```python
from multiprocessing import Pool

def map_reduce_parallel(data, map_func, reduce_func):
    # 병렬 맵 연산
    with Pool() as pool:
        mapped = pool.map(map_func, data)

    # 리듀스 연산
    result = reduce_func(mapped)
    return result

# 예시: 단어 카운팅
def word_count_map(text):
    words = text.split()
    return [(word, 1) for word in words]

def word_count_reduce(mapped_results):
    word_counts = {}
    for result in mapped_results:
        for word, count in result:
            word_counts[word] = word_counts.get(word, 0) + count
    return word_counts
```

### 12.3 근사 알고리즘

**NP-완전 문제에 대한 실용적 해결:**
```python
# TSP 근사 알고리즘 (최소 신장 트리 기반)
def tsp_approximation(graph):
    # 최소 신장 트리 계산 (크루스칼 알고리즘)
    mst = kruskal_mst(graph)

    # 프리오더 트리 순회로 TSP 경로 생성
    path = preorder_traversal(mst)

    # 시작점으로 돌아오는 순환 경로 생성
    path.append(path[0])

    return path, calculate_path_cost(graph, path)
```

---

## 13. 디버깅과 성능 분석

### 13.1 알고리즘 디버깅 기법

**단정문 사용:**
```python
def binary_search_debug(arr, target):
    left, right = 0, len(arr) - 1
    iterations = 0

    while left <= right:
        iterations += 1
        mid = (left + right) // 2

        # 디버깅 정보 출력
        print(f"반복 {iterations}: left={left}, right={right}, mid={mid}, arr[mid]={arr[mid]}")

        assert left <= mid <= right, "인덱스 범위 오류"
        assert arr[left-1] <= arr[mid] <= arr[right+1] if left > 0 and right < len(arr)-1 else True, "정렬 상태 위반"

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

### 13.2 프로파일링 도구 활용

**Python cProfile:**
```python
import cProfile
import pstats

def profile_algorithm():
    # 알고리즘 실행
    result = your_algorithm_function()

    return result

# 프로파일링 실행
profiler = cProfile.Profile()
profiler.enable()
result = profile_algorithm()
profiler.disable()

# 결과 분석
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats(10)  # 상위 10개 함수 출력
```

---

## 14. 결론 및 학습 포인트

### 14.1 알고리즘 선택의 지침

**문제 유형별 추천:**

| 문제 유형 | 추천 알고리즘 | 시간 복잡도 |
| --- | --- | --- |
| **정렬된 데이터 검색** | 이진 검색 | O(log n) |
| **데이터 정렬** | 퀵 정렬 (일반), 병합 정렬 (안정성) | O(n log n) |
| **최단 경로** | 다익스트라 (양수 가중치), 벨만-포드 (음수) | O((V+E) log V) |
| **그래프 탐색** | BFS (최단 경로), DFS (탐색) | O(V+E) |
| **부분합 최대** | 카데인 알고리즘 | O(n) |

### 14.2 실무 개발자를 위한 조언

**1. 올바른 도구 선택:**
- 작은 데이터셋: 간단한 알고리즘
- 큰 데이터셋: 효율적인 알고리즘
- 실시간 요구사항: 최적화된 알고리즘

**2. 성능 측정의 중요성:**
- 이론적 복잡도 ≠ 실제 성능
- 프로파일링으로 병목 지점 식별
- 캐시 효율성과 메모리 접근 패턴 고려

**3. 확장성 고려:**
- 데이터 크기가 증가할 때 성능 변화 예측
- 분산 처리와 병렬화 가능성 검토

**4. 코드 품질 유지:**
- 가독성 있는 알고리즘 구현
- 적절한 주석과 문서화
- 단위 테스트와 성능 테스트

---

## 15. 초보자 Q&A: 자주 하는 질문들

### Q1. "어떤 정렬 알고리즘을 써야 하나요?"
**A:** 상황에 따라 다릅니다!
- **일반적인 경우**: 퀵 정렬 (평균적으로 가장 빠름)
- **안정 정렬 필요**: 병합 정렬 (순서 유지)
- **거의 정렬된 데이터**: 삽입 정렬 (O(n)에 가까움)
- **메모리 제한적**: 힙 정렬 (제자리 정렬)

### Q2. "빅오가 중요한 이유가 뭔가요?"
**A:** 코드의 확장성을 예측하기 위해서입니다.
```
예: 100개 데이터에서 O(n²) 알고리즘이 1초 걸림
     1,000개 데이터에서는? → 100초 (사용 불가!)
     같은 작업을 O(n log n)으로 하면? → 약 2초 (사용 가능!)
```

### Q3. "재귀가 왜 위험한가요?"
**A:** 스택 오버플로우 때문입니다.
```python
def bad_recursion(n):
    if n == 0:
        return 0
    return bad_recursion(n - 1) + 1

# n이 10,000이면? → RecursionError!
# Python의 기본 스택 제한을 초과함
```

### Q4. "동적 프로그래밍은 언제 사용하나요?"
**A:** 최적화 문제가 있고, 중복 계산이 많을 때
- ✅ 피보나치, LCS, 배낭 문제
- ✅ 최단 경로, 최대 부분합
- ❌ 단순 계산, 한 번만 계산하는 문제

### Q5. "실무에서 알고리즘이 얼마나 중요하나요?"
**A:** 매우 중요합니다!
```
실제 사례:
- 구글 검색: PageRank 알고리즘으로 웹페이지 순위 결정
- 네비게이션: A* 알고리즘으로 최적 경로 찾기
- 추천 시스템: 협업 필터링으로 사용자 맞춤 추천
- 소셜 미디어: 그래프 알고리즘으로 친구 추천
```

### Q6. "어디서 알고리즘을 더 공부할 수 있나요?"
**A:**
- **LeetCode**: 실전 코딩 인터뷰 연습
- **GeeksforGeeks**: 상세한 알고리즘 설명
- **CLRS 책**: 컴퓨터 알고리즘의 바이블
- **Coursera**: 무료 온라인 코스

---

*"알고리즘은 컴퓨터 과학의 심장이다. 올바른 알고리즘 선택이 우수한 소프트웨어의 출발점이다."*

> **학습 팁**: 이론보다 실천이 중요합니다. 작은 문제부터 직접 구현해보세요!

> 알고리즘은 단순한 코딩 기술이 아닌 문제 해결의 체계적 방법론입니다. 다양한 알고리즘을 이해하고 상황에 맞게 적용하는 능력이 진정한 개발자의 역량입니다.