#"Rubber band" effect


The "rubber band" effect is a common graphical effect in user interfaces where a line or rectangle is drawn as the user drags the mouse, and it visually stretches or moves to follow the cursor. This effect is often used in applications for selecting areas, drawing shapes, or providing visual feedback during mouse interactions.

### **Implementing Rubber Band Effect in Win32**

Here's a step-by-step guide to implementing a rubber band line effect using the Win32 API:

### **1. Handling Mouse Events**

To create the rubber band effect, you need to handle mouse events to track the starting point and the current position as the mouse is dragged.

### **2. Drawing the Rubber Band Line**

You will use GDI (Graphics Device Interface) functions to draw the line or rectangle.

### **Example Code**

Below is a simple example of how to implement a rubber band effect where a line is drawn while the user drags the mouse. This example demonstrates how to handle the `WM_LBUTTONDOWN`, `WM_MOUSEMOVE`, and `WM_LBUTTONUP` messages to draw the line:

```cpp
#include <windows.h>

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
void DrawRubberBandLine(HDC hdc, POINT startPoint, POINT endPoint);

POINT startPoint = {-1, -1}; // Initial start point for rubber band effect

int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    const char CLASS_NAME[] = "Sample Window Class";

    WNDCLASS wc = {};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

    HWND hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        "Rubber Band Line Example",
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

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    static POINT prevPoint = {-1, -1};

    switch (uMsg) {
        case WM_LBUTTONDOWN: {
            startPoint.x = LOWORD(lParam);
            startPoint.y = HIWORD(lParam);
            prevPoint = startPoint;
            SetCapture(hwnd); // Capture mouse input
            break;
        }
        case WM_MOUSEMOVE: {
            if (wParam & MK_LBUTTON) { // Check if left button is pressed
                HDC hdc = GetDC(hwnd);
                // Draw the previous rubber band line in the background
                DrawRubberBandLine(hdc, startPoint, prevPoint);
                // Draw the current rubber band line
                POINT currentPoint = {LOWORD(lParam), HIWORD(lParam)};
                DrawRubberBandLine(hdc, startPoint, currentPoint);
                prevPoint = currentPoint;
                ReleaseDC(hwnd, hdc);
            }
            break;
        }
        case WM_LBUTTONUP: {
            if (GetCapture() == hwnd) {
                // Draw final line
                HDC hdc = GetDC(hwnd);
                POINT endPoint = {LOWORD(lParam), HIWORD(lParam)};
                DrawRubberBandLine(hdc, startPoint, endPoint);
                ReleaseDC(hwnd, hdc);
                ReleaseCapture(); // Release mouse capture
                startPoint.x = -1; // Reset start point
                prevPoint.x = -1;
            }
            break;
        }
        case WM_PAINT: {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hwnd, &ps);
            // Optional: Redraw the rubber band line if needed
            EndPaint(hwnd, &ps);
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

void DrawRubberBandLine(HDC hdc, POINT startPoint, POINT endPoint) {
    // Use a white pen for drawing the line
    HPEN hPen = CreatePen(PS_SOLID, 1, RGB(0, 0, 0));
    HGDIOBJ oldPen = SelectObject(hdc, hPen);
    // Draw the line
    MoveToEx(hdc, startPoint.x, startPoint.y, NULL);
    LineTo(hdc, endPoint.x, endPoint.y);
    // Clean up
    SelectObject(hdc, oldPen);
    DeleteObject(hPen);
}
```

### **Explanation**

1. **Handling Mouse Events**:
   - **`WM_LBUTTONDOWN`**: Captures the starting point of the rubber band effect and sets mouse capture.
   - **`WM_MOUSEMOVE`**: Draws the line from the starting point to the current mouse position as the mouse is dragged.
   - **`WM_LBUTTONUP`**: Finalizes the line drawing and releases mouse capture.

2. **Drawing the Rubber Band Line**:
   - **`DrawRubberBandLine`**: Uses GDI functions to draw a line from `startPoint` to `endPoint`.

3. **Redraw Handling**:
   - **`WM_PAINT`**: Handles redrawing if needed (not fully utilized in this example).

### **Best Practices**

- **Double Buffering**: To avoid flickering, consider using a memory device context (DC) for drawing and then blit the result to the screen.
- **Performance**: Minimize drawing operations during mouse movement to ensure smooth performance.
- **User Feedback**: Ensure that the visual feedback is clear and responsive to user actions.

### **Summary**

Implementing a rubber band effect in Win32 involves handling mouse events to track user input and using GDI functions to draw the visual effect. By capturing mouse input and drawing dynamically, you can provide interactive feedback in your applications.