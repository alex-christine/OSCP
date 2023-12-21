---
description: Page covering the basics of the DLL concept
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Dynamic Link Libraries

## Overview

A [DLL](https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library) is a library that contains code and data that can be used by more than one program at the same time. For example, in Windows operating systems, the `Comdlg32` DLL performs common dialog box related functions. Each program can use the functionality that is contained in this DLL to implement an `Open` dialog box. It helps promote code reuse and efficient memory usage.

For the Windows operating systems, much of the functionality of the operating system is provided by DLL. Additionally, when a program is run on one of these Windows operating systems, much of the functionality of the program may be provided by DLLs.

The use of DLLs helps promote modularization of code, code reuse, efficient memory usage, and reduced disk space. So, the operating system and the programs load faster, run faster, and take less disk space on the computer.

When a program uses a DLL, an issue that is called dependency may cause the program not to run. When a program uses a DLL, a dependency is created. If another program overwrites and breaks this dependency, the original program may not successfully run.

With the introduction of the .NET Framework, most dependency problems have been eliminated by using assemblies.

## DLL Search Order

### Factors

Windows has a prescribed [search order](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order) in which it attempts to find a DLL referenced by an application. Many factors affect search order such as:

* **DLL Redirection:** The _DLL loader_ is the part of the operating system (OS) that resolves references to DLLs, loads them, and links them. [DLL redirection](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-redirection) is one of the techniques by whichthe behavior of the _DLL loader_ can be influenced, and one can control which one of several candidate DLLs it actually loads.
* **API Sets:** All versions of Windows share a common base of operating system (OS) components that's called the _core OS_ (in some contexts this common base is also called _OneCore_). In core OS components, Win32 APIs are organized into functional groups called [API Sets](https://learn.microsoft.com/en-us/windows/win32/apiindex/windows-apisets)
* **Side-by-side (SxS) manifest redirection:** (desktop apps only) One can redirect by using an application manifest (also known as a side-by-side application [manifest](https://learn.microsoft.com/en-us/windows/win32/sbscs/manifests), or a fusion manifest)
* **Loaded-module list**. The system can check to see whether a DLL with the same module name is already loaded into memory (no matter which folder it was loaded from)
* **Known DLLs**. If the DLL is on the list of known DLLs for the version of Windows on which the application is running, then the system uses its copy of the known DLL (and the known DLL's dependent DLLs, if any)
* **Safe DLL Search Mode:** (enabled by default) moves the user's current folder later in the search order.
  * To disable safe DLL search mode, create the `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\SafeDllSearchMode` registry value, and set it to 0
  * Calling the [SetDllDirectory](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-setdlldirectorya) function effectively disables safe DLL search mode (while the specified folder is in the search path), and changes the search order from standard

### General Search Order

Broadly speaking the search order (assuming safe search mode is on) for a DLL referenced by an application is:

1. **Directory from which the application loaded**
2. **System Directory**: Use the [GetSystemDirectory](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) function to retrieve the path of this folder
3. **16-bit System Directory:** There's no function that obtains the path of this folder, but it is searched
4. **Windows Directory:** Use the [GetWindowsDirectory](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) function to get the path of this folder
5. **Current Directory**
6. **Directories listed in the `PATH` environment variable**

## DLL Operation

When a DLL is loaded in an application [two methods of linking](https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library#types-of-dlls) allow the application to call library functions:

1. **Load-Time Dynamic Linking:** an application makes explicit calls to exported DLL functions like local functions.&#x20;
   * Use of load-time dynamic linking, requires the coder to **provide both a header** (`.h`) **file and an import library** (`.lib`) file when the application is compiled and linked. The linker will then provide the system with the information that is required to load the DLL and resolve the exported DLL function locations at load time
2. **Run-Time Dynamic Linking:** in which an application calls either the `LoadLibrary` function or the `LoadLibraryEx` function to load the DLL at run time. After the DLL is successfully loaded, the application coder can use the `GetProcAddress` function to obtain the address of the exported DLL function that they want to call.
   * When run-time dynamic linking is used, an import library file (**.lib**) is not needed
   * Some benefits of run-time dynamic linking include
     * Improved startup performance
     * Improved ease of use
     * With run-time dynamic linking, an application can branch to load different modules as required. This is important when developing multiple-language versions

### DLL Entry Point

When a DLL is created, it is possible to define an optional [entry point function](https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library#the-dll-entry-point) (named `DllMain`). This function is called when processes or threads attach themselves to the DLL or detached themselves from the DLL. It can be used to initialize data structures or to destroy data structures as required by the DLL. Additionally, if the application is multi-threaded, it can use thread local storage (TLS) to allocate memory that is private to each thread in the entry point function.

If a DLL doesn't have a `DllMain` entry point function, it only provides resources.

Microsoft [provides](https://learn.microsoft.com/en-us/windows/win32/dlls/dllmain#example) this basic example of a DLL entry point function:

```cpp
BOOL WINAPI DllMain(
    HINSTANCE hinstDLL,  // handle to DLL module
    DWORD fdwReason,     // reason for calling function
    LPVOID lpvReserved )  // reserved
{
    // Perform actions based on the reason for calling.
    switch( fdwReason ) 
    { 
        case DLL_PROCESS_ATTACH:
            // Initialize once for each new process.
            // Return FALSE to fail DLL load.
            break;

        case DLL_THREAD_ATTACH:
            // Do thread-specific initialization.
            break;

        case DLL_THREAD_DETACH:
            // Do thread-specific cleanup.
            break;

        case DLL_PROCESS_DETACH:
        
            if (lpvReserved != nullptr)
            {
                break; // do not do cleanup if process termination scenario
            }
            
            // Perform any necessary cleanup.
            break;
    }
    
    return TRUE;  // Successful DLL_PROCESS_ATTACH.
}
```

Note sometimes the name of the argument `fdwReason` is shown as `ul_reason_for_call` but everything else is the same. This is shown in the truncated example:

```cpp
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
            break;
        ...
```

In either case, the entry function usually contains a four-case switch statement (`DLL_PROCESS_ATTACH`, `DLL_THREAD_ATTACH`, `DLL_THREAD_DETACH`, `DLL_PROCESS_DETACH`) which handle situations when the DLL is loaded or unloaded by a process or thread. They are commonly used to perform initialization tasks for the DLL or tasks related to exiting the DLL. The example above only contains a `break` statement for each of the cases and does not really serve any purpose other than to demonstrate the structure for a `DllMain` function.

Per [Microsoft's documentation](https://learn.microsoft.com/en-us/windows/win32/dlls/dllmain#parameters), the four entry/exit states are:

<table><thead><tr><th width="262">Value</th><th>Description</th></tr></thead><tbody><tr><td><code>DLL_PROCESS_ATTACH</code> (<code>0</code>)</td><td>The DLL is being loaded into the virtual address space of the current process as a result of the process starting up or as a result of a call to <a href="https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya"><code>LoadLibrary</code></a>.</td></tr><tr><td><code>DLL_PROCESS_DETACH</code> (<code>1</code>)</td><td>The DLL is being unloaded from the virtual address space of the calling process because it was loaded unsuccessfully or the reference count has reached zero (the processes has either terminated or called <a href="https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-freelibrary"><code>FreeLibrary</code></a> one time for each time it called <a href="https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya"><code>LoadLibrary</code></a>)</td></tr><tr><td><code>DLL_THREAD_ATTACH</code> (<code>2</code>)</td><td>The current process is creating a new thread. When this occurs, the system calls the entry-point function of all DLLs currently attached to the process. The call is made in the context of the new thread.</td></tr><tr><td><code>DLL_THREAD_DETACH</code> (<code>3</code>)</td><td>A thread is exiting cleanly. If the DLL has stored a pointer to allocated memory in a TLS slot, it should use this opportunity to free the memory. The system calls the entry-point function of all currently loaded DLLs with this value. The call is made in the context of the exiting thread.</td></tr></tbody></table>

#### Return Value

When the system calls the `DllMain` function with the `DLL_PROCESS_ATTACH` value, the function returns `TRUE` if it succeeds or `FALSE` if initialization fails.&#x20;

If the return value is `FALSE` when `DllMain` is called because the process uses the [`LoadLibrary`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya) function, `LoadLibrary` returns `NULL`. (The system immediately calls your entry-point function with `DLL_PROCESS_DETACH` and unloads the DLL.)&#x20;

If the return value is `FALSE` when `DllMain` is called during process initialization, the process terminates with an error.

When the system calls the `DllMain` function with any value other than `DLL_PROCESS_ATTACH`, the return value is ignored.
