# C 언어 함수 관련 예제 모음

## 1) 함수 분할과 모듈화 (헤더 파일과 소스 파일 분리)

**myadd.h**

```c
#ifndef MYADD_H
#define MYADD_H
int add(int a, int b);
#endif
```

**myadd.c**

```c
#include "myadd.h"
int add(int a, int b) {
    return a + b;
}
```

**main.c**

```c
#include <stdio.h>
#include "myadd.h"
int main(void) {
    int r = add(2, 3);
    printf("%d\n", r);
    return 0;
}
```

---

## 2) 함수 포인터(Function Pointer) 기본과 활용

```c
#include <stdio.h>

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }

int main(void) {
    int (*op)(int, int) = add;
    printf("%d\n", op(5, 2));
    op = sub;
    printf("%d\n", op(5, 2));
    return 0;
}
```

---

## 3) 콜백 함수(Callback) 구현 실습

```c
#include <stdio.h>

void for_each(int *arr, int n, void (*cb)(int)) {
    for (int i = 0; i < n; i++) cb(arr[i]);
}

void print_int(int x) { printf("%d ", x); }

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};
    for_each(a, 5, print_int);
    printf("\n");
    return 0;
}
```

---

## 4) 가변 인자 함수 (stdarg.h 활용)

```c
#include <stdio.h>
#include <stdarg.h>

int sum(int count, ...) {
    va_list ap;
    va_start(ap, count);
    int s = 0;
    for (int i = 0; i < count; i++) s += va_arg(ap, int);
    va_end(ap);
    return s;
}

int main(void) {
    printf("%d\n", sum(4, 1, 2, 3, 4));
    return 0;
}
```

---

## 5) inline 함수와 매크로 함수 비교

```c
#include <stdio.h>

#define SQ_MACRO(x) ((x) * (x))
static inline int sq_inline(int x) { return x * x; }

int main(void) {
    int a = 5;
    printf("%d\n", SQ_MACRO(a + 1));
    printf("%d\n", sq_inline(a + 1));
    return 0;
}
```

---

## 6) 라이브러리 함수 활용 및 사용자 정의 라이브러리 제작

**mymath.h**

```c
#ifndef MYMATH_H
#define MYMATH_H
int cube(int x);
#endif
```

**mymath.c**

```c
#include "mymath.h"
int cube(int x) {
    return x * x * x;
}
```

**main.c**

```c
#include <stdio.h>
#include <math.h>      /* 표준 라이브러리 예: sqrt */
#include "mymath.h"    /* 사용자 정의 라이브러리 */

int main(void) {
    double r = sqrt(16.0);
    int c = cube(3);
    printf("%.0f %d\n", r, c);
    return 0;
}
```
