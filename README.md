[MIGONGh.html](https://github.com/user-attachments/files/33615736/MIGONGh.html)
# -<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<title>迷宫线条 · Neon Maze</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box}
  html,body{height:100%;overflow:hidden}
  body{
    background:radial-gradient(circle at 50% -10%,#16203a 0%,#060812 65%);
    color:#cfe3ff;
    font-family:ui-sans-serif,system-ui,-apple-system,"PingFang SC","Microsoft YaHei",sans-serif;
    user-select:none;-webkit-user-select:none;
    -webkit-tap-highlight-color:transparent;
  }
  #app{position:fixed;inset:0}
  .screen{position:absolute;inset:0;display:none;flex-direction:column;overflow:hidden}
  .screen.active{display:flex}

  /* ============ 菜单 ============ */
  #menuScreen{align-items:center;justify-content:center;padding:26px;overflow-y:auto}
  .brand{
    font-size:30px;font-weight:800;letter-spacing:4px;
    background:linear-gradient(90deg,#5fe6ff,#a78bfa,#ff5fa2);
    -webkit-background-clip:text;background-clip:text;color:transparent;
    text-align:center;
  }
  .brandSub{font-size:13px;color:#6d86ab;letter-spacing:2px;margin-top:8px;text-align:center}
  .diffGrid{
    display:grid;grid-template-columns:1fr 1fr;gap:14px;
    width:min(520px,100%);margin-top:28px;
  }
  .diffCard{
    position:relative;text-align:left;padding:16px 16px 14px;border-radius:16px;cursor:pointer;
    background:linear-gradient(160deg,rgba(30,45,80,.85),rgba(10,16,30,.9));
    border:1px solid rgba(95,230,255,.18);
    color:inherit;font-family:inherit;
    transition:transform .16s cubic-bezier(.2,1.3,.4,1),border-color .2s,box-shadow .2s;
    overflow:hidden;
  }
  .diffCard::before{
    content:'';position:absolute;left:0;top:0;bottom:0;width:4px;
    background:var(--c,#5fe6ff);opacity:.85;
  }
  .diffCard:active{transform:scale(.97)}
  @media(hover:hover){
    .diffCard:hover{transform:translateY(-3px);border-color:var(--c);box-shadow:0 8px 30px -8px var(--c)}
  }
  .dcName{font-size:18px;font-weight:700;color:var(--c);letter-spacing:2px}
  .dcMeta{font-size:11px;color:#7d96ba;margin-top:6px;letter-spacing:.5px}
  .dcProg{
    font-size:11px;margin-top:10px;color:#93a9c9;
    display:inline-flex;align-items:center;gap:6px;
  }
  .dcProg b{color:#e8f4ff;font-size:13px}
  .dcBar{
    height:3px;border-radius:3px;background:rgba(255,255,255,.08);
    margin-top:9px;overflow:hidden;
  }
  .dcBarFill{height:100%;border-radius:3px;background:var(--c);transition:width .4s}

  /* ============ 选关 ============ */
  #levelScreen{padding:0}
  .lvHeader{
    display:flex;align-items:center;gap:12px;
    padding:18px 18px 12px;flex:0 0 auto;
  }
  .iconBtn{
    width:38px;height:38px;flex:0 0 auto;border-radius:12px;border:1px solid rgba(120,170,230,.22);
    background:rgba(30,45,80,.6);color:#bcd6f5;font-size:17px;cursor:pointer;
    display:flex;align-items:center;justify-content:center;font-family:inherit;
    transition:background .15s,transform .12s;
  }
  .iconBtn:active{transform:scale(.92);background:rgba(60,90,140,.7)}
  .lvTitle{font-size:17px;font-weight:700;letter-spacing:1px}
  .lvTitle span{font-size:11px;color:#6d86ab;font-weight:400;margin-left:8px;letter-spacing:.5px}
  .levelGrid{
    flex:1 1 auto;min-height:0;overflow-y:auto;
    display:grid;grid-template-columns:repeat(auto-fill,minmax(54px,1fr));
    gap:9px;padding:6px 18px 32px;align-content:start;
    -webkit-overflow-scrolling:touch;
  }
  .levelBtn{
    aspect-ratio:1/1;border-radius:13px;cursor:pointer;font-family:inherit;
    font-size:15px;font-weight:600;color:#cfe3ff;
    background:linear-gradient(160deg,rgba(45,70,115,.75),rgba(18,28,48,.9));
    border:1px solid rgba(120,180,255,.22);
    transition:transform .12s,border-color .15s,box-shadow .15s;
    display:flex;align-items:center;justify-content:center;
  }
  .levelBtn:active{transform:scale(.93)}
  .levelBtn.done{
    color:#041018;
    background:linear-gradient(150deg,#5fe6ff,#7c8cff);
    border-color:transparent;box-shadow:0 0 16px -4px rgba(95,230,255,.7);
  }
  .levelBtn.locked{
    color:#3b4d68;background:rgba(16,24,40,.6);
    border-color:rgba(90,120,170,.10);cursor:not-allowed;
    box-shadow:none;
  }
  .levelBtn.locked:active{transform:none}

  /* ============ 游戏 ============ */
  #hud{
    display:flex;justify-content:space-between;align-items:center;gap:10px;
    padding:12px 16px 4px;font-size:12px;letter-spacing:.5px;
    color:#7f9bc4;flex:0 0 auto;
  }
  #hud b{color:#5fe6ff;font-weight:600;font-size:14px}
  #hint{color:#9fb6d8;font-size:12px;transition:color .2s;text-align:right}
  #hint.warn{color:#ffd166}
  #wrap{flex:1 1 auto;position:relative;min-height:0;overflow:hidden}
  canvas{display:block;position:absolute;inset:0;touch-action:none}
  .tip{
    position:absolute;bottom:8px;left:0;right:0;text-align:center;
    font-size:11px;color:#3f577a;letter-spacing:.5px;pointer-events:none;
  }

  /* ============ 弹窗 ============ */
  #overlay{
    position:fixed;inset:0;display:flex;align-items:center;justify-content:center;
    background:rgba(4,7,15,.82);backdrop-filter:blur(6px);
    opacity:0;pointer-events:none;transition:opacity .3s;z-index:10;padding:20px;
  }
  #overlay.show{opacity:1;pointer-events:auto}
  .panel{
    text-align:center;padding:30px 32px;border-radius:20px;max-width:340px;width:100%;
    background:linear-gradient(160deg,rgba(30,45,80,.94),rgba(12,18,34,.97));
    border:1px solid rgba(95,230,255,.25);
    box-shadow:0 0 60px rgba(0,200,255,.15),inset 0 0 40px rgba(0,150,255,.06);
    transform:translateY(12px) scale(.96);
    transition:transform .32s cubic-bezier(.2,1.3,.4,1);
  }
  #overlay.show .panel{transform:translateY(0) scale(1)}
  .panel h1{
    font-size:24px;margin-bottom:10px;letter-spacing:2px;font-weight:700;
    background:linear-gradient(90deg,#5fe6ff,#a78bfa,#ff5fa2);
    -webkit-background-clip:text;background-clip:text;color:transparent;
  }
  .panel p{font-size:13px;color:#8fa8c9;margin-bottom:22px;line-height:1.7}
  .ovBtns{display:flex;flex-direction:column;gap:10px}
  .panel button{
    padding:12px 20px;border-radius:999px;border:none;cursor:pointer;
    font-size:15px;font-weight:600;letter-spacing:1px;font-family:inherit;
    transition:transform .15s,box-shadow .15s;
  }
  .panel button:active{transform:scale(.96)}
  .btnPrimary{
    color:#041018;background:linear-gradient(135deg,#5fe6ff,#7c8cff);
    box-shadow:0 0 24px rgba(95,230,255,.4);
  }
  .btnGhost{
    color:#9fb6d8;background:rgba(60,90,140,.28);
    border:1px solid rgba(120,170,230,.2);
  }
</style>
</head>
<body>

<div id="app">

  <!-- ============ 难度菜单 ============ -->
  <div id="menuScreen" class="screen active">
    <div class="brand">迷宫线条</div>
    <div class="brandSub">NEON MAZE · 选择难度</div>
    <div class="diffGrid" id="diffGrid"></div>
  </div>

  <!-- ============ 关卡选择 ============ -->
  <div id="levelScreen" class="screen">
    <div class="lvHeader">
      <button class="iconBtn" id="lvBack">←</button>
      <div class="lvTitle" id="lvTitle">简单<span>选择关卡</span></div>
    </div>
    <div class="levelGrid" id="levelGrid"></div>
  </div>

  <!-- ============ 游戏 ============ -->
  <div id="gameScreen" class="screen">
    <div id="hud">
      <button class="iconBtn" id="gameBack">←</button>
      <div id="hint" style="flex:1;text-align:center">准备开始…</div>
      <div>步数 <b id="steps">0</b></div>
    </div>
    <div id="wrap"><canvas id="game"></canvas></div>
    <div class="tip">方向键 / WASD · 或点击滑动选择岔路</div>
  </div>

</div>

<!-- ============ 通关弹窗 ============ -->
<div id="overlay">
  <div class="panel">
    <h1 id="ovTitle">通关！</h1>
    <p id="ovText"></p>
    <div class="ovBtns">
      <button id="ovMain" class="btnPrimary">下一关</button>
      <button id="ovBack" class="btnGhost">返回选关</button>
    </div>
  </div>
</div>

<script>
(function(){
'use strict';

/* ============================================================
   难度与关卡配置
   ============================================================ */
const TOTAL_LEVELS = 32;

const DIFFICULTIES = [
  { name:'简单', color:'#5fe6ff', baseCols:7,  maxCols:14,
    baseSpeed:0.130, minSpeed:0.100, speedStep:0.0010,
    desc:'7×7 起 · 节奏舒缓' },
  { name:'普通', color:'#7c8cff', baseCols:9,  maxCols:16,
    baseSpeed:0.120, minSpeed:0.092, speedStep:0.0010,
    desc:'9×9 起 · 标准节奏' },
  { name:'困难', color:'#c084fc', baseCols:11, maxCols:18,
    baseSpeed:0.110, minSpeed:0.085, speedStep:0.0009,
    desc:'11×11 起 · 岔路密集' },
  { name:'地狱', color:'#ff5fa2', baseCols:13, maxCols:20,
    baseSpeed:0.100, minSpeed:0.078, speedStep:0.0008,
    desc:'13×13 起 · 极速挑战' },
];

/* 每 4 关迷宫边长 +1 */
function levelConfig(diffIdx, levelIdx){
  const cfg = DIFFICULTIES[diffIdx];
  const growth = Math.floor(levelIdx / 4);
  const cols = Math.min(cfg.baseCols + growth, cfg.maxCols);
  const speed = Math.max(cfg.minSpeed, cfg.baseSpeed - levelIdx * cfg.speedStep);
  return { cols, rows: cols, speed };
}

/* ============================================================
   进度存储
   ============================================================ */
const STORAGE_KEY = 'neonMaze.progress.v1';
let progress = [1, 1, 1, 1];   // 每个难度已解锁到第几关（1 起始）

(function loadProgress(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return;
    const p = JSON.parse(raw);
    if (!Array.isArray(p)) return;
    for (let i = 0; i < 4; i++){
      if (typeof p[i] === 'number' && isFinite(p[i])){
        progress[i] = Math.max(1, Math.min(TOTAL_LEVELS, Math.floor(p[i])));
      }
    }
  }catch(e){ /* 忽略 */ }
})();

function saveProgress(){
  try{ localStorage.setItem(STORAGE_KEY, JSON.stringify(progress)); }catch(e){}
}

/* ============================================================
   基础常量
   ============================================================ */
const DIRS = [
  {dc: 0, dr: -1},   // 0 上
  {dc: 1, dr:  0},   // 1 右
  {dc: 0, dr:  1},   // 2 下
  {dc:-1, dr:  0},   // 3 左
];
const OPP = [2, 3, 0, 1];

/* ============================================================
   DOM
   ============================================================ */
const menuScreen  = document.getElementById('menuScreen');
const levelScreen = document.getElementById('levelScreen');
const gameScreen  = document.getElementById('gameScreen');
const diffGrid    = document.getElementById('diffGrid');
const levelGrid   = document.getElementById('levelGrid');
const lvTitle     = document.getElementById('lvTitle');
const lvBack      = document.getElementById('lvBack');
const gameBack    = document.getElementById('gameBack');
const canvas      = document.getElementById('game');
const ctx         = canvas.getContext('2d');
const wrap        = document.getElementById('wrap');
const hintEl      = document.getElementById('hint');
const stepsEl     = document.getElementById('steps');
const overlay     = document.getElementById('overlay');
const ovTitle     = document.getElementById('ovTitle');
const ovText      = document.getElementById('ovText');
const ovMain      = document.getElementById('ovMain');
const ovBack      = document.getElementById('ovBack');

/* ============================================================
   全局状态
   ============================================================ */
let screen = 'menu';          // menu | level | game
let currentDiff  = 0;
let currentLevel = 0;         // 0 起始
let DPR = 1;

const game = {
  maze: null,
  W: 0, H: 0,
  cellSize: 20, ox: 0, oy: 0,
  pathCells: [],
  pos: {c: 0, r: 0},
  moving: null,
  mode: 'idle',               // idle | moving | won
  options: [],
  autoDir: -1,
  autoTimer: 0,
  lastDir: -1,
  shake: 0,
  time: 0,
  steps: 0,
  speed: 0.11,
};

/* ============================================================
   工具
   ============================================================ */
function gridToPixel(gc, gr){
  const cs = game.cellSize;
  return {
    x: game.ox + gc * cs + cs / 2,
    y: game.oy + gr * cs + cs / 2,
  };
}
function dirBetween(a, b){
  const dc = b.c - a.c, dr = b.r - a.r;
  for (let d = 0; d < 4; d++) if (DIRS[d].dc === dc && DIRS[d].dr === dr) return d;
  return 0;
}
function setHint(text, warn){
  hintEl.textContent = text;
  hintEl.classList.toggle('warn', !!warn);
}

/* ============================================================
   迷宫生成（递归回溯）
   ============================================================ */
function generateMaze(cols, rows){
  const walls = Array.from({length: rows}, () =>
    Array.from({length: cols}, () => [true, true, true, true]));
  const visited = Array.from({length: rows}, () => new Array(cols).fill(false));

  const sc = (Math.random() * cols) | 0;
  const sr = (Math.random() * rows) | 0;
  const stack = [{c: sc, r: sr}];
  visited[sr][sc] = true;
  let count = 1;
  const total = cols * rows;

  while (count < total && stack.length){
    const cur = stack[stack.length - 1];
    const nb = [];
    for (let d = 0; d < 4; d++){
      const nc = cur.c + DIRS[d].dc, nr = cur.r + DIRS[d].dr;
      if (nc >= 0 && nc < cols && nr >= 0 && nr < rows && !visited[nr][nc]){
        nb.push({c: nc, r: nr, d});
      }
    }
    if (nb.length === 0){ stack.pop(); continue; }
    const p = nb[(Math.random() * nb.length) | 0];
    walls[cur.r][cur.c][p.d] = false;
    walls[p.r][p.c][OPP[p.d]] = false;
    visited[p.r][p.c] = true;
    stack.push({c: p.c, r: p.r});
    count++;
  }
  return {cols, rows, walls};
}

/* ============================================================
   BFS
   ============================================================ */
function bfsDistances(maze, start){
  const {cols, rows, walls} = maze;
  const dist = Array.from({length: rows}, () => new Array(cols).fill(-1));
  dist[start.r][start.c] = 0;
  const q = [start];
  let head = 0;
  while (head < q.length){
    const cur = q[head++];
    for (let d = 0; d < 4; d++){
      if (walls[cur.r][cur.c][d]) continue;
      const nc = cur.c + DIRS[d].dc, nr = cur.r + DIRS[d].dr;
      if (nc < 0 || nc >= cols || nr < 0 || nr >= rows) continue;
      if (dist[nr][nc] !== -1) continue;
      dist[nr][nc] = dist[cur.r][cur.c] + 1;
      q.push({c: nc, r: nr});
    }
  }
  return dist;
}

/* 随机起点 + 最远终点 */
function pickStartEnd(maze){
  let best = null;
  for (let attempt = 0; attempt < 14; attempt++){
    const s = {
      c: (Math.random() * maze.cols) | 0,
      r: (Math.random() * maze.rows) | 0,
    };
    const dist = bfsDistances(maze, s);
    let maxD = -1, far = s;
    for (let r = 0; r < maze.rows; r++){
      for (let c = 0; c < maze.cols; c++){
        if (dist[r][c] > maxD){ maxD = dist[r][c]; far = {c, r}; }
      }
    }
    if (!best || maxD > best.maxD) best = {start: s, end: far, maxD};
  }
  return {start: best.start, end: best.end};
}

/* ============================================================
   画布布局
   ============================================================ */
function layout(){
  if (!game.maze) return;
  const rect = wrap.getBoundingClientRect();
  const W = Math.max(rect.width, 10);
  const H = Math.max(rect.height, 10);
  if (W < 10 || H < 10) return;

  DPR = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width  = Math.round(W * DPR);
  canvas.height = Math.round(H * DPR);
  canvas.style.width  = W + 'px';
  canvas.style.height = H + 'px';

  game.W = W; game.H = H;

  const pad = 16;
  const cs = Math.min((W - pad * 2) / game.maze.cols, (H - pad * 2) / game.maze.rows);
  game.cellSize = cs;
  game.ox = (W - cs * game.maze.cols) / 2;
  game.oy = (H - cs * game.maze.rows) / 2;
}

/* ============================================================
   震动
   ============================================================ */
function triggerShake(strength){
  strength = strength || 1;
  game.shake = Math.max(game.shake, 9 * strength);
  if (navigator.vibrate){
    try{ navigator.vibrate(strength > 1.5 ? [60, 50, 90] : 22); }catch(e){}
  }
}

/* ============================================================
   界面切换
   ============================================================ */
function showScreen(name){
  screen = name;
  menuScreen.classList.toggle('active', name === 'menu');
  levelScreen.classList.toggle('active', name === 'level');
  gameScreen.classList.toggle('active', name === 'game');
  if (name === 'game') requestAnimationFrame(layout);
}

/* ============================================================
   难度菜单
   ============================================================ */
function renderMenu(){
  diffGrid.innerHTML = '';
  DIFFICULTIES.forEach((cfg, i) => {
    const unlocked = progress[i];
    const card = document.createElement('button');
    card.className = 'diffCard';
    card.style.setProperty('--c', cfg.color);
    card.innerHTML = `
      <div class="dcName">${cfg.name}</div>
      <div class="dcMeta">${cfg.desc}</div>
      <div class="dcProg">进度 <b>${unlocked}</b> / ${TOTAL_LEVELS}</div>
      <div class="dcBar"><div class="dcBarFill" style="width:${(unlocked / TOTAL_LEVELS) * 100}%"></div></div>
    `;
    card.addEventListener('click', () => showLevelSelect(i));
    diffGrid.appendChild(card);
  });
}

/* ============================================================
   关卡选择
   ============================================================ */
function showLevelSelect(diffIdx){
  const cfg = DIFFICULTIES[diffIdx];
  lvTitle.innerHTML = `${cfg.name}<span>选择关卡 · 已解锁 ${progress[diffIdx]}/${TOTAL_LEVELS}</span>`;
  lvTitle.style.color = cfg.color;

  levelGrid.innerHTML = '';
  const maxUnlocked = progress[diffIdx];

  for (let i = 0; i < TOTAL_LEVELS; i++){
    const lvNum = i + 1;
    const unlocked = lvNum <= maxUnlocked;
    const btn = document.createElement('button');
    btn.className = 'levelBtn';
    btn.textContent = lvNum;

    if (!unlocked){
      btn.classList.add('locked');
      btn.disabled = true;
      btn.textContent = '🔒';
    } else if (lvNum < maxUnlocked){
      btn.classList.add('done');
    }

    if (unlocked){
      btn.addEventListener('click', () => enterGame(diffIdx, i));
    }
    levelGrid.appendChild(btn);
  }

  showScreen('level');
  // 滚动到最新解锁的关卡附近
  requestAnimationFrame(() => {
    const idx = Math.min(maxUnlocked - 1, TOTAL_LEVELS - 1);
    const target = levelGrid.children[idx];
    if (target){
      const top = target.offsetTop - levelGrid.clientHeight / 2 + target.offsetHeight / 2;
      levelGrid.scrollTop = Math.max(0, top);
    }
  });
}

/* ============================================================
   进入游戏
   ============================================================ */
function enterGame(diffIdx, levelIdx){
  currentDiff = diffIdx;
  currentLevel = levelIdx;
  showScreen('game');
  startLevel();
  requestAnimationFrame(layout);
}

/* ============================================================
   关卡初始化
   ============================================================ */
function startLevel(){
  const cfg = levelConfig(currentDiff, currentLevel);
  const maze = generateMaze(cfg.cols, cfg.rows);
  const se = pickStartEnd(maze);
  maze.start = se.start;
  maze.end = se.end;

  game.maze = maze;
  game.speed = cfg.speed;
  game.pathCells = [{c: se.start.c, r: se.start.r}];
  game.pos = {c: se.start.c, r: se.start.r};
  game.moving = null;
  game.mode = 'idle';
  game.options = [];
  game.autoDir = -1;
  game.autoTimer = 0;
  game.lastDir = -1;
  game.shake = 0;
  game.time = 0;
  game.steps = 0;

  stepsEl.textContent = '0';
  overlay.classList.remove('show');
  setHint('自动前进中…');
  resolveOptions(se.start, -1);
}

/* ============================================================
   方向解析
   ============================================================ */
function resolveOptions(cell, fromDir){
  const walls = game.maze.walls[cell.r][cell.c];
  const back = fromDir < 0 ? -1 : OPP[fromDir];
  const opts = [];
  for (let d = 0; d < 4; d++){
    if (!walls[d] && d !== back) opts.push(d);
  }

  if (opts.length === 0){
    if (game.pathCells.length <= 1){
      game.mode = 'idle';
      game.options = [];
      setHint('这里没有出路了', true);
      return;
    }
    setHint('死路 · 自动回溯', true);
    startBacktrack();
    return;
  }

  if (opts.length === 1){
    game.mode = 'idle';
    game.options = [];
    game.autoDir = opts[0];
    game.autoTimer = 0.07;
    return;
  }

  game.mode = 'idle';
  game.options = opts;
  setHint('岔路 · 选择方向', true);
}

/* ============================================================
   移动
   ============================================================ */
function beginMove(dir){
  const cur = game.pathCells[game.pathCells.length - 1];
  const nc = cur.c + DIRS[dir].dc;
  const nr = cur.r + DIRS[dir].dr;
  const isTurn = game.lastDir >= 0 && game.lastDir !== dir;

  game.moving = {
    fromC: game.pos.c, fromR: game.pos.r,
    toC: nc, toR: nr,
    dir: dir,
    t: 0,
    dur: game.speed,
    backtrack: false,
  };
  game.mode = 'moving';
  game.options = [];
  game.autoDir = -1;
  game.steps++;
  stepsEl.textContent = game.steps;

  if (isTurn) triggerShake(1);
  game.lastDir = dir;

  setHint('自动前进中…');
}

function startBacktrack(){
  if (game.pathCells.length <= 1){
    game.mode = 'idle';
    setHint('无路可退', true);
    return;
  }
  const dead = game.pathCells.pop();
  const prev = game.pathCells[game.pathCells.length - 1];
  const dir = dirBetween(dead, prev);

  game.moving = {
    fromC: game.pos.c, fromR: game.pos.r,
    toC: prev.c, toR: prev.r,
    dir: dir,
    t: 0,
    dur: game.speed * 0.75,
    backtrack: true,
  };
  game.mode = 'moving';
  game.lastDir = dir;
}

/* ============================================================
   抵达格子
   ============================================================ */
function onArrive(cell, dir){
  if (cell.c === game.maze.end.c && cell.r === game.maze.end.r){
    winLevel();
    return;
  }

  const walls = game.maze.walls[cell.r][cell.c];
  const back = OPP[dir];
  const opts = [];
  for (let d = 0; d < 4; d++){
    if (!walls[d] && d !== back) opts.push(d);
  }

  if (opts.length === 0){
    if (game.pathCells.length <= 1){
      game.mode = 'idle';
      setHint('这里没有出路了', true);
      return;
    }
    setHint('死路 · 自动回溯', true);
    startBacktrack();
    return;
  }

  if (opts.length === 1){
    game.mode = 'idle';
    game.autoDir = opts[0];
    game.autoTimer = 0.07;
    return;
  }

  game.mode = 'idle';
  game.options = opts;
  setHint('岔路 · 选择方向', true);
}

/* ============================================================
   通关
   ============================================================ */
function winLevel(){
  game.mode = 'won';
  game.options = [];
  game.moving = null;
  game.autoDir = -1;
  triggerShake(2.5);
  setHint('抵达终点！');

  const lvIndex = currentLevel;
  if (lvIndex + 1 < TOTAL_LEVELS && progress[currentDiff] < lvIndex + 2){
    progress[currentDiff] = lvIndex + 2;
    saveProgress();
  }

  setTimeout(() => {
    const isLast = lvIndex + 1 >= TOTAL_LEVELS;
    ovTitle.textContent = isLast ? '难度通关！' : '关卡通过！';
    ovText.textContent = isLast
      ? `你完成了「${DIFFICULTIES[currentDiff].name}」全部 ${TOTAL_LEVELS} 关，共 ${game.steps} 步。`
      : `本关用了 ${game.steps} 步，下一关已解锁。`;
    ovMain.textContent = isLast ? '返回选关' : '下一关';
    ovMain.dataset.act = isLast ? 'back' : 'next';
    overlay.classList.add('show');
  }, 420);
}

ovMain.addEventListener('click', () => {
  overlay.classList.remove('show');
  const act = ovMain.dataset.act;
  if (act === 'next' && currentLevel + 1 < TOTAL_LEVELS){
    setTimeout(() => enterGame(currentDiff, currentLevel + 1), 120);
  } else {
    setTimeout(() => showLevelSelect(currentDiff), 120);
  }
});

ovBack.addEventListener('click', () => {
  overlay.classList.remove('show');
  setTimeout(() => showLevelSelect(currentDiff), 120);
});

/* ============================================================
   更新
   ============================================================ */
function update(dt){
  game.time += dt;

  if (game.shake > 0){
    game.shake = Math.max(0, game.shake - dt * 46);
  }

  if (game.mode === 'moving' && game.moving){
    const m = game.moving;
    m.t += dt / m.dur;

    if (m.t >= 1){
      game.pos.c = m.toC;
      game.pos.r = m.toR;
      if (!m.backtrack){
        game.pathCells.push({c: m.toC, r: m.toR});
      }
      const arrivedDir = m.dir;
      const cell = {c: m.toC, r: m.toR};
      game.moving = null;
      game.mode = 'idle';
      onArrive(cell, arrivedDir);
    } else {
      const e = m.t * m.t * (3 - 2 * m.t);
      game.pos.c = m.fromC + (m.toC - m.fromC) * e;
      game.pos.r = m.fromR + (m.toR - m.fromR) * e;
    }
    return;
  }

  if (game.autoDir >= 0 && game.mode === 'idle'){
    game.autoTimer -= dt;
    if (game.autoTimer <= 0){
      const d = game.autoDir;
      game.autoDir = -1;
      beginMove(d);
    }
  }
}

/* ============================================================
   渲染
   ============================================================ */
function render(){
  if (!game.maze) return;
  const W = game.W, H = game.H;
  if (W < 10 || H < 10) return;

  let sx = 0, sy = 0;
  if (game.shake > 0.3){
    const s = game.shake;
    sx = (Math.random() - 0.5) * s;
    sy = (Math.random() - 0.5) * s;
  }

  ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
  ctx.clearRect(0, 0, W, H);

  const bg = ctx.createRadialGradient(W / 2, 0, 0, W / 2, 0, Math.max(W, H) * 1.1);
  bg.addColorStop(0, '#111d35');
  bg.addColorStop(1, '#05070f');
  ctx.fillStyle = bg;
  ctx.fillRect(0, 0, W, H);

  ctx.save();
  ctx.translate(sx, sy);

  drawMaze();
  drawPath();
  drawEndpoints();
  drawOptions();

  ctx.restore();
}

function drawMaze(){
  const {cols, rows, walls} = game.maze;
  const cs = game.cellSize;

  ctx.save();
  ctx.strokeStyle = 'rgba(86, 140, 210, 0.30)';
  ctx.lineWidth = Math.max(1, cs * 0.075);
  ctx.lineCap = 'round';
  ctx.beginPath();

  for (let r = 0; r < rows; r++){
    for (let c = 0; c < cols; c++){
      const x = game.ox + c * cs;
      const y = game.oy + r * cs;
      const w = walls[r][c];
      if (w[0]){ ctx.moveTo(x, y);          ctx.lineTo(x + cs, y); }
      if (w[3]){ ctx.moveTo(x, y);          ctx.lineTo(x, y + cs); }
      if (r === rows - 1 && w[2]){ ctx.moveTo(x, y + cs); ctx.lineTo(x + cs, y + cs); }
      if (c === cols - 1 && w[1]){ ctx.moveTo(x + cs, y); ctx.lineTo(x + cs, y + cs); }
    }
  }
  ctx.stroke();
  ctx.restore();
}

function drawPath(){
  const pts = game.pathCells.map(c => gridToPixel(c.c, c.r));
  pts.push(gridToPixel(game.pos.c, game.pos.r));
  if (pts.length < 2) return;

  const cs = game.cellSize;

  ctx.save();
  ctx.lineJoin = 'round';
  ctx.lineCap  = 'round';

  ctx.beginPath();
  ctx.moveTo(pts[0].x, pts[0].y);
  for (let i = 1; i < pts.length; i++) ctx.lineTo(pts[i].x, pts[i].y);
  ctx.strokeStyle = 'rgba(0, 220, 255, 0.20)';
  ctx.lineWidth = Math.max(7, cs * 0.46);
  ctx.shadowColor = '#00e5ff';
  ctx.shadowBlur = 20;
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(pts[0].x, pts[0].y);
  for (let i = 1; i < pts.length; i++) ctx.lineTo(pts[i].x, pts[i].y);
  ctx.strokeStyle = '#37d8ff';
  ctx.lineWidth = Math.max(2, cs * 0.16);
  ctx.shadowColor = '#00d0ff';
  ctx.shadowBlur = 14;
  ctx.stroke();

  const head = pts[pts.length - 1];
  const pulse = 0.85 + 0.15 * Math.sin(game.time * 8);
  ctx.beginPath();
  ctx.arc(head.x, head.y, Math.max(3, cs * 0.17) * pulse, 0, Math.PI * 2);
  ctx.fillStyle = '#ffffff';
  ctx.shadowColor = '#5fe6ff';
  ctx.shadowBlur = 22;
  ctx.fill();

  ctx.restore();
}

function drawEndpoints(){
  const cs = game.cellSize;
  const s = gridToPixel(game.maze.start.c, game.maze.start.r);
  const e = gridToPixel(game.maze.end.c, game.maze.end.r);

  ctx.save();
  ctx.beginPath();
  ctx.arc(s.x, s.y, cs * 0.20, 0, Math.PI * 2);
  ctx.fillStyle = 'rgba(90, 200, 255, 0.30)';
  ctx.fill();
  ctx.restore();

  const pulse = 0.5 + 0.5 * Math.sin(game.time * 3.2);
  ctx.save();
  ctx.beginPath();
  ctx.arc(e.x, e.y, cs * (0.24 + pulse * 0.10), 0, Math.PI * 2);
  ctx.fillStyle = 'rgba(255, 70, 165, 0.22)';
  ctx.shadowColor = '#ff3ca0';
  ctx.shadowBlur = 26;
  ctx.fill();

  ctx.beginPath();
  ctx.arc(e.x, e.y, cs * 0.15, 0, Math.PI * 2);
  ctx.fillStyle = '#ff3ca0';
  ctx.shadowBlur = 18;
  ctx.fill();
  ctx.restore();
}

function drawOptions(){
  if (game.mode !== 'idle' || game.options.length === 0) return;

  const cs = game.cellSize;
  const cur = game.pathCells[game.pathCells.length - 1];
  const head = gridToPixel(cur.c, cur.r);
  const pulse = 0.55 + 0.45 * Math.sin(game.time * 5);

  for (let i = 0; i < game.options.length; i++){
    const d = game.options[i];
    const nc = cur.c + DIRS[d].dc;
    const nr = cur.r + DIRS[d].dr;
    const p = gridToPixel(nc, nr);
    const mx = (head.x + p.x) / 2;
    const my = (head.y + p.y) / 2;

    ctx.save();
    ctx.globalAlpha = 0.35 + pulse * 0.5;

    ctx.beginPath();
    ctx.moveTo(head.x, head.y);
    ctx.lineTo(p.x, p.y);
    ctx.setLineDash([cs * 0.16, cs * 0.13]);
    ctx.strokeStyle = '#ffd166';
    ctx.lineWidth = Math.max(2, cs * 0.09);
    ctx.lineCap = 'round';
    ctx.stroke();
    ctx.setLineDash([]);

    ctx.beginPath();
    ctx.arc(mx, my, cs * 0.10, 0, Math.PI * 2);
    ctx.fillStyle = '#ffd166';
    ctx.shadowColor = '#ffd166';
    ctx.shadowBlur = 16;
    ctx.fill();
    ctx.restore();
  }
}

/* ============================================================
   主循环
   ============================================================ */
let lastTime = 0;
function loop(ts){
  if (!lastTime) lastTime = ts;
  let dt = (ts - lastTime) / 1000;
  lastTime = ts;
  if (dt > 0.05) dt = 0.05;

  if (screen === 'game'){
    update(dt);
    render();
  }
  requestAnimationFrame(loop);
}

/* ============================================================
   输入
   ============================================================ */
function handleDirInput(d){
  if (screen !== 'game') return;
  if (game.mode !== 'idle') return;
  if (game.options.indexOf(d) >= 0) beginMove(d);
}

const KEYMAP = {
  ArrowUp: 0, ArrowRight: 1, ArrowDown: 2, ArrowLeft: 3,
  w: 0, d: 1, s: 2, a: 3,
  W: 0, D: 1, S: 2, A: 3,
};

window.addEventListener('keydown', (e) => {
  if (e.key in KEYMAP){
    if (screen === 'game'){ e.preventDefault(); handleDirInput(KEYMAP[e.key]); }
  }
  if (e.key === 'Escape' && screen === 'game'){
    showLevelSelect(currentDiff);
  }
  if (e.key === 'Enter' && overlay.classList.contains('show')){
    e.preventDefault();
    ovMain.click();
  }
}, {passive: false});

let pointerStart = null;

canvas.addEventListener('pointerdown', (e) => {
  if (screen !== 'game') return;
  pointerStart = {x: e.clientX, y: e.clientY, t: performance.now()};
}, {passive: true});

canvas.addEventListener('pointerup', (e) => {
  if (!pointerStart || screen !== 'game') return;
  const dx = e.clientX - pointerStart.x;
  const dy = e.clientY - pointerStart.y;
  const dist = Math.hypot(dx, dy);
  const start = pointerStart;
  pointerStart = null;

  if (game.mode !== 'idle' || game.options.length === 0) return;

  if (dist > 22 && performance.now() - start.t < 900){
    let d;
    if (Math.abs(dx) > Math.abs(dy)) d = dx > 0 ? 1 : 3;
    else d = dy > 0 ? 2 : 0;
    handleDirInput(d);
    return;
  }

  const rect = canvas.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  const c = Math.floor((x - game.ox) / game.cellSize);
  const r = Math.floor((y - game.oy) / game.cellSize);
  const cur = game.pathCells[game.pathCells.length - 1];
  const dc = c - cur.c, dr = r - cur.r;
  for (let d = 0; d < 4; d++){
    if (DIRS[d].dc === dc && DIRS[d].dr === dr){
      handleDirInput(d);
      return;
    }
  }
}, {passive: true});

canvas.addEventListener('pointercancel', () => { pointerStart = null; }, {passive: true});

/* ============================================================
   返回按钮
   ============================================================ */
lvBack.addEventListener('click', () => { renderMenu(); showScreen('menu'); });

gameBack.addEventListener('click', () => {
  game.mode = 'idle';
  game.moving = null;
  game.autoDir = -1;
  overlay.classList.remove('show');
  showLevelSelect(currentDiff);
});

/* ============================================================
   尺寸自适应
   ============================================================ */
let resizeRaf = 0;
function onResize(){
  if (screen !== 'game') return;
  if (resizeRaf) return;
  resizeRaf = requestAnimationFrame(() => {
    resizeRaf = 0;
    layout();
  });
}
window.addEventListener('resize', onResize);
window.addEventListener('orientationchange', () => {
  if (screen === 'game') setTimeout(layout, 240);
});

/* ============================================================
   启动
   ============================================================ */
renderMenu();
showScreen('menu');
requestAnimationFrame(loop);

})();
</script>
</body>
</html>
