# 实验任务

* 阅读VirtualAlloc、VirtualFree、VirtualProtect等函数的官方文档
* 编程使用malloc分配一段内存，测试是否这段内存所在的整个4KB都可以写入读取
* 使用VirtualAlloc分配一段，可读可写的内存，写入内存，然后将这段内存改为只读，再读数据和写数据，看是否会有异常情况。然后VirtualFree这段内存，再测试对这段内存的读写释放正常。

# 实验准备

1. Windows系统中，通常是以4KB为单位进行管理的。也就是要么这4KB，都可用，要么都不可用。这样，所需要的管理数据就小得多。所以，malloc还不是最底层的内存管理方式。malloc我们称为堆内存管理。malloc可以分配任意大小的数据，但是，malloc并不管理一块数据是否有效的问题。而是由更底层的虚拟内存管理来进行的。
2. 分配内存是用 VirtualAlloc，释放使用VirtualFree，修改属性使用 VirtualProtec大家记住这三个函数。只要是VirtualAlloc分配的内存，就可以使用。VirtualAlloc甚至可以指定希望将内存分配在哪个地址上。malloc函数底层也会调用VirtualAlloc函数。当没有足够的整页的的内存可用时，malloc会调用VirtualAlloc，所以，实际的内存分配，没有那么频繁。
3. 当malloc在内存分配时，如果已经可用的分页中，还有剩余的空间足够用，那么malloc就在这个可用的分页中拿出需要的内存空间，返回地址。如果已经可用的分页不够用，再去分配新的分页。然后返回可用的地址。所以，malloc分配可以比较灵活，但是系统内部，不会把内存搞得特别细碎。都是分块的。

# 实验过程

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

<img src="D:\软件与系统安全\lab4\pic\1.png" style="zoom:80%;" />

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

  <img src="D:\软件与系统安全\lab4\pic\2.png" style="zoom:80%;" />

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

