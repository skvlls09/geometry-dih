<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GD – Auto-Restart</title>
  <style>
    *{box-sizing:border-box;margin:0;padding:0}
    body{
      height:100vh;
      display:flex;
      justify-content:center;
      align-items:center;
      background:linear-gradient(135deg,#0f0c29,#302b63,#24243e);
      font-family:system-ui, sans-serif;
      overflow:hidden;
    }
    #c{
      width:900px;
      max-width:95vw;
      border-radius:12px;
      box-shadow:0 0 30px #00ffffaa, inset 0 0 20px #0006;
      image-rendering: crisp-edges;
      transition:transform .1s, filter .3s;
    }
    #hint{
      position:absolute;
      bottom:15px;
      left:50%;
      transform:translateX(-50%);
      color:#fff8;
      font-size:14px;
      pointer-events:none;
    }
  </style>
</head>
<body>
<canvas id="c" width="900" height="450"></canvas>
<div id="hint">Click / Space / Tap to jump</div>

<script>
const c=document.getElementById('c'), ctx=c.getContext('2d');
ctx.imageSmoothingEnabled=false;

let gr=.42, jmp=-11.5, baseSpeed=5.8, speed=baseSpeed, obs=[], frame=0, dead=false, shake=0, score=0, deathFrame=0,
    stars=Array.from({length:60},()=>({x:Math.random()*c.width,y:Math.random()*c.height,s:Math.random()*2+1,a:Math.random()})),
    p={x:120,y:260,w:34,h:34,v:0,g:false,rot:0};

function reset(){
  dead=false; deathFrame=0; frame=0; score=0; speed=baseSpeed; obs=[]; shake=0;
  p={x:120,y:260,w:34,h:34,v:0,g:false,rot:0};
  c.style.filter='none';
}

function draw(){
  ctx.fillStyle='#0b0b2b'; ctx.fillRect(0,0,c.width,c.height);
  stars.forEach(s=>{s.x-=s.s*0.4; if(s.x<0)s.x=c.width; ctx.globalAlpha=s.a; ctx.fillStyle='#fff'; ctx.fillRect(s.x,s.y,s.s,s.s);});
  ctx.globalAlpha=1;

  const groundY=c.height-60;
  ctx.fillStyle='#1a1a3a'; ctx.fillRect(0,groundY,c.width,c.height-groundY);
  for(let i=0;i<c.width;i+=60){
    ctx.strokeStyle='#00ffff22'; ctx.lineWidth=2;
    ctx.beginPath(); ctx.moveTo(i-frame*speed%60,groundY); ctx.lineTo(i-frame*speed%60+40,groundY+20); ctx.stroke();
  }

  /* player */
  if(!p.g){p.rot+=0.12;}else{p.rot=0;}
  ctx.save();
  ctx.translate(p.x+p.w/2,p.y+p.h/2);
  ctx.rotate(p.rot);
  const grad=ctx.createLinearGradient(-p.w/2,-p.h/2,p.w/2,p.h/2);
  grad.addColorStop(0,'#00ffff'); grad.addColorStop(1,'#0088cc');
  ctx.fillStyle=grad;
  roundRect(ctx,-p.w/2,-p.h/2,p.w,p.h,8);
  ctx.shadowColor='#00ffff'; ctx.shadowBlur=15;
  ctx.fill(); ctx.shadowBlur=0;
  ctx.fillStyle='#000';
  ctx.beginPath(); ctx.arc(-8,-6,3,0,Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.arc(8,-6,3,0,Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.arc(0,6,4,0,Math.PI); ctx.fill();
  ctx.restore();

  /* spikes */
  ctx.fillStyle='#ff2a6d';
  obs.forEach(o=>{
    ctx.beginPath();
    ctx.moveTo(o.x,o.y+o.h);
    ctx.lineTo(o.x+o.w/2,o.y);
    ctx.lineTo(o.x+o.w,o.y+o.h);
    ctx.closePath();
    ctx.shadowColor='#ff2a6d88'; ctx.shadowBlur=20;
    ctx.fill(); ctx.shadowBlur=0;
  });

  /* score */
  ctx.fillStyle='#fff'; ctx.font='bold 28px ui-sans-serif'; ctx.textAlign='left';
  ctx.fillText('Score: '+score,25,40);

  if(deathFrame>0){
    ctx.fillStyle=`rgba(0,0,0,${Math.min(0.6,deathFrame/40)})`;
    ctx.fillRect(0,0,c.width,c.height);
  }
  ctx.textAlign='left';
}

function roundRect(ctx,x,y,w,h,r){
  ctx.beginPath();
  ctx.moveTo(x+r,y); ctx.lineTo(x+w-r,y); ctx.quadraticCurveTo(x+w,y,x+w,y+r);
  ctx.lineTo(x+w,y+h-r); ctx.quadraticCurveTo(x+w,y+h,x+w-r,y+h);
  ctx.lineTo(x+r,y+h); ctx.quadraticCurveTo(x,y+h,x,y+h-r);
  ctx.lineTo(x,y+r); ctx.quadraticCurveTo(x,y,x+r,y);
  ctx.closePath();
}

function update(){
  if(deathFrame>0){
    deathFrame++;
    shake*=0.9;
    c.style.transform=`translate(${(Math.random()-0.5)*shake}px,${(Math.random()-0.5)*shake}px)`;
    c.style.filter=`blur(${Math.min(4,deathFrame/15)}px)`;
    if(deathFrame===60)reset();
    return;
  }
  frame++;
  score=Math.floor(frame/10);
  speed=baseSpeed+Math.floor(frame/600)*0.8;

  if(!p.g)p.v+=gr;
  p.y+=p.v;
  p.g=(p.y+p.h>=c.height-60);
  if(p.g){p.y=c.height-60-p.h;p.v=0;}

  if(frame%100==0){
    const h=48+Math.random()*20;
    obs.push({x:c.width,y:c.height-60-h,w:32,h:h});
  }
  obs.forEach(o=>o.x-=speed);
  obs=obs.filter(o=>o.x+o.w>0);
  obs.forEach(o=>{
    if(p.x<o.x+o.w&&p.x+p.w>o.x&&p.y<o.y+o.h&&p.y+p.h>o.y){dead=true;shake=14;deathFrame=1;}
  });
}

function loop(){update();draw();requestAnimationFrame(loop);}
function jump(){if(p.g&&!deathFrame){p.v=jmp;p.rot=0;}}
c.addEventListener('click',jump);
document.addEventListener('keydown',e=>{if(e.code==='Space'){e.preventDefault();jump();}});
loop();
</script>
</body>
</html>
