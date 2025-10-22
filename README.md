<!DOCTYPE html>
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
.user-btn.Arjun:hover { background:#eab308; }
.user-btn.Akshat { background:#ef4444; }
.user-btn.Akshat:hover { background:#dc2626; }
.user-btn.Narain { background:#3b82f6; }
.user-btn.Narain:hover { background:#2563eb; }
#chatPage { display:flex; height:100vh; }
#sidebar { width:250px; background:#2b2d31; display:flex; flex-direction:column; padding:10px; align-items:center; }
#sidebar h3 { text-align:center; border-bottom:1px solid #444; padding-bottom:10px; width:100%; }
#sidebar img { width:80px; height:80px; border-radius:50%; margin-bottom:10px; object-fit:cover; border:3px solid #444; }
#sidebar button, #sidebar input, #sidebar label { width:90%; margin:5px 0; border:none; border-radius:5px; padding:10px; cursor:pointer; background:#5865F2; color:white; }
#main { flex:1; display:flex; flex-direction:column; }
#messages { flex:1; padding:10px; overflow-y:auto; }
.msg { display:flex; align-items:center; margin:5px 0; padding:10px; border-radius:10px; background:#40444B; flex-wrap:wrap; }
.msg img { width:35px; height:35px; border-radius:50%; margin-right:10px; object-fit:cover; }
.msg .name { font-weight:bold; margin-right:5px; }
#inputArea { display:flex; padding:10px; background:#2b2d31; flex-wrap:wrap; gap:5px; }
#inputArea input, #inputArea button, #inputArea label { padding:10px; border-radius:5px; border:none; cursor:pointer; }
#inputArea input { flex:1; }
#inputArea button, #inputArea label { background:#5865F2; color:white; }
.poll { background:#52555b; padding:10px; border-radius:8px; margin:5px 0; width:100%; }
.poll button { margin:2px; background:#3b82f6; color:white; border:none; padding:5px 8px; border-radius:4px; cursor:pointer; }
</style>
</head>
<body>

<!-- Firebase SDKs -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>

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
    <label for="changePicBtn">Change Picture</label>
    <input type="file" id="changePicBtn" accept="image/*" style="display:none">
    <button id="joinVoiceBtn">Join Voice Chat</button>
    <button id="leaveVoiceBtn" disabled>Leave Voice Chat</button>
    <input type="file" id="fileUploadBtn">
    <button onclick="createPoll()">Create Poll</button>
  </div>
  <div id="main">
    <div id="messages"></div>
    <div id="inputArea">
      <input id="msgInput" placeholder="Type a message..." onkeydown="if(event.key==='Enter')sendMessage()">
      <button onclick="sendMessage()">Send</button>
      <label for="fileUploadBtn">Send File</label>
    </div>
  </div>
</div>

<script>
// Initialize Firebase
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT_ID.firebaseio.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "XXXX",
  appId: "XXXX"
};
firebase.initializeApp(firebaseConfig);

let currentUser = null;
let chatHistory = [];
let userPics = {};
const colors = { Arjun:'#facc15', Akshat:'#ef4444', Narain:'#3b82f6' };

const loginPage = document.getElementById('loginPage');
const chatPage = document.getElementById('chatPage');
const messagesDiv = document.getElementById('messages');
const uploadPicBtn = document.getElementById('uploadPicBtn');
const changePicBtn = document.getElementById('changePicBtn');
const fileUploadBtn = document.getElementById('fileUploadBtn');
const profilePicElem = document.getElementById('profilePic');

// Firebase references
const chatRef = firebase.database().ref('groupChat');
const picsRef = firebase.database().ref('userPics');

function login(name){
  currentUser = name;
  loginPage.style.display='none';
  chatPage.style.display='flex';
  document.getElementById('userName').textContent = name;
  document.getElementById('userName').style.color = colors[name];
  profilePicElem.src = userPics[currentUser] || 'https://via.placeholder.com/80';
  loadMessages();
}

// Load data from Firebase
chatRef.on('value', snapshot=>{
  chatHistory = snapshot.val() || [];
  loadMessages();
});

picsRef.on('value', snapshot=>{
  userPics = snapshot.val() || {};
  if(currentUser) profilePicElem.src = userPics[currentUser] || 'https://via.placeholder.com/80';
  loadMessages();
});

// Upload / Change profile picture
function uploadProfile(file){
  const reader = new FileReader();
  reader.onload = evt=>{
    userPics[currentUser] = evt.target.result;
    picsRef.set(userPics);
    profilePicElem.src = evt.target.result;
    loadMessages();
  };
  reader.readAsDataURL(file);
}

uploadPicBtn.addEventListener('change', e=>{
  if(e.target.files[0]) uploadProfile(e.target.files[0]);
});
changePicBtn.addEventListener('change', e=>{
  if(e.target.files[0]) uploadProfile(e.target.files[0]);
});

// Send message
function sendMessage(){
  const input = document.getElementById('msgInput');
  const text = input.value.trim();
  if(!text) return;
  chatHistory.push({user:currentUser,text});
  chatRef.set(chatHistory);
  input.value='';
  loadMessages();
}

// Send file
fileUploadBtn.addEventListener('change', e=>{
  const file = e.target.files[0];
  if(file){
    const reader = new FileReader();
    reader.onload = evt=>{
      chatHistory.push({user:currentUser,file:{name:file.name,data:evt.target.result}});
      chatRef.set(chatHistory);
      loadMessages();
    };
    reader.readAsDataURL(file);
  }
});

// Polls
function createPoll(){
  const question = prompt('Enter poll question:');
  if(!question) return;
  const options = prompt('Enter options separated by comma:');
  if(!options) return;
  const opts = options.split(',').map(o=>({option:o.trim(),votes:0}));
  chatHistory.push({user:currentUser,poll:{question,options:opts}});
  chatRef.set(chatHistory);
  loadMessages();
}

function votePoll(index,optIndex){
  chatHistory[index].poll.options[optIndex].votes++;
  chatRef.set(chatHistory);
  loadMessages();
}

// Render messages
function loadMessages(){
  messagesDiv.innerHTML='';
  chatHistory.forEach((m,i)=>{
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
    if(m.text) div.appendChild(document.createTextNode(m.text));
    if(m.file){
      const link = document.createElement('a');
      link.href = m.file.data;
      link.download = m.file.name;
      link.textContent = '[File] '+m.file.name;
      link.style.color='lightblue';
      link.style.marginLeft='5px';
      div.appendChild(link);
    }
    if(m.poll){
      const pollDiv = document.createElement('div');
      pollDiv.className='poll';
      const q = document.createElement('div');
      q.textContent = 'Poll: '+m.poll.question;
      pollDiv.appendChild(q);
      m.poll.options.forEach((o,oi)=>{
        const btn = document.createElement('button');
        btn.textContent = `${o.option} (${o.votes})`;
        btn.onclick = ()=>votePoll(i,oi);
        pollDiv.appendChild(btn);
      });
      div.appendChild(pollDiv);
    }
    messagesDiv.appendChild(div);
  });
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
}
</script>
</body>
</html>
