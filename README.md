<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>The Trade Intelligence</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    margin:0;
    font-family:Segoe UI,Arial;
    color:#fff;
    background:#050812;
}
.header{
    text-align:center;
    padding:18px;
    font-size:32px;
    font-weight:900;
    letter-spacing:1px;
    background:linear-gradient(to right,#0f1c3f,#091226);
    box-shadow:0 4px 15px rgba(0,0,0,.6);
}

.grid{
    display:grid;
    grid-template-columns:repeat(4,1fr);
    gap:20px;
    padding:20px;
}

.card{
    border-radius:16px;
    padding:14px;
    height:210px;
    background:linear-gradient(145deg,#1e2d5f,#0c1433);
    box-shadow:0 10px 30px rgba(0,0,0,.8);
    cursor:pointer;
    transition:.25s;
}
.card:hover{transform:translateY(-4px)}

.instrument{text-align:center;font-size:20px;font-weight:800}
.row{text-align:center;font-size:12px;margin-top:3px;opacity:.9}
.action{text-align:center;font-size:26px;font-weight:900;margin-top:14px}
.subtext{text-align:center;font-size:14px;margin-top:6px;font-weight:700}

.buy{color:#00ff9c}
.sell{color:#ff5c5c}
.warn{
    color:#ffd000;
    background:#000;
    padding:3px 8px;
    border-radius:6px;
    font-size:22px;
    font-weight:900;
    display:inline-block;
}

.overlay{
    position:fixed; inset:0;
    background:rgba(0,0,0,.7);
    display:none;
    justify-content:center;
    align-items:center;
    z-index:100;
}

.modal{
    width:480px;
    background:linear-gradient(145deg,#1c2b5a,#0b122d);
    border-radius:20px;
    padding:20px;
    box-shadow:0 25px 60px rgba(0,0,0,.9);
}

.modal h2{text-align:center;margin-top:0}

.section{margin-top:12px}
.section b{display:block;font-size:13px;margin-bottom:4px}

.pill{
    display:inline-block;
    padding:6px 14px;
    margin:4px;
    border-radius:14px;
    border:1px solid rgba(255,255,255,.3);
    background:rgba(255,255,255,.08);
    cursor:pointer;
    font-size:12px;
}
.pill.active{
    background:#fff;
    color:#000;
    font-weight:800;
}

.close{
    width:100%;
    margin-top:12px;
    padding:8px;
    border:none;
    border-radius:8px;
    background:#333;
    color:#fff;
}
</style>
</head>

<body>

<div class="header">THE TRADE INTELLIGENCE</div>
<div class="grid" id="board"></div>

<div class="overlay" id="overlay">
<div class="modal">
<h2 id="title"></h2>

<div class="section"><b>Trend</b>
<span class="pill" onclick="toggle('trend','Bullish')">Bullish</span>
<span class="pill" onclick="toggle('trend','Bearish')">Bearish</span>
</div>

<div class="section"><b>Primary Category</b>
<span class="pill" onclick="toggle('clean',true)">Clean</span>
</div>

<div class="section"><b>Parallel Channel</b>
<span class="pill" onclick="toggle('pc','Bullish')">Bullish Parallel Channel</span>
<span class="pill" onclick="toggle('pc','Bearish')">Bearish Parallel Channel</span>
</div>

<div class="section"><b>Double Top / Bottom</b>
<span class="pill" onclick="toggle('dt','Proper')">Proper</span>
<span class="pill" onclick="toggle('dt','Deep')">Deep Pullback</span>
</div>

<div class="section"><b>Consolidation</b>
<span class="pill" onclick="toggle('con','2RR')">2RR</span>
<span class="pill" onclick="toggle('con','<1.5RR')">&lt;1.5RR</span>
<span class="pill" onclick="toggle('con','Neckline')">At Neckline</span>
</div>

<div class="section"><b>Inside Candle</b>
<span class="pill" onclick="toggle('inside','Clean Neckline')">Clean Neckline</span>
<span class="pill" onclick="toggle('inside','Submerged')">Submerged</span>
</div>

<div class="section"><b>Traps</b>
<span class="pill" onclick="toggle('pct',true)">PCT</span>
<span class="pill" onclick="toggle('liq',true)">Liquidity Trap</span>
</div>

<div class="section"><b>SMMA</b>
<span class="pill" onclick="toggle('smma',true)">Accumulation at SMMA</span>
</div>

<button class="close" onclick="closeModal()">Close</button>
</div>
</div>

<script>
const instruments=["EURUSD","GBPUSD","EURGBP","USDJPY","GBPJPY","AUDUSD","USDCAD","NZDUSD","SP500","UK100","NAS100","XAUUSD","BTCUSD"];
const board=document.getElementById("board");
const overlay=document.getElementById("overlay");
const title=document.getElementById("title");

let active=null;
let state={};

instruments.forEach(i=>{
state[i]={
trend:null,clean:false,pc:null,dt:null,con:null,inside:null,pct:false,liq:false,smma:false
};
const c=document.createElement("div");
c.className="card";
c.id=i;
c.innerHTML=`
<div class="instrument">${i}</div>
<div class="row" id="${i}_trend">Trend: —</div>
<div class="row" id="${i}_cat">Category: —</div>
<div class="action">—</div>
<div class="subtext"></div>`;
c.onclick=()=>openModal(i);
board.appendChild(c);
});

function openModal(n){
active=n;
title.innerText=n;
overlay.style.display="flex";
refreshModal();
}
function closeModal(){overlay.style.display="none"}

function toggle(key,val){
if(typeof state[active][key]==="boolean"){
state[active][key]=!state[active][key];
}else{
state[active][key]=state[active][key]===val?null:val;
}
update(active);
refreshModal();
}

function refreshModal(){
document.querySelectorAll(".pill").forEach(p=>p.classList.remove("active"));
const s=state[active];

document.querySelectorAll(".pill").forEach(p=>{
const t=p.innerText;

if(t==="Bullish" && s.trend==="Bullish")p.classList.add("active");
if(t==="Bearish" && s.trend==="Bearish")p.classList.add("active");
if(t==="Clean" && s.clean)p.classList.add("active");

if(t.includes("Bullish Parallel") && s.pc==="Bullish")p.classList.add("active");
if(t.includes("Bearish Parallel") && s.pc==="Bearish")p.classList.add("active");

if(t==="Proper" && s.dt==="Proper")p.classList.add("active");
if(t==="Deep Pullback" && s.dt==="Deep")p.classList.add("active");

if(t==="2RR" && s.con==="2RR")p.classList.add("active");
if(t.includes("<") && s.con==="<1.5RR")p.classList.add("active");
if(t==="At Neckline" && s.con==="Neckline")p.classList.add("active");

if(t==="Clean Neckline" && s.inside==="Clean Neckline")p.classList.add("active");
if(t==="Submerged" && s.inside==="Submerged")p.classList.add("active");

if(t==="PCT" && s.pct)p.classList.add("active");
if(t==="Liquidity Trap" && s.liq)p.classList.add("active");
if(t==="Accumulation at SMMA" && s.smma)p.classList.add("active");
});
}

function update(n){
const s=state[n];
document.getElementById(n+"_trend").innerText="Trend: "+(s.trend||"—");

let cat=[];
if(s.clean) cat.push("Clean");
if(s.pc) cat.push(s.pc+" Parallel Channel");
if(s.dt) cat.push("Double Top/Bottom");
if(s.con) cat.push("Consolidation");
if(s.inside) cat.push(s.inside);
if(s.liq) cat.push("Liquidity Trap");
if(s.pct) cat.push("PCT");
if(s.smma) cat.push("Accumulation at SMMA");

document.getElementById(n+"_cat").innerText="Category: "+(cat.length?cat.join(" & "):"—");
}
</script>

</body>
</html>
