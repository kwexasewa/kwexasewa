<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ChatMe</title>
  <style>
    :root {
      --bg: #ffffff;
      --text: #000000;
      --msg-bg: #f1f1f1;
    }
    body.dark {
      --bg: #121212;
      --text: #ffffff;
      --msg-bg: #1e1e1e;
    }
    body {
      font-family: sans-serif;
      background-color: var(--bg);
      color: var(--text);
      padding: 20px;
      max-width: 600px;
      margin: auto;
      transition: background 0.3s, color 0.3s;
    }
    #messages {
      border: 1px solid #ccc;
      padding: 10px;
      height: 300px;
      overflow-y: auto;
      margin-bottom: 10px;
      background: var(--msg-bg);
      border-radius: 10px;
    }
    input, button {
      margin: 5px 0;
      padding: 10px;
      width: 100%;
      border-radius: 5px;
    }
    #chat { display: none; }
    .message {
      margin: 5px 0;
      padding: 8px;
      border-radius: 8px;
      background-color: var(--msg-bg);
    }
    img.chat-img {
      max-width: 100%;
      border-radius: 8px;
      margin-top: 5px;
    }
    .theme-toggle {
      margin-top: 10px;
      cursor: pointer;
      text-align: center;
    }
  </style>
</head>
<body>
  <h2>🔥 ChatMe ڤێرژنی تێست</h2>

  <div class="theme-toggle">
    <button onclick="toggleTheme()">🌓 تاریک و ڕوون</button>
  </div>

  <div id="auth">
    <input type="email" id="email" placeholder="ئیمەیڵ">
    <input type="password" id="password" placeholder="وشەی نهێنی">
    <button onclick="register()">خۆتۆمارکردن</button>
    <button onclick="login()">چوونەژوورەوە</button>
  </div>

  <div id="chat">
    <div id="messages"></div>
    <input type="text" id="messageInput" placeholder="چی لە دڵتایە؟">
    <input type="file" id="imageInput">
    <button onclick="sendMessage()">بینێرە</button>
    <button onclick="logout()">ماڵئاوایی</button>
  </div>

  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-storage-compat.js"></script>

  <!-- Your Firebase config -->
  <script>
    const firebaseConfig = {
      apiKey: "AIzaSyBDx5XIjLL8qKus69ePVGVo7mTgFXR_TyI",
      authDomain: "chatme-app-22332.firebaseapp.com",
      projectId: "chatme-app-22332",
      storageBucket: "chatme-app-22332.appspot.com",
      messagingSenderId: "YOUR_SENDER_ID",
      appId: "YOUR_APP_ID"
    };
    firebase.initializeApp(firebaseConfig);
  </script>

  <script>
    const auth = firebase.auth();
    const db = firebase.firestore();
    const storage = firebase.storage();
    const messagesRef = db.collection("messages");

    function register() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      auth.createUserWithEmailAndPassword(email, password)
        .then(() => alert("Registered!"))
        .catch(err => alert(err.message));
    }

    function login() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      auth.signInWithEmailAndPassword(email, password)
        .then(() => {
          document.getElementById("auth").style.display = "none";
          document.getElementById("chat").style.display = "block";
          listenForMessages();
        })
        .catch(err => alert(err.message));
    }

    function logout() {
      auth.signOut();
      location.reload();
    }

    function sendMessage() {
      const input = document.getElementById("messageInput");
      const imageInput = document.getElementById("imageInput");
      const text = input.value;
      const file = imageInput.files[0];
      input.value = "";
      imageInput.value = "";

      if (file) {
        const ref = storage.ref().child('images/' + Date.now() + '_' + file.name);
        ref.put(file).then(snapshot => {
          snapshot.ref.getDownloadURL().then(url => {
            messagesRef.add({
              text,
              imageUrl: url,
              sender: auth.currentUser.email,
              createdAt: firebase.firestore.FieldValue.serverTimestamp()
            });
          });
        });
      } else {
        messagesRef.add({
          text,
          sender: auth.currentUser.email,
          createdAt: firebase.firestore.FieldValue.serverTimestamp()
        });
      }
    }

    function listenForMessages() {
      messagesRef.orderBy("createdAt").onSnapshot(snapshot => {
        const messages = document.getElementById("messages");
        messages.innerHTML = "";
        snapshot.forEach(doc => {
          const msg = doc.data();
          const div = document.createElement("div");
          div.className = "message";
          div.innerHTML = `<strong>${msg.sender}</strong>: ${msg.text || ""}`;
          if (msg.imageUrl) {
            const img = document.createElement("img");
            img.src = msg.imageUrl;
            img.className = "chat-img";
            div.appendChild(img);
          }
          messages.appendChild(div);
        });
        messages.scrollTop = messages.scrollHeight;
      });
    }

    function toggleTheme() {
      document.body.classList.toggle("dark");
    }
  </script>
</body>
</html>
