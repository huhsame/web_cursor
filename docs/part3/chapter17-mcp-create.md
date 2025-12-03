# Chapter 17. MCP 직접 만들기 (심화)

> 💡 **이 챕터는 선택 사항입니다!**
>
> MCP 사용에 익숙해진 후,
> 나만의 MCP를 만들고 싶을 때 참고하세요.

---

## 17.1 왜 MCP를 직접 만들까요?

### 이런 경우에 유용합니다

| 상황 | 예시 |
|------|------|
| 내부 시스템 연결 | 회사 API, 내부 DB |
| 특수한 기능 필요 | 맞춤형 데이터 처리 |
| 공개된 MCP가 없음 | 새로운 서비스 연동 |
| 학습 목적 | MCP 구조 이해 |

### 만들 수 있는 것들

- 날씨 API 연동 MCP
- 환율 계산 MCP
- 회사 내부 DB 조회 MCP
- 특정 API 호출 MCP
- 파일 처리 MCP

---

## 17.2 FastMCP 소개

### FastMCP란?

> **MCP 서버를 쉽게 만들 수 있는 Python 라이브러리**입니다.
>
> 복잡한 프로토콜을 몰라도 함수만 만들면 됩니다!

### 왜 Python인가요?

| 언어 | MCP 라이브러리 | 난이도 |
|------|---------------|--------|
| **Python** | FastMCP | 쉬움 |
| TypeScript | @modelcontextprotocol/sdk | 중간 |
| Go | mcp-go | 어려움 |

> 💡 **Python이 가장 간단합니다!**
> 다른 언어로도 만들 수 있지만, 학습용으로는 Python 추천.

---

## 17.3 개발 환경 준비

### Step 1: Python 설치 확인

터미널에서:

```bash
python3 --version
```

**Python 3.10 이상**이 필요합니다.

설치 안 되어 있다면:
- Mac: `brew install python`
- Windows: [python.org](https://www.python.org/) 에서 다운로드

### Step 2: 프로젝트 폴더 생성

```bash
mkdir my-mcp-server
cd my-mcp-server
```

### Step 3: 가상환경 생성 (선택)

```bash
python3 -m venv venv
source venv/bin/activate  # Mac/Linux
# 또는
venv\Scripts\activate  # Windows
```

### Step 4: FastMCP 설치

```bash
pip install fastmcp
```

---

## 17.4 첫 번째 MCP 만들기

### 덧셈 MCP 만들기

가장 간단한 예제로 **두 숫자를 더하는 MCP**를 만들어봅시다.

### Step 1: 파일 생성

`server.py` 파일 생성:

```python
from fastmcp import FastMCP

# MCP 서버 생성
mcp = FastMCP("calculator")

# 도구(Tool) 정의
@mcp.tool()
def add_numbers(a: int, b: int) -> int:
    """두 숫자를 더합니다."""
    return a + b

@mcp.tool()
def multiply_numbers(a: int, b: int) -> int:
    """두 숫자를 곱합니다."""
    return a * b

# 서버 실행
if __name__ == "__main__":
    mcp.run()
```

### 코드 설명

```python
from fastmcp import FastMCP      # FastMCP 불러오기
mcp = FastMCP("calculator")      # "calculator"라는 이름의 MCP 생성

@mcp.tool()                      # 이 함수를 MCP 도구로 등록
def add_numbers(a: int, b: int) -> int:
    """두 숫자를 더합니다."""   # 이 설명을 AI가 읽습니다!
    return a + b

mcp.run()                        # 서버 실행
```

### Step 2: 서버 실행

```bash
python server.py
```

실행되면:
```
INFO:     Started server process
INFO:     Uvicorn running on http://localhost:8000
```

---

## 17.5 Cursor에 연결하기

### Step 1: mcp.json 수정

`.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "calculator": {
      "command": "python",
      "args": ["server.py"],
      "cwd": "/path/to/my-mcp-server"
    }
  }
}
```

> ⚠️ **cwd를 실제 경로로 바꾸세요!**

### Step 2: Cursor 재시작

MCP 목록에서 **calculator**가 ✅ 상태인지 확인

### Step 3: 테스트

Cursor Chat에서:

```
calculator MCP로 15 + 27 계산해줘
```

예상 결과:
```
AI: calculator MCP의 add_numbers 도구를 사용했습니다.
15 + 27 = 42
```

---

## 17.6 실용적인 MCP 만들기

### 환율 계산 MCP

실제로 쓸 수 있는 **환율 계산 MCP**를 만들어봅시다.

```python
from fastmcp import FastMCP
import requests

mcp = FastMCP("currency")

@mcp.tool()
def get_exchange_rate(from_currency: str, to_currency: str) -> dict:
    """
    두 통화 간의 환율을 조회합니다.

    Args:
        from_currency: 기준 통화 (예: USD, KRW, JPY)
        to_currency: 대상 통화 (예: USD, KRW, JPY)

    Returns:
        환율 정보
    """
    # 무료 환율 API 사용
    url = f"https://api.exchangerate-api.com/v4/latest/{from_currency}"
    response = requests.get(url)
    data = response.json()

    rate = data["rates"].get(to_currency)

    return {
        "from": from_currency,
        "to": to_currency,
        "rate": rate,
        "message": f"1 {from_currency} = {rate} {to_currency}"
    }

@mcp.tool()
def convert_currency(amount: float, from_currency: str, to_currency: str) -> dict:
    """
    금액을 다른 통화로 변환합니다.

    Args:
        amount: 변환할 금액
        from_currency: 기준 통화
        to_currency: 대상 통화

    Returns:
        변환된 금액
    """
    rate_info = get_exchange_rate(from_currency, to_currency)
    rate = rate_info["rate"]
    converted = amount * rate

    return {
        "original": f"{amount} {from_currency}",
        "converted": f"{converted:.2f} {to_currency}",
        "rate": rate
    }

if __name__ == "__main__":
    mcp.run()
```

### 사용 예시

```
100달러를 원화로 바꾸면 얼마야?
```

```
AI: currency MCP의 convert_currency를 사용했습니다.

100 USD = 133,450.00 KRW
(환율: 1 USD = 1,334.50 KRW)
```

---

## 17.7 MCP 도구 작성 팁

### 1. 명확한 Docstring 작성

AI가 **Docstring을 읽고** 도구 사용법을 이해합니다.

```python
@mcp.tool()
def search_products(
    keyword: str,
    min_price: int = 0,
    max_price: int = 1000000
) -> list:
    """
    상품을 검색합니다.

    Args:
        keyword: 검색할 키워드
        min_price: 최소 가격 (기본값: 0)
        max_price: 최대 가격 (기본값: 1000000)

    Returns:
        검색된 상품 목록
    """
    # 구현...
```

### 2. 타입 힌트 사용

```python
# 좋은 예 ✅
def add(a: int, b: int) -> int:

# 나쁜 예 ❌
def add(a, b):
```

### 3. 에러 처리

```python
@mcp.tool()
def get_user(user_id: int) -> dict:
    """사용자 정보를 조회합니다."""
    try:
        user = database.get_user(user_id)
        if not user:
            return {"error": "사용자를 찾을 수 없습니다"}
        return user
    except Exception as e:
        return {"error": str(e)}
```

### 4. 반환값 구조화

```python
# 좋은 예 ✅
return {
    "success": True,
    "data": result,
    "message": "조회 완료"
}

# 나쁜 예 ❌
return result
```

---

## 17.8 MCP 배포하기 (선택)

### 로컬에서만 쓸 경우

지금까지 한 방식으로 충분합니다!
- 내 컴퓨터에서 실행
- Cursor에서 연결해서 사용

### 팀과 공유하려면?

**클라우드 서버에 배포**하면 됩니다.

1. **서버 준비** (AWS, GCP, Vercel 등)
2. **MCP 서버 배포**
3. **HTTP 방식으로 연결**

```json
{
  "mcpServers": {
    "my-mcp": {
      "type": "http",
      "url": "https://my-mcp-server.com/mcp"
    }
  }
}
```

> 💡 **고급 주제이므로 필요할 때 찾아보세요!**

---

## 17.9 더 배우려면

### 공식 문서

- **MCP 공식 문서**: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **FastMCP**: [github.com/jlowin/fastmcp](https://github.com/jlowin/fastmcp)

### 예제 MCP 살펴보기

GitHub에서 다른 사람들이 만든 MCP를 참고:

```
github.com/topics/model-context-protocol
```

### 커뮤니티

- Smithery에서 인기 MCP 분석
- Discord, Reddit 등에서 정보 공유

---

## 핵심 정리

```
✅ FastMCP = Python으로 MCP를 쉽게 만드는 라이브러리
✅ @mcp.tool() 데코레이터로 도구 등록
✅ Docstring이 중요! (AI가 읽음)
✅ 타입 힌트 필수
✅ 로컬에서 테스트 후 필요시 배포
```

---

## Part 3를 마치며

축하합니다! 🎉

이제 여러분은:

1. **MCP가 무엇인지** 이해했습니다
2. **Cursor에 MCP를 연결**할 수 있습니다
3. **Context7으로 최신 문서를 참조**할 수 있습니다
4. **Supabase MCP로 DB를 조작**할 수 있습니다
5. (선택) **나만의 MCP를 만들** 수 있습니다

AI의 능력을 확장하는 MCP를 잘 활용해서
더 효율적인 개발을 해보세요!

---

## 부록으로

> 자주 묻는 질문과 문제 해결 가이드를 확인하세요!
>
> 👉 [자주 묻는 질문 (FAQ)](../appendix/faq.md)
