[Uploading index.html…]()
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>NEON REIGN // ULTRA</title>
<style>
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#020107;color:#fff;font-family:Inter,Arial,sans-serif}
canvas{width:100%;height:100%;display:block}
#hud{position:fixed;inset:0;pointer-events:none}
#top{position:absolute;left:24px;right:24px;top:18px;display:flex;justify-content:space-between;align-items:flex-start}
.brand{font-weight:1000;letter-spacing:7px;font-size:24px}.sub{font-size:10px;letter-spacing:3px;opacity:.7;margin-top:5px}
.stats{text-align:right;font-size:11px;letter-spacing:2px}.stats strong{font-size:34px;letter-spacing:0}
#center{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);text-align:center;opacity:.8}
.cross{width:15px;height:15px;border:1px solid #fff;border-radius:50%;box-shadow:0 0 12px #ff27e8}
#message{margin-top:15px;font-size:12px;font-weight:900;letter-spacing:4px}
#power{position:absolute;left:50%;bottom:73px;transform:translateX(-50%);width:330px;height:10px;border:1px solid #ff39dd;border-radius:20px;background:#120616;box-shadow:0 0 20px #5b17ff55}
#power i{display:block;width:0;height:100%;border-radius:20px;background:linear-gradient(90deg,#5529ff,#ff2bd8,#fff);box-shadow:0 0 20px #ff2bd8}
#bottom{position:absolute;left:24px;bottom:20px;font-size:11px;line-height:1.7;color:#d7d2e4}
#mini{position:absolute;right:24px;bottom:20px;text-align:right;font-size:10px;color:#b8aeca}
kbd{border:1px solid #725080;background:#10091a;border-radius:4px;padding:2px 5px;color:#fff}
#damage{position:absolute;inset:0;border:0 solid #ff159f;transition:.1s}
#flash{position:absolute;inset:0;background:#ff25df;opacity:0;pointer-events:none}
</style>
</head>
<body>
<canvas id="c"></canvas>
<div id="hud">
 <div id="top"><div><div class="brand">NEON REIGN</div><div class="sub">ULTRA CYBER ARCADE // NIGHT CIRCUIT</div></div><div class="stats">SCORE<br><strong id="score">0000</strong><br>STREAK <b id="streak">0</b> · COMBO <b id="combo">x1</b></div></div>
 <div id="center"><div class="cross"></div><div id="message">CLICK TO ENTER</div></div>
 <div id="power"><i></i></div>
 <div id="bottom"><kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd> MOVE &nbsp; <kbd>SPACE</kbd> JUMP &nbsp; <kbd>LMB</kbd> CHARGE / SHOOT &nbsp; <kbd>R</kbd> RESET &nbsp; <kbd>SHIFT</kbd> SPRINT</div>
 <div id="mini">TIP // GREEN ARC = PERFECT RELEASE<br>BACKBOARD BANKS BUILD COMBO</div>
 <div id="damage"></div><div id="flash"></div>
</div>
<script type="module">
import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js";
import {EffectComposer} from "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/postprocessing/EffectComposer.js";
import {RenderPass} from "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/postprocessing/RenderPass.js";
import {UnrealBloomPass} from "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/postprocessing/UnrealBloomPass.js";

const canvas=document.querySelector("#c");
const renderer=new THREE.WebGLRenderer({canvas,antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.setSize(innerWidth,innerHeight);
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.toneMapping=THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure=1.2;

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x020107);
scene.fog=new THREE.FogExp2(0x07030e,.018);

const camera=new THREE.PerspectiveCamera(68,innerWidth/innerHeight,.05,500);
camera.position.set(0,4.6,10);

const composer=new EffectComposer(renderer);
composer.addPass(new RenderPass(scene,camera));
const bloom=new UnrealBloomPass(new THREE.Vector2(innerWidth,innerHeight),1.15,.72,.15);
composer.addPass(bloom);

scene.add(new THREE.HemisphereLight(0x7448ff,0x030107,1.5));
const key=new THREE.DirectionalLight(0xe8d9ff,2.5);
key.position.set(-12,18,10);key.castShadow=true;key.shadow.mapSize.set(2048,2048);scene.add(key);

const glow=(color,intensity,dist,pos)=>{
 const l=new THREE.PointLight(color,intensity,dist);l.position.set(...pos);scene.add(l);return l;
};

const neonMat=(color,intensity=4)=>{
 return new THREE.MeshStandardMaterial({color,emissive:color,emissiveIntensity:intensity,roughness:.25,metalness:.55});
};

function box(w,h,d,x,y,z,color,emit=1){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),neonMat(color,emit));
 m.position.set(x,y,z);m.castShadow=true;m.receiveShadow=true;scene.add(m);return m;
}

const floor=new THREE.Mesh(new THREE.BoxGeometry(24,.4,38),new THREE.MeshStandardMaterial({color:0x0b0817,roughness:.35,metalness:.7}));
floor.position.y=-.25;floor.receiveShadow=true;scene.add(floor);

function line(a,b,color=0xff26dc){
 const g=new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(a[0],.012,a[1]),new THREE.Vector3(b[0],.012,b[1])]);
 scene.add(new THREE.Line(g,new THREE.LineBasicMaterial({color})));
}
line([-11,-18],[11,-18]);line([-11,18],[11,18]);line([-11,-18],[-11,18]);line([11,-18],[11,18]);line([-11,0],[11,0],0x7137ff);
for(let z=-18;z<=18;z+=2)line([-11,z],[11,z],0x3b1b82);

for(let i=-11;i<=11;i+=2){
 box(.07,.9,.07,i,.45,-18,0xff2cd9,6);
 box(.07,.9,.07,i,.45,18,0x6b36ff,6);
}
for(let z=-16;z<=16;z+=2){
 box(.07,.9,.07,-11,.45,z,0x6b36ff,6);
 box(.07,.9,.07,11,.45,z,0xff2cd9,6);
}

const rimMat=neonMat(0xff2adf,7);
const rim=new THREE.Mesh(new THREE.TorusGeometry(1.08,.07,16,64),rimMat);
rim.rotation.x=Math.PI/2;rim.position.set(0,3.35,-15.4);rim.castShadow=true;scene.add(rim);
const board=new THREE.Mesh(new THREE.BoxGeometry(5.4,.16,3.2),new THREE.MeshStandardMaterial({color:0x130d27,emissive:0x33083d,emissiveIntensity:1.5,metalness:.5,roughness:.2,transparent:true,opacity:.9}));
board.position.set(0,4.45,-15.55);scene.add(board);
glow(0xff20d7,10,28,[0,4,-15]);
for(let x=-.9;x<=.9;x+=.3)line([x,-14.9],[x, -15.9],0xb77dff);

function plant(x,z,s){
 const g=new THREE.Group();g.position.set(x,0,z);g.scale.setScalar(s);
 const stemM=new THREE.MeshStandardMaterial({color:0x301a31,emissive:0x100314,emissiveIntensity:2});
 for(let i=0;i<6;i++){
   const stem=new THREE.Mesh(new THREE.CylinderGeometry(.025,.065,2.8,7),stemM);
   stem.position.set((Math.random()-.5)*.7,1.35,(Math.random()-.5)*.4);stem.rotation.z=(Math.random()-.5)*.9;g.add(stem);
   const leaf=new THREE.Mesh(new THREE.CapsuleGeometry(.12,.55,4,8),new THREE.MeshStandardMaterial({color:0x18562d,emissive:0x062a18,emissiveIntensity:2.2}));
   leaf.position.set((Math.random()-.5)*1.0,1.5+Math.random()*1.3,(Math.random()-.5)*.6);
   leaf.rotation.set(Math.random(),Math.random(),(Math.random()-.5)*1.2);g.add(leaf);
 }
 scene.add(g);return g;
}
const plants=[];
for(let i=0;i<36;i++)plants.push(plant((Math.random()<.5?-1:1)*(12+Math.random()*5),(Math.random()-.5)*50,.6+Math.random()*1.5));

for(let i=0;i<28;i++){
 const x=(Math.random()<.5?-1:1)*(15+Math.random()*14),z=-25+Math.random()*55,h=2+Math.random()*10;
 box(1.2+Math.random()*2.5,h,1.2+Math.random()*2.5,x,h/2,z,Math.random()<.5?0x351cff:0xff1acb,.8);
 if(Math.random()<.65)glow(Math.random()<.5?0x6b3cff:0xff22d8,3,18,[x,h*.7,z]);
}

const player=new THREE.Group();scene.add(player);player.position.set(0,0,8);
const suit=neonMat(0x121025,1.4), skin=new THREE.MeshStandardMaterial({color:0x8d604b,roughness:.6});
const torso=new THREE.Mesh(new THREE.CapsuleGeometry(.5,1.25,8,16),suit);torso.position.y=1.45;torso.castShadow=true;player.add(torso);
const head=new THREE.Mesh(new THREE.SphereGeometry(.4,24,20),skin);head.position.y=2.5;head.castShadow=true;player.add(head);
const visor=new THREE.Mesh(new THREE.BoxGeometry(.58,.12,.1),neonMat(0xff2ad9,8));visor.position.set(0,2.52,-.36);player.add(visor);
for(const s of [-1,1]){
 const leg=new THREE.Mesh(new THREE.CylinderGeometry(.16,.2,1,10),suit);leg.position.set(s*.23,.55,0);leg.castShadow=true;player.add(leg);
 const arm=new THREE.Mesh(new THREE.CylinderGeometry(.13,.17,.95,10),suit);arm.position.set(s*.62,1.5,0);arm.rotation.z=s*.3;arm.castShadow=true;player.add(arm);
}

const ball=new THREE.Mesh(new THREE.SphereGeometry(.39,32,24),new THREE.MeshStandardMaterial({color:0xa338ff,emissive:0x5515ff,emissiveIntensity:3,roughness:.22,metalness:.15}));
ball.castShadow=true;scene.add(ball);

const particles=[];
function fx(pos,color,count=35,power=5,life=.7){
 for(let i=0;i<count;i++){
  const m=new THREE.Mesh(new THREE.SphereGeometry(.025+Math.random()*.055,6,6),new THREE.MeshBasicMaterial({color,transparent:true}));
  m.position.copy(pos);scene.add(m);
  particles.push({m,v:new THREE.Vector3((Math.random()-.5)*power,Math.random()*power,(Math.random()-.5)*power),life:life*(.5+Math.random())});
 }
}
function shock(pos,color=0xff28e4){
 const m=new THREE.Mesh(new THREE.TorusGeometry(.15,.035,8,48),new THREE.MeshBasicMaterial({color,transparent:true}));
 m.position.copy(pos);scene.add(m);particles.push({m,v:new THREE.Vector3(),life:.5,ring:true});
}

const keys={};
addEventListener("keydown",e=>{keys[e.key.toLowerCase()]=true;if(e.code==="Space")keys.space=true;if(e.key.toLowerCase()==="r")reset()});
addEventListener("keyup",e=>{keys[e.key.toLowerCase()]=false;if(e.code==="Space")keys.space=false});

let yaw=0,pitch=-.1,charging=false,power=0,free=false,vel=new THREE.Vector3(),time=0;
let score=0,streak=0,combo=1,canScore=true;

canvas.addEventListener("click",()=>canvas.requestPointerLock());
addEventListener("mousemove",e=>{if(document.pointerLockElement===canvas){yaw-=e.movementX*.0023;pitch-=e.movementY*.0023;pitch=THREE.MathUtils.clamp(pitch,-.65,.3)}});
addEventListener("mousedown",e=>{if(e.button===0)charging=true});
addEventListener("mouseup",e=>{if(e.button===0&&charging){shoot();charging=false;power=0}});

function reset(){free=false;vel.set(0,0,0);ball.position.copy(player.position).add(new THREE.Vector3(0,2,-.85).applyAxisAngle(new THREE.Vector3(0,1,0),yaw));document.querySelector("#message").textContent="READY";}
function shoot(){
 const p=10+power*20;
 const d=new THREE.Vector3(Math.sin(yaw)*Math.cos(pitch),-.18-Math.sin(pitch),-Math.cos(yaw)*Math.cos(pitch)).normalize();
 free=true;vel.copy(d.multiplyScalar(p));time=0;canScore=true;
 fx(ball.position,0xa13bff,10,2,.3);
}
function scoreBasket(){
 const pts=2+Math.min(combo,5);
 score+=pts;streak++;combo=Math.min(9,combo+1);
 document.querySelector("#score").textContent=String(score).padStart(4,"0");
 document.querySelector("#streak").textContent=streak;document.querySelector("#combo").textContent="x"+combo;
 document.querySelector("#message").textContent=streak%5===0?"HYPER COMBO // PERFECT":"PURPLE FLAME // +"+pts;
 fx(new THREE.Vector3(0,3.3,-15.4),0xff27df,90,9,1);
 fx(new THREE.Vector3(0,3.3,-15.4),0x762bff,70,7,1);
 shock(new THREE.Vector3(0,3.3,-15.4),0xff2be2);
 document.querySelector("#flash").style.opacity=.12;
 setTimeout(()=>document.querySelector("#flash").style.opacity=0,80);
 canScore=false;
 setTimeout(reset,600);
}

function update(dt){
 let mv=new THREE.Vector3((keys.d?1:0)-(keys.a?1:0),0,(keys.s?1:0)-(keys.w?1:0));
 const sprint=keys.shift?1.65:1;
 if(mv.lengthSq())mv.normalize().multiplyScalar(6.5*sprint*dt).applyAxisAngle(new THREE.Vector3(0,1,0),yaw);
 player.position.add(mv);
 player.position.x=THREE.MathUtils.clamp(player.position.x,-9.3,9.3);player.position.z=THREE.MathUtils.clamp(player.position.z,-11,14);
 player.rotation.y=yaw;
 player.userData.vy=(player.userData.vy||0)-17*dt;
 if(keys.space&&player.position.y<.01)player.userData.vy=6.7;
 player.position.y+=player.userData.vy*dt;
 if(player.position.y<0){player.position.y=0;player.userData.vy=0}
 if(!free)ball.position.copy(player.position).add(new THREE.Vector3(0,2,-.9).applyAxisAngle(new THREE.Vector3(0,1,0),yaw));
 else{
  time+=dt;vel.y-=13*dt;ball.position.addScaledVector(vel,dt);
  ball.rotation.x+=vel.z*dt*2;ball.rotation.z-=vel.x*dt*2;
  if(ball.position.y<.4){
   ball.position.y=.4;vel.y=Math.abs(vel.y)*.68;vel.x*=.84;vel.z*=.84;
   fx(ball.position,0xff27df,35,4,.65);fx(ball.position,0x7e2cff,15,3,.5);shock(ball.position);
   document.querySelector("#message").textContent="NEON THUD // IMPACT";
  }
  if(canScore&&Math.hypot(ball.position.x,ball.position.z+15.4)<1.05&&Math.abs(ball.position.y-3.35)<.55&&vel.z<0)scoreBasket();
  if(ball.position.z<-20||time>5.5){streak=0;combo=1;document.querySelector("#streak").textContent=0;document.querySelector("#combo").textContent="x1";reset();document.querySelector("#message").textContent="MISS // RESET";}
 }
 if(charging){power=Math.min(1,power+dt*.72);document.querySelector("#power i").style.width=(power*100)+"%";document.querySelector("#message").textContent=power>.92?"MAX POWER":"CHARGING "+Math.round(power*100)+"%";}
 for(let i=particles.length-1;i>=0;i--){
  const p=particles[i];p.life-=dt;p.m.position.addScaledVector(p.v,dt);p.v.y-=7*dt;
  if(p.ring)p.m.scale.addScalar(dt*5);
  if(p.life<=0){scene.remove(p.m);particles.splice(i,1)}
 }
 plants.forEach((p,i)=>{p.rotation.y=Math.sin(performance.now()*.0007+i)*.12});
 const target=player.position.clone().add(new THREE.Vector3(0,1.65,0));
 const camPos=player.position.clone().add(new THREE.Vector3(0,4.7,8.5).applyAxisAngle(new THREE.Vector3(0,1,0),yaw));
 camera.position.lerp(camPos,.12);
 camera.lookAt(target.add(new THREE.Vector3(Math.sin(yaw)*2,Math.sin(pitch)*2,-Math.cos(yaw)*2)));
}

reset();
let last=performance.now();
function loop(t){const dt=Math.min(.033,(t-last)/1000);last=t;update(dt);composer.render();requestAnimationFrame(loop)}
loop(last);
addEventListener("resize",()=>{camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();renderer.setSize(innerWidth,innerHeight);composer.setSize(innerWidth,innerHeight)});
</script>
</body></html>
