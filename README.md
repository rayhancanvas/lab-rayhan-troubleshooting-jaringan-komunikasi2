<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Lab rayhanTKJ - Ping & Traceroute</title>
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{background:#050a08;color:#00ff88;font-family:Consolas,monospace;padding:18px}
header{text-align:center;border:1px solid #00ff88;padding:20px;margin-bottom:18px;box-shadow:0 0 15px #00ff8844}
header h1{font-size:26px;text-shadow:0 0 10px #00ff88}
header p{color:#8bc9a8;margin-top:8px}
.container{display:grid;grid-template-columns:1fr 1.2fr 1fr;gap:15px}
.panel{background:#09140f;border:1px solid #00ff88;border-radius:8px;padding:15px;min-width:0}
.panel h2{font-size:15px;margin-bottom:15px;padding-bottom:10px;border-bottom:1px solid #174b32}
.terminal,.logs{height:350px;overflow-y:auto;background:#020604;border:1px solid #174b32;padding:12px;font-size:12px;line-height:1.7;overflow-wrap:anywhere}
.input-area{display:flex;gap:7px;margin-top:10px}
input{flex:1;min-width:0;background:#020604;color:#00ff88;border:1px solid #00ff88;padding:11px;font-family:Consolas,monospace;outline:none}
button{background:#00ff88;color:#001b0d;border:1px solid #00ff88;padding:10px 12px;font-weight:bold;cursor:pointer;font-family:Consolas,monospace}
button:hover{background:#b7ffd8}
button:disabled{opacity:.5;cursor:not-allowed}
.quick{display:flex;flex-wrap:wrap;gap:6px;margin-top:10px}
.quick button{background:transparent;color:#00ff88;padding:8px}
.topology{min-height:390px;display:flex;flex-direction:column;align-items:center;justify-content:center}
.device{width:150px;text-align:center;padding:13px;border:1px solid #00ff88;border-radius:8px;background:#061b10;transition:.3s}
.device span{display:block;font-size:28px;margin-bottom:5px}
.device small{display:block;color:#8bc9a8;margin-top:4px}
.device.active{background:#07502c;box-shadow:0 0 18px #00ff88}
.device.failed{border-color:#ff6262;color:#ff6262;box-shadow:0 0 15px #ff6262}
.link{height:38px;width:3px;background:#24543b;position:relative}
.link.active{background:#00ff88;box-shadow:0 0 10px #00ff88}
.link.active::after{content:"";position:absolute;width:9px;height:9px;border-radius:50%;background:#00ff88;left:-3px;top:0;animation:packet .65s linear infinite}
@keyframes packet{to{top:29px}}
.status{border:1px dashed #174b32;padding:11px;text-align:center;font-size:12px;margin-top:8px}
.good{color:#00ff88}.warn{color:#ffd166}.bad{color:#ff6262}
.entry{padding:7px 0;border-bottom:1px solid #143323}
footer{text-align:center;color:#67977b;font-size:11px;margin-top:20px}
@media(max-width:950px){.container{grid-template-columns:1fr}.terminal,.logs{height:260px}}
</style>
</head>
<body>

<header>
<h1>⚡ LAB rayhanTKJ ⚡</h1>
<p>MODUL TROUBLESHOOTING KONEKSI JARINGAN</p>
<p>SIMULASI CLI PING & PACKET TRACEROUTE</p>
</header>

<div class="container">

<section class="panel">
<h2>01 // COMMAND TERMINAL</h2>
<div class="terminal" id="terminal">
<div class="good">================================</div>
<div class="good"> LAB rayhanTKJ NETWORK LAB</div>
<div class="good"> SIMULATOR READY</div>
<div class="good">================================</div>
<div>Ketik help untuk melihat perintah.</div>
<div class="good">root@rayhan:~#</div>
</div>
<div class="input-area">
<input id="command" placeholder="ping 192.168.1.1" autocomplete="off">
<button id="run">RUN</button>
</div>
<div class="quick">
<button data-cmd="ping 192.168.1.1">PING</button>
<button data-cmd="tracert 8.8.8.8">TRACERT</button>
<button data-cmd="ping 192.168.1.99">TEST TIMEOUT</button>
<button data-cmd="help">HELP</button>
<button id="clear">CLEAR</button>
</div>
</section>

<section class="panel">
<h2>02 // NETWORK TOPOLOGY</h2>
<div class="topology">
<div class="device" id="pc"><span>💻</span><b>PC CLIENT</b><small>192.168.1.10</small></div>
<div class="link" id="link1"></div>
<div class="device" id="sw"><span>🔀</span><b>SWITCH</b><small>HOP 1</small></div>
<div class="link" id="link2"></div>
<div class="device" id="router"><span>🌐</span><b>ROUTER</b><small>HOP 2</small></div>
<div class="link" id="link3"></div>
<div class="device" id="server"><span>🖥️</span><b>SERVER</b><small>HOP 3</small></div>
</div>
<div class="status" id="status">STATUS: MENUNGGU PERINTAH</div>
</section>

<section class="panel">
<h2>03 // ICMP RESPONSE LOG</h2>
<div class="logs" id="logs">
<div class="good">[SYSTEM] Menunggu proses diagnostik...</div>
</div>
</section>

</div>

<footer>
LAB rayhanTKJ | MEDIA PEMBELAJARAN JARINGAN KOMPUTER
<br>SIMULASI PING & TRACEROUTE
</footer>

<script>
const $=id=>document.getElementById(id);
const terminal=$("terminal"),logs=$("logs"),input=$("command");
let busy=false;

function term(message,cls=""){
 const d=document.createElement("div");
 d.className=cls;
 d.textContent=message;
 terminal.appendChild(d);
 terminal.scrollTop=terminal.scrollHeight;
}

function log(message,cls="good"){
 const d=document.createElement("div");
 d.className="entry "+cls;
 d.textContent=message;
 logs.appendChild(d);
 logs.scrollTop=logs.scrollHeight;
}

function delay(ms){
 return new Promise(resolve=>setTimeout(resolve,ms));
}

function reset(){
 ["pc","sw","router","server"].forEach(id=>{
  $(id).classList.remove("active","failed");
 });
 ["link1","link2","link3"].forEach(id=>{
  $(id).classList.remove("active");
 });
}

function validIP(ip){
 const p=ip.split(".");
 return p.length===4 && p.every(x=>
  /^\d{1,3}$/.test(x) && Number(x)>=0 && Number(x)<=255
 );
}

function unreachable(ip){
 return ip==="192.168.1.99" || ip==="203.0.113.99";
}

async function hop(index){
 const devices=["pc","sw","router","server"];
 const links=["link1","link2","link3"];
 $(devices[index]).classList.add("active");
 if(index>0) $(links[index-1]).classList.add("active");
 await delay(550);
}

async function ping(ip){
 reset();
 logs.innerHTML="";
 $("status").textContent="STATUS: MENGIRIM ICMP...";

 log("[PING] Tujuan: "+ip);

 for(let i=0;i<4;i++) await hop(i);

 if(unreachable(ip)){
  $("server").classList.add("failed");
  for(let i=0;i<4;i++){
   log("Request Timed Out - Tidak ada balasan dari "+ip,"bad");
   await delay(350);
  }
  $("status").textContent="STATUS: CONNECTION FAILED";
  term("Ping gagal: host tidak memberikan respons.","bad");
 }else{
  for(let i=0;i<4;i++){
   const rtt=Math.floor(Math.random()*35)+4;
   log("Reply from "+ip+": bytes=32 time="+rtt+"ms TTL=64");
   await delay(350);
  }
  $("status").textContent="STATUS: CONNECTION SUCCESS";
  term("Ping berhasil. Host dapat dijangkau.","good");
 }
}

async function tracert(ip){
 reset();
 logs.innerHTML="";
 $("status").textContent="STATUS: TRACING ROUTE...";
 log("[TRACERT] Melacak rute menuju "+ip);

 const hops=[
  {ip:"192.168.1.1",ttl:1},
  {ip:"10.10.10.1",ttl:2},
  {ip:ip,ttl:3}
 ];

 for(let i=0;i<4;i++) await hop(i);

 for(let i=0;i<hops.length;i++){
  if(unreachable(ip)&&i===2){
   log((i+1)+"    * * *    Request Timed Out","bad");
   $("server").classList.add("failed");
  }else{
   const rtt=Math.floor(Math.random()*35)+5;
   log((i+1)+"    "+hops[i].ip+"    "+rtt+" ms    TTL="+hops[i].ttl);
  }
  await delay(450);
 }

 if(unreachable(ip)){
  $("status").textContent="STATUS: ROUTE INCOMPLETE";
  term("Traceroute selesai dengan hop tujuan timeout.","warn");
 }else{
  $("status").textContent="STATUS: ROUTE COMPLETED";
  term("Traceroute selesai. Jalur simulasi ditemukan.","good");
 }
}

async function runCommand(){
 if(busy)return;

 const raw=input.value.trim();
 if(!raw)return;

 input.value="";
 term("root@rayhan:~# "+raw,"good");

 const parts=raw.split(/\s+/);
 const action=parts[0].toLowerCase();
 const ip=parts[1];

 if(action==="help"){
  term("DAFTAR PERINTAH:","good");
  term("ping [ip] - Menguji koneksi jaringan");
  term("tracert [ip] - Melacak jalur paket");
  term("clear - Membersihkan terminal");
  term("help - Menampilkan bantuan");
  return;
 }

 if(action==="clear"){
  clearTerminal();
  return;
 }

 if(action!=="ping"&&action!=="tracert"){
  term("Perintah tidak dikenal. Ketik help.","bad");
  return;
 }

 if(!ip||!validIP(ip)){
  term("Format IP tidak valid. Contoh: ping 192.168.1.1","bad");
  return;
 }

 busy=true;
 $("run").disabled=true;

 try{
  if(action==="ping") await ping(ip);
  else await tracert(ip);
 }finally{
  busy=false;
  $("run").disabled=false;
  input.focus();
 }
}

function clearTerminal(){
 terminal.innerHTML="";
 term("Terminal dibersihkan.","good");
 term("root@rayhan:~#");
}

$("run").addEventListener("click",runCommand);
input.addEventListener("keydown",e=>{
 if(e.key==="Enter")runCommand();
});
$("clear").addEventListener("click",clearTerminal);

document.querySelectorAll("[data-cmd]").forEach(btn=>{
 btn.addEventListener("click",()=>{
  input.value=btn.dataset.cmd;
  runCommand();
 });
});
</script>
</body>
</html>
