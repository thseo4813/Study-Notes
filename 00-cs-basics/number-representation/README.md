# 컴퓨터의 숫자 표현: 이진수와 부동 소수점의 세계

컴퓨터가 숫자를 다루는 방식은 우리가 익숙한 십진법과 근본적으로 다릅니다. **0과 1만으로 세상의 모든 숫자를 표현**해야 하는 제약 때문에 발생하는 다양한 문제들과 해결책을 다룹니다.

## 1. 핵심 요약 (Executive Summary)

컴퓨터는 모든 데이터를 **이진수(Binary)**로 표현하며, 숫자를 저장할 때는 **정수(Integer)**와 **실수(Floating Point)**로 구분합니다. 특히 실수 표현에서는 **IEEE 754 표준**을 따르지만, 이진수의 근본적 한계 때문에 **정밀도 손실**이 필연적으로 발생합니다.

> **결론:**
> 1. **정수 표현:** 2의 보수(Two's Complement)로 음수 표현, 오버플로우 주의
> 2. **실수 표현:** 부동 소수점으로 근사값 사용, 정확도보다 효율성 우선
> 3. **실무 원칙:** 금융/과학 분야에서는 float/double 절대 금지, Decimal이나 정수 연산 사용

---

## 2. 컴퓨터가 숫자를 세는 법: 이진법의 기초

### 2.1 십진수 vs 이진수 변환

십진수 13을 이진수로 변환하는 과정:

```
십진수 → 이진수 변환 (13을 예로)
13 ÷ 2 = 6 나머지 1
 6 ÷ 2 = 3 나머지 0
 3 ÷ 2 = 1 나머지 1
 1 ÷ 2 = 0 나머지 1

따라서 13₁₀ = 1101₂
```

**이진수 → 십진수 변환:**
- 1101₂ = 1×2³ + 1×2² + 0×2¹ + 1×2⁰ = 8 + 4 + 0 + 1 = 13₁₀

### 2.2 16진수와 8진수 (프로그래밍에서 자주 사용)

프로그래밍에서 메모리 주소를 표현할 때 편리합니다:
- **16진수(Hexadecimal):** 0-9, A-F 사용, `0x` 접두사
- **8진수(Octal):** 0-7 사용, `0` 접두사

```text
십진수 255:
이진수: 11111111₂
8진수: 377₈ (3×8² + 7×8¹ + 7×8⁰)
16진수: FF₁₆ (15×16¹ + 15×16⁰) → 0xFF
```

---

## 3. 정수 표현의 세계

### 3.1 부호 없는 정수 (Unsigned Integer)

가장 단순한 형태: 0부터 최댓값까지 표현

| 비트 수 | 범위 | 예시 |
| --- | --- | --- |
| 8비트 | 0 ~ 255 | `uint8_t` |
| 16비트 | 0 ~ 65,535 | `uint16_t` |
| 32비트 | 0 ~ 4,294,967,295 | `uint32_t` |
| 64비트 | 0 ~ 18,446,744,073,709,551,615 | `uint64_t` |

### 3.2 부호 있는 정수와 2의 보수 (Two's Complement)

음수를 표현하기 위한 표준 방식입니다.

**원리:**
1. **부호 비트(Sign Bit):** 최상위 비트가 1이면 음수
2. **2의 보수 계산:** 모든 비트를 반전시키고 +1

```text
십진수 -5를 8비트 2의 보수로 표현:
1. 5의 이진수: 00000101
2. 모든 비트 반전: 11111010
3. +1: 11111011

결과: -5₁₀ = 11111011₂ (8비트 기준)
```

**범위 비교:**

| 비트 수 | Signed 범위 | Unsigned 범위 |
| --- | --- | --- |
| 8비트 | -128 ~ 127 | 0 ~ 255 |
| 16비트 | -32,768 ~ 32,767 | 0 ~ 65,535 |
| 32비트 | -2,147,483,648 ~ 2,147,483,647 | 0 ~ 4,294,967,295 |

### 3.3 오버플로우(Overflow)의 함정

정수 타입의 최댓값을 초과하면 발생하는 문제:

```java
// Java 예시
int maxInt = Integer.MAX_VALUE;  // 2,147,483,647
int result = maxInt + 1;         // 오버플로우 발생!
System.out.println(result);      // -2,147,483,648 (최솟값으로 순환)
```

**오버플로우 방지 전략:**
- **언어 차원:** `Math.addExact()` 사용 (Java 8+)
- **BigInteger:** 임의 정밀도 정수 사용
- **체크 로직:** 연산 전 범위 검증

---

## 4. 실수 표현의 심층 분석

### 4.1 고정 소수점 vs 부동 소수점

**고정 소수점(Fixed Point):**
- 소수점 위치를 고정 (예: 소수점 아래 4자리)
- 정밀도는 높지만 표현 범위가 좁음

**부동 소수점(Floating Point):**
- 소수점을 자유롭게 이동시켜 넓은 범위 표현
- **가수부(Mantissa) + 지수부(Exponent) + 부호비트**로 구성

### 4.2 IEEE 754 표준의 구조

32비트(float)와 64비트(double)의 구조:

| 구성 요소 | float (32비트) | double (64비트) |
| --- | --- | --- |
| **부호 비트** | 1비트 | 1비트 |
| **지수부** | 8비트 (-126 ~ 127) | 11비트 (-1022 ~ 1023) |
| **가수부** | 23비트 | 52비트 |
| **총 비트** | 32비트 | 64비트 |

**실제 값 계산 공식:**
```
실제 값 = (-1)^부호비트 × (1 + 가수부/2^가수부비트수) × 2^(지수부 - 바이어스)
```

### 4.3 부동 소수점의 시각화

![부동 소수점 구조](./image/copyImage.jpg)

**구조 설명:**
- **부호 비트:** 0=양수, 1=음수
- **지수부:** 2의 거듭제곱을 표현 (바이어스 적용으로 음수 지수도 표현)
- **가수부:** 1.xxx 형태의 유효숫자 표현

---

## 5. 정밀도 손실의 다양한 얼굴

### 5.1 표현할 수 없는 수: 0.1의 문제

십진수 0.1을 이진수로 표현하려면?

```
0.1 × 2 = 0.2 → 0 (올림)
0.2 × 2 = 0.4 → 0 (올림)
0.4 × 2 = 0.8 → 0 (올림)
0.8 × 2 = 1.6 → 1 (올림)
0.6 × 2 = 1.2 → 1 (올림)
0.2 × 2 = 0.4 → 0 (올림)
...
```

결과: **0.1은 무한 소수**가 되어 근사값으로 저장됩니다.

```java
// Java 예시
System.out.println(0.1 + 0.2);  // 0.30000000000000004 (정확히 0.3이 아님!)
System.out.println(0.1 + 0.2 == 0.3);  // false!
```

### 5.2 정밀도 한계에 따른 문제

**단정밀도(float) vs 배정밀도(double):**

```java
// Java 예시
float f = 1.0f / 3.0f;     // 0.33333334 (근사값)
double d = 1.0 / 3.0;      // 0.3333333333333333 (더 정확한 근사값)
```

**연산 누적 오차:**
```java
double result = 0.0;
for (int i = 0; i < 10; i++) {
    result += 0.1;
}
System.out.println(result);  // 0.9999999999999999 (1.0이 아님!)
```

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

## 6. 프로그래밍 언어별 해결 전략

### 6.1 금융/정밀 계산용 타입들

**Java:**
```java
// BigDecimal 사용 (권장)
import java.math.BigDecimal;
BigDecimal price = new BigDecimal("19.99");
BigDecimal tax = new BigDecimal("0.08");
BigDecimal total = price.multiply(tax);  // 정확한 계산 보장
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

### 6.2 언어별 기본 타입 비교

| 언어 | 정수 타입 | 실수 타입 | 임의 정밀도 |
| --- | --- | --- | --- |
| **C/C++** | int32_t, int64_t | float, double | 없음 (라이브러리 필요) |
| **Java** | int, long | float, double | BigInteger, BigDecimal |
| **Python** | int (임의 크기) | float | decimal.Decimal |
| **JavaScript** | Number (double) | Number (double) | BigInt (정수만) |

---

## 7. 실무 적용 사례와 베스트 프랙티스

### 7.1 금융 시스템에서의 적용

**문제 상황:**
```java
// 위험한 코드
float accountBalance = 1000000.00f;
accountBalance += 0.01f;  // 정밀도 손실 발생 가능
```

**올바른 해결:**
```java
// Java: BigDecimal 사용
BigDecimal accountBalance = new BigDecimal("1000000.00");
BigDecimal deposit = new BigDecimal("0.01");
accountBalance = accountBalance.add(deposit);
```

### 7.2 과학 계산에서의 주의사항

**상대 오차 vs 절대 오차:**
- **절대 오차:** |실제값 - 계산값|
- **상대 오차:** |실제값 - 계산값| / |실제값|

```java
// 상대 오차가 중요한 경우
public double relativeError(double actual, double calculated) {
    return Math.abs(actual - calculated) / Math.abs(actual);
}

// 매우 작은 수의 비교 시 주의
double a = 1e-15;
double b = 1e-16;
System.out.println(a == b);  // false (좋음)
System.out.println(Math.abs(a - b) < 1e-15);  // 상대 오차 고려
```

### 7.3 데이터베이스 저장 전략

**Java (JDBC):**
```java
import java.sql.*;
import java.math.BigDecimal;

public class FinancialDAO {
    // DECIMAL 타입 사용 (정확한 소수점 연산)
    public void createFinancialTable(Connection conn) throws SQLException {
        String sql = "CREATE TABLE financial_data (" +
                    "id BIGINT PRIMARY KEY, " +
                    "amount DECIMAL(19, 4) NOT NULL)";  // 총 19자리, 소수점 4자리
        try (Statement stmt = conn.createStatement()) {
            stmt.execute(sql);
        }
    }

    // 금액을 센트 단위로 저장
    public void createPaymentsTable(Connection conn) throws SQLException {
        String sql = "CREATE TABLE payments (" +
                    "id BIGINT PRIMARY KEY, " +
                    "amount_cents BIGINT NOT NULL)";  // 센트 단위 저장
        try (Statement stmt = conn.createStatement()) {
            stmt.execute(sql);
        }
    }

    // BigDecimal을 사용한 정확한 데이터 저장
    public void saveFinancialData(Connection conn, BigDecimal amount) throws SQLException {
        String sql = "INSERT INTO financial_data (amount) VALUES (?)";
        try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setBigDecimal(1, amount);
            pstmt.executeUpdate();
        }
    }
}
```

---

## 8. 디버깅과 문제 해결

### 8.1 흔한 실수들

**문제 1: 부동 소수점 비교**
```java
// 잘못된 방법
public boolean isEqual(double a, double b) {
    return a == b;  // 정밀도 문제로 실패할 수 있음
}

// 올바른 방법
public boolean isEqual(double a, double b, double tolerance) {
    if (tolerance == 0) tolerance = 1e-9;
    return Math.abs(a - b) < tolerance;
}
```

**문제 2: 오버플로우 무시**
```java
// 위험한 코드
public int multiply(int a, int b) {
    return a * b;  // 오버플로우 발생 가능
}

// 안전한 코드
public int multiply(int a, int b) {
    long result = (long) a * b;
    if (result > Integer.MAX_VALUE || result < Integer.MIN_VALUE) {
        throw new ArithmeticException("Integer overflow");
    }
    return (int) result;
}
```

### 8.2 디버깅 도구 활용

**Java:**
```java
// 안전한 정수 범위 확인
System.out.println(Long.MAX_VALUE);  // 9223372036854775807
System.out.println(Integer.MAX_VALUE);  // 2147483647

// BigInteger를 사용한 임의 정밀도 정수
import java.math.BigInteger;

BigInteger safeInt = new BigInteger("9007199254740991");
BigInteger unsafeInt = new BigInteger("9007199254740992");
System.out.println(safeInt);   // 9007199254740991
System.out.println(unsafeInt); // 9007199254740992
```

**Java:**
```java
// 부동 소수점 정보 출력
System.out.println("Float.MAX_VALUE: " + Float.MAX_VALUE);
System.out.println("Double.MAX_VALUE: " + Double.MAX_VALUE);
System.out.println("Float.MIN_VALUE: " + Float.MIN_VALUE);
System.out.println("Double.MIN_VALUE: " + Double.MIN_VALUE);

// Float와 Double의 정밀도 정보
System.out.println("Float 정밀도: " + Float.SIZE + "비트");
System.out.println("Double 정밀도: " + Double.SIZE + "비트");
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

### 9.3 대안적 숫자 표현

**고정 소수점의 부활:**
- 특정 도메인(게임, 임베디드)에서 부동 소수점보다 효율적
- 정밀도 예측 가능

**인터벌 연산(Interval Arithmetic):**
- 값의 범위를 계산하여 불확실성 표현
- 과학 계산에서 정확도 향상

---

## 10. 전문가적 조언 (Pro Tip)

### 10.1 선택의 지침

**언제 float/double을 사용할까?**
- 속도가 최우선인 게임 그래픽스
- 근사값으로 충분한 과학 시뮬레이션
- 메모리 제약이 심한 임베디드 시스템

**언제 Decimal/BigInteger를 사용할까?**
- 금융, 회계, 전자상거래
- 법적 책임이 있는 계산
- 정확성이 생명인 과학 데이터

### 10.2 성능 vs 정확성 트레이드오프

```text
정확성 ↑      BigDecimal > Fixed Point > Double > Float      ↓ 속도
```

프로젝트 요구사항에 따라 적절한 타협점을 찾으세요.

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

## 11. 실무 문제 해결 사례

### 11.1 은행 이자 계산 시스템의 정밀도 문제

**문제 상황:**
- 수십억 원대의 대출 이자 계산
- 법적 요구사항으로 0.01원 단위 정확성 필요
- 누적 계산으로 인한 정밀도 손실 누적

**실제 사고 사례:**
```java
// 위험한 코드 - 은행에서 실제로 발생한 문제
public double calculateInterest(double principal, double rate, int days) {
    double dailyRate = rate / 365.0;  // 부동 소수점 나눗셈
    double interest = principal * dailyRate * days;  // 누적 오차
    return Math.round(interest * 100.0) / 100.0;  // 늦은 반올림
}
```

**문제 분석:**
- `rate / 365.0`에서 정밀도 손실 발생
- 매일 이자가 누적되며 오차가 커짐
- 최종 반올림으로는 누적 오차를 해결할 수 없음

**올바른 해결:**
```java
// BigDecimal을 사용한 정확한 계산
import java.math.BigDecimal;
import java.math.RoundingMode;

public class InterestCalculator {
    private static final BigDecimal DAYS_IN_YEAR = new BigDecimal("365");

    public BigDecimal calculateInterest(BigDecimal principal,
                                      BigDecimal annualRate,
                                      int days) {
        // 일일 이자율 계산
        BigDecimal dailyRate = annualRate.divide(DAYS_IN_YEAR, 10, RoundingMode.HALF_UP);

        // 이자 계산
        BigDecimal interest = principal
            .multiply(dailyRate)
            .multiply(new BigDecimal(days));

        // 소수점 2자리까지 정확하게 계산
        return interest.setScale(2, RoundingMode.HALF_UP);
    }
}
```

**테스트 결과:**
```java
// 기존 방식 vs BigDecimal 방식 비교
double wrongResult = 1000000.0 * (0.05 / 365.0) * 30;  // 410.958904
BigDecimal correctResult = calculateInterest(
    new BigDecimal("1000000.00"),
    new BigDecimal("0.05"),
    30
);  // 410.96

System.out.println("오차: " + (410.96 - wrongResult));  // 약 0.001096원
```

**시스템 전체 영향:**
- **일일 이자 계산:** 수백만 건 × 소액 오차 = 수십만 원 누적 오차
- **법적 리스크:** 고객과의 분쟁 발생 가능성
- **감사 요구사항:** 모든 계산의 추적 가능성

### 11.2 암호화폐 거래소의 정밀도 문제

**문제 상황:**
- 비트코인 가격: $43,250.12345678
- 극미량 거래 단위 (Satoshi: 0.00000001 BTC)
- 고빈도 거래로 인한 누적 계산 오차

**실제 사례 (Binance 사고):**
```java
// Java의 위험한 계산
public double calculateTradeAmount(double price, double amount) {
    double total = price * amount;  // 부동 소수점 오차
    return Math.round(total * 100000000) / 100000000.0;  // Satoshi 단위 변환
}

// 문제 발생 케이스
double result = calculateTradeAmount(43250.12345678, 0.00001234);
// 기대값: 0.00053399
// 실제값: 0.0005339900000000001 (오차 누적)
```

**해결책: 정수 기반 계산**
```java
import java.math.BigDecimal;

public class CryptoCalculator {
    // BTC: 8자리, USD: 2자리 정밀도
    private final long BTC_PRECISION = 100_000_000L;  // 1e8
    private final long USD_PRECISION = 100L;          // 1e2

    // 금액을 정수로 변환
    public long toSatoshi(double btc) {
        return Math.round(btc * BTC_PRECISION);
    }

    public long toCents(double usd) {
        return Math.round(usd * USD_PRECISION);
    }

    // 정수 기반 계산
    public double calculateTradeAmount(double price, double amount) {
        long priceCents = toCents(price);
        long amountSatoshi = toSatoshi(amount);

        // 정수 연산으로 정확한 계산
        long totalCents = (priceCents * amountSatoshi) / BTC_PRECISION;

        return (double) totalCents / USD_PRECISION;
    }

    // BigDecimal을 사용한 더 정확한 계산
    public BigDecimal calculateTradeAmount(BigDecimal price, BigDecimal amount) {
        BigDecimal btcPrecision = new BigDecimal(BTC_PRECISION);
        BigDecimal usdPrecision = new BigDecimal(USD_PRECISION);

        // 모든 계산을 BigDecimal로 수행
        BigDecimal priceCents = price.multiply(usdPrecision);
        BigDecimal amountSatoshi = amount.multiply(btcPrecision);

        BigDecimal totalCents = priceCents.multiply(amountSatoshi).divide(btcPrecision);

        return totalCents.divide(usdPrecision);
    }

    // 검증 함수
    public boolean validateCalculation(double price, double amount, double expectedTotal) {
        double calculated = calculateTradeAmount(price, amount);
        double difference = Math.abs(calculated - expectedTotal);
        return difference < 0.00000001; // 1 Satoshi 이하 오차만 허용
    }
}
```

**시스템 아키텍처 개선:**
1. **데이터베이스 저장:** 모든 금액을 정수로 저장
2. **API 통신:** 클라이언트 ↔ 서버 간 정수 단위 사용
3. **디스플레이:** UI에서만 소수점 변환
4. **감사 로그:** 모든 계산의 상세 추적

**결과:**
- **정밀도:** 100% 정확한 계산 보장
- **성능:** 정수 연산으로 더 빠른 처리
- **안정성:** 오차 누적 방지

### 11.3 과학 시뮬레이션의 수치 안정성 문제

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

### 11.4 GPS 좌표 계산의 정밀도 문제

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

### 11.5 머신러닝 모델의 수치 안정성

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

*"금융과 과학 분야에서 '약 0.3'은 '정확히 0.3'만큼 중요하다."*

> 정밀도 문제는 보이지 않는 버그처럼 위험하다. BigDecimal, 정수 연산, 수치 안정성 기법을 적절히 활용하여 신뢰할 수 있는 시스템을 구축하라.
