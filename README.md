// Gabriel Farley, Ewan Carver, and Grace Perry | 18 Nov 2025 | LETLOOSE
//----------------------------------------------------------------------
//GLOBALS
//-------------------------------------------------------

char screen = 's';   // s = start, t = settings, p = play, u = pause, g = game over, a = app stats
Button btnStart, btnPause, btnSettings, btnBack;

import gifAnimation.*;
Player p1;
Carl carl1;
LevelManage lvm;
ArrayList<Platform> platforms = new ArrayList<Platform>();
ArrayList<Bullet> bullets = new ArrayList<Bullet>();
ArrayList<Carl> carls = new ArrayList<Carl>();

ArrayList<EnemyBullet> enemyBullets = new ArrayList<EnemyBullet>();
PImage title;

int playerHP = 100;
int score = 0;

// Camera floats and controls
float camX = 0;
float camY = 0;
float zoom = 1.0;         // actual zoom used for drawing
float targetZoom = 1.0;   // where we want the zoom to go

//-------------------------------------------------------

void setup() {
  size(1200, 800);
  pixelDensity(1);
  noSmooth();
  p1 = new Player(this);
  lvm = new LevelManage();
  btnStart    = new Button("Start", 640/2+10, height/2+100, 640, 240);
  btnSettings    = new Button("Settings", 560/2+10, height/2+260, 560, 200);
  title = loadImage("LetLooseTitle.png");
  screen = 's';
}

//-------------------------------------------------------

void draw() {
  background(22);
  // SCREEN MANAGE
  switch(screen) {


  case 's': // start screen - Ewan Carver
    btnStart.display();
    btnSettings.display();
    if (btnStart.clicked()) {

      screen = 'p';
    } else if (btnSettings.clicked()) {

      screen = 't';
    }
    drawStart();
    break;
  case 't':
    background(20);
    drawSettings();
    break;

  case 'u':
    background(20);
    btnStart.display();
    if (btnStart.clicked()) {
      screen = 'p';
    }
    drawPause();
    break;
  case 'g':


    drawGameOver();
    break;
  case 'a':

    drawStats();
    break;
  case 'p':
    play();
    drawHUD();
    break;
  }
}

void play() {


  // Smoothly interpolate zoom (like camera position)
  float zoomLerpSpeed = 0.1; // smaller = slower/smoother
  zoom = lerp(zoom, targetZoom, zoomLerpSpeed);

  // Target camera position (centered on player)
  float targetCamX = width/(2*zoom) - p1.x;
  float targetCamY = height/(2*zoom) - p1.y; // or 0 if you only want horizontal scrolling

  // Smoothly interpolate (lerp) toward the target
  float lerpSpeed = 0.1; // smaller = smoother/slower camera
  camX = lerp(camX, targetCamX, lerpSpeed);
  camY = lerp(camY, targetCamY, lerpSpeed);

  // Convert mouse position from screen coords to world coords (important)
  float worldMouseX = (mouseX / zoom) - camX;
  float worldMouseY = (mouseY / zoom) - camY;

  // Apply camera transform for drawing world
  pushMatrix();
  scale(zoom);
  translate(camX, camY);

  // Draw world
  lvm.display();
  for (Platform p : platforms) {
    p.display();
    p.update();
  }
  // --- Update all Carls ---
  for (int i = carls.size()-1; i >= 0; i--) {
    carls.get(i).update(p1);
    carls.get(i).display();
  }
  for (int i = carls.size()-1; i >= 0; i--) {
  Carl c = carls.get(i);
  c.update(p1);
  c.display();
  if (c.hp <= 0) carls.remove(i);
}
  //Update enbullets
  for (int i = enemyBullets.size()-1; i >= 0; i--) {
    EnemyBullet eb = enemyBullets.get(i);
    eb.update();
    eb.display();

    if (dist(eb.x, eb.y, p1.x, p1.y) < 30) {
      playerHP -= 10;
      eb.dead = true;

      if (playerHP <= 0) screen = 'g';
    }

    if (eb.dead) enemyBullets.remove(i);
  }
  // Update player (physics)
  p1.update(platforms);

  // Update gun aiming (provide world mouse)
  p1.gun.update(worldMouseX, worldMouseY);

  // Update bullets (they are in world space)
  for (int i = bullets.size()-1; i >= 0; i--) {
    Bullet b = bullets.get(i);
    b.update();
    b.display();
    if (b.dead) bullets.remove(i);
  }




  // Draw player + gun
  p1.display();

  popMatrix();
}
void drawHUD() {
  fill(255);
  textSize(24);
  textAlign(LEFT);
  text("HP: " + playerHP, 20, 40);

  textAlign(RIGHT);
  text("Score: " + score, width - 20, 40);
}
void drawStart() {
  background(31, 0, 0);
  textAlign(CENTER);
  // textSize(32);
  //text("START SCREEN", width/2, 50);
  imageMode(CENTER);
  image(title, width/2, 800/2);
  btnStart.display();
  btnSettings.display();
}

void drawPause() {
  background(120, 200, 140);
  textSize(32);
  fill(255);
  text("PAUSE SCREEN", width/2, 50);
  // btnPause.display();
}
// Grace
void drawSettings() {
  background(200, 150, 120);
  textSize(32);
  fill(255);
  text("SETTINGS", width/2, 50);
  btnSettings.display();
}
// Gabriel
void drawGameOver() {
  background(0);
  textSize(32);
  fill(255);
  text("GAME OVER", width/2, 50);
  //btnRetry.display();
}
// Ewan
void drawStats() {
  background(0);
  textSize(32);
  fill(0, 255, 0);
  text("GET READY TO GET STATISTICAL", width/2, 50);
  //btnRetry.display();
}

void mousePressed() {
  // Fire a bullet from the gun when mouse pressed.
  // Convert mouse to world coords again (mouseX,mouseY are screen coords)
  // Only shoot in play mode
  if (screen == 'p') {
    float worldMouseX = (mouseX / zoom) - camX;
    float worldMouseY = (mouseY / zoom) - camY;
    p1.gun.fire(worldMouseX, worldMouseY, bullets);
  }
}

void keyPressed() {
  if (key == 'a' || key == 'A') p1.moveLeft = true;
  if (keyCode == LEFT) p1.moveLeft = true;
  if (key == 'd'|| key == 'D') p1.moveRight = true;
  if (keyCode == RIGHT) p1.moveRight = true;
  if (key == ' ') p1.jump();
  if (keyCode == UP) p1.jump();
  if (key == 'w'|| key == 'W') p1.jump();
  if (key == '+') targetZoom *= 1.1; // zoom in w/camera
  if (key == '-') targetZoom /= 1.1; // zoom out


  if (key == '1') screen = 's';
  if (key == '2') screen = 't';
  if (key == '3') screen = 'p';
  if (key == '4') screen = 'u';
  if (key == '5') screen = 'g';
  if (key == '6') screen = 'a';
}

// When the key is released
void keyReleased() {
  if (key == 'a' || key == 'A') p1.moveLeft = false;
  if (keyCode == LEFT) p1.moveLeft = false;
  if (key == 'd'|| key == 'D') p1.moveRight = false;
  if (keyCode == RIGHT) p1.moveRight = false;
}

class Bullet {
  float x, y;
  float vx, vy;
  float speed = 25;
  float radius = 5;
  int life = 90;
  boolean dead = false;

  Bullet(float x, float y, float angle) {
    this.x = x;
    this.y = y;
    vx = cos(angle) * speed;
    vy = sin(angle) * speed;
  }

  void update() {
    x += vx;
    y += vy;
    life--;
    if (life <= 0) dead = true;

    // Collision with carls
    for (int i = carls.size()-1; i >= 0; i--) {
      Carl c = carls.get(i);
      if (dist(x, y, c.x, c.y) < 40) {
        c.damage(25);
        dead = true;
        break;
      }
    }
  }

  void display() {
    pushMatrix();
    translate(x, y);
    noStroke();
    fill(255, 220, 60);
    ellipse(0, 0, radius*2, radius*2);
    popMatrix();
  }
}

// -------------------------------------------
// BUTTON CLASS
// -------------------------------------------
class Button {
  String label;
  float x, y, w, h;
  PImage button;

  Button(String label, float x, float y, float w, float h) {
    this.label = label;
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    if (label.equals("Start")) {

      button = loadImage("StartButton.png");
    } else if (label.equals("Settings")) {

      button = loadImage("SettingsButton.png");
    }
  }

  void display() {
    imageMode(CENTER);
    image(button, x, y, w, h);
    // fill(255);
    //stroke(0);
    //rect(x, y, w, h, 10);
    //fill(0);
    //textAlign(CENTER, CENTER);
    //textSize(16);
    //text(label, x + w/2, y + h/2);
  }

  boolean clicked() {
    return (mouseX > x-w/2 && mouseX < x+w/2 && mouseY > y-h/2 && mouseY < y+h/2 && mousePressed);
  }
}

class Carl {
  float x, y;
  float speed = 1.2;
  float orbitDistance = 200;
  float hp = 5;
  float shootCooldown = 120;
  float shootTimer = 0;
  float attackMode = 0;     // 0 = normal shots, 1 = spread shotsd
  float activationDistance = 50;   // how close player must be
  boolean active = false;
  float idleTime = 0;
  float idleAmplitude = 15;   // how far it floats
  float idleSpeed = 0.03;     // how fast it floats

  float startX, startY;       // original spawn position


  PImage sprite;

  Carl(float x, float y) {
    this.x = x;
    this.y = y;
    startX = x;
    startY = y;
    sprite = loadImage("Carl.gif");
  }

  void update(Player p) {
    if (hp <= 0) return;

    float d = dist(x, y, p.x, p.y);
    // --- Idle mode (enemy is dormant) ---
    if (!active) {
      if (d < activationDistance) {
        active = true;
      } else {
        idleFloat();    // << run idle motion
        return;
      }
    }
    // Check activation
    if (!active) {
      if (d < activationDistance) {
        active = true;  // wakes up
      } else {
        return;         // stays dormant
      }
    }

    // From here on, the enemy is active
    // ---------------------------------------------

    // Smooth follow/orbit logic
    float dx = p.x - x;
    float dy = p.y - y;
    float distToPlayer = dist(x, y, p.x, p.y);
    float angle = atan2(dy, dx);

    float targetX, targetY;

    if (distToPlayer > orbitDistance + 60) {
      targetX = x + cos(angle) * speed;
      targetY = y + sin(angle) * speed;
    } else if (distToPlayer < orbitDistance - 60) {
      targetX = x - cos(angle) * speed;
      targetY = y - sin(angle) * speed;
    } else {
      targetX = x + cos(angle + HALF_PI) * speed;
      targetY = y + sin(angle + HALF_PI) * speed;
    }

    x = lerp(x, targetX, 0.6);
    y = lerp(y, targetY, 0.6);

    // Shooting
    shootTimer--;
    if (shootTimer <= 0) {
      if (attackMode == 0) shootDirect(p);
      if (attackMode == 1) shootSpread(p);

      attackMode = int(random(2));
      shootTimer = shootCooldown;
    }
  }


  void shootDirect(Player p) {
    enemyBullets.add(new EnemyBullet(x, y, p.x, p.y));
  }

  void shootSpread(Player p) {
    // Bullet spread stuffs woooooooo
    float base = atan2(p.y - y, p.x - x);
    float spread = radians(25);

    float[] angles = { base - spread, base, base + spread };

    for (float a : angles) {
      enemyBullets.add(new EnemyBullet(x, y, x + cos(a), y + sin(a)));
    }
  }

  void damage(float dam) {
    hp -= dam;
    if (hp <= 0) {
      score += 1;    // add to global score
    }
  }
  
void idleFloat() {
  idleTime += idleSpeed;

  // Horizontal bobbing
  x = startX + sin(idleTime) * idleAmplitude;

  // Vertical bobbing (optional)
  y = startY + cos(idleTime * 0.8) * (idleAmplitude * 0.7);
}

  void display() {
    if (hp <= 0) return;

    pushMatrix();
    translate(x, y);

    // Only face the player when active
    float ang = active ? atan2(p1.y - y, p1.x - x) : 0;

    rotate(ang);
    imageMode(CENTER);
    image(sprite, 0, 0, 80, 80);
    popMatrix();
  }
}

class EnemyBullet {
  float x, y, vx, vy;
  float speed = 9;
  float radius = 6;
  int life = 240;
  boolean dead = false;

  EnemyBullet(float x, float y, float tx, float ty) {
    this.x = x;
    this.y = y;

    float angle = atan2(ty - y, tx - x);
    vx = cos(angle) * speed;
    vy = sin(angle) * speed;
  }

  void update() {
    x += vx;
    y += vy;
    life--;
    if (life <= 0) dead = true;
  }

  void display() {
    pushMatrix();
    translate(x, y);
    noStroke();
    fill(255, 60, 60);
    ellipse(0, 0, radius*2, radius*2);
    popMatrix();
  }
}

class Gun {
  Player p;
  float angle = 0;
  float length = 35;   // how far barrel extends from player center
  float thickness = 8; // visual thickness
  float offsetRadius = 0; // if you want the gun to orbit slightly away from center

  Gun(Player p) {
    this.p = p;
  }

  void update(float targetX, float targetY) {
    // angle pointing from player center to mouse in world coords
    angle = atan2(targetY - p.y, targetX - p.x);
  }

  void display() {
    pushMatrix();
    translate(p.x, p.y); // player center in world coords
    rotate(angle);
    noStroke();
    fill(80);
    // Draw barrel (rectangle). Draw from player center to the right (0..length)
      rectMode(CORNER);
    // shift upward so it's centered vertically
    rect(offsetRadius, -thickness/2, length, thickness, 3);
    // draw small muzzle
    fill(200,160,50);
    rect(offsetRadius + length, -thickness/2 - 2, 6, thickness + 4, 2);
    popMatrix();
  }

  void fire(float targetX, float targetY, ArrayList<Bullet> bullets) {
    // spawn bullet at muzzle position (world)
    float muzzleX = p.x + cos(angle) * (offsetRadius + length + 6); // a bit ahead of barrel
    float muzzleY = p.y + sin(angle) * (offsetRadius + length + 6);
    bullets.add(new Bullet(muzzleX, muzzleY, angle));
  }
}

class LevelManage {
  levTest lvt;
  levOne lv1;
  boolean tl, l1;
  LevelManage() {
    //tl = true;
    //lvt = new levTest();
    l1 = true;
    lv1 = new levOne();
  }
  void display() {

    if (tl) {

      lvt.display();
    } else if (l1) {

      lv1.display();
    }
  }
}


class Platform {
  float x, y, w, h;
  color c1;
  float xVel, yVel;
  float minX, maxX, minY, maxY;
  boolean moving;
  float prevX, prevY;

  Platform(float x, float y, float w, float h, color c1) {
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    this.c1 = c1;
    moving = false;
    prevX = x;
    prevY = y;
  }

  Platform(float x, float y, float w, float h, color c1,
           float xVel, float yVel, float minX, float maxX, float minY, float maxY) {
    this.x = x; this.y = y; this.w = w; this.h = h; this.c1 = c1;
    this.xVel = xVel; this.yVel = yVel; this.minX = minX; this.maxX = maxX; this.minY = minY; this.maxY = maxY;
    this.moving = true;
    prevX = x; prevY = y;
  }

  void update() {
    prevX = x;
    prevY = y;
    if (moving) {
      x += xVel;
      y += yVel;
      if (x < minX || x + w > maxX) xVel *= -1;
      if (y < minY || y + h > maxY) yVel *= -1;
    }
  }

  void display() {
    fill(c1);
    rectMode(CORNER);
    rect(x, y, w, h);
  }

  float getDX() { return x - prevX; }
  float getDY() { return y - prevY; }
}

class Player { //<>//
  float x, y, w, h;
  float xVel, yVel;
  float gravity, jumpStrength;
  boolean moveLeft, moveRight;
  boolean isOnGround, faceRight;
  float ms;
  Gif runGif;
  PImage still;
  Gun gun; // ← gun attached to player

  Player(PApplet app) {
    x = 0;
    y = 0;
    w = 50;
    h = 50;
    ms = 5;
    xVel = 0;
    yVel = 0;
    gravity = 0.8;
    jumpStrength = -15;
    runGif  = new Gif(app, "Runguy.gif");
    runGif.play();
    still = loadImage("Stillguy.png");
    gun = new Gun(this); // create gun bound to player
  }

  void display() {
    pushMatrix();
    imageMode(CENTER);
    if (moveLeft) {
      scale(-1, 1);
      image(runGif, -x, y, w, w);
      faceRight = false;
    } else {
      if (moveRight) {
        image(runGif, x, y, w, w);
        faceRight = true;
      } else {
        if (faceRight) {
          image(still, x, y, w, w);
        } else {
          scale(-1, 1);
          image(still, -x, y, w, w);
        }
      }
    }
     popMatrix();
        // Draw gun AFTER player sprite so it sits on top
        gun.display();
  }


  void update(ArrayList<Platform> platforms) {
    // --- Move with moving platforms BEFORE gravity ---
    if (isOnGround) {
      for (Platform p : platforms) {
        if (collidesWith(p) && p.moving) {
          x += p.getDX();
          y += p.getDY();
        }
      }
    }

    // --- Apply player input ---
    if (moveLeft)  xVel = -ms;
    else if (moveRight) xVel = ms;
    else xVel = 0;

    // --- Horizontal Movement ---
    x += xVel;
    resolveHorizontalCollisions(platforms);

    // --- Apply Gravity ---
    yVel += gravity;
    y += yVel;

    // --- Vertical Movement ---
    resolveVerticalCollisions(platforms);
    


  }

  void resolveHorizontalCollisions(ArrayList<Platform> platforms) {
  for (Platform p : platforms) {

    if (collidesWith(p)) {

      // Amount the player overlaps the platform horizontally
      float overlapLeft  = (p.x + p.w) - (x - w/2);
      float overlapRight = (x + w/2) - p.x;

      // Amount the player overlaps vertically
      float overlapTop    = (y + h/2) - p.y;
      float overlapBottom = (p.y + p.h) - (y - h/2);

      // Only resolve horizontal if the horizontal overlap is smaller
      if (min(overlapLeft, overlapRight) < min(overlapTop, overlapBottom)) {

        if (xVel > 0) {  
          // moving right, hit platform on its left side
          x = p.x - w/2;
        } else if (xVel < 0) {
          // moving left, hit platform on right side
          x = p.x + p.w + w/2;
        }

        xVel = 0;
      }
    }
  }
}


  void resolveVerticalCollisions(ArrayList<Platform> platforms) {
    boolean groundedThisFrame = false;

    for (Platform p : platforms) {
      if (collidesWith(p)) {
        // Falling DOWN onto platform (solid top)
        if (yVel > 0 && (y - h/2) < p.y) {
          y = p.y - h/2;
          yVel = 0;
          groundedThisFrame = true;
          if (p.moving) {
            x += p.getDX();
            y += p.getDY();
          }
        }
        // Jumping UP into platform
        else if (yVel < 0 && (y + h/2) > (p.y + p.h)) {
          y = p.y + p.h + h/2;
          yVel = 0;
        }
      }
    }

    isOnGround = groundedThisFrame;
    if (isOnGround && abs(yVel) < 0.1) yVel = 0;
  }

  boolean collidesWith(Platform p) {
    return (x + w/2 > p.x &&
      x - w/2 < p.x + p.w &&
      y + h/2 > p.y &&
      y - h/2 < p.y + p.h);
  }

  void jump() {
    if (isOnGround) {
      yVel = jumpStrength;
      isOnGround = false;
    }
  }
}

class levOne {
  Carl carl1;

  levOne() {
    p1.x = 0;
    p1.y = 0;
    carl1 = new Carl(300, -200);
    carls.add(carl1);
    // Static platforms
    platforms.add(new Platform(0, 0, 900, 20, color(150, 200, 255))); //blue
    platforms.add(new Platform(900, -80, 200, 100, color(255, 100, 100))); //red
    platforms.add(new Platform(1100, -400, 100, 420, color(150, 100, 100))); //purple
    platforms.add(new Platform(800, -200, 100, 20, color(255, 100, 100))); //red
    platforms.add(new Platform(1000, -300, 100, 20, color(100, 255, 100))); //green






    /*
    // Horizontal moving platform
     platforms.add(new Platform(260, -250, 100, 20, color(100, 255, 100),
     2, 0, 260, 400, 250, 250)); // moves left-right
     
     // Vertical moving platform
     platforms.add(new Platform(50, -200, 80, 20, color(255, 255, 100),
     0, 2, 50, 50, -200, 300)); // moves up-down
     
     */
  }

  void display() {
    carl1.update(p1);
    carl1.display();
  }
}

class levTest {
  levTest() {
    p1.x = 300;
    p1.y = 300;
    // Static platforms---------
    platforms.add(new Platform(0, 480, width, 20, color(150, 200, 255)));
    platforms.add(new Platform(100, 200, 120, 500, color(255, 100, 100)));

    // Horizontal moving platform
    platforms.add(new Platform(260, 250, 100, 20, color(100, 255, 100),
                               2, 0, 260, 400, 250, 250)); // moves left-right

    // Vertical moving platform
    platforms.add(new Platform(50, 200, 80, 20, color(255, 255, 100),
                               0, 2, 50, 50, 100, 300)); // moves up-down
  }

  void display() { }
}
