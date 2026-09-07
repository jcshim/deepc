## 1️⃣ 개발 환경 준비

1. **필수 패키지 설치**
   WSL(Ubuntu 기준)에서 실행:

   ```bash
   sudo apt update
   sudo apt install build-essential
   ```

   → `gcc`(컴파일러), `g++`, `make` 등이 설치됩니다.

2. **에디터 준비**

   * 간단히: `nano`, `vim` (WSL 기본 제공)
   * GUI IDE 연동: **VS Code** + *Remote - WSL 확장* → 학생들이 가장 많이 씁니다.
   * WSL에서 `code .` 입력하면 VS Code에서 바로 파일 열림.

---

## 2️⃣ 기본 실습 흐름

### (1) C 프로그램 작성

```c
// hello.c
#include <stdio.h>

int main() {
    printf("Hello WSL!\n");
    return 0;
}
```

### (2) 컴파일

```bash
gcc hello.c -o hello
```

### (3) 실행

```bash
./hello
```

---

## 3️⃣ 학습 방법 제안

### 🔹 기초

* 표준입출력(`printf`, `scanf`)
* 조건문, 반복문
* 함수 작성 및 호출
* 배열, 포인터, 문자열

### 🔹 심화

* 파일 입출력 (`fopen`, `fscanf`, `fprintf`)
* 구조체와 메모리 관리 (`malloc`, `free`)
* 간단한 알고리즘 실습 (정렬, 탐색)
* Linux API 활용 (`fork`, `exec`, `pipe`)

---

## 4️⃣ 활용 팁

* **Makefile 작성**
  여러 소스파일을 컴파일하는 법 학습:

  ```makefile
  all: main

  main: main.o util.o
      gcc -o main main.o util.o

  clean:
      rm -f *.o main
  ```

* **디버깅**
  `gdb hello` 로 디버깅 연습 가능.
  학생들에게 **breakpoint / step / print 변수** 같은 기초 디버깅 방법 알려주면 효과적입니다.

* **과제 예시**

  * 숫자 맞추기 게임
  * 텍스트 파일 단어 빈도 세기
  * 간단한 은행 계좌 관리 프로그램 (구조체, 파일 입출력 활용)

---

✅ 정리하면, WSL에서는 **리눅스 개발 환경 그대로 C언어 학습 가능**하고,
**gcc + VS Code + gdb** 조합을 활용하면 학생들이 윈도우에서도 리눅스 스타일로 C 프로그래밍을 쉽게 배우게 할 수 있습니다.
