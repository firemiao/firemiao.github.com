---
layout: post
title: Use __declspec(thread) in dll occured Exception
---
今天在写一个缓存的东西的时候，因为场景比较简单，特殊，所以决定借用TLS来处理数据的同步。  
刚开始用__declspec(thread)声明TLS变量，在WIN7下跑着很欢畅，但是在XP下一跑就挂鸟。  
DEBUG发现，声明的变量内存地址无效，也就是说系统没有为该TLS变量申请内存。GOOGLE没看到有用的信息，最后在微软的KB库中
发现如下一文:[PRB: Calling LoadLibrary() to Load a DLL That Has Static TLS](http://support.microsoft.com/kb/118816/en-us)

##Cause  
> This is a limitation of LoadLibrary and __declspec. The global variable space for a thread is allocated at run time. The size is based on a calculation of the requirements of the application plus the requirements of all of the libraries that are statically linked. If a DLL uses static TLS and is dynamic-linked in an application, when LoadLibrary or FreeLibrary is called, the system must find all the threads that exist in the process and enlarge or compact their TLS memory according to the size of static TLS in the newly loaded DLL. This process is too much for operating systems to manage, which can cause an exception either when the DLL is dynamically loaded or code references the data.  


##Resolutions  
>DLLs that use \_\_declspec(thread) should not be loaded with LoadLibrary.
The DLL code should be modified to use such TLS functions as TlsAlloc, and to allocate TLS if the DLL might be loaded with LoadLibrary. Or, the DLL that is using __declspec(thread) should only be implicitly loaded into the application.
