<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WhatsApp Clone - Video & Chat</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; background: #e5ddd5; display: flex; flex-direction: column; height: 100vh; }
        header { background: #075E54; color: white; padding: 15px; text-align: center; font-weight: bold; }
        #id-display { background: #fff; padding: 10px; text-align: center; font-size: 13px; border-bottom: 1px solid #ddd; word-break: break-all; }
        #video-container { display: flex; background: #000; height: 180px; gap: 5px; padding: 5px; }
        video { width: 50%; height: 100%; object-fit: cover; border-radius: 8px; background: #222; }
        #chat-box { flex: 1; overflow-y: auto; padding: 15px; display: flex; flex-direction: column; gap: 10px; }
        .msg { padding: 8px 12px; border-radius: 8px; max-width: 80%; font-size: 14px; }
        .sent { align-self: flex-end; background: #dcf8c6; }
        .received { align-self: flex-start; background: white; }
        .controls { background: #f0f0f0; padding: 10px; display: flex; flex-direction: column; gap: 8px; }
        .row { display: flex; gap: 5px; }
        input { flex: 1; padding: 10px; border-radius: 20px; border: 1px solid #ddd; outline: none; }
        button { background: #128C7E; color: white; border: none; padding: 10px 15px; border-radius: 20px; font-weight: bold; cursor: pointer; }
    </style>
</head>
<body>

<header>WhatsApp Video Chat</header>
<div id="id-display">আপনার আইডি: <b id="my-id">অপেক্ষা করুন...</b></div>

<div id="video-container">
    <video id="localVideo" autoplay muted playsinline></video>
    <video id="remoteVideo" autoplay playsinline></video>
</div>

<div id="chat-box"></div>

<div class="controls">
    <div class="row">
        <input type="text" id="targetId" placeholder="বন্ধুর আইডি দিন">
        <button onclick="startApp()">Connect/Call</button>
    </div>
    <div class="row">
        <input type="text" id="msgInput" placeholder="মেসেজ লিখুন...">
        <button onclick="sendMsg()">Send</button>
    </div>
</div>

<script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>

<script>
    let localStream;
    let currentConn;

    // ১. PeerJS কানেকশন সেটআপ
    const peer = new Peer(undefined, {
        config: {'iceServers': [
            { url: 'stun:stun.l.google.com:19302' },
            { url: 'stun:stun1.l.google.com:19302' }
        ]}
    });

    peer.on('open', id => {
        document.getElementById('my-id').innerText = id;
    });

    // ২. ক্যামেরা চালু করা
    navigator.mediaDevices.getUserMedia({ video: true, audio: true }).then(stream => {
        localStream = stream;
        document.getElementById('localVideo').srcObject = stream;
    }).catch(err => alert("ক্যামেরা পারমিশন দিন!"));

    // ৩. কল ও চ্যাট রিসিভ করা
    peer.on('call', call => {
        call.answer(localStream);
        call.on('stream', remoteStream => {
            document.getElementById('remoteVideo').srcObject = remoteStream;
        });
    });

    peer.on('connection', conn => {
        currentConn = conn;
        setupChat();
    });

    // ৪. কল ও চ্যাট শুরু করা
    function startApp() {
        const friendId = document.getElementById('targetId').value;
        if(!friendId) return alert("বন্ধুর আইডি দিন!");

        // ভিডিও কল
        const call = peer.call(friendId, localStream);
        call.on('stream', remoteStream => {
            document.getElementById('remoteVideo').srcObject = remoteStream;
        });

        // চ্যাট কানেকশন
        currentConn = peer.connect(friendId);
        setupChat();
    }

    function setupChat() {
        currentConn.on('data', data => {
            appendMessage(data, 'received');
        });
        currentConn.on('open', () => console.log("Connected for chat"));
    }

    function sendMsg() {
        const msg = document.getElementById("msgInput").value;
        if (msg && currentConn) {
            currentConn.send(msg);
            appendMessage(msg, 'sent');
            document.getElementById("msgInput").value = "";
        } else {
            alert("আগে Connect করুন!");
        }
    }

    function appendMessage(text, type) {
        const div = document.createElement("div");
        div.classList.add("msg", type);
        div.innerText = text;
        const chatBox = document.getElementById("chat-box");
        chatBox.appendChild(div);
        chatBox.scrollTop = chatBox.scrollHeight;
    }

    peer.on('error', err => alert("Error: " + err.type));
</script>
</body>
</html>
