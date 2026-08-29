<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no">
<title>3D Runner</title>

<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#87ceeb;font-family:Arial}
#game{width:100vw;height:100vh;position:relative}
#score{
 position:absolute;top:15px;left:15px;z-index:10;
 color:white;font-size:22px;font-weight:bold;
 text-shadow:2px 2px 3px #000
}
#buttons{
 position:absolute;bottom:20px;width:100%;z-index:10;
 display:flex;justify-content:center;gap:12px
}
button{
 width:70px;height:60px;border:0;border-radius:15px;
 font-size:25px;background:rgba(255,255,255,.85)
}
#start{
 position:absolute;inset:0;z-index:20;
 background:rgba(0,0,0,.65);color:white;
 display:flex;flex-direction:column;
 justify-content:center;align-items:center
}
#start h1{font-size:40px}
#start button{
 width:180px;background:#20c957;color:white;
 font-size:20px
}
</style>
</head>

<body>

<div id="game"></div>

<div id="score">Score: 0</div>

<div id="buttons">
<button onclick="moveLeft()">⬅️</button>
<button onclick="jump()">⬆️</button>
<button onclick="moveRight()">➡️</button>
</div>

<div id="start">
<h1>🏃 3D RUNNER</h1>
<p>Swipe ya buttons se move karein</p>
<button onclick="startGame()">START</button>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>

let scene,camera,renderer;
let player;
let lane=0;
let jumping=false;
let playing=false;
let score=0;
let roadLines=[];

const lanes=[-3,0,3];

function createGame(){

scene=new THREE.Scene();

scene.background=new THREE.Color(0x87ceeb);

camera=new THREE.PerspectiveCamera(
60,
innerWidth/innerHeight,
0.1,
1000
);

camera.position.set(0,4,8);

renderer=new THREE.WebGLRenderer({
antialias:true
});

renderer.setSize(innerWidth,innerHeight);

document.getElementById("game")
.appendChild(renderer.domElement);

/* Light */

const light=new THREE.HemisphereLight(
0xffffff,
0x555555,
1.5
);

scene.add(light);

/* Road */

const roadGeometry=
new THREE.BoxGeometry(12,.2,200);

const roadMaterial=
new THREE.MeshStandardMaterial({
color:0x333333
});

const road=
new THREE.Mesh(
roadGeometry,
roadMaterial
);

road.position.z=-70;

scene.add(road);

/* Lane lines */

for(let x of [-2,2]){

for(let z=0;z>-180;z-=8){

const geo=
new THREE.BoxGeometry(.12,.03,4);

const mat=
new THREE.MeshBasicMaterial({
color:0xffffff
});

const line=
new THREE.Mesh(geo,mat);

line.position.set(x,.13,z);

scene.add(line);

roadLines.push(line);

}
}

/* Player */

const body=
new THREE.Mesh(
new THREE.BoxGeometry(1.2,2,1),
new THREE.MeshStandardMaterial({
color:0xe63946
})
);

body.position.set(0,1.1,3);

scene.add(body);

player=body;

animate();
}

function moveLeft(){

if(!playing)return;

if(lane>-1){
lane--;
player.position.x=lanes[lane+1];
}
}

function moveRight(){

if(!playing)return;

if(lane<1){
lane++;
player.position.x=lanes[lane+1];
}
}

function jump(){

if(!playing||jumping)return;

jumping=true;

let startY=1.1;
let t=0;

function jumpAnimation(){

t+=0.08;

player.position.y=
startY+Math.sin(t*Math.PI)*3;

if(t<1){

requestAnimationFrame(jumpAnimation);

}else{

player.position.y=startY;
jumping=false;

}

}

jumpAnimation();
}

function startGame(){

playing=true;
score=0;

document.getElementById("score")
.innerText="Score: 0";

document.getElementById("start")
.style.display="none";
}

function animate(){

requestAnimationFrame(animate);

if(playing){

score+=0.05;

document.getElementById("score")
.innerText=
"Score: "+Math.floor(score);

/* Moving road */

for(let line of roadLines){

line.position.z+=0.5;

if(line.position.z>10){
line.position.z=-180;
}

}

}

camera.lookAt(
player.position.x,
1,
-10
);

renderer.render(scene,camera);
}

window.addEventListener(
"resize",
()=>{
camera.aspect=innerWidth/innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(innerWidth,innerHeight);
}
);

let sx=0;

document.addEventListener(
"touchstart",
e=>{
sx=e.touches[0].clientX;
}
);

document.addEventListener(
"touchend",
e=>{

let ex=e.changedTouches[0].clientX;
let dx=ex-sx;

if(Math.abs(dx)>50){

if(dx>0)moveRight();
else moveLeft();

}

}
);

document.addEventListener(
"keydown",
e=>{

if(e.key==="ArrowLeft")moveLeft();
if(e.key==="ArrowRight")moveRight();
if(e.key==="ArrowUp")jump();

}
);

createGame();

</script>

</body>
</html>
