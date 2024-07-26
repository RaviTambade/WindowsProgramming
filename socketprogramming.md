# Socket programming in Win32


Socket programming in Win32 involves creating and managing sockets to facilitate communication over a network or between processes. Here, I'll provide a simple example of a TCP client-server application using Win32 sockets. The example will include a server that listens for incoming connections and a client that connects to the server and sends a message.

### **1. TCP Server Example**

The server will listen on a specified port for incoming connections. Once a connection is established, it will receive a message from the client and send a response.

**Server Code:**

```c
#include <winsock2.h>
#include <stdio.h>

#pragma comment(lib, "ws2_32.lib")  // Link with ws2_32.lib

#define SERVER_PORT 12345
#define BUFFER_SIZE 512

int main() {
    WSADATA wsaData;
    SOCKET listenSocket = INVALID_SOCKET;
    SOCKET clientSocket = INVALID_SOCKET;
    struct sockaddr_in serverAddr;
    struct sockaddr_in clientAddr;
    int clientAddrSize = sizeof(clientAddr);
    char recvBuffer[BUFFER_SIZE];
    int recvSize;
    const char* response = "Message received";

    // Initialize Winsock
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        printf("WSAStartup failed.\n");
        return 1;
    }

    // Create a socket
    listenSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    if (listenSocket == INVALID_SOCKET) {
        printf("Socket creation failed.\n");
        WSACleanup();
        return 1;
    }

    // Set up the server address structure
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    serverAddr.sin_port = htons(SERVER_PORT);

    // Bind the socket
    if (bind(listenSocket, (struct sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        printf("Bind failed.\n");
        closesocket(listenSocket);
        WSACleanup();
        return 1;
    }

    // Listen for incoming connections
    if (listen(listenSocket, 1) == SOCKET_ERROR) {
        printf("Listen failed.\n");
        closesocket(listenSocket);
        WSACleanup();
        return 1;
    }

    // Accept a client socket
    clientSocket = accept(listenSocket, (struct sockaddr*)&clientAddr, &clientAddrSize);
    if (clientSocket == INVALID_SOCKET) {
        printf("Accept failed.\n");
        closesocket(listenSocket);
        WSACleanup();
        return 1;
    }

    // Receive data from the client
    recvSize = recv(clientSocket, recvBuffer, BUFFER_SIZE, 0);
    if (recvSize > 0) {
        recvBuffer[recvSize] = '\0';  // Null-terminate the string
        printf("Received: %s\n", recvBuffer);

        // Send a response to the client
        send(clientSocket, response, (int)strlen(response), 0);
    } else {
        printf("Receive failed.\n");
    }

    // Clean up
    closesocket(clientSocket);
    closesocket(listenSocket);
    WSACleanup();

    return 0;
}
```

### **2. TCP Client Example**

The client will connect to the server, send a message, and then receive a response from the server.

**Client Code:**

```c
#include <winsock2.h>
#include <stdio.h>

#pragma comment(lib, "ws2_32.lib")  // Link with ws2_32.lib

#define SERVER_IP "127.0.0.1"  // Localhost
#define SERVER_PORT 12345
#define BUFFER_SIZE 512

int main() {
    WSADATA wsaData;
    SOCKET connectSocket = INVALID_SOCKET;
    struct sockaddr_in serverAddr;
    char sendBuffer[] = "Hello, Server!";
    char recvBuffer[BUFFER_SIZE];
    int recvSize;

    // Initialize Winsock
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        printf("WSAStartup failed.\n");
        return 1;
    }

    // Create a socket
    connectSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    if (connectSocket == INVALID_SOCKET) {
        printf("Socket creation failed.\n");
        WSACleanup();
        return 1;
    }

    // Set up the server address structure
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = inet_addr(SERVER_IP);
    serverAddr.sin_port = htons(SERVER_PORT);

    // Connect to the server
    if (connect(connectSocket, (struct sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        printf("Connect failed.\n");
        closesocket(connectSocket);
        WSACleanup();
        return 1;
    }

    // Send data to the server
    send(connectSocket, sendBuffer, (int)strlen(sendBuffer), 0);

    // Receive data from the server
    recvSize = recv(connectSocket, recvBuffer, BUFFER_SIZE, 0);
    if (recvSize > 0) {
        recvBuffer[recvSize] = '\0';  // Null-terminate the string
        printf("Received: %s\n", recvBuffer);
    } else {
        printf("Receive failed.\n");
    }

    // Clean up
    closesocket(connectSocket);
    WSACleanup();

    return 0;
}
```

### **Explanation**

1. **Server:**
   - Initializes Winsock using `WSAStartup`.
   - Creates a listening socket using `socket`.
   - Binds the socket to an address and port using `bind`.
   - Listens for incoming connections with `listen`.
   - Accepts a connection using `accept`.
   - Receives data from the client using `recv`.
   - Sends a response back to the client using `send`.
   - Cleans up resources with `closesocket` and `WSACleanup`.

2. **Client:**
   - Initializes Winsock using `WSAStartup`.
   - Creates a socket using `socket`.
   - Sets up the server address and port.
   - Connects to the server using `connect`.
   - Sends data to the server with `send`.
   - Receives a response from the server using `recv`.
   - Cleans up resources with `closesocket` and `WSACleanup`.

### **Compiling and Running**

To compile these programs, make sure you have the Windows SDK installed and use a compatible compiler. For example, using Visual Studio:

1. Create a new Win32 Console Application project.
2. Add the above server code to `server.cpp` and client code to `client.cpp`.
3. Build the projects.
4. Run the server executable (`server.exe`) and then run the client executable (`client.exe`).

The server should print the message it receives from the client, and the client should display the response it gets from the server.

This example demonstrates basic socket programming in Win32 for a TCP client-server application. For more complex scenarios, consider handling errors, managing multiple clients, and implementing more robust communication protocols.