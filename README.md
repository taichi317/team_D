PImage stage, nouka1, nouka2, nouka3, nouka4, noukaAttack;
PImage inago1, inago2;
PImage suzume1, suzume2, heartImg, startScreenImg;

//プレイヤー用
int x = 0;
int posi = 0;
float y = 40;
float vy = 0;
boolean onGround = true;
float gravity = 0.5;
float jumpPower = -10;
int life = 3;

//スクロール用
int[] a = {0, 0};
int groundY = 40;
int min = 0;
int max = 256;
int wid = 240;
int hig = 240;

//イナゴ
int inagoCount = 3;
float[] inagoX = new float[inagoCount];
float[] inagoY = new float[inagoCount];
float inagoW = 40, inagoH = 40;
boolean[] hitCooldown = new boolean[inagoCount];
int[] hitTimer = new int[inagoCount];
boolean[] inagoAlive = new boolean[inagoCount];

//スズメ
int suzumeCount = 3;
float[] suzumeX = new float[suzumeCount];
float[] suzumeY = new float[suzumeCount];
float suzumeW = 40, suzumeH = 40;
boolean[] suzumehitCooldown = new boolean[suzumeCount];
int[] suzumehitTimer = new int[suzumeCount];
boolean[] suzumeAlive = new boolean[suzumeCount];

//状態管理
int gameState = 0; // 0:スタート, 1:プレイ中, 2:ゲームオーバー

//攻撃
boolean isAttacking = false;
int attackStartTime = 0;
int attackDuration = 20;

void setup() {
  size(800, 600);
  loadImages();
  initializeGame();
}

void loadImages() {
  stage = loadImage("IMG_1917.jpg");
  nouka1 = loadImage("無題2_20250804213754.png");
  //nouka2 = loadImage("無題2_20250804213754.png"); // 仮に同じ画像
  //nouka3 = loadImage("無題2_20250804213754.png");
  //nouka4 = loadImage("無題2_20250804213754.png");
  //noukaAttack = loadImage("無題2_20250804213754.png"); // 攻撃画像があれば差し替え
  inago1 = loadImage("IMG_8964イナゴ2.JPG");
  suzume1 = loadImage("imageすずめ1.PNG");
  suzume2 = loadImage("imageすずめ2.PNG");
  //heartImg = loadImage("heart.png"); // ライフ画像
  //startScreenImg = loadImage("startScreen.png");
}

void initializeGame() {
  x = min;
  y = groundY;
  life = 3;
  isAttacking = false;

  for (int i = 0; i < inagoCount; i++) {
    inagoX[i] = random(min + 100, max - 100);
    inagoY[i] = groundY;
    hitCooldown[i] = false;
    hitTimer[i] = 0;
    inagoAlive[i] = true;
  }
  for (int i = 0; i < suzumeCount; i++) {
    suzumeX[i] = random(min + 100, max - 100);
    suzumeY[i] = groundY - 30;
    suzumehitCooldown[i] = false;
    suzumehitTimer[i] = 0;
    suzumeAlive[i] = true;
  }
}

void draw() {
  background(255);

  if (gameState == 0) {
    image(startScreenImg, 0, 0);
    fill(0);
    textAlign(CENTER, CENTER);
    textSize(32);
    text("クリックでスタート", width / 2, height - 50);
    return;
  }

  if (life <= 0) {
    fill(255, 0, 0);
    textSize(50);
    textAlign(CENTER, CENTER);
    text("GAME OVER", width / 2, height / 2);
    text("Press R to restart", width / 2, height / 2 + 50);
    noLoop();
    return;
  }

  if (allEnemiesDefeated()) {
    fill(0, 200, 0);
    textSize(50);
    textAlign(CENTER, CENTER);
    text("GAME CLEAR!", width / 2, height / 2);
    noLoop();
    return;
  }

  for (int i = 0; i < life; i++) {
    if (heartImg != null) image(heartImg, 10 + i * (heartImg.width + 5), 10);
  }

  if (x < min + wid / 2) {
    a[0] = 1;
    a[1] = 0;
    image(stage, 0, 0);
  } else if (x > max - wid / 2) {
    a[0] = 1;
    a[1] = 0;
    image(stage, -(max - wid), 0);
  } else {
    a[0] = 0;
    a[1] = 1;
    image(stage, -x + wid / 2, 0);
  }

  vy += gravity;
  y += vy;
  if (y > groundY) {
    y = groundY;
    vy = 0;
    onGround = true;
  } else {
    onGround = false;
  }

  // 攻撃終了処理
  if (isAttacking && frameCount - attackStartTime > attackDuration) {
    isAttacking = false;
  }

  // キャラ描画
  PImage playerImg = isAttacking ? noukaAttack : (posi == 0 ? nouka1 : nouka3);
  image(playerImg, x * a[0] + 123 * a[1], y);

  // 敵描画（共通関数使用）
  drawEnemies(inagoX, inagoY, inagoAlive, hitCooldown, hitTimer, inago1, inagoW, inagoH);
  drawEnemies(suzumeX, suzumeY, suzumeAlive, suzumehitCooldown, suzumehitTimer, 
              (frameCount / 6) % 2 == 0 ? suzume1 : suzume2, suzumeW, suzumeH);
}

void drawEnemies(float[] ex, float[] ey, boolean[] alive, boolean[] cooldown, int[] timers, PImage img, float ew, float eh) {
  for (int i = 0; i < ex.length; i++) {
    if (!alive[i]) continue;
    float drawX = ex[i] * a[0] + wid / 2 * a[1] - x * a[1];
    image(img, drawX, ey[i], ew, eh);

    if (hit(x, y, 50, 50, ex[i], ey[i], ew, eh)) {
      if (isAttacking) {
        alive[i] = false;
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

boolean allEnemiesDefeated() {
  for (boolean b : inagoAlive) if (b) return false;
  for (boolean b : suzumeAlive) if (b) return false;
  return true;
}

void keyPressed() {
  if (key == 'r' || key == 'R') {
    if (life <= 0 || allEnemiesDefeated()) {
      initializeGame();
      loop();
    }
  }

  if (key == CODED) {
    if (keyCode == RIGHT && x < max - wid / 2) {
      x += 5;
      posi = 0;
    } else if (keyCode == LEFT && x > min) {
      x -= 5;
      posi = 1;
    }
  }

  if (key == ' ' && onGround) {
    vy = jumpPower;
    onGround = false;
  }
}

void mousePressed() {
  if (gameState == 0) {
    gameState = 1;
    loop();
    return;
  }

  if (mouseButton == RIGHT) {
    if (!isAttacking) {
      isAttacking = true;
      attackStartTime = frameCount;
    }
  }
}


 
