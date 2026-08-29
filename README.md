<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width,initial-scale=1.0,user-scalable=no">

<title>Rail Rush 3D</title>

<style>
*{
  box-sizing:border-box;
}

html,body{
  margin:0;
  width:100%;
  height:100%;
  overflow:hidden;
  background:#87ceeb;
  touch-action:none;
  font-family:Arial,sans-serif;
}

#game{
  width:100vw;
  height:100vh;
}

#hud{
  position:fixed;
  top:15px;
  left:15px;
  right:15px;
  z-index:10;
  display:flex;
  justify-content:space-between;
  color:white;
  font-size:18px;
  font-weight:bold;
  text-shadow:2px 2px 4px #000;
  pointer-events:none;
}

#menu{
  position:fixed;
  inset:0;
  z-index:20;
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
  background:rgba(0,0,0,.65);
  color:white;
  text-align:center;
}

#menu h1{
  font-size:40px;
  margin:0 0 10px;
}

#menu p{
  line-height:1.6;
}

#play{
  border:0;
  border-radius:15px;
  padding:15px 45px;
  font-size:21px;
  font-weight:bold;
  color:white;
  background:#16c172;
}
</style>
</head>

<body>

<div id="game"></div>

<div id="hud">
  <span id="score">Score: 0</span>
  <span id="speed">Speed: 1x</span>
</div>

<div id="menu">
  <h1>🚆 RAIL RUSH 3D</h1>

  <p>
    Swipe LEFT / RIGHT to change track<br>
    Swipe UP to jump<br>
    Swipe DOWN to slide
  </p>

  <button id="play">PLAY</button>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>

let scene;
let camera;
let renderer;
let player;

let playing=false;
let lane=1;

let score=0;
let speed=0.35;

let jumping=false;
let sliding=false;

const tracks=[-3,0,3];

let startX=0;
let startY=0;


/* =========================
   SCENE
========================= */

function init(){

  scene=new THREE.Scene();

  scene.background=
    new THREE.Color(0x87ceeb);

  scene.fog=
    new THREE.Fog(
      0x87ceeb,
      30,
      180
    );


  camera=
    new THREE.PerspectiveCamera(
      65,
      innerWidth/innerHeight,
      .1,
      500
    );

  camera.position.set(
    0,
    4.5,
    10
  );


  renderer=
    new THREE.WebGLRenderer({
      antialias:true
    });

  renderer.setSize(
    innerWidth,
    innerHeight
  );

  renderer.setPixelRatio(
    Math.min(devicePixelRatio,2)
  );

  document
    .getElementById("game")
    .appendChild(renderer.domElement);


  /* LIGHT */

  const light=
    new THREE.HemisphereLight(
      0xffffff,
      0x444444,
      1.7
    );

  scene.add(light);


  const sun=
    new THREE.DirectionalLight(
      0xffffff,
      1.2
    );

  sun.position.set(
    10,
    20,
    10
  );

  scene.add(sun);


  createGround();
  createTracks();
  createPlayer();

  animate();
}


/* =========================
   GROUND
========================= */

function createGround(){

  const ground=
    new THREE.Mesh(
      new THREE.BoxGeometry(
        50,
        .2,
        300
      ),
      new THREE.MeshStandardMaterial({
        color:0x4ca83f
      })
    );

  ground.position.set(
    0,
    -.3,
    -130
  );

  scene.add(ground);
}


/* =========================
   THREE RAIL TRACKS
========================= */

function createTracks(){

  for(
    let x of tracks
  ){

    /* sleepers */

    for(
      let z=5;
      z>-280;
      z-=3
    ){

      const sleeper=
        new THREE.Mesh(
          new THREE.BoxGeometry(
            2.1,
            .15,
            .35
          ),
          new THREE.MeshStandardMaterial({
            color:0x5b3a29
          })
        );

      sleeper.position.set(
        x,
        0,
        z
      );

      scene.add(sleeper);
    }


    /* rails */

    for(
      let side of [-.65,.65]
    ){

      const rail=
        new THREE.Mesh(
          new THREE.BoxGeometry(
            .12,
            .12,
            290
          ),
          new THREE.MeshStandardMaterial({
            color:0xaaaaaa,
            metalness:.7
          })
        );

      rail.position.set(
        x+side,
        .15,
        -135
      );

      scene.add(rail);
    }
  }
}


/* =========================
   PLAYER
========================= */

function createPlayer(){

  player=
    new THREE.Group();


  /* BODY */

  const body=
    new THREE.Mesh(
      new THREE.BoxGeometry(
        1.05,
        1.5,
        .7
      ),
      new THREE.MeshStandardMaterial({
        color:0x1976d2
      })
    );

  body.position.y=2.1;

  player.add(body);


  /* HEAD */

  const head=
    new THREE.Mesh(
      new THREE.SphereGeometry(
        .48,
        20,
        20
      ),
      new THREE.MeshStandardMaterial({
        color:0xffc49a
      })
    );

  head.position.y=3.35;

  player.add(head);


  /* HAIR */

  const hair=
    new THREE.Mesh(
      new THREE.SphereGeometry(
        .5,
        16,
        16
      ),
      new THREE.MeshStandardMaterial({
        color:0x222222
      })
    );

  hair.scale.y=.55;
  hair.position.y=3.68;

  player.add(hair);


  /* ARMS */

  player.leftArm=
    limb(
      .25,
      1.3,
      0xffc49a
    );

  player.rightArm=
    limb(
      .25,
      1.3,
      0xffc49a
    );

  player.leftArm.position.set(
    -.72,2.15,0
  );

  player.rightArm.position.set(
    .72,2.15,0
  );

  player.add(player.leftArm);
  player.add(player.rightArm);


  /* LEGS */

  player.leftLeg=
    limb(
      .3,
      1.35,
      0x222222
    );

  player.rightLeg=
    limb(
      .3,
      1.35,
      0x222222
    );

  player.leftLeg.position.set(
    -.3,.65,0
  );

  player.rightLeg.position.set(
    .3,.65,0
  );

  player.add(player.leftLeg);
  player.add(player.rightLeg);


  player.position.set(
    0,
    0,
    4
  );

  scene.add(player);
}


function limb(
  width,
  height,
  color
){

  return new THREE.Mesh(
    new THREE.BoxGeometry(
      width,
      height,
      width
    ),
    new THREE.MeshStandardMaterial({
      color:color
    })
  );
}


/* =========================
   START
========================= */

function startGame(){

  playing=true;

  score=0;
  speed=.35;

  lane=1;

  player.position.x=0;
  player.position.y=0;

  document
    .getElementById("menu")
    .style.display="none";
}


/* =========================
   LEFT / RIGHT
========================= */

function left(){

  if(!playing)return;

  if(lane>0){

    lane--;

    player.position.x=
      tracks[lane];
  }
}


function right(){

  if(!playing)return;

  if(lane<2){

    lane++;

    player.position.x=
      tracks[lane];
  }
}


/* =========================
   JUMP
========================= */

function jump(){

  if(
    !playing ||
    jumping ||
    sliding
  )return;

  jumping=true;

  const start=
    performance.now();

  function update(){

    const t=
      Math.min(
        (performance.now()-start)/700,
        1
      );

    player.position.y=
      Math.sin(t*Math.PI)*3;

    if(t<1){

      requestAnimationFrame(update);

    }else{

      player.position.y=0;
      jumping=false;
    }
  }

  update();
}


/* =========================
   SLIDE
========================= */

function slide(){

  if(
    !playing ||
    sliding ||
    jumping
  )return;

  sliding=true;

  player.scale.y=.5;

  setTimeout(
    ()=>{
      player.scale.y=1;
      sliding=false;
    },
    650
  );
}


/* =========================
   SWIPE
========================= */

document.addEventListener(
  "touchstart",
  e=>{

    startX=
      e.touches[0].clientX;

    startY=
      e.touches[0].clientY;
  },
  {passive:false}
);


document.addEventListener(
  "touchend",
  e=>{

    e.preventDefault();

    const endX=
      e.changedTouches[0].clientX;

    const endY=
      e.changedTouches[0].clientY;

    const dx=
      endX-startX;

    const dy=
      endY-startY;


    if(
      Math.abs(dx)>
      Math.abs(dy)
    ){

      if(dx>45)right();

      if(dx<-45)left();

    }else{

      if(dy<-45)jump();

      if(dy>45)slide();
    }

  },
  {passive:false}
);


/* =========================
   KEYBOARD
========================= */

document.addEventListener(
  "keydown",
  e=>{

    if(e.key==="ArrowLeft")
      left();

    if(e.key==="ArrowRight")
      right();

    if(e.key==="ArrowUp")
      jump();

    if(e.key==="ArrowDown")
      slide();
  }
);


/* =========================
   PLAY BUTTON
========================= */

document
  .getElementById("play")
  .addEventListener(
    "click",
    startGame
  );


/* =========================
   GAME LOOP
========================= */

let last=
  performance.now();


function animate(){

  requestAnimationFrame(
    animate
  );

  const now=
    performance.now();

  const delta=
    (now-last)/1000;

  last=now;


  if(playing){

    score+=
      delta*8;


    speed=
      .35+
      Math.floor(score/100)*.025;


    /* running animation */

    if(
      !jumping &&
      !sliding
    ){

      const t=
        now*.012;

      player.leftLeg.rotation.x=
        Math.sin(t)*.8;

      player.rightLeg.rotation.x=
        Math.sin(
          t+Math.PI
        )*.8;

      player.leftArm.rotation.x=
        Math.sin(
          t+Math.PI
        )*.5;

      player.rightArm.rotation.x=
        Math.sin(t)*.5;
    }


    document
      .getElementById("score")
      .innerText=
      "Score: "+
      Math.floor(score);


    document
      .getElementById("speed")
      .innerText=
      "Speed: "+
      (
        1+
        Math.floor(score/100)
      )+
      "x";


    /* CAMERA */

    camera.position.x +=
      (
        player.position.x-
        camera.position.x
      )*.08;

    camera.position.y=
      4.5+
      player.position.y*.15;

    camera.lookAt(
      player.position.x,
      1.8,
      -12
    );
  }


  renderer.render(
    scene,
    camera
  );
}


/* =========================
   RESIZE
========================= */

window.addEventListener(
  "resize",
  ()=>{

    camera.aspect=
      innerWidth/
      innerHeight;

    camera.updateProjectionMatrix();

    renderer.setSize(
      innerWidth,
      innerHeight
    );
  }
);


init();

</script>

</body>
</html>
