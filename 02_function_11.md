## 1. 함수의 선언, 정의, 호출

```c
#include <stdio.h>

// 함수 선언 (프로토타입)
int add(int a, int b);

// main 함수
int main() {
    int result = add(3, 4); // 함수 호출
    printf("결과: %d\n", result);
    return 0;
}

// 함수 정의
int add(int a, int b) {
    return a + b;
}
```

---

## 2. 매개변수 전달 방식

### (1) 값 전달

```c
#include <stdio.h>

void square(int x) {  
    x = x * x;  
    printf("square 함수 안: %d\n", x);  
}

int main() {
    int a = 5;
    square(a);
    printf("main 함수 안: %d\n", a);  // 원래 값은 변하지 않음
    return 0;
}
```

### (2) 포인터(참조) 전달

```c
#include <stdio.h>

void square(int *x) {  
    *x = (*x) * (*x);  
}

int main() {
    int a = 5;
    square(&a);
    printf("결과: %d\n", a);  // 값이 바뀜
    return 0;
}
```

---

## 3. 함수 반환값과 다중 반환 구현

### (1) 단일 반환값

```c
#include <stdio.h>

int getDouble(int x) {
    return x * 2;
}

int main() {
    int result = getDouble(7);
    printf("결과: %d\n", result);
    return 0;
}
```

### (2) 포인터를 이용한 다중 반환

```c
#include <stdio.h>

void calc(int a, int b, int *sum, int *prod) {
    *sum = a + b;
    *prod = a * b;
}

int main() {
    int s, p;
    calc(3, 4, &s, &p);
    printf("합: %d, 곱: %d\n", s, p);
    return 0;
}
```

### (3) 구조체를 이용한 다중 반환

```c
#include <stdio.h>

typedef struct {
    int sum;
    int prod;
} Result;

Result calc(int a, int b) {
    Result r;
    r.sum = a + b;
    r.prod = a * b;
    return r;
}

int main() {
    Result r = calc(3, 4);
    printf("합: %d, 곱: %d\n", r.sum, r.prod);
    return 0;
}
```
### 4) 지역 변수와 전역 변수의 차이 및 사용 범위


```c
// 4) 지역 변수와 전역 변수의 차이 및 사용 범위
#include <stdio.h>
int g = 10;
void f(void) {
    int l = 5;
    printf("l=%d g=%d\n", l, g);
}
int main(void) {
    f();
    return 0;
}
```

### 5) 정적 변수(static)와 함수의 수명 주기
```c
// 5) 정적 변수(static)와 함수의 수명 주기
#include <stdio.h>
void count(void) {
    static int s = 0;
    s = s + 1;
    printf("%d\n", s);
}
int main(void) {
    count();
    count();
    count();
    return 0;
}
```

### 6) 재귀 함수(Recursive Function)와 종료 조건

```c
// 6) 재귀 함수와 종료 조건
#include <stdio.h>
int fact(int n) {
    if (n <= 1) {
        return 1;
    }
    return n * fact(n - 1);
}
int main(void) {
    printf("%d\n", fact(5));
    return 0;
}
```
