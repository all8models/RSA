# 🔐 RSA 암호 구조 이해와 구현 (RSA Cryptosystem Study & Implementation)

이 프로젝트는 현대 인터넷 보안의 핵심인 **공개키 암호 알고리즘(RSA)**의 수학적 원리를 탐구하고, 이를 Python 코드로 직접 구현하여 시각적으로 이해하기 위한 교육용 자료입니다.

Jupyter Notebook을 통해 소수 생성부터 모듈러 연산, 확장 유클리드 호제법을 이용한 키 생성, 암호화/복호화, 그리고 디지털 서명까지 단계별로 직접 실행하고 시각화된 결과를 확인할 수 있습니다.

---

## 📂 프로젝트 구조

- **[`rsa_구조_이해.ipynb`](file:///Users/windows/work/Demo/RSA/rsa_구조_이해.ipynb)**: 단계별 수학적 배경 설명, Python 구현 코드, Matplotlib을 활용한 시각화 그래프가 포함된 핵심 메인 실습 노트북입니다.
- **[`plan.md`](file:///Users/windows/work/Demo/RSA/plan.md)**: RSA의 기본 비유(구슬 팔찌), 작동 원리, 안전성, 하이브리드 암호화(AES 혼용) 및 양자 컴퓨터의 위협 등을 요약 정리한 설명 문서입니다.

---

## 🧮 핵심 수학 및 동작 원리

RSA 암호는 **"두 소수의 곱을 구하는 것은 쉽지만, 그 곱을 다시 소인수분해하는 것은 매우 어렵다"**는 수학적 난제(일방향 함수)에 기반합니다.

### 1. 키 생성 과정
1. **두 소수 생성**: 서로 다른 임의의 큰 소수 $p$, $q$를 비밀리에 선택합니다.
2. **모듈러스 $n$ 계산**: $n = p \times q$를 계산합니다. (공개키 및 개인키의 법(modulus)으로 사용되며 공개됩니다.)
3. **오일러 피 함수 $\phi(n)$ 계산**: $\phi(n) = (p-1)(q-1)$을 계산합니다. (비밀로 유지됩니다.)
4. **공개 지수 $e$ 선택**: $1 < e < \phi(n)$ 이며 $\gcd(e, \phi(n)) = 1$을 만족하는 값(일반적으로 $65537$)을 선택합니다.
5. **개인 지수 $d$ 계산**: $e \times d \equiv 1 \pmod{\phi(n)}$을 만족하는 모듈러 역원 $d$를 확장 유클리드 호제법으로 구합니다. (비밀로 유지됩니다.)

* **공개키 (Public Key)**: $(n, e)$
* **개인키 (Private Key)**: $(n, d)$

### 2. 암호화 및 복호화
* **암호화 (Encryption)**: 송신자는 공개키 지수 $e$와 $n$을 사용하여 평문 $M$을 암호문 $C$로 변환합니다.
  $$C = M^e \bmod n$$
* **복호화 (Decryption)**: 수신자는 개인키 지수 $d$와 $n$을 사용하여 암호문 $C$를 원래의 평문 $M$으로 복원합니다.
  $$M = C^d \bmod n$$

### 3. 디지털 서명 및 검증
* **서명 (Signing)**: 송신자는 자신의 개인키 지수 $d$를 사용하여 문서 $M$의 서명 $S$를 생성합니다.
  $$S = M^d \bmod n$$
* **검증 (Verification)**: 수신자는 송신자의 공개키 지수 $e$를 사용하여 서명이 올바른지 검증합니다.
  $$M = S^e \bmod n$$

---

## 💻 주요 코드 구현 요약

`rsa_구조_이해.ipynb` 노트북에는 다음과 같은 파이썬 핵심 코드가 포함되어 있습니다.

### 🔑 소수 생성 및 키 생성
```python
import sympy as sp
import math

# 지정된 비트 수의 소수 생성
def generate_large_prime(bits=16):
    return sp.randprime(2**(bits-1), 2**bits)

# 확장 유클리드 호제법을 이용한 모듈러 역원 계산
def mod_inverse(e, phi):
    def extended_gcd(a, b):
        if b == 0:
            return a, 1, 0
        g, x1, y1 = extended_gcd(b, a % b)
        return g, y1, x1 - (a // b) * y1
    
    g, x, _ = extended_gcd(e, phi)
    if g != 1:
        raise ValueError("역원이 존재하지 않습니다.")
    return x % phi
```

### 🔒 암호화 & 복호화 / 서명 & 검증
```python
# RSA 암호화 및 복호화
def rsa_encrypt(message, e, n):
    return pow(message, e, n)

def rsa_decrypt(cipher, d, n):
    return pow(cipher, d, n)

# RSA 서명 및 검증
def rsa_sign(message, d, n):
    return pow(message, d, n)

def rsa_verify(signature, e, n):
    return pow(signature, e, n)
```

### 🔠 문자열 메시지 암호화
```python
# 문자열을 바이트 정수로 변환하여 암호화 진행
def string_to_int(text):
    return int.from_bytes(text.encode('utf-8'), byteorder='big')

def int_to_string(number):
    try:
        return number.to_bytes((number.bit_length() + 7) // 8, byteorder='big').decode('utf-8')
    except:
        return f"[디코딩 불가: {number}]"
```

---

## 📊 시각화 기능

실습 노트북에서는 다음과 같은 시각화 그래프를 제공하여 암호화 알고리즘의 거동을 눈으로 쉽게 확인할 수 있습니다.

1. **RSA 핵심 값의 관계**: 소수 $p$, $q$로부터 유도되는 $n$(공개)과 $\phi(n)$(비밀)의 상관 관계 도식화
2. **암호화/복호화 단계**: 평문에서 암호문으로의 변환 및 개인키 지수 $d$를 통한 복구 과정 도식화
3. **일방향성 시각화**: 암호화 함수 $f(x) = x^e \bmod n$의 결과가 복잡하게 분산(흩어짐)되어 있어 $e$만으로는 역산이 어려움을 보여주는 산점도 그래프

---

## 🛠 시작하기

### 1. 환경 준비
이 프로젝트는 가상환경을 사용하여 라이브러리를 설치할 수 있습니다.
```bash
# 가상환경 활성화 (macOS)
source .venv/bin/activate
```

### 2. 패키지 설치
노트북 실행 및 시각화를 위해 아래 라이브러리가 필요합니다.
```bash
pip install sympy matplotlib numpy jupyter
```

### 3. 주피터 노트북 실행
```bash
jupyter notebook rsa_구조_이해.ipynb
```
또는 VS Code나 PyCharm 등 개발 환경에서 바로 노트북을 열어 실행하실 수 있습니다.
