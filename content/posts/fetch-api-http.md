+++
title = "Fetch API và Giao tiếp HTTP trong JavaScript"
date = "2025-12-19T10:00:00+07:00"
draft = false
tags = ["JavaScript", "HTTP", "API", "Web"]
categories = ["JavaScript"]
+++

# Fetch API và Giao tiếp HTTP trong JavaScript

**Fetch API** là giao diện hiện đại trong JavaScript để thực hiện các HTTP request. Nó thay thế XMLHttpRequest (XHR) cũ với cú pháp đơn giản hơn và dựa trên Promise.

## HTTP là gì?

**HTTP (HyperText Transfer Protocol)** là giao thức truyền thông giữa client và server trên web. Mỗi HTTP request bao gồm:

- **Method** (GET, POST, PUT, DELETE, ...)
- **URL** (địa chỉ tài nguyên)
- **Headers** (thông tin bổ sung)
- **Body** (dữ liệu gửi đi - với POST, PUT)

## Cú pháp cơ bản của Fetch

```javascript
fetch(url, options)
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));
```

Hoặc với Async/Await:

```javascript
async function getData() {
    try {
        const response = await fetch(url, options);
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error(error);
    }
}
```

## HTTP GET Request

GET dùng để **lấy dữ liệu** từ server:

```javascript
async function layDanhSachSanPham() {
    try {
        const response = await fetch('https://api.example.com/products');
        
        // Kiểm tra response thành công
        if (!response.ok) {
            throw new Error(`Lỗi HTTP: ${response.status}`);
        }
        
        const products = await response.json();
        console.log('Danh sách sản phẩm:', products);
        return products;
        
    } catch (error) {
        console.error('Lỗi khi lấy sản phẩm:', error);
    }
}

layDanhSachSanPham();
```

### GET với Query Parameters

```javascript
async function timKiemSanPham(keyword, page = 1) {
    // Tạo URL với query parameters
    const params = new URLSearchParams({
        q: keyword,
        page: page,
        limit: 10
    });
    
    const url = `https://api.example.com/products/search?${params}`;
    // URL: https://api.example.com/products/search?q=laptop&page=1&limit=10
    
    const response = await fetch(url);
    return response.json();
}

timKiemSanPham('laptop', 2);
```

## HTTP POST Request

POST dùng để **gửi dữ liệu** tới server:

```javascript
async function taoNguoiDungMoi(userData) {
    try {
        const response = await fetch('https://api.example.com/users', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(userData)
        });
        
        if (!response.ok) {
            throw new Error(`Lỗi: ${response.status}`);
        }
        
        const newUser = await response.json();
        console.log('Đã tạo user:', newUser);
        return newUser;
        
    } catch (error) {
        console.error('Lỗi khi tạo user:', error);
    }
}

// Sử dụng
taoNguoiDungMoi({
    name: 'Nguyễn Văn A',
    email: 'nguyenvana@example.com',
    password: 'matkhau123'
});
```

## HTTP PUT Request - Cập nhật dữ liệu

```javascript
async function capNhatNguoiDung(userId, updatedData) {
    try {
        const response = await fetch(`https://api.example.com/users/${userId}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(updatedData)
        });
        
        if (!response.ok) {
            throw new Error(`Lỗi: ${response.status}`);
        }
        
        return await response.json();
        
    } catch (error) {
        console.error('Lỗi cập nhật:', error);
    }
}

capNhatNguoiDung(123, { name: 'Tên mới' });
```

## HTTP DELETE Request - Xóa dữ liệu

```javascript
async function xoaNguoiDung(userId) {
    try {
        const response = await fetch(`https://api.example.com/users/${userId}`, {
            method: 'DELETE',
            headers: {
                'Authorization': 'Bearer your-token-here'
            }
        });
        
        if (!response.ok) {
            throw new Error(`Lỗi: ${response.status}`);
        }
        
        console.log('Đã xóa user thành công');
        return true;
        
    } catch (error) {
        console.error('Lỗi xóa:', error);
        return false;
    }
}

xoaNguoiDung(123);
```

## Xử lý Response

### Các phương thức parse response

```javascript
async function xuLyResponse() {
    const response = await fetch(url);
    
    // Parse JSON
    const json = await response.json();
    
    // Parse text
    const text = await response.text();
    
    // Parse Blob (file, image)
    const blob = await response.blob();
    
    // Parse ArrayBuffer (binary data)
    const buffer = await response.arrayBuffer();
    
    // Parse FormData
    const formData = await response.formData();
}
```

### Kiểm tra trạng thái Response

```javascript
async function kiemTraResponse() {
    const response = await fetch(url);
    
    console.log(response.status);      // 200, 404, 500, ...
    console.log(response.statusText);  // "OK", "Not Found", ...
    console.log(response.ok);          // true nếu status 200-299
    console.log(response.headers);     // Headers object
}
```

## Headers trong Request

```javascript
async function requestVoiHeaders() {
    const response = await fetch('https://api.example.com/data', {
        method: 'GET',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIs...',
            'X-Custom-Header': 'custom-value',
            'Accept-Language': 'vi-VN'
        }
    });
    
    return response.json();
}
```

## Upload File với FormData

```javascript
async function uploadFile(file) {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('description', 'Mô tả file');
    
    try {
        const response = await fetch('https://api.example.com/upload', {
            method: 'POST',
            body: formData
            // Không cần set Content-Type, browser tự xử lý
        });
        
        return await response.json();
        
    } catch (error) {
        console.error('Lỗi upload:', error);
    }
}

// Sử dụng với input file
const fileInput = document.getElementById('fileInput');
fileInput.addEventListener('change', (e) => {
    uploadFile(e.target.files[0]);
});
```

## Xử lý Timeout

Fetch không hỗ trợ timeout mặc định, ta cần dùng AbortController:

```javascript
async function fetchWithTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
        const response = await fetch(url, {
            signal: controller.signal
        });
        clearTimeout(timeoutId);
        return await response.json();
        
    } catch (error) {
        if (error.name === 'AbortError') {
            throw new Error('Request timeout sau ' + timeout + 'ms');
        }
        throw error;
    }
}

// Sử dụng
try {
    const data = await fetchWithTimeout('https://api.slow.com/data', 3000);
} catch (error) {
    console.error(error.message);
}
```

## Ví dụ thực tế: CRUD hoàn chỉnh

```javascript
// API Service cho quản lý sản phẩm
const ProductAPI = {
    baseURL: 'https://api.example.com/products',
    
    // Lấy tất cả sản phẩm
    async getAll() {
        const res = await fetch(this.baseURL);
        return res.json();
    },
    
    // Lấy một sản phẩm
    async getById(id) {
        const res = await fetch(`${this.baseURL}/${id}`);
        return res.json();
    },
    
    // Tạo sản phẩm mới
    async create(product) {
        const res = await fetch(this.baseURL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(product)
        });
        return res.json();
    },
    
    // Cập nhật sản phẩm
    async update(id, product) {
        const res = await fetch(`${this.baseURL}/${id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(product)
        });
        return res.json();
    },
    
    // Xóa sản phẩm
    async delete(id) {
        const res = await fetch(`${this.baseURL}/${id}`, {
            method: 'DELETE'
        });
        return res.ok;
    }
};

// Sử dụng
async function demo() {
    const products = await ProductAPI.getAll();
    console.log(products);
    
    const newProduct = await ProductAPI.create({ name: 'Laptop', price: 1000 });
    console.log(newProduct);
}
```

## Kết luận

**Fetch API** là công cụ mạnh mẽ và linh hoạt cho giao tiếp HTTP trong JavaScript. Kết hợp với Async/Await, nó giúp code trở nên rõ ràng và dễ bảo trì. Đây là kiến thức nền tảng quan trọng cho mọi lập trình viên web.

---

*Bài viết thuộc series JavaScript cho lập trình mạng - Blog Ngô Phạm Ngọc Tú*
