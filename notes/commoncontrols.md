
#  Common controls in Win32

In Win32 programming, common controls are pre-defined UI elements provided by the Windows API that simplify the creation of standard interface components. They include elements like buttons, edit controls, list boxes, and more. These controls are part of the Common Controls Library, which is an extension of the Windows API designed to make it easier to implement standard user interface components.

### **Common Controls Overview**

Common controls are managed through the `CommCtrl` library. This library provides a set of pre-defined controls that are common across many Windows applications. They help ensure consistency and can save development time.

### **1. Initialization of Common Controls**

Before creating any common controls, you need to initialize the Common Controls Library. This is typically done in the `WinMain` function of your application:

```cpp
#include <windows.h>
#include <commctrl.h>

int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    INITCOMMONCONTROLSEX icc;
    icc.dwSize = sizeof(icc);
    icc.dwICC = ICC_WIN95_CLASSES; // Initialize a set of common controls
    InitCommonControlsEx(&icc);

    // Create your window and other UI elements here

    return 0;
}
```

### **2. Types of Common Controls**

Here’s a brief overview of some commonly used controls:

#### **2.1 Button**

Buttons can be created using the `BUTTON` class and include standard push buttons, checkboxes, radio buttons, and more.

**Example: Creating a Push Button**

```cpp
HWND hButton = CreateWindowEx(
    0, "BUTTON", "Click Me",
    WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
    10, 10, 100, 30,
    hwnd, (HMENU)ID_BUTTON, hInstance, NULL
);
```

#### **2.2 Edit Control**

Edit controls are used for single-line or multi-line text input.

**Example: Creating a Single-Line Edit Control**

```cpp
HWND hEdit = CreateWindowEx(
    0, "EDIT", NULL,
    WS_TABSTOP | WS_VISIBLE | WS_CHILD | ES_LEFT,
    10, 50, 200, 30,
    hwnd, (HMENU)ID_EDIT, hInstance, NULL
);
```

#### **2.3 List Box**

List boxes display a list of items and allow the user to select one or more items.

**Example: Creating a List Box**

```cpp
HWND hListBox = CreateWindowEx(
    0, "LISTBOX", NULL,
    WS_TABSTOP | WS_VISIBLE | WS_CHILD | LBS_STANDARD,
    10, 90, 200, 100,
    hwnd, (HMENU)ID_LISTBOX, hInstance, NULL
);

// Add items to the list box
SendMessage(hListBox, LB_ADDSTRING, 0, (LPARAM)"Item 1");
SendMessage(hListBox, LB_ADDSTRING, 0, (LPARAM)"Item 2");
```

#### **2.4 Combo Box**

Combo boxes combine a text box with a drop-down list of items.

**Example: Creating a Combo Box**

```cpp
HWND hComboBox = CreateWindowEx(
    0, "COMBOBOX", NULL,
    WS_TABSTOP | WS_VISIBLE | WS_CHILD | CBS_DROPDOWN,
    10, 200, 200, 100,
    hwnd, (HMENU)ID_COMBOBOX, hInstance, NULL
);

// Add items to the combo box
SendMessage(hComboBox, CB_ADDSTRING, 0, (LPARAM)"Item 1");
SendMessage(hComboBox, CB_ADDSTRING, 0, (LPARAM)"Item 2");
```

#### **2.5 Status Bar**

A status bar typically displays information about the state of the application.

**Example: Creating a Status Bar**

```cpp
HWND hStatusBar = CreateWindowEx(
    0, STATUSCLASSNAME, NULL,
    WS_CHILD | WS_VISIBLE | SBARS_SIZEGRIP,
    0, 0, 0, 0,
    hwnd, (HMENU)ID_STATUSBAR, hInstance, NULL
);

int statusParts[] = {100, 200, -1};
SendMessage(hStatusBar, SB_SETPARTS, (WPARAM)ARRAYSIZE(statusParts), (LPARAM)statusParts);
SendMessage(hStatusBar, SB_SETTEXT, (WPARAM)0, (LPARAM)L"Ready");
```

#### **2.6 Toolbar**

Toolbars provide a set of buttons for common commands.

**Example: Creating a Toolbar**

```cpp
HWND hToolbar = CreateWindowEx(
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
```

### **3. Handling Control Notifications**

Common controls send notifications to their parent window when user actions occur. Handle these notifications in the window procedure:

```cpp
case WM_COMMAND: {
    switch (LOWORD(wParam)) {
        case ID_BUTTON:
            // Handle button click
            MessageBox(hwnd, "Button Clicked", "Info", MB_OK);
            break;
        case ID_EDIT:
            // Handle edit control actions
            break;
        case ID_LISTBOX:
            // Handle list box actions
            break;
        case ID_COMBOBOX:
            // Handle combo box actions
            break;
    }
    break;
}
```

### **4. Summary**

- **Common Controls**: Predefined UI elements like buttons, edit controls, list boxes, combo boxes, status bars, and toolbars.
- **Initialization**: Use `InitCommonControlsEx` to initialize the controls library.
- **Creation**: Use `CreateWindowEx` with appropriate class names.
- **Handling Notifications**: Process control notifications in the window procedure to respond to user actions.

By utilizing these common controls, you can create more sophisticated and user-friendly Win32 applications with minimal effort.