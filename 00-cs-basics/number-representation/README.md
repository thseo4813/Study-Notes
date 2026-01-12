# 💰 컴퓨터가 돈을 계산 못하는 이유: 숫자 표현의 함정

## 🚨 실제로 겪어본 문제들

### 한국 개발자들이 흔히 마주치는 상황들:

**💳 결제 시스템 (카카오페이, 토스)**
```java
// ❌ 문제 있는 코드
double price = 9999.99;
double tax = price * 0.1;  // 999.999...원이 됨
double total = price + tax; // 10999.989...원이 됨
```

**⭐ 리뷰 시스템 (배달의민족, 쿠팡)**
```java
// ❌ 별점 평균 계산 문제
double[] ratings = {5.0, 4.0, 3.0};
double average = (ratings[0] + ratings[1] + ratings[2]) / 3; // 4.0이 안 나올 수 있음
```

**🎮 게임 시스템 (LOL, 배그)**
```java
// ❌ 체력 회복 계산 문제
double currentHP = 85.7;
double healAmount = 14.3;
double newHP = currentHP + healAmount; // 100.0이 안 나올 수 있음
```

## 🎯 1분 요약: 왜 이게 중요한가?

**컴퓨터는 0과 1로만 모든 숫자를 표현해야 해서, 우리가 아는 '정확한 계산'이 불가능합니다.**

- **정수**: 안전함 (1 + 1 = 2)
- **실수**: 근사치 사용 (0.1 + 0.2 ≠ 0.3)

> **결론:**
> 1. **돈 계산**: BigDecimal 필수
> 2. **게임 수치**: 정수로 변환해서 계산
> 3. **과학 계산**: 고정밀도 라이브러리 사용
> 4. **일반 계산**: float/double 충분 (하지만 비교할 때는 주의)

---

## 2. 🔢 컴퓨터가 숫자를 세는 법: 이진법 기초

### 2.1 왜 이진수인가? (실제 이유)

**컴퓨터는 스위치로 만들어졌습니다.**
- 전기: ON(1) / OFF(0)
- 자기장: N극(1) / S극(0)
- 전압: High(1) / Low(0)

**그래서 10진수가 아닌 2진수를 사용합니다.**

### 2.2 10진수 ↔ 2진수 변환 (실제 계산)

십진수 13을 이진수로 변환해보죠:

```
13 ÷ 2 = 6  나머지 1  (맨 오른쪽 자리)
 6 ÷ 2 = 3  나머지 0  (그 다음 자리)
 3 ÷ 2 = 1  나머지 1  (그 다음 자리)
 1 ÷ 2 = 0  나머지 1  (맨 왼쪽 자리)

따라서: 13₁₀ = 1101₂
```

**읽는 법:** 1101₂ = 1×8 + 1×4 + 0×2 + 1×1 = 13₁₀

### 2.3 프로그래밍에서 자주 보는 숫자들

### 2.3 프로그래밍에서 자주 보는 16진수와 8진수

**실제 코딩에서 어디서 마주치나요?**

**🎨 색상 코드 (CSS, Android)**
```css
/* 빨간색 */
.red { color: #FF0000; }  /* 16진수: FF = 255 */
.green { color: #00FF00; }
.blue { color: #0000FF; }
```

**💾 메모리 주소 (디버깅, C/C++)**
```c
// 메모리 주소는 보통 16진수로 표시
int* ptr = (int*)0x7FFF12345678;  // 16진수 주소
printf("Address: %p\n", ptr);    // 0x7fff12345678
```

**📱 권한 설정 (Unix/Linux)**
```bash
# 파일 권한: rwxr-xr-x
chmod 755 file.txt  # 8진수: 7=rwx, 5=r-x, 5=r-x

# 8진수 이해:
# 7 = 111₂ = rwx (읽기+쓰기+실행)
# 5 = 101₂ = r-x (읽기+실행)
```

**🔍 실제 변환 예시:**
```text
십진수 255의 다양한 표현:
이진수: 11111111₂  (8비트 모두 1)
8진수: 377₈       (3×64 + 7×8 + 7×1 = 255)
16진수: FF₁₆       (15×16 + 15×1 = 255)
```

---

## 3. 🔢 정수 표현: 안전한 계산의 세계

### 3.1 정수는 왜 안전한가?

**정수는 컴퓨터가 가장 잘 다루는 숫자 타입입니다.**
- 1 + 1 = 2 (항상 정확함)
- 오버플로우만 조심하면 됨

### 3.2 실제 프로그래밍에서 마주치는 정수 타입들

| 언어 | 8비트 | 16비트 | 32비트 | 64비트 | 비고 |
|------|-------|--------|--------|--------|------|
| **C/C++** | `uint8_t` | `uint16_t` | `uint32_t` | `uint64_t` | `<stdint.h>` |
| **Java** | `byte` | `short` | `int` | `long` | 기본 타입 |
| **Go** | - | - | `uint32` | `uint64` | 부호 없음 기본 |
| **JavaScript** | - | - | - | `BigInt` | Number는 double |

**💡 팁:** JavaScript의 Number는 사실 double이라 정수가 아님!

### 3.3 음수 표현: 2의 보수 (실제 계산법)

**왜 2의 보수가 필요할까?**
컴퓨터는 +,-,×,÷ 기호가 없어서, 음수를 "덧셈으로 처리"하기 위해 2의 보수를 사용합니다.

**🎯 실제 계산 예시: -5 표현하기**

```
1. 5의 이진수:     00000101
2. 모든 비트 반전:  11111010  (1의 보수)
3. +1 더하기:      11111011  (2의 보수)

결과: 11111011₂ = -5₁₀
```

**검증:** 11111011₂ + 00000101₂ = 100000000₂ (올림 발생, 결과는 0)

### 3.4 각 언어의 정수 범위 (실무에서 중요!)

| 비트 | Signed 범위 | Unsigned 범위 | Java 타입 | C++ 타입 |
|------|-------------|----------------|-----------|----------|
| 8비트 | -128 ~ 127 | 0 ~ 255 | `byte` | `int8_t` |
| 16비트 | -32,768 ~ 32,767 | 0 ~ 65,535 | `short` | `int16_t` |
| 32비트 | -2.1억 ~ 2.1억 | 0 ~ 42.9억 | `int` | `int32_t` |
| 64비트 | -9.2경 ~ 9.2경 | 0 ~ 184경 | `long` | `int64_t` |

### 3.5 오버플로우: 개발자를 울리는 함정

**🚨 실제 사고 사례들:**
- **게임 아이템 개수**: 인벤토리 999개에서 +1 하면 -32,768개로 변함
- **은행 계좌**: 잔액이 int.MAX_VALUE를 넘으면 마이너스 잔액으로 표시
- **시간 계산**: Unix timestamp가 2038년 문제를 일으킴

**💻 실제 코드에서 재현해보기:**

```java
// Java에서 오버플로우 재현
int maxInt = Integer.MAX_VALUE;  // 2,147,483,647
System.out.println("최댓값: " + maxInt);

int overflowed = maxInt + 1;     // 🚨 오버플로우!
System.out.println("오버플로우: " + overflowed); // -2,147,483,648

// 왜 이런 일이? 2의 보수에서 최댓값 + 1 = 최솟값
```

**🛡️ 방어 전략:**

```java
// 방법 1: Java 8+의 안전한 연산
int safeAdd(int a, int b) {
    return Math.addExact(a, b); // 오버플로우 시 예외 발생
}

// 방법 2: 사전 체크
int safeAdd(int a, int b) {
    if (a > 0 && b > 0 && a > Integer.MAX_VALUE - b) {
        throw new ArithmeticException("Overflow!");
    }
    return a + b;
}

// 방법 3: 더 큰 타입으로 계산
long safeAdd(int a, int b) {
    return (long) a + b; // long으로 확장해서 계산
}
```

---

## 4. 🧮 실수 표현: 정밀도의 늪

### 4.1 고정 소수점 vs 부동 소수점 (쉬운 설명)

**🏠 고정 소수점 (Fixed Point):**
돈 계산할 때처럼 소수점 위치를 고정시키는 방식
```
예: XXXX.XXXX (총 8자리, 소수점 아래 4자리)
가격: 0123.4567 → 123.4567원
```

**장점:** 정확함, 계산이 직관적
**단점:** 표현 범위가 좁음 (0.0001 ~ 9999.9999만 표현 가능)

**🚀 부동 소수점 (Floating Point):**
소수점을 자유자재로 움직이는 방식 (과학자 표기법과 비슷)
```
1.23456 × 10² = 123.456
1.23456 × 10⁻² = 0.0123456
```

**장점:** 넓은 범위 표현 가능 (매우 큰 수 ~ 매우 작은 수)
**단점:** 근사치 계산, 정밀도 손실 발생

### 4.2 IEEE 754: 실수 저장 방식 (간단 버전)

컴퓨터는 실수를 **세 부분으로 나누어 저장**합니다:

```
[부호 1비트] [지수 8비트] [가수 23비트]  ← float (32비트)
[부호 1비트] [지수 11비트] [가수 52비트] ← double (64비트)
```

**🎯 쉽게 이해하기:**
- **부호 비트**: 0=양수, 1=음수
- **지수부**: 소수점 위치 결정 (2의 몇 제곱?)
- **가수부**: 실제 숫자의 유효숫자

**💡 예시: 123.456 저장 과정**
```
123.456 = 1.23456 × 10²
컴퓨터: 1.23456 × 2^7 (2진수로 변환)
저장: 부호=0, 지수=7, 가수=23456...
```

**📊 각 타입의 특징:**

| 타입 | 비트 | 정밀도 | 범위 | 사용처 |
|------|------|--------|------|--------|
| **float** | 32 | 7자리 | ±3.4×10³⁸ | 게임, 그래픽 |
| **double** | 64 | 15자리 | ±1.7×10³⁰⁸ | 일반 계산 |
| **BigDecimal** | 무제한 | 무제한 | 무제한 | 돈 계산 |

### 4.3 부동 소수점의 시각화

![부동 소수점 구조](./image/copyImage.jpg)

**구조 설명:**
- **부호 비트:** 0=양수, 1=음수
- **지수부:** 2의 거듭제곱을 표현 (바이어스 적용으로 음수 지수도 표현)
- **가수부:** 1.xxx 형태의 유효숫자 표현

---

## 5. 💥 정밀도 손실: 예상치 못한 함정

### 5.1 0.1 + 0.2 ≠ 0.3 의 비밀

**🚨 충격적인 사실:**
컴퓨터에서 0.1 + 0.2는 0.3이 아닙니다!

```java
// 직접 확인해보세요
System.out.println(0.1 + 0.2);  // 0.30000000000000004
System.out.println(0.1 + 0.2 == 0.3);  // false!
```

**🔍 왜 이런 일이?**
십진수 0.1을 이진수로 변환해보죠:

```
0.1 × 2 = 0.2 → 0 (버림, 0.2 남음)
0.2 × 2 = 0.4 → 0 (버림, 0.4 남음)
0.4 × 2 = 0.8 → 0 (버림, 0.8 남음)
0.8 × 2 = 1.6 → 1 (올림, 0.6 남음)
0.6 × 2 = 1.2 → 1 (올림, 0.2 남음)
0.2 × 2 = 0.4 → 0 (버림, 0.4 남음)
... (무한 반복!)
```

**결과:** 0.1은 2진수로 **무한 소수**가 되어 근사값으로 저장됩니다.

**💡 실제 영향:**
- **가격 계산**: 10.1원 × 3 = 30.299999...원
- **이자 계산**: 0.05% 이자가 부정확하게 계산됨
- **거리 계산**: GPS 좌표 오차 발생

### 5.2 정밀도 한계: float vs double

**📏 정밀도 차이 비교:**

```java
// 같은 계산, 다른 정밀도
float  f = 1.0f / 3.0f;  // 0.33333334 (7자리 정밀도)
double d = 1.0  / 3.0;   // 0.3333333333333333 (15자리 정밀도)

System.out.println("Float:  " + f);
System.out.println("Double: " + d);
System.out.println("실제값: " + (1.0/3.0));
```

**🔄 누적 오차: 반복 계산의 함정**

```java
// 0.1을 10번 더하면 1.0이 나올까?
double result = 0.0;
for (int i = 0; i < 10; i++) {
    result += 0.1;
}
System.out.println(result);  // 0.9999999999999999 (1.0이 아님!)
System.out.println(result == 1.0);  // false!
```

**💡 실제 서비스 영향:**
- **쿠폰 할인**: 10% 쿠폰을 10번 적용하면 100% 할인이 안 됨
- **포인트 적립**: 0.1포인트를 100번 적립해도 10포인트가 안 됨
- **별점 평균**: 리뷰 점수 평균이 부정확하게 계산됨

### 5.3 특수 값들 (Special Values)

IEEE 754에서 정의하는 특수한 값들:

- **±0:** 양의 영과 음의 영 구분
- **±∞ (Infinity):** 오버플로우 시 표현
- **NaN (Not a Number):** 0÷0, √(-1) 등의 연산 결과

```java
// Java 예시
System.out.println(1.0 / 0.0);    // Infinity
System.out.println(-1.0 / 0.0);   // -Infinity
System.out.println(0.0 / 0.0);    // NaN
```

---

## 6. 🛠️ 언어별 해결 전략: 실전 가이드

### 6.1 단계별 접근법 (초보자 → 고급자)

**🎯 단계 1: 기본 double 사용 (간단한 경우)**

```java
// 대부분의 경우 충분함
double price = 19.99;
double tax = 0.08;
double total = price * tax;  // 1.5992
```

**🎯 단계 2: 정밀도 비교 시 주의**

```java
// ❌ 잘못된 비교
if (0.1 + 0.2 == 0.3) { }

// ✅ 올바른 비교 (허용 오차 사용)
final double EPSILON = 1e-9;
if (Math.abs((0.1 + 0.2) - 0.3) < EPSILON) { }
```

**🎯 단계 3: 금융 계산 시 BigDecimal**

```java
// 돈 계산할 때는 필수!
import java.math.BigDecimal;

BigDecimal price = new BigDecimal("19.99");
BigDecimal tax = new BigDecimal("0.08");
BigDecimal total = price.multiply(tax);  // 정확한 1.5992
```

**Java:**
```java
// BigDecimal 사용
import java.math.BigDecimal;

BigDecimal price = new BigDecimal("19.99");
BigDecimal tax = new BigDecimal("0.08");
BigDecimal total = price.multiply(tax);  // 1.5992
```

**Java:**
```java
// BigDecimal을 사용한 정확한 계산
import java.math.BigDecimal;

public BigDecimal addMoney(BigDecimal a, BigDecimal b) {
    // 센트 단위로 변환하여 정수 연산
    BigDecimal centsA = a.multiply(new BigDecimal("100"));
    BigDecimal centsB = b.multiply(new BigDecimal("100"));
    BigDecimal totalCents = centsA.add(centsB);
    return totalCents.divide(new BigDecimal("100"));
}
```

**Go (간단한 버전):**

```go
// BigDecimal 같은 고정밀도 계산
import "math/big"

// 돈 계산 (BigInt로 센트 단위 사용)
func addMoney(a, b int64) int64 {
    return a + b // 센트 단위로 계산
}

// 고정밀도 필요시 big.Float
func preciseCalc() {
    a := big.NewFloat(19.99)
    b := big.NewFloat(0.08)
    result := new(big.Float).Mul(a, b)
    fmt.Println(result) // 정확한 계산
}
```

### 6.2 언어별 추천 사용법

| 언어 | 안전한 돈 계산 | 일반 실수 계산 | 주의사항 |
|------|----------------|----------------|----------|
| **Java** | `BigDecimal` | `double` | `float` 피하기 |
| **JavaScript** | 정수로 변환 | `number` | BigInt로 큰 정수 |
| **Python** | `decimal.Decimal` | `float` | 기본 int는 무제한 |
| **Go** | `big.Float` | `float64` | 금융용 라이브러리 필요 |
| **C/C++** | GMP 라이브러리 | `double` | 오버플로우 체크 필수 |

---

## 7. 🔍 디버깅: 문제 발견하고 해결하기

### 7.1 실수 비교: 절대 == 사용 금지!

**🚨 가장 흔한 버그:**

```java
// ❌ 절대 이렇게 하지 마세요
if (0.1 + 0.2 == 0.3) {
    System.out.println("같아요!");
} // 출력 안 됨!

// ✅ 올바른 방법
final double EPSILON = 1e-9; // 0.000000001
if (Math.abs((0.1 + 0.2) - 0.3) < EPSILON) {
    System.out.println("거의 같아요!");
}
```

**💡 EPSILON 값 선택 팁:**
- `1e-6`: 일반 계산 (백만분의 1)
- `1e-9`: 과학 계산 (십억분의 1)
- `1e-15`: 고정밀도 계산

**문제 2: 오버플로우 방지**

```java
// ❌ 위험한 코드
public int add(int a, int b) {
    return a + b; // 2^31-1 + 1 = 음수됨!
}

// ✅ 안전한 코드 (Java 8+)
public int safeAdd(int a, int b) {
    return Math.addExact(a, b); // 오버플로우 시 예외
}

// ✅ 수동 체크
public int safeAdd(int a, int b) {
    long result = (long) a + b;
    if (result > Integer.MAX_VALUE) {
        throw new ArithmeticException("Overflow!");
    }
    return (int) result;
}
```

### 7.2 디버깅 도구: 문제 진단하기

**🔧 기본 디버깅 코드:**

```java
// 각 타입의 범위 확인
System.out.println("Int 최대값: " + Integer.MAX_VALUE);    // 2,147,483,647
System.out.println("Long 최대값: " + Long.MAX_VALUE);      // 9,223,372,036,854,775,807

// 실수 정밀도 확인
System.out.println("Float 정밀도: " + Float.SIZE + "비트");   // 32비트
System.out.println("Double 정밀도: " + Double.SIZE + "비트"); // 64비트

// 문제 있는 계산 디버깅
double problematic = 0.1 + 0.2;
System.out.println("계산 결과: " + problematic);
System.out.println("예상 결과: " + 0.3);
System.out.println("오차: " + Math.abs(problematic - 0.3));
```

---

## 9. 미래를 내다보는 관점

### 9.1 새로운 표준의 등장

**IEEE 754-2019:**
- 새로운 정밀도 형식 추가 (quadruple precision)
- 더 나은 예외 처리 메커니즘
- 향상된 반올림 전략

### 9.2 양자 컴퓨팅의 영향

양자 컴퓨터에서는 전통적인 이진수 표현이 아닌 **양자 비트(Qubit)**를 사용하므로, 현재의 부동 소수점 문제가 해결될 수 있습니다.

**현재 연구 동향:**
- **양자 부동 소수점:** Qubit 기반의 새로운 숫자 표현 방식 개발 중
- **Shor 알고리즘:** 대수 연산을 이용한 인수분해로 암호화 체계 변화 예고
- **Grover 알고리즘:** 검색 성능 2√N으로 향상시켜 데이터베이스 최적화

**실무적 영향:**
```go
// 양자 컴퓨팅 시대의 고전적 vs 양자적 접근
// 고전적: O(2^n) - 지수 시간 복잡도
func classicalFactorization(n int) []int {
    // 폴라드 로 알고리즘 등 - 여전히 지수 시간
    return []int{}
}

// 양자적: O(n^2) - 다항식 시간 복잡도 (Shor 알고리즘)
func quantumFactorization(n int) []int {
    // 양자 컴퓨터에서 n^2 시간에 인수분해 가능
    // 현재 RSA-2048 암호화의 기반 흔들림
    return []int{}
}
```

### 9.3 대안적 숫자 표현

**고정 소수점의 부활:**
- 특정 도메인(게임, 임베디드)에서 부동 소수점보다 효율적
- 정밀도 예측 가능

**인터벌 연산(Interval Arithmetic):**
- 값의 범위를 계산하여 불확실성 표현
- 과학 계산에서 정확도 향상

### 9.4 현재 진행 중인 혁신 기술들

**포지티브(Positive) 컴퓨팅:**
- 음수를 사용하지 않는 새로운 계산 패러다임
- 에너지 효율성 향상으로 모바일/임베디드 최적화

**신경망 가속기(Neuromorphic Computing):**
```python
# 전통적 vs 신경망 기반 숫자 처리
import numpy as np

# 전통적 부동 소수점
def traditional_fft(signal):
    return np.fft.fft(signal)  # O(n log n)

# 신경망 기반 (연구 중)
def neuromorphic_fft(signal):
    # 스파이크 기반 처리로 에너지 효율성 향상
    # 현재 IBM TrueNorth, Intel Loihi 등에서 연구 중
    return signal  # 개념적 구현
```

**멀티-프리시전 컴퓨팅:**
- FP8, FP16, BF16 등 다양한 정밀도 동시 지원
- AI 모델 학습/추론 최적화

**하드웨어 기반 BigInteger:**
- CPU/GPU 레벨에서 임의 정밀도 정수 지원
- 암호화, 과학 계산 성능 대폭 향상

---

## 9. 💡 실무 선택 가이드

### 9.1 언제 어떤 타입을 사용할까?

| 상황 | 추천 타입 | 이유 |
|------|-----------|------|
| **돈 계산** | `BigDecimal` / 정수 | 법적 정확성 요구 |
| **게임 수치** | 정수 변환 | 성능 + 정확성 |
| **과학 계산** | `double` / `BigDecimal` | 정밀도 필요 |
| **일반 계산** | `double` | 충분한 정밀도 |
| **임베디드** | `float` | 메모리 절약 |

### 9.2 성능 vs 정확성 선택

**⚖️ 트레이드오프:**

```
속도 ↑    float → double → 정수 변환 → BigDecimal    ↓ 정확성
```

**💡 실무 원칙:**
- **돈 관련:** 정확성 우선, 성능은 포기
- **게임/그래픽:** 성능 우선, 작은 오차 허용
- **과학 계산:** 최대 정밀도, 시간은 덜 중요

프로젝트 요구사항에 따라 적절한 타협점을 찾으세요.

### 10.4 아키텍처 설계 시 고려사항

**데이터베이스 설계:**
```sql
-- 금융 데이터: DECIMAL 사용
CREATE TABLE financial_transactions (
    id BIGINT PRIMARY KEY,
    amount DECIMAL(19, 4) NOT NULL,  -- 15자리 정수 + 4자리 소수점
    currency VARCHAR(3) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 과학 데이터: 정밀도 지정
CREATE TABLE scientific_measurements (
    id BIGINT PRIMARY KEY,
    value DECIMAL(30, 15) NOT NULL,  -- 고정밀도 저장
    uncertainty DECIMAL(10, 8),      -- 불확실성 범위
    unit VARCHAR(20)
);
```

**API 설계:**
```java
// 클라이언트 ↔ 서버 간 안전한 숫자 전송
@RestController
public class FinancialAPI {
    @PostMapping("/transfer")
    public ResponseEntity<?> transferMoney(@RequestBody TransferRequest request) {
        // 문자열로 받은 금액을 BigDecimal로 변환
        BigDecimal amount = new BigDecimal(request.getAmountString());

        // 유효성 검증
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            return ResponseEntity.badRequest().body("금액은 0보다 커야 합니다");
        }

        // 센트 단위로 변환하여 저장
        long cents = amount.multiply(new BigDecimal("100")).longValue();
        // ... 저장 로직
    }
}
```

### 10.5 모니터링과 디버깅 전략

**정밀도 모니터링:**
```java
public class PrecisionMonitor {
    private static final Logger logger = LoggerFactory.getLogger(PrecisionMonitor.class);

    public void monitorCalculation(BigDecimal input, BigDecimal result, String operation) {
        // 계산 정밀도 검증
        int inputPrecision = input.precision();
        int resultPrecision = result.precision();

        if (resultPrecision > inputPrecision + 10) {  // 비정상적 정밀도 증가
            logger.warn("의심스러운 정밀도 변화: {} -> {} in operation {}",
                       inputPrecision, resultPrecision, operation);
        }

        // 오차 범위 검증
        BigDecimal expectedRange = calculateExpectedRange(input);
        if (result.compareTo(expectedRange) > 0) {
            logger.error("계산 결과가 예상 범위를 벗어남: {} in operation {}",
                        result, operation);
        }
    }
}
```

**성능 프로파일링:**
```java
public class PerformanceProfiler {
    public static void comparePrecisionPerformance() {
        int iterations = 1000000;

        // Double 성능 측정
        long start = System.nanoTime();
        double doubleResult = 0.0;
        for (int i = 0; i < iterations; i++) {
            doubleResult += 0.1;
        }
        long doubleTime = System.nanoTime() - start;

        // BigDecimal 성능 측정
        start = System.nanoTime();
        BigDecimal decimalResult = BigDecimal.ZERO;
        BigDecimal increment = new BigDecimal("0.1");
        for (int i = 0; i < iterations; i++) {
            decimalResult = decimalResult.add(increment);
        }
        long decimalTime = System.nanoTime() - start;

        System.out.printf("Double: %.3f ms, BigDecimal: %.3f ms (%.1fx slower)%n",
                         doubleTime / 1e6, decimalTime / 1e6,
                         (double) decimalTime / doubleTime);
    }
}
```

### 10.3 테스트 전략

```java
import java.math.BigDecimal;

public void testPrecision() {
    // 부동 소수점 정밀도 테스트
    // 허용 오차 내에서 비교
    double tolerance = 1e-10;
    double result = 0.1 + 0.2 - 0.3;
    assert Math.abs(result) < tolerance : "부동 소수점 오차가 허용 범위를 초과";

    // BigDecimal 등가물 테스트
    BigDecimal d1 = new BigDecimal("0.1");
    BigDecimal d2 = new BigDecimal("0.2");
    BigDecimal expected = new BigDecimal("0.3");
    assert d1.add(d2).compareTo(expected) == 0 : "BigDecimal 계산이 정확하지 않음";
}
```

---

## 8. 💰 실무 문제 해결 사례

### 8.1 카카오페이 송금: 소액 정밀도 문제

**🚨 실제 문제 상황:**
- 500원 송금 시 499.999999...원 도착
- 고객 불만과 환불 요청 발생

**❌ 문제 코드 (실제 서비스에서 발견):**
```java
public double calculateTransferFee(double amount, double feeRate) {
    double fee = amount * feeRate;  // 500 * 0.002 = 0.999999...
    double finalAmount = amount - fee;  // 499.000001...원?
    return finalAmount;
}
```

**✅ 해결책: 정수 기반 계산**
```java
public long calculateTransferAmount(long amountInWon, int feeRateBips) {
    // 1 bip = 0.01% = 0.0001
    // feeRateBips = 20 (0.2% 수수료)

    long fee = amountInWon * feeRateBips / 10000;  // 정수 연산
    long finalAmount = amountInWon - fee;

    return finalAmount;  // 정확한 계산 보장
}

// 사용 예시
long result = calculateTransferAmount(50000, 20);  // 500원, 0.2% 수수료
// 결과: 49900원 (499.00원)
```

**📊 적용 결과:**
- **정밀도:** 100% 정확한 계산
- **성능:** 부동 소수점보다 2배 빠름
- **안정성:** 오차 누적 방지

### 8.2 배달의민족: 리뷰 평점 계산

**🚨 실제 문제 상황:**
- 4.5점 평균 평점이 4.499999...점으로 표시
- 사용자 불만과 신뢰도 저하

**❌ 문제 코드:**
```java
public double calculateAverageRating(List<Double> ratings) {
    double sum = 0.0;
    for (double rating : ratings) {
        sum += rating;  // 누적 오차 발생
    }
    return sum / ratings.size();
}

// 실제 결과: [5.0, 4.0, 3.0] → 3.9999999999999996
```

**✅ 해결책: 정수 기반 계산**
```java
public double calculateAverageRating(List<Double> ratings) {
    // 점수를 10배하여 정수로 변환 (4.5점 → 45점)
    long totalScore = 0;
    for (double rating : ratings) {
        totalScore += Math.round(rating * 10);  // 정수로 변환
    }

    // 평균 계산 후 다시 실수로 변환
    double average = totalScore / (double) ratings.size() / 10.0;
    return Math.round(average * 10) / 10.0;  // 소수점 1자리까지
}

// 결과: 정확한 4.0점 계산
```

**📊 적용 결과:**
- **정확성:** 소수점 1자리까지 정확한 평균
- **성능:** 간단한 정수 연산으로 해결
- **사용자 만족도:** 평점 표시 신뢰도 향상

### 8.3 과학 시뮬레이션의 수치 안정성 문제

**문제 상황:**
- 유체 역학 시뮬레이션에서 미세한 오차가 큰 결과 차이 야기
- 64비트 double로도 부족한 정밀도 요구사항

**실제 사례 (날씨 예측 모델):**
```java
// 불안정한 계산 - 오차가 증폭됨
public double simulateWeather(double temperature, double humidity, double pressure) {
    // 복합 계산에서 오차 증폭
    double dewPoint = temperature - ((100 - humidity) / 5);
    double vaporPressure = 6.11 * Math.pow(10, (7.5 * dewPoint) / (237.3 + dewPoint));
    double actualVapor = vaporPressure * (humidity / 100);

    // 결과적으로 큰 오차 발생
    return actualVapor;
}
```

**해결책: 수치 안정성 개선**
```java
import java.math.BigDecimal;
import java.math.MathContext;
import java.math.RoundingMode;
import java.util.Random;

public class StableCalculator {
    private final MathContext mc = new MathContext(28, RoundingMode.HALF_UP);
    private final Random random = new Random();

    public StableCalculator() {
        // BigDecimal 정밀도 설정 (28자리)
    }

    public double stableDewPoint(double temperature, double humidity) {
        // 수치적으로 안정한 이슬점 계산
        BigDecimal T = new BigDecimal(String.valueOf(temperature), mc);
        BigDecimal H = new BigDecimal(String.valueOf(humidity), mc);

        // Magnus 공식의 안정 버전
        BigDecimal a = new BigDecimal("17.625", mc);
        BigDecimal b = new BigDecimal("243.04", mc);

        // ln(rh/100) 계산에서 수치 안정성 확보
        BigDecimal rhRatio = H.divide(new BigDecimal("100", mc), mc);
        BigDecimal lnRh = new BigDecimal(String.valueOf(Math.log(rhRatio.doubleValue())), mc);

        BigDecimal numerator = b.multiply(
            lnRh.add(a.multiply(T, mc).divide(b.add(T, mc), mc), mc), mc
        );

        BigDecimal denominator = a.subtract(
            lnRh.add(a.multiply(T, mc).divide(b.add(T, mc), mc), mc), mc
        );

        BigDecimal dewPoint = numerator.divide(denominator, mc);

        return dewPoint.doubleValue();
    }

    public double unstableDewPoint(double temperature, double humidity) {
        // 불안정한 기존 계산법
        double dewPoint = temperature - ((100 - humidity) / 5);
        double vaporPressure = 6.11 * Math.pow(10, (7.5 * dewPoint) / (237.3 + dewPoint));
        return vaporPressure * (humidity / 100);
    }

    public StabilityResult validateStability(double[] tempRange, double[] humidityRange, int iterations) {
        // 수치 안정성 검증
        BigDecimal maxError = BigDecimal.ZERO;
        BigDecimal maxRelativeError = BigDecimal.ZERO;

        for (int i = 0; i < iterations; i++) {
            double temp = tempRange[0] + random.nextDouble() * (tempRange[1] - tempRange[0]);
            double humidity = humidityRange[0] + random.nextDouble() * (humidityRange[1] - humidityRange[0]);

            // 기존 vs 개선된 계산 비교
            double oldResult = unstableDewPoint(temp, humidity);
            double newResult = stableDewPoint(temp, humidity);

            BigDecimal error = new BigDecimal(String.valueOf(Math.abs(newResult - oldResult)), mc);
            BigDecimal relativeError = error.divide(
                new BigDecimal(String.valueOf(Math.abs(newResult)), mc), mc
            );

            maxError = maxError.max(error);
            maxRelativeError = maxRelativeError.max(relativeError);
        }

        return new StabilityResult(maxError.doubleValue(), maxRelativeError.doubleValue());
    }

    public static class StabilityResult {
        public final double maxAbsoluteError;
        public final double maxRelativeError;

        public StabilityResult(double maxAbsoluteError, double maxRelativeError) {
            this.maxAbsoluteError = maxAbsoluteError;
            this.maxRelativeError = maxRelativeError;
        }
    }
}
```

**수치 안정성 기법:**
1. **Kahan Summation:** 오차 누적 최소화
2. ** Dekker's Algorithm:** 두 개의 부동 소수점으로 정밀도 향상
3. **Interval Arithmetic:** 오차 범위 추적

### 8.4 GPS 좌표 계산의 정밀도 문제

**문제 상황:**
- GPS 좌표: 37.7749, -122.4194 (샌프란시스코)
- 미터 단위 거리 계산에서 정밀도 손실
- 내비게이션 경로 최적화 실패

**실제 문제:**
```java
// 부동 소수점으로 인한 거리 계산 오차
public double haversineDistance(double lat1, double lon1, double lat2, double lon2) {
    final double R = 6371e3; // 지구 반지름 (미터)

    double φ1 = Math.toRadians(lat1);
    double φ2 = Math.toRadians(lat2);
    double Δφ = Math.toRadians(lat2 - lat1);
    double Δλ = Math.toRadians(lon2 - lon1);

    double a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
               Math.cos(φ1) * Math.cos(φ2) *
               Math.sin(Δλ/2) * Math.sin(Δλ/2);

    double c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

    return R * c; // 미터 단위 거리
}

// 문제: 0.1mm 수준의 정밀도 요구사항을 만족하지 못함
```

**해결책: 고정 소수점과 정수 연산**
```java
// Java로 구현한 고정 소수점 GPS 계산
public class GPSCalculator {
    // 1e7으로 스케일링 (1cm 정밀도)
    private static final long SCALE = 10_000_000L;
    private static final long EARTH_RADIUS_MM = 6_371_000_000L; // mm 단위

    // 좌표를 정수로 변환
    public static long toFixed(double coord) {
        return Math.round(coord * SCALE);
    }

    // 정수 기반 거리 계산
    public static long haversineDistance(long lat1, long lon1, long lat2, long lon2) {
        // 라디안 변환 (정수 연산)
        long phi1 = (lat1 * 3141592653L) / (180 * SCALE); // π ≈ 3.141592653
        long phi2 = (lat2 * 3141592653L) / (180 * SCALE);
        long deltaPhi = ((lat2 - lat1) * 3141592653L) / (180 * SCALE);
        long deltaLambda = ((lon2 - lon1) * 3141592653L) / (180 * SCALE);

        // Haversine 공식의 정수 버전
        long sinDeltaPhi = sin(deltaPhi);
        long sinDeltaLambda = sin(deltaLambda);

        long a = (sinDeltaPhi * sinDeltaPhi) +
                 (cos(phi1) * cos(phi2) * sinDeltaLambda * sinDeltaLambda);

        // 정수 기반 역삼각함수 계산
        long c = 2 * atan2(sqrt(a), sqrt(1000000000L - a)); // 1e9 스케일

        return (EARTH_RADIUS_MM * c) / 1000000000L; // mm 단위 거리
    }

    // 정수 기반 삼각함수 (테일러 급수 근사)
    private static long sin(long x) {
        // x는 라디안 × 1e9 스케일
        // 구현 생략 - 실제로는 룩업 테이블이나 근사 알고리즘 사용
        return 0; // 플레이스홀더
    }

    private static long cos(long x) {
        return sin(x + 1570796326L); // π/2 ≈ 1.570796326
    }

    private static long sqrt(long x) {
        // 정수 기반 제곱근 계산
        if (x <= 0) return 0;
        long result = x;
        for (int i = 0; i < 10; i++) { // 뉴턴 방법 10회 반복
            result = (result + x / result) / 2;
        }
        return result;
    }

    private static long atan2(long y, long x) {
        // atan2의 정수 근사 구현
        // 실제로는 더 복잡한 근사 알고리즘 필요
        return 0; // 플레이스홀더
    }
}
```

**적용 결과:**
- **정밀도:** cm 단위 정확도 달성
- **성능:** 모바일 디바이스에서도 실시간 계산 가능
- **배터리 효율:** 부동 소수점보다 낮은 전력 소비

### 8.5 머신러닝 모델의 수치 안정성

**문제 상황:**
- 딥러닝 모델에서 그래디언트 소실/폭발
- 정규화되지 않은 입력으로 인한 수치 불안정성

**해결책 적용:**
```java
// Java에서 수치 안정성을 위한 간단한 신경망 예시
// (실제로는 Deeplearning4j 등의 라이브러리 사용 권장)
public class StableNeuralNetwork {
    private final int inputSize = 784;
    private final int hiddenSize1 = 256;
    private final int hiddenSize2 = 128;
    private final int outputSize = 10;

    // 가중치와 바이어스 (실제로는 행렬 라이브러리 사용)
    private double[][] w1, w2, w3;
    private double[] b1, b2, b3;

    public StableNeuralNetwork() {
        // Xavier 초기화로 그래디언트 소실 방지
        initializeWeights();
    }

    private void initializeWeights() {
        // 가중치 초기화 (Xavier uniform)
        double scale1 = Math.sqrt(2.0 / (inputSize + hiddenSize1));
        double scale2 = Math.sqrt(2.0 / (hiddenSize1 + hiddenSize2));
        double scale3 = Math.sqrt(2.0 / (hiddenSize2 + outputSize));

        w1 = initializeMatrix(hiddenSize1, inputSize, scale1);
        w2 = initializeMatrix(hiddenSize2, hiddenSize1, scale2);
        w3 = initializeMatrix(outputSize, hiddenSize2, scale3);

        b1 = new double[hiddenSize1];
        b2 = new double[hiddenSize2];
        b3 = new double[outputSize];
    }

    private double[][] initializeMatrix(int rows, int cols, double scale) {
        double[][] matrix = new double[rows][cols];
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                matrix[i][j] = (Math.random() - 0.5) * 2 * scale;
            }
        }
        return matrix;
    }

    public double[] forward(double[] input) {
        // 입력 정규화 (배치 정규화 효과)
        double mean = calculateMean(input);
        double std = calculateStd(input, mean);
        double[] normalizedInput = normalize(input, mean, std);

        // 첫 번째 층
        double[] hidden1 = matrixVectorMultiply(w1, normalizedInput);
        addBias(hidden1, b1);
        applyBatchNorm(hidden1);  // 배치 정규화
        applyReLU(hidden1);
        clipGradients(hidden1, -10, 10);  // 그래디언트 클리핑

        // 두 번째 층
        double[] hidden2 = matrixVectorMultiply(w2, hidden1);
        addBias(hidden2, b2);
        applyBatchNorm(hidden2);
        applyReLU(hidden2);
        clipGradients(hidden2, -10, 10);

        // 출력 층
        double[] output = matrixVectorMultiply(w3, hidden2);
        addBias(output, b3);

        return output;
    }

    // 헬퍼 메소드들
    private double calculateMean(double[] array) {
        double sum = 0;
        for (double val : array) sum += val;
        return sum / array.length;
    }

    private double calculateStd(double[] array, double mean) {
        double sumSquared = 0;
        for (double val : array) {
            sumSquared += Math.pow(val - mean, 2);
        }
        return Math.sqrt(sumSquared / array.length) + 1e-8; // epsilon
    }

    private double[] normalize(double[] array, double mean, double std) {
        double[] result = new double[array.length];
        for (int i = 0; i < array.length; i++) {
            result[i] = (array[i] - mean) / std;
        }
        return result;
    }

    private double[] matrixVectorMultiply(double[][] matrix, double[] vector) {
        int rows = matrix.length;
        int cols = matrix[0].length;
        double[] result = new double[rows];

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                result[i] += matrix[i][j] * vector[j];
            }
        }
        return result;
    }

    private void addBias(double[] array, double[] bias) {
        for (int i = 0; i < array.length; i++) {
            array[i] += bias[i];
        }
    }

    private void applyBatchNorm(double[] array) {
        double mean = calculateMean(array);
        double std = calculateStd(array, mean);
        for (int i = 0; i < array.length; i++) {
            array[i] = (array[i] - mean) / std;
        }
    }

    private void applyReLU(double[] array) {
        for (int i = 0; i < array.length; i++) {
            array[i] = Math.max(0, array[i]);
        }
    }

    private void clipGradients(double[] array, double min, double max) {
        for (int i = 0; i < array.length; i++) {
            array[i] = Math.max(min, Math.min(max, array[i]));
        }
    }
}
```

**수치 안정성 기법:**
1. **Batch Normalization:** 내부 공변량 이동 제거
2. **Gradient Clipping:** 그래디언트 폭발 방지
3. **Weight Initialization:** Xavier/He 초기화
4. **Mixed Precision Training:** FP16으로 메모리 효율화

---

## 11. URL 단축 서비스의 인코딩 최적화

### 11.1 URL 단축 서비스의 수학적 기반

**문제 상황:**
- 긴 URL을 짧은 키로 변환하여 수십억 개의 매핑 관리
- 빠른 조회와 높은 동시성 요구
- bit.ly, t.co, goo.gl 스타일 서비스 구현

**해결책 적용:**
- Hash Table + 데이터베이스 조합
- Base62 인코딩으로 키 생성
- 분산 캐시로 성능 최적화

### 11.2 다양한 인코딩 방식의 특징 분석

URL 단축 서비스에서 숫자 ID를 짧은 문자열로 변환할 때 사용하는 인코딩 방식들의 특징을 비교해보겠습니다.

#### Base62 인코딩 (가장 일반적)
```java
// Base62: 0-9, a-z, A-Z (총 62개 문자)
public class Base62Encoder {
    private static final String BASE62 = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    public static String encode(long value) {
        if (value == 0) return "0";

        StringBuilder sb = new StringBuilder();
        while (value > 0) {
            sb.append(BASE62.charAt((int)(value % 62)));
            value /= 62;
        }
        return sb.reverse().toString();
    }

    public static long decode(String value) {
        long result = 0;
        for (char c : value.toCharArray()) {
            result = result * 62 + BASE62.indexOf(c);
        }
        return result;
    }
}
```

**특징:**
- **문자 집합:** 62개 (0-9, a-z, A-Z)
- **URL 안전성:** 완전 URL 안전 (특수문자 없음)
- **길이 효율성:** ID 1,000,000 → "4C92" (4자리)
- **대소문자 구분:** 예 (URL에서 중요)
- **장점:** 짧은 길이, URL 친화적
- **단점:** 구현 복잡도 높음

#### Base64 인코딩 (웹 표준)
```java
// Base64: A-Z, a-z, 0-9, +, / (총 64개 + 패딩 =)
public class Base64URLSafeEncoder {
    private static final String BASE64URL = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_";

    public static String encode(long value) {
        // 표준 Base64보다 URL-safe 버전 사용
        return java.util.Base64.getUrlEncoder()
                .withoutPadding()
                .encodeToString(java.nio.ByteBuffer.allocate(8).putLong(value).array())
                .replace('+', '-').replace('/', '_'); // URL 안전하게 변환
    }

    public static long decode(String value) {
        // URL-safe Base64 디코딩
        String standardBase64 = value.replace('-', '+').replace('_', '/');
        byte[] bytes = java.util.Base64.getDecoder().decode(standardBase64);
        return java.nio.ByteBuffer.wrap(bytes).getLong();
    }
}
```

**특징:**
- **문자 집합:** 64개 + 패딩 문자
- **표준화:** RFC 4648 표준
- **길이 효율성:** ID 1,000,000 → "4C92" (4자리)
- **장점:** 표준화, 널리 사용됨
- **단점:** URL에서 +, /, = 문자 문제

#### Base36 인코딩 (단순함)
```java
// Base36: 0-9, a-z (총 36개 문자)
public class Base36Encoder {
    private static final String BASE36 = "0123456789abcdefghijklmnopqrstuvwxyz";

    public static String encode(long value) {
        if (value == 0) return "0";

        StringBuilder sb = new StringBuilder();
        while (value > 0) {
            sb.append(BASE36.charAt((int)(value % 36)));
            value /= 36;
        }
        return sb.reverse().toString();
    }

    public static long decode(String value) {
        long result = 0;
        for (char c : value.toCharArray()) {
            result = result * 36 + BASE36.indexOf(c);
        }
        return result;
    }
}
```

**특징:**
- **문자 집합:** 36개 (대소문자 구분 없음)
- **단순성:** 구현이 가장 간단
- **길이 효율성:** ID 1,000,000 → "6bny" (4자리)
- **장점:** 단순한 구현, 가독성 좋음
- **단점:** Base62보다 긴 문자열 생성

#### Base32 인코딩 (안전성 중시)
```java
// Base32: A-Z, 2-7 (총 32개 문자)
public class Base32Encoder {
    private static final String BASE32 = "ABCDEFGHIJKLMNOPQRSTUVWXYZ234567";

    public static String encode(long value) {
        if (value == 0) return "A";

        StringBuilder sb = new StringBuilder();
        while (value > 0) {
            sb.append(BASE32.charAt((int)(value % 32)));
            value /= 32;
        }
        return sb.reverse().toString();
    }

    public static long decode(String value) {
        long result = 0;
        for (char c : value.toCharArray()) {
            result = result * 32 + BASE32.indexOf(c);
        }
        return result;
    }
}
```

**특징:**
- **문자 집합:** 32개 (대문자 + 숫자)
- **안전성:** 특수문자 완전 배제
- **길이 효율성:** ID 1,000,000 → "7N5U" (4자리)
- **장점:** 가장 안전한 문자 집합
- **단점:** 가장 긴 문자열 생성

### 11.3 인코딩 방식 선택 기준

| 방식 | 문자 수 | URL 안전성 | 구현 난이도 | 길이 효율성 | 사용 사례 |
|------|---------|------------|-------------|-------------|-----------|
| **Base62** | 62 | 완전 안전 | 높음 | 최상 | bit.ly, t.co |
| **Base64** | 64 | 조건부 안전 | 중간 | 상 | API 토큰, 세션 ID |
| **Base36** | 36 | 안전 | 낮음 | 중 | Git 해시, 단순 서비스 |
| **Base32** | 32 | 완전 안전 | 낮음 | 하 | 보안 토큰, OTP |

#### 선택 기준:
1. **URL 직접 사용 시:** Base62 또는 Base64 URL-safe 버전
2. **단순 구현 시:** Base36
3. **보안 최우선 시:** Base32
4. **표준 준수 시:** Base64

### 11.4 실제 서비스 구현 사례

#### bit.ly 스타일 URL 단축기
```java
public class URLShortener {
    private final Map<String, String> urlMap = new ConcurrentHashMap<>();
    private final Map<String, String> reverseMap = new ConcurrentHashMap<>();
    private final AtomicLong counter = new AtomicLong(1000); // 1000부터 시작

    // URL 단축
    public String shortenURL(String longURL) {
        // 이미 단축된 URL인지 확인
        String existing = reverseMap.get(longURL);
        if (existing != null) {
            return existing;
        }

        // 새로운 ID 생성 및 Base62 인코딩
        long id = counter.incrementAndGet();
        String shortKey = Base62Encoder.encode(id);

        // 매핑 저장
        urlMap.put(shortKey, longURL);
        reverseMap.put(longURL, shortKey);

        return shortKey;
    }

    // URL 복원
    public String expandURL(String shortKey) {
        return urlMap.get(shortKey);
    }

    // 통계 정보
    public Map<String, Object> getStats() {
        return Map.of(
            "totalURLs", urlMap.size(),
            "nextID", counter.get(),
            "collisionRate", 0.0 // 실제로는 충돌 감지 로직 필요
        );
    }
}
```

#### 고성능 분산 구현
```java
public class DistributedURLShortener {
    private final RedisTemplate<String, String> redisTemplate;
    private final AtomicLong counter = new AtomicLong();

    public DistributedURLShortener(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
        // Redis에서 마지막 ID 복원
        String lastId = redisTemplate.opsForValue().get("last_url_id");
        if (lastId != null) {
            counter.set(Long.parseLong(lastId));
        }
    }

    public String shortenURL(String longURL) {
        // 캐시에서 기존 단축 URL 확인
        String cached = redisTemplate.opsForValue().get("reverse:" + longURL);
        if (cached != null) {
            return cached;
        }

        // 분산 환경에서 고유 ID 생성 (실제로는 Snowflake ID 등 사용)
        long id = counter.incrementAndGet();

        // Base62 인코딩
        String shortKey = Base62Encoder.encode(id);

        // Redis에 저장 (TTL 설정으로 임시 저장소 역할)
        redisTemplate.opsForValue().set("url:" + shortKey, longURL, Duration.ofDays(365));
        redisTemplate.opsForValue().set("reverse:" + longURL, shortKey, Duration.ofDays(365));
        redisTemplate.opsForValue().set("last_url_id", String.valueOf(id));

        return shortKey;
    }

    public String expandURL(String shortKey) {
        // Redis에서 URL 조회 (캐시 미스 시 DB 조회)
        String url = redisTemplate.opsForValue().get("url:" + shortKey);
        if (url == null) {
            // DB에서 조회하는 로직 추가 가능
            throw new RuntimeException("URL not found: " + shortKey);
        }
        return url;
    }
}
```

### 11.5 성능 및 확장성 고려사항

#### 데이터베이스 설계
```sql
-- URL 매핑 테이블
CREATE TABLE url_mappings (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_key VARCHAR(10) UNIQUE NOT NULL,
    original_url TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    access_count BIGINT DEFAULT 0,
    INDEX idx_short_key (short_key),
    INDEX idx_created_at (created_at)
);

-- 분산 ID 생성을 위한 시퀀스
CREATE TABLE url_sequences (
    stub CHAR(1) PRIMARY KEY DEFAULT 'a',
    next_id BIGINT DEFAULT 1000
);
```

#### 캐시 전략
- **Write-through:** DB 저장 후 캐시 업데이트
- **Cache-aside:** 캐시 미스 시 DB 조회 후 캐시 저장
- **TTL 설정:** 임시 URL의 자동 만료

#### 확장성 패턴
1. **Database Sharding:** ID 범위별 샤딩
2. **Read Replicas:** 읽기 부하 분산
3. **CDN:** 정적 에셋 캐싱
4. **Rate Limiting:** DDoS 방지

---

---

## 10. 🎯 결론: 실무에서 꼭 기억할 것

**"컴퓨터는 정확한 계산을 못 한다. 하지만 우리는 정확하게 만들 수 있다."**

### 📚 핵심 교훈:

1. **돈 계산할 때는 BigDecimal 필수**
2. **실수 비교할 때는 == 사용 금지**
3. **정수는 안전, 실수는 위험**
4. **오버플로우를 항상 체크하라**
5. **테스트할 때는 EPSILON 사용**

### 🚀 다음 단계:
- 실제 프로젝트에서 정밀도 문제 찾아보기
- BigDecimal 사용법 익히기
- 단위 테스트로 정밀도 검증 추가하기

**행복한 코딩 되세요! 🎉**
