<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>🐾フィガロの四目並べ🐾</title>

<!-- ⭐ PWA 追加 -->
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#111111">

<!-- ⭐ iPhone向け PWA 表示用 -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<link rel="apple-touch-icon" href="icon-192.png">

<style>
:root{
  --bg:#111;
  --board:#222;
  --cell:#333;
  --text:#fff;
  --xcolor:#4fa7ff;
  --ocolor:#ff5a5a;
}
.light{
  --bg:#fafafa;
  --board:#ffffff;
  --cell:#e6e6e6;
  --text:#111;
}
.turn-o{ background:#2a0000; }
.turn-x{ background:#001a33; }

body{
  background:var(--bg);
  color:var(--text);
  font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;
  margin:0;
  display:flex;
  flex-direction:column;
  align-items:center;
  gap:10px;
  padding-bottom:80px;
  transition:background .25s;
}

.board-wrap{
  width:min(96vw,480px);
  height:min(96vw,480px);
  display:flex;
  align-items:center;
  justify-content:center;
}

#board{
  background:var(--board);
  padding:8px;
  border-radius:10px;
  display:grid;
  grid-template-columns:repeat(7,1fr);
  gap:4px;
  width:100%;
  height:100%;
}

.cell{
  background:var(--cell);
  border-radius:8px;
  display:flex;
  align-items:center;
  justify-content:center;
  aspect-ratio:1/1;
  font-size:32px;
  line-height:1;
  user-select:none;
  position:relative;
}

.block{ background:#555; }
.mark-o{ color:var(--ocolor); }
.mark-x{ color:var(--xcolor); }

.protect-overlay{
  position:absolute;
  inset:2px;
  border-radius:10px;
  background:radial-gradient(circle, rgba(255,255,255,.35), rgba(255,255,255,0));
  border:2px solid rgba(255,255,255,.4);
  pointer-events:none;
}

.win{
  animation:shine .7s infinite alternate;
  box-shadow:0 0 12px 3px gold inset;
}
@keyframes shine{
  from{ filter:brightness(1); }
  to{ filter:brightness(2); }
}

.explode{
  animation:
    explodeFlash .45s ease-out both,
    explodeShake .45s ease-out both;
}
@keyframes explodeFlash{
  0%{background:#f00;transform:scale(1);}
  40%{background:#f55;transform:scale(1.2);}
  100%{background:var(--cell);}
}
@keyframes explodeShake{
  0%{transform:translate(0,0);}
  25%{transform:translate(-2px,2px);}
  50%{transform:translate(2px,-2px);}
  75%{transform:translate(-2px,-2px);}
  100%{transform:translate(0,0);}
}

.ruined{
  background:repeating-linear-gradient(135deg,#1a1a1a 0 6px,#0d0d0d 6px 12px);
  filter:brightness(.6);
  pointer-events:none;
}

.shake{
  animation:shakeX .35s;
}
@keyframes shakeX{
  0%,100%{transform:translateX(0);}
  25%{transform:translateX(-6px);}
  50%{transform:translateX(6px);}
  75%{transform:translateX(-6px);}
}

.themeBtn{
  font-size:26px;
  background:none;
  border:none;
  cursor:pointer;
}

.skill-panel{
  width:min(96vw,480px);
  display:flex;
  gap:6px;
}

.skill-btn{
  flex:1;
  padding:10px 6px 6px 6px;
  border-radius:14px;
  border:none;
  cursor:pointer;
  color:#fff;
  font-size:13px;
  position:relative;
  box-shadow:0 4px 10px rgba(0,0,0,.35);
  display:flex;
  flex-direction:column;
  align-items:center;
  gap:2px;
}
.skill-o{ background:#b83232; }
.skill-x{ background:#245d9c; }

.badge{
  position:absolute;
  top:4px;
  right:6px;
  background:#0008;
  padding:2px 7px;
  border-radius:999px;
  font-size:11px;
}

.skill-btn.active{
  outline:3px solid gold;
  box-shadow:0 0 14px 2px gold;
}
.skill-btn:disabled{
  filter:grayscale(.9);
  opacity:.6;
}

.top-controls{ transform:rotate(180deg); }

.info{ font-size:14px; }

.control-row{
  width:min(96vw,480px);
  display:flex;
  gap:8px;
}
.reset-btn{
  flex:1;
  padding:10px;
  border-radius:12px;
  border:none;
  background:#666;
  color:#fff;
}
</style>
</head>

<body>

<h3>🐱フィガロの〇✕ゲーム🐱</h3>

<button id="modeBtn" class="themeBtn">🌙</button>

<div class="skill-panel top-controls">
  <button id="x-block" class="skill-btn skill-x">🔒 封鎖 <span class="badge" id="x-block-b"></span></button>
  <button id="x-override" class="skill-btn skill-x">♻ 上書 <span class="badge" id="x-override-b"></span></button>
  <button id="x-protect" class="skill-btn skill-x">🛡 プロテクト <span class="badge" id="x-protect-b"></span></button>
</div>

<div id="turnInfo" class="info"></div>

<div class="board-wrap">
  <div id="board"></div>
</div>

<div class="skill-panel">
  <button id="o-block" class="skill-btn skill-o">🔒 封鎖 <span class="badge" id="o-block-b"></span></button>
  <button id="o-override" class="skill-btn skill-o">♻ 上書 <span class="badge" id="o-override-b"></span></button>
  <button id="o-protect" class="skill-btn skill-o">🛡 プロテクト <span class="badge" id="o-protect-b"></span></button>
</div>

<div class="control-row">
  <button id="reset" class="reset-btn">リセット</button>
</div>

<script>
// --- 変数・初期状態 ---
const N=7;
let board=[];
let current="O";
let gameOver=false;
let phase="mine";
let usedSkillThisTurn=false;
let mines={O:null,X:null};
let skill={
  O:{block:1,override:1,protect:1,mode:null},
  X:{block:1,override:1,protect:1,mode:null}
};

// --- 初期化 ---
function init(){
  const b=document.getElementById("board");
  b.innerHTML="";
  board=[];
  current="O";
  gameOver=false;
  phase="mine";
  usedSkillThisTurn=false;
  mines={O:null,X:null};

  for(let r=0;r<N;r++){
    board[r]=[];
    for(let c=0;c<N;c++){
      board[r][c]={val:"",protect:false,ruined:false};
      const el=document.createElement("div");
      el.className="cell";
      el.onclick=()=>cellClick(r,c);
      b.appendChild(el);
    }
  }

  let k=0;
  while(k<10){
    const r=Math.floor(Math.random()*N);
    const c=Math.floor(Math.random()*N);
    if(board[r][c].val===""){
      board[r][c].val="#";
      k++;
    }
  }

  updateTurn();
  updateSkillButtons();
  render();
}

function cellEl(r,c){ return document.getElementById("board").children[r*N+c]; }

function render(){
  for(let r=0;r<N;r++){
    for(let c=0;c<N;c++){
      const el=cellEl(r,c);
      const d=board[r][c];

      el.className="cell";
      el.textContent="";

      if(d.ruined){
        el.classList.add("ruined");
        continue;
      }
      if(d.val==="#") el.classList.add("block");
      if(d.val==="O"){el.textContent="〇"; el.classList.add("mark-o");}
      if(d.val==="X"){el.textContent="✕"; el.classList.add("mark-x");}

      if(d.protect){
        const o=document.createElement("div");
        o.className="protect-overlay";
        el.appendChild(o);
      }
    }
  }
}

function updateTurn(){
  const info=document.getElementById("turnInfo");
  document.body.classList.remove("turn-o","turn-x");
  document.body.classList.add(current==="O"?"turn-o":"turn-x");

  if(phase==="mine"){
    info.textContent=`地雷配置：${current==="O"?"〇":"✕"}`;
  }else if(!gameOver){
    info.textContent=`手番：${current==="O"?"〇":"✕"}`;
  }
}

// --- ボード満杯チェック & 空白化 ---
function isBoardFull(){
  for(let r=0;r<N;r++){
    for(let c=0;c<N;c++){
      const d=board[r][c];
      if(d.ruined) continue;
      if(d.val==="") return false;
    }
  }
  return true;
}

function openRandomCells(count=10){
  const candidates=[];
  for(let r=0;r<N;r++){
    for(let c=0;c<N;c++){
      const d=board[r][c];
      if(d.ruined) continue;
      candidates.push([r,c]);
    }
  }
  for(let i=candidates.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [candidates[i],candidates[j]]=[candidates[j],candidates[i]];
  }
  for(let i=0;i<Math.min(count,candidates.length);i++){
    const [r,c]=candidates[i];
    board[r][c].val="";
    board[r][c].protect=false;
  }
  render();
}

function nextTurn(){
  current=current==="O"?"X":"O";
  usedSkillThisTurn=false;
  updateTurn();
  if(isBoardFull()) openRandomCells(10);
}

// --- 勝利判定 ---
function checkWin(r,c){
  const dirs=[[1,0],[0,1],[1,1],[1,-1]];
  for(const[dR,dC] of dirs){
    let cells=[[r,c]];
    let rr=r+dR,cc=c+dC;
    while(true){
      rr=(rr+N)%N; cc=(cc+N)%N;
      if(board[rr][cc].val===current) cells.push([rr,cc]); else break;
      rr+=dR; cc+=dC;
    }
    rr=r-dR; cc=c-dC;
    while(true){
      rr=(rr+N)%N; cc=(cc+N)%N;
      if(board[rr][cc].val===current) cells.push([rr,cc]); else break;
      rr-=dR; cc-=dC;
    }
    if(cells.length>=4){
      cells.forEach(([r,c])=>cellEl(r,c).classList.add("win"));
      document.getElementById("turnInfo").textContent=`${current==="O"?"〇":"✕"} の勝利！`;
      gameOver=true;
      return true;
    }
  }
  return false;
}

// --- 地雷処理 ---
function isMine(r,c){
  return (mines.O && mines.O.r===r && mines.O.c===c) ||
         (mines.X && mines.X.r===r && mines.X.c===c);
}
function clearMine(r,c){
  if(mines.O && mines.O.r===r && mines.O.c===c) mines.O=null;
  if(mines.X && mines.X.r===r && mines.X.c===c) mines.X=null;
}
function explode(r,c){
  const el=cellEl(r,c);
  el.classList.add("explode");
  setTimeout(()=>{ el.classList.remove("explode"); board[r][c].ruined=true; render(); },450);
}
function triggerMine(r,c){
  if(!isMine(r,c)) return false;
  clearMine(r,c);
  explode(r,c);
  nextTurn();
  return true;
}

// --- セルクリック ---
function cellClick(r,c){
  if(gameOver) return;
  const d=board[r][c];
  if(d.ruined) return;

  if(phase==="mine"){
    if(!mines[current]){
      mines[current]={r,c};
      if(mines.O && mines.X){ phase="play"; current="O"; usedSkillThisTurn=false; updateTurn();}
      else nextTurn();
    }
    return;
  }

  if(triggerMine(r,c)) return;

  const mode=skill[current].mode;
  if(usedSkillThisTurn && mode){
    const el=cellEl(r,c);
    el.classList.add("shake");
    setTimeout(()=>el.classList.remove("shake"),350);
    return;
  }

  if(mode==="block"){
    if(d.val==="O"||d.val==="X"||d.ruined) return;
    d.val="#"; skill[current].block--; skill[current].mode=null; usedSkillThisTurn=true;
    render(); updateSkillButtons(); return;
  }

  if(mode==="override"){
    if(d.protect || d.val===current){ const el=cellEl(r,c); el.classList.add("shake"); setTimeout(()=>el.classList.remove("shake"),350); return; }
    if(d.val===""||d.val==="#"||d.ruined) return;
    d.val=current; d.protect=false; skill[current].override--; skill[current].mode=null; usedSkillThisTurn=true;
    render(); updateSkillButtons(); if(checkWin(r,c)) return; nextTurn(); return;
  }

  if(mode==="protect"){
    if(d.val!==""||d.val==="#") return;
    d.val=current; d.protect=true; skill[current].protect--; skill[current].mode=null; usedSkillThisTurn=true;
    render(); updateSkillButtons(); if(checkWin(r,c)) return; nextTurn(); return;
  }

  if(d.val!==""||d.val==="#"||d.ruined) return;
  d.val=current; d.protect=false;
  render(); if(checkWin(r,c)) return; nextTurn();
}

// --- テーマ切替 ---
const themeBtn=document.getElementById("modeBtn");
themeBtn.onclick=()=>{document.body.classList.toggle("light"); themeBtn.textContent=document.body.classList.contains("light")?"☀️":"🌙";};

// --- スキルボタン ---
function updateSkillButtons(){
  xBlockB.textContent=skill.X.block; xOverrideB.textContent=skill.X.override; xProtectB.textContent=skill.X.protect;
  oBlockB.textContent=skill.O.block; oOverrideB.textContent=skill.O.override; oProtectB.textContent=skill.O.protect;

  [xBlock,xOverride,xProtect,oBlock,oOverride,oProtect].forEach(btn=>btn.disabled=false);
  if(skill.X.block===0)xBlock.disabled=true; if(skill.X.override===0)xOverride.disabled=true; if(skill.X.protect===0)xProtect.disabled=true;
  if(skill.O.block===0)oBlock.disabled=true; if(skill.O.override===0)oOverride.disabled=true; if(skill.O.protect===0)oProtect.disabled=true;
}

function clearActive(){ [xBlock,xOverride,xProtect,oBlock,oOverride,oProtect].forEach(b=>b.classList.remove("active")); }

const xBlock=document.getElementById("x-block");
const xOverride=document.getElementById("x-override");
const xProtect=document.getElementById("x-protect");
const oBlock=document.getElementById("o-block");
const oOverride=document.getElementById("o-override");
const oProtect=document.getElementById("o-protect");

const xBlockB=document.getElementById("x-block-b");
const xOverrideB=document.getElementById("x-override-b");
const xProtectB=document.getElementById("x-protect-b");
const oBlockB=document.getElementById("o-block-b");
const oOverrideB=document.getElementById("o-override-b");
const oProtectB=document.getElementById("o-protect-b");

function bind(btn,p,t){
  btn.onclick=()=>{
    if(gameOver||phase==="mine")return;
    if(current!==p)return;
    if(usedSkillThisTurn)return;
    if(skill[p][t]<=0)return;
    skill[p].mode=(skill[p].mode===t)?null:t;
    clearActive();
    if(skill[p].mode===t)btn.classList.add("active");
  };
}

bind(xBlock,"X","block"); bind(xOverride,"X","override"); bind(xProtect,"X","protect");
bind(oBlock,"O","block"); bind(oOverride,"O","override"); bind(oProtect,"O","protect");

// --- リセット（確認付き） ---
document.getElementById("reset").onclick=()=>{
  if(confirm("本当にリセットしますか？")){
    skill={O:{block:1,override:1,protect:1,mode:null},X:{block:1,override:1,protect:1,mode:null}};
    init();
  }
};

// --- 初期化呼び出し ---
init();

// --- PWA Service Worker ---
if ("serviceWorker" in navigator) {
  window.addEventListener("load", function () {
    navigator.serviceWorker.register("./service-worker.js");
  });
}
</script>

</body>
</html>


