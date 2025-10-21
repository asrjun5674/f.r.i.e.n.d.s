
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Group Audio Chat Demo</title>
<style>
  body { font-family: Arial,sans-serif; margin:0; background:#1e1f22; color:#fff; }
  #loginPage, #chatPage { display:none; height:100vh; }
  #loginPage { display:flex; flex-direction:column; justify-content:center; align-items:center; }
  .user-btn { border:none; color:#fff; padding:15px 25px; margin:10px; border-radius:10px; font-size:18px; cursor:pointer; transition:0.2s; }
  .user-btn.Arjun { background:#facc15; color:#000; }
  .user-btn.Arjun:hover { background:#eab308; }
  .user-btn.Akshat { background:#ef4444; }
  .user-btn.Akshat:hover { background:#dc2626; }
  .user-btn.Narain { background:#3b82f6; }
  .user-btn.Narain:hover { background:#2563eb; }
  #chatPage { display:flex; }
  #sidebar { width:220px; background:#2b2d31; display:flex; flex-direction:column; padding:10px; }
  #sidebar h3 { text-align:center; border-bottom:1px solid #444; padding-bottom:10px; }
  #main { flex:1; display:flex; flex-direction:column; }
  #messages { flex:1; padding:10px; overflow-y:auto; }
  .msg { display:flex; align-items:center; margin:5px 0; padding:10px; border-radius:10px; background:#40444B; }
  .msg img { width:35px; height:35px; border-radius:50%; margin-right:10px; object-fit:cover; }
  .msg .name { font-weight:bold; margin-right:5px; }
  #inputArea { display:flex; padding:10px; background:#2b2d31; }
  #inputArea input { flex:1; padding:10px; border:none; border-radius:5px; }
  #inputArea button, #joinVoiceBtn, #leaveVoiceBtn { margin-left:10px; background:#5865F2; border:none; color:white; padding:10px 15px; border-radius:5px; cursor:pointer; }
</style>
</head>
<body>

<div id="loginPage">
  <h1>Login as:</h1>
  <button class="user-btn Arjun" onclick="login('Arjun')">Arjun</button>
  <button class="user-btn Akshat" onclick="login('Akshat')">Akshat</button>
  <button class="user-btn Narain" onclick="login('Narain')">Narain</button>
  <input type="file" id="uploadPicBtn" accept="image/*" />
  <div class="note">Profile picture is optional.</div>
</div>

<div id="chatPage">
  <div id="sidebar">
    <h3 id="userName"></h3>
    <button id="joinVoiceBtn">Join Voice Chat</button>
    <button id="leaveVoiceBtn" disabled>Leave Voice Chat</button>
  </div>
  <div id="main">
    <div id="messages"></div>
    <div id="inputArea">
      <input id="msgInput" placeholder="Type a message..." onkeydown="if(event.key==='Enter')sendMessage()">
      <button onclick="sendMessage()">Send</button>
    </div>
  </div>
</div>

<script>
let currentUser = null;
let chatHistory = JSON.parse(localStorage.getItem('groupChat')) || [];
let userPics = JSON.parse(localStorage.getItem('userPics')) || {};
const colors = { Arjun:'#facc15', Akshat:'#ef4444', Narain:'#3b82f6' };

const loginPage = document.getElementById('loginPage');
const chatPage = document.getElementById('chatPage');
const messagesDiv = document.getElementById('messages');
const uploadPicBtn = document.getElementById('uploadPicBtn');
const joinVoiceBtn = document.getElementById('joinVoiceBtn');
const leaveVoiceBtn = document.getElementById('leaveVoiceBtn');

function login(name){
  currentUser = name;
  loginPage.style.display='none';
  chatPage.style.display='flex';
  const userNameElem = document.getElementById('userName');
  userNameElem.textContent = name;
  userNameElem.style.color = colors[name];
  if(userPics[currentUser]) uploadPicBtn.disabled = true;
  loadMessages();
}

uploadPicBtn.addEventListener('change', e=>{
  const file = e.target.files[0];
  if(file){
    const reader = new FileReader();
    reader.onload = function(evt){
      userPics[currentUser] = evt.target.result;
      localStorage.setItem('userPics', JSON.stringify(userPics));
      uploadPicBtn.disabled=true;
      loadMessages();
    }
    reader.readAsDataURL(file);
  }
});

// Send message
function sendMessage(){
  const input = document.getElementById('msgInput');
  const text = input.value.trim();
  if(!text) return;
  chatHistory.push({user:currentUser,text});
  localStorage.setItem('groupChat', JSON.stringify(chatHistory));
  input.value='';
  loadMessages();
}

// Load messages
function loadMessages(){
  messagesDiv.innerHTML='';
  chatHistory.forEach(m=>{
    const div = document.createElement('div');
    div.className='msg';
    const img = document.createElement('img');
    img.src = userPics[m.user] || 'https://via.placeholder.com/35';
    div.appendChild(img);
    const nameSpan = document.createElement('span');
    nameSpan.textContent = m.user + ': ';
    nameSpan.className='name';
    nameSpan.style.color = colors[m.user];
    div.appendChild(nameSpan);
    div.appendChild(document.createTextNode(m.text));
    messagesDiv.appendChild(div);
  });
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
}

// Update messages between tabs
window.addEventListener('storage', ()=>{
  chatHistory = JSON.parse(localStorage.getItem('groupChat')) || [];
  userPics = JSON.parse(localStorage.getItem('userPics')) || {};
  loadMessages();
});

// Live voice chat (basic WebRTC demo)
let localStream=null;
let pc=null;

joinVoiceBtn.addEventListener('click', async ()=>{
  try{
    localStream = await navigator.mediaDevices.getUserMedia({audio:true});
    pc = new RTCPeerConnection();
    localStream.getTracks().forEach(t=>pc.addTrack(t, localStream));
    pc.ontrack = e=>{
      const audio = document.createElement('audio');
      audio.srcObject = e.streams[0];
      audio.autoplay = true;
      document.body.appendChild(audio);
    };
    joinVoiceBtn.disabled=true;
    leaveVoiceBtn.disabled=false;
    alert('Voice chat started. Open multiple tabs to simulate group voice chat.');
  }catch(err){
    alert('Microphone access denied or error: '+err.message);
  }
});

leaveVoiceBtn.addEventListener('click', ()=>{
  if(localStream) localStream.getTracks().forEach(t=>t.stop());
  if(pc) pc.close();
  joinVoiceBtn.disabled=false;
  leaveVoiceBtn.disabled=true;
});
</script>

</body>
</html>
