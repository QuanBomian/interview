## C++基础知识

#### static作用

1. 用于函数内部修饰变量，即函数内的静态变量。这种变量的生存期长于该函数，使得函数具有一定的“状态”。使用静态变量的函数一般是不可重入的，也不是线程安全的。
2. 用在文件级别（函数体之外），修饰变量或函数，表示该变量或函数只在本文件(tu)可见，其他文件(tu)看不到也访问不到该变量或函数。专业的说法叫“具有 internal linkage”（简言之：不暴露给别的 translation unit）。
3. 用于修饰 class 的数据成员，即所谓“静态成员”。这种数据成员的生存期大于class 的对象（实体 instance）。静态数据成员是每个 class 有一份，普通数据成员是每个 instance 有一份，因此也分别叫做 class variable 和 instance variable。
4. 用于修饰 class 的成员函数，即所谓“静态成员函数”。这种成员函数只能访问class variable 和其他静态程序函数，不能访问 instance variable 或 instance method。

#### extern作用

1. 用在symbol之前（函数和变量），声明存在这个变量，但可能不在当前的编译单元内，提示编译器到其他的TU内找。
2. extern “c”进行链接指定，说明声明的symbol具有C-linkage，使用C语言的name mangling来查找symbol

#### volatile作用

1. volatile保证每次对变量的访问都是访问内存，并且访问volatile之间的顺序是不会被优化改变(单线程，多线程下不保证)。c/ c++中的volatile应只用于外设驱动程序

2. volatile 不能解决多线程中的问题。按照 Hans Boehm & Nick Maclaren 的总结，volatile 只在三种场合下是合适的。和信号处理（signal handler）相关的场合；和内存映射硬件（memory mapped hardware）相关的场合；和非本地跳转（setjmp 和 longjmp）相关的场合。

3. const volatile

   const表示在当前程序中只读，volatile表示不能优化访存，但是外部操作可能修改，中断服务

#### new与malloc区别

1. new是操作符，malloc是库函数
2. new返回相应对象的指针，而malloc返回void*
3. new如果产生问题会std::bad_alloc，而malloc出问题会返回NULL
4. new会调用构造函数，malloc不会