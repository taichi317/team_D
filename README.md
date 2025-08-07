//========= 横スクロールアクションゲーム 完全版（レベル制・ゲームオーバー対応） ===========

import processing.sound.*;

SoundFile bgm;
SoundFile attackSound;
SoundFile killSound;

PImage stage, nouka1, noukaAttack;
PImage inago1, suzume1, suzume2, heartImg, startScreenImg, goalImg;

int x = 0;
float y = 250;
float vy = 0;
boolean onGround = true;
float gravity = 0.5;
float jumpPower = -12;
int life = 3;

int groundY = 250;
int max = 2400;
float scrollX = 0;

int inagoCount = 2;
float[] inagoX, inagoY;
boolean[] inagoAlive;
boolean[] inagoHitCooldown;
int[] inagoHitTimer;

int suzumeCount = 2;
float[] suzumeX, suzumeY;
boolean[] suzumeAlive;
boolean[] suzumeHitCooldown;
int[] suzumeHitTimer;

int gameState = 0; // 0:スタート画面, 1:プレイ中
boolean isAttacking = false;
int attackStartTime = 0;
int attackDuration = 20;

boolean rightPressed = false;
boolean leftPressed = false;

float goalX = 2300;
float goalY = 220;
float goalW = 150;
float goalH = 120;

int score = 0;
int level = 1;

// プレイヤー当たり判定
float playerHitW = 60;
float playerHitH = 65;
float playerHitOffsetX = 8;
float playerHitOffsetY = 8;

void setup() {
  size(800, 400);
  loadImages();
  initializeGame();

  bgm = new SoundFile(this, "bgm.mp3");
  bgm.loop();

  attackSound = new SoundFile(this, "attackSound.mp3");
  killSound = new SoundFile(this, "killSound.mp3");
}

void loadImages() {
  stage = loadImage("IMG_1918.jpg");
  stage.resize(800, height);

  nouka1 = loadImage("無題2_20250804213754.png");
  nouka1.resize(130, 130);
  noukaAttack = loadImage("無題8_20250804220912.png");
  noukaAttack.resize(130, 130);

  inago1 = loadImage("IMG_0474.png");
  suzume1 = loadImage("IMG_0476.png");
  suzume2 = loadImage("IMG_0477.png");

  heartImg = loadImage("IMG_1927.png");
  heartImg.resize(40, 40);

  startScreenImg = loadImage("IMG_1926.jpg");
  goalImg = loadImage("IMG_1925.jpg");
  goalImg.resize(int(goalW), int(goalH));
}

void initializeGame() {
  x = 0;
  y = groundY;
  life = 3;
  scrollX = 0;
  isAttacking = false;
  score = 0;

  inagoCount = 2 + level;
  inagoX = new float[inagoCount];
  inagoY = new float[inagoCount];
  inagoAlive = new boolean[inagoCount];
  inagoHitCooldown = new boolean[inagoCount];
  inagoHitTimer = new int[inagoCount];

  for (int i = 0; i < inagoCount; i++) {
    inagoX[i] = random(400, max - 200);
    inagoY[i] = groundY;
    inagoAlive[i] = true;
    inagoHitCooldown[i] = false;
    inagoHitTimer[i] = 0;
  }

  suzumeCount = 2 + level;
  suzumeX = new float[suzumeCount];
  suzumeY = new float[suzumeCount];
  suzumeAlive = new boolean[suzumeCount];
  suzumeHitCooldown = new boolean[suzumeCount];
  suzumeHitTimer = new int[suzumeCount];

  for (int i = 0; i < suzumeCount; i++) {
    suzumeX[i] = random(600, max - 200);
    suzumeY[i] = random(-200, -50);
    suzumeAlive[i] = true;
    suzumeHitCooldown[i] = false;
    suzumeHitTimer[i] = 0;
  }
}

void draw() {
  background(255);

  if (gameState == 0) {
    image(startScreenImg, 0, 0, width, height);
    fill(0);
    textSize(32);
    textAlign(CENTER, CENTER);
    text("クリックでスタート", width / 2, height - 60);
    return;
  }

  if (life <= 0) {
    fill(255, 0, 0);
    textSize(50);
    textAlign(CENTER, CENTER);
    text("GAME OVER", width / 2, height / 2);
    textSize(30);
    text("Press R to restart", width / 2, height / 2 + 50);
    noLoop();
    return;
  }

  float moveSpeed = 5 + scrollX / 500;
  if (rightPressed && x < max) x += moveSpeed;
  if (leftPressed && x > 0) x -= moveSpeed * 0.5;
  scrollX = constrain(x - width / 2, 0, max - width);

  for (int i = 0; i < 3; i++) {
    image(stage, i * 800 - scrollX, 0);
  }

  if (hit(x + playerHitOffsetX, y + playerHitOffsetY, playerHitW, playerHitH, goalX, goalY, goalW, goalH)) {
    level++;
    initializeGame();
    return;
  }

  for (int i = 0; i < life; i++) {
    image(heartImg, 10 + i * (heartImg.width + 5), 10);
  }

  fill(0);
  textSize(20);
  textAlign(LEFT, TOP);
  text("SCORE: " + score, 10, 50);
  text("LEVEL: " + level, 10, 75);

  image(goalImg, goalX - scrollX, goalY, goalW, goalH);

  vy += gravity;
  y += vy;
  onGround = false;

  if (y >= groundY) {
    y = groundY;
    vy = 0;
    onGround = true;
  }

  if (isAttacking && frameCount - attackStartTime > attackDuration) {
    isAttacking = false;
  }

  image(isAttacking ? noukaAttack : nouka1, x - scrollX, y);

  drawEnemies(inagoX, inagoY, inagoAlive, inagoHitCooldown, inagoHitTimer, inago1, 110, 110, true);
  drawEnemies(suzumeX, suzumeY, suzumeAlive, suzumeHitCooldown, suzumeHitTimer, (frameCount / 6) % 2 == 0 ? suzume1 : suzume2, 100, 100, false);
}

void drawEnemies(float[] ex, float[] ey, boolean[] alive, boolean[] cooldown, int[] timers, PImage img, float ew, float eh, boolean isInago) {
  for (int i = 0; i < ex.length; i++) {
    if (!alive[i]) continue;

    if (isInago) {
      ex[i] -= 3;
      ey[i] = groundY;
    } else {
      float dx = x - ex[i];
      float dy = y - ey[i];
      float dist = sqrt(dx * dx + dy * dy);
      float speed = 4;
      ex[i] += (dx / dist) * speed;
      ey[i] += (dy / dist) * speed;
    }

    image(img, ex[i] - scrollX, ey[i], ew, eh);

    float px = x + playerHitOffsetX - 10;
    float py = y + playerHitOffsetY;

    if (hit(px, py, playerHitW, playerHitH, ex[i], ey[i], ew, eh)) {
      if (isAttacking) {
        alive[i] = false;
        score += 100;
        killSound.play();
      } else if (!cooldown[i]) {
        life--;
        cooldown[i] = true;
        timers[i] = frameCount;
      }
    }

    if (cooldown[i] && frameCount - timers[i] > 60) {
      cooldown[i] = false;
    }
  }
}

boolean hit(float x1, float y1, float w1, float h1, float x2, float y2, float w2, float h2) {
  return !(x1 + w1 < x2 || x2 + w2 < x1 || y1 + h1 < y2 || y2 + h2 < y1);
}

void keyPressed() {
  if (key == 'r' || key == 'R') {
    level = 1;
    initializeGame();
    loop();
  }
  if (key == 'd' || key == 'D') rightPressed = true;
  if (key == 'a' || key == 'A') leftPressed = true;
  if ((key == 'w' || key == 'W') && onGround) {
    vy = jumpPower;
  }
}

void keyReleased() {
  if (key == 'd' || key == 'D') rightPressed = false;
  if (key == 'a' || key == 'A') leftPressed = false;
}

void mousePressed() {
  if (gameState == 0) {
    gameState = 1;
    return;
  }
  if (mouseButton == LEFT && !isAttacking) {
    isAttacking = true;
    attackStartTime = frameCount;
    attackSound.play();
  }
}



 
