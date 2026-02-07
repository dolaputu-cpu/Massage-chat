<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WhatsApp Clone - Video Call</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; background: #e5ddd5; display: flex; flex-direction: column; height: 100vh; }
        header { background: #075E54; color: white; padding: 15px; text-align: center; font-size: 20px; }
        #peer-id-display { background: #dcf8c6; padding: 10px; text-align: center; font-weight: bold; border-bottom: 1px solid #ccc; }
        #video-container { display: flex; justify-content: center; gap: 10px; background: #000; padding: 10px; }
        video { width: 45%; border-radius: 10px; background: #333; border: 2px solid #128C7E; }
        #chat-box { flex: 1; overflow-y: auto; padding: 20px; display: flex; flex-direction: column; }
        .message { background: white; padding: 10px; border-radius: 10px; margin-bottom: 10px; max-width: 70%; box-shadow: 0 1px 2px rgba(0,0,0,0.1); }
        .controls { background: #f0f0f0; padding: 10px; display: flex; gap: 10px; }
        input { flex: 1; padding: 12px; border-radius: 25px; border: 1px solid #ccc; outline: none; }
        button { background: #128C7E; color: white; border: none; padding: 10px 20px; border-radius: 25px; cursor: pointer; }
    </style>
</head>
<body>

<header>WhatsApp Video Chat</header>
<div id="peer-id-display">আপনার আইডি: <span id="my-id">অপেক্ষা করুন...</span></div>

<div id="video-container">
    <video id="localVideo" autoplay muted playsinline></video>
    <video id="remoteVideo" autoplay playsinline></video>
</div>

<div id="chat-box"></div>

<div class="controls">
    <input type="text" id="targetId" placeholder="বন্ধুর আইডি দিন">
    <button onclick="makeCall()">📞 Call</button>
</div>

<div class="controls">
    <input type="text" id="msgInput" placeholder="মেসেজ লিখুন...">
    <button onclick="sendMsg()">Send</button>
</div>

<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
<script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>

<script>
    // ১. Firebase Config
    const firebaseConfig = {
        apiKey: "AIzaSyD8g7xWChWPNs_6Cx10kg50PrnFw_0tPLw",
        authDomain: "massagechatapps.firebaseapp.com",
        databaseURL: "https://massagechatapps-default-rtdb.firebaseio.com",
        projectId: "massagechatapps",
        storageBucket: "massagechatapps.firebasestorage.app",
        messagingSenderId: "69587961177",
        appId: "1:69587961177:web:43476ee94350218948f848"
    };
    firebase.initializeApp(firebaseConfig);
    const database = firebase.database();

    // ২. PeerJS Setup
    const peer = new Peer(undefined, {
        host: '0.peerjs.com',
        port: 443,
        secure: true
    });

    let localStream;

    peer.on('open', id => {
        document.getElementById('my-id').innerText = id;
    });

    // ক্যামেরা চালু করা
    navigator.mediaDevices.getUserMedia({ video: true, audio: true }).then(stream => {
        localStream = stream;
        document.getElementById('localVideo').srcObject = stream;
    }).catch(err => alert("ক্যামেরা পারমিশন দিন!"));

    // কল রিসিভ করা
    peer.on('call', call => {
        call.answer(localStream);
        call.on('stream', remoteStream => {
            document.getElementById('remoteVideo').srcObject = remoteStream;
        });
    });

    // কল দেওয়া
    function makeCall() {
        const friendId = document.getElementById('targetId').value;
        if(!friendId) return alert("আইডি দিন");
        const call = peer.call(friendId, localStream);
        call.on('stream', remoteStream => {
            document.getElementById('remoteVideo').srcObject = remoteStream;
        });
    }

    // ৩. চ্যাট ফাংশন
    function sendMsg() {
        const text = document.getElementById("msgInput").value;
        if (text) {
            database.ref("messages").push().set({ message: text });
            document.getElementById("msgInput").value = "";
        }
    }

    database.ref("messages").on("child_added", snapshot => {
        const msgDiv = document.createElement("div");
        msgDiv.classList.add("message");
        msgDiv.innerText = snapshot.val().message;
        document.getElementById("chat-box").appendChild(msgDiv);
    });
</script>
</body>
</html>
