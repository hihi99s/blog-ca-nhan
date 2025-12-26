+++
title = "Bảo mật cơ bản trong Giao tiếp Mạng"
date = "2025-12-12T10:00:00+07:00"
draft = false
tags = ["Security", "HTTPS", "SSL", "Network"]
categories = ["Lập trình mạng"]
+++

# Bảo mật cơ bản trong Giao tiếp Mạng

Bảo mật là yếu tố quan trọng hàng đầu trong lập trình mạng. Bài viết này sẽ giới thiệu các khái niệm và kỹ thuật bảo mật cơ bản khi phát triển ứng dụng web và mạng.

## Tại sao cần Bảo mật Mạng?

### Các mối đe dọa phổ biến

```
┌───────────────┐
│   ATTACKER    │
└───────┬───────┘
        │
        ▼
┌───────────────────────────────────────────────────┐
│                    INTERNET                        │
│  ┌─────────────┐                ┌─────────────┐   │
│  │ Eavesdrop   │   ┌────────┐   │ Man-in-the  │   │
│  │ (Nghe lén)  │◄──│ Data   │──►│ Middle      │   │
│  └─────────────┘   └────────┘   └─────────────┘   │
│                                                    │
│  ┌─────────────┐                ┌─────────────┐   │
│  │ Replay      │                │ Data        │   │
│  │ Attack      │                │ Tampering   │   │
│  └─────────────┘                └─────────────┘   │
└───────────────────────────────────────────────────┘
```

1. **Eavesdropping (Nghe lén)**: Đọc dữ liệu truyền tải
2. **Man-in-the-Middle**: Chặn và sửa đổi dữ liệu
3. **Replay Attack**: Gửi lại request cũ
4. **Data Tampering**: Thay đổi dữ liệu

## HTTP vs HTTPS

### HTTP - Không an toàn

```
Client ──────── Plain Text ────────► Server
         "password=123456"
              ↑
         Attacker có thể đọc!
```

### HTTPS - An toàn

```
Client ──────── Encrypted ────────► Server
         "x#$%^&*@!..."
              ↑
         Attacker không hiểu!
```

## SSL/TLS là gì?

**SSL (Secure Sockets Layer)** và **TLS (Transport Layer Security)** là các giao thức mã hóa bảo vệ dữ liệu truyền tải.

### TLS Handshake

```
Client                                    Server
   │                                        │
   │──────── Client Hello ────────────────►│
   │         (Supported ciphers, TLS version)
   │                                        │
   │◄─────── Server Hello ─────────────────│
   │         (Chosen cipher, Certificate)  │
   │                                        │
   │         Verify Certificate             │
   │         Generate Pre-Master Secret     │
   │                                        │
   │──────── Key Exchange ────────────────►│
   │                                        │
   │         Generate Session Keys          │
   │                                        │
   │◄═══════ Encrypted Communication ══════│
```

### Các thành phần

1. **Certificate**: Chứng chỉ xác thực server
2. **Public Key**: Khóa công khai để mã hóa
3. **Private Key**: Khóa riêng để giải mã
4. **Session Key**: Khóa phiên cho symmetric encryption

## Các phương pháp Mã hóa

### Symmetric Encryption (Mã hóa đối xứng)

Cùng một key để mã hóa và giải mã:

```
┌─────────────┐     Key      ┌─────────────┐
│ Plain Text  │────────────►│ Cipher Text │
│ "Hello"     │   Encrypt   │ "x8#k@"     │
└─────────────┘              └─────────────┘

┌─────────────┐     Key      ┌─────────────┐
│ Cipher Text │────────────►│ Plain Text  │
│ "x8#k@"     │   Decrypt   │ "Hello"     │
└─────────────┘              └─────────────┘
```

**Ví dụ**: AES, DES, 3DES

```javascript
// Ví dụ với Web Crypto API
async function encryptAES(plainText, key) {
    const encoder = new TextEncoder();
    const data = encoder.encode(plainText);
    
    const cryptoKey = await crypto.subtle.importKey(
        'raw',
        key,
        { name: 'AES-GCM', length: 256 },
        false,
        ['encrypt']
    );
    
    const iv = crypto.getRandomValues(new Uint8Array(12));
    const encrypted = await crypto.subtle.encrypt(
        { name: 'AES-GCM', iv },
        cryptoKey,
        data
    );
    
    return { encrypted, iv };
}
```

### Asymmetric Encryption (Mã hóa bất đối xứng)

Dùng cặp Public Key và Private Key:

```
                         ┌─────────────┐
                         │ Public Key  │◄── Ai cũng có
                         └──────┬──────┘
                                │ Encrypt
                                ▼
┌─────────────┐         ┌─────────────┐
│ Plain Text  │────────►│ Cipher Text │
└─────────────┘         └──────┬──────┘
                               │ Decrypt
                               ▼
                        ┌─────────────┐
                        │ Private Key │◄── Chỉ server có
                        └─────────────┘
```

**Ví dụ**: RSA, ECC

### Hashing (Băm)

Chuyển đổi một chiều, không thể giải mã:

```
"password123" ──► SHA256 ──► "ef92b778bafe771e89245b89ecb..."
                                      ↑
                              Không thể đảo ngược!
```

```javascript
// Hash password với SHA-256
async function hashPassword(password) {
    const encoder = new TextEncoder();
    const data = encoder.encode(password);
    const hashBuffer = await crypto.subtle.digest('SHA-256', data);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}

// Sử dụng
const hash = await hashPassword('mypassword');
console.log(hash);
```

## Authentication & Authorization

### Authentication (Xác thực)

Xác minh "Bạn là ai?"

```javascript
// JWT Authentication
async function login(email, password) {
    const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
    });
    
    if (response.ok) {
        const { token } = await response.json();
        localStorage.setItem('jwt', token);
        return true;
    }
    return false;
}

// Gửi request có xác thực
async function authenticatedRequest(url) {
    const token = localStorage.getItem('jwt');
    
    return fetch(url, {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    });
}
```

### Authorization (Phân quyền)

Xác minh "Bạn được phép làm gì?"

```javascript
// Server-side authorization check
function checkPermission(user, resource, action) {
    const permissions = {
        admin: ['read', 'write', 'delete'],
        editor: ['read', 'write'],
        viewer: ['read']
    };
    
    return permissions[user.role]?.includes(action);
}
```

## Các lỗ hổng bảo mật phổ biến

### 1. SQL Injection

```javascript
// ❌ Nguy hiểm
const query = `SELECT * FROM users WHERE id = ${userId}`;
// userId = "1; DROP TABLE users;--" => Xóa toàn bộ bảng!

// ✅ An toàn - Parameterized Query
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);
```

### 2. XSS (Cross-Site Scripting)

```javascript
// ❌ Nguy hiểm
element.innerHTML = userInput;
// userInput = "<script>stealCookies()</script>"

// ✅ An toàn
element.textContent = userInput;

// Hoặc escape HTML
function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}
```

### 3. CSRF (Cross-Site Request Forgery)

```html
<!-- Attacker's website -->
<img src="https://yourbank.com/transfer?amount=1000&to=attacker" />
```

**Phòng chống**: CSRF Token

```javascript
// Server generate token
const csrfToken = generateRandomToken();
session.csrfToken = csrfToken;

// Client gửi kèm token
fetch('/api/transfer', {
    method: 'POST',
    headers: {
        'X-CSRF-Token': csrfToken
    },
    body: JSON.stringify(data)
});
```

## Best Practices

### 1. Luôn dùng HTTPS

```javascript
// Redirect HTTP to HTTPS (server-side)
if (req.protocol !== 'https') {
    res.redirect(`https://${req.hostname}${req.url}`);
}
```

### 2. Validate Input

```javascript
function validateEmail(email) {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
}

function validatePassword(password) {
    // Ít nhất 8 ký tự, có chữ hoa, chữ thường, số
    const regex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/;
    return regex.test(password);
}
```

### 3. Hash Passwords (không lưu plain text)

```javascript
// Sử dụng bcrypt (Node.js)
const bcrypt = require('bcrypt');

// Hash password
const hashedPassword = await bcrypt.hash(password, 10);

// Verify password
const isMatch = await bcrypt.compare(inputPassword, hashedPassword);
```

### 4. Secure Headers

```javascript
// Express.js với helmet
const helmet = require('helmet');
app.use(helmet());

// Các headers quan trọng:
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY
// Content-Security-Policy: ...
// Strict-Transport-Security: max-age=...
```

### 5. Rate Limiting

```javascript
// Express rate limit
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 phút
    max: 100 // 100 requests/IP
});

app.use('/api/', limiter);
```

### 6. Cors Configuration

```javascript
const cors = require('cors');

app.use(cors({
    origin: 'https://yourdomain.com',
    methods: ['GET', 'POST'],
    credentials: true
}));
```

## Checklist Bảo mật

- [ ] Sử dụng HTTPS
- [ ] Validate tất cả input
- [ ] Hash passwords với salt
- [ ] Implement authentication/authorization
- [ ] Protect against SQL Injection
- [ ] Protect against XSS
- [ ] Implement CSRF protection
- [ ] Set secure HTTP headers
- [ ] Rate limiting
- [ ] Log security events
- [ ] Keep dependencies updated

## Kết luận

**Bảo mật** là yếu tố không thể thiếu trong lập trình mạng. Hiểu về HTTPS/TLS, mã hóa, và các lỗ hổng phổ biến giúp bạn xây dựng ứng dụng an toàn. Luôn áp dụng các best practices và cập nhật kiến thức bảo mật thường xuyên.

---

*Bài viết thuộc series Lập trình mạng - Blog Ngô Phạm Ngọc Tú*
