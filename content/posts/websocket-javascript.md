+++
title = "WebSocket trong JavaScript: Giao tiếp Realtime"
date = "2025-12-16T10:00:00+07:00"
draft = false
tags = ["JavaScript", "WebSocket", "Realtime", "Web"]
categories = ["JavaScript"]
+++

# WebSocket trong JavaScript: Giao tiếp Realtime

**WebSocket** là giao thức cho phép giao tiếp hai chiều (full-duplex) giữa client và server thông qua một kết nối TCP duy nhất. Đây là công nghệ quan trọng cho các ứng dụng realtime như chat, game online, live updates.

## Tại sao cần WebSocket?

### Vấn đề với HTTP truyền thống

HTTP là giao thức **request-response**: client gửi request, server trả response, kết nối đóng.

```
Polling truyền thống:
Client ──► Server: "Có tin nhắn mới không?"
Server ──► Client: "Không"
(1 giây sau)
Client ──► Server: "Có tin nhắn mới không?"
Server ──► Client: "Không"
(1 giây sau)
Client ──► Server: "Có tin nhắn mới không?"
Server ──► Client: "Có! Đây là tin nhắn..."
```

**Vấn đề:** Lãng phí bandwidth, server overload, độ trễ cao.

### Giải pháp WebSocket

```
WebSocket:
Client ◄══► Server: Kết nối liên tục
Server ──► Client: "Có tin nhắn mới!" (push ngay lập tức)
Client ──► Server: "Tôi muốn gửi tin nhắn"
Server ──► Client: "User A đang typing..."
```

**Ưu điểm:** Realtime, tiết kiệm bandwidth, độ trễ thấp.

## WebSocket Handshake

WebSocket bắt đầu bằng HTTP upgrade request:

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

Server response:
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

Sau handshake, kết nối chuyển sang WebSocket protocol.

## Sử dụng WebSocket trong JavaScript

### Tạo kết nối

```javascript
// Tạo WebSocket connection
const socket = new WebSocket('ws://localhost:8080');

// Hoặc với SSL (wss://)
const secureSocket = new WebSocket('wss://secure.server.com/socket');
```

### Các Event của WebSocket

```javascript
const socket = new WebSocket('ws://localhost:8080');

// Khi kết nối thành công
socket.onopen = function(event) {
    console.log('Đã kết nối WebSocket!');
    socket.send('Xin chào Server!');
};

// Khi nhận được message
socket.onmessage = function(event) {
    console.log('Nhận được:', event.data);
    
    // Parse JSON nếu server gửi JSON
    const data = JSON.parse(event.data);
    console.log(data);
};

// Khi có lỗi
socket.onerror = function(error) {
    console.error('WebSocket Error:', error);
};

// Khi kết nối đóng
socket.onclose = function(event) {
    if (event.wasClean) {
        console.log(`Đóng sạch, code=${event.code}, reason=${event.reason}`);
    } else {
        console.log('Kết nối bị ngắt đột ngột');
    }
};
```

### Gửi dữ liệu

```javascript
// Gửi text
socket.send('Hello World!');

// Gửi JSON
const message = {
    type: 'chat',
    content: 'Xin chào!',
    sender: 'user123',
    timestamp: Date.now()
};
socket.send(JSON.stringify(message));

// Gửi binary data
const buffer = new ArrayBuffer(8);
socket.send(buffer);

// Gửi Blob
const blob = new Blob(['binary data']);
socket.send(blob);
```

### Kiểm tra trạng thái kết nối

```javascript
// WebSocket.readyState
// 0 - CONNECTING: Đang kết nối
// 1 - OPEN: Đã kết nối, có thể gửi/nhận
// 2 - CLOSING: Đang đóng
// 3 - CLOSED: Đã đóng

if (socket.readyState === WebSocket.OPEN) {
    socket.send('Message');
}
```

### Đóng kết nối

```javascript
// Đóng bình thường
socket.close();

// Đóng với code và reason
socket.close(1000, 'Người dùng thoát');
```

## Ví dụ: Chat Application đơn giản

### Client Side (JavaScript)

```javascript
class ChatClient {
    constructor(serverUrl) {
        this.serverUrl = serverUrl;
        this.socket = null;
        this.messageHandlers = [];
    }
    
    connect() {
        return new Promise((resolve, reject) => {
            this.socket = new WebSocket(this.serverUrl);
            
            this.socket.onopen = () => {
                console.log('Đã kết nối chat server');
                resolve();
            };
            
            this.socket.onerror = (error) => {
                reject(error);
            };
            
            this.socket.onmessage = (event) => {
                const data = JSON.parse(event.data);
                this.handleMessage(data);
            };
            
            this.socket.onclose = () => {
                console.log('Mất kết nối, đang reconnect...');
                setTimeout(() => this.connect(), 3000);
            };
        });
    }
    
    handleMessage(data) {
        switch(data.type) {
            case 'chat':
                this.displayMessage(data);
                break;
            case 'user_joined':
                console.log(`${data.username} đã tham gia`);
                break;
            case 'user_left':
                console.log(`${data.username} đã rời đi`);
                break;
            case 'typing':
                this.showTypingIndicator(data.username);
                break;
        }
    }
    
    sendMessage(content) {
        if (this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(JSON.stringify({
                type: 'chat',
                content: content,
                timestamp: Date.now()
            }));
        }
    }
    
    sendTyping() {
        if (this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(JSON.stringify({
                type: 'typing'
            }));
        }
    }
    
    displayMessage(data) {
        const messagesDiv = document.getElementById('messages');
        const messageEl = document.createElement('div');
        messageEl.className = 'message';
        messageEl.innerHTML = `
            <strong>${data.sender}:</strong> ${data.content}
            <span class="time">${new Date(data.timestamp).toLocaleTimeString()}</span>
        `;
        messagesDiv.appendChild(messageEl);
        messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }
    
    showTypingIndicator(username) {
        const indicator = document.getElementById('typing-indicator');
        indicator.textContent = `${username} đang nhập...`;
        setTimeout(() => {
            indicator.textContent = '';
        }, 2000);
    }
}

// Sử dụng
const chat = new ChatClient('ws://localhost:8080/chat');

chat.connect().then(() => {
    console.log('Chat ready!');
});

// Gửi tin nhắn
document.getElementById('send-btn').addEventListener('click', () => {
    const input = document.getElementById('message-input');
    chat.sendMessage(input.value);
    input.value = '';
});

// Typing indicator
document.getElementById('message-input').addEventListener('input', () => {
    chat.sendTyping();
});
```

### HTML Structure

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>WebSocket Chat</title>
    <style>
        #messages {
            height: 400px;
            overflow-y: auto;
            border: 1px solid #ccc;
            padding: 10px;
        }
        .message {
            margin: 10px 0;
            padding: 8px;
            background: #f0f0f0;
            border-radius: 8px;
        }
        .time {
            font-size: 0.8em;
            color: #666;
        }
        #typing-indicator {
            color: #999;
            font-style: italic;
        }
    </style>
</head>
<body>
    <h1>Chat Room</h1>
    <div id="messages"></div>
    <div id="typing-indicator"></div>
    <input type="text" id="message-input" placeholder="Nhập tin nhắn...">
    <button id="send-btn">Gửi</button>
    <script src="chat.js"></script>
</body>
</html>
```

## WebSocket vs HTTP Polling vs Server-Sent Events

| Tiêu chí | WebSocket | HTTP Polling | SSE |
|----------|-----------|--------------|-----|
| **Chiều** | Hai chiều | Một chiều (client→server) | Một chiều (server→client) |
| **Kết nối** | Liên tục | Ngắt quãng | Liên tục |
| **Realtime** | Có | Không | Có |
| **Browser Support** | Tốt | Tất cả | Tốt (trừ IE) |
| **Use case** | Chat, Gaming | Fallback | Live feeds |

## Best Practices

### 1. Xử lý Reconnection

```javascript
function createWebSocket() {
    const socket = new WebSocket(url);
    
    socket.onclose = () => {
        console.log('Đang reconnect...');
        setTimeout(createWebSocket, 3000);
    };
    
    return socket;
}
```

### 2. Heartbeat/Ping-Pong

```javascript
setInterval(() => {
    if (socket.readyState === WebSocket.OPEN) {
        socket.send(JSON.stringify({ type: 'ping' }));
    }
}, 30000);
```

### 3. Message Queue khi offline

```javascript
const messageQueue = [];

function sendMessage(msg) {
    if (socket.readyState === WebSocket.OPEN) {
        socket.send(msg);
    } else {
        messageQueue.push(msg);
    }
}

socket.onopen = () => {
    while (messageQueue.length > 0) {
        socket.send(messageQueue.shift());
    }
};
```

## Kết luận

**WebSocket** là công nghệ quan trọng cho ứng dụng realtime trong JavaScript. Nó cho phép giao tiếp hai chiều hiệu quả giữa client và server, phù hợp cho chat, gaming, live updates, và nhiều ứng dụng khác cần độ trễ thấp.

---

*Bài viết thuộc series JavaScript cho lập trình mạng - Blog Ngô Phạm Ngọc Tú*
