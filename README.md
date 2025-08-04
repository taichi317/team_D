//﻿# team_D

//キャラ画像
Plmage  stage,nouka1,nouka2,nouka3,nouka4,inago1,inago2,
inago3,inago4,suzume1,suzume2,suzume3,suzume4 ,heratImg ,suzumeImg1,suzumeImg2,suzume3,suzume4  ; //ここにつかう画像の初期化

//プレイヤー用
int x =   ; //キャラクターのステージ上のｘ座標（スクロール対応）
int posi=0  ;//向き（０：右向き、１：左向き）
float y=     ;//地面の高さ,キャラのY座標（初期は地面）
float vy =0;//y方向の速度
boolean onGround= true;//地面にいるかどうか
float gravity=0.5;
float jumpPower=-10;
int life = 3  ;//ライフ三つ

//スクロール用
int [] a={0 , 0};//描画位置計算用の補助変数
int groundY=地上からの高さ;
int min =0;//ステージの左端
int max=   ;//ステージの右端
int wid=   ;//画像の幅（ウィンドウサイズ）
int hig =  ;//画像の高さ（ウィンドサイズ）


//イナゴ
int inagoCount=   ;
float [] inagoX=new float{   ,   ,   };
float [] inagoY=new float{   ,   ,   };
float inagoW=  ;
float inagoH=  ;
boolean[] hitCooldown=new boolean[inagoCount];
int[]hitTimer = new int [inagoCount];

//状態管理
int gameState= 0;//0=スタート画面、１＝ゲーム中、２＝ゲームオーバー

//スズメ
int suzumeCount=   ;
float [] suzumeX= new float{   ,   ,   };
float [] suzumeY= new float{   ,   ,   };
float suzumeW=  ;
float suzumeH=  ;
boolean[] suzumehitCooldown= new boolean[suzumeCount];
int[] suzumehitTimer = new int [suzumeCount];

//攻撃
boolean isAttacking =false;
int attackStartTime = 0;
int attackDuration  = 20;

//各敵に「いきているかどうか」のフラグを追加
boolean[] inagoAlive = new boolean[inagoCount];
boolean[] suzumeAlive = new boolean[suzumeCount];

void setup(){
  
  //ライフがなくなったらゲーム終了
  if (life<=0){
    background(0);
    fill(255,0,0);
    textSize(50);
    textAlign(CENTER,CENTER);
    text("GAME OVER",width/2,height/2);
    noLoop();
    return;
  }
  size( wid   ,hig     );
    background(255);
    loadImages();
    initializeGame();
}
void initializeGame(){
  x = wid/2;
  y = groundY;
  vy = 0;
  onGround =true;
  life = 3;
  isAttacking =false;

  for(int i = 0 ;i<inagoCount;i++)
  inagoX[i]=random(min,max);
  inagoY[i]=groundY;
  //イナゴがプレイヤーに当たってから再び当たれるまでのクールダウン処理
  hitCooldown[i]=false;//まだ当たり判定中
  hitTimer[i]=0;//クールダウンのカウント開始時間をリセット
  inagoAlive[i]= true;//イナゴが生きている状態に初期化

  for(int i = 0 ;i<suzumeCount;i++)
  suzumeX[i]=random(min+100,max-100);
  inagoY[i]=groundY-30;
  suzumehitCooldown[i]=false;
  suzumehitTimer[i]=0;
  suzemeAlive[i]=true;
}

  void loadImages(){
  stage =loadImage("");
  nouka1=loadImage("");
  nouka2=loadImage("");
  nouka3=loadImage("");
  nouka4=loadImage("");
  inago=loadImage("");
  heartImg=loadImage("")
  suzumeImg1=loadImage("");
  suzumeImg2=loadImage("");
  suzumeImg3=loadImage("");
  suzumeImg4=loadImage("");

  for (int i=0;i<inagoCount;i++)
  {
  inagoY[i]=getGroundY();
  hitCooldown[i]=false;
  hitTimer[i]=0;}
  }
  }

void draw(){
  background(255);
  if(gameState==0){
    //スタート画面
    image(startScreenImg,0,0,wid,hig);
    fill(255);
    textSize(32);
    text("クリックでスタート",wid/2,hig-50);
    return;
  }
  for(int i=0;i<life;i++)
  {
    image( heartImg ,10+i*(heartImg.width+5),10);
  }
   if (life<=0){
    background(0);
    fill(255,0,0);
    textSize(50);
    textAlign(CENTER,CENTER);
    text("GAME OVER",width/2,height/2);
    textSize(20);
    text("Press R to restart",width/2,height/2+50)+
    noLoop();
    return;
  } 
    
    
    //draw stage
    if(x<min+wid/2){
        a[0]=1;
        a[1]=0;
        image(stage,0,0);//ステージの左端を表示
    }
    else if (x>max-wid/2){
        a[0]=1; 
        a[1]=0;
        image(stage,-(max-wid)  ,    );//ステージの右端を表示（max-wid=       )
            }
            else{
                a[0]=0;
                a[1]=1;
                image(stage,-x+wid/2  ,   );//プレイヤーが画面中央に来るようにステージをスクロール
            }
    //draw gravity
    vy+=gravity;
    y+=vy;    
    if (y>groundY){
        y=groundY;
        vy=0;
        onGround=true;
    } else {
      onGround=false;
    }  
    //農家キャラの描画
    int anim=(frameCount/6)%2;

    if(posi==0){//右向き
        if(anim==0){
            image( nouka1     ,x*a[0]+画面の中央座標*a[1],noukaのY座標);
        }else{
            image(nouka2,x*a[0]+画面の中央座標*a[1],noukaのY座標);
        }

    }
    else{//左向き
    if(anim==0){
        image(nouka3,x*a[0]+画面の中央座標*a[1],noukaのY座標);
    }else{
        image(nouka4,x*a[0]+画面の中央座標*a[1],noukaのY座標);
    }

    } 
    for (int i=0;i<inagoCount;i++)
    {
    float drawX=inagoX[i]*a[0]+wid/2*a[1]-x*a[1];
    image(inago,drawX,inagoY[i],inagoW,inagoH);//イナゴのイラストを表示

    //ここにイラスト対応の当たり判定を追加する
    if(!hitCooldown[i]&&hit(
      x,y,nouka1.width,nouka1.height,inagoX[i], inagoY[i], inagoW, inagoH)){
        life--;
        hitCooldown[i]=true;
        hitTimer[i]=frameCount;
      }
      //クールダウンの解除処理（１秒後に再判定できるように）
      if(hitCooldown[i]&&frameCount-hitTimer[i]>60){
        hitCooldown[i]=false;
      }

      //スズメの描画
      int suzumeAnim =(frameCount/6)%4;//4枚アニメ
      for (int i=0; i<suzumeCount;i++)
      {
        float suzuemeDrawX=suzumeX[i]*a[0]+wid/2*a[1]-x*a[1];
        //アニメ画面を切り替え
        PImage surrentSuzumeImg;
        switch(suzumeAnim){
          case 0:currentSuzumeImg=suzumeImg1;break;
          case 1:currentSuzumeImg=suzumeImg2;break;
          case 2:currentSuzumeImg=suzumeImg3;break;
          default:currentSuzumeImg=suzumeImg4;break;
        }
       image(currentSuzumeImg,suzumeDrawX,suzumeY[i],suzumeW,suzumeH);

      if(!suzumehitCooldown[i]&&hit(
      x,y,nouka1.width,nouka1.height,inagoX[i], inagoY[i], inagoW, inagoH)){
        life--;
        suzumehitCooldown[i]=true;
        suzumehitTimer[i]=frameCount;
      }
      //クールダウンの解除処理（１秒後に再判定できるように）
      if(suzumehitCooldown[i]&&frameCount-hitTimer[i]>60){
         suzumehitCooldown[i]=false;
      }
      
}
    }

fill(255,0,0)
}


void keyPressed(){
  if(key=='r'||key=='R'){
    if(life<=0){
      resetGame();//ライフが０の時だけ再スタート
    }
  }
  
  if(key==CODED){
    if(keyCode==RIGHT){
      if(x<max-wid/2){
        x=x+5;
        posi=0;
      }
    }else if(keyCode==LEFT){
      if(x>min){
        x=x-5;
        posi=1;
      }
    }
  }
  if(key== ' '&& onGround){
    vy=jumpPower;
    onGround=false;
  }
//二つの短刑（x、y、幅、高さ）がぶつかっているのかどうか判定
boolean hit (float x1,float y1,float w1,float h1,
float x2,float y2,float w2,float h2){
  return !(x1+w1<x2||x2+w2<x1||y1+h1<y2||y2+h2<y1);
  }

  //再スタート
  void resetGame(){
    x=      ;
    y=      ;
    vy=0;
    onGround=false;
    life=3;
    //敵の位置も再設定
    for(int i =0; i<inagoX.length;i++)
    {inagoX[i]=      ;
    hitCooldown[i]=false;
    hitTimer[i]=0;}
    for(int i =0;i<suzumeCount;i++)
    {
      suzumeX[i]=random(min+100,max-100);
      suzuemhitcooldown[i]=false;
      suzumehitTimer[i]=0;
    }
    loop();//draw()のループを再開
    
    }  }
  //マウスでの操作
  void mousePressed(){
  if (gameState==0){
    gameState=1;
    loop();
    return;
     } 
  //マウスの右クリックで攻撃開始
  if (mouseButton==RIGHT){
    if(!isAttacking){
      isAttacking=true;
      attackStartTime=frameCount;
    }
  }

  }

}



