PImage stage, nouka1,nouka2, noukaAttack;
PImage inago1,inago2, suzume1, suzume2, heartImg, startScreenImg;

//プレイヤー用
int x = 0;
int posi = 0;
float y = 290;
float vy = 0;
boolean onGround = true;
float gravity = 0.5;
float jumpPower = -10;
int life = 3;

//スクロール用
int[] a = {0, 0};
int groundY = 290;
int min = 0;
int max = 1000;
int wid = 1000;
int hig = 600;

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
int gameState = 1; // 0:スタート画面は省略, 1:プレイ中, 2:ゲームオーバー

//攻撃
boolean isAttacking = false;
int attackStartTime = 0;
int attackDuration = 20;

//左右への動き
boolean rightPressed = false;
boolean leftPressed = false;

//ゴール
PImage goalImg;
float goalX = 950;  // ゴールのX座標（右端付近）
float goalY = 220;  // 地面と同じ高さ
float goalW = 50;
float goalH = 50;



void setup() {
  size(800, 400);
  loadImages();
  initializeGame();
}

void loadImages() {
  stage = loadImage("IMG_1918.jpg");
  stage.resize(width, height);

  nouka1 = loadImage("無題8_20250804220912.png");
  nouka1.resize(75, 75);
  nouka2 =loadImage("無題6_20250804223505.png");
  nouka2.resize(75,75);

  noukaAttack = loadImage("無題2_20250804213754.png"); // 同じ画像を代用
  noukaAttack.resize(75, 75);

  inago1 = loadImage("IMG_0474.png");
  inago1.resize(40, 40);
  inago2 =loadImage("IMG_0475.png");
  inago2.resize(40,40);

  suzume1 = loadImage("IMG_0476.png");
  suzume1.resize(40, 40);
  suzume2 = loadImage("IMG_0477.png");
  suzume2.resize(40, 40);

   heartImg = loadImage("IMG_1927.png");
   heartImg.resize(40,40);// コメントアウト中でもOK
  startScreenImg = loadImage("IMG_1926.jpg");
}

void initializeGame() {
  x = min;
  y = groundY;
  life = 3;
  isAttacking = false;

  // 配列は空なので初期化不要ですが、念のため
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
  //動きの処理
  if (rightPressed && x < max - wid / 2) {
  x += 5;
  posi = 0;
}
if (leftPressed && x > min) {
  x -= 5;
  posi = 1;
}
  
  
  background(255);
  
  

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

  if (isAttacking && frameCount - attackStartTime > attackDuration) {
    isAttacking = false;
  }

  image(isAttacking ? noukaAttack : nouka1, x * a[0] + 123 * a[1], y);

  drawEnemies(inagoX, inagoY, inagoAlive, hitCooldown, hitTimer, inago1, inagoW, inagoH, 0.5);
  drawEnemies(suzumeX, suzumeY, suzumeAlive, suzumehitCooldown, suzumehitTimer,
              (frameCount / 6) % 2 == 0 ? suzume1 : suzume2, suzumeW, suzumeH, 1.0);
}

void drawEnemies(float[] ex, float[] ey, boolean[] alive, boolean[] cooldown, int[] timers, PImage img, float ew, float eh, float speed) {
  for (int i = 0; i < ex.length; i++) {
    if (!alive[i]) continue;

    ex[i] -= speed;

    float drawX = ex[i] * a[0] + wid / 2 * a[1] - x * a[1];
    image(img, drawX, ey[i], ew, eh);

    if (hit(x, y, 75, 75, ex[i], ey[i], ew, eh)) {
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
    if (keyCode == RIGHT) {
      rightPressed = true;
    } else if (keyCode == LEFT) {
      leftPressed = true;
    }
  }
}

void keyReleased() {
  if (key == CODED) {
    if (keyCode == RIGHT) {
      rightPressed = false;
    } else if (keyCode == LEFT) {
      leftPressed = false;
    }
  }
}


void mousePressed() {
  // 右クリックでジャンプ（地面にいる時のみ）
  if (mouseButton == RIGHT && onGround) {
    vy = jumpPower;
    onGround = false;
  }

  //左クリックは攻撃（必要なら有効化）
   if (mouseButton == LEFT) {
     if (!isAttacking) {
       isAttacking = true;
       attackStartTime = frameCount;
     }
   }
}
