
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>F.R.I.E.N.D.S Chat</title>
<style>
body { margin:0; font-family: Arial,sans-serif; background:#1e1f22; color:#fff; }
#loginPage, #chatPage { display:none; height:100vh; }
#loginPage { display:flex; flex-direction:column; justify-content:center; align-items:center; }
.user-btn { border:none; color:#fff; padding:15px 25px; margin:10px; border-radius:10px; font-size:18px; cursor:pointer; transition:0.2s; }
.user-btn.Arjun { background:#facc15; color:#000; }
.user-btn.Akshat { background:#ef4444; }
.user-btn.Narain { background:#3b82f6; }
#chatPage { display:flex; height:100vh; }
#sidebar { width:250px; background:#2b2d31; display:flex; flex-direction:column; padding:10px; align-items:center; }
#sidebar img { width:80px; height:80px; border-radius:50%; margin-bottom:10px; object-fit:cover; border:3px solid #444; }
#sidebar h3 { text-align:center; border-bottom:1px solid #444; padding-bottom:10px; width:100%; }
#main { flex:1; display:flex; flex-direction:column; }
#messages { flex:1; padding:10px; overflow-y:auto; }
.msg { display:flex; align-items:center; margin:5px 0; padding:10px; border-radius:10px; background:#40444B; flex-wrap:wrap; }
.msg img { width:35px; height:35px; border-radius:50%; margin-right:10px; object-fit:cover; }
.msg .name { font-weight:bold; margin-right:5px; }
#inputArea { display:flex; padding:10px; background:#2b2d31; flex-wrap:wrap; gap:5px; }
#inputArea input, #inputArea button, #inputArea label { padding:10px; border-radius:5px; border:none; cursor:pointer; }
#inputArea input { flex:1; }
#inputArea button, #inputArea label { background:#5865F2; color:white; }
</style>
</head>
<body>

<div id="loginPage">
  <h1>Login as:</h1>
  <button class="user-btn Arjun" onclick="login('Arjun')">Arjun</button>
  <button class="user-btn Akshat" onclick="login('Akshat')">Akshat</button>
  <button class="user-btn Narain" onclick="login('Narain')">Narain</button>
  <input type="file" id="uploadPicBtn" accept="image/*">
  <div class="note">Profile picture is optional.</div>
</div>

<div id="chatPage">
  <div id="sidebar">
    <img id="profilePic" src="https://via.placeholder.com/80">
    <h3 id="userName"></h3>
    <input type="file" id="changePicBtn" accept="image/*">
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
let colors = { Arjun:'#facc15', Akshat:'#ef4444', Narain:'#3b82f6' };

// Store per-user chat and pictures in localStorage
let usersData = JSON.parse(localStorage.getItem('usersData')) || {};

function login(name){
  currentUser = name;
  loginPage.style.display='none';
  chatPage.style.display='flex';
  const userNameElem = document.getElementById('userName');
  userNameElem.textContent = name;
  userNameElem.style.color = colors[name];
  
  if(!usersData[currentUser]) usersData[currentUser]={messages:[],pic:null};
  profilePicElem.src = usersData[currentUser].pic || 'https://via.placeholder.com/80';
  loadMessages();
}

const profilePicElem = document.getElementById('profilePic');
const changePicBtn = document.getElementById('changePicBtn');

changePicBtn.addEventListener('change', e=>{
  const file = e.target.files[0];
  if(file){
    const reader = new FileReader();
    reader.onload = evt=>{
      usersData[currentUser].pic = evt.target.result;
      localStorage.setItem('usersData', JSON.stringify(usersData));
      profilePicElem.src = evt.target.result;
      loadMessages();
    };
    reader.readAsDataURL(file);
  }
});

function sendMessage(){
  const input = document.getElementById('msgInput');
  const text = input.value.trim();
  if(!text) return;
  usersData[currentUser].messages.push({text,date:Date.now()});
  localStorage.setItem('usersData', JSON.stringify(usersData));
  input.value='';
  loadMessages();
}

function loadMessages(){
  const messagesDiv = document.getElementById('messages');
  messagesDiv.innerHTML='';
  const messages = usersData[currentUser].messages;
  messages.forEach(msg=>{
    const div = document.createElement('div');
    div.className='msg';
    const img = document.createElement('img');
    img.src = usersData[currentUser].pic || 'https://via.placeholder.com/35';
    div.appendChild(img);
    const nameSpan = document.createElement('span');
    nameSpan.textContent = currentUser + ': ';
    nameSpan.className='name';
    nameSpan.style.color = colors[currentUser];
    div.appendChild(nameSpan);
    div.appendChild(document.createTextNode(msg.text));
    messagesDiv.appendChild(div);
  });
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
}
</script>
</body>
</html>
