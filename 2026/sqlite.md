학생들에게 나누어줄 수 있는 가장 단순하고 직관적인 SQLite3 연동 실습 자료입니다.

---

### **1. 실습 소스코드 (`main.c`)**

SQLite3 DB를 생성 및 연결하고, 버전을 출력하는 최소 코드입니다.

```c
#include <stdio.h>
#include "sqlite3.h"

void main() {
    sqlite3 *db;
    if (sqlite3_open("test.db", &db) == SQLITE_OK)
        printf("SQLite 연결 성공! 버전: %s\n", sqlite3_libversion());
    sqlite3_close(db);
}

```

---

### **2. 가장 쉬운 실습 명령어 (배포용 txt)**

```text
====================================================================
  [실습] MSVC에서 C언어와 SQLite3 연동하기
====================================================================

1. 준비물 다운로드 (sqlite.org -> Download)
--------------------------------------------------------------------
  - sqlite-amalgamation-*.zip (sqlite3.c, sqlite3.h 추출)
  - sqlite-dll-win-*.zip          (sqlite3.def, sqlite3.dll 추출)
  * 위 4개 파일을 main.c와 같은 폴더에 넣습니다.


2. MSVC 명령어 실습 (Developer Command Prompt 실행)
--------------------------------------------------------------------

[1단계] SQLite3 DLL용 정적 라이브러리(.lib) 생성
> lib /def:sqlite3.def /out:sqlite3.lib
  *(결과: sqlite3.lib 생성)*


[2단계] main.c 컴파일 및 sqlite3.lib 링크
> cl main.c sqlite3.lib
  *(결과: main.exe 생성)*


[3단계] 실행 및 결과 확인
> main.exe

  * 출력 결과:
    SQLite 연결 성공! 버전: 3.xx.x
====================================================================

```
