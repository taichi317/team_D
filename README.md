PImage stage, nouka1, nouka2, noukaAttack;
PImage inago1, inago2, suzume1, suzume2, heartImg, startScreenImg, goalImg;

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
int gameState = 0; // 0:スタート画面は省略, 1:プレイ中, 2:ゲームオーバー

//攻撃
boolean isAttacking = false;
int attackStartTime = 0;
int attackDuration = 20;

//左右への動き
boolean rightPressed = false;
boolean leftPressed = false;

//ゴール
float goalX = 1000;
float goalY = 220;
float goalW = 50;
float goalH = 50;

//スタートボタンの座標と大きさ　135と273にこれに関するプログラムある
float startBtnX=275;
float startBtnY=125;
float startBtnW=245;
float startBtnH=60;

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
  nouka2 = loadImage("無題6_20250804223505.png");
  nouka2.resize(75, 75);
  noukaAttack = loadImage("無題2_20250804213754.png");
  noukaAttack.resize(75, 75);

  inago1 = loadImage("IMG_0474.png");
  inago1.resize(40, 40);
  inago2 = loadImage("IMG_0475.png");
  inago2.resize(40, 40);

  suzume1 = loadImage("IMG_0476.png");
  suzume1.resize(40, 40);
  suzume2 = loadImage("IMG_0477.png");
  suzume2.resize(40, 40);

  heartImg = loadImage("IMG_1927.png");
  heartImg.resize(40, 40);

  startScreenImg = loadImage("IMG_1926.jpg");

  goalImg = loadImage("IMG_1925.jpg"); // ← ファイル名を適切に
  goalImg.resize(int(goalW), int(goalH));
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

void draw() {background(255);
  
  //コジコジ：ここから
   if (gameState == 0) {
    if (startScreenImg != null) {
      image(startScreenImg, 0, 0, width, height);
      // ボタン範囲を見えるようにしたい時（デバッグ用）
       //noFill(); stroke(255, 0, 0); rect(startBtnX, startBtnY, startBtnW, startBtnH);
    } else {
      fill(0);
      textSize(50);
      textAlign(CENTER, CENTER);
      text("Press SPACE to Start", width / 2, height / 2);
    }
    return; // draw終了
  }
//ここまでスタート画面のプログラム
  
  
  
  // キャラの左右移動
  if (rightPressed && x < max - wid / 2) {
    x += 5;
    posi = 0;
  }
  if (leftPressed && x > min) {
    x -= 5;
    posi = 1;
  }

 

  // ゲームオーバー処理
  if (life <= 0) {
    fill(255, 0, 0);
    textSize(50);
    textAlign(CENTER, CENTER);
    text("GAME OVER", width / 2, height / 2);
    text("Press R to restart", width / 2, height / 2 + 50);
    noLoop();
    return;
  }

  // ゴール判定（ゲームクリア）
  if (hit(x, y, 75, 75, goalX, goalY, goalW, goalH)) {
    fill(0, 200, 0);
    textSize(50);
    textAlign(CENTER, CENTER);
    text("GAME CLEAR!", width / 2, height / 2);
    text("Press R to restart", width / 2, height / 2 + 50);
    noLoop();
    return;
  }

  // ライフ表示
  for (int i = 0; i < life; i++) {
    if (heartImg != null) image(heartImg, 10 + i * (heartImg.width + 5), 10);
  }

  // 背景スクロール処理
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

  // ゴール描画
  float goalDrawX = goalX * a[0] + wid / 2 * a[1] - x * a[1];
  image(goalImg, goalDrawX, goalY, goalW, goalH);

  // 重力とジャンプ
  vy += gravity;
  y += vy;
  if (y > groundY) {
    y = groundY;
    vy = 0;
    onGround = true;
  } else {
    onGround = false;
  }

  // 攻撃の時間制御
  if (isAttacking && frameCount - attackStartTime > attackDuration) {
    isAttacking = false;
  }

  // キャラ描画
  image(isAttacking ? noukaAttack : nouka1, x * a[0] + 10* a[1], y);

  // 敵の描画
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
    initializeGame();
    loop();
  }
if (key == 'd' || key == 'D') {
    rightPressed = true;
  } else if (key == 'a' || key == 'A') {
    leftPressed = true;
  }

  
}

void keyReleased() {
  if (key == 'd' || key == 'D') {
  rightPressed = false;
} else if (key == 'a' || key == 'A') {
  leftPressed = false;
  
}
if ((key == 'w' || key == 'W') && onGround) {
  vy = jumpPower;
  onGround = false;
}


}

void mousePressed() {
  

  if (mouseButton == LEFT) {
    if (!isAttacking) {
      isAttacking = true;
      attackStartTime = frameCount;
    }
  }
  //ここがクリックしてほしいところを決める関数
  if (gameState == 0) {
    // スタート画面中にクリックされたら範囲チェック
    if (mouseX >= startBtnX && mouseX <= startBtnX + startBtnW &&
        mouseY >= startBtnY && mouseY <= startBtnY + startBtnH) {
      gameState = 1;
      loop();
    }
    return; // スタート画面だった場合は処理終了
  }
}
