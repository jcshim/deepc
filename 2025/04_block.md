// breakout_fixed.c - raylib Rainbow Breakout (semi-transparent blocks, compat-safe)
// Build (MinGW):  gcc breakout_fixed.c -o breakout -lraylib -lopengl32 -lgdi32 -lwinmm
// Build (Linux):  gcc breakout_fixed.c -o breakout -lraylib -lm -lpthread -ldl -lrt -lX11
// MSVC/VS: 프로젝트에 raylib 추가 후 빌드. 콘솔 숨기려면 /SUBSYSTEM:WINDOWS + /ENTRY:mainCRTStartup

#include "raylib.h"
#include <math.h>
#include <stdio.h>
#include <stdbool.h>

#define SCREEN_W  900
#define SCREEN_H  600

#define ROWS   7
#define COLS   12
#define BLOCK_W 60.0f
#define BLOCK_H 24.0f
#define BLOCK_GAP 6.0f
#define TOP_MARGIN 80.0f

#define PADDLE_W 110.0f
#define PADDLE_H 14.0f
#define PADDLE_SPEED 580.0f

#define BALL_RADIUS 8.0f
#define BALL_SPEED  320.0f

typedef struct {
    Rectangle rect;
    bool alive;
    Color color;
} Block;

static Block blocks[ROWS][COLS];

static inline float flerp(float a, float b, float t) { return a + (b - a) * t; }

// 간단 HSV→RGB (h:0~360, s,v:0~1) → Color (a는 255)
static Color HsvToColor(float h, float s, float v)
{
    while (h < 0.0f) h += 360.0f;
    while (h >= 360.0f) h -= 360.0f;

    float c = v * s;
    float x = c * (1.0f - fabsf(fmodf(h / 60.0f, 2.0f) - 1.0f));
    float m = v - c;

    float r = 0.0f, g = 0.0f, b = 0.0f;
    if (h < 60.0f) { r = c; g = x; b = 0.0f; }
    else if (h < 120.0f) { r = x; g = c; b = 0.0f; }
    else if (h < 180.0f) { r = 0.0f; g = c; b = x; }
    else if (h < 240.0f) { r = 0.0f; g = x; b = c; }
    else if (h < 300.0f) { r = x; g = 0.0f; b = c; }
    else { r = c; g = 0.0f; b = x; }

    unsigned char R = (unsigned char)((r + m) * 255.0f + 0.5f);
    unsigned char G = (unsigned char)((g + m) * 255.0f + 0.5f);
    unsigned char B = (unsigned char)((b + m) * 255.0f + 0.5f);
    return (Color) { R, G, B, 255 };
}

static void InitBlocks(void) {
    const float gridW = (float)COLS * (BLOCK_W + BLOCK_GAP) - BLOCK_GAP;
    const float startX = (SCREEN_W - gridW) * 0.5f;
    for (int r = 0; r < ROWS; ++r) {
        for (int c = 0; c < COLS; ++c) {
            const float x = startX + (float)c * (BLOCK_W + BLOCK_GAP);
            const float y = TOP_MARGIN + (float)r * (BLOCK_H + BLOCK_GAP);
            blocks[r][c].rect = (Rectangle){ x, y, BLOCK_W, BLOCK_H };
            blocks[r][c].alive = true;

            // 무지개색: 열과 행으로 hue 변조
            float hue = fmodf(((float)c / (float)COLS) * 360.0f + (float)r * (360.0f / (float)ROWS) * 0.35f, 360.0f);
            blocks[r][c].color = HsvToColor(hue, 0.75f, 0.95f);
        }
    }
}

static void ResetGame(Vector2* paddle, Vector2* ball, Vector2* ballVel, int* lives, int* score) {
    *paddle = (Vector2){ SCREEN_W * 0.5f - PADDLE_W * 0.5f, SCREEN_H - 60.0f };
    *ball = (Vector2){ SCREEN_W * 0.5f, SCREEN_H - 80.0f };
    *ballVel = (Vector2){ BALL_SPEED * 0.7071f, -BALL_SPEED * 0.7071f };
    *lives = 3;
    *score = 0;
    InitBlocks();
}

static bool CircleRectCollision(Vector2 center, float radius, Rectangle rect, Vector2* pushNormal) {
    float cx = (center.x < rect.x) ? rect.x : (center.x > rect.x + rect.width ? rect.x + rect.width : center.x);
    float cy = (center.y < rect.y) ? rect.y : (center.y > rect.y + rect.height ? rect.y + rect.height : center.y);
    float dx = center.x - cx;
    float dy = center.y - cy;
    float dist2 = dx * dx + dy * dy;
    if (dist2 <= radius * radius) {
        float overlapLeft = (center.x + radius) - rect.x;
        float overlapRight = (rect.x + rect.width) - (center.x - radius);
        float overlapTop = (center.y + radius) - rect.y;
        float overlapBottom = (rect.y + rect.height) - (center.y - radius);
        float minOverlap = overlapLeft; *pushNormal = (Vector2){ -1.0f, 0.0f };
        if (overlapRight < minOverlap) { minOverlap = overlapRight;  *pushNormal = (Vector2){ 1.0f, 0.0f }; }
        if (overlapTop < minOverlap) { minOverlap = overlapTop;    *pushNormal = (Vector2){ 0.0f,-1.0f }; }
        if (overlapBottom < minOverlap) { /*minOverlap = overlapBottom;*/ *pushNormal = (Vector2){ 0.0f, 1.0f }; }
        return true;
    }
    return false;
}

int main(void) {
    SetConfigFlags(FLAG_VSYNC_HINT);
    InitWindow(SCREEN_W, SCREEN_H, "Rainbow Breakout (raylib - compat)");

    Vector2 paddle, ball, ballVel;
    int lives = 3, score = 0, totalBlocks = ROWS * COLS, broken = 0;
    bool paused = false, gameOver = false, gameClear = false;

    ResetGame(&paddle, &ball, &ballVel, &lives, &score);
    broken = 0;

    SetTargetFPS(120);

    while (!WindowShouldClose()) {
        float dt = GetFrameTime();

        if (IsKeyPressed(KEY_P)) paused = !paused;
        if ((gameOver || gameClear) &&
            (IsKeyPressed(KEY_ENTER) || IsMouseButtonPressed(MOUSE_LEFT_BUTTON))) {
            ResetGame(&paddle, &ball, &ballVel, &lives, &score);
            broken = 0;
            gameOver = gameClear = false;
        }

        if (!paused && !gameOver && !gameClear) {
            // 패들 이동
            float move = 0.0f;
            if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A))  move -= 1.0f;
            if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) move += 1.0f;
            paddle.x += move * PADDLE_SPEED * dt;

            if (IsMouseButtonDown(MOUSE_BUTTON_LEFT) || IsMouseButtonDown(MOUSE_BUTTON_RIGHT)) {
                float target = (float)GetMouseX() - PADDLE_W * 0.5f;
                paddle.x = flerp(paddle.x, target, 12.0f * dt);
            }
            if (paddle.x < 0.0f) paddle.x = 0.0f;
            if (paddle.x + PADDLE_W > (float)SCREEN_W) paddle.x = (float)SCREEN_W - PADDLE_W;

            // 공 이동
            ball.x += ballVel.x * dt;
            ball.y += ballVel.y * dt;

            // 벽 충돌
            if (ball.x - BALL_RADIUS <= 0.0f) { ball.x = BALL_RADIUS;                          ballVel.x *= -1.0f; }
            if (ball.x + BALL_RADIUS >= (float)SCREEN_W) { ball.x = (float)SCREEN_W - BALL_RADIUS;       ballVel.x *= -1.0f; }
            if (ball.y - BALL_RADIUS <= 0.0f) { ball.y = BALL_RADIUS;                          ballVel.y *= -1.0f; }

            // 패들 충돌
            Rectangle paddleRect = (Rectangle){ paddle.x, paddle.y, PADDLE_W, PADDLE_H };
            Vector2 nrm;
            if (CircleRectCollision(ball, BALL_RADIUS, paddleRect, &nrm) && ballVel.y > 0.0f) {
                float hit = (ball.x - (paddle.x + PADDLE_W * 0.5f)) / (PADDLE_W * 0.5f);
                if (hit < -1.0f) hit = -1.0f; if (hit > 1.0f) hit = 1.0f;
                ballVel.x = BALL_SPEED * hit;
                ballVel.y = -fabsf(BALL_SPEED * (0.75f + 0.25f * fabsf(hit)));
                ball.y = paddle.y - BALL_RADIUS - 1.0f;
            }

            // 블록 충돌
            for (int r = 0; r < ROWS; ++r) {
                for (int c = 0; c < COLS; ++c) {
                    if (!blocks[r][c].alive) continue;
                    Vector2 nn;
                    if (CircleRectCollision(ball, BALL_RADIUS, blocks[r][c].rect, &nn)) {
                        blocks[r][c].alive = false;
                        broken++;
                        score += 10;
                        if (fabsf(nn.x) > 0.5f) ballVel.x *= -1.0f;
                        if (fabsf(nn.y) > 0.5f) ballVel.y *= -1.0f;
                        ball.x += nn.x * 2.0f;
                        ball.y += nn.y * 2.0f;
                        goto after_block;
                    }
                }
            }
        after_block:;

            // 바닥
            if (ball.y - BALL_RADIUS > (float)SCREEN_H) {
                lives--;
                if (lives <= 0) {
                    gameOver = true;
                }
                else {
                    ball = (Vector2){ paddle.x + PADDLE_W * 0.5f, paddle.y - 20.0f };
                    ballVel = (Vector2){ BALL_SPEED * 0.65f, -BALL_SPEED * 0.75f };
                }
            }

            // 클리어
            if (broken >= totalBlocks) gameClear = true;
        }

        // ===== 그리기 =====
        BeginDrawing();
        ClearBackground((Color) { 18, 18, 24, 255 });

        DrawText("R A I N B O W   B R E A K O U T", 20, 14, 28, RAYWHITE);
        DrawText(TextFormat("Score: %d", score), 20, 52, 20, LIGHTGRAY);
        DrawText(TextFormat("Lives: %d", lives), 180, 52, 20, LIGHTGRAY);
        if (paused) DrawText("PAUSED (Press P)", SCREEN_W / 2 - 120, 52, 20, YELLOW);

        // 블록 (반투명)
        for (int r = 0; r < ROWS; ++r) {
            for (int c = 0; c < COLS; ++c) {
                if (!blocks[r][c].alive) continue;
                Color fill = Fade(blocks[r][c].color, 0.65f);
                DrawRectangleRec(blocks[r][c].rect, fill);
                DrawRectangleLinesEx(blocks[r][c].rect, 1.5f, ColorAlpha(BLACK, 0.25f));
            }
        }

        // 패들(라운드: 채우기 + 4인자 라인)
        Rectangle pr = (Rectangle){ paddle.x, paddle.y, PADDLE_W, PADDLE_H };
        DrawRectangleRounded(pr, 0.4f, 8, (Color) { 230, 230, 235, 255 });
        DrawRectangleRoundedLines(pr, 0.4f, 8, ColorAlpha(BLACK, 0.35f));  // 4 args (호환)

        // 공
        DrawCircleV(ball, BALL_RADIUS, (Color) { 240, 240, 255, 255 });
        DrawCircleLines((int)ball.x, (int)ball.y, BALL_RADIUS, ColorAlpha(BLACK, 0.35f));

        DrawText("Move: A/D or \x1b[<-][->]   Mouse drag: hold button   Pause: P   Restart: Enter",
            20, SCREEN_H - 28, 18, GRAY);

        if (gameOver) {
            DrawRectangle(0, 0, SCREEN_W, SCREEN_H, ColorAlpha(BLACK, 0.6f));
            DrawText("GAME OVER", SCREEN_W / 2 - 160, SCREEN_H / 2 - 40, 48, RED);
            DrawText("Press [ENTER] to restart", SCREEN_W / 2 - 180, SCREEN_H / 2 + 20, 22, RAYWHITE);
        }
        if (gameClear) {
            DrawRectangle(0, 0, SCREEN_W, SCREEN_H, ColorAlpha(BLACK, 0.6f));
            DrawText("YOU WIN!", SCREEN_W / 2 - 120, SCREEN_H / 2 - 40, 48, GREEN);
            DrawText("Press [ENTER] to play again", SCREEN_W / 2 - 190, SCREEN_H / 2 + 20, 22, RAYWHITE);
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
