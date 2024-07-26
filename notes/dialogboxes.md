# Dialog boxes in Win32

In Win32 programming, dialog boxes are a common way to interact with users. They are used to prompt for input, display information, or allow users to make selections. There are two primary types of dialog boxes: modal and modeless.

### **1. Modal Dialog Boxes**

A modal dialog box requires the user to interact with it before they can return to the main application window. It captures all user input, preventing interaction with other windows of the application until the dialog box is closed.

#### **Creating a Modal Dialog Box**

1. **Define the Dialog Box Template**

   Define a dialog box template using a resource script (e.g., `dialog.rc`):

   ```cpp
   // dialog.rc
   IDD_MYDIALOG DIALOGEX 0, 0, 200, 100
   STYLE DS_SETFONT | WS_POPUP | WS_CAPTION | WS_SYSMENU
   FONT 8, "MS Shell Dlg"
   BEGIN
       DEFPUSHBUTTON   "OK",IDOK,50,70,50,14
       PUSHBUTTON      "Cancel",IDCANCEL,110,70,50,14
       LTEXT           "Enter your name:",IDC_STATIC,10,10,80,10
       EDITTEXT        IDC_EDIT_NAME,10,25,180,14,ES_AUTOHSCROLL
   END
   ```

2. **Create and Display the Modal Dialog Box**

   Use the `DialogBox` function to create and display the dialog box. This function blocks the caller until the dialog box is closed.

   ```cpp
   // Main window procedure
   LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
       switch (uMsg) {
           case WM_COMMAND:
               if (LOWORD(wParam) == IDM_SHOWDIALOG) {
                   // Show the modal dialog box
                   DialogBox(hInstance, MAKEINTRESOURCE(IDD_MYDIALOG), hwnd, DialogProc);
               }
               break;
           case WM_DESTROY:
               PostQuitMessage(0);
               break;
           default:
               return DefWindowProc(hwnd, uMsg, wParam, lParam);
       }
       return 0;
   }

   // Dialog procedure for the modal dialog box
   INT_PTR CALLBACK DialogProc(HWND hDlg, UINT uMsg, WPARAM wParam, LPARAM lParam) {
       switch (uMsg) {
           case WM_COMMAND:
               if (LOWORD(wParam) == IDOK) {
                   // Handle OK button
                   // Retrieve the text from the edit control
                   char name[100];
                   GetDlgItemText(hDlg, IDC_EDIT_NAME, name, sizeof(name));
                   MessageBox(hDlg, name, "Name Entered", MB_OK);
                   EndDialog(hDlg, IDOK);
               } else if (LOWORD(wParam) == IDCANCEL) {
                   EndDialog(hDlg, IDCANCEL);
               }
               break;
           default:
               return FALSE;
       }
       return TRUE;
   }
   ```

### **2. Modeless Dialog Boxes**

A modeless dialog box allows the user to interact with other windows of the application while the dialog box is open. It does not block the rest of the application.

#### **Creating a Modeless Dialog Box**

1. **Define the Dialog Box Template**

   Define the dialog box template as you would for a modal dialog.

2. **Create and Display the Modeless Dialog Box**

   Use the `CreateDialog` function to create a modeless dialog box. This function returns a handle to the dialog box, which you can use to show and manage it.

   ```cpp
   // Main window procedure
   LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
       static HWND hModelessDlg;

       switch (uMsg) {
           case WM_COMMAND:
               if (LOWORD(wParam) == IDM_SHOWDIALOG) {
                   if (!hModelessDlg) {
                       // Create the modeless dialog box
                       hModelessDlg = CreateDialog(hInstance, MAKEINTRESOURCE(IDD_MYDIALOG), hwnd, DialogProc);
                       ShowWindow(hModelessDlg, SW_SHOW);
                   }
               }
               break;
           case WM_DESTROY:
               PostQuitMessage(0);
               break;
           default:
               return DefWindowProc(hwnd, uMsg, wParam, lParam);
       }
       return 0;
   }

   // Dialog procedure for the modeless dialog box
   INT_PTR CALLBACK DialogProc(HWND hDlg, UINT uMsg, WPARAM wParam, LPARAM lParam) {
       switch (uMsg) {
           case WM_COMMAND:
               if (LOWORD(wParam) == IDOK) {
                   // Handle OK button
                   // Retrieve the text from the edit control
                   char name[100];
                   GetDlgItemText(hDlg, IDC_EDIT_NAME, name, sizeof(name));
                   MessageBox(hDlg, name, "Name Entered", MB_OK);
               } else if (LOWORD(wParam) == IDCANCEL) {
                   DestroyWindow(hDlg);
                   hModelessDlg = NULL;
               }
               break;
           case WM_CLOSE:
               DestroyWindow(hDlg);
               hModelessDlg = NULL;
               break;
           default:
               return FALSE;
       }
       return TRUE;
   }
   ```

### **3. Summary**

- **Modal Dialog Boxes**:
  - Requires user interaction before returning to the main application.
  - Created with `DialogBox` function.
  - Blocks the main application window until the dialog box is closed.

- **Modeless Dialog Boxes**:
  - Allows interaction with other windows of the application while open.
  - Created with `CreateDialog` function.
  - Does not block the main application window.

By using modal and modeless dialog boxes appropriately, you can create a more interactive and user-friendly application, providing different levels of user engagement based on your application's needs.