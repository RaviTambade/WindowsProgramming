# Subclassing a child window in Win32

Subclassing a child window in Win32 involves altering the behavior of a specific window (child window) by subclassing it. Subclassing allows you to intercept and handle messages sent to a window, providing a way to modify or extend the default behavior of the window.

### **Steps for Subclassing a Child Window**

1. **Subclassing a Window**
2. **Handling Messages in Subclass Procedure**
3. **Restoring the Original Window Procedure**

### **1. Subclassing a Window**

Subclassing involves setting a new window procedure for a window. This new procedure is called whenever the window receives a message.

**Steps:**

1. **Save the Original Window Procedure**: Before subclassing, save the original window procedure pointer.
2. **Set the New Window Procedure**: Use the `SetWindowSubclass` function to set a new procedure for the child window.

**Example:**

```c
#include <windows.h>

WNDPROC g_pOldWndProc; // Global variable to store the original window procedure

// Subclass procedure for the child window
LRESULT CALLBACK ChildWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam) {
    switch (message) {
        case WM_PAINT:
            // Handle the WM_PAINT message
            {
                PAINTSTRUCT ps;
                HDC hdc = BeginPaint(hWnd, &ps);
                // Custom drawing code here
                EndPaint(hWnd, &ps);
            }
            break;

        case WM_LBUTTONDOWN:
            // Handle left mouse button click
            MessageBox(hWnd, L"Child window clicked!", L"Info", MB_OK);
            break;

        // Forward messages to the original window procedure
        default:
            return CallWindowProc(g_pOldWndProc, hWnd, message, wParam, lParam);
    }
    return 0;
}
```

### **2. Handling Messages in Subclass Procedure**

In the subclass procedure (`ChildWndProc` in the example above), handle specific messages you are interested in and forward others to the original window procedure.

**Example of handling specific messages:**

```c
switch (message) {
    case WM_PAINT:
        // Custom painting code
        break;

    case WM_LBUTTONDOWN:
        // Custom click handling code
        break;

    default:
        // Forward other messages to the original procedure
        return CallWindowProc(g_pOldWndProc, hWnd, message, wParam, lParam);
}
```

### **3. Restoring the Original Window Procedure**

If you need to restore the original window procedure (e.g., when destroying the window or subclassing is no longer needed), use `RemoveWindowSubclass`.

**Example:**

```c
BOOL UnsubclassChildWindow(HWND hWnd) {
    return RemoveWindowSubclass(hWnd, ChildWndProc, 0);
}
```

### **Complete Example**

Here's a complete example demonstrating how to subclass a child window to handle specific messages:

```c
#include <windows.h>

WNDPROC g_pOldWndProc; // To store the original window procedure

// Subclass procedure for the child window
LRESULT CALLBACK ChildWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam) {
    switch (message) {
        case WM_PAINT:
            {
                PAINTSTRUCT ps;
                HDC hdc = BeginPaint(hWnd, &ps);
                FillRect(hdc, &ps.rcPaint, (HBRUSH)GetStockObject(WHITE_BRUSH));
                EndPaint(hWnd, &ps);
            }
            break;

        case WM_LBUTTONDOWN:
            MessageBox(hWnd, L"Child window clicked!", L"Info", MB_OK);
            break;

        default:
            return CallWindowProc(g_pOldWndProc, hWnd, message, wParam, lParam);
    }
    return 0;
}

// Window procedure for the parent window
LRESULT CALLBACK ParentWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam) {
    switch (message) {
        case WM_CREATE:
            {
                HWND hChild = CreateWindowEx(
                    0, L"BUTTON", L"Click Me",
                    WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
                    10, 10, 100, 30,
                    hWnd, NULL, ((LPCREATESTRUCT)lParam)->hInstance, NULL
                );

                // Subclass the child window
                g_pOldWndProc = (WNDPROC)SetWindowSubclass(hChild, ChildWndProc, 0, 0);
            }
            break;

        case WM_DESTROY:
            PostQuitMessage(0);
            break;

        default:
            return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}

// WinMain function
int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PWSTR lpCmdLine, int nCmdShow) {
    const wchar_t CLASS_NAME[] = L"Parent Window Class";

    WNDCLASS wc = {0};
    wc.lpfnWndProc = ParentWndProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

    HWND hWnd = CreateWindowEx(
        0,
        CLASS_NAME,
        L"Parent Window",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
        NULL, NULL, hInstance, NULL
    );

    if (hWnd == NULL) {
        return 0;
    }

    ShowWindow(hWnd, nCmdShow);
    UpdateWindow(hWnd);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}
```

### **Explanation**

- **Subclass Procedure (`ChildWndProc`)**: Handles custom messages for the child window and forwards others to the original procedure.
- **Parent Window Procedure (`ParentWndProc`)**: Creates a child window and sets the subclass procedure for it using `SetWindowSubclass`.
- **`SetWindowSubclass`**: Sets a new window procedure for the child window and saves the old one.
- **`CallWindowProc`**: Forwards messages to the original window procedure if not handled in the subclass procedure.

### **Important Considerations**

1. **Ensure Thread Safety**: Make sure that subclassing operations are thread-safe if your application is multithreaded.
2. **Memory Management**: Properly manage memory and resources, especially when dealing with multiple subclassing or dynamic window creation.
3. **Avoid Infinite Loops**: Be careful with message handling to avoid creating infinite loops or recursive calls.
4. **Handle WM_DESTROY**: Clean up resources and restore the original procedure if needed when the window is destroyed.

By following these practices, you can effectively manage and customize the behavior of child windows in a Win32 application using subclassing.