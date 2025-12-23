컴퓨터의 **실수 표현(IEEE 754)**만큼이나 개발자들이 자주 오해하고, 실무에서 치명적인 버그(데이터 손실, 깨짐)를 유발하는 또 다른 주제는 **'문자열 인코딩(Character Encoding)과 유니코드'**입니다.

GitHub 업로드용 `README.md` 형식으로 정리했습니다.

---

# 컴퓨터의 문자 표현: 유니코드(Unicode)와 UTF-8의 오해와 진실

## 1. 핵심 요약 (Executive Summary)

컴퓨터는 내부적으로 0과 1만 다룰 수 있으므로, 문자(Character)를 저장하기 위해서는 숫자에 1:1로 매핑하는 **약속(Map)**이 필요하다. 과거에는 국가별로 다른 인코딩(EUC-KR, CP949 등)을 사용하여 호환성 문제가 심각했으나, 현재는 전 세계 모든 문자를 하나로 통합한 **유니코드(Unicode)**가 표준이다.

> **결론:** 현대의 모든 시스템(Web, DB, Linux, Source Code)에서 인코딩은 무조건 **UTF-8**을 사용해야 한다. 특히 데이터베이스(MySQL/MariaDB) 설정 시 `utf8`이 아닌 **`utf8mb4`**를 사용해야 이모지(Emoji)와 특수문자가 깨지지 않는다.

---

## 2. 문자 집합(Charset) vs 인코딩(Encoding)

많은 개발자가 이 두 개념을 혼동하지만, 엄격히 구분해야 한다.

| 구분 | 설명 | 비유 | 예시 |
| --- | --- | --- | --- |
| **문자 집합 (Character Set)** | 표현할 수 있는 문자들의 리스트와 식별 번호(Code Point)의 모음. | **추상적인 지도** | 유니코드(Unicode), ASCII |
| **인코딩 (Encoding)** | 문자 집합의 번호를 컴퓨터가 이해하는 2진수(Binary)로 변환하는 기술적 방식. | **지도를 접는 방법** | UTF-8, UTF-16, EUC-KR |

### 2.1 유니코드 (Unicode)

* **정의:** "세상의 모든 문자에 고유 번호를 부여하자."
* **형태:** `U+` 뒤에 16진수 숫자를 붙여 표현한다.
* `A` = `U+0041`
* `가` = `U+AC00`
* `🔥` (Fire Emoji) = `U+1F525`


* **중요:** 유니코드는 **'번호'일 뿐, 저장 방식(바이트)**이 아니다.

---

## 3. 핵심 원리: UTF-8은 어떻게 저장되는가?

가장 널리 쓰이는 **UTF-8**은 **가변 길이 인코딩(Variable-width encoding)** 방식을 사용한다. 문자의 종류에 따라 1~4바이트를 유동적으로 할당하여 메모리를 절약한다.

### 3.1 UTF-8 비트 구조

| 문자 범위 (Code Point) | 필요 바이트 | 비트 패턴 (x는 데이터 비트) |
| --- | --- | --- |
| `U+0000` ~ `U+007F` (영어/숫자) | **1 Byte** | `0xxxxxxx` |
| `U+0080` ~ `U+07FF` (유럽어 등) | **2 Bytes** | `110xxxxx` `10xxxxxx` |
| `U+0800` ~ `U+FFFF` (한글 등) | **3 Bytes** | `1110xxxx` `10xxxxxx` `10xxxxxx` |
| `U+10000` ~ `U+10FFFF` (이모지) | **4 Bytes** | `11110xxx` `10xxxxxx` `10xxxxxx` `10xxxxxx` |

### 3.2 저장 예시 비교

* **'A' (`U+0041`)**: 범위가 1바이트 구간이므로 `01000001` (8비트) 그대로 저장. (ASCII와 100% 호환)
* **'가' (`U+AC00`)**: 범위가 3바이트 구간이므로, 2진수 데이터를 쪼개서 `1110xxxx`, `10xxxxxx`, `10xxxxxx` 틀 안에 채워 넣음.

---

## 4. 왜 오차가 발생(글자가 깨짐)하는가?

"믜", "퍞" 같은 글자가 깨지거나 `` (Replacement Character)로 보이는 현상은 **해석기(Decoder)와 저장기(Encoder)의 약속이 달라서** 발생한다.

### 4.1 Mojibake (글자 깨짐) 메커니즘

1. **전제:** 서버가 '한글'이라는 문자열을 **UTF-8** 방식으로 포장(3바이트)해서 보냈다.
2. **상황:** 브라우저나 클라이언트가 이를 **EUC-KR**(2바이트 방식)로 해석하려고 시도한다.
3. **결과:** 바이트 경계가 어긋나면서 전혀 엉뚱한 문자로 매핑되거나, 매핑되지 않는 바이트를 만나 물음표(?)나 깨진 문자로 표시한다.

---

## 5. 더 나은 접근 방식 (Industry Standard Solutions)

### 5.1 데이터베이스: `utf8` vs `utf8mb4`

MySQL 계열(MySQL, MariaDB)에서 가장 흔한 실수다.

* **MySQL의 `utf8`:** 3바이트까지만 저장 가능하다. (과거 최적화 문제로 인한 비표준 구현)
* **문제점:** 이모지(`🔥`, 4바이트)나 일부 고어, 특수 한자를 저장하려고 하면 **데이터가 잘리거나 에러가 발생**한다.
* **해결:** 반드시 **`utf8mb4`** (4 Byte support)를 Charset으로 설정해야 한다.

```sql
-- [BAD] 이모지 저장 불가
CREATE DATABASE mydb CHARACTER SET utf8 COLLATE utf8_general_ci;

-- [GOOD] 업계 표준 (이모지 포함 전 세계 문자 완벽 지원)
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

```

### 5.2 프로그래밍: 문자열 길이 산정 (`len()`의 함정)

언어마다 문자열 길이를 세는 방식이 다르므로 유효성 검사(Validation) 시 주의해야 한다.

```python
# Python 3 (유니코드 기준)
s = "A가🔥"
print(len(s)) 
# 출력: 3 (사람이 보는 글자 수 기준)

# Byte 기준 (네트워크 전송, DB 컬럼 크기 산정 시 필요)
print(len(s.encode('utf-8')))
# 출력: 1('A') + 3('가') + 4('🔥') = 8 bytes

```

> **Action Item:** DB의 `VARCHAR(N)`은 보통 글자 수를 의미하지만, 바이트 제한이 있는 레거시 시스템과 통신할 때는 반드시 **Byte Length**를 체크해야 한다.

---

## 6. 전문가적 조언 (Pro Tip)

### 6.1 정규화 (Normalization): NFC vs NFD

운영체제마다 한글을 저장하는 내부 방식이 다르다. 파일 업로드/다운로드 기능을 구현할 때 주로 발생한다.

* **NFC (Normalization Form C):** '각' (한 덩어리) - **Windows, Linux, Web 표준**
* **NFD (Normalization Form D):** 'ㄱ' + 'ㅏ' + 'ㄱ' (자소 분리) - **macOS**

**시나리오:** 맥북 사용자가 올린 파일명 "보고서.pdf"가 윈도우 사용자에게 "ㅂㅗㄱㅗㅅㅓ.pdf"로 보여서 파일이 열리지 않는 현상.
**해결:** 서버에서 파일명을 받을 때 반드시 **NFC로 정규화(Normalize)** 하는 코드를 수행한 후 저장해야 한다.

```python
import unicodedata

# macOS에서 온 자소 분리된 문자열
mac_str = 'ᄒ' + 'ᅡ' + 'ᆫ' + 'ᄀ' + 'ᅳ' + 'ᆯ' # 실제로는 한 글자처럼 보임
print(mac_str) 

# 서버 저장 전 정규화 필수
normalized_str = unicodedata.normalize('NFC', mac_str)

```

### 6.2 BOM (Byte Order Mark) 주의

UTF-8 파일 맨 앞에 `0xEF, 0xBB, 0xBF`라는 3바이트 표식이 붙는 경우가 있다(UTF-8 with BOM). 이는 윈도우 메모장 등에서 주로 붙인다.

* **위험:** JSON 파싱이나 소스 코드 컴파일 시 "Unexpected token" 에러를 유발한다.
* **대응:** 텍스트 파일을 읽을 때 BOM이 있다면 제거하거나, `UTF-8-SIG` (Python 등) 인코딩 옵션을 사용하여 처리해야 한다.

---
