[index.html](https://github.com/user-attachments/files/32050505/index.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>NEON REIGN — Cyber Arcade Basketball</title>
<style>
*{box-sizing:border-box}html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#05030b;color:#fff;font-family:Arial,Helvetica,sans-serif}
#game{display:block;width:100%;height:100%}
#hud{position:fixed;inset:0;pointer-events:none;text-shadow:0 0 12px #ff39dc}
.top{position:absolute;top:18px;left:22px;right:22px;display:flex;justify-content:space-between;align-items:flex-start}
.logo{font-size:25px;font-weight:900;letter-spacing:5px}.score{font-size:22px;font-weight:800}.score b{font-size:38px}
#msg{position:absolute;top:78px;left:50%;transform:translateX(-50%);font-weight:900;letter-spacing:3px;opacity:.9}
#help{position:absolute;bottom:18px;left:18px;max-width:600px;font-size:13px;line-height:1.5;color:#ddd}
#cross{position:absolute;left:50%;top:50%;width:12px;height:12px;transform:translate(-50%,-50%);border:1px solid #fff;border-radius:50%;opacity:.65}
#charge{position:absolute;left:50%;bottom:70px;transform:translateX(-50%);width:260px;height:9px;border:1px solid #ff4ce1;border-radius:10px;overflow:hidden;background:#13071a}
#charge i{display:block;height:100%;width:0;background:linear-gradient(90deg,#7b2cff,#ff35df);box-shadow:0 0 18px #ff35df}
button{pointer-events:auto;background:#13071d;color:#fff;border:1px solid #ff3dde;padding:9px 14px;border-radius:7px;cursor:pointer}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="hud">
  <div class="top"><div class="logo">NEON REIGN</div><div class="score">SCORE <b id="score">0</b> &nbsp; | &nbsp; STREAK <b id="streak">0</b></div></div>
  <div id="msg">READY</div>
  <div id="cross"></div>
  <div id="charge"><i></i></div>
  <div id="help"><b>WASD</b> move · <b>MOUSE</b> aim · <b>HOLD LMB</b> charge shot · <b>RELEASE</b> throw · <b>SPACE</b> jump · <b>R</b> reset ball</div>
</div>

<script type="module">
import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js";
import {OrbitControls} from "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/controls/OrbitControls.js";

const canvas=document.querySelector("#game"), renderer=new THREE.WebGLRenderer({canvas,antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2)); renderer.setSize(innerWidth,innerHeight);
renderer.shadowMap.enabled=true; renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.toneMapping=THREE.ACESFilmicToneMapping; renderer.toneMappingExposure=1.35;

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x03020a);
scene.fog=new THREE.FogExp2(0x080311,.025);

const camera=new THREE.PerspectiveCamera(67,innerWidth/innerHeight,.1,500);
camera.position.set(0,5.4,10);

const hemi=new THREE.HemisphereLight(0x7d4dff,0x050208,1.4); scene.add(hemi);
const moon=new THREE.DirectionalLight(0xd6b7ff,2.3); moon.position.set(-8,16,5); moon.castShadow=true; scene.add(moon);

const neon=(color,intensity=6,dist=25)=>{
 const l=new THREE.PointLight(color,intensity,dist); scene.add(l); return l;
};

const court=new THREE.Mesh(new THREE.BoxGeometry(22,.35,34),new THREE.MeshStandardMaterial({color:0x0c0920,roughness:.42,metalness:.5}));
court.position.y=-.2; court.receiveShadow=true; scene.add(court);

function line(x1,z1,x2,z2,color=0xff32e6){
 const pts=[new THREE.Vector3(x1,.012,z1),new THREE.Vector3(x2,.012,z2)];
 const g=new THREE.BufferGeometry().setFromPoints(pts);
 const m=new THREE.LineBasicMaterial({color,transparent:true,opacity:.9});
 scene.add(new THREE.Line(g,m));
}
line(-10,-16,10,-16);line(-10,16,10,16);line(-10,-16,-10,16);line(10,-16,10,16);
line(-10,0,10,0); line(-5,-16,-5,-11);line(5,-16,5,-11);
for(let z=-16;z<=16;z+=2) line(-10,z,10,z,0x4528ff);

function glowBox(w,h,d,pos,color){
 const mat=new THREE.MeshStandardMaterial({color,emissive:color,emissiveIntensity:3,roughness:.25,metalness:.5});
 const o=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat);o.position.set(...pos);o.castShadow=true;scene.add(o);return o;
}
for(let x=-11;x<=11;x+=2){glowBox(.08,1.2,.08,[x,.55,-17],0xff26d9);glowBox(.08,1.2,.08,[x,.55,17],0x6d3cff)}
for(let z=-15;z<=15;z+=2){glowBox(.08,1.2,.08,[-11,.55,z],0x5b34ff);glowBox(.08,1.2,.08,[11,.55,z],0xff26d9)}

const hoopGroup=new THREE.Group(); scene.add(hoopGroup);
const rimMat=new THREE.MeshStandardMaterial({color:0xff39e6,emissive:0xff39e6,emissiveIntensity:5,metalness:.6,roughness:.2});
const rim=new THREE.Mesh(new THREE.TorusGeometry(1.05,.075,16,64),rimMat);
rim.rotation.x=Math.PI/2; rim.position.set(0,3.35,-14); hoopGroup.add(rim);
const board=new THREE.Mesh(new THREE.BoxGeometry(5,.16,3.1),new THREE.MeshStandardMaterial({color:0x17102c,emissive:0x2a0b3d,emissiveIntensity:1.8,transparent:true,opacity:.92}));
board.position.set(0,4.45,-14.15); hoopGroup.add(board);
const backLight=neon(0xff28db,7,20); backLight.position.set(0,4,-14);
for(let i=0;i<13;i++){
 const rope=new THREE.Mesh(new THREE.CylinderGeometry(.012,.012,1.7,6),new THREE.MeshBasicMaterial({color:0xf6d6ff,transparent:true,opacity:.65}));
 rope.position.set((i-6)*.15,2.55,-14); hoopGroup.add(rope);
}

const ball=new THREE.Mesh(new THREE.SphereGeometry(.38,24,24),new THREE.MeshStandardMaterial({color:0x9b35ff,emissive:0x3c0bff,emissiveIntensity:2,roughness:.28}));
ball.castShadow=true;scene.add(ball);

function makePlayer(){
 const g=new THREE.Group();
 const suit=new THREE.MeshStandardMaterial({color:0x100b25,emissive:0x38104f,emissiveIntensity:1.3,metalness:.45});
 const skin=new THREE.MeshStandardMaterial({color:0x8b5a46,roughness:.65});
 const body=new THREE.Mesh(new THREE.CapsuleGeometry(.48,1.25,8,16),suit);body.position.y=1.45;body.castShadow=true;g.add(body);
 const head=new THREE.Mesh(new THREE.SphereGeometry(.39,20,20),skin);head.position.y=2.45;head.castShadow=true;g.add(head);
 const visor=new THREE.Mesh(new THREE.BoxGeometry(.52,.13,.12),new THREE.MeshStandardMaterial({color:0xff36df,emissive:0xff36df,emissiveIntensity:5}));
 visor.position.set(0,2.48,-.35);g.add(visor);
 for(const s of [-1,1]){
   const leg=new THREE.Mesh(new THREE.CylinderGeometry(.16,.19,.95,10),suit);leg.position.set(s*.22,.55,0);leg.castShadow=true;g.add(leg);
   const arm=new THREE.Mesh(new THREE.CylinderGeometry(.13,.16,.9,10),suit);arm.position.set(s*.62,1.55,0);arm.rotation.z=s*.28;arm.castShadow=true;g.add(arm);
 }
 return g;
}
const player=makePlayer(); player.position.set(0,0,8); scene.add(player);

const particles=[];
function burst(pos,color=0xff3de0,count=32,speed=4){
 for(let i=0;i<count;i++){
   const p=new THREE.Mesh(new THREE.SphereGeometry(.035+Math.random()*.045,6,6),new THREE.MeshBasicMaterial({color,transparent:true}));
   p.position.copy(pos);scene.add(p);
   const v=new THREE.Vector3((Math.random()-.5)*speed,Math.random()*speed*.9,(Math.random()-.5)*speed);
   particles.push({m:p,v,life:.5+Math.random()*.5});
 }
}
function ring(pos,color){
 const r=new THREE.Mesh(new THREE.TorusGeometry(.12,.035,8,32),new THREE.MeshBasicMaterial({color,transparent:true}));
 r.position.copy(pos);scene.add(r);particles.push({m:r,v:new THREE.Vector3(),life:.45,ring:true});
}

const plants=[];
function plant(x,z,scale=1){
 const g=new THREE.Group();g.position.set(x,0,z);g.scale.setScalar(scale);
 const stemMat=new THREE.MeshStandardMaterial({color:0x321f35,emissive:0x16051c});
 for(let i=0;i<5;i++){
   const stem=new THREE.Mesh(new THREE.CylinderGeometry(.025,.06,2.2,7),stemMat);
   stem.position.y=1.1;stem.rotation.z=(Math.random()-.5)*.7;g.add(stem);
   const leaf=new THREE.Mesh(new THREE.ConeGeometry(.18,.7,7),new THREE.MeshStandardMaterial({color:0x1d542d,emissive:0x062d1a,emissiveIntensity:2}));
   leaf.position.set((Math.random()-.5)*.7,1.6+Math.random(),(Math.random()-.5)*.4);leaf.rotation.z=(Math.random()-.5);g.add(leaf);
 }
 scene.add(g);plants.push(g);
}
for(let i=0;i<25;i++){const side=Math.random()<.5?-1:1;plant(side*(12+Math.random()*5),(Math.random()-.5)*38,.7+Math.random()*1.3)}

for(let i=0;i<18;i++){
 const h=2+Math.random()*7, x=(Math.random()<.5?-1:1)*(14+Math.random()*10),z=-24+Math.random()*48;
 glowBox(1.5+Math.random()*2,h,1.5,[x,h/2,z],Math.random()<.5?0x3217ff:0xff1bc9);
}
for(let i=0;i<12;i++){
 const l=neon(Math.random()<.5?0xff2ad9:0x5c3cff,3+Math.random()*4,20);
 l.position.set((Math.random()-.5)*32,4+Math.random()*8,(Math.random()-.5)*50);
}

const keys={}; addEventListener("keydown",e=>{keys[e.key.toLowerCase()]=true;if(e.code==="Space")keys.space=true;if(e.key.toLowerCase()==="r")resetBall()});
addEventListener("keyup",e=>{keys[e.key.toLowerCase()]=false;if(e.code==="Space")keys.space=false});
let yaw=0,pitch=-.12,charging=false,charge=0,score=0,streak=0;
let ballVel=new THREE.Vector3(),ballFree=false,shotTime=0;
addEventListener("mousemove",e=>{if(document.pointerLockElement===canvas){yaw-=e.movementX*.0025;pitch-=e.movementY*.0025;pitch=Math.max(-.7,Math.min(.35,pitch));}});
canvas.addEventListener("click",()=>canvas.requestPointerLock());
addEventListener("mousedown",e=>{if(e.button===0)charging=true});
addEventListener("mouseup",e=>{if(e.button===0&&charging){throwBall();charging=false;charge=0}});

function resetBall(){ballFree=false;ballVel.set(0,0,0);ball.position.copy(player.position).add(new THREE.Vector3(0,2.1,-.8));}
function throwBall(){
 const power=10+charge*19;
 const dir=new THREE.Vector3(Math.sin(yaw)*Math.cos(pitch),-Math.sin(pitch)+.22,-Math.cos(yaw)*Math.cos(pitch)).normalize();
 ballFree=true;ballVel.copy(dir.multiplyScalar(power));shotTime=0;
}
function scored(){
 score+=2+Math.min(streak,3);streak++;document.querySelector("#score").textContent=score;document.querySelector("#streak").textContent=streak;
 document.querySelector("#msg").textContent="PURPLE FLAME • "+(2+Math.min(streak,3))+" POINTS";
 burst(new THREE.Vector3(0,3.35,-14),0xff2be5,70,8); burst(new THREE.Vector3(0,3.35,-14),0x762cff,45,6);ring(new THREE.Vector3(0,3.35,-14),0xff3de0);
 setTimeout(()=>{document.querySelector("#msg").textContent="KEEP THE STREAK";resetBall()},650);
}
function update(dt){
 const move=new THREE.Vector3((keys.d?1:0)-(keys.a?1:0),0,(keys.s?1:0)-(keys.w?1:0));
 if(move.lengthSq())move.normalize().multiplyScalar(6*dt).applyAxisAngle(new THREE.Vector3(0,1,0),yaw);
 player.position.add(move);player.position.x=THREE.MathUtils.clamp(player.position.x,-9.2,9.2);player.position.z=THREE.MathUtils.clamp(player.position.z,-12,14);
 player.rotation.y=yaw;
 if(keys.space&&player.position.y<=.01)player.userData.vy=6.3;
 player.userData.vy=(player.userData.vy||0)-16*dt;player.position.y+=player.userData.vy*dt;if(player.position.y<0){player.position.y=0;player.userData.vy=0}
 if(!ballFree)ball.position.copy(player.position).add(new THREE.Vector3(0,2.0,-.9).applyAxisAngle(new THREE.Vector3(0,1,0),yaw));
 else{
   shotTime+=dt;ballVel.y-=12*dt;ball.position.addScaledVector(ballVel,dt);
   if(ball.position.y<.38){ball.position.y=.38;ballVel.y=Math.abs(ballVel.y)*.68;ballVel.x*=.82;ballVel.z*=.82;burst(ball.position,0xff37df,22,4);ring(ball.position,0xff42e5);document.querySelector("#msg").textContent="THUD // NEON IMPACT";}
   const dx=ball.position.x, dz=ball.position.z+14, dy=ball.position.y-3.35;
   if(Math.hypot(dx,dz)<1.05 && Math.abs(dy)<.55 && ballVel.z<0){scored();return}
   if(ball.position.z<-18||shotTime>5){streak=0;document.querySelector("#streak").textContent=0;resetBall();document.querySelector("#msg").textContent="MISS — RESET";}
 }
 if(charging){charge=Math.min(1,charge+dt*.75);document.querySelector("#charge i").style.width=(charge*100)+"%";document.querySelector("#msg").textContent=charge>.9?"MAX POWER":"CHARGING "+Math.round(charge*100)+"%";}
 for(let i=particles.length-1;i>=0;i--){const p=particles[i];p.life-=dt;p.m.position.addScaledVector(p.v,dt);p.v.y-=7*dt;if(p.ring){p.m.scale.addScalar(dt*4)}if(p.life<=0){scene.remove(p.m);particles.splice(i,1)}}
 plants.forEach((p,i)=>p.rotation.y=Math.sin(performance.now()*.001+i)*.08);
 const camTarget=player.position.clone().add(new THREE.Vector3(0,3.0,7).applyAxisAngle(new THREE.Vector3(0,1,0),yaw));
 camera.position.lerp(player.position.clone().add(new THREE.Vector3(0,4.4,8).applyAxisAngle(new THREE.Vector3(0,1,0),yaw)),.12);
 camera.lookAt(player.position.clone().add(new THREE.Vector3(0,1.7,0)).add(new THREE.Vector3(Math.sin(yaw)*2,Math.sin(pitch)*2,-Math.cos(yaw)*2)));
}
resetBall();
let last=performance.now();
function loop(t){const dt=Math.min(.033,(t-last)/1000);last=t;update(dt);renderer.render(scene,camera);requestAnimationFrame(loop)}
loop(last);
addEventListener("resize",()=>{camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();renderer.setSize(innerWidth,innerHeight)});
</script>
</body>
</html>
