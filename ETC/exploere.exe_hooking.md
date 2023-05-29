# exploere.exe Hooking

When someone accesses a file on the computer, the file explorer is used to search for the file, so how about hooking the file explorer to hide the folder you want to hide?

Q. How to manage files in Windows?<br>
A. First of all, the way Windows fundamentally manages files is not an absolute path.<br>

Because exploere.exe is a program that runs in user mode, use WINAPI to receive the directory structure, parse it, and display it on the screen.<br>

# WINDOWS API About File

File search APIs provided by Windows include "FindFirstFileA", "FindFirstFileExA", "FindFirstFileExW", and "FindFirstFileEw"<br>

Searches for specific files in file path. Because you can use wildcards, it is also possible to search all files.<br>

```
#C
#define _CRT_SECURE_NO_WARNINGS

#include <stdio.h>
#include <Windows.h>
#include <TlHelp32.h>
#include <tchar.h>

int main(int argc, char* argv[])
{
    DWORD PID = 0;
    HANDLE hSnapShot = INVALID_HANDLE_VALUE;
    PROCESSENTRY32 pe32;
    pe32.dwSize = sizeof(PROCESSENTRY32);

    char* filename;
    filename = (char*)malloc(sizeof(char));

    hSnapShot = CreateToolhelp32Snapshot(TH32CS_SNAPALL, NULL);
    Process32First(hSnapShot, &pe32);
    do
    {
        PID = pe32.th32ProcessID;
        printf("PID : %d -> ", PID);

        filename = (char*)pe32.szExeFile;
        printf("FileName : %s\n", filename);
    } while (Process32Next(hSnapShot, &pe32));
    CloseHandle(hSnapShot);

    return 0;
}
```

One of the techniques commonly used in malicious ode is to inspect and enumerate the process list.<br>
The characteristic of malicious code is to find and neutralize anti-debugging or anti-virus programs, or find a process to find a target process for hooking and injection.<br>
API functions used in this case include Process32First, Process32Next, and CreateToolhelp32Snapshot.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/85dd67e1-c169-4fea-b529-e2648e845c1f)


If you want to find a specific process, do the following :<br>


```
#define _CRT_SECURE_NO_WARNINGS

#include <stdio.h>
#include <Windows.h>
#include <TlHelp32.h>
#include <tchar.h>

int main(int argc, char* argv[])
{
    DWORD PID = 0;
    HANDLE hSnapShot = INVALID_HANDLE_VALUE;
    PROCESSENTRY32 pe32 = { 0 };
    pe32.dwSize = sizeof(PROCESSENTRY32);

    hSnapShot = CreateToolhelp32Snapshot(TH32CS_SNAPALL, NULL);
    if (Process32First(hSnapShot, &pe32))
    {
        do
        {
            if (!_tcsicmp(pe32.szExeFile, TEXT("explorer.exe")))
            {
                printf("[+] Find Process\n");
                PID = pe32.th32ProcessID;
                printf("[+] PID : %d\n", PID);
                break;
            }
        } while (Process32Next(hSnapShot, &pe32));
        CloseHandle(hSnapShot);
    }
    return 0;
}
```


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/89f08529-9c11-4165-bfc8-079ccdc92dfb)

<h3>Shell.NameSpace</h3> : Create and returns a Folder object for the specified folder.

```
#JScript
retVal = Shell.NameSpace(
  vDir // Folder The folder to create the object for. Can be a string specifying either a path to a folder or a value of ShellSpecialFolderConstants
)
```

MSDN : https://learn.microsoft.com/ko-kr/windows/win32/shell/folder

<br>
Let's hook "FindFirstFileW", "FindNextFileW" and see how the arguments are used.<br>
<br>
※For reference, the fileData.CFileName structure is 0x2c away from the start of the structure<br>
<br>
Windows also has a shell, and drives, directories, files, network, drives, printers, contorl panels, and the desktop are functions provided by the shell,<br>
and explorer.exe is an extended GUI implementation. To recap, the shell namespace is a larger and more comprehensive version of the Windows filesystem.<br>
<br>
What can be done with it is that developers can extend the functionality of the shell. OneDrive is a typical example, and it can be seen that this shell namespace was used to create virutal folders and files and <br>
list files that are not on the actual computer. In other words, using this, you can create a powerful customized file hierarchy that is different from the general file system structure.<br>
In addition to this, the Recycle Bin is aslo diffent from a normal folder.<br>
<br>
File classification in the shell namespace uses a "PIDL" item identifier list, not a path.<br>
Of course, "PIDL" and file path can be converted to each other, but "PIDL" structure management is easier.<br>
There are also "CSIDLs", which provide predefined constants for special folders such as Recycle Bin, Desktop, Favorites, Picutres, Documents, and so on.<br>
It tells you to use this "CSIDL" to obtain the path and "PIDL" of the system fodler and use it conveniently.<br>

MSDN : https://learn.microsoft.com/ko-kr/windows/win32/shell/csidl

<br>

It is necesary to know a little more about what "PIDL" is.<br>
Like a filesystem, a namespace contains objects called folders and files. And folders or files that are not part of the filesystem are called virtual objects.<br>
Anyway, to implement this, shell namespace is what you can immediately think of to identify each object here is "absoulte path'.
However, since virtual objects do not exist in the file system, a different approach is required, and the Windows shell gives each object an ID for this purpose.<br>
<br>

```
typedef struct _SHITEMID { 
    USHORT cb; //Size
    BYTE   abID[1]; //identifier of the object
} SHITEMID, * LPSHITEMID;
```

In the "SHITEMID" structure, "abID" is an object identifier.<br>
However, the length is not fixed as shown above and is determined by the folder containing this object.<br>
And since there is no standard for determining "abID", this this ID has meaning only in folders that are connected to each other.<br>
Therefore, programs that use it should treat it as a token that is used only in the namespace of a particular folder. And Since the length of the "abID" array is variable, cb contains the entire size of this structure.<br>

Shell Item ID is rarely used alone by default, and it is used by containing IDs of several objects in the "ITEMIDLIST" structure.<br>
<br>

```
#Contains a list of item identifiers.
typedef struct _ITEMIDLIST {
  SHITEMID mkid;
} ITEMIDLIST;
```

Example : my Desktop \\ Uses \\ C: \\ mydir \\ myfile.txt "PIDL"

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/9710705b-f581-4845-a5cf-3783f0d49aa3)

Tip
- IShellFolder interface :  Corresponding methods exposed on all Shell namespace folder objects are used to manage folders.<br>
- Bind : Bindings are primarily used to establish a connection between an interface and the object that implements that interface.<br>
         For example, in the Windows API you can use the IShellFolder interface to perform operations related to folders. The BindToObject function binds with a folder object that implements this interface. This establishes a connection between the objects, and allows you to perform operations on that folder by calling the methods of the IShellFolder interface.<br>
- PITEMID_CHILD : Clones the child "ITEMIDLIST" structure"<br>
         

<br>
Below is how to traverse the "PIDL" file.<br>
1. use the "SHGetDesktopFolder" function to obtain the "IShellFolder" interface of the desktop.<br>
2. Obtain "PIDL" using the path of the folder. This can be done with "ILCreateFromPath".<br>
3. Bind "IShellFodler" obtained in step 1 to "PIDL" obtained in step 2. Using "IShellFolder::BindToOjbect"<br>
4. Obtain the "IEnumIDlist" interface using "IShellFolder::EnumObjects"<br>
5. Obtain child's "PIDL" using "IENUMIDlist::Next"<br>

```
#define _CRT_SECURE_NO_WARNINGS

#include <stdio.h>
#include <Windows.h>
#include <TlHelp32.h>
#include <tchar.h>
#include <shlobj.h>
#include <shlwapi.h>
#include <locale.h>
#include <shobjidl_core.h>

int main()
{

    setlocale(LC_ALL, "korean");

    LPSHELLFOLDER pFolder = NULL;
    //Obtain PIDL Folder List
    PIDLIST_ABSOLUTE dir = ILCreateFromPath("C:\\Users\\white\\Desktop");
    IShellFolder* shellFolder;
    IEnumIDList* list;

    //Child "PIDL" to be retrieved while traversing the folder.
    PITEMID_CHILD child;
    LPWSTR path = new WCHAR[MAX_PATH];

    //Obtain Desktop interface
    if (SHGetDesktopFolder(&shellFolder) == S_OK)
    {
        //Bind to the path want with the acquired interface
        if (shellFolder->BindToObject(dir, NULL, IID_IShellFolder, (void**)&shellFolder) == S_OK)
        {
            //Find Folder & files
            if (shellFolder->EnumObjects(NULL, 0x40 | 0x20, &list) == S_OK)
            {
                //continuously touring
                while (list->Next(1, &child, NULL) == S_OK)
                {
                    //pidl->path conversion
                    SHGetPathFromIDListEx(child, (PWSTR)path, 0x500, GPFIDL_DEFAULT);

                    //output in unicode format
                    printf("[+] %ls\n", path);

                    //The received pidl is returned again.
                    CoTaskMemFree(child);
                }
            }
            //memory cleanup
            shellFolder->Release();
            list->Release();
            delete[] path;
            ILFree(dir);
        }
        else
        {
            printf("[+] Binding fail!\n");
        }
    }
    else
    {
        printf("[+] GetDesktopFolder fail!\n");
    }
    return 0;
}
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/78d1eb26-d301-4fb9-a9ae-072ad48621eb)

<br>

MSDN : https://learn.microsoft.com/en-us/windows/win32/api/shobjidl_core/nf-shobjidl_core-ishellfolder-bindtoobject<br>

<h2>Now let's start hooking.</h2>



# Difference between PIDL (pointer to an identifier list) and FindFirstFile function

<strong>Object identification method:</strong><br>
- PIDL: PIDL is a binary data structure used to uniquely identify an object. Instead of representing a path to a file or folder, PIDL is represented as a sequence of bytes by which an object can be uniquely identified within the system.<br>
- FindFirstFile: The FindFirstFile function is used to find an object through a path in the file system. Starts a search for that path, and returns the handle of the first object on that path.<br>








