<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width,initial-scale=1.0,
      maximum-scale=1.0,user-scalable=no">

<title>Sky Runner 3D</title>

<style>
html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#79cfff;
    font-family:Arial,sans-serif;
    touch-action:none;
}

#game{
    width:100%;
    height:100%;
}

#hud{
    position:fixed;
    top:12px;
    left:12px;
    right:12px;
    z-index:20;
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
    z-index:50;
    display:flex;
    flex-direction:column;
    justify-content:center;
    align-items:center;
    text-align:center;
    color:white;
    background:rgba(0,0,0,.68);
}

#menu h1{
    margin:0 0 8px;
    font-size:42px;
    text-shadow:3px 3px 5px #000;
}

#menu p{
    max-width:320px;
    line-height:1.5;
}

#startBtn{
    border:0;
    border-radius:16px;
    padding:15px 42px;
    font-size:21px;
    font-weight:bold;
    background:#19c85b;
    color:white;
}

#characterBtn{
    margin-top:12px;
    border:0;
    border-radius:12px;
    padding:10px 20px;
    font-size:16px;
}

#characterInfo{
    margin-top:8px;
    font-size:14px;
}

#loading{
    position:fixed;
    inset:0;
    z-index:100;
    background:#79cfff;
    display:flex;
    justify-content:center;
    align-items:center;
    color:white;
    font-size:22px;
}
</style>
</head>

<body>

<div id="loading">Loading 3D Game...</div>

<div id="game"></div>

<div id="hud">
    <div id="score">Score: 0</div>
    <div id="high">Best: 0</div>
    <div id="speed">Speed: 1x</div>
</div>

<div id="menu">
    <h1 id="title">🏃 SKY RUNNER 3D</h1>
    <p id="message">
        Swipe left/right to change lane.<br>
        Swipe up to jump.<br>
        Swipe down to slide.
    </p>

    <button id="startBtn">START GAME</button>

    <button id="characterBtn">
        CHANGE CHARACTER
    </button>

    <div id="characterInfo">
        Character 1
    </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>

/* =========================================
   BASIC VARIABLES
========================================= */

let scene;
let camera;
let renderer;

let player;

let running=false;
let gameEnded=false;

let lane=1;

let score=0;
let highScore=
    Number(localStorage.getItem("skyRunnerHigh")) || 0;

let gameSpeed=0.45;
let speedLevel=1;

let jumping=false;
let sliding=false;

let jumpStart=0;

let objects=[];
let coins=[];
let obstacles=[];
let scenery=[];

let spawnTimer=0;
let coinTimer=0;

let characterNumber=1;

const lanes=[-3,0,3];


/* =========================================
   COLORS FOR CHARACTER CHANGE
========================================= */

const characterColors=[
    {
        shirt:0x1769aa,
        pants:0x222222,
        hair:0x222222
    },
    {
        shirt:0xe63946,
        pants:0x333333,
        hair:0x111111
    },
    {
        shirt:0x7b2cbf,
        pants:0x111111,
        hair:0x553311
    }
];


/* =========================================
   CREATE SCENE
========================================= */

function createScene(){

    scene=new THREE.Scene();

    scene.background=
        new THREE.Color(0x78cfff);

    scene.fog=
        new THREE.Fog(
            0x78cfff,
            35,
            180
        );


    camera=
        new THREE.PerspectiveCamera(
            65,
            innerWidth/innerHeight,
            0.1,
            500
        );

    camera.position.set(
        0,
        5,
        11
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
        Math.min(
            devicePixelRatio,
            2
        )
    );

    document
        .getElementById("game")
        .appendChild(
            renderer.domElement
        );


    /* LIGHT */

    const hemi=
        new THREE.HemisphereLight(
            0xffffff,
            0x4b5563,
            1.7
        );

    scene.add(hemi);


    const sun=
        new THREE.DirectionalLight(
            0xffffff,
            1.3
        );

    sun.position.set(
        -20,
        30,
        20
    );

    scene.add(sun);


    createWorld();
    createPlayer();

    animate();

    document
        .getElementById("loading")
        .style.display="none";
}


/* =========================================
   WORLD
========================================= */

function createWorld(){

    /* ROAD */

    const road=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                12,
                .25,
                300
            ),
            new THREE.MeshStandardMaterial({
                color:0x353535
            })
        );

    road.position.set(
        0,
        -.15,
        -130
    );

    scene.add(road);


    /* GRASS */

    const grass=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                60,
                .15,
                300
            ),
            new THREE.MeshStandardMaterial({
                color:0x48a23a
            })
        );

    grass.position.set(
        0,
        -.3,
        -130
    );

    scene.add(grass);


    /* LANE LINES */

    for(
        let z=5;
        z>-280;
        z-=8
    ){

        for(
            let x of [-2,2]
        ){

            const line=
                new THREE.Mesh(
                    new THREE.BoxGeometry(
                        .12,
                        .04,
                        4
                    ),
                    new THREE.MeshBasicMaterial({
                        color:0xffffff
                    })
                );

            line.position.set(
                x,
                .02,
                z
            );

            scene.add(line);

            objects.push(line);
        }
    }


    /* RAIL TRACKS */

    for(
        let x of [-3,0,3]
    ){

        const rail1=
            new THREE.Mesh(
                new THREE.BoxGeometry(
                    .08,
                    .08,
                    280
                ),
                new THREE.MeshStandardMaterial({
                    color:0xaaaaaa
                })
            );

        rail1.position.set(
            x-.7,
            .08,
            -130
        );

        scene.add(rail1);

        const rail2=
            rail1.clone();

        rail2.position.x=x+.7;

        scene.add(rail2);
    }


    createBuildings();

    createTrees();
}


/* =========================================
   BUILDINGS
========================================= */

function createBuildings(){

    for(
        let z=-10;
        z>-220;
        z-=12
    ){

        createBuilding(
            -10,
            z
        );

        createBuilding(
            10,
            z-5
        );
    }
}


function createBuilding(x,z){

    const height=
        5+
        Math.random()*10;

    const building=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                5,
                height,
                7
            ),
            new THREE.MeshStandardMaterial({
                color:
                    new THREE.Color(
                        .25+
                        Math.random()*.25,
                        .35+
                        Math.random()*.2,
                        .45+
                        Math.random()*.2
                    )
            })
        );

    building.position.set(
        x,
        height/2,
        z
    );

    scene.add(building);

    scenery.push(building);


    /* WINDOWS */

    for(
        let y=2;
        y<height;
        y+=2.2
    ){

        const windowMesh=
            new THREE.Mesh(
                new THREE.BoxGeometry(
                    .5,
                    .8,
                    .08
                ),
                new THREE.MeshBasicMaterial({
                    color:0xffe66d
                })
            );

        windowMesh.position.set(
            x>0
                ?x-2.53
                :x+2.53,
            y,
            z
        );

        scene.add(windowMesh);

        scenery.push(windowMesh);
    }
}


/* =========================================
   TREES
========================================= */

function createTrees(){

    for(
        let z=-5;
        z>-220;
        z-=10
    ){

        createTree(
            -9,
            z
        );

        createTree(
            9,
            z-4
        );
    }
}


function createTree(x,z){

    const tree=
        new THREE.Group();


    const trunk=
        new THREE.Mesh(
            new THREE.CylinderGeometry(
                .3,
                .4,
                2,
                8
            ),
            new THREE.MeshStandardMaterial({
                color:0x704214
            })
        );

    trunk.position.y=1;

    tree.add(trunk);


    const leaves=
        new THREE.Mesh(
            new THREE.SphereGeometry(
                1.5,
                12,
                12
            ),
            new THREE.MeshStandardMaterial({
                color:0x168a32
            })
        );

    leaves.position.y=2.8;

    tree.add(leaves);


    tree.position.set(
        x,
        0,
        z
    );

    scene.add(tree);

    scenery.push(tree);
}


/* =========================================
   PLAYER
========================================= */

function createPlayer(){

    player=
        new THREE.Group();


    const c=
        characterColors[
            characterNumber-1
        ];


    /* BODY */

    const body=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                1.05,
                1.55,
                .7
            ),
            new THREE.MeshStandardMaterial({
                color:c.shirt
            })
        );

    body.position.y=2.15;

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

    head.position.y=3.4;

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
                color:c.hair
            })
        );

    hair.scale.y=.55;

    hair.position.y=3.72;

    player.add(hair);


    /* ARMS */

    player.leftArm=
        createLimb(
            .25,
            1.35,
            0xffc49a
        );

    player.rightArm=
        createLimb(
            .25,
            1.35,
            0xffc49a
        );

    player.leftArm.position.set(
        -.72,
        2.2,
        0
    );

    player.rightArm.position.set(
        .72,
        2.2,
        0
    );

    player.add(
        player.leftArm
    );

    player.add(
        player.rightArm
    );


    /* LEGS */

    player.leftLeg=
        createLimb(
            .32,
            1.35,
            c.pants
        );

    player.rightLeg=
        createLimb(
            .32,
            1.35,
            c.pants
        );

    player.leftLeg.position.set(
        -.3,
        .65,
        0
    );

    player.rightLeg.position.set(
        .3,
        .65,
        0
    );

    player.add(
        player.leftLeg
    );

    player.add(
        player.rightLeg
    );


    player.position.set(
        0,
        0,
        4
    );

    scene.add(player);
}


function createLimb(
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


/* =========================================
   CHARACTER CHANGE
========================================= */

function changeCharacter(){

    characterNumber++;

    if(
        characterNumber>
        characterColors.length
    ){
        characterNumber=1;
    }

    characterInfo.innerText=
        "Character "+
        characterNumber;

    if(!running){

        scene.remove(player);

        createPlayer();
    }
}


/* =========================================
   LANE MOVEMENT
========================================= */

function moveLeft(){

    if(
        !running ||
        gameEnded
    )return;

    if(lane>0){

        lane--;

        player.position.x=
            lanes[lane];
    }
}


function moveRight(){

    if(
        !running ||
        gameEnded
    )return;

    if(lane<2){

        lane++;

        player.position.x=
            lanes[lane];
    }
}


/* =========================================
   JUMP
========================================= */

function jump(){

    if(
        !running ||
        jumping ||
        sliding
    )return;

    jumping=true;

    jumpStart=
        performance.now();
}


function updateJump(){

    if(!jumping)return;

    const elapsed=
        performance.now()-
        jumpStart;

    const duration=700;

    const t=
        Math.min(
            elapsed/duration,
            1
        );

    player.position.y=
        Math.sin(
            t*Math.PI
        )*3;


    if(t>=1){

        player.position.y=0;

        jumping=false;
    }
}


/* =========================================
   SLIDE
========================================= */

function slide(){

    if(
        !running ||
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


/* =========================================
   COIN
========================================= */

function createCoin(){

    const coin=
        new THREE.Mesh(
            new THREE.TorusGeometry(
                .38,
                .12,
                12,
                24
            ),
            new THREE.MeshStandardMaterial({
                color:0xffd000,
                metalness:.7
            })
        );

    coin.position.set(
        lanes[
            Math.floor(
                Math.random()*3
            )
        ],
        1.5+
        Math.random()*1.5,
        -90
    );

    scene.add(coin);

    coins.push(coin);
}


/* =========================================
   TRAIN
========================================= */

function createTrain(){

    const train=
        new THREE.Group();


    const body=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                2.4,
                2.8,
                7
            ),
            new THREE.MeshStandardMaterial({
                color:0xd62828
            })
        );

    body.position.y=1.5;

    train.add(body);


    /* FRONT */

    const front=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                2,
                1.3,
                .15
            ),
            new THREE.MeshStandardMaterial({
                color:0x222222
            })
        );

    front.position.set(
        0,
        1.8,
        -3.55
    );

    train.add(front);


    /* WINDOWS */

    for(
        let z=-2.3;
        z<=2.3;
        z+=1.5
    ){

        const windowMesh=
            new THREE.Mesh(
                new THREE.BoxGeometry(
                    1.6,
                    .7,
                    .08
                ),
                new THREE.MeshBasicMaterial({
                    color:0x9ee7ff
                })
            );

        windowMesh.position.set(
            0,
            2.3,
            z
        );

        train.add(windowMesh);
    }


    train.position.set(
        lanes[
            Math.floor(
                Math.random()*3
            )
        ],
        0,
        -100
    );

    scene.add(train);

    obstacles.push(train);
}


/* =========================================
   ROAD OBSTACLE
========================================= */

function createObstacle(){

    const obstacle=
        new THREE.Group();


    const base=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                1.7,
                1.3,
                1.7
            ),
            new THREE.MeshStandardMaterial({
                color:0xf77f00
            })
        );

    base.position.y=.65;

    obstacle.add(base);


    const top=
        new THREE.Mesh(
            new THREE.BoxGeometry(
                1.9,
                .3,
                1.9
            ),
            new THREE.MeshStandardMaterial({
                color:0xffffff
            })
        );

    top.position.y=1.35;

    obstacle.add(top);


    obstacle.position.set(
        lanes[
            Math.floor(
                Math.random()*3
            )
        ],
        0,
        -90
    );

    scene.add(obstacle);

    obstacles.push(obstacle);
}


/* =========================================
   COLLISION
========================================= */

function checkCollision(){

    const playerBox=
        new THREE.Box3()
        .setFromObject(player);


    for(
        let i=obstacles.length-1;
        i>=0;
        i--
    ){

        const object=
            obstacles[i];

        const box=
            new THREE.Box3()
            .setFromObject(object);


        if(
            playerBox.intersectsBox(box)
        ){

            /* Jump avoids low obstacles */

            if(
                jumping &&
                object.children.length<3
            ){
                continue;
            }

            gameOver();

            return;
        }
    }


    for(
        let i=coins.length-1;
        i>=0;
        i--
    ){

        const coin=coins[i];

        const box=
            new THREE.Box3()
            .setFromObject(coin);


        if(
            playerBox.intersectsBox(box)
        ){

            score+=10;

            scene.remove(coin);

            coins.splice(i,1);
        }
    }
}


/* =========================================
   SPAWNING
========================================= */

function spawnObjects(delta){

    spawnTimer+=delta;

    coinTimer+=delta;


    if(
        spawnTimer>
        Math.max(
            .65,
            1.35-
            speedLevel*.06
        )
    ){

        spawnTimer=0;

        if(
            Math.random()<.55
        ){
            createTrain();
        }else{
            createObstacle();
        }
    }


    if(
        coinTimer>.55
    ){

        coinTimer=0;

        createCoin();
    }
}


/* =========================================
   MOVE WORLD
========================================= */

function moveWorld(){

    const all=
        obstacles.concat(
            coins,
            scenery,
            objects
        );


    for(
        let i=0;
        i<all.length;
        i++
    ){

        const object=all[i];

        /* scenery slower visual effect */

        let movement=
            gameSpeed;

        if(
            scenery.includes(object)
        ){
            movement=
                gameSpeed;
        }

        object.position.z+=
            movement;


        if(
            object.position.z>20
        ){

            scene.remove(object);

            let index;

            index=
                obstacles.indexOf(object);

            if(index!==-1)
                obstacles.splice(
                    index,
                    1
                );


            index=
                coins.indexOf(object);

            if(index!==-1)
                coins.splice(
                    index,
                    1
                );
        }
    }
}


/* =========================================
   START GAME
========================================= */

function startGame(){

    running=true;
    gameEnded=false;

    score=0;
    gameSpeed=.45;
    speedLevel=1;

    lane=1;

    player.position.x=0;
    player.position.y=0;

    document
        .getElementById("menu")
        .style.display="none";
}


/* =========================================
   GAME OVER
========================================= */

function gameOver(){

    if(gameEnded)return;

    gameEnded=true;
    running=false;


    const finalScore=
        Math.floor(score);


    if(
        finalScore>
        highScore
    ){

        highScore=
            finalScore;

        localStorage.setItem(
            "skyRunnerHigh",
            highScore
        );
    }


    document
        .getElementById("title")
        .innerText=
        "💥 GAME OVER";


    document
        .getElementById("message")
        .innerHTML=
        "Score: "+
        finalScore+
        "<br>Best: "+
        highScore;


    document
        .getElementById("startBtn")
        .innerText=
        "PLAY AGAIN";


    document
        .getElementById("menu")
        .style.display="flex";
}


/* =========================================
   MAIN LOOP
========================================= */

let lastTime=
    performance.now();


function animate(){

    requestAnimationFrame(
        animate
    );


    const now=
        performance.now();

    const delta=
        (now-lastTime)/1000;

    lastTime=now;


    if(running){

        /* SCORE */

        score+=
            delta*
            (8+
            speedLevel);


        /* SPEED INCREASE */

        speedLevel=
            1+
            Math.floor(
                score/250
            );

        gameSpeed=
            .45+
            speedLevel*.045;


        /* RUNNING */

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
                )*.55;

            player.rightArm.rotation.x=
                Math.sin(t)*.55;
        }


        updateJump();

        spawnObjects(delta);

        moveWorld();

        checkCollision();


        /* CAMERA */

        camera.position.x+=
            (
                player.position.x-
                camera.position.x
            )*.08;

        camera.position.y=
            4.5+
            player.position.y*.2;

        camera.lookAt(
            player.position.x,
            1.8,
            -12
        );


        document
            .getElementById("score")
            .innerText=
            "Score: "+
            Math.floor(score);


        document
            .getElementById("high")
            .innerText=
            "Best: "+
            highScore;


        document
            .getElementById("speed")
            .innerText=
            "Speed: "+
            speedLevel+
            "x";
    }


    renderer.render(
        scene,
        camera
    );
}


/* =========================================
   SWIPE CONTROLS
========================================= */

let startX=0;
let startY=0;


document.addEventListener(
    "touchstart",
    function(e){

        startX=
            e.touches[0].clientX;

        startY=
            e.touches[0].clientY;
    },
    {passive:false}
);


document.addEventListener(
    "touchend",
    function(e){

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

            if(dx>45)
                moveRight();

            if(dx<-45)
                moveLeft();

        }else{

            if(dy<-45)
                jump();

            if(dy>45)
                slide();
        }

    },
    {passive:false}
);


/* =========================================
   KEYBOARD
========================================= */

document.addEventListener(
    "keydown",
    function(e){

        if(e.key==="ArrowLeft")
            moveLeft();

        if(e.key==="ArrowRight")
            moveRight();

        if(e.key==="ArrowUp")
            jump();

        if(e.key==="ArrowDown")
            slide();
    }
);


/* =========================================
   BUTTONS
========================================= */

document
    .getElementById("startBtn")
    .addEventListener(
        "click",
        startGame
    );


document
    .getElementById("characterBtn")
    .addEventListener(
        "click",
        changeCharacter
    );


/* =========================================
   RESIZE
========================================= */

window.addEventListener(
    "resize",
    function(){

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


/* =========================================
   HIGH SCORE DISPLAY
========================================= */

document
    .getElementById("high")
    .innerText=
    "Best: "+
    highScore;


/* START */

createScene();

</script>

</body>
</html>
