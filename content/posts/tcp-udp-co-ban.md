+++
title = "TCP và UDP: Hai giao thức vận chuyển cơ bản"
date = "2025-12-17T10:00:00+07:00"
draft = false
tags = ["Network", "TCP", "UDP", "Protocol"]
categories = ["Lập trình mạng"]
+++

# TCP và UDP: Hai giao thức vận chuyển cơ bản

Trong lập trình mạng, **TCP** và **UDP** là hai giao thức vận chuyển (Transport Layer) quan trọng nhất. Hiểu rõ đặc điểm và use case của từng giao thức giúp bạn thiết kế ứng dụng mạng hiệu quả.

## Mô hình OSI và TCP/IP

Trước khi đi vào chi tiết, hãy nhìn lại vị trí của TCP/UDP trong mô hình mạng:

```
┌─────────────────────┐
│  Application Layer  │ ← HTTP, FTP, SMTP, DNS
├─────────────────────┤
│  Transport Layer    │ ← TCP, UDP ⭐
├─────────────────────┤
│  Network Layer      │ ← IP, ICMP
├─────────────────────┤
│  Data Link Layer    │ ← Ethernet, WiFi
├─────────────────────┤
│  Physical Layer     │ ← Cáp, sóng radio
└─────────────────────┘
```

## TCP (Transmission Control Protocol)

### Đặc điểm chính

1. **Hướng kết nối (Connection-oriented)**: Phải thiết lập kết nối trước khi truyền dữ liệu
2. **Đáng tin cậy (Reliable)**: Đảm bảo dữ liệu đến đúng thứ tự, không mất mát
3. **Kiểm soát luồng (Flow Control)**: Điều chỉnh tốc độ truyền phù hợp
4. **Kiểm soát tắc nghẽn (Congestion Control)**: Tránh quá tải mạng

### Quy trình bắt tay 3 bước (Three-way Handshake)

```
Client                            Server
   │                                 │
   │──────── SYN ──────────────────►│
   │                                 │
   │◄───── SYN + ACK ───────────────│
   │                                 │
   │──────── ACK ──────────────────►│
   │                                 │
   │        Kết nối được thiết lập   │
   │◄════════════════════════════════│
```

**Giải thích:**
1. **SYN**: Client gửi yêu cầu kết nối
2. **SYN + ACK**: Server chấp nhận và phản hồi
3. **ACK**: Client xác nhận, kết nối được thiết lập

### Quy trình đóng kết nối (Four-way Handshake)

```
Client                            Server
   │                                 │
   │──────── FIN ──────────────────►│
   │                                 │
   │◄──────── ACK ──────────────────│
   │                                 │
   │◄──────── FIN ──────────────────│
   │                                 │
   │──────── ACK ──────────────────►│
   │                                 │
   │           Kết nối đóng          │
```

### Cấu trúc TCP Segment

```
┌────────────────────────────────────────────────┐
│          Source Port (16 bit)                  │
├────────────────────────────────────────────────┤
│          Destination Port (16 bit)             │
├────────────────────────────────────────────────┤
│          Sequence Number (32 bit)              │
├────────────────────────────────────────────────┤
│          Acknowledgment Number (32 bit)        │
├────────────────────────────────────────────────┤
│ Flags: SYN, ACK, FIN, RST, PSH, URG           │
├────────────────────────────────────────────────┤
│          Window Size (16 bit)                  │
├────────────────────────────────────────────────┤
│          Checksum + Urgent Pointer             │
├────────────────────────────────────────────────┤
│          Data (payload)                        │
└────────────────────────────────────────────────┘
```

### Ví dụ TCP trong Java

```java
// TCP Server
import java.net.*;
import java.io.*;

public class TCPServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("Server đang chờ kết nối...");
        
        Socket clientSocket = serverSocket.accept();
        System.out.println("Client đã kết nối!");
        
        BufferedReader in = new BufferedReader(
            new InputStreamReader(clientSocket.getInputStream()));
        PrintWriter out = new PrintWriter(
            clientSocket.getOutputStream(), true);
        
        String message = in.readLine();
        System.out.println("Nhận được: " + message);
        
        out.println("Xin chào từ server!");
        
        clientSocket.close();
        serverSocket.close();
    }
}
```

```java
// TCP Client
import java.net.*;
import java.io.*;

public class TCPClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", 8080);
        
        PrintWriter out = new PrintWriter(
            socket.getOutputStream(), true);
        BufferedReader in = new BufferedReader(
            new InputStreamReader(socket.getInputStream()));
        
        out.println("Xin chào từ client!");
        
        String response = in.readLine();
        System.out.println("Server trả lời: " + response);
        
        socket.close();
    }
}
```

## UDP (User Datagram Protocol)

### Đặc điểm chính

1. **Không hướng kết nối (Connectionless)**: Không cần thiết lập kết nối
2. **Không đáng tin cậy (Unreliable)**: Không đảm bảo dữ liệu đến đích
3. **Nhanh và nhẹ**: Ít overhead hơn TCP
4. **Hỗ trợ Broadcast/Multicast**: Gửi đến nhiều đích cùng lúc

### Cấu trúc UDP Datagram

```
┌────────────────────────────────────────────────┐
│          Source Port (16 bit)                  │
├────────────────────────────────────────────────┤
│          Destination Port (16 bit)             │
├────────────────────────────────────────────────┤
│          Length (16 bit)                       │
├────────────────────────────────────────────────┤
│          Checksum (16 bit)                     │
├────────────────────────────────────────────────┤
│          Data (payload)                        │
└────────────────────────────────────────────────┘
```

**Lưu ý:** Header UDP chỉ có 8 bytes, nhỏ hơn nhiều so với TCP (20+ bytes).

### Ví dụ UDP trong Java

```java
// UDP Server
import java.net.*;

public class UDPServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket(9090);
        byte[] buffer = new byte[1024];
        
        System.out.println("UDP Server đang chờ...");
        
        // Nhận datagram
        DatagramPacket request = new DatagramPacket(buffer, buffer.length);
        socket.receive(request);
        
        String message = new String(request.getData(), 0, request.getLength());
        System.out.println("Nhận được: " + message);
        
        // Gửi phản hồi
        String response = "Đã nhận: " + message;
        byte[] responseData = response.getBytes();
        DatagramPacket responsePacket = new DatagramPacket(
            responseData, responseData.length,
            request.getAddress(), request.getPort());
        socket.send(responsePacket);
        
        socket.close();
    }
}
```

```java
// UDP Client
import java.net.*;

public class UDPClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket();
        InetAddress serverAddress = InetAddress.getByName("localhost");
        
        // Gửi datagram
        String message = "Xin chào UDP!";
        byte[] sendData = message.getBytes();
        DatagramPacket sendPacket = new DatagramPacket(
            sendData, sendData.length, serverAddress, 9090);
        socket.send(sendPacket);
        
        // Nhận phản hồi
        byte[] buffer = new byte[1024];
        DatagramPacket receivePacket = new DatagramPacket(buffer, buffer.length);
        socket.receive(receivePacket);
        
        String response = new String(receivePacket.getData(), 0, receivePacket.getLength());
        System.out.println("Server trả lời: " + response);
        
        socket.close();
    }
}
```

## So sánh TCP vs UDP

| Tiêu chí | TCP | UDP |
|----------|-----|-----|
| **Kết nối** | Hướng kết nối | Không kết nối |
| **Độ tin cậy** | Đảm bảo giao hàng | Không đảm bảo |
| **Thứ tự** | Đúng thứ tự | Không đảm bảo |
| **Tốc độ** | Chậm hơn | Nhanh hơn |
| **Header** | 20-60 bytes | 8 bytes |
| **Flow Control** | Có | Không |
| **Use case** | Web, Email, File transfer | Streaming, Gaming, DNS |

## Khi nào dùng TCP? Khi nào dùng UDP?

### Sử dụng TCP khi:
- Cần đảm bảo dữ liệu đến đầy đủ, đúng thứ tự
- Ứng dụng web (HTTP/HTTPS)
- Transfer file (FTP)
- Email (SMTP, IMAP)
- Database connections

### Sử dụng UDP khi:
- Cần tốc độ cao, chấp nhận mất mát nhỏ
- Live streaming video/audio
- Online gaming
- VoIP (Voice over IP)
- DNS queries
- IoT sensors

## Ví dụ thực tế

### Trường hợp 1: Chat Application
```
→ Dùng TCP
Lý do: Tin nhắn phải đến đầy đủ và đúng thứ tự
```

### Trường hợp 2: Live Video Streaming
```
→ Dùng UDP
Lý do: Chấp nhận mất vài frame, ưu tiên realtime
```

### Trường hợp 3: File Download
```
→ Dùng TCP
Lý do: File phải hoàn chỉnh, không mất bytes
```

### Trường hợp 4: Online Game
```
→ Dùng UDP (với custom reliability layer nếu cần)
Lý do: Cần độ trễ thấp, vị trí cũ không quan trọng
```

## Kết luận

**TCP** và **UDP** là hai giao thức nền tảng trong lập trình mạng. TCP đảm bảo độ tin cậy cao, phù hợp với ứng dụng cần dữ liệu chính xác. UDP ưu tiên tốc độ, phù hợp với ứng dụng realtime. Việc chọn đúng giao thức ảnh hưởng trực tiếp đến hiệu năng và trải nghiệm người dùng.

---

*Bài viết thuộc series Lập trình mạng cơ bản - Blog Ngô Phạm Ngọc Tú*
