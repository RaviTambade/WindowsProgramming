# Interprocess Communication (IPC)

Interprocess Communication (IPC) in Win32 refers to mechanisms that allow different processes to communicate and synchronize their actions. IPC is crucial for applications that need to share data or coordinate actions across process boundaries. Win32 provides several IPC mechanisms, each suited to different scenarios.

### Common IPC Mechanisms in Win32

1. **Named Pipes**
2. **Shared Memory**
3. **Message Queues**
4. **Mailslots**
5. **Sockets**

Here’s a detailed look at each mechanism:

### 1. **Named Pipes**

**Overview:**
- Named pipes are used for communication between processes on the same machine or across networked machines.
- They provide a way for processes to send and receive data in a stream-like manner.

**Functions:**
- **`CreateNamedPipe`**: Creates a named pipe server.
  ```c
  HANDLE CreateNamedPipe(
    LPCSTR                lpName,
    DWORD                 dwOpenMode,
    DWORD                 dwPipeMode,
    DWORD                 nMaxInstances,
    DWORD                 nOutBufferSize,
    DWORD                 nInBufferSize,
    DWORD                 dwDefaultTimeOut,
    LPSECURITY_ATTRIBUTES lpSecurityAttributes
  );
  ```
- **`ConnectNamedPipe`**: Establishes a connection to a named pipe server.
  ```c
  BOOL ConnectNamedPipe(
    HANDLE       hNamedPipe,
    LPOVERLAPPED lpOverlapped
  );
  ```
- **`ReadFile` / `WriteFile`**: Reads from and writes to the pipe.
  ```c
  BOOL ReadFile(
    HANDLE       hFile,
    LPVOID       lpBuffer,
    DWORD        nNumberOfBytesToRead,
    LPDWORD      lpNumberOfBytesRead,
    LPOVERLAPPED lpOverlapped
  );

  BOOL WriteFile(
    HANDLE       hFile,
    LPCVOID      lpBuffer,
    DWORD        nNumberOfBytesToWrite,
    LPDWORD      lpNumberOfBytesWritten,
    LPOVERLAPPED lpOverlapped
  );
  ```

**Example Usage:**
Creating a simple named pipe server and client.

**Server:**
```c
HANDLE hPipe = CreateNamedPipe(
    "\\\\.\\pipe\\MyPipe",
    PIPE_ACCESS_DUPLEX,
    PIPE_TYPE_MESSAGE | PIPE_READMODE_MESSAGE | PIPE_WAIT,
    1,
    1024 * 16,
    1024 * 16,
    0,
    NULL
);

if (hPipe == INVALID_HANDLE_VALUE) {
    // Handle error
}

BOOL connected = ConnectNamedPipe(hPipe, NULL);
if (connected) {
    // Use ReadFile and WriteFile to communicate
}
```

**Client:**
```c
HANDLE hPipe = CreateFile(
    "\\\\.\\pipe\\MyPipe",
    GENERIC_READ | GENERIC_WRITE,
    0,
    NULL,
    OPEN_EXISTING,
    0,
    NULL
);

if (hPipe == INVALID_HANDLE_VALUE) {
    // Handle error
}

// Use ReadFile and WriteFile to communicate
```

### 2. **Shared Memory**

**Overview:**
- Shared memory allows multiple processes to access the same region of memory.
- It is efficient for high-speed data transfer and requires proper synchronization.

**Functions:**
- **`CreateFileMapping`**: Creates or opens a file mapping object.
  ```c
  HANDLE CreateFileMapping(
    HANDLE                hFile,
    LPSECURITY_ATTRIBUTES lpFileMappingAttributes,
    DWORD                 flProtect,
    DWORD                 dwMaximumSizeHigh,
    DWORD                 dwMaximumSizeLow,
    LPCSTR                lpName
  );
  ```
- **`MapViewOfFile`**: Maps a view of a file mapping into the address space of a process.
  ```c
  LPVOID MapViewOfFile(
    HANDLE hFileMappingObject,
    DWORD  dwDesiredAccess,
    DWORD  dwFileOffsetHigh,
    DWORD  dwFileOffsetLow,
    SIZE_T dwNumberOfBytesToMap
  );
  ```
- **`UnmapViewOfFile`**: Unmaps a mapped view of a file.
  ```c
  BOOL UnmapViewOfFile(
    LPCVOID lpBaseAddress
  );
  ```

**Example Usage:**
Creating a shared memory segment.

**Server:**
```c
HANDLE hMapFile = CreateFileMapping(
    INVALID_HANDLE_VALUE,
    NULL,
    PAGE_READWRITE,
    0,
    1024,
    "SharedMemoryName"
);

LPVOID pBuf = MapViewOfFile(
    hMapFile,
    FILE_MAP_ALL_ACCESS,
    0,
    0,
    0
);

// Write data to shared memory
sprintf((char*)pBuf, "Hello, Shared Memory!");
```

**Client:**
```c
HANDLE hMapFile = OpenFileMapping(
    FILE_MAP_ALL_ACCESS,
    FALSE,
    "SharedMemoryName"
);

LPVOID pBuf = MapViewOfFile(
    hMapFile,
    FILE_MAP_ALL_ACCESS,
    0,
    0,
    0
);

// Read data from shared memory
printf("Data: %s\n", (char*)pBuf);
```

### 3. **Message Queues**

**Overview:**
- Windows message queues are typically used for interprocess communication within the same machine, especially for GUI applications.
- Processes can post and retrieve messages from each other's queues.

**Functions:**
- **`PostMessage`**: Posts a message to a message queue.
  ```c
  BOOL PostMessage(
    HWND   hWnd,
    UINT   Msg,
    WPARAM wParam,
    LPARAM lParam
  );
  ```
- **`SendMessage`**: Sends a message to a window procedure.
  ```c
  LRESULT SendMessage(
    HWND   hWnd,
    UINT   Msg,
    WPARAM wParam,
    LPARAM lParam
  );
  ```

**Example Usage:**
Sending a custom message between processes.

**Server:**
```c
// Define a custom message
#define WM_USER_UPDATE  (WM_USER + 1)

// Window procedure to handle messages
LRESULT CALLBACK WndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    switch (msg) {
        case WM_USER_UPDATE:
            // Handle custom message
            break;
    }
    return DefWindowProc(hwnd, msg, wParam, lParam);
}
```

**Client:**
```c
HWND hWndTarget = FindWindow(NULL, "ServerWindowName");
PostMessage(hWndTarget, WM_USER_UPDATE, 0, 0);
```

### 4. **Mailslots**

**Overview:**
- Mailslots are used for one-way communication between processes on the same machine or across a network.
- They are useful for broadcasting messages to multiple recipients.

**Functions:**
- **`CreateMailslot`**: Creates a mailslot for receiving messages.
  ```c
  HANDLE CreateMailslot(
    LPCSTR                lpName,
    DWORD                 nMaxMessageSize,
    DWORD                 nReadTimeout,
    LPSECURITY_ATTRIBUTES lpSecurityAttributes
  );
  ```
- **`WriteFile`**: Sends messages to a mailslot.
  ```c
  BOOL WriteFile(
    HANDLE       hFile,
    LPCVOID      lpBuffer,
    DWORD        nNumberOfBytesToWrite,
    LPDWORD      lpNumberOfBytesWritten,
    LPOVERLAPPED lpOverlapped
  );
  ```

**Example Usage:**
Creating a mailslot server and client.

**Server:**
```c
HANDLE hMailslot = CreateMailslot(
    "\\\\.\\mailslot\\MyMailslot",
    0,
    MAILSLOT_WAIT_FOREVER,
    NULL
);

char buffer[256];
DWORD bytesRead;
BOOL success = ReadFile(
    hMailslot,
    buffer,
    sizeof(buffer),
    &bytesRead,
    NULL
);

printf("Received: %s\n", buffer);
```

**Client:**
```c
HANDLE hMailslot = CreateFile(
    "\\\\.\\mailslot\\MyMailslot",
    GENERIC_WRITE,
    FILE_SHARE_READ,
    NULL,
    OPEN_EXISTING,
    0,
    NULL
);

const char* message = "Hello, Mailslot!";
DWORD bytesWritten;
WriteFile(
    hMailslot,
    message,
    strlen(message) + 1,
    &bytesWritten,
    NULL
);
```

### 5. **Sockets**

**Overview:**
- Sockets are used for communication over a network or between processes on the same machine.
- They provide a robust way to handle network communication.

**Functions:**
- **`socket`**: Creates a socket.
  ```c
  SOCKET socket(
    int af,
    int type,
    int protocol
  );
  ```
- **`bind`**: Binds a socket to an address and port.
  ```c
  int bind(
    SOCKET s,
    const struct sockaddr* name,
    int namelen
  );
  ```
- **`connect`**: Connects a socket to a remote address.
  ```c
  int connect(
    SOCKET s,
    const struct sockaddr* name,
    int namelen
  );
  ```
- **`send` / `recv`**: Sends and receives data through a socket.
  ```c
  int send(
    SOCKET s,
    const char* buf,
    int len,
    int flags
  );

  int recv(
    SOCKET s,
    char* buf,
    int len,
    int flags
  );
  ```

**Example Usage:**
Creating a simple TCP client and server.

**Server:**
```c
SOCKET server = socket(AF