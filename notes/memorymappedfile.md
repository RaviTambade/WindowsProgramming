# Memory-mapped files

Memory-mapped files are a powerful feature in Win32 systems that enable files or portions of files to be mapped directly into the address space of a process. This mechanism provides a way to work with file data as if it were in memory, offering advantages in performance and simplicity for certain types of file operations. Here’s a comprehensive look at how memory-mapped files work in Win32:

### 1. **Concept of Memory-Mapped Files**

**Memory-Mapped Files:**
- Memory-mapped files map a file or a portion of a file into the address space of a process.
- This allows the process to access the file's contents as if they were in memory, using normal memory operations.

**Advantages:**
- **Efficiency:** Reduces the overhead of file I/O operations, as the OS handles reading and writing data directly to and from memory.
- **Simplicity:** Simplifies file access by treating file data as if it were a block of memory, eliminating the need for explicit read/write operations.
- **Concurrency:** Allows multiple processes to share and manipulate the same file content.

### 2. **Creating and Using Memory-Mapped Files**

**Functions Involved:**

1. **`CreateFile`**: Opens or creates a file that will be mapped into memory.
   ```c
   HANDLE CreateFile(
     LPCSTR                lpFileName,
     DWORD                 dwDesiredAccess,
     DWORD                 dwShareMode,
     LPSECURITY_ATTRIBUTES lpSecurityAttributes,
     DWORD                 dwCreationDisposition,
     DWORD                 dwFlagsAndAttributes,
     HANDLE                hTemplateFile
   );
   ```

2. **`CreateFileMapping`**: Creates a file mapping object that can be used to map a file into memory.
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

3. **`MapViewOfFile`**: Maps a view of a file mapping into the address space of the calling process.
   ```c
   LPVOID MapViewOfFile(
     HANDLE hFileMappingObject,
     DWORD  dwDesiredAccess,
     DWORD  dwFileOffsetHigh,
     DWORD  dwFileOffsetLow,
     SIZE_T dwNumberOfBytesToMap
   );
   ```

4. **`UnmapViewOfFile`**: Unmaps a mapped view of a file from the address space of the calling process.
   ```c
   BOOL UnmapViewOfFile(
     LPCVOID lpBaseAddress
   );
   ```

5. **`CloseHandle`**: Closes the handle to the file mapping object when it is no longer needed.
   ```c
   BOOL CloseHandle(
     HANDLE hObject
   );
   ```

**Example Usage:**

Here's a step-by-step example of how to create and use a memory-mapped file:

```c
#include <windows.h>
#include <stdio.h>

int main() {
    HANDLE hFile = CreateFile(
        "example.dat",
        GENERIC_READ | GENERIC_WRITE,
        0,
        NULL,
        CREATE_ALWAYS,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );

    if (hFile == INVALID_HANDLE_VALUE) {
        printf("Could not open file.\n");
        return 1;
    }

    HANDLE hFileMapping = CreateFileMapping(
        hFile,
        NULL,
        PAGE_READWRITE,
        0,
        1024,  // Size of the mapping
        NULL
    );

    if (hFileMapping == NULL) {
        printf("Could not create file mapping.\n");
        CloseHandle(hFile);
        return 1;
    }

    LPVOID lpMapView = MapViewOfFile(
        hFileMapping,
        FILE_MAP_ALL_ACCESS,
        0,
        0,
        0
    );

    if (lpMapView == NULL) {
        printf("Could not map view of file.\n");
        CloseHandle(hFileMapping);
        CloseHandle(hFile);
        return 1;
    }

    // Use the memory-mapped file
    char* pBuf = (char*)lpMapView;
    sprintf(pBuf, "Hello, Memory-Mapped File!");

    printf("Data written to memory-mapped file: %s\n", pBuf);

    // Clean up
    UnmapViewOfFile(lpMapView);
    CloseHandle(hFileMapping);
    CloseHandle(hFile);

    return 0;
}
```

### 3. **Memory-Mapped Files and Concurrency**

**Shared Memory:**
- Memory-mapped files can be used for inter-process communication (IPC) by mapping the same file into the address space of multiple processes.
- Changes made by one process are visible to others that map the same file.

**Synchronization:**
- Synchronization mechanisms (e.g., mutexes, semaphores) are needed to manage concurrent access and avoid conflicts when multiple processes or threads access the same memory-mapped file.

### 4. **File Mapping Protection**

**Access Rights:**
- When creating a file mapping, you specify the protection flags such as `PAGE_READONLY`, `PAGE_READWRITE`, or `PAGE_EXECUTE_READWRITE`.
- The protection flags determine how the memory-mapped file can be accessed (read, write, execute).

**Size Limitations:**
- The size of the memory-mapped file is limited by the available virtual address space. For 32-bit systems, this might be constrained compared to 64-bit systems.

### Summary

Memory-mapped files in Win32 provide an efficient and flexible way to work with file data by mapping it directly into a process's address space. This approach simplifies file operations and can improve performance by leveraging the operating system's capabilities to handle file I/O. However, managing synchronization and access control is crucial when dealing with multiple processes or threads.