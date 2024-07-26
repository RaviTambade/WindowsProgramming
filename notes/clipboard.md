#  Clipboard API in Win32

The Clipboard API in Win32 is a set of functions that allow applications to interact with the system clipboard. This API is used for copying and pasting data between applications, and it supports various data formats such as text, images, and custom data formats.

Here’s a detailed look at how to use the Clipboard API in Win32:

### 1. **Overview of Clipboard API**

The Clipboard API provides functions to:
- Open and close the clipboard.
- Empty or set the contents of the clipboard.
- Retrieve data from the clipboard.
- Set data on the clipboard.

### 2. **Common Functions**

**1. `OpenClipboard`:**
   - Opens the clipboard for examination and prevents other applications from modifying it.
   - Syntax:
     ```c
     BOOL OpenClipboard(HWND hWndNewOwner);
     ```
   - `hWndNewOwner` is typically the handle of the window that will become the new owner of the clipboard.

**2. `CloseClipboard`:**
   - Closes the clipboard.
   - Syntax:
     ```c
     BOOL CloseClipboard(void);
     ```

**3. `EmptyClipboard`:**
   - Empties the clipboard.
   - Syntax:
     ```c
     BOOL EmptyClipboard(void);
     ```

**4. `SetClipboardData`:**
   - Places data on the clipboard in a specified format.
   - Syntax:
     ```c
     HANDLE SetClipboardData(UINT uFormat, HANDLE hMem);
     ```
   - `uFormat` specifies the format of the data (e.g., `CF_TEXT` for plain text).
   - `hMem` is a handle to the data to be placed on the clipboard.

**5. `GetClipboardData`:**
   - Retrieves data from the clipboard in a specified format.
   - Syntax:
     ```c
     HANDLE GetClipboardData(UINT uFormat);
     ```
   - `uFormat` specifies the format of the data to retrieve.

**6. `GlobalAlloc` and `GlobalLock`:**
   - `GlobalAlloc` allocates memory, and `GlobalLock` locks it into the address space of the calling process. These functions are used when setting clipboard data.
   - Syntax for `GlobalAlloc`:
     ```c
     HGLOBAL GlobalAlloc(UINT uFlags, SIZE_T dwBytes);
     ```
   - Syntax for `GlobalLock`:
     ```c
     LPVOID GlobalLock(HGLOBAL hMem);
     ```

**7. `GlobalUnlock`:**
   - Unlocks a global memory object.
   - Syntax:
     ```c
     BOOL GlobalUnlock(HGLOBAL hMem);
     ```

### 3. **Example Usage**

Here’s a basic example that demonstrates how to copy a text string to the clipboard and then paste it:

**Copying Text to Clipboard:**

```c
#include <windows.h>
#include <stdio.h>

void CopyTextToClipboard(const char* text) {
    // Open the clipboard
    if (!OpenClipboard(NULL)) {
        printf("Failed to open clipboard.\n");
        return;
    }

    // Empty the clipboard
    if (!EmptyClipboard()) {
        printf("Failed to empty clipboard.\n");
        CloseClipboard();
        return;
    }

    // Allocate global memory for the text
    HGLOBAL hMem = GlobalAlloc(GMEM_MOVEABLE, strlen(text) + 1);
    if (hMem == NULL) {
        printf("Failed to allocate memory.\n");
        CloseClipboard();
        return;
    }

    // Lock the memory and copy the text
    char* pBuf = (char*)GlobalLock(hMem);
    if (pBuf == NULL) {
        printf("Failed to lock memory.\n");
        GlobalFree(hMem);
        CloseClipboard();
        return;
    }
    strcpy(pBuf, text);
    GlobalUnlock(hMem);

    // Set the data on the clipboard
    if (SetClipboardData(CF_TEXT, hMem) == NULL) {
        printf("Failed to set clipboard data.\n");
        GlobalFree(hMem);
    }

    // Close the clipboard
    CloseClipboard();
}

int main() {
    CopyTextToClipboard("Hello, Clipboard!");
    return 0;
}
```

**Pasting Text from Clipboard:**

```c
#include <windows.h>
#include <stdio.h>

void PasteTextFromClipboard() {
    // Open the clipboard
    if (!OpenClipboard(NULL)) {
        printf("Failed to open clipboard.\n");
        return;
    }

    // Get the data from the clipboard
    HGLOBAL hMem = GetClipboardData(CF_TEXT);
    if (hMem == NULL) {
        printf("Failed to get clipboard data.\n");
        CloseClipboard();
        return;
    }

    // Lock the memory and read the text
    char* pBuf = (char*)GlobalLock(hMem);
    if (pBuf == NULL) {
        printf("Failed to lock memory.\n");
        CloseClipboard();
        return;
    }

    // Print the text
    printf("Clipboard contains: %s\n", pBuf);
    GlobalUnlock(hMem);

    // Close the clipboard
    CloseClipboard();
}

int main() {
    PasteTextFromClipboard();
    return 0;
}
```

### 4. **Data Formats**

**Common Formats:**
- **`CF_TEXT`**: Plain text.
- **`CF_BITMAP`**: Bitmap image.
- **`CF_UNICODETEXT`**: Unicode text.
- **`CF_HDROP`**: File drop list.

**Custom Formats:**
- Applications can define custom formats using `RegisterClipboardFormat`.

**Example of Registering a Custom Format:**
```c
UINT uFormat = RegisterClipboardFormat("MyCustomFormat");
```

**Retrieving Custom Format Data:**
```c
HANDLE hData = GetClipboardData(uFormat);
```

### 5. **Clipboard Ownership**

- Applications that open the clipboard with `OpenClipboard` become the owner of the clipboard until `CloseClipboard` is called.
- This prevents other applications from modifying the clipboard data while it's being used.

### Summary

The Clipboard API in Win32 allows for efficient copying, pasting, and manipulation of data between applications. By utilizing functions such as `OpenClipboard`, `SetClipboardData`, and `GetClipboardData`, applications can easily interact with the clipboard, supporting various data formats and enabling seamless data transfer.