+++
title = "DOM và Event trong Lập trình Web"
date = "2025-12-18T10:00:00+07:00"
draft = false
tags = ["JavaScript", "DOM", "Web", "Event"]
categories = ["JavaScript"]
+++

# DOM và Event trong Lập trình Web

**DOM (Document Object Model)** là giao diện lập trình cho phép JavaScript tương tác với cấu trúc HTML của trang web. Hiểu về DOM và Event là nền tảng quan trọng cho mọi lập trình viên web.

## DOM là gì?

DOM biểu diễn trang web dưới dạng **cây các node (nodes tree)**. Mỗi phần tử HTML là một node trong cây:

```
document
└── html
    ├── head
    │   └── title
    └── body
        ├── h1
        └── div
            ├── p
            └── button
```

## Truy cập phần tử DOM

### Các phương thức chọn phần tử

```javascript
// Chọn theo ID - trả về 1 element
const header = document.getElementById('header');

// Chọn theo class - trả về HTMLCollection
const items = document.getElementsByClassName('item');

// Chọn theo tag name
const paragraphs = document.getElementsByTagName('p');

// Chọn theo CSS selector - trả về element đầu tiên
const firstButton = document.querySelector('.btn-primary');

// Chọn tất cả theo CSS selector - trả về NodeList
const allButtons = document.querySelectorAll('.btn');
```

### Ví dụ thực tế

```html
<div id="app">
    <h1 class="title">Tiêu đề</h1>
    <ul id="list">
        <li class="item">Item 1</li>
        <li class="item">Item 2</li>
        <li class="item">Item 3</li>
    </ul>
    <button id="btn-add">Thêm</button>
</div>
```

```javascript
// Chọn các phần tử
const app = document.getElementById('app');
const title = document.querySelector('.title');
const items = document.querySelectorAll('.item');
const btnAdd = document.getElementById('btn-add');

console.log(title.textContent);     // "Tiêu đề"
console.log(items.length);          // 3
```

## Thao tác với nội dung

### textContent vs innerHTML

```javascript
const div = document.querySelector('#myDiv');

// textContent - chỉ text thuần
div.textContent = 'Xin chào!';
div.textContent = '<b>Bold</b>';  // Hiển thị: <b>Bold</b> (không render HTML)

// innerHTML - render HTML
div.innerHTML = '<b>Bold</b>';     // Hiển thị: Bold (in đậm)
div.innerHTML = '<span class="highlight">Highlighted</span>';
```

### Thay đổi thuộc tính

```javascript
const img = document.querySelector('img');
const link = document.querySelector('a');

// Đọc thuộc tính
console.log(img.src);
console.log(link.href);

// Thay đổi thuộc tính
img.src = 'new-image.jpg';
img.alt = 'Hình ảnh mới';
link.href = 'https://google.com';

// getAttribute / setAttribute
link.getAttribute('href');
link.setAttribute('target', '_blank');

// Xóa thuộc tính
link.removeAttribute('target');
```

### Thay đổi CSS

```javascript
const box = document.querySelector('.box');

// Thay đổi style trực tiếp
box.style.backgroundColor = 'blue';
box.style.width = '200px';
box.style.border = '2px solid red';

// Thêm/xóa class
box.classList.add('active');
box.classList.remove('hidden');
box.classList.toggle('dark-mode');
box.classList.contains('active');  // true/false

// Thay đổi nhiều class
box.className = 'box large primary';
```

## Tạo và xóa phần tử

### Tạo phần tử mới

```javascript
// Tạo element
const newDiv = document.createElement('div');
newDiv.className = 'card';
newDiv.textContent = 'Nội dung card';

// Tạo với innerHTML
newDiv.innerHTML = `
    <h3>Tiêu đề</h3>
    <p>Mô tả ngắn</p>
`;

// Thêm vào DOM
const container = document.getElementById('container');
container.appendChild(newDiv);          // Thêm cuối
container.prepend(newDiv);              // Thêm đầu
container.insertBefore(newDiv, ref);    // Thêm trước ref
```

### Xóa phần tử

```javascript
const element = document.querySelector('.delete-me');

// Cách 1: remove()
element.remove();

// Cách 2: removeChild() - qua parent
element.parentNode.removeChild(element);

// Xóa tất cả children
container.innerHTML = '';
```

### Ví dụ: Todo List

```javascript
function themCongViec(text) {
    const list = document.getElementById('todo-list');
    
    const li = document.createElement('li');
    li.className = 'todo-item';
    li.innerHTML = `
        <span>${text}</span>
        <button class="btn-delete">Xóa</button>
    `;
    
    // Thêm event xóa
    li.querySelector('.btn-delete').addEventListener('click', () => {
        li.remove();
    });
    
    list.appendChild(li);
}

// Sử dụng
themCongViec('Học JavaScript');
themCongViec('Làm bài tập DOM');
```

## Events - Sự kiện

Event là các hành động xảy ra trên trang web (click, gõ phím, scroll, ...).

### Cách gắn Event

```javascript
const button = document.getElementById('myButton');

// Cách 1: addEventListener (khuyến nghị)
button.addEventListener('click', function(event) {
    console.log('Button clicked!');
});

// Cách 2: Arrow function
button.addEventListener('click', (e) => {
    console.log('Clicked!', e.target);
});

// Cách 3: onclick property
button.onclick = function() {
    console.log('Clicked!');
};
```

### Các loại Event phổ biến

```javascript
// Mouse events
element.addEventListener('click', handler);      // Click chuột
element.addEventListener('dblclick', handler);   // Double click
element.addEventListener('mouseenter', handler); // Chuột vào
element.addEventListener('mouseleave', handler); // Chuột ra
element.addEventListener('mousemove', handler);  // Chuột di chuyển

// Keyboard events
input.addEventListener('keydown', handler);      // Nhấn phím
input.addEventListener('keyup', handler);        // Thả phím
input.addEventListener('keypress', handler);     // Gõ ký tự

// Form events
form.addEventListener('submit', handler);        // Submit form
input.addEventListener('focus', handler);        // Focus vào input
input.addEventListener('blur', handler);         // Mất focus
input.addEventListener('change', handler);       // Giá trị thay đổi
input.addEventListener('input', handler);        // Đang nhập

// Window events
window.addEventListener('load', handler);        // Trang load xong
window.addEventListener('scroll', handler);      // Scroll trang
window.addEventListener('resize', handler);      // Resize cửa sổ
```

### Event Object

```javascript
document.addEventListener('click', function(event) {
    console.log(event.type);          // "click"
    console.log(event.target);        // Element được click
    console.log(event.currentTarget); // Element gắn listener
    console.log(event.clientX);       // Tọa độ X của chuột
    console.log(event.clientY);       // Tọa độ Y của chuột
});

document.addEventListener('keydown', function(event) {
    console.log(event.key);           // "Enter", "a", "Escape"
    console.log(event.code);          // "KeyA", "Enter"
    console.log(event.ctrlKey);       // true nếu giữ Ctrl
    console.log(event.shiftKey);      // true nếu giữ Shift
});
```

### Event Propagation (Lan truyền sự kiện)

```html
<div id="outer">
    <div id="inner">
        <button id="btn">Click me</button>
    </div>
</div>
```

```javascript
// Event bubbling (mặc định) - từ trong ra ngoài
document.getElementById('btn').addEventListener('click', () => {
    console.log('Button clicked');
});
document.getElementById('inner').addEventListener('click', () => {
    console.log('Inner div clicked');
});
document.getElementById('outer').addEventListener('click', () => {
    console.log('Outer div clicked');
});
// Click button => Log: Button -> Inner -> Outer

// Ngăn bubbling
btn.addEventListener('click', (e) => {
    e.stopPropagation();
    console.log('Only button');
});
```

### Event Delegation

Thay vì gắn event cho từng element, gắn cho parent:

```javascript
// KHÔNG TỐT - gắn event cho mỗi item
const items = document.querySelectorAll('.item');
items.forEach(item => {
    item.addEventListener('click', handleClick);
});

// TỐT - Event Delegation
const list = document.getElementById('list');
list.addEventListener('click', (e) => {
    // Kiểm tra target có phải là item không
    if (e.target.classList.contains('item')) {
        console.log('Item clicked:', e.target.textContent);
    }
});
```

## Ví dụ tổng hợp: Form Validation

```html
<form id="registerForm">
    <input type="text" id="username" placeholder="Tên đăng nhập">
    <span class="error" id="usernameError"></span>
    
    <input type="email" id="email" placeholder="Email">
    <span class="error" id="emailError"></span>
    
    <input type="password" id="password" placeholder="Mật khẩu">
    <span class="error" id="passwordError"></span>
    
    <button type="submit">Đăng ký</button>
</form>
```

```javascript
const form = document.getElementById('registerForm');
const username = document.getElementById('username');
const email = document.getElementById('email');
const password = document.getElementById('password');

// Validate khi nhập
username.addEventListener('input', () => {
    const error = document.getElementById('usernameError');
    if (username.value.length < 3) {
        error.textContent = 'Tên phải có ít nhất 3 ký tự';
        username.classList.add('invalid');
    } else {
        error.textContent = '';
        username.classList.remove('invalid');
    }
});

// Validate khi submit
form.addEventListener('submit', (e) => {
    e.preventDefault(); // Ngăn form submit mặc định
    
    let isValid = true;
    
    if (username.value.length < 3) isValid = false;
    if (!email.value.includes('@')) isValid = false;
    if (password.value.length < 6) isValid = false;
    
    if (isValid) {
        console.log('Form hợp lệ, gửi dữ liệu...');
        // Gửi form bằng fetch()
    } else {
        console.log('Form không hợp lệ');
    }
});
```

## Kết luận

**DOM** và **Event** là hai khái niệm nền tảng trong lập trình web JavaScript. Nắm vững cách thao tác DOM và xử lý Event giúp bạn xây dựng các ứng dụng web tương tác và động.

---

*Bài viết thuộc series JavaScript cho lập trình mạng - Blog Ngô Phạm Ngọc Tú*
