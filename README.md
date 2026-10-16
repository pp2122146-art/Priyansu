
<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>WhatsApp AI Dashboard</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#0b141a;color:#e9edef;font-family:Arial,sans-serif}
header{background:#202c33;padding:18px;font-size:22px;font-weight:bold}
header span{color:#25d366}
.layout{display:flex;min-height:90vh}
nav{width:220px;background:#111b21;padding:20px}
nav button{display:block;width:100%;padding:13px;margin-bottom:10px;
background:#202c33;color:white;border:0;border-radius:8px;text-align:left;cursor:pointer}
main{padding:22px;flex:1;min-width:0}
.panel{background:#202c33;padding:20px;border-radius:12px;margin-bottom:18px}
input,textarea{display:block;width:100%;padding:12px;margin:10px 0;
background:#111b21;border:1px solid #43545d;color:white;border-radius:7px}
button.action{background:#25d366;color:#062b1b;border:0;padding:12px 16px;
border-radius:7px;cursor:pointer;font-weight:bold}
.status{font-size:13px;color:#ffd166}
#chat{height:260px;overflow:auto;background:#111b21;padding:12px;border-radius:8px}
.msg{padding:10px;margin:8px 0;border-radius:8px;background:#263b32;overflow-wrap:anywhere}
@media(max-width:650px){.layout{display:block}nav{width:100%}main{padding:12px}}
</style>
</head>
<body>
<header><span>●</span> WhatsApp AI <small>Dashboard</small></header>
<div class="layout">
<nav>
<button onclick="page('home')">Dashboard</button>
<button onclick="page('ai')">AI Assistant</button>
<button onclick="page('wa')">WhatsApp Connection</button>
</nav>
<main id="main"></main>
</div>
<script>
const main=document.getElementById('main');
let apiKey='';

function page(p){
if(p==='home')main.innerHTML=`
<h2>Dashboard</h2>
<div class="panel">
<h3>System status</h3>
<p>AI: <span id="aistatus">Not connected</span></p>
<p>WhatsApp: <span class="status">Backend not connected</span></p>
<p>असली WhatsApp messages पाने के लिए backend आवश्यक है।</p>
</div>
<div class="panel"><h3>Quick setup</h3>
<p>1. AI API key सेट करें।</p>
<p>2. Backend को deploy करें।</p>
<p>3. Meta WhatsApp Cloud API जोड़ें।</p></div>`;
if(p==='ai')main.innerHTML=`
<h2>AI Assistant</h2>
<div class="panel">
<label>Groq API Key</label>
<input id="key" type="password" placeholder="gsk_...">
<button class="action" onclick="saveKey()">Connect AI</button>
<p class="status">Key इस पेज पर memory में रहेगी; reload पर मिट जाएगी।</p>
</div>
<div class="panel">
<div id="chat"></div>
<textarea id="question" placeholder="AI से कुछ पूछो..."></textarea>
<button class="action" onclick="askAI()">Send</button>
<p id="result" class="status"></p>
</div>`;
if(p==='wa')main.innerHTML=`
<h2>WhatsApp Connection</h2>
<div class="panel">
<h3>Backend settings</h3>
<label>Deployed backend HTTPS URL</label>
<input id="backend" placeholder="Render से मिला URL">
<label>WhatsApp recipient number</label>
<input id="recipient" placeholder="Country code सहित नंबर">
<button class="action" onclick="connectBackend()">Check connection</button>
<p id="waresult" class="status">
अभी WhatsApp connected नहीं है।
</p>
</div>
<div class="panel">
<h3>Setup requirements</h3>
<p>• Meta WhatsApp Business Platform</p>
<p>• Phone Number ID और access token backend पर</p>
<p>• Incoming messages के लिए webhook</p>
<p>सुरक्षा के लिए WhatsApp access token इस HTML में न डालें।</p>
</div>`;
}
function saveKey(){
apiKey=document.getElementById('key').value.trim();
document.getElementById('result').textContent='';
document.getElementById('aistatus')?.replaceChildren();
alert(apiKey?'AI key इस session में सेट हो गई।':'पहले API key डालें।');
}
async function askAI(){
const q=document.getElementById('question').value.trim();
if(!q)return;
if(!apiKey){alert('पहले AI Assistant में API key डालें।');return}
const chat=document.getElementById('chat');
const user=document.createElement('div');
user.className='msg';user.textContent='You: '+q;chat.append(user);
document.getElementById('question').value='';
const reply=document.createElement('div');
reply.className='msg';reply.textContent='AI: जवाब आ रहा है...';chat.append(reply);
try{
const r=await fetch('https://api.groq.com/openai/v1/chat/completions',{
method:'POST',
headers:{'Content-Type':'application/json','Authorization':'Bearer '+apiKey},
body:JSON.stringify({
model:'llama-3.3-70b-versatile',
messages:[{role:'system',content:'You are a helpful assistant. Reply in the language the user uses.'},{role:'user',content:q}]
})
});
const data=await r.json();
if(!r.ok)throw Error(data.error?.message||'API request failed');
reply.textContent='AI: '+data.choices[0].message.content;
}catch(e){reply.textContent='Error: '+e.message}
chat.scrollTop=chat.scrollHeight;
}
async function connectBackend(){
const url=document.getElementById('backend').value.trim().replace(/\/+$/,'');
const out=document.getElementById('waresult');
if(!url){out.textContent='पहले deployed backend URL डालें।';return}
out.textContent='Checking...';
try{
const r=await fetch(url+'/health');
if(!r.ok)throw Error('Backend health check failed');
out.textContent='Backend reachable. WhatsApp connection अलग से verify करनी होगी।';
}catch(e){out.textContent='Connection failed. Backend URL और CORS settings जाँचें।'}
}
page('home');
</script>
</body>
</html>
