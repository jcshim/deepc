# library 만들기

## Visual Studio 를 실행 > 도구 > 명령줄 > 개발자 명령 프람프트
---

### 📄 add.c

```c
// add.c
int add(int a, int b) {
    return a + b;
}
```

---

### 📄 sub.c

```c
// sub.c
int sub(int a, int b) {
    return a - b;
}
```

---

### 📄 main.c

```c
// main.c
#include <stdio.h>

// 함수 원형 선언
int add(int a, int b);
int sub(int a, int b);

int main(void) {
    int x = 10;
    int y = 4;

    printf("add(%d, %d) = %d\n", x, y, add(x, y));
    printf("sub(%d, %d) = %d\n", x, y, sub(x, y));

    return 0;
}
```

---

이렇게 세 파일(`main.c`, `add.c`, `sub.c`)을 준비하고, 앞서 알려드린 명령어:

```bat
cl /c add.c sub.c
lib /out:my.lib add.obj sub.obj
cl main.c my.lib
```

을 Visual Studio 개발자 프롬프트에서 실행하면 **main.exe**가 생성되고, 실행하면 다음과 같은 출력이 나옵니다:

```
add(10, 4) = 14
sub(10, 4) = 6
```

---

👉 원하시면 제가 **빌드 및 실행 전체 과정**을 `build.bat` 같은 배치 파일로 만들어드릴 수도 있습니다. 해드릴까요?
