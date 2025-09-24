## Raylib 활용한 프로젝트 7선

1. **픽셀 그리기 패드 (Paint Lite)**

* 목표: 마우스로 화면에 점/선/사각형 그리기
* 개념: 마우스 입력, 상태 저장(벡터/배열), 화면 지우기/초기화
* 최소: 커서 위치에 점 찍기, 색상 2\~3개 전환, 지우기
* 확장: 선 굵기 조절, 도형 선택(사각형/원), PNG 저장(스크린샷)

2. **Pong 1인용 (벽 튕기기)**

* 목표: 키보드로 패들 움직여 공 튕기기
* 개념: 키보드 입력, 충돌 판정(벽·패들), 벡터 속도
* 최소: 공 1개, 패들 1개, 점수(반환 횟수) 표시
* 확장: 난이도(공 속도 가속), 생명 3개, 아이템

3. **낙하물 피하기 (Dodge)**

* 목표: 좌우로 캐릭터를 움직이며 위에서 떨어지는 블록 피하기
* 개념: 스폰/소멸, AABB 충돌, 게임 루프·상태(대기/플레이/게임오버)
* 최소: 블록 3\~5개 랜덤 낙하, 충돌 시 게임오버, 생존 시간 표시
* 확장: 점수/하이스코어, 속도 점차 증가, 무적 아이템

4. **타자 연습 (Typing Tutor Mini)**

* 목표: 화면에 나오는 단어를 제한시간 내 입력
* 개념: 텍스트 렌더링, 입력 버퍼 관리, 타이머
* 최소: 단어 1개씩 순차 제시, 정답·오답 처리, 남은 시간
* 확장: 난이도(단어 길이), 콤보/정확도, 단어 사전 파일 로드

5. **벽돌깨기 초간단 (Breakout Lite)**

* 목표: 공로 벽돌을 제거
* 개념: 타일 그리드, 충돌 분기(상/하/좌/우), 반사
* 최소: 3×5 벽돌, 클리어/실패 상태
* 확장: 여러 스테이지, 아이템(패들 길이↑), 이펙트

6. **그리드 미로 탐험 (Maze Walk)**

* 목표: 화살표 키로 미로 탈출
* 개념: 2D 배열, 타일 충돌(벽/길), 간단한 BFS 생성(선택)
* 최소: 고정된 미로, 시작/도착, 걸음 수 표시
* 확장: 랜덤 미로 생성, 미니맵, 타이머/랭크

7. **두더지 잡기 (Whack-a-Mole)**

* 목표: 랜덤으로 나타나는 두더지를 제한시간 내 클릭
* 개념: 랜덤 스폰, 히트 판정(원/사각형), 점수/쿨다운
* 최소: 3×3 홀, 두더지 1마리씩 팝업, 남은 시간·점수
* 확장: 멀티 팝업, 연속 히트 콤보, 미스 패널티

---

## Windows(MSVC)용 초간단 템플릿

```c
// main.c - raylib 최소 템플릿
#include "raylib.h"

int main(void) {
    const int W = 800, H = 450;
    InitWindow(W, H, "raylib template");
    SetTargetFPS(60);

    while (!WindowShouldClose()) {
        // 업데이트(Update)

        // 렌더(Draw)
        BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawText("Hello, raylib!", 24, 24, 28, DARKGRAY);
        EndDrawing();
    }
    CloseWindow();
    return 0;
}
```

> 빌드 팁(요약): raylib 바이너리/정적 라이브러리를 받아 프로젝트에 링크하거나, vcpkg로 `raylib` 설치 후 프로젝트에 추가하세요. (MSVC: 프로젝트 속성 → 링커에 `raylib.lib` 추가, `Additional Include/Library Directories` 설정)

---

## 샘플 완성 예제: **낙하물 피하기 (Dodge)**

* 방향키 ← → 로 플레이어(사각형)를 이동
* 위에서 블록이 떨어짐, 닿으면 게임오버
* 생존 시간(초) 표시, R키로 재시작

```c
// dodge.c - 80줄대 간단 예제
#include "raylib.h"
#include <stdlib.h>

typedef struct {
    Rectangle rect;
    float vy;
    bool active;
} Block;

#define MAX_BLOCKS 8

int main(void) {
    const int W = 800, H = 450;
    InitWindow(W, H, "Dodge - raylib mini");
    SetTargetFPS(60);

    Rectangle player = { W/2.0f - 20, H - 60, 40, 40 };
    float speed = 300.0f;

    Block blocks[MAX_BLOCKS] = {0};
    float spawnTimer = 0.0f;
    float spawnInterval = 0.7f;

    bool gameOver = false;
    float surviveTime = 0.0f;
    float bestTime = 0.0f;

    while (!WindowShouldClose()) {
        float dt = GetFrameTime();

        // 재시작
        if (gameOver && IsKeyPressed(KEY_R)) {
            gameOver = false;
            surviveTime = 0.0f;
            for (int i=0;i<MAX_BLOCKS;i++) blocks[i].active = false;
        }

        if (!gameOver) {
            // 플레이어 이동
            if (IsKeyDown(KEY_LEFT))  player.x -= speed * dt;
            if (IsKeyDown(KEY_RIGHT)) player.x += speed * dt;
            if (player.x < 0) player.x = 0;
            if (player.x + player.width > W) player.x = W - player.width;

            // 블록 스폰
            spawnTimer += dt;
            if (spawnTimer >= spawnInterval) {
                spawnTimer = 0.0f;
                for (int i=0;i<MAX_BLOCKS;i++) {
                    if (!blocks[i].active) {
                        blocks[i].active = true;
                        blocks[i].rect.width = 30 + rand()%50;
                        blocks[i].rect.height = 20;
                        blocks[i].rect.x = (float)(rand() % (W - (int)blocks[i].rect.width));
                        blocks[i].rect.y = -blocks[i].rect.height;
                        blocks[i].vy = 140.0f + (float)(rand()%120);
                        break;
                    }
                }
            }

            // 블록 이동 & 충돌
            for (int i=0;i<MAX_BLOCKS;i++) if (blocks[i].active) {
                blocks[i].rect.y += blocks[i].vy * dt;
                if (blocks[i].rect.y > H) blocks[i].active = false;
                if (CheckCollisionRecs(player, blocks[i].rect)) {
                    gameOver = true;
                    if (surviveTime > bestTime) bestTime = surviveTime;
                }
            }

            // 난이도 점증
            surviveTime += dt;
            if (spawnInterval > 0.3f) spawnInterval -= 0.02f * dt;
        }

        // 그리기
        BeginDrawing();
            ClearBackground(RAYWHITE);

            // 플레이어 & 블록
            DrawRectangleRec(player, gameOver ? GRAY : DARKGREEN);
            for (int i=0;i<MAX_BLOCKS;i++) if (blocks[i].active)
                DrawRectangleRec(blocks[i].rect, RED);

            // UI
            DrawText(TextFormat("Time: %.2fs", surviveTime), 20, 20, 24, DARKGRAY);
            DrawText(TextFormat("Best: %.2fs", bestTime),   20, 50, 24, DARKGRAY);
            if (gameOver) {
                DrawText("GAME OVER - Press R to Restart", 160, 200, 24, MAROON);
            }
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```
