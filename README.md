<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>My 3D Runner</title>

  <style>
    html,body{
      margin:0;
      width:100%;
      height:100%;
      overflow:hidden;
      background:#65b5e8;
      font-family:Arial;
    }

    #game{
      width:100%;
      height:100%;
      display:flex;
      align-items:center;
      justify-content:center;
      flex-direction:column;
      color:white;
    }

    #player{
      width:55px;
      height:110px;
      background:#1976d2;
      border-radius:20px;
      position:absolute;
      bottom:150px;
      left:calc(50% - 27px);
      transition:left .2s;
    }

    #head{
      width:45px;
      height:45px;
      background:#ffc49a;
      border-radius:50%;
      position:absolute;
      top:-45px;
      left:5px;
    }

    #road{
      position:absolute;
      bottom:0;
      width:100%;
      height:45%;
      background:#333;
    }

    #score{
      position:absolute;
      top:15px;
      left:15px;
      font-size:22px;
      font-weight:bold;
      z-index:5;
    }

    #start{
      position:absolute;
      z-index:10;
      padding:15px 40px;
      font-size:22px;
      border:0;
      border-radius:12px;
      background:#18b957;
      color:white;
    }
  </style>
</head>

<body>

<div id="game">

  <div id="score">Score: 0</div>

  <div id="road"></div>

  <div id="player">
    <div id="head"></div>
  </div>

  <button id="start">PLAY</button>

</div>

<script>

let playing=false;
let score=0;

let lane=1;

const positions=[
  "25%",
  "50%",
  "75%"
];

document
.getElementById("start")
.onclick=function(){

  playing=true;

  this.style.display="none";

  gameLoop();
};


function moveLeft(){

  if(lane>0){
    lane--;
    updatePlayer();
  }
}


function moveRight(){

  if(lane<2){
    lane++;
    updatePlayer();
  }
}


function updatePlayer(){

  document
  .getElementById("player")
  .style.left=
    "calc("+
    positions[lane]+
    " - 27px)";
}


function jump(){

  if(!playing)return;

  const p=
    document.getElementById("player");

  p.style.bottom="280px";

  setTimeout(
    ()=>{
      p.style.bottom="150px";
    },
    500
  );
}


function gameLoop(){

  if(!playing)return;

  score++;

  document
  .getElementById("score")
  .innerText=
    "Score: "+score;

  requestAnimationFrame(
    gameLoop
  );
}


let sx=0;
let sy=0;


document.addEventListener(
  "touchstart",
  e=>{

    sx=e.touches[0].clientX;
    sy=e.touches[0].clientY;

  },
  {passive:false}
);


document.addEventListener(
  "touchend",
  e=>{

    const ex=
      e.changedTouches[0].clientX;

    const ey=
      e.changedTouches[0].clientY;

    const dx=ex-sx;
    const dy=ey-sy;


    if(
      Math.abs(dx)>
      Math.abs(dy)
    ){

      if(dx>40)
        moveRight();

      if(dx<-40)
        moveLeft();

    }else{

      if(dy<-40)
        jump();

    }

  },
  {passive:false}
);

</script>

</body>
</html>
