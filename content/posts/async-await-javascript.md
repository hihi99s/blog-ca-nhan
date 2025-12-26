+++
title = "Async/Await trong JavaScript: Hướng dẫn chi tiết"
date = "2025-12-20T10:00:00+07:00"
draft = false
tags = ["JavaScript", "Async", "Web"]
categories = ["JavaScript"]
+++

# Async/Await trong JavaScript: Hướng dẫn chi tiết

Trong lập trình JavaScript hiện đại, việc xử lý các tác vụ bất đồng bộ là điều không thể tránh khỏi. **Async/Await** là cú pháp được giới thiệu trong ES2017 (ES8) giúp code bất đồng bộ trở nên dễ đọc và dễ bảo trì hơn rất nhiều.

## Tại sao cần Async/Await?

Trước khi có Async/Await, chúng ta phải sử dụng **callback** hoặc **Promise chains**, dẫn đến code khó đọc:

### Callback Hell (Địa ngục callback)

```javascript
// Callback hell - khó đọc và bảo trì
getData(function(a) {
    getMoreData(a, function(b) {
        getEvenMoreData(b, function(c) {
            getFinalData(c, function(d) {
                console.log(d);
            });
        });
    });
});
```

### Promise Chains

```javascript
// Promise chains - tốt hơn nhưng vẫn rườm rà
getData()
    .then(a => getMoreData(a))
    .then(b => getEvenMoreData(b))
    .then(c => getFinalData(c))
    .then(d => console.log(d))
    .catch(error => console.error(error));
```

### Async/Await - Giải pháp tối ưu

```javascript
// Async/Await - rõ ràng như code đồng bộ
async function fetchAllData() {
    try {
        const a = await getData();
        const b = await getMoreData(a);
        const c = await getEvenMoreData(b);
        const d = await getFinalData(c);
        console.log(d);
    } catch (error) {
        console.error(error);
    }
}
```

## Cách hoạt động của Async/Await

### Từ khóa `async`

Khi đặt `async` trước một function, function đó sẽ **luôn trả về một Promise**:

```javascript
async function sayHello() {
    return "Xin chào!";
}

// Tương đương với:
function sayHello() {
    return Promise.resolve("Xin chào!");
}

// Sử dụng
sayHello().then(message => console.log(message)); // "Xin chào!"
```

### Từ khóa `await`

`await` chỉ được sử dụng **bên trong async function** và nó sẽ:
- Tạm dừng thực thi function
- Chờ Promise hoàn thành
- Trả về giá trị resolved của Promise

```javascript
async function demo() {
    console.log("Bắt đầu");
    
    // Tạm dừng 2 giây
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    console.log("Sau 2 giây");
}

demo();
// Output:
// "Bắt đầu"
// (chờ 2 giây)
// "Sau 2 giây"
```

## Ví dụ thực tế: Gọi API

```javascript
async function layThongTinNguoiDung(userId) {
    try {
        // Gọi API lấy thông tin user
        const response = await fetch(`https://api.example.com/users/${userId}`);
        
        // Kiểm tra response
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        // Parse JSON
        const userData = await response.json();
        
        console.log("Thông tin người dùng:", userData);
        return userData;
        
    } catch (error) {
        console.error("Lỗi khi lấy thông tin:", error.message);
        throw error;
    }
}

// Sử dụng
layThongTinNguoiDung(123);
```

## Xử lý nhiều Promise song song

### Tuần tự (Sequential) - Chậm

```javascript
async function tuanTu() {
    const start = Date.now();
    
    const user = await fetch('/api/user');      // Chờ xong mới chạy tiếp
    const posts = await fetch('/api/posts');    // Chờ xong mới chạy tiếp
    const comments = await fetch('/api/comments');
    
    console.log(`Thời gian: ${Date.now() - start}ms`);
    // Giả sử mỗi request mất 1s => Tổng: ~3000ms
}
```

### Song song (Parallel) - Nhanh

```javascript
async function songSong() {
    const start = Date.now();
    
    // Khởi động tất cả requests cùng lúc
    const [user, posts, comments] = await Promise.all([
        fetch('/api/user'),
        fetch('/api/posts'),
        fetch('/api/comments')
    ]);
    
    console.log(`Thời gian: ${Date.now() - start}ms`);
    // Tổng: ~1000ms (thời gian của request lâu nhất)
}
```

## Xử lý lỗi với try/catch

```javascript
async function fetchData() {
    try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        return data;
    } catch (error) {
        // Xử lý lỗi mạng hoặc lỗi parse JSON
        if (error instanceof TypeError) {
            console.error("Lỗi mạng:", error.message);
        } else {
            console.error("Lỗi khác:", error.message);
        }
        return null;
    } finally {
        // Luôn chạy dù có lỗi hay không
        console.log("Hoàn thành request");
    }
}
```

## Async/Await với vòng lặp

### Tuần tự trong vòng lặp

```javascript
async function processItems(items) {
    for (const item of items) {
        // Mỗi item được xử lý tuần tự
        await processItem(item);
    }
}
```

### Song song trong vòng lặp

```javascript
async function processItemsParallel(items) {
    // Tất cả items được xử lý cùng lúc
    await Promise.all(items.map(item => processItem(item)));
}
```

## Những lưu ý quan trọng

1. **`await` chỉ dùng trong `async` function** - Sử dụng ngoài sẽ gây lỗi cú pháp

2. **Async function luôn trả về Promise** - Kể cả khi return giá trị thường

3. **Không quên `try/catch`** - Lỗi không được xử lý sẽ gây Unhandled Promise Rejection

4. **Cân nhắc tuần tự vs song song** - Sử dụng `Promise.all()` khi các tác vụ độc lập

## Kết luận

**Async/Await** là công cụ mạnh mẽ giúp viết code bất đồng bộ trong JavaScript một cách rõ ràng và dễ bảo trì. Hiểu và sử dụng thành thạo Async/Await là kỹ năng quan trọng cho mọi lập trình viên JavaScript hiện đại.

---

*Bài viết thuộc series JavaScript cho lập trình mạng - Blog Ngô Phạm Ngọc Tú*
