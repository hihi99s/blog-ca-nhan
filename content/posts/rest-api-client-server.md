+++
title = "REST API và Kiến trúc Client-Server"
date = "2025-12-13T10:00:00+07:00"
draft = false
tags = ["REST", "API", "HTTP", "Client-Server"]
categories = ["Lập trình mạng"]
+++

# REST API và Kiến trúc Client-Server

**REST (Representational State Transfer)** là kiến trúc phổ biến nhất để xây dựng API cho ứng dụng web. Bài viết này sẽ giải thích REST API và mô hình Client-Server một cách chi tiết.

## Kiến trúc Client-Server

### Mô hình cơ bản

```
┌──────────────┐         HTTP Request          ┌──────────────┐
│              │ ─────────────────────────────►│              │
│    CLIENT    │                               │    SERVER    │
│              │◄───────────────────────────── │              │
└──────────────┘         HTTP Response         └──────────────┘
     (Browser,                                      (API,
      Mobile App,                                    Database,
      Desktop App)                                   Business Logic)
```

### Đặc điểm
- **Client**: Gửi request, hiển thị dữ liệu cho user
- **Server**: Xử lý logic, lưu trữ dữ liệu, trả response
- **Stateless**: Server không lưu trạng thái client giữa các request
- **Separation of Concerns**: Tách biệt UI và logic

## REST là gì?

REST là một **kiến trúc** (không phải giao thức) với các nguyên tắc thiết kế API:

### 6 Ràng buộc của REST

1. **Client-Server**: Tách biệt client và server
2. **Stateless**: Mỗi request chứa đủ thông tin, server không lưu session
3. **Cacheable**: Response có thể được cache
4. **Uniform Interface**: Giao diện thống nhất
5. **Layered System**: Có thể có các layer trung gian (proxy, load balancer)
6. **Code on Demand** (optional): Server có thể gửi code để client thực thi

## HTTP Methods trong REST

| Method | Mục đích | Ví dụ | Idempotent |
|--------|----------|-------|------------|
| **GET** | Lấy dữ liệu | Lấy danh sách users | Có |
| **POST** | Tạo mới | Tạo user mới | Không |
| **PUT** | Cập nhật toàn bộ | Thay thế thông tin user | Có |
| **PATCH** | Cập nhật một phần | Đổi email user | Có |
| **DELETE** | Xóa | Xóa user | Có |

**Idempotent**: Gọi nhiều lần cho cùng kết quả.

## Thiết kế URL RESTful

### Quy tắc đặt tên

```
✅ Tốt:
GET    /users              - Lấy tất cả users
GET    /users/123          - Lấy user id 123
POST   /users              - Tạo user mới
PUT    /users/123          - Cập nhật user 123
DELETE /users/123          - Xóa user 123

GET    /users/123/posts    - Lấy posts của user 123
GET    /posts?author=123   - Lấy posts với filter

❌ Không tốt:
GET    /getUsers
GET    /user/get/123
POST   /createNewUser
GET    /getAllPostsOfUser/123
```

### Quy ước

1. **Dùng danh từ số nhiều**: `/users`, `/products`, `/orders`
2. **Dùng lowercase và hyphen**: `/user-profiles`, không dùng camelCase
3. **Không dùng động từ trong URL**: Method xác định hành động
4. **Nested resources cho quan hệ**: `/users/123/orders`
5. **Query params cho filter, sort, pagination**:
   ```
   GET /products?category=electronics&sort=price&page=2
   ```

## HTTP Status Codes

### Nhóm 2xx - Thành công

| Code | Ý nghĩa |
|------|---------|
| 200 | OK - Request thành công |
| 201 | Created - Tạo mới thành công |
| 204 | No Content - Thành công, không có body |

### Nhóm 4xx - Lỗi Client

| Code | Ý nghĩa |
|------|---------|
| 400 | Bad Request - Request không hợp lệ |
| 401 | Unauthorized - Chưa xác thực |
| 403 | Forbidden - Không có quyền |
| 404 | Not Found - Không tìm thấy |
| 422 | Unprocessable Entity - Validation lỗi |

### Nhóm 5xx - Lỗi Server

| Code | Ý nghĩa |
|------|---------|
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

## Cấu trúc Request/Response

### Request

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

{
    "name": "Nguyễn Văn A",
    "email": "nguyenvana@example.com",
    "password": "matkhau123"
}
```

### Response

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/456

{
    "id": 456,
    "name": "Nguyễn Văn A",
    "email": "nguyenvana@example.com",
    "createdAt": "2025-12-13T10:30:00Z"
}
```

## Ví dụ: REST API hoàn chỉnh

### Định nghĩa endpoints

```
Quản lý Sản phẩm (Products)
───────────────────────────
GET    /api/products           - Lấy danh sách
GET    /api/products/:id       - Lấy chi tiết
POST   /api/products           - Tạo mới
PUT    /api/products/:id       - Cập nhật
DELETE /api/products/:id       - Xóa

Lọc và phân trang
─────────────────
GET /api/products?category=electronics
GET /api/products?minPrice=100&maxPrice=500
GET /api/products?page=1&limit=20&sort=-createdAt
```

### Client JavaScript với Fetch

```javascript
const API_BASE = 'https://api.example.com';

// API Service
const ProductAPI = {
    // Lấy tất cả sản phẩm
    async getAll(params = {}) {
        const query = new URLSearchParams(params).toString();
        const url = `${API_BASE}/products${query ? '?' + query : ''}`;
        
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    },
    
    // Lấy sản phẩm theo ID
    async getById(id) {
        const response = await fetch(`${API_BASE}/products/${id}`);
        if (!response.ok) {
            if (response.status === 404) throw new Error('Không tìm thấy sản phẩm');
            throw new Error(`HTTP ${response.status}`);
        }
        return response.json();
    },
    
    // Tạo sản phẩm mới
    async create(productData) {
        const response = await fetch(`${API_BASE}/products`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${getToken()}`
            },
            body: JSON.stringify(productData)
        });
        
        if (!response.ok) {
            const error = await response.json();
            throw new Error(error.message || 'Lỗi tạo sản phẩm');
        }
        return response.json();
    },
    
    // Cập nhật sản phẩm
    async update(id, productData) {
        const response = await fetch(`${API_BASE}/products/${id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${getToken()}`
            },
            body: JSON.stringify(productData)
        });
        
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    },
    
    // Xóa sản phẩm
    async delete(id) {
        const response = await fetch(`${API_BASE}/products/${id}`, {
            method: 'DELETE',
            headers: {
                'Authorization': `Bearer ${getToken()}`
            }
        });
        
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return true;
    }
};

// Sử dụng
async function demo() {
    try {
        // Lấy danh sách
        const products = await ProductAPI.getAll({ 
            category: 'electronics',
            page: 1,
            limit: 10
        });
        console.log('Sản phẩm:', products);
        
        // Tạo mới
        const newProduct = await ProductAPI.create({
            name: 'Laptop HP',
            price: 15000000,
            category: 'electronics'
        });
        console.log('Đã tạo:', newProduct);
        
        // Cập nhật
        const updated = await ProductAPI.update(newProduct.id, {
            price: 14000000
        });
        console.log('Đã cập nhật:', updated);
        
        // Xóa
        await ProductAPI.delete(newProduct.id);
        console.log('Đã xóa');
        
    } catch (error) {
        console.error('Lỗi:', error.message);
    }
}
```

### Response Format chuẩn

```javascript
// Thành công - Single item
{
    "success": true,
    "data": {
        "id": 1,
        "name": "Laptop",
        "price": 15000000
    }
}

// Thành công - List với pagination
{
    "success": true,
    "data": [
        { "id": 1, "name": "Laptop" },
        { "id": 2, "name": "Phone" }
    ],
    "pagination": {
        "page": 1,
        "limit": 10,
        "total": 150,
        "totalPages": 15
    }
}

// Lỗi
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Email không hợp lệ",
        "details": [
            { "field": "email", "message": "Định dạng email sai" }
        ]
    }
}
```

## Authentication

### JWT (JSON Web Token)

```javascript
// Login - nhận token
async function login(email, password) {
    const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    localStorage.setItem('token', data.token);
    return data;
}

// Request với token
async function protectedRequest() {
    const token = localStorage.getItem('token');
    
    const response = await fetch('/api/protected-resource', {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    });
    
    return response.json();
}
```

## Best Practices

### 1. Versioning API

```
/api/v1/users
/api/v2/users
```

### 2. Error handling nhất quán

```javascript
{
    "error": {
        "code": "RESOURCE_NOT_FOUND",
        "message": "User không tồn tại"
    }
}
```

### 3. Rate limiting

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1609459200
```

### 4. HATEOAS (Hypermedia)

```javascript
{
    "id": 123,
    "name": "Product",
    "_links": {
        "self": "/api/products/123",
        "category": "/api/categories/5",
        "reviews": "/api/products/123/reviews"
    }
}
```

## Kết luận

**REST API** là chuẩn phổ biến nhất để xây dựng web services. Nắm vững các nguyên tắc REST, HTTP methods, status codes và thiết kế URL giúp bạn xây dựng API chuyên nghiệp, dễ sử dụng và bảo trì.

---

*Bài viết thuộc series Lập trình mạng - Blog Ngô Phạm Ngọc Tú*
