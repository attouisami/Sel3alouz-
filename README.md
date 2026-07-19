I want to store files.
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>لعبة البحث عن الكلمات - قطع غيار</title>
<style>
  :root{
    --gold:#d4af37;
    --gold-light:#f0d97e;
    --bg:#0d0d0f;
    --panel:#1a1a1e;
    --panel-2:#232327;
    --text:#f2f2f2;
    --muted:#9a9a9a;
    --found:#2e7d32;
  }
  *{box-sizing:border-box;}
  body{
    margin:0;
    min-height:100vh;
    background:radial-gradient(circle at 50% 0%, #1c1c20 0%, var(--bg) 70%);
    color:var(--text);
    font-family:'Tahoma','Segoe UI',sans-serif;
    display:flex;
    flex-direction:column;
    align-items:center;
    padding:20px 12px 60px;
  }
  h1{
    color:var(--gold);
    font-size:1.6rem;
    margin:6px 0 2px;
    text-align:center;
    letter-spacing:1px;
  }
  .subtitle{
    color:var(--muted);
    font-size:.9rem;
    margin-bottom:18px;
    text-align:center;
  }
  .top-bar{
    display:flex;
    gap:14px;
    margin-bottom:16px;
    flex-wrap:wrap;
    justify-content:center;
  }
  .stat{
    background:var(--panel);
    border:1px solid #2c2c30;
    border-radius:10px;
    padding:8px 16px;
    font-size:.85rem;
    color:var(--gold-light);
    min-width:90px;
    text-align:center;
  }
  .stat b{
    display:block;
    font-size:1.1rem;
    color:var(--gold);
    margin-top:2px;
  }
  .game-wrap{
    display:flex;
    gap:24px;
    flex-wrap:wrap;
    justify-content:center;
    align-items:flex-start;
    width:100%;
    max-width:900px;
  }
  #grid{
    display:grid;
    gap:3px;
    background:var(--panel);
    padding:10px;
    border-radius:14px;
    border:1px solid #2c2c30;
    box-shadow:0 10px 30px rgba(0,0,0,.5);
    user-select:none;
    touch-action:none;
  }
  .cell{
    width:36px;
    height:36px;
    display:flex;
    align-items:center;
    justify-content:center;
    background:var(--panel-2);
    border-radius:6px;
    font-weight:bold;
    font-size:1rem;
    cursor:pointer;
    transition:background .15s, transform .1s;
  }
  .cell.selected{
    background:var(--gold);
    color:#111;
    transform:scale(1.05);
  }
  .cell.found{
    background:var(--found);
    color:#fff;
  }
  .word-panel{
    background:var(--panel);
    border:1px solid #2c2c30;
    border-radius:14px;
    padding:16px 20px;
    min-width:200px;
  }
  .word-panel h3{
    margin:0 0 10px;
    color:var(--gold);
    font-size:1rem;
    border-bottom:1px solid #2c2c30;
    padding-bottom:8px;
  }
  .word-list{
    list-style:none;
    padding:0;
    margin:0;
    display:flex;
    flex-direction:column;
    gap:8px;
  }
  .word-list li{
    font-size:.95rem;
    color:var(--text);
    padding:6px 10px;
    background:var(--panel-2);
    border-radius:8px;
    transition:.2s;
  }
  .word-list li.done{
    color:var(--muted);
    text-decoration:line-through;
    background:transparent;
    border:1px solid var(--found);
  }
  .win-msg{
    display:none;
    margin-top:16px;
    background:linear-gradient(135deg,var(--gold),var(--gold-light));
    color:#111;
    padding:12px 20px;
    border-radius:10px;
    font-weight:bold;
    text-align:center;
  }
  .win-msg.show{display:block;}
  button{
    margin-top:14px;
    background:transparent;
    border:1px solid var(--gold);
    color:var(--gold);
    padding:8px 18px;
    border-radius:8px;
    cursor:pointer;
    font-size:.9rem;
  }
  button:hover{background:var(--gold);color:#111;}
  footer{
    margin-top:30px;
    color:var(--muted);
    font-size:.75rem;
    text-align:center;
  }
</style>
</head>
<body>

<h1>🔧 بحث عن الكلمات - قطع غيار</h1>
<div class="subtitle">دور على أسماء قطع السيارات المخبّية في الشبكة</div>

<div class="top-bar">
  <div class="stat">الوقت<b id="timer">00:00</b></div>
  <div class="stat">لقيت<b id="foundCount">0 / 0</b></div>
</div>

<div class="game-wrap">
  <div id="grid"></div>
  <div class="word-panel">
    <h3>الكلمات</h3>
    <ul class="word-list" id="wordList"></ul>
    <div class="win-msg" id="winMsg">🎉 ربحت! لقيت كامل الكلمات</div>
    <button id="restartBtn">لعبة جديدة</button>
  </div>
</div>

<footer>لعبة بسيطة صنعت باش تترفع على GitHub</footer>

<script>
const WORDS = ["فرامل","محرك","بطارية","فلتر","إطار","مصباح","زيت","رادياتور","عجلة","ناقل"];
const SIZE = 12;
const ARABIC_LETTERS = "ابتثجحخدذرزسشصضطظعغفقكلمنهوي";

let grid = [];
let placedWords = [];
let foundWords = new Set();
let isSelecting = false;
let selectedCells = [];
let startTime = null;
let timerInterval = null;

const gridEl = document.getElementById('grid');
const wordListEl = document.getElementById('wordList');
const foundCountEl = document.getElementById('foundCount');
const timerEl = document.getElementById('timer');
const winMsgEl = document.getElementById('winMsg');

const DIRECTIONS = [
  {dr:0, dc:1}, {dr:0, dc:-1},
  {dr:1, dc:0}, {dr:-1, dc:0},
  {dr:1, dc:1}, {dr:-1, dc:-1},
  {dr:1, dc:-1}, {dr:-1, dc:1},
];

function emptyGrid(){
  return Array.from({length:SIZE}, () => Array(SIZE).fill(null));
}

function tryPlaceWord(word){
  const attempts = 100;
  for(let a=0; a<attempts; a++){
    const dir = DIRECTIONS[Math.floor(Math.random()*DIRECTIONS.length)];
    const row = Math.floor(Math.random()*SIZE);
    const col = Math.floor(Math.random()*SIZE);
    const endRow = row + dir.dr*(word.length-1);
    const endCol = col + dir.dc*(word.length-1);
    if(endRow < 0 || endRow >= SIZE || endCol < 0 || endCol >= SIZE) continue;

    let fits = true;
    const cells = [];
    for(let i=0; i<word.length; i++){
      const r = row + dir.dr*i;
      const c = col + dir.dc*i;
      const existing = grid[r][c];
      if(existing !== null && existing !== word[i]){ fits = false; break; }
      cells.push({r,c});
    }
    if(!fits) continue;

    cells.forEach((cell,i) => { grid[cell.r][cell.c] = word[i]; });
    return cells;
  }
  return null;
}

function fillEmptyCells(){
  for(let r=0; r<SIZE; r++){
    for(let c=0; c<SIZE; c++){
      if(grid[r][c] === null){
        grid[r][c] = ARABIC_LETTERS[Math.floor(Math.random()*ARABIC_LETTERS.length)];
      }
    }
  }
}

function buildGame(){
  grid = emptyGrid();
  placedWords = [];
  foundWords = new Set();

  const sorted = [...WORDS].sort((a,b) => b.length - a.length);
  sorted.forEach(word => {
    const cells = tryPlaceWord(word);
    if(cells) placedWords.push({word, cells});
  });
  fillEmptyCells();
  renderGrid();
  renderWordList();
  updateStats();
  startTimer();
  winMsgEl.classList.remove('show');
}

function renderGrid(){
  gridEl.innerHTML = '';
  gridEl.style.gridTemplateColumns = `repeat(${SIZE}, 36px)`;
  for(let r=0; r<SIZE; r++){
    for(let c=0; c<SIZE; c++){
      const div = document.createElement('div');
      div.className = 'cell';
      div.textContent = grid[r][c];
      div.dataset.r = r;
      div.dataset.c = c;
      div.addEventListener('mousedown', () => startSelect(r,c));
      div.addEventListener('mouseenter', () => moveSelect(r,c));
      div.addEventListener('touchstart', (e) => { e.preventDefault(); startSelect(r,c); });
      div.addEventListener('touchmove', (e) => {
        e.preventDefault();
        const touch = e.touches[0];
        const el = document.elementFromPoint(touch.clientX, touch.clientY);
        if(el && el.classList.contains('cell')){
          moveSelect(parseInt(el.dataset.r), parseInt(el.dataset.c));
        }
      });
      gridEl.appendChild(div);
    }
  }
  document.addEventListener('mouseup', endSelect);
  document.addEventListener('touchend', endSelect);
}

function renderWordList(){
  wordListEl.innerHTML = '';
  placedWords.forEach(({word}) => {
    const li = document.createElement('li');
    li.textContent = word;
    li.dataset.word = word;
    wordListEl.appendChild(li);
  });
}

function cellEl(r,c){
  return gridEl.querySelector(`[data-r="${r}"][data-c="${c}"]`);
}

function startSelect(r,c){
  isSelecting = true;
  selectedCells = [{r,c}];
  highlightSelection();
}

function moveSelect(r,c){
  if(!isSelecting) return;
  const start = selectedCells[0];
  const dr = r - start.r;
  const dc = c - start.c;
  const steps = Math.max(Math.abs(dr), Math.abs(dc));
  if(steps === 0){ selectedCells = [start]; highlightSelection(); return; }

  const stepR = dr === 0 ? 0 : dr / Math.abs(dr);
  const stepC = dc === 0 ? 0 : dc / Math.abs(dc);
  if(Math.abs(dr) !== 0 && Math.abs(dc) !== 0 && Math.abs(dr) !== Math.abs(dc)) return;

  const cells = [];
  for(let i=0; i<=steps; i++){
    cells.push({r: start.r + stepR*i, c: start.c + stepC*i});
  }
  selectedCells = cells;
  highlightSelection();
}

function highlightSelection(){
  document.querySelectorAll('.cell.selected').forEach(el => el.classList.remove('selected'));
  selectedCells.forEach(({r,c}) => {
    const el = cellEl(r,c);
    if(el && !el.classList.contains('found')) el.classList.add('selected');
  });
}

function endSelect(){
  if(!isSelecting) return;
  isSelecting = false;
  checkSelection();
  document.querySelectorAll('.cell.selected').forEach(el => el.classList.remove('selected'));
  selectedCells = [];
}

function checkSelection(){
  if(selectedCells.length < 2) return;
  const selectedStr = selectedCells.map(({r,c}) => grid[r][c]).join('');
  const reversedStr = selectedStr.split('').reverse().join('');

  const match = placedWords.find(pw =>
    !foundWords.has(pw.word) && (pw.word === selectedStr || pw.word === reversedStr)
  );

  if(match){
    foundWords.add(match.word);
    match.cells.forEach(({r,c}) => cellEl(r,c).classList.add('found'));
    const li = wordListEl.querySelector(`[data-word="${match.word}"]`);
    if(li) li.classList.add('done');
    updateStats();
    if(foundWords.size === placedWords.length){
      clearInterval(timerInterval);
      winMsgEl.classList.add('show');
    }
  }
}

function updateStats(){
  foundCountEl.textContent = `${foundWords.size} / ${placedWords.length}`;
}

function startTimer(){
  clearInterval(timerInterval);
  startTime = Date.now();
  timerEl.textContent = '00:00';
  timerInterval = setInterval(() => {
    const elapsed = Math.floor((Date.now() - startTime)/1000);
    const m = String(Math.floor(elapsed/60)).padStart(2,'0');
    const s = String(elapsed%60).padStart(2,'0');
    timerEl.textContent = `${m}:${s}`;
  }, 1000);
}

document.getElementById('restartBtn').addEventListener('click', buildGame);

buildGame();
</script>
</body>
</html>
