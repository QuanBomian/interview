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

 #### C++多态与虚函数

##### C++多态的实现

C++有静态多态和动态多态两种多态，静态多态通过函数重载和函数模板来实现，动态多态通过虚函数来实现

它需要一个指向基类的指针或者基类的引用和虚函数

通过指针和引用来调用虚函数的同时，会查找虚函数表的指针，找到虚函数表的函数指针调用

C++对象模型中为每个类都维护了一个虚函数表，当继承关系中有虚函数的覆盖时，会把继承下来的虚函数表中相应的函数指针替换为当前类中的虚函数

##### 存在虚函数的类中析构函数要定义成虚函数

因为存在虚函数代表要使用多态，基类指针用来管理派生类资源，使用delete来销毁对象时，会造成只调用基类部分的析构函数，而派生类的析构函数没有调用，造成了资源泄露

#### 析构函数能抛出异常吗

不建议在析构函数中抛出异常，如果析构函数没有显示声明或说明异常规范，则默认为noexcept(true)，如果析构函数内产生的异常没有在函数体内捕获，则会直接std::terminate。

程序在运行期间产生了异常，会一直向上寻找内处理异常的catch，并且要释放资源，如果在释放资源的同时在产生异常并抛出异常则无法处理资源的泄露，而且前一个异常没有处理完有来了一个异常，程序就会崩溃

#### 构造函数和析构函数中调用虚函数吗？

可以调用虚函数，但不会有效果。执行一个派生类的构造函数时。构造函数和析构函数中的虚函数会去虚拟化，将会总是被静态地当做本类型的函数解析。

#### 指针和引用的区别

1. 指针是对象的地址，而引用是一个对象的别名

2. 指针可以不不初始化，引用必须进行初始化

3. 指针可以指向空，而引用不能为空引用，用方法可以产生空引用

   ```C++
   T& NullRef()
   {
   	return *static_cast<T*>(nullptr);
   }
   ```

   

4. 指针具有顶层const和底层const两种用法，而引用只具有底层const熟悉，因为引用一经绑定就无法改变，不必有顶层const

5. 指针相较引用更为灵活，而引用更为安全

#### 指针和数组   

指针是一种复合类型，他引用每种类型的对象

数组是相同元素的集合

两者是完全不同的类型

数组名可以在某些情况下退化成指针常量

作为函数参数时会退化 



#### 智能指针

1. 我自己实现的智能指针

```C++
template<typename T>
class smart_ptr
{
public:
	int getCount()
	{
		return *count;
	}
	smart_ptr(T* p):count{new int(1)},object(p){}
	smart_ptr(const smart_ptr& other)
	{
		count = other.count;
		object = other.object;
		++(*count);
	}
	smart_ptr& operator=(const smart_ptr& other)
	{
		if(this != &other)
		{
			(*other.count)++;
			if(*--count == 0)
			{
				delete object;
				delete count;
			}
			count = other.count;
			object = other.object;
		}
		return *this;
	}
	~smart_ptr()
	{
		if(*--count == 0)
		{
			delete object;
			delete count;
			object = nullptr;
			count = nullptr;
		}
	}

private:
	T* object = nullptr;
	int* count = nullptr;
};
```

2. weak_ptr和shared_ptr

weak_ptr是sp的弱引用版本，不负责管理资源的生存期



#### 4种类型转换

作者：醉一场红尘

链接：https://zhuanlan.zhihu.com/p/54228612

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

1、static_cast:可以实现C++中内置基本数据类型之间的相互转换，enum、struct、 int、char、float等。它不能进行无关类型(如非基类和子类)指针之间的转换。

int c=static_cast<int>(7.987);

如果涉及到类的话，static_cast只能在有**相互联系的类型**中进行相互转换,不一定包含虚函数。

 2、const_cast: const_cast操作不能在不同的种类间转换。相反，它仅仅把一个它作用的表达式转换成常量。它可以使一个本来不是const类型的数据转换成const类型的，或者把const属性去掉。
 3、reinterpret_cast: （interpret是解释的意思，reinterpret即为重新解释，此标识符的意思即为数据的二进制形式重新解释，但是不改变其值。）有着和C风格的强制转换同样的能力。它可以转化任何内置的数据类型为其他任何的数据类型，也可以转化任何指针类型为其他的类型。它甚至可以转化内置的数据类型为指针，无须考虑类型安全或者常量的情形。不到万不得已绝对不用。
 4、dynamic_cast: 
（1）其他三种都是编译时完成的，dynamic_cast是运行时处理的，运行时要进行类型检查。
（2）不能用于内置的基本数据类型的强制转换。
（3）dynamic_cast转换如果成功的话返回的是指向类的指针或引用，转换失败的话则会返回NULL。
（4）使用dynamic_cast进行转换的，基类中一定要有虚函数，否则编译不通过。
  需要检测有虚函数的原因：类中存在虚函数，就说明它有想要让基类指针或引用指向派生类对象的情况，此时转换才有意义。 
        这是由于运行时类型检查需要运行时类型信息，而这个信息存储在类的虚函数表（关于虚函数表的概念，详细可见<Inside c++ object model>）中，
        只有定义了虚函数的类才有虚函数表。
（5） 在类的转换时，在类层次间进行上行转换时，dynamic_cast和static_cast的效果是一样的。在进行下行转换 时，dynamic_cast具有类型检查的功能，比               static_cast更安全。向上转换即为指向子类对象的向下转换，即将父类指针转化子类指针。向下转换的成功与否还与将要转换的类型有关，即要转换的指针指向的对象的实际类型与转换以后的对象类型一定要相同，否则转换失败。

#### 内存对齐

每个平台的编译器都有自己的默认“对齐系数”，gcc默认为#pragma pack(4)，msvc默认为8

可以通过#pragma pack更改

有效对齐值：默认对齐系数和结构体中最长数据类型长度中小的那个，也称对齐单位

规则：

(1) 结构体的第一个成员的offset为0，其他成员的offset为该**结构体的对齐单位和该成员长度中较小的那个**的倍数

(2) 结构体总大小为对齐单位的倍数



#### 内联函数有什么优点？内联函数与宏定义的区别？

1. 宏在预编译期展开，内联函数可能会在编译器展开在函数调用的地方，inline关键词只是建议编译器做内联展开。
2. 宏可以在当前作用域生成变量，内联函数则不能
3. 宏定义无法检查参数类型，它只进行符号的替换。内联函数是函数，所以会进行类型检查
4. 过多的内联代码会使得原本可以存储在指令缓存的指令分散，降低了缓存的命中率。
5. inline会导致代码提及膨胀，使得库的体积变大，造成额外的换页行为，进而可能会导致数据缓存的命中率降低。
6. inline可以减少调用的开销，如上下文的保存与恢复
7. inline允许编译器进一步的优化

#### C++内存

1. 堆区、栈区、全局区(.data)、常量区(.rodata)

   堆区存放动态分配的内存，调用malloc分配内存

   栈区 函数的参数，局部变量

   常量区 存放常量 const  int   "12345"等

   全局区 已经初始化或未初始化的全局变量或局部静态变量

#### STL里的内存池实现

- STL内存分配分为一级分配器和二级分配器，一级分配器就是采用malloc分配内存，二级分配器采用内存池。

二级分配器设计的非常巧妙，分别给8k，16k,..., 128k等比较小的内存片都维持一个空闲链表，每个链表的头节点由一个数组来维护。需要分配内存时从合适大小的链表中取一块下来。假设需要分配一块10K的内存，那么就找到最小的大于等于10k的块，也就是16K，从16K的空闲链表里取出一个用于分配。释放该块内存时，将内存节点归还给链表。
如果要分配的内存大于128K则直接调用一级分配器。
为了节省维持链表的开销，采用了一个union结构体，分配器使用union里的next指针来指向下一个节点，而用户则使用union的空指针来表示该节点的地址。

作者：牛客网

链接：https://zhuanlan.zhihu.com/p/30498004

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

#### 定位内存泄露

1. Windows平台下可以使用crtdbg.h头文件

```C++
#define _CRTDBG_MAP_ALLOC
#include<stdlib.h>
#include<crtdbg.h>
#include<stdio.h>
#include<iostream>
void GetMemory(char *p, int num)
{
	p = (char*)malloc(sizeof(char) * num);
}

int main(int argc, char** argv)
{
	char *str = NULL;
	GetMemory(str, 100);
	std::cout << "Memory leak test!" << std::endl;
	_CrtDumpMemoryLeaks();
	getchar();
	return 0;
}
```

_CrtDumpMemoryLeaks函数可以在Debug的output窗口输出泄露信息

```C++
#define _CRTDBG_MAP_ALLOC
#include<stdlib.h>
#include<crtdbg.h>
#include<stdio.h>
#include<iostream>
_CrtMemState s1, s2, s3;
void GetMemory(char *p, int num)
{
	p = (char*)malloc(sizeof(char) * num);
}

int main(int argc, char** argv)
{
	char *str = NULL;
	_CrtMemCheckpoint(&s1);
	GetMemory(str, 100);
	_CrtMemCheckpoint(&s2);
	std::cout << "Memory leak test!" << std::endl;
	if(_CrtMemDifference(&s3,&s1,&s2))
	{
		_CrtMemDumpStatistics(&s3);
	}
	_CrtDumpMemoryLeaks();
	getchar();
	return 0;
}
```

可以使用_CrtMemCheckpoint生成内存的快照



2. Linux下使用mtrace

```
MALLOC_TRACE=/home/YourUserName/path/to/program/MallocTraceOutputFile.txt 
export MALLOC_TRACE
```

```C++
#include <mcheck.h>

mtrace();

//memory allocation and free

muntrace()
```

```
mtrace <exec_file_name> <malloc_trace_filename>

#示例：
mtrace a.out MallocTraceOutputFile.txt
```

3. valgrind

```=
valgrind --leak-check = full ./a.out
```

####    strcpy实现

1. 不考虑内存重叠

```C++
char* strcpy(char* dst, const char* src){
    assert((src != NULL) && (dst !=NULL);
    char* result = dst;
    while((*dst++ = *src++) != '\0');
    return result;
    
}
```

[1] 检测传入的指针是否为空，使用assert说明合法性应该有用户保证

[2] result用来返回原始的dst的位置，用于链式表达式

[3] while用来copy并且最后在dst上添上'\0'后终结

2. 考虑内存重叠

```C++
char* my_strcpy(char* dst, const char* src) {
	assert((src != NULL) && (dst != NULL));
	char* result = dst;
	size_t size = strlen(src) + 1;
	if (dst > src && dst < src + size ) {
		dst = dst + size - 1;
		src = src + size - 1; //移动到'\0'处
		while (size--)
			*dst-- = *src--;
	}
	else {
		while (size--)
			*dst++ = *src++;
	}
	return result;
}
```

#### strcat函数实现

```C++
char* strcat(char* dst, const char* src){
    assert((src != NULL) && (dst != NULL));
    char* result = dst;
    while(*dst!= '\0')
        ++dst;
    while((*dst++ = *src++) != '\0');
    
    return result;
}
```

#### strcmp函数实现

```C++
int strcmp(const char* str1, const char* str2){
    while((*str1 != '\0') && (*str1 = * str2)){
        str1++;
        str2++;
    }
    if(*(unsigned char*)str1 > *(unsigned char*) str2)
        return 1;
    else if(*(unsigned char*)str1 < *(unsigned char*) str2)
        return -1;
    else 
        return 0;
    
}
```

