# [초보자용] WSL 기반 C 언어 & Make 유틸리티 입문 실습 교안

## 1. WSL (Linux용 Windows 하위 시스템) 설치

윈도우 왼쪽 아래 "찾기"에서 **Windows PowerShell**을 **관리자 권한**으로 실행한 후 입력합니다.

```powershell
wsl --install -d Ubuntu
```

PC를 재부팅하면 자동으로 터미널 창이 열립니다. 이 창에서 사용할 **일반 사용자 아이디(Username)** 와 **비밀번호(Password)** 를 입력합니다.

---

## 2. 루트(root) 계정 비밀번호 설정

시스템 최고 관리자(root) 권한을 이용하기 위해 비밀번호를 설정합니다.

```bash
sudo passwd root
```

1. 현재 일반 사용자 비밀번호를 1번 입력합니다.
2. 사용할 root 새 비밀번호를 2번 입력합니다.

> 비밀번호를 입력할 때 화면에 문자가 보이지 않아도 정상입니다.

---

## 3. 리눅스 패키지 목록 업데이트 & 업그레이드

최신 패키지 정보를 불러오고 시스템을 업데이트합니다.

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 4. C언어 개발 도구 (GCC, Make, Nano) 설치

컴파일러(gcc), 자동 빌드 도구(make), 텍스트 에디터(nano)를 설치합니다.

```bash
sudo apt install -y build-essential nano
```

설치 확인:

```bash
gcc --version
make --version
```

---

## 5. 단일 파일 작성 및 컴파일 테스트 (test.c)

### Step 1. 실습 폴더 생성 및 이동

```bash
mkdir myproj
cd myproj
```

### Step 2. test.c 파일 작성 (nano 에디터 사용)

```bash
nano test.c
```

파일 내용:

```c
#include <stdio.h>

int main() {
    printf("Hello WSL C World!\n");
    return 0;
}
```

nano 저장 및 종료 방법:

| 동작 | 단축키 |
|---|---|
| 저장 | `Ctrl + O` → `Enter` |
| 종료 | `Ctrl + X` |

### Step 3. 컴파일 및 실행

```bash
gcc -o test test.c
./test
```

출력 결과:

```
Hello WSL C World!
```

---

## 6. Make 유틸리티 활용 (test.c, add.c, sub.c 분할 빌드)

3개의 소스 파일로 분할 작성하고 Makefile로 한 번에 빌드합니다.

### Step 1. 소스 파일 작성

**(1) add.c** 작성 (`nano add.c`)

```c
int add(int a, int b) {
    return a + b;
}
```

**(2) sub.c** 작성 (`nano sub.c`)

```c
int sub(int a, int b) {
    return a - b;
}
```

**(3) test.c** 수정 (`nano test.c`)

```c
#include <stdio.h>

int add(int a, int b);
int sub(int a, int b);

int main() {
    printf("3 + 2 = %d\n", add(3, 2));
    printf("3 - 2 = %d\n", sub(3, 2));
    return 0;
}
```

### Step 2. Makefile 생성

`nano Makefile` (파일명의 **대문자 M**에 주의)

```makefile
test: test.c add.c sub.c
	gcc -o test test.c add.c sub.c

clean:
	rm -f test
```

> **주의**: `gcc`와 `rm` 앞의 들여쓰기는 반드시 **Tab 키**를 사용해야 합니다. 스페이스바를 사용하면 오류가 발생합니다.

### Step 3. Make 실행 및 정리

**(1) 자동 빌드**

```bash
make
```

**(2) 프로그램 실행**

```bash
./test
```

출력 결과:

```
3 + 2 = 5
3 - 2 = 1
```

**(3) 빌드 파일 정리**

```bash
make clean
```
