
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Audio Chat with Themes & Profile</title>
<style>
  body { font-family: Arial, sans-serif; margin: 0; background: #1e1f22; color: #fff; }
  #loginPage, #chatPage { display: none; height: 100vh; }
  #loginPage { display: flex; flex-direction: column; justify-content: center; align-items: center; }
  .user-btn { border: none; color: #fff; padding: 15px 25px; margin: 10px; border-radius: 10px; font-size: 18px; cursor: pointer; transition: 0.2s; }
  .user-btn.Arjun { background-color: #facc15; color: #000; }
  .user-btn.Arjun:hover { background-color: #eab308; }
  .user-btn.Akshat { background-color: #ef4444; }
  .user-btn.Akshat:hover { background-color: #dc2626; }
  .user-btn.Narain { background-color: #3b82f6; }
  .user-btn.Narain:hover { background-color: #2563eb; }
  #chatPage { display: flex; }
  #sidebar { width: 220px; background: #2b2d31; display: flex; flex-direction: column; padding: 10px; }
  #sidebar h3 { text-align: center; border-bottom: 1px solid #444; padding-bottom: 10px; }
  #main { flex: 1; display: flex; flex-direction: column; }
  #messages { flex: 1; padding: 10px; overflow-y: auto; }
  .msg { display: flex; align-items: center; margin: 5px 0; padding: 10px; border-radius: 10px; }
  .msg img { width: 35px; height: 35px; border-radius: 50%; margin-right: 10px; object-fit: cover; }
  .msg .name { font-weight: bold; margin-right: 5px; }
  #inputArea { display: flex; padding: 10px; background: #2b2d31; }
  #inputArea input { flex: 1; padding: 10px; border: none; border-radius: 5px; }
  #inputArea button { margin-left: 10px; background: #5865F2; border: none; color: white; padding: 10px 15px; border-radius: 5px; cursor: pointer; }
  #voiceControls { display: flex; flex-direction: column; margin-top: 10px; }
  #voiceControls button, #uploadPicBtn { margin: 5px 0; padding: 10px; border: none; border-radius: 5px; cursor: pointer; background: #5865F2; color: white; }
  .note { font-size: 13px; color: #99a0b3; margin-top: 8px; }
</style>
</head>
<body>

<!-- Login Page -->
<div id="loginPage">
  <h1>Login as:</h1>
  <button class="user-btn Arjun" onclick="login('Arjun')">Arjun</button>
  <button class="user-btn Akshat" onclick="login('Akshat')">Akshat</button>
  <button class="user-btn Narain" onclick="login('Narain')">Narain</button>
  <input type="file" id="uploadPicBtn" accept="image/*" />
  <div class="note">Profile picture is optional.</div>
</div>

<!-- Chat Page -->
<div id="chatPage">
  <div id="sidebar">
    <h3 id="userName"></h3>
    <div id="voiceControls">
      <button id="joinVoiceBtn">Join Voice Chat</button>
      <button id="leaveVoiceBtn" disabled>Leave Voice Chat</button>
      <button id="recordAudioBtn">Record & Send Voice</button>
    </div>
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
let chatHistory = JSON.parse(localStorage.getItem('chatHistory')) || { Arjun: [], Akshat: [], Narain: [] };
let userPics = JSON.parse(localStorage.getItem('userPics')) || {};
['Arjun','Akshat','Narain'].forEach(k => { if(!chatHistory[k]) chatHistory[k]=[]; });

const colors = { Arjun: '#facc15', Akshat: '#ef4444', Narain: '#3b82f6' };
const loginPage = document.getElementById('loginPage');
const chatPage = document.getElementById('chatPage');
const messagesDiv = document.getElementById('messages');
const uploadPicBtn = document.getElementById('uploadPicBtn');

function login(name) {
  currentUser = name;
  loginPage.style.display = 'none';
  chatPage.style.display = 'flex';
  const userNameElem = document.getElementById('userName');
  userNameElem.textContent = name;
  userNameElem.style.color = colors[name];
  loadMessages();
  if(userPics[currentUser]) uploadPicBtn.disabled = true;
}

uploadPicBtn.addEventListener('change', e => {
  const file = e.target.files[0];
  if(file) {
    const reader = new FileReader();
    reader.onload = function(evt) {
      userPics[currentUser] = evt.target.result;
      localStorage.setItem('userPics', JSON.stringify(userPics));
      loadMessages();
      uploadPicBtn.disabled = true;
    };
    reader.readAsDataURL(file);
  }
});

function loadMessages() {
  const messages = chatHistory[currentUser];
  messagesDiv.innerHTML = '';
  messages.forEach(m => {
    const div = document.createElement('div');
    div.className = 'msg';
    const img = document.createElement('img');
    img.src = userPics[m.user] || 'https://via.placeholder.com/35';
    div.appendChild(img);
    const nameSpan = document.createElement('span');
    nameSpan.textContent = m.user + ': ';
    nameSpan.className = 'name';
    nameSpan.style.color = colors[m.user];
    div.appendChild(nameSpan);
    if(m.audio) {
      const audio = document.createElement('audio');
      audio.controls = true;
      audio.src = m.audio;
      div.appendChild(audio);
    } else {
      div.appendChild(document.createTextNode(m.text));
    }
    messagesDiv.appendChild(div);
  });
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
}

function sendMessage() {
  const input = document.getElementById('msgInput');
  const text = input.value.trim();
  if(!text) return;
  chatHistory[currentUser].push({ user: currentUser, text });
  localStorage.setItem('chatHistory', JSON.stringify(chatHistory));
  input.value = '';
  loadMessages();
}
</script>
</body>
</html>
