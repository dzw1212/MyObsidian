*https://github.com/skywind3000/kcp*

*KCP是一个快速可靠的ARQ（自动重传请求）协议，由国内开发者skywind3000创建。它是在UDP基础上实现的传输层协议，旨在提供比TCP更低的延迟和更高的传输效率。*

# 示例UDP Client与Server

以下是一个示例，在本地实现Server与Client之间的UDP通信：
1. Server端创建一个Socket，指定端口并绑定；
2. Client需要知道Server的IP地址与端口，在第一次sendto Socket时会随机分配空闲端口；

UDP Server：

```cpp
#include <iostream>
#include <cstring>
#include <winsock2.h>
#include <WS2tcpip.h>

#pragma comment(lib, "ws2_32.lib")

// winsock版本
// 1.0	最早的Winsock，功能有限	    兼容老系统
// 1.1	增加了些扩展功能	        Windows 95/98
// 2.0	重大更新，支持更多协议	    Windows 98 / 2000
// 2.2	当前主流，最稳定	        Windows XP及以后

#define PORT 12345
#define BUFFER_SIZE 1024
#define MAX_IP_STR_LEN INET6_ADDRSTRLEN

int main() {
    WSADATA wsaData;
    SOCKET serverSocket;
    struct sockaddr_in serverAddr, clientAddr;
    int clientAddrLen = sizeof(clientAddr);
    char buffer[BUFFER_SIZE];
    char clientIP[MAX_IP_STR_LEN];

    // 初始化Winsock
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed: " << WSAGetLastError() << std::endl;
        return 1;
    }

    // 创建UDP socket
    serverSocket = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (serverSocket == INVALID_SOCKET) {
        std::cerr << "Socket creation failed: " << WSAGetLastError() << std::endl;
        WSACleanup();
        return 1;
    }

    // 设置服务器地址
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    serverAddr.sin_port = htons(PORT);

    // 绑定socket
    if (bind(serverSocket, (struct sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        std::cerr << "Bind failed: " << WSAGetLastError() << std::endl;
        closesocket(serverSocket);
        WSACleanup();
        return 1;
    }

    std::cout << "UDP服务器正在监听端口 " << PORT << std::endl;

    while (true) {
        std::cout << "\n等待接收数据..." << std::endl;

        // 接收数据
        memset(buffer, 0, BUFFER_SIZE);
        int recvLen = recvfrom(serverSocket, buffer, BUFFER_SIZE, 0,
            (struct sockaddr*)&clientAddr, &clientAddrLen);

        if (recvLen == SOCKET_ERROR) {
            std::cerr << "recvfrom failed: " << WSAGetLastError() << std::endl;
            continue;
        }

        //std::cout << "收到来自 " << inet_ntoa(clientAddr.sin_addr) << ":"
        //    << ntohs(clientAddr.sin_port) << " 的消息: " << buffer << std::endl;

        // 使用现代方法转换IP地址 - 替代inet_ntoa
        const char* ipStr = inet_ntop(AF_INET, &clientAddr.sin_addr,
            clientIP, MAX_IP_STR_LEN);

        if (ipStr == nullptr) {
            std::cerr << "IP地址转换失败: " << WSAGetLastError() << std::endl;
            strcpy_s(clientIP, MAX_IP_STR_LEN, "未知IP");
        }

        std::cout << "收到来自 " << clientIP << ":"
            << ntohs(clientAddr.sin_port) << " 的消息: " << buffer << std::endl;


        // 准备响应
        std::string response = "服务器已收到你的消息: ";
        response += buffer;

        // 发送响应
        if (sendto(serverSocket, response.c_str(), response.length(), 0,
            (struct sockaddr*)&clientAddr, clientAddrLen) == SOCKET_ERROR) {
            std::cerr << "sendto failed: " << WSAGetLastError() << std::endl;
        }
        else {
            std::cout << "已发送响应给客户端" << std::endl;
        }
    }

    // 清理
    closesocket(serverSocket);
    WSACleanup();
    return 0;
}
```


UDP Client：

```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <winsock2.h>
#include <WS2tcpip.h>

#pragma comment(lib, "ws2_32.lib")

#define SERVER_IP "127.0.0.1"
#define PORT 12345
#define BUFFER_SIZE 1024

int main() {
    WSADATA wsaData;
    SOCKET clientSocket;
    struct sockaddr_in serverAddr;
    char buffer[BUFFER_SIZE];

    // 初始化Winsock
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed: " << WSAGetLastError() << std::endl;
        return 1;
    }

    // 创建UDP socket
    clientSocket = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (clientSocket == INVALID_SOCKET) {
        std::cerr << "Socket creation failed: " << WSAGetLastError() << std::endl;
        WSACleanup();
        return 1;
    }

    // 设置服务器地址
    serverAddr.sin_family = AF_INET;
    //serverAddr.sin_addr.s_addr = inet_addr(SERVER_IP);
    serverAddr.sin_port = htons(PORT);

    // 使用inet_pton而不是inet_addr
    if (inet_pton(AF_INET, SERVER_IP, &serverAddr.sin_addr) <= 0) {
        std::cerr << "Invalid address: " << SERVER_IP << std::endl;
        closesocket(clientSocket);
        WSACleanup();
        return 1;
    }

    std::cout << "UDP客户端已启动，连接到服务器 " << SERVER_IP << ":" << PORT << std::endl;

    while (true) {
        std::string message;
        std::cout << "请输入要发送的消息 (输入 'quit' 退出): ";
        std::getline(std::cin, message);

        if (message == "quit") {
            break;
        }

        // 发送数据到服务器
        std::cout << "发送消息: " << message << std::endl;
        if (sendto(clientSocket, message.c_str(), message.length(), 0,
            (struct sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
            std::cerr << "sendto failed: " << WSAGetLastError() << std::endl;
            continue;
        }

        // 接收服务器响应
        memset(buffer, 0, BUFFER_SIZE);
        int serverAddrLen = sizeof(serverAddr);
        int recvLen = recvfrom(clientSocket, buffer, BUFFER_SIZE, 0,
            (struct sockaddr*)&serverAddr, &serverAddrLen);

        if (recvLen == SOCKET_ERROR) {
            std::cerr << "recvfrom failed: " << WSAGetLastError() << std::endl;
        }
        else {
            std::cout << "服务器响应: " << buffer << std::endl;
        }
    }

    // 清理
    closesocket(clientSocket);
    WSACleanup();
    return 0;
}
```

# 集成KCP

