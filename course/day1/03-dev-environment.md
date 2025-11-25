# Day 1 - 3교시: 개발 환경 설정

## 학습 목표
- Node.js를 설치하고 이해한다
- Git을 설치하고 기본 명령어를 익힌다
- GitHub 계정을 만들고 연동한다
- 터미널 사용법을 익힌다

---

## 1. Node.js란?

### 쉬운 설명

> JavaScript는 원래 브라우저 안에서만 실행됐어요.
> 마치 물고기가 물 안에서만 살 수 있는 것처럼요.
>
> **Node.js**는 JavaScript를 컴퓨터에서도
> 실행할 수 있게 해주는 도구입니다.
> 물고기가 육지에서도 살 수 있게 해주는
> 특수 장비 같은 거예요!

### 공식적인 설명
```
Node.js는 Chrome V8 JavaScript 엔진으로 빌드된
JavaScript 런타임입니다.
서버 사이드 애플리케이션 개발을 가능하게 하며,
npm(Node Package Manager)을 통해
수많은 오픈소스 패키지를 사용할 수 있습니다.
```

### 왜 Node.js가 필요한가요?

| 이유 | 설명 |
|------|------|
| Next.js 실행 | Next.js가 Node.js 위에서 동작함 |
| 패키지 설치 | npm으로 라이브러리 설치 |
| 개발 서버 | 로컬에서 웹사이트 미리보기 |
| 빌드 | 배포용 파일 생성 |

---

## 2. Node.js 설치하기

### Step 1: 다운로드

1. [nodejs.org](https://nodejs.org) 접속
2. **LTS 버전** 다운로드 (안정적인 버전)

```
💡 LTS = Long Term Support (장기 지원 버전)
   최신 기능보다 안정성이 중요하므로 LTS를 선택하세요.
```

### Step 2: 설치

**Windows:**
1. 다운로드된 `.msi` 파일 실행
2. "Next" 계속 클릭
3. 설치 완료

**Mac:**
1. 다운로드된 `.pkg` 파일 실행
2. 안내에 따라 설치
3. 완료

### Step 3: 설치 확인

**터미널 열기:**
- Windows: `Win + R` → `cmd` 입력 → 엔터
- Mac: `Cmd + Space` → `terminal` 입력 → 엔터

**명령어 입력:**
```bash
node --version
```

**예상 결과:**
```
v20.10.0   (버전 숫자는 다를 수 있음)
```

**npm도 확인:**
```bash
npm --version
```

**예상 결과:**
```
10.2.3   (버전 숫자는 다를 수 있음)
```

---

## 3. 터미널 기초

### 쉬운 설명

> 터미널은 **컴퓨터와 대화하는 창**입니다.
>
> 마우스로 클릭하는 대신,
> 글자를 입력해서 컴퓨터에게 명령합니다.
>
> 처음엔 어렵지만, 익숙해지면 마우스보다 빨라요!

### 꼭 알아야 할 명령어 5가지

| 명령어 | 뜻 | 예시 |
|--------|-----|------|
| `cd` | 폴더 이동 | `cd Desktop` |
| `ls` (Mac) / `dir` (Win) | 파일 목록 보기 | `ls` |
| `mkdir` | 폴더 만들기 | `mkdir my-project` |
| `pwd` | 현재 위치 확인 | `pwd` |
| `clear` | 화면 지우기 | `clear` |

### 실습: 터미널 사용해보기

```bash
# 1. 현재 위치 확인
pwd

# 2. 바탕화면으로 이동
cd Desktop

# 3. 새 폴더 만들기
mkdir test-folder

# 4. 폴더 목록 확인
ls

# 5. 만든 폴더로 이동
cd test-folder

# 6. 다시 위치 확인
pwd
```

### 경로 이해하기

```
절대 경로: /Users/홍길동/Desktop/my-project
          (처음부터 전체 경로)

상대 경로: ./my-project 또는 my-project
          (현재 위치 기준)

상위 폴더: ..
          (한 단계 위로)
```

**예시:**
```bash
cd ..              # 상위 폴더로 이동
cd ../other-folder # 상위 폴더의 다른 폴더로 이동
cd ~               # 홈 폴더로 이동
```

---

## 4. Git이란?

### 쉬운 설명

> 문서 작업할 때 이런 파일들 본 적 있죠?
>
> - 보고서_최종.docx
> - 보고서_최종_수정.docx
> - 보고서_최종_수정_진짜최종.docx
> - 보고서_최종_수정_진짜최종_이게진짜.docx
>
> Git은 이런 혼란을 없애줍니다.
> **변경 사항을 자동으로 기록**해주거든요!

### 공식적인 설명
```
Git은 분산 버전 관리 시스템(DVCS)입니다.
코드의 변경 이력을 추적하고,
여러 개발자가 동시에 작업할 수 있게 하며,
이전 버전으로의 복구를 가능하게 합니다.
```

### Git의 장점

```
1. 타임머신 기능
   └─ 언제든 과거 버전으로 돌아갈 수 있음

2. 협업 기능
   └─ 여러 사람이 동시에 작업 가능

3. 백업 기능
   └─ 코드가 날아가도 복구 가능

4. 브랜치 기능
   └─ 새 기능을 안전하게 실험 가능
```

---

## 5. Git 설치하기

### Windows

1. [git-scm.com](https://git-scm.com) 접속
2. "Download for Windows" 클릭
3. 설치 파일 실행
4. 기본 옵션으로 "Next" 계속 클릭

### Mac

**방법 1: Xcode Command Line Tools**
```bash
xcode-select --install
```

**방법 2: Homebrew (추천)**
```bash
# Homebrew 설치 (없는 경우)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Git 설치
brew install git
```

### 설치 확인

```bash
git --version
```

**예상 결과:**
```
git version 2.43.0
```

---

## 6. Git 초기 설정

### 사용자 정보 설정 (필수!)

```bash
# 이름 설정 (GitHub 가입할 이름과 동일하게)
git config --global user.name "홍길동"

# 이메일 설정 (GitHub 가입할 이메일과 동일하게)
git config --global user.email "gildong@email.com"
```

### 설정 확인

```bash
git config --list
```

---

## 7. GitHub 계정 만들기

### GitHub이란?

> Git으로 관리하는 코드를 **온라인에 저장**하는 곳입니다.
>
> 마치 Google Drive가 문서를 클라우드에 저장하듯,
> GitHub은 코드를 클라우드에 저장합니다.
>
> 전 세계 개발자들이 코드를 공유하는
> **코드계의 인스타그램** 같은 곳이에요!

### 계정 만들기

1. [github.com](https://github.com) 접속
2. "Sign up" 클릭
3. 이메일, 비밀번호, 사용자명 입력
4. 이메일 인증

```
💡 사용자명 팁:
   - 영문, 숫자, 하이픈(-) 사용 가능
   - 나중에 포트폴리오가 되므로 신중하게!
   - 예: hong-gildong, gildong-dev
```

---

## 8. Git 기본 명령어

### 핵심 명령어 4가지

```bash
# 1. 저장소 초기화 (프로젝트 시작할 때 1번만)
git init

# 2. 변경된 파일 추가 (저장할 파일 선택)
git add .

# 3. 커밋 (변경 사항 저장)
git commit -m "변경 내용 설명"

# 4. 푸시 (GitHub에 업로드)
git push
```

### 그림으로 이해하기

```
[작업 디렉토리]     [스테이징 영역]     [로컬 저장소]     [원격 저장소]
     │                   │                  │               │
     │    git add        │                  │               │
     │──────────────────▶│                  │               │
     │                   │    git commit    │               │
     │                   │─────────────────▶│               │
     │                   │                  │   git push    │
     │                   │                  │──────────────▶│
     │                   │                  │               │
   파일 수정           준비 완료          저장 완료       GitHub 업로드
```

### 쉬운 비유

```
1. git add    = 택배 상자에 물건 담기
2. git commit = 상자 포장하고 송장 붙이기
3. git push   = 택배 발송하기
```

---

## 9. GitHub에 처음 연결하기

### SSH 키 생성 (한 번만 하면 됨)

```bash
# SSH 키 생성
ssh-keygen -t ed25519 -C "your_email@example.com"

# 엔터 3번 (기본값 사용)
```

### SSH 키 GitHub에 등록

```bash
# Mac: 공개키 복사
cat ~/.ssh/id_ed25519.pub | pbcopy

# Windows: 공개키 복사
cat ~/.ssh/id_ed25519.pub | clip
```

**GitHub에서:**
1. GitHub 접속 → 오른쪽 위 프로필 → Settings
2. 왼쪽 메뉴에서 "SSH and GPG keys"
3. "New SSH key" 클릭
4. Title: "My Computer" (아무 이름)
5. Key: 복사한 내용 붙여넣기
6. "Add SSH key" 클릭

### 연결 테스트

```bash
ssh -T git@github.com
```

**예상 결과:**
```
Hi username! You've successfully authenticated...
```

---

## 10. 실습: Git 워크플로우 체험

### Step 1: 테스트 폴더 만들기

```bash
cd Desktop
mkdir git-practice
cd git-practice
```

### Step 2: Git 초기화

```bash
git init
```

**결과:**
```
Initialized empty Git repository in .../git-practice/.git/
```

### Step 3: 파일 만들기

```bash
# Mac/Linux
echo "안녕하세요!" > hello.txt

# Windows (PowerShell)
"안녕하세요!" | Out-File hello.txt
```

### Step 4: 상태 확인

```bash
git status
```

**결과:**
```
Untracked files:
  hello.txt
```

### Step 5: 파일 추가 & 커밋

```bash
git add .
git commit -m "첫 번째 커밋: hello.txt 추가"
```

### Step 6: 로그 확인

```bash
git log --oneline
```

**결과:**
```
abc1234 첫 번째 커밋: hello.txt 추가
```

---

## 핵심 정리

```
✅ Node.js = JavaScript를 컴퓨터에서 실행하는 환경
✅ npm = Node.js 패키지 관리자
✅ Git = 코드 버전 관리 도구
✅ GitHub = 코드를 온라인에 저장하는 서비스
✅ git add → commit → push 흐름 기억하기
```

---

## 설치 체크리스트

```
□ Node.js 설치됨 (node --version 확인)
□ npm 설치됨 (npm --version 확인)
□ Git 설치됨 (git --version 확인)
□ Git 사용자 설정 완료
□ GitHub 계정 생성 완료
□ SSH 키 등록 완료
```

---

## 자주 발생하는 문제

### "command not found" 에러
```
터미널을 껐다가 다시 열어보세요.
그래도 안 되면 컴퓨터를 재시작하세요.
```

### Git 한글 깨짐
```bash
git config --global core.quotepath false
```

### SSH 연결 실패
```
1. SSH 키가 제대로 생성되었는지 확인
2. GitHub에 등록한 키가 맞는지 확인
3. ssh-agent가 실행 중인지 확인
```

---

## 다음 시간 예고

> 드디어 첫 Next.js 프로젝트를 만듭니다!
> 설치한 도구들을 실제로 사용해볼 거예요.
> 커서와 함께 "Hello World"를 웹페이지로 만들어봅시다!
