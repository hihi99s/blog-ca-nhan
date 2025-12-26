+++
title = "Bất đồng bộ và Xử lý đồng thời trong Lập trình"
date = "2025-12-14T10:00:00+07:00"
draft = false
tags = ["JavaScript", "Java", "Async", "Concurrency"]
categories = ["Lập trình mạng"]
+++

# Bất đồng bộ và Xử lý đồng thời trong Lập trình

Trong lập trình hiện đại, việc xử lý nhiều tác vụ cùng lúc là điều cần thiết. Bài viết này sẽ giải thích khái niệm **bất đồng bộ (Asynchronous)** và **xử lý đồng thời (Concurrency)** trong JavaScript và Java.

## Đồng bộ vs Bất đồng bộ

### Xử lý đồng bộ (Synchronous)

Các tác vụ thực hiện **tuần tự**, tác vụ sau phải chờ tác vụ trước hoàn thành:

```
Tác vụ 1: ████████████████ (3 giây)
Tác vụ 2:                  ████████████████ (3 giây)
Tác vụ 3:                                  ████████████████ (3 giây)
                                           ↓
Tổng thời gian: 9 giây
```

### Xử lý bất đồng bộ (Asynchronous)

Các tác vụ có thể **chạy đồng thời**, không cần chờ nhau:

```
Tác vụ 1: ████████████████ (3 giây)
Tác vụ 2: ████████████████ (3 giây)
Tác vụ 3: ████████████████ (3 giây)
                          ↓
Tổng thời gian: ~3 giây
```

## Bất đồng bộ trong JavaScript

JavaScript là **single-threaded** nhưng sử dụng **Event Loop** để xử lý bất đồng bộ.

### Event Loop

```
┌───────────────────────────┐
│      Call Stack           │ ← Thực thi code
└───────────────────────────┘
            ↕
┌───────────────────────────┐
│      Event Loop           │ ← Kiểm tra Queue
└───────────────────────────┘
            ↕
┌───────────────────────────┐
│   Callback Queue          │ ← Callback chờ thực thi
└───────────────────────────┘
            ↕
┌───────────────────────────┐
│   Web APIs                │ ← setTimeout, fetch, etc.
└───────────────────────────┘
```

### Callback

```javascript
console.log("Bắt đầu");

setTimeout(() => {
    console.log("Sau 2 giây");
}, 2000);

console.log("Tiếp tục");

// Output:
// Bắt đầu
// Tiếp tục
// Sau 2 giây (sau 2s)
```

### Promise

```javascript
function fetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const success = true;
            if (success) {
                resolve({ data: "Dữ liệu từ server" });
            } else {
                reject(new Error("Lỗi fetch"));
            }
        }, 1000);
    });
}

// Sử dụng
fetchData()
    .then(result => console.log(result.data))
    .catch(error => console.error(error));
```

### Async/Await

```javascript
async function main() {
    try {
        console.log("Bắt đầu fetch...");
        const result = await fetchData();
        console.log("Kết quả:", result.data);
    } catch (error) {
        console.error("Lỗi:", error);
    }
}

main();
```

### Promise.all - Chạy song song

```javascript
async function fetchMultiple() {
    const start = Date.now();
    
    // Chạy song song
    const [users, posts, comments] = await Promise.all([
        fetch('/api/users').then(r => r.json()),
        fetch('/api/posts').then(r => r.json()),
        fetch('/api/comments').then(r => r.json())
    ]);
    
    console.log(`Hoàn thành trong ${Date.now() - start}ms`);
    return { users, posts, comments };
}
```

### Promise.race - Lấy kết quả nhanh nhất

```javascript
async function fetchWithTimeout(url, timeout) {
    const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('Timeout!')), timeout);
    });
    
    const fetchPromise = fetch(url);
    
    return Promise.race([fetchPromise, timeoutPromise]);
}
```

## Xử lý đồng thời trong Java

Java sử dụng **Multi-threading** để xử lý đồng thời.

### Tạo Thread

```java
// Cách 1: Extend Thread
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread đang chạy: " + Thread.currentThread().getName());
    }
}

// Cách 2: Implement Runnable
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable đang chạy: " + Thread.currentThread().getName());
    }
}

// Cách 3: Lambda
Thread thread = new Thread(() -> {
    System.out.println("Lambda thread");
});

// Sử dụng
public static void main(String[] args) {
    new MyThread().start();
    new Thread(new MyRunnable()).start();
    
    Thread t = new Thread(() -> System.out.println("Hello!"));
    t.start();
}
```

### Thread Lifecycle

```
NEW ──► RUNNABLE ──► RUNNING ──► TERMINATED
              ↑         │
              │         ▼
              └── BLOCKED/WAITING
```

### Synchronization

Khi nhiều thread truy cập cùng tài nguyên:

```java
class Counter {
    private int count = 0;
    
    // KHÔNG AN TOÀN - Race condition
    public void increment() {
        count++;  // Đọc, tăng, ghi - không atomic
    }
    
    // AN TOÀN - Synchronized
    public synchronized void safeIncrement() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}

// Demo race condition
public class RaceConditionDemo {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                counter.increment();
            }
        });
        
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                counter.increment();
            }
        });
        
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        
        // Kết quả có thể < 20000 do race condition
        System.out.println("Count: " + counter.getCount());
    }
}
```

### ExecutorService - Thread Pool

```java
import java.util.concurrent.*;

public class ThreadPoolDemo {
    public static void main(String[] args) {
        // Tạo thread pool với 4 threads
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        // Submit các tasks
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " chạy bởi " + 
                    Thread.currentThread().getName());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        // Đợi hoàn thành và shutdown
        executor.shutdown();
        try {
            executor.awaitTermination(1, TimeUnit.MINUTES);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### Future - Lấy kết quả từ async task

```java
import java.util.concurrent.*;

public class FutureDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        // Submit task trả về kết quả
        Future<Integer> future = executor.submit(() -> {
            Thread.sleep(2000);
            return 42;
        });
        
        System.out.println("Đang tính toán...");
        
        // Lấy kết quả (blocking)
        Integer result = future.get();
        System.out.println("Kết quả: " + result);
        
        executor.shutdown();
    }
}
```

### CompletableFuture - Async nâng cao

```java
import java.util.concurrent.*;

public class CompletableFutureDemo {
    public static void main(String[] args) {
        // Async task
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {}
            return "Kết quả";
        });
        
        // Chain operations
        future.thenApply(s -> s.toUpperCase())
              .thenAccept(s -> System.out.println("Xử lý: " + s));
        
        // Chạy nhiều task song song
        CompletableFuture<String> task1 = CompletableFuture.supplyAsync(() -> "Task 1");
        CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> "Task 2");
        CompletableFuture<String> task3 = CompletableFuture.supplyAsync(() -> "Task 3");
        
        CompletableFuture.allOf(task1, task2, task3).join();
        
        System.out.println("Tất cả tasks hoàn thành!");
    }
}
```

## So sánh JavaScript vs Java

| Khía cạnh | JavaScript | Java |
|-----------|-----------|------|
| **Model** | Single-threaded + Event Loop | Multi-threaded |
| **Async** | Promise, async/await | Future, CompletableFuture |
| **Parallel** | Worker threads (hạn chế) | Thread, ThreadPool |
| **Sync** | Không cần | synchronized, Lock |
| **Use case** | I/O bound, web | CPU bound, server |

## Best Practices

### JavaScript
1. Sử dụng `async/await` thay vì callback hell
2. Dùng `Promise.all()` cho các tác vụ độc lập
3. Xử lý error với try/catch
4. Tránh blocking operations

### Java
1. Sử dụng ThreadPool thay vì tạo thread mới
2. Luôn synchronize khi truy cập shared resources
3. Dùng `CompletableFuture` cho async phức tạp
4. Cẩn thận với deadlock

## Kết luận

**Bất đồng bộ** và **xử lý đồng thời** là kỹ năng quan trọng trong lập trình hiện đại. JavaScript sử dụng Event Loop và Promises, trong khi Java dùng Multi-threading. Hiểu rõ cách hoạt động của từng mô hình giúp bạn viết code hiệu quả và tránh các lỗi phổ biến như race condition, deadlock.

---

*Bài viết thuộc series Lập trình mạng nâng cao - Blog Ngô Phạm Ngọc Tú*
