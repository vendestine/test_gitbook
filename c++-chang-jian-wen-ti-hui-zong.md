# C++ 常见问题汇总

### 1. 在main执行之前和之后执行的代码可能是什么？

**main之前可能会执行什么**：

首先一般认知来讲，main函数是程序的入口，这句话在开发的角度来讲是没有什么问题的，但是从操作系统的角度来看，它就不是很准确了。

拿gcc编译器来说，\_start函数才是真正的入口点，所以我们就从\_start函数来分析这个问题 —start函数伪代码如下

```
　　　　__start:
　　　　 :
　　　　 init stack;
　　　　 init heap;
　　　　 open stdin;
　　　　 open stdout;
　　　　 open stderr;
　　　　 :
　　　　 push argv;
　　　　 push argc;
　　　　 call _main; (调用 main)
　　　　 :
　　　　 destory heap;
　　　　 close stdin;
　　　　 close stdout;
　　　　 close stderr;
　　　　 :
　　　　 call __exit;
```

所以看的出来在main函数之前是初始化系统的相关资源，如栈，堆，IO，异常，全局静态对象等等 初始化资源的话大致的顺序就是

1. 设置栈（初始化栈指针）
2. 初始化静态static变量和global全局变量，(.data段的内容) 如果对象是非内置类型，就会调用构造函数
3. 将未初始化部分的全局变量赋初值，(.bss段的内容)
4. 然后就将main函数的参数argc，argv和envp压入栈中

**attribute**((constructor))关键字放在函数前，会将该函数修饰成构造函数，所以在也在main之前

\


**main之后可能会执行什么**：

* main函数执行完毕后，返回到入口函数，入口函数进行清理工作，包括全局变量的析构、堆销毁、关闭I/O等，然后系统调用结束进程

可以用 atexit 注册一个函数，或者给函数修饰\_\_attribute\_\_((destructor))，都可以在main函数后执行这个函数再结束进程。

\


**Reference:** [\[1\]. c/c++ main函数执行之前/后](https://blog.51cto.com/u\_15072903/3542475) [\[2\]. 搞定技术面试-在 main函数之前发生了什么](https://juejin.cn/post/6844904032004374541) [\[3\]. 在不使用 main 函数的情况下编写 C/C++ 程序](https://www.techiedelight.com/zh/c-program-without-main-function/) [\[4\]. 深入理解BSS段与data段的区别](https://www.jianshu.com/p/ddfb284c1f7a)

\


### 结构体内存问题

它有内存对齐的现象

1. 为什么要内存对齐 字节对齐主要是为了提高内存的访问效率，操作系统在访问内存的时候，每次读取一定长度，如果没有对齐，访问一个变量可能会产生二次访问的问题。
2. 内存对齐的原则 总结了一下公式： 公式1:前面的地址必须是后面的地址正数倍,不是就补齐 公式2:整个Struct的地址必须是最大字节的整数倍
3. 可以手动设置对齐方式 `#pragma pack(n)`

**Reference** [\[1\]. 如何理解 struct 的内存对齐？](https://www.zhihu.com/question/27862634) [\[2\]. 结构体字节对齐问题探究](https://blog.51cto.com/yang/4374041) [\[3\]. C语言 | 关于结构体内存对齐，看这篇就够了](https://cloud.tencent.com/developer/article/1703257)

\
