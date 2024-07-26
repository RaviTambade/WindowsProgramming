# `WinMain` and the Window Procedure

In Win32 programming, `WinMain` and the Window Procedure are two essential components for creating and managing a Win32 application. Here’s a detailed look at each, including their roles and how they interact:

### **1. WinMain**

`WinMain` is the entry point of a Win32 application. It is analogous to the `main` function in console applications. This function initializes the application, creates the main window, and starts the message loop.

**Syntax of `WinMain`:**

```c
int WINAPI WinMain(
    HINSTANCE hInstance,    // Handle to the current instance of the application
    HINSTANCE hPrevInstance, // Handle to the previous instance (always NULL for Win32)
    LPSTR lpCmdLine,        // Command line arguments as a string
    int nCmdShow            // Show state of the window (SW_SHOWNORMAL, SW_HIDE, etc.)
);
```

**Parameters:**

- `hInstance`: Handle to the instance of the application. This is used to identify the instance of the application and to load resources.
- `hPrevInstance`: Always NULL in Win32 applications. It was used in 16-bit Windows to indicate a previous instance of the application.
- `lpCmdLine`: Command line arguments passed to the application.
- `nCmdShow`: Specifies how the window should be shown (e.g., minimized, maximized).

**Basic Structure of `WinMain`:**

```c
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    // Register window class
    WNDCLASS wc = {0};
    wc.lpfnWndProc = WindowProc; // Set the window procedure
    wc.hInstance = hInstance;
    wc.lpszClassName = "SampleWindowClass";
    RegisterClass(&wc);

    // Create the main window
    HWND hwnd = CreateWindowEx(
        0,
        "SampleWindowClass",
        "Sample Window",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
        NULL,
        NULL,
        hInstance,
        NULL
    );

    if (hwnd == NULL) {
        return 0; // Failed to create window
    }

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);

    // Message loop
    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0; // Exit the application
}
```

### **2. Window Procedure**

The Window Procedure (or WindowProc) is a callback function that processes messages sent to a window. Each window has an associated Window Procedure that handles messages such as painting requests, user input, and other system events.

**Syntax of Window Procedure:**

```c
LRESULT CALLBACK WindowProc(
    HWND hwnd,    // Handle to the window
    UINT uMsg,    // Message identifier
    WPARAM wParam, // Additional message-specific information
    LPARAM lParam  // Additional message-specific information
);
```

**Parameters:**

- `hwnd`: Handle to the window receiving the message.
- `uMsg`: The message identifier (e.g., `WM_PAINT`, `WM_KEYDOWN`).
- `wParam`: Additional message-specific information (depends on the message).
- `lParam`: Additional message-specific information (depends on the message).

**Basic Structure of Window Procedure:**

```c
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
        case WM_DESTROY:
            PostQuitMessage(0); // Post a quit message and exit the message loop
            return 0;

        case WM_PAINT:
            {
                PAINTSTRUCT ps;
                HDC hdc = BeginPaint(hwnd, &ps);
                // Custom drawing code here
                EndPaint(hwnd, &ps);
            }
            return 0;

        default:
            return DefWindowProc(hwnd, uMsg, wParam, lParam); // Default processing for messages
    }
}
```

### **Interaction Between `WinMain` and Window Procedure**

1. **Registration and Creation:**
   - `WinMain` registers a window class with `RegisterClass`, specifying the `WindowProc` as the window procedure.
   - It then creates a window with `CreateWindowEx`, associating the window with the registered class.

2. **Message Loop:**
   - `WinMain` enters a message loop with `GetMessage`, `TranslateMessage`, and `DispatchMessage`.
   - `DispatchMessage` sends messages to the `WindowProc` for processing.

3. **Message Handling:**
   - The `WindowProc` handles specific messages (e.g., `WM_PAINT` for painting, `WM_DESTROY` for cleanup) and forwards others to `DefWindowProc` for default handling.

### **Complete Example**

Here’s a complete Win32 application example incorporating both `WinMain` and `WindowProc`:

```c
#include <windows.h>

// Window Procedure
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
        case WM_DESTROY:
            PostQuitMessage(0);
            return 0;

        case WM_PAINT:
            {
                PAINTSTRUCT ps;
                HDC hdc = BeginPaint(hwnd, &ps);
                FillRect(hdc, &ps.rcPaint, (HBRUSH)(COLOR_WINDOW + 1));
                EndPaint(hwnd, &ps);
            }
            return 0;

        case WM_KEYDOWN:
            if (wParam == VK_ESCAPE) {
                PostMessage(hwnd, WM_CLOSE, 0, 0);
            }
            return 0;

        default:
            return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
}

// Entry Point
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    const char CLASS_NAME[] = "SampleWindowClass";

    WNDCLASS wc = {0};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

    HWND hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        "Sample Window",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
        NULL,
        NULL,
        hInstance,
        NULL
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

- **`WinMain`**: The entry point of a Win32 application that sets up and starts the message loop.
- **Window Procedure**: Handles messages sent to a window, determining how the window responds to user actions and system events.
- **Interaction**: `WinMain` creates the window and message loop, while the Window Procedure handles the specifics of message processing.

By understanding and properly using `WinMain` and the Window Procedure, you can effectively manage the lifecycle and behavior of a Win32 application.