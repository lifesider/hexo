title: 解决Linux下内联汇编的宏融合寻址问题
date: 2016-01-14 18:04:10
tags: Linux汇编寻址
categories: 程序艺术
---

Windows下生成DLL时，直接使用内联汇编的宏融合（Macro Fusion）模式，可以提高效率，如

    __asm {  
      
    　　movdqa　　　　xmm0, g_data0;  
      
    　　paddw 　　　　xmm0, g_data1;  
      
    　　movdqa　　　  g_data2, xmm0;  
      
    }  

 

这里的g_data0/g_data1/g_data2都是全局变量。

 

类似的代码，移植到Linux时，会遇到SIGSEGV问题，原因如下：

Linux下生成so库时，需要使用-fPIC编译选项，但PIC即位置无关代码却与Windows的Relocatable即可重定位表是冲突的。

Linux下如此修改则可：

    __asm {  
      
    　　lea　　　　　 eax, g_data0;  
      
    　　mov 　　　　  eax, [eax];  
      
    　　movdqa　　　　xmm0, [eax];  
      
    　　lea　　　　　 eax, g_data1;  
      
    　　mov 　　　　  eax, [eax];  
      
    　　movdqa　　　　xmm1, [eax];  
      
    　　paddw 　　　　xmm0, xmm1;  
      
    　　lea　　　　　 eax, g_data0;  
      
    　　mov 　　　　  eax, [eax];  
      
    　　movdqa　　　  [eax], xmm0;  
      
    }  


可见PIC模式会导致代码量增加，效率变慢，但其主要优势在于代码可以多进程共享的情况下占用更少的空间。