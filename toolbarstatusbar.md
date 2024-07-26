Implementing a toolbar and status bar in a Win32 application involves using the Windows API to create and manage these UI components. Both the toolbar and status bar are common elements in applications that provide additional functionality and user feedback.

### **1. Toolbar Implementation**

The toolbar typically provides buttons for frequently used commands or actions. You can use the `ToolBar` control for this purpose.

**Steps to Implement a Toolbar:**

1. **Include Required Headers:**

   ```cpp
   #include <windows.h>
   #include <commctrl.h>
   ```

2. **Initialize the Common Controls Library:**

   Ensure that you initialize the common controls library in your `WinMain` function.

   ```cpp
   INITCOMMONCONTROLSEX icc;
   icc.dwSize = sizeof(icc);
   icc.dwICC = ICC_BAR_CLASSES;
   InitCommonControlsEx(&icc);
   ```

3. **Create the Toolbar:**

   Create the toolbar and add buttons to it.

   ```cpp
   HWND hToolbar;
   TBBUTTON tbb[3]; // Array to hold button definitions
   memset(tbb, 0, sizeof(tbb));
   
   // Define buttons
   tbb[0].iBitmap = 0; // Index of the bitmap
   tbb[0].idCommand = ID_FILE_NEW; // Command ID
   tbb[0].fsState = TBSTATE_ENABLED;
   tbb[0].fsStyle = TBSTYLE_BUTTON;
   tbb[0].iString = (INT_PTR)L"New";
   
   // Initialize the toolbar
   hToolbar = CreateWindowEx(
       0, TOOLBARCLASSNAME, NULL,
       WS_CHILD | WS_VISIBLE | WS_BORDER | CCS_TOP,
       0, 0, 0, 0,
       hwnd, (HMENU)ID_TOOLBAR, hInstance, NULL
   );
   
   SendMessage(hToolbar, TB_BUTTONSTRUCTSIZE, (WPARAM)sizeof(TBBUTTON), 0);
   SendMessage(hToolbar, TB_ADDBUTTONS, (WPARAM)ARRAYSIZE(tbb), (LPARAM)&tbb);
   ```

4. **Handle the Toolbar Commands:**

   Handle toolbar commands in your window procedure.

   ```cpp
   case WM_COMMAND:
       switch (LOWORD(wParam)) {
           case ID_FILE_NEW:
               // Handle the New command
               break;
           // Handle other commands
       }
       break;
   ```

### **2. Status Bar Implementation**

The status bar provides feedback about the current state of the application.

**Steps to Implement a Status Bar:**

1. **Create the Status Bar:**

   Create the status bar and set its parts.

   ```cpp
   HWND hStatusBar;
   hStatusBar = CreateWindowEx(
       0, STATUSCLASSNAME, NULL,
       WS_CHILD | WS_VISIBLE | SBARS_SIZEGRIP,
       0, 0, 0, 0,
       hwnd, (HMENU)ID_STATUSBAR, hInstance, NULL
   );
   
   int statusParts[] = {100, 200, -1}; // Define status bar parts
   SendMessage(hStatusBar, SB_SETPARTS, (WPARAM)ARRAYSIZE(statusParts), (LPARAM)statusParts);
   ```

2. **Set Status Bar Text:**

   You can set the text for each part of the status bar.

   ```cpp
   SendMessage(hStatusBar, SB_SETTEXT, (WPARAM)0, (LPARAM)L"Ready");
   ```

3. **Handle the Status Bar Resizing:**

   Update the status bar when the window is resized.

   ```cpp
   case WM_SIZE:
       SendMessage(hStatusBar, WM_SIZE, 0, 0);
       SendMessage(hToolbar, WM_SIZE, 0, 0);
       break;
   ```

### **Complete Example:**

Here’s a complete example that demonstrates creating a toolbar and a status bar in a Win32 application.

```cpp
#include <windows.h>
#include <commctrl.h>

#define ID_TOOLBAR  101
#define ID_STATUSBAR 102
#define ID_FILE_NEW  103

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);

int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    const char CLASS_NAME[] = "Sample Window Class";
    
    WNDCLASS wc = {};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);

    RegisterClass(&wc);

    HWND hwnd = CreateWindowEx(
        0, CLASS_NAME, "Toolbar and StatusBar Example",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
        NULL, NULL, hInstance, NULL
    );

    if (hwnd == NULL) {
        return 0;
    }

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);

    INITCOMMONCONTROLSEX icc;
    icc.dwSize = sizeof(icc);
    icc.dwICC = ICC_BAR_CLASSES | ICC_STANDARD_CLASSES;
    InitCommonControlsEx(&icc);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    static HWND hToolbar, hStatusBar;
    static HINSTANCE hInstance;

    switch (uMsg) {
        case WM_CREATE: {
            hInstance = ((LPCREATESTRUCT)lParam)->hInstance;

            // Create the toolbar
            hToolbar = CreateWindowEx(
                0, TOOLBARCLASSNAME, NULL,
                WS_CHILD | WS_VISIBLE | WS_BORDER | CCS_TOP,
                0, 0, 0, 0,
                hwnd, (HMENU)ID_TOOLBAR, hInstance, NULL
            );
            
            TBBUTTON tbb[] = {
                {0, ID_FILE_NEW, TBSTATE_ENABLED, TBSTYLE_BUTTON, {0}, 0, (INT_PTR)L"New"}
            };
            
            SendMessage(hToolbar, TB_BUTTONSTRUCTSIZE, (WPARAM)sizeof(TBBUTTON), 0);
            SendMessage(hToolbar, TB_ADDBUTTONS, (WPARAM)ARRAYSIZE(tbb), (LPARAM)&tbb);
            SendMessage(hToolbar, TB_AUTOSIZE, 0, 0);

            // Create the status bar
            hStatusBar = CreateWindowEx(
                0, STATUSCLASSNAME, NULL,
                WS_CHILD | WS_VISIBLE | SBARS_SIZEGRIP,
                0, 0, 0, 0,
                hwnd, (HMENU)ID_STATUSBAR, hInstance, NULL
            );
            
            int statusParts[] = {100, 200, -1};
            SendMessage(hStatusBar, SB_SETPARTS, (WPARAM)ARRAYSIZE(statusParts), (LPARAM)statusParts);
            SendMessage(hStatusBar, SB_SETTEXT, (WPARAM)0, (LPARAM)L"Ready");
            break;
        }
        case WM_SIZE: {
            SendMessage(hToolbar, WM_SIZE, 0, 0);
            SendMessage(hStatusBar, WM_SIZE, 0, 0);
            break;
        }
        case WM_COMMAND: {
            if (LOWORD(wParam) == ID_FILE_NEW) {
                MessageBox(hwnd, "New File Command Triggered", "Info", MB_OK);
            }
            break;
        }
        case WM_DESTROY: {
            PostQuitMessage(0);
            break;
        }
        default:
            return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
    return 0;
}
```

### **Summary**

- **Toolbar**: Provides a set of buttons for frequently used commands. Initialize with `CreateWindowEx` using `TOOLBARCLASSNAME` and configure buttons with `TBBUTTON` structures.
- **Status Bar**: Displays status messages or information about the application's state. Initialize with `CreateWindowEx` using `STATUSCLASSNAME` and set text with `SB_SETTEXT`.

By following these steps, you can effectively implement a toolbar and a status bar in your Win32 application, enhancing user interaction and providing useful feedback.