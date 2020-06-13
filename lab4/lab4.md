# 实验任务

* 阅读VirtualAlloc、VirtualFree、VirtualProtect等函数的官方文档
* 编程使用malloc分配一段内存，测试是否这段内存所在的整个4KB都可以写入读取
* 使用VirtualAlloc分配一段，可读可写的内存，写入内存，然后将这段内存改为只读，再读数据和写数据，看是否会有异常情况。然后VirtualFree这段内存，再测试对这段内存的读写释放正常。
* 验证不同进程的相同的地址可以保存不同的数据。
  * （1）在VS中，设置固定基地址，编写两个不同可执行文件。同时运行这两个文件。然后使用调试器附加到两个程序的进程，查看内存，看两个程序是否使用了相同的内存地址；
  * （2）在不同的进程中，尝试使用VirtualAlloc分配一块相同地址的内存，写入不同的数据。再读出。
* （难度较高）配置一个Windbg双机内核调试环境，查阅Windbg的文档，了解
  * （1）Windbg如何在内核调试情况下看物理内存，也就是通过物理地址访问内存
  * （2）如何查看进程的虚拟内存分页表，在分页表中找到物理内存和虚拟内存的对应关系。然后通过Windbg的物理内存查看方式和虚拟内存的查看方式，看同一块物理内存中的数据情况。

# 实验准备

1. Windows系统中，通常是以4KB为单位进行管理的。也就是要么这4KB，都可用，要么都不可用。这样，所需要的管理数据就小得多。所以，malloc还不是最底层的内存管理方式。malloc我们称为堆内存管理。malloc可以分配任意大小的数据，但是，malloc并不管理一块数据是否有效的问题。而是由更底层的虚拟内存管理来进行的。
2. 分配内存是用 VirtualAlloc，释放使用VirtualFree，修改属性使用 VirtualProtec大家记住这三个函数。只要是VirtualAlloc分配的内存，就可以使用。VirtualAlloc甚至可以指定希望将内存分配在哪个地址上。malloc函数底层也会调用VirtualAlloc函数。当没有足够的整页的的内存可用时，malloc会调用VirtualAlloc，所以，实际的内存分配，没有那么频繁。
3. 当malloc在内存分配时，如果已经可用的分页中，还有剩余的空间足够用，那么malloc就在这个可用的分页中拿出需要的内存空间，返回地址。如果已经可用的分页不够用，再去分配新的分页。然后返回可用的地址。所以，malloc分配可以比较灵活，但是系统内部，不会把内存搞得特别细碎。都是分块的。
4. 由于我们直接调试的操作系统内核，所以需要两台计算机安装两个Windows，然后连个计算机使用串口进行链接。好在我们有虚拟机。所以我们需要再虚拟机中安装一个Windows（安装镜像自己找，XP就可以），然后通过虚拟串口和host pipe链接的方式，让被调试系统和windbg链接，windbg可以调试。使用Windbg  内核调试 VirtualBox 关键字搜索，能找到很多教程。如果觉得Windows虚拟机太重量级了，可以用Linux虚拟机+gdb也能进行相关的实验，以gdb 远程内核调试 为关键字搜索，也能找到很多教程。

# 实验过程

## PART I

> https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect
>
> https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualfree
>
> https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc

* MALLOC实验代码

  ```c
  #include <stdio.h>  /* printf, scanf, NULL */
  #include <stdlib.h>  /* malloc, free, rand, system */
  int main()
  {
      int test = 0;
      char* a = (char*)malloc(10);
      for (int i = 0; i < 4096; i++)a[i] = 'a';
      for (int i = 0; i < 4096; i++)
          if (a[i] == 'a')
              test++;
      return 0;
  }
  ```

  编程使用malloc分配一段内存，测试是否这段内存所在的整个4KB都可以写入读取.

  c语言中char型占1字节，4KB是4096个字节。test的值为4096，可知4KB空间都可以进行读取

<img src="pic\1.png" style="zoom:80%;" />

* VirtualAlloc实验代码

  使用VirtualAlloc分配一段，可读可写的内存，写入内存

  【由 VirtualAlloc 分配的 内存（可读可写）内存可以正常的写入和读取】

  ```c
  #include<iostream>
  #include<windows.h>
  #include<stdio.h>
  #include<stdlib.h>
  #include<time.h>
  
  using namespace std;
  int main() {
  	LPVOID pV;
  	pV = VirtualAlloc(NULL, 1000 * 1024 * 1024, MEM_RESERVE | MEM_TOP_DOWN, PAGE_READWRITE);
  	if (pV == NULL)
  		cout << "没有足够虚拟空间" << endl;
  	MEMORYSTATUS memStatusVirtual1;
  	GlobalMemoryStatus(&memStatusVirtual1);
  	cout << "虚拟内存分配：" << endl;
  	printf("指针地址=%x/n", pV);
  	
  }
  ```

  <img src="pic\2.png" style="zoom:80%;" />

将这段内存改为只读，再读数据和写数据，看是否会有异常情况

【将访问属性修改为 PAGE_READONLY 后，该段内存无法写入，但可以正常读取】

```c
vp = VirtualProtect(
        lVirtualProtect (PVOID 基地址，SIZE_T 大小，DWORD 新属性，DWORD 旧属性)
        PAGELIMIT * dwPageSize,	// 需要改变访问属性的区域大小
        PAGE_READONLY,		// 只读
        &oldProtect	// 在修改前，旧的访问属性将被存储
    );
```

更改一页的页面属性，改为只读后无法访问，还原后可以访问

```c
DWORD protect;
            iP[0]=8;
            VirtualProtect(pV,4096,PAGE_READONLY,&protect);
            int * iP=(int*)pV;
		iP[1024]=9;//可以访问，因为在那一页之外
            //iP[0]=9;不可以访问，只读
            //还原保护属性
            VirtualProtect(pV,4096,PAGE_READWRITE,&protect);
   		cout<<"初始值="<<iP[0]<<endl;//可以访问
```

释放内存代码【内存释放后将无法读取和写入】

```c
 //释放物理内存
            VirtualFree((int*)pV+2000,50*1024*1024,MEM_DECOMMIT);
            int* a=(int*)pV;
            a[10]=2;
            MEMORYSTATUS memStatusVirtual3;
            GlobalMemoryStatus(&memStatusVirtual3);
            cout<<"物理内存释放："<<endl;
cout<<"增加物理内存="<<memStatusVirtual3.dwAvailPhys-memStatusVirtual2.dwAvailPhys<<endl;
cout<<"增加可用页文件="<<memStatusVirtual3.dwAvailPageFile-memStatusVirtual2.dwAvailPageFile<<endl;
   cout<<"增加可用进程空间="
<<memStatusVirtual3.dwAvailVirtual-memStatusVirtual2.dwAvailVirtual<<endl<<endl;
```

## PART II

验证不同进程的相同的地址可以保存不同的数据。

* （1）在VS中，设置固定基地址，编写两个不同可执行文件。同时运行这两个文件。然后使用调试器附加到两个程序的进程，查看内存，看两个程序是否使用了相同的内存地址；
* （2）在不同的进程中，尝试使用VirtualAlloc分配一块相同地址的内存，写入不同的数据。再读出。

```c
//lab4-1
#include<stdio.h>

int main()
{
	printf("test 1");
}
//lab4-2
#include<stdio.h>

int main()
{
	printf("test 2");
}
```

* 修改固定基地址（两个文件都要修改，此处仅以2为例）

<img src="pic\3.png" style="zoom:80%;" />

* 执行上述两段代码，查看结果，发现两个程序的地址完全相同，但是正常执行，可见不同进程的相同的地址可以保存不同的数据

<img src="pic\4.png" style="zoom:80%;" />

<img src="pic\5.png" style="zoom:80%;" />

- 1，2两个工程将固定基址选项取消，仍回到随机分配基地址状态。同时执行以下代码，本来应该发现地址完全相同，由此可见系统的内存保护。但是我的结果不成功啊？

  ```c
  #include <windows.h>
  #include<stdio.h>
  void main()
  {
  
      SYSTEM_INFO sSysInfo;	// Useful information about the system
      GetSystemInfo(&sSysInfo);
      DWORD dwPageSize = sSysInfo.dwPageSize;
  
      //分配内存，标记为提交、可读可写
      LPVOID lpvBase = VirtualAlloc(
          (LPVOID)0x30000000,	// The starting address of the region to allocate
          dwPageSize,
          MEM_COMMIT | MEM_RESERVE,
          PAGE_READWRITE);
      if (lpvBase == NULL)
          return;
  
      LPTSTR ustr;
      ustr = (LPTSTR)lpvBase;
    
      for (DWORD i = 0; i < dwPageSize; i++)
      {
          ustr[i] = '1'; //test 1用1 ，test2用2
          printf("%c", ustr[i]);
      }
      return;
  
  }
  ```

- 两个程序的结果如下

<img src="pic\6.png" style="zoom:80%;" />

<img src="pic\7.png" style="zoom:80%;" />

***



* 配置一个Windbg双机内核调试环境，查阅Windbg的文档，了解（1）Windbg如何在内核调试情况下看物理内存，也就是通过物理地址访问内存（2）如何查看进程的虚拟内存分页表，在分页表中找到物理内存和虚拟内存的对应关系。然后通过Windbg的物理内存查看方式和虚拟内存的查看方式，看同一块物理内存中的数据情况。
* 虚拟机安装windows7系统，主机上安装好[windbg](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)
* 为虚拟机配置虚拟串口，为建立 HOST 到 GUEST 的调试通信连接

# 实验要点

## 内存管理

* 以4KB（页）作为基本管理单元的虚拟内存管理。
* 虚拟内存管理是一套虚拟地址和物理地址对应的机制。
* 程序访问的内存都是虚拟内存地址，由CPU自动根据系统内核区中的地址对应关系表（分页表）来进行虚拟内存和物理内存地址的对应。
* 每个进程都有一个分页表。
* 每个进程都有一个完整的虚拟内存地址空间，x86情况下为4GB（0x00000000-0xffffffff）
* 但不是每个地址都可以使用（虚拟内存地址没有对应的物理内存）
* 使用VirtualAlloc API可以分配虚拟内存（以页为单位）、使用VirtualFree释放内存分页。
* 使用VirtualProtect 修改内存也保护属性（可读可写可执行）
* 数据执行保护（DEP）的基本原理
* malloc和free等C函数（也包括HeapAlloc和HeapFree等）管理的是堆内存，堆内存区只是全部内存区的一个部分。
* 堆内存管理是建立在虚拟内存管理的机制上的二次分配。
* 真正的地址有效还是无效是以分页为单位的。
* 内存分页可以直接映射到磁盘文件（FileMapping）、系统内核有内存分页是映射物理内存还是映射磁盘文件的内存交换机制。
* 完成内存分页管理的相关实验