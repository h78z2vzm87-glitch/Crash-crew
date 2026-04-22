<!DOCTYPE html>
<html>
<head>
  <title>Crash Crew MVP</title>
  <style>
    body { margin: 0; background:#111; color:white; font-family:Arial; }
    canvas { display:block; margin:0 auto; background:#1a1a1a; }
    #ui {
      position: fixed;
      top: 10px;
      left: 10px;
      font-size: 14px;
    }
  </style>
</head>
<body>
<div id="ui">
  <b>Crash Crew MVP</b><br>
  P1: WASD + E (grab)<br>
  P2: Arrow Keys + Shift (grab)<br>
  Deliver box to green zone
</div>

<canvas id="c"></canvas>

<script>
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const keys = {};

window.addEventListener("keydown", e => keys[e.key] = true);
window.addEventListener("keyup", e => keys[e.key] = false);

// Players
function Player(x,y,color,controls){
  return {
    x,y,vx:0,vy:0,color,controls,hasBox:false
  }
}

const p1 = Player(200,200,"#4af", {up:"w",down:"s",left:"a",right:"d",grab:"e"});
const p2 = Player(300,200,"#fa4", {up:"ArrowUp",down:"ArrowDown",left:"ArrowLeft",right:"ArrowRight",grab:"Shift"});

// Box
const box = { x:500,y:300,vx:0,vy:0,held:false };

// Goal
const goal = { x:canvas.width-200, y:canvas.height/2, r:80 };

// Chaos timer
let chaos = 0;

// Helpers
function dist(a,b){
  return Math.hypot(a.x-b.x,a.y-b.y);
}

function updatePlayer(p){
  let speed = 0.6;

  if(keys[p.controls.left]) p.vx -= speed;
  if(keys[p.controls.right]) p.vx += speed;
  if(keys[p.controls.up]) p.vy -= speed;
  if(keys[p.controls.down]) p.vy += speed;

  // grab
  if(keys[p.controls.grab]){
    if(dist(p,box) < 60){
      p.hasBox = true;
      box.held = true;
    }
  } else {
    if(p.hasBox){
      p.hasBox = false;
      box.held = false;
    }
  }

  p.x += p.vx;
  p.y += p.vy;

  p.vx *= 0.9;
  p.vy *= 0.9;

  // bounds
  p.x = Math.max(0,Math.min(canvas.width,p.x));
  p.y = Math.max(0,Math.min(canvas.height,p.y));
}

function updateBox(){
  if(box.held){
    // stick to nearest player
    let holder = dist(p1,box) < dist(p2,box) ? p1 : p2;
    box.x = holder.x;
    box.y = holder.y;
  } else {
    box.vx *= 0.98;
    box.vy *= 0.98;
    box.x += box.vx;
    box.y += box.vy;
  }
}

// CHAOS SYSTEM (this is the fun part)
function chaosSystem(){
  chaos++;

  if(chaos % 120 === 0){
    // random push
    box.vx += (Math.random()-0.5)*10;
    box.vy += (Math.random()-0.5)*10;
  }

  if(chaos % 300 === 0){
    // gravity flip moment
    p1.vy -= 5;
    p2.vy += 5;
  }
}

// Win check
function checkWin(){
  if(dist(box,goal) < goal.r){
    alert("SUCCESS! You delivered it (barely). Refresh to replay.");
    document.location.reload();
  }
}

// Draw
function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);

  // goal
  ctx.fillStyle = "rgba(0,255,0,0.3)";
  ctx.beginPath();
  ctx.arc(goal.x,goal.y,goal.r,0,Math.PI*2);
  ctx.fill();

  // box
  ctx.fillStyle = "#fff";
  ctx.fillRect(box.x-15,box.y-15,30,30);

  // players
  function drawP(p){
    ctx.fillStyle = p.color;
    ctx.beginPath();
    ctx.arc(p.x,p.y,15,0,Math.PI*2);
    ctx.fill();
  }

  drawP(p1);
  drawP(p2);
}

// Loop
function loop(){
  updatePlayer(p1);
  updatePlayer(p2);
  updateBox();
  chaosSystem();
  checkWin();
  draw();
  requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>
