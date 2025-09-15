# Panel
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>MEHRDAD WireGuard Panel</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700&display=swap');
html, body {
  margin:0; padding:0; width:100%; height:100%; overflow:hidden;
  font-family: 'Orbitron', sans-serif; background:#0b0b0b;
}
canvas#bg { position: fixed; top:0; left:0; z-index:-1; width:100%; height:100%; }
.panel { position: fixed; top:50%; left:50%; transform:translate(-50%, -50%);
  background: rgba(15,15,15,0.95); padding:40px; border-radius:20px;
  box-shadow: 0 0 50px #6f00ff,0 0 80px #ff00d4; text-align:center; width:420px; max-width:90%;
}
h1 { color:#6f00ff; font-size:42px; margin-bottom:20px;
     text-shadow: 0 0 10px #6f00ff,0 0 20px #ff00d4,0 0 30px #00ffff; }
input, button { width:90%; padding:14px; margin:10px 0; border-radius:10px; border:none; font-size:16px; outline:none; }
input { background:#111; color:#0f0; font-family: monospace; }
button { background: linear-gradient(90deg,#6f00ff,#ff00d4); color:#fff; cursor:pointer; font-weight:bold; transition:0.3s; text-transform:uppercase; box-shadow: 0 0 10px #6f00ff,0 0 20px #ff00d4,0 0 30px #00ffff; }
button:hover { background: linear-gradient(90deg,#ff00d4,#00ffff); box-shadow:0 0 20px #ff00d4,0 0 40px #6f00ff,0 0 50px #00ffff; }
.server-card { display: none; flex-direction: column; background:#222; padding:15px; border-radius:15px; margin-top:10px; }
.flag-btn { background:none; border:1px solid #6f00ff; border-radius:10px; color:#fff; padding:8px; margin:5px 0; cursor:pointer; font-size:18px; transition:0.3s; }
.flag-btn:hover { background:#6f00ff; color:#000; }
textarea { width:90%; height:200px; margin-top:15px; background:#111; color:#0f0; padding:10px; border-radius:10px; font-family: monospace; font-size:14px; resize:none; }
</style>
</head>
<body>
<script src="https://cdnjs.cloudflare.com/ajax/libs/tweetnacl/1.0.3/nacl.min.js"></script>
<canvas id="bg"></canvas>

<div class="panel" id="loginPanel">
  <h1>MEHRDAD</h1>
  <input type="password" id="password" placeholder="Enter Password">
  <button onclick="checkLogin()">Unlock</button>
</div>

<div class="panel" id="mainPanel" style="display:none;">
  <h1>WireGuard</h1>
  <button onclick="toggleServerCard()">SERVER</button>
  <div class="server-card" id="serverCard">
    <button class="flag-btn" onclick="selectServer('UAE')">🇦🇪 UAE</button>
    <button class="flag-btn" onclick="selectServer('Germany')">🇩🇪 Germany</button>
    <button class="flag-btn" onclick="selectServer('USA')">🇺🇸 USA</button>
    <button class="flag-btn" onclick="selectServer('France')">🇫🇷 France</button>
    <button class="flag-btn" onclick="selectServer('Qatar')">🇶🇦 Qatar</button>
  </div>
  <input type="text" id="configName" placeholder="Config name (example.conf)">
  <button onclick="createConfig()">Create Config</button>
  <button onclick="downloadConfig()">Download Config</button>
  <textarea id="configOutput" readonly></textarea>
</div>

<script>
function checkLogin(){
  if(document.getElementById("password").value==="MEHRDAD"){
    document.getElementById("loginPanel").style.display="none";
    document.getElementById("mainPanel").style.display="block";
  } else alert("Wrong Password!");
}

function toggleServerCard(){
  const card=document.getElementById("serverCard");
  card.style.display=card.style.display==="flex"?"none":"flex";
}

let selectedServer="";
function selectServer(server){
  selectedServer=server;
  document.getElementById("serverCard").style.display="none";
  alert("Selected: "+server);
}

function randomIPv4(){ return Array(4).fill(0).map(()=>Math.floor(Math.random()*254+1)).join('.')+"/32"; }
function randomIPv6(){ return Array(8).fill(0).map(()=>Math.floor(Math.random()*65536).toString(16)).join(':')+"/128"; }

let configText="";
function createConfig(){
  if(!selectedServer){ alert("Please select a server!"); return; }
  
  const keyPair = nacl.box.keyPair();
  const privateKey = btoa(String.fromCharCode.apply(null, keyPair.secretKey.slice(0,32)));
  const publicKey = btoa(String.fromCharCode.apply(null, keyPair.publicKey));

  const ifaceIPv4 = randomIPv4();
  const ifaceIPv6 = randomIPv6();

  const allowedIPv4 = Array(5).fill(0).map(()=>randomIPv4()).join(", ");
  const allowedIPv6 = Array(5).fill(0).map(()=>randomIPv6()).join(", ");

  configText=`[Interface]
PrivateKey = ${privateKey}
Address = ${ifaceIPv4}, ${ifaceIPv6}
DNS = 10.202.10.10, 148.163.44.32

[Peer]
PublicKey = ${publicKey}
AllowedIPs = ${allowedIPv4}, ${allowedIPv6}
Endpoint = 203.0.113.77:51820
PersistentKeepalive = 15`;

  document.getElementById("configOutput").value=configText;
}

function downloadConfig(){
  if(!configText){ alert("No config created yet!"); return; }
  const name=document.getElementById("configName").value||"default.conf";
  const blob=new Blob([configText],{type:"text/plain"});
  const link=document.createElement("a");
  link.href=URL.createObjectURL(blob);
  link.download=name.endsWith(".conf")?name:name+".conf";
  link.click();
}

const canvas=document.getElementById('bg');
const ctx=canvas.getContext('2d');
canvas.width=window.innerWidth; canvas.height=window.innerHeight;
window.addEventListener('resize',()=>{canvas.width=window.innerWidth; canvas.height=window.innerHeight;});
let particles=[];
for(let i=0;i<200;i++){
  particles.push({x: Math.random()*canvas.width, y: Math.random()*canvas.height, dx:(Math.random()-0.5)*1.5, dy:(Math.random()-0.5)*1.5, radius:Math.random()*2+1, color:`hsl(${Math.random()*360},100%,50%)`});
}
function animate(){
  ctx.fillStyle='rgba(11,11,11,0.15)';
  ctx.fillRect(0,0,canvas.width,canvas.height);
  for(let i=0;i<particles.length;i++){
    const p=particles[i]; p.x+=p.dx; p.y+=p.dy;
    if(p.x<0||p.x>canvas.width)p.dx*=-1;
    if(p.y<0||p.y>canvas.height)p.dy*=-1;
    for(let j=i+1;j<particles.length;j++){
      const q=particles[j];
      const dist=Math.hypot(p.x-q.x,p.y-q.y);
      if(dist<120){ ctx.strokeStyle=`hsla(${Math.random()*360},100%,50%,0.3)`; ctx.beginPath(); ctx.moveTo(p.x,p.y); ctx.lineTo(q.x,q.y); ctx.stroke(); }
    }
    ctx.fillStyle=p.color; ctx.beginPath(); ctx.arc(p.x,p.y,p.radius,0,Math.PI*2); ctx.fill();
  }
  requestAnimationFrame(animate);
}
animate();
</script>
</body>
</html>