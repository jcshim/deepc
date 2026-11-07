raylib는 복잡한 객체지향 구조나 의존성 없이 **C언어의 절차적 특징을 극대화하여 직관적이고 가볍게 설계된 2D/3D 그래픽 라이브러리**입니다.

---

## 1. 핵심 아키텍처 (모듈형 구조)

raylib는 내부적으로 완벽하게 분리된 **모듈형 C 라이브러리**로 구성되어 있습니다. 각 모듈은 독립된 기능 영역을 담당합니다.

| 모듈명 | 담당 기능 | 대표 함수/역할 |
| --- | --- | --- |
| **`rcore`** | 창 생성, 입력(키보드/마우스/게임패드) 및 시간 제어 | `InitWindow()`, `IsKeyPressed()`, `SetTargetFPS()` |
| **`rshapes`** | 기본 2D 도형 렌더링 | `DrawRectangle()`, `DrawCircle()`, `DrawLine()` |
| **`rtextures`** | 이미지 로딩 및 2D 텍스처 렌더링 | `LoadTexture()`, `DrawTexture()` |
| **`rtext`** | 폰트 로딩 및 텍스트 렌더링 | `LoadFont()`, `DrawText()` |
| **`rmodels`** | 3D 모델, 메시, 조명 및 카메라 제어 | `DrawCube()`, `DrawModel()`, `BeginMode3D()` |
| **`raudio`** | 사운드 이펙트 및 음악 스트리밍 | `InitAudioDevice()`, `LoadSound()`, `PlayMusicStream()` |
| **`rlgl`** | OpenGL 추상화 레이어 (하위 단 제어) | OpenGL 명령을 내부적으로 C 스타일로 래핑 |

---

## 2. raylib의 기본 동작 구조 (게임 루프)

raylib의 모든 프로그램은 [초기화 ➔ 게임 루프(입력/업데이트/그리기) ➔ 해제]라는 명확한 3단계 순서 구조를 가집니다.

```c
#include "raylib.h"

int main(void) {
    // 1. [초기화] 창 및 리소스 생성
    InitWindow(800, 450, "raylib structure");
    SetTargetFPS(60);

    // 2. [게임 루프] WindowShouldClose()가 true가 될 때까지 반복
    while (!WindowShouldClose()) {
        // (1) 입력 처리 & 데이터 업데이트 (Update)
        // ...

        // (2) 화면 그리기 (Render)
        BeginDrawing();
            ClearBackground(RAYWHITE);
            // 2D/3D 도형 및 텍스처 그리기
        EndDrawing();
    }

    // 3. [해제] 자원 정리 및 종료
    CloseWindow();
    return 0;
}

```

---

## 3. 구조적 특징 및 장점

1. **단순한 데이터 구조체 (`struct`) 중심**
* Class나 객체 대신 `Vector2`, `Vector3`, `Rectangle`, `Color`, `Camera` 같은 가벼운 C 구조체를 기본 인자로 넘겨 다룹니다.


2. **C/C++ 표준에 의존하지 않는 객체 (No External Dependencies)**
* GLFW, Glad 등을 내부에서 스스로 다루기 때문에 설치와 빌드가 단일 라이브러리로 끝납니다.


3. **2D와 3D 모드의 자연스러운 전환**
* `BeginMode3D(camera);`와 `EndMode3D();` 블록 내부에서 3D 렌더링을 진행하고, 밖에서는 2D UI를 겹쳐 그릴 수 있는 단순한 샌드위치 구조입니다.
