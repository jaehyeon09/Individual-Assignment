## 스와이프 벽돌깨기는 플레이어가 마우스를 이용해 공의 발사 방향을 조절하고, 공을 발사하여 화면 위쪽에 배치된 벽돌을 제거하는 게임이다. 공이 벽돌이나 화면의 벽에 부딪히면 튕겨 나가며, 벽돌을 파괴할 때마다 점수가 증가한다. 공이 다시 아래쪽으로 돌아오면 다음 단계가 진행되고, 벽돌이 화면 아래쪽까지 내려오면 게임이 종료된다.
## 게임 진행 방식
## 게임이 시작되면 화면에 벽돌이 배치되고, 플레이어는 마우스로 공의 발사 방향을 조절한다. 공을 발사하면 벽돌과 충돌하면서 벽돌의 내구도가 감소하고, 벽돌을 제거하면 점수를 획득한다.
## 공이 화면 아래쪽으로 돌아오면 다음 단계로 넘어가며 새로운 벽돌 줄이 추가된다. 이러한 과정을 반복하면서 벽돌이 화면 아래쪽의 제한선에 도달하지 않도록 최대한 많은 벽돌을 제거하는 것이 게임의 목표이다.
```
#include "raylib.h"
#include <math.h>

#define W 450
#define H 800
#define ROWS 11
#define COLS 7

float ballX = 225.0f;
float ballY = 730.0f;

float speedX = 0.0f;
float speedY = 0.0f;

float aimX = 225.0f;
float aimY = 600.0f;

int bricks[ROWS][COLS];

int score = 0;
int level = 1;
int moving = 0;
int aiming = 0;
int gameOver = 0;

float BrickX(int c)
{
    return 15.0f + c * 60.0f;
}

float BrickY(int r)
{
    return 110.0f + r * 55.0f;
}

void ResetGame(void)
{
    int r, c;

    ballX = 225.0f;
    ballY = 730.0f;

    speedX = 0.0f;
    speedY = 0.0f;

    score = 0;
    level = 1;

    moving = 0;
    aiming = 0;
    gameOver = 0;

    for (r = 0; r < ROWS; r++)
    {
        for (c = 0; c < COLS; c++)
        {
            bricks[r][c] = 0;
        }
    }

    for (c = 0; c < COLS; c++)
    {
        bricks[0][c] = GetRandomValue(1, 3);
    }
}

void NewRow(void)
{
    int r, c;

    for (r = ROWS - 1; r > 0; r--)
    {
        for (c = 0; c < COLS; c++)
        {
            bricks[r][c] = bricks[r - 1][c];
        }
    }

    for (c = 0; c < COLS; c++)
    {
        bricks[0][c] = 0;

        if (GetRandomValue(0, 100) < 75)
        {
            bricks[0][c] = GetRandomValue(1, level + 2);
        }
    }

    for (r = 0; r < ROWS; r++)
    {
        for (c = 0; c < COLS; c++)
        {
            if (bricks[r][c] > 0)
            {
                if (BrickY(r) + 40.0f >= 680.0f)
                {
                    gameOver = 1;
                }
            }
        }
    }
}

void UpdateGame(void)
{
    int r, c;
    float dt;
    Rectangle rect;
    Vector2 ballPosition;

    dt = GetFrameTime();

    if (!moving || gameOver)
        return;

    ballX += speedX * dt;
    ballY += speedY * dt;

    // Wall collision
    if (ballX < 7.0f)
    {
        ballX = 7.0f;
        speedX = -speedX;
    }

    if (ballX > W - 7.0f)
    {
        ballX = W - 7.0f;
        speedX = -speedX;
    }

    if (ballY < 85.0f)
    {
        ballY = 85.0f;
        speedY = -speedY;
    }

    // Brick collision
    ballPosition.x = ballX;
    ballPosition.y = ballY;

    for (r = 0; r < ROWS; r++)
    {
        for (c = 0; c < COLS; c++)
        {
            if (bricks[r][c] <= 0)
                continue;

            rect.x = BrickX(c);
            rect.y = BrickY(r);
            rect.width = 55.0f;
            rect.height = 40.0f;

            if (CheckCollisionCircleRec(ballPosition, 7.0f, rect))
            {
                bricks[r][c]--;
                score++;

                speedY = -speedY;

                return;
            }
        }
    }

    // Return ball
    if (ballY > H)
    {
        moving = 0;

        if (ballX < 7.0f)
            ballX = 7.0f;

        if (ballX > W - 7.0f)
            ballX = W - 7.0f;

        ballY = 730.0f;

        level++;

        NewRow();
    }
}

void DrawGame(void)
{
    int r, c;
    Rectangle rect;
    Color color;

    ClearBackground(DARKBLUE);

    DrawText("BRICK BREAKER", 125, 25, 22, WHITE);
    DrawText(TextFormat("SCORE : %d", score), 20, 75, 20, WHITE);
    DrawText(TextFormat("LEVEL : %d", level), 330, 75, 20, SKYBLUE);

    // Draw bricks
    for (r = 0; r < ROWS; r++)
    {
        for (c = 0; c < COLS; c++)
        {
            if (bricks[r][c] <= 0)
                continue;

            rect.x = BrickX(c);
            rect.y = BrickY(r);
            rect.width = 55.0f;
            rect.height = 40.0f;

            color = BLUE;

            if (bricks[r][c] >= 5)
                color = RED;
            else if (bricks[r][c] >= 3)
                color = ORANGE;

            DrawRectangleRec(rect, color);

            DrawText(
                TextFormat("%d", bricks[r][c]),
                (int)rect.x + 22,
                (int)rect.y + 10,
                20,
                WHITE
            );
        }
    }

    // Draw ball
    DrawCircle((int)ballX, (int)ballY, 7, WHITE);

    // Draw aim line
    if (aiming && !moving)
    {
        DrawLine(
            (int)ballX,
            (int)ballY,
            (int)aimX,
            (int)aimY,
            SKYBLUE
        );
    }

    // Bottom line
    DrawLine(0, 680, W, 680, RED);

    // Game over
    if (gameOver)
    {
        DrawRectangle(0, 0, W, H, Fade(BLACK, 0.7f));

        DrawText("GAME OVER", 120, 320, 35, RED);

        DrawText(
            TextFormat("SCORE : %d", score),
            165, 380, 22, WHITE
        );

        DrawText("CLICK TO RESTART", 115, 450, 20, WHITE);
    }
}

int main(void)
{
    Vector2 mouse;
    float dx, dy, length;

    InitWindow(W, H, "Brick Breaker");
    SetTargetFPS(60);

    ResetGame();

    while (!WindowShouldClose())
    {
        mouse = GetMousePosition();

        if (gameOver)
        {
            if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT))
            {
                ResetGame();
            }
        }
        else
        {
            // Start aiming
            if (!moving && !aiming)
            {
                if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT))
                {
                    if (mouse.y > 550.0f)
                    {
                        aiming = 1;
                    }
                }
            }

            // Move aim
            if (aiming && IsMouseButtonDown(MOUSE_BUTTON_LEFT))
            {
                aimX = mouse.x;
                aimY = mouse.y;

                if (aimY > 710.0f)
                    aimY = 710.0f;
            }

            // Shoot ball
            if (aiming && IsMouseButtonReleased(MOUSE_BUTTON_LEFT))
            {
                aiming = 0;

                dx = aimX - ballX;
                dy = aimY - ballY;

                length = sqrtf(dx * dx + dy * dy);

                if (length > 0.0f)
                {
                    speedX = dx / length * 500.0f;
                    speedY = dy / length * 500.0f;

                    if (speedY > -100.0f)
                        speedY = -100.0f;

                    moving = 1;
                }
            }

            UpdateGame();
        }

        BeginDrawing();

        DrawGame();

        EndDrawing();
    }

    CloseWindow();

    return 0;
}
```
