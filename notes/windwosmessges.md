
# Windows messages 

In Win32 programming, Windows messages are fundamental to how the operating system communicates with applications and how applications interact with the user and system. Understanding how to handle and use Windows messages is essential for creating responsive and functional applications.

### **Understanding Windows Messages**

Windows messages are used for communication between the system and applications, as well as between different parts of an application. These messages are sent to a window's message queue and processed by the window's procedure.

### **Key Concepts**

1. **Message Loop**
2. **Window Procedure**
3. **Common Messages**
4. **Custom Messages**
5. **Handling Messages**

### **1. Message Loop**

The message loop is a fundamental part of a Win32 application. It retrieves messages from the application's message queue and dispatches them to the appropriate window procedure.

**Basic Message Loop Example:**

```c
MSG msg;
while (GetMessage(&msg, NULL, 0, 0)) {
    TranslateMessage(&msg);  // Translates virtual-key codes into character messages
    DispatchMessage(&msg);   // Dispatches the message to the window procedure
}
```

### **2. Window Procedure**

The window procedure is a function that processes messages sent to a window. Each window has an associated window procedure that handles messages such as user inputs and system requests.

**Window Procedure Syntax:**

```c
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
        case WM_PAINT:
            // Handle paint messages
            break;
        
        case WM_DESTROY:
            PostQuitMessage(0);
            break;
        
        // Handle other messages here
        default:
            return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
    return 0;
}
```

### **3. Common Messages**

Here are some common Windows messages and their typical uses:

- **WM_CREATE**: Sent when a window is being created.
- **WM_DESTROY**: Sent when a window is being destroyed. It is used to post a quit message to the message loop.
- **WM_PAINT**: Sent when a window needs to be repainted. Handle this message to perform custom drawing.
- **WM_SIZE**: Sent when a window is resized. Use this to adjust layout or perform resizing operations.
- **WM_KEYDOWN**: Sent when a key is pressed.
- **WM_LBUTTONDOWN**: Sent when the left mouse button is pressed.

**Examples:**

- **WM_PAINT Example:**

```c
case WM_PAINT:
    {
        PAINTSTRUCT ps;
        HDC hdc = BeginPaint(hwnd, &ps);
        // Custom painting code here
        EndPaint(hwnd, &ps);
    }
    break;
```

- **WM_KEYDOWN Example:**

```c
case WM_KEYDOWN:
    switch (wParam) {
        case VK_ESCAPE:
            PostMessage(hwnd, WM_CLOSE, 0, 0);
            break;
        // Handle other keys here
    }
    break;
```

### **4. Custom Messages**

Custom messages are user-defined messages that can be used to communicate between different parts of an application or to extend functionality.

**Define a Custom Message:**

```c
#define WM_USER_CUSTOM (WM_USER + 1)
```

**Send a Custom Message:**

```c
PostMessage(hwnd, WM_USER_CUSTOM, wParam, lParam);
```

**Handle a Custom Message:**

```c
case WM_USER_CUSTOM:
    // Handle custom message
    break;
```

### **5. Handling Messages**

Proper handling of messages is crucial for a responsive application. Here’s how to manage messages effectively:

- **Filter and Dispatch**: Use `TranslateMessage` to process keyboard input and `DispatchMessage` to send messages to the window procedure.
- **Forward Messages**: Use `DefWindowProc` to ensure default processing of messages that are not explicitly handled.
- **Optimize Processing**: Keep message handling efficient to avoid blocking the message loop.

### **Complete Example**

Here’s a simple example of a Win32 application that handles a few common messages:

```c
#include <windows.h>

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
        case WM_DESTROY:
            PostQuitMessage(0);
            break;

        case WM_PAINT:
            {
                PAINTSTRUCT ps;
                HDC hdc = BeginPaint(hwnd, &ps);
                RECT rect = {10, 10, 100, 100};
                FillRect(hdc, &rect, (HBRUSH)GetStockObject(BLACK_BRUSH));
                EndPaint(hwnd, &ps);
            }
            break;

        case WM_KEYDOWN:
            if (wParam == VK_ESCAPE) {
                PostMessage(hwnd, WM_CLOSE, 0, 0);
            }
            break;

        default:
            return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
    return 0;
}

int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PWSTR lpCmdLine, int nCmdShow) {
    const wchar_t CLASS_NAME[] = L"Sample Window Class";

    WNDCLASS wc = {0};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

    HWND hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        L"Sample Window",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
        NULL, NULL, hInstance, NULL
    );

    if (hwnd == NULL) {
        return 0;
    }

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}
```

### **Summary**

- **Message Loop**: Essential for processing messages.
- **Window Procedure**: Handles messages sent to windows.
- **Common Messages**: Include `WM_PAINT`, `WM_DESTROY`, `WM_SIZE`, etc.
- **Custom Messages**: Defined by the user for application-specific needs.
- **Handling Messages**: Efficient handling and forwarding are crucial for responsive applications.

Understanding and properly handling Windows messages is key to developing robust Win32 applications.