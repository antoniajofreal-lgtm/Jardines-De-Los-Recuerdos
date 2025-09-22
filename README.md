<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1.0" />
<title>El Jardín de los Recuerdos 🌳</title>
<style>
  :root {
    --bottom-space: 200px;
  }
  html,body{
    height:100%;
    margin:0;
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial;
    background: url("https://cdn.pixabay.com/photo/2015/06/19/21/24/avenue-815297_1280.jpg") center/cover no-repeat fixed;
    color: #153e2e;
  }

  #startScreen {
    position:fixed; inset:0;
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    background: rgba(255,255,255,0.86);
    z-index: 900;
    padding:20px; box-sizing:border-box; text-align:center;
  }
  /* Título de inicio más grande */
  #startScreen h1 { margin:0 0 8px; font-size:2.6rem; color:#1f6b34; font-weight:bold; }
  #startScreen p { color:#2b6a36; max-width:640px; font-size:1.2rem; }

  .btn {
    padding:10px 18px; border-radius:10px; border:0; cursor:pointer;
    background:#57b86a; color:white; font-size:1rem; margin:8px;
  }
  .btn.alt { background:#6ca3f5; }

  #gameContainer { display:none; padding:18px; min-height:100vh; box-sizing:border-box; }
  header.appbar { display:flex; justify-content:center; margin-bottom:8px; }
  /* Título de la app (appbar) más grande */
  header.appbar h2 { margin:0; color:#143e29; background: rgba(255,255,255,0.8); padding:8px 20px; border-radius:10px; font-size:2rem; font-weight:bold; }

  /* HUD (vidas/puntos) más visible */
  .hud { display:flex; justify-content:space-between; align-items:center; gap:12px; margin-bottom:14px; color:#1e6b2a; font-size:1.3rem; font-weight:bold; }
  #elements {
    display:grid;
    gap:14px;
    justify-content:center;
    padding-bottom: var(--bottom-space);
    margin-top:6px;
  }
  .cell {
    width:84px; height:84px; border-radius:12px; background: rgba(255,255,255,0.95);
    display:flex; align-items:center; justify-content:center; font-size:40px;
    box-shadow:0 6px 14px rgba(0,0,0,0.12);
    touch-action: manipulation; user-select:none;
  }
  .cell.active { transform: scale(1.08); background:#c7f1d6; transition: all .16s; }

  #gardenerWrapper {
    position:fixed; right:12px; bottom:12px; display:flex; flex-direction:column; align-items:center;
    gap:10px; z-index:1000; pointer-events:none;
  }
  /* Burbuja del jardinero más grande */
  #speechBubble {
    min-width:240px; max-width:360px; font-size:22px; background: rgba(255,255,255,0.98);
    color:#173d2a; border-radius:14px; padding:20px; border:3px solid rgba(0,0,0,0.15);
    box-shadow:0 12px 28px rgba(0,0,0,0.16);
    text-align:center;
  }
  #speechBubble.error { 
    background:#e60000; 
    color:#fff; 
    border-color:#990000; 
    font-weight:bold; 
  }
  #gardenerImg { width:120px; display:block; }

  .overlay { position:fixed; inset:0; display:none; align-items:center; justify-content:center; z-index:1200; background: rgba(255,255,255,0.95); text-align:center; }
  .overlay h2 { color:#1f6b34; margin:0 0 8px; font-size:2rem; }

  @media(max-width:520px){
    .cell{ width:72px; height:72px; font-size:34px; }
    #speechBubble{ font-size:18px; max-width:260px; }
    header.appbar h2 { font-size:1.6rem; }
    .hud { font-size:1.1rem; }
    :root { --bottom-space: 170px; }
  }
</style>
</head>
<body>

<div id="startScreen" role="dialog" aria-modal="true">
  <h1>🌳 El Jardín de los Recuerdos</h1>
  <p>Ayuda al jardinero a cuidar su jardín recordando secuencias. 2 secuencias por ronda — sin penalizaciones, 3 vidas.</p>
  <div style="margin-top:14px;">
    <button id="startBtn" class="btn">🌸 Comenzar Juego</button>
    <button id="musicBtnStart" class="btn alt">🔊 Música</button>
  </div>
</div>

<div id="gameContainer" aria-live="polite">
  <header class="appbar"><h2>El Jardín de los Recuerdos</h2></header>
  <div class="hud">
    <div id="scoreDisplay">Puntos: 0</div>
    <div id="livesDisplay">Vidas: 🌸🌸🌸</div>
  </div>
  <main id="elements"></main>
  <div style="display:flex; justify-content:center; gap:8px; margin-top:10px;">
    <button id="restartBtn" class="btn" style="background:#f6a45d">🔄 Reiniciar</button>
    <button id="musicBtnGame" class="btn alt">🔊 Música</button>
  </div>
</div>

<div id="gardenerWrapper">
  <div id="speechBubble">¡Hola! Presiona "Comenzar Juego".</div>
  <img id="gardenerImg" src="https://cdn.pixabay.com/photo/2014/04/03/11/53/gardener-311325_1280.png" alt="Jardinero">
</div>

<div id="winOverlay" class="overlay">
  <div>
    <h2>🌟 ¡Gracias por jugar! 🌟</h2>
    <p id="winStats"></p>
    <div style="margin-top:12px;"><button id="winRestart" class="btn">Volver a jugar</button></div>
  </div>
</div>
<div id="loseOverlay" class="overlay">
  <div>
    <h2>💪 ¡Ánimo! 💪</h2>
    <p id="loseStats"></p>
    <div style="margin-top:12px;"><button id="loseRestart" class="btn">Reintentar</button></div>
  </div>
</div>

<audio id="bgMusic" loop src="https://files.freemusicarchive.org/storage-freemusicarchive-org/music/ccCommunity/Jahzzar/Traveller/Jahzzar_-_05_-_Siesta.mp3"></audio>
<audio id="sndSeq" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
<audio id="sndErr" src="https://actions.google.com/sounds/v1/cartoon/metal_thud_and_clang.ogg"></audio>

<script>
const startScreen=document.getElementById('startScreen');
const startBtn=document.getElementById('startBtn');
const musicBtnStart=document.getElementById('musicBtnStart');
const gameContainer=document.getElementById('gameContainer');
const elementsWrap=document.getElementById('elements');
const scoreDisplay=document.getElementById('scoreDisplay');
const livesDisplay=document.getElementById('livesDisplay');
const restartBtn=document.getElementById('restartBtn');
const musicBtnGame=document.getElementById('musicBtnGame');
const speechBubble=document.getElementById('speechBubble');
const gardenerImg=document.getElementById('gardenerImg');
const winOverlay=document.getElementById('winOverlay');
const loseOverlay=document.getElementById('loseOverlay');
const winRestart=document.getElementById('winRestart');
const loseRestart=document.getElementById('loseRestart');
const winStats=document.getElementById('winStats');
const loseStats=document.getElementById('loseStats');

const bgMusic=document.getElementById('bgMusic');
const sndSeq=document.getElementById('sndSeq');
const sndErr=document.getElementById('sndErr');

gardenerImg.addEventListener('error',()=>{ gardenerImg.src='🧑‍🌾'; });

const rounds=[
  { name:"Sembrar las frutas 🌱", pool:['🌸','🌻','🌷','🌼'], cells:4 },
  { name:"Cosechar las frutas 🍎", pool:['🍎','🍌','🍐','🍇','🍓','🍊'], cells:6 },
  { name:"A juntar la cosecha 🧺", pool:['🥕','🌽','🍅','🍆','🥒','🥔','🥦','🧅'], cells:8 }
];

let currentRound=1,sequence=[],playerIndex=0,sequenceCount=0,score=0,lives=3,listening=false,musicOn=false;
let correctMoves=0,errors=0;

function setBubble(text,ms=3000,isError=false){
  speechBubble.textContent=text;
  speechBubble.classList.toggle('error',isError);
  speechBubble.style.display='block';
  setTimeout(()=>{ if(speechBubble.textContent===text) speechBubble.style.display='none'; },ms);
}
function updateHUD(){
  scoreDisplay.textContent='Puntos: '+score;
  livesDisplay.textContent='Vidas: '+'🌸'.repeat(Math.max(0,lives));
}

async function safePlay(audioEl){ if(!audioEl)return; try{ audioEl.currentTime=0; await audioEl.play(); }catch{} }
async function toggleMusic(){
  if(musicOn){ bgMusic.pause(); musicOn=false; }
  else { await safePlay(bgMusic); musicOn=!bgMusic.paused; }
  musicBtnStart.textContent=musicOn?'🔇 Música':'🔊 Música';
  musicBtnGame.textContent=musicOn?'🔇 Música':'🔊 Música';
}
musicBtnStart.addEventListener('click',toggleMusic);
musicBtnGame.addEventListener('click',toggleMusic);

function buildBoard(roundIndex){
  elementsWrap.innerHTML='';
  elementsWrap.style.gridTemplateColumns=`repeat(${rounds[roundIndex-1].cells/2}, minmax(72px,84px))`;
  const pool=rounds[roundIndex-1].pool;
  for(let i=0;i<rounds[roundIndex-1].cells;i++){
    const div=document.createElement('div');
    div.className='cell'; div.dataset.idx=i; div.textContent=pool[i];
    div.addEventListener('pointerdown',(ev)=>{ev.preventDefault(); onCellPressed(i);});
    elementsWrap.appendChild(div);
  }
}
function generateSequence(len){
  const poolLen=rounds[currentRound-1].cells;
  return Array.from({length:len},()=>Math.floor(Math.random()*poolLen));
}
async function showSequence(seq){
  listening=false; setBubble('Observa la secuencia...',3000);
  await new Promise(r=>setTimeout(r,2500));
  const cells=[...elementsWrap.querySelectorAll('.cell')];
  for(let idx of seq){
    const cell=cells[idx]; if(cell){ cell.classList.add('active'); safePlay(sndSeq);
    await new Promise(r=>setTimeout(r,700)); cell.classList.remove('active'); await new Promise(r=>setTimeout(r,150)); }
  }
  listening=true; setBubble('¡Tu turno! Repite la secuencia',3000);
}
function startRound(){
  if(currentRound>rounds.length){ gameOverWin(); return; }
  sequenceCount=0; setBubble('Ronda '+currentRound+': '+rounds[currentRound-1].name,3000);
  buildBoard(currentRound); setTimeout(()=> startNewSequence(),900);
}
function startNewSequence(){
  const len=2+(currentRound-1)+sequenceCount;
  sequence=generateSequence(len); playerIndex=0;
  showSequence(sequence);
}
function onCellPressed(idx){
  if(!listening) return;
  if(idx!==sequence[playerIndex]){
    safePlay(sndErr); lives=Math.max(0,lives-1); errors++; updateHUD();
    setBubble('¡Ups! Esa no es la correcta.',3000,true); listening=false; playerIndex=0;
    if(lives<=0){ setTimeout(()=>gameOverLose(),700); }
    else { setTimeout(()=>showSequence(sequence),900); }
    return;
  }
  safePlay(sndSeq); correctMoves++;
  const cell=[...elementsWrap.querySelectorAll('.cell')][idx];
  if(cell){ cell.classList.add('active'); setTimeout(()=>cell.classList.remove('active'),220); }
  playerIndex++; score+=10; updateHUD();
  if(playerIndex>=sequence.length){
    score+=20; updateHUD(); sequenceCount++; listening=false;
    if(sequenceCount<2){ setBubble('¡Muy bien! Otra secuencia.',3000); setTimeout(()=>startNewSequence(),1000);}
    else { setBubble('¡Ronda completada! 🎉',3000); currentRound++; setTimeout(()=>startRound(),1500); }
  }
}
function gameOverWin(){
  gameContainer.style.display='none'; winOverlay.style.display='flex';
  winStats.textContent=`Puntos: ${score} | Rondas completadas: ${rounds.length} | Secuencias correctas: ${correctMoves} | Errores: ${errors}`;
}
function gameOverLose(){
  gameContainer.style.display='none'; loseOverlay.style.display='flex';
  loseStats.textContent=`Puntos: ${score} | Rondas alcanzadas: ${currentRound-1} | Secuencias correctas: ${correctMoves} | Errores: ${errors}`;
}
function startBtnClicked(){
  startScreen.style.display='none'; gameContainer.style.display='block';
  currentRound=1; score=0; lives=3; sequenceCount=0; playerIndex=0; correctMoves=0; errors=0;
  updateHUD(); setTimeout(()=>startRound(),400);
}
startBtn.addEventListener('click',startBtnClicked);
restartBtn.addEventListener('click',()=>{ winOverlay.style.display='none'; loseOverlay.style.display='none'; startBtnClicked(); });
winRestart.addEventListener('click',()=>{ winOverlay.style.display='none'; startBtnClicked(); });
loseRestart.addEventListener('click',()=>{ loseOverlay.style.display='none'; startBtnClicked(); });
updateHUD(); setBubble('¡Hola! Presiona "Comenzar Juego".',3000);
</script>
</body>
</html>
