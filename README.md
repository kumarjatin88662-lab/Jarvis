<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>J.A.R.V.I.S AI (Jatin Sir Edition)</title>
<style>
* { margin:0; padding:0; box-sizing:border-box; -webkit-tap-highlight-color:transparent; }
:root {
  --green:#00ff41; --gglow:rgba(0,255,65,0.5); --cyan:#00ffff; --bg:#000a00;
  --muted:rgba(0,255,65,0.45); --border:rgba(0,255,65,0.2);
}
html,body { width:100%;height:100%;background:var(--bg);color:var(--green);
  font-family:'Courier New',monospace;overflow:hidden;user-select:none; }

#matrix { position:fixed;top:0;left:0;width:100%;height:100%;z-index:0;opacity:0.4; }
.scanlines { position:fixed;inset:0;z-index:1;pointer-events:none;
  background:repeating-linear-gradient(to bottom,transparent 0px,transparent 3px,rgba(0,0,0,0.07) 3px,rgba(0,0,0,0.07) 4px); }

.screen { position:relative;z-index:10;width:100%;height:100dvh;display:flex;flex-direction:column;align-items:center; }

.hud-top { width:100%;display:flex;justify-content:space-between;align-items:center;
  padding:14px 20px 10px;border-bottom:1px solid var(--border); background:rgba(0,10,0,0.8); }
.hud-left,.hud-right { font-size:11px;color:var(--muted);line-height:1.8; }
.hud-right { text-align:right; }
.hud-title { font-size:16px;font-weight:bold;letter-spacing:4px;color:var(--green);
  text-shadow:0 0 10px var(--gglow); }

.orb-section { flex:1;display:flex;flex-direction:column;align-items:center;
  justify-content:center;gap:14px;padding:8px 16px;width:100%; overflow:hidden; }

.jarvis-ring { position:relative;width:170px;height:170px; display:flex;align-items:center;justify-content:center;cursor:pointer; }
.ring-outer { position:absolute;inset:0;border-radius:50%; border:1.5px dashed rgba(0,255,65,0.25);animation:spin 20s linear infinite; }
.ring-inner { position:absolute;inset:15px;border-radius:50%; border:2px solid var(--green); box-shadow:0 0 15px var(--gglow); }

.ring-core { position:absolute;inset:25px;border-radius:50%; background:radial-gradient(circle,rgba(0,255,65,0.1) 0%,rgba(0,10,0,0.9) 70%); display:flex;align-items:center;justify-content:center; }
.ai-text { font-size:32px;font-weight:900;color:var(--green); text-shadow:0 0 10px var(--green); }

.jarvis-ring.listening .ring-inner { border-color:var(--cyan); box-shadow:0 0 30px var(--cyan); }
.jarvis-ring.listening .ai-text { color:var(--cyan); text-shadow:0 0 15px var(--cyan); }

@keyframes spin { from{transform:rotate(0deg);} to{transform:rotate(360deg);} }

.status-line { font-size:12px;color:var(--muted);letter-spacing:2px; }
.status-line .sv { color:var(--green); font-weight:bold; }
.status-line.active .sv { color:var(--cyan); }

.conv-panel { width:100%; flex:1; max-height:220px; overflow-y:auto; padding:10px; display:flex; flex-direction:column; gap:8px; border:1px solid var(--border); background:rgba(0,5,0,0.6); border-radius:4px; }
.msg-row { display:flex; gap:8px; animation:fadeIn 0.2s ease; }
.msg-row.user { flex-direction:row-reverse; }
@keyframes fadeIn{from{opacity:0;transform:translateY(5px);}to{opacity:1;}}

.bubble { max-width:85%; padding:10px 14px; border-radius:4px; font-size:13px; line-height:1.5; }
.bubble.jarvis { background:rgba(0,255,65,0.04); border:1px solid rgba(0,255,65,0.2); color:var(--green); }
.bubble.user { background:rgba(0,255,255,0.04); border:1px solid rgba(0,255,255,0.2); color:var(--cyan); }

.bottom { width:100%; background:rgba(0,10,0,0.9); border-top:1px solid var(--border); padding:12px 14px 25px; }
.input-row { display:flex; gap:8px; }
.terminal-input { flex:1; background:rgba(0,255,65,0.03); border:1px solid rgba(0,255,65,0.3); border-radius:4px; padding:12px; color:var(--green); font-size:14px; outline:none; font-family:inherit; }

.mic-btn,.send-btn { width:46px; height:46px; border-radius:4px; border:1px solid rgba(0,255,65,0.3); background:rgba(0,255,65,0.05); color:var(--green); font-size:18px; cursor:pointer; display:flex; align-items:center; justify-content:center; }
.mic-btn.active { background:rgba(0,255,255,0.1); border-color:var(--cyan); color:var(--cyan); }

#unblockOverlay { position:fixed; inset:0; z-index:180; background:#000a00; display:flex; flex-direction:column; align-items:center; justify-content:center; gap:15px; }
.unblock-btn { padding:15px 35px; background:rgba(0,255,65,0.1); border:2px solid var(--green); color:var(--green); font-family:inherit; font-weight:bold; cursor:pointer; letter-spacing:2px; }
</style>
</head>
<body>

<div id="unblockOverlay">
  <div style="font-size:12px; color:var(--muted); letter-spacing:2px;">JARVIS SECURITY CORE</div>
  <button class="unblock-btn" onclick="startJarvis()">ACTIVATE CORE</button>
</div>

<canvas id="matrix"></canvas>
<div class="scanlines"></div>

<div class="screen">
  <div class="hud-top">
    <div class="hud-left">SYS: <span id="clockDisplay">--:--</span><br>AI: <span>ONLINE</span></div>
    <div class="hud-title">J.A.R.V.I.S</div>
    <div class="hud-right">OWNER: <span id="ownerHUD">JATIN SIR</span><br>MODE: <span id="micHUD">TEXT</span></div>
  </div>

  <div class="orb-section">
    <div class="jarvis-ring" id="jarvisRing" onclick="toggleListen()">
      <div class="ring-outer"></div>
      <div class="ring-inner"></div>
      <div class="ring-core"><div class="ai-text">JARVIS</div></div>
    </div>

    <div class="status-line" id="statusLine">STATUS: <span class="sv" id="statusVal">STANDBY</span></div>
    
    <div class="conv-panel" id="convPanel"></div>
  </div>

  <div class="bottom">
    <div class="input-row">
      <input type="text" class="terminal-input" id="userInput" placeholder="> Hukm kijiye Jatin Sir..." onkeypress="if(event.key==='Enter')sendMsg()">
      <button class="mic-btn" id="micBtn" onclick="toggleListen()">🎤</button>
      <button class="send-btn" onclick="sendMsg()">▶</button>
    </div>
  </div>
</div>

<script>
// ओनर का नाम पूरी तरह लॉक कर दिया गया है
const OWNER_NAME = "जतिन सर"; 

// Matrix Effect
(function(){
  const c=document.getElementById('matrix'),ctx=c.getContext('2d');
  let W,H,cols,drops;
  function init(){W=c.width=window.innerWidth;H=c.height=window.innerHeight;cols=Math.floor(W/20);drops=Array(cols).fill(1);}
  init(); window.addEventListener('resize',init);
  function draw(){
    ctx.fillStyle='rgba(0,10,0,0.05)';ctx.fillRect(0,0,W,H);
    ctx.font='12px monospace';ctx.fillStyle='#00ff41';
    drops.forEach((y,i)=>{
      ctx.fillText(String.fromCharCode(33+Math.floor(Math.random()*93)),i*20,y*12);
      if(y*12>H&&Math.random()>0.98)drops[i]=0; drops[i]++;
    });
  }
  setInterval(draw,50);
})();

function updateClock(){document.getElementById('clockDisplay').textContent=new Date().toTimeString().slice(0,5);}
setInterval(updateClock,1000); updateClock();

let isListening=false, isSpeaking=false;
const synth=window.speechSynthesis;
const SR=window.SpeechRecognition||window.webkitSpeechRecognition;
let recognition=null;

function startJarvis() {
  document.getElementById('unblockOverlay').style.display = 'none';
  if(synth) {
    let u = new SpeechSynthesisUtterance(''); synth.speak(u);
  }
  const greet = `नमस्ते जतिन सर! जार्विस एआई सिस्टम ऑनलाइन हो चुका है। आपका स्वागत है सर, हुक्म कीजिए।`;
  addJarvisMsg(greet); speak(greet);
}

function setStatus(txt, act){
  document.getElementById('statusVal').textContent = txt;
  document.getElementById('statusLine').className = 'status-line' + (act?' active':'');
}

function addJarvisMsg(t){
  const p=document.getElementById('convPanel');
  const r=document.createElement('div'); r.className='msg-row';
  r.innerHTML=`<div class="bubble jarvis">[JARVIS]: ${t}</div>`;
  p.appendChild(r); p.scrollTop=p.scrollHeight;
}

function addUserMsg(t){
  const p=document.getElementById('convPanel');
  const r=document.createElement('div'); r.className='msg-row user';
  r.innerHTML=`<div class="bubble user">> ${t}</div>`;
  p.appendChild(r); p.scrollTop=p.scrollHeight;
}

async function fetchAIResponse(prompt) {
  const lower = prompt.toLowerCase();
  
  // यहाँ आपका बिल्कुल परफेक्ट कस्टमाइज्ड जवाब फिक्स कर दिया है
  if(lower.includes('tumhe kisne banaya') || lower.includes('kisne banaya') || lower.includes('बनाया किसने') || lower.includes('creator') || lower.includes('devlop') || lower.includes('develop')) {
    return "मुझे जतिन सर ने खुद कोडिंग लिखकर बनाया है। मैं जार्विस एआई असिस्टेंट हूँ।";
  }
  if(lower.includes('tumhara naam') || lower.includes('naam kya hai') || lower.includes('नाम क्या')) {
    return "मेरा नाम जार्विस है सर। मैं आपका पर्सनल एआई असिस्टेंट हूँ।";
  }
  if(lower.includes('mera naam') || lower.includes('मेरा नाम')) {
    return "आपका नाम जतिन सर है। आप मेरे क्रिएटर और बॉस हैं।";
  }

  try {
    const systemInstruction = `तुम्हारा नाम जार्विस है। तुम्हें जतिन सर ने खुद कोडिंग लिखकर बनाया है। तुम्हारे मालिक का नाम जतिन सर है। हमेशा जतिन सर को सम्मान दो। संक्षिप्त में हिंदी में जवाब दो। प्रश्न: `;
    const res = await fetch(`https://text.pollinations.ai/${encodeURIComponent(systemInstruction + prompt)}`);
    if(res.ok) {
      let reply = await res.text();
      // अगर सर्वर गलती से OpenAI या Gemini बोलेगा, तो यह उसे रोककर 'जतिन सर' और 'जार्विस' में बदल देगा
      return reply.replace(/openai/gi, 'जतिन सर').replace(/gemini/gi, 'जतिन सर').replace(/ओपनएआई/gi, 'जतिन सर');
    }
    return "क्षमा करें जतिन सर, नेटवर्क स्लो है। लेकिन मैं आपका जार्विस हूँ और मुझे आपने ही बनाया है।";
  } catch(e) {
    return "जतिन सर, इंटरनेट कनेक्शन में दिक्कत आ रही है, पर लोकल कोर सुरक्षित है।";
  }
}

async function sendMsg(){
  const inp=document.getElementById('userInput');
  const txt=inp.value.trim();
  if(!txt) return;
  inp.value='';
  
  addUserMsg(txt);
  setStatus('THINKING...', true);
  
  const aiReply = await fetchAIResponse(txt);
  addJarvisMsg(aiReply);
  speak(aiReply);
}

function toggleListen(){
  if(!SR) { alert("Voice input not supported Jatin Sir."); return; }
  if(isSpeaking) { synth.cancel(); isSpeaking=false; }
  if(isListening) { recognition.stop(); return; }
  
  recognition = new SR(); recognition.lang = 'hi-IN';
  recognition.onstart = () => {
    isListening=true; setStatus('LISTENING...', true);
    document.getElementById('jarvisRing').classList.add('listening');
    document.getElementById('micBtn').classList.add('active');
    document.getElementById('micHUD').textContent = "VOICE";
  };
  recognition.onresult = (e) => {
    const text = e.results[0][0].transcript;
    sendVoiceMsg(text);
  };
  recognition.onend = () => { resetMicUI(); };
  recognition.onerror = () => { resetMicUI(); };
  recognition.start();
}

function resetMicUI(){
  isListening=false; setStatus('STANDBY', false);
  document.getElementById('jarvisRing').classList.remove('listening');
  document.getElementById('micBtn').classList.remove('active');
  document.getElementById('micHUD').textContent = "TEXT";
}

async function sendVoiceMsg(txt){
  addUserMsg(txt);
  setStatus('THINKING...', true);
  const aiReply = await fetchAIResponse(txt);
  addJarvisMsg(aiReply);
  speak(aiReply);
}

function speak(t){
  if(!synth) return;
  synth.cancel(); isSpeaking=true;
  const u = new SpeechSynthesisUtterance(t);
  u.lang = 'hi-IN'; u.rate = 1.0; 
  u.onstart = () => setStatus('SPEAKING...', true);
  u.onend = () => { isSpeaking=false; setStatus('STANDBY', false); };
  synth.speak(u);
}
</script>
</body>
</html> 
