# Note of C++

### Function signature
Without parameter names, like this:

```c++
int foo(char*);
```



A function's **signature** includes the function's name and the number, order and type of its formal parameters.

Two overloaded functions must not have the same signature

The return value is **not** part of a function's signature

These two functions have the same signature

```c++
int Divide (int n, int m) ;
double Divide (int n, int m) ;
```

### Overloading Functions
[overloading functions URL](https://www.csee.umbc.edu/courses/undergraduate/202/spring07/Lectures/ChangSynopses/modules/m04-overload/slides.php?print)


### pragma once
Using `#pragma once` allows the [C preprocessor](https://en.wikipedia.org/wiki/C_preprocessor) to include a header file when it is needed and to ignore an `#include` directive otherwise. This has the effect of altering the behavior of the [C preprocessor](https://en.wikipedia.org/wiki/C_preprocessor) itself, and allows programmers to express file dependencies in a simple fashion, obviating the need for manual management.

https://en.wikipedia.org/wiki/Pragma_once


### strncmp
https://www.runoob.com/cprogramming/c-function-strncmp.html


### Argc and Argv
https://stackoverflow.com/questions/3024197/what-does-int-argc-char-argv-mean


### multi-line define
https://stackoverflow.com/questions/6281368/multi-line-define-directives


### assert
http://www.cplusplus.com/reference/cassert/assert/

我们可以在编译时指定assert是否crash进程


### extern
https://stackoverflow.com/questions/10422034/when-to-use-extern-in-c


### __thread
https://www.jianshu.com/p/997b533842c8

https://stackoverflow.com/questions/32245103/how-does-the-gcc-thread-work


### 存储类说明符
https://zh.cppreference.com/w/cpp/language/storage_duration


### 可变参数/变参宏
https://www.cnblogs.com/hanyonglu/archive/2011/05/07/2039916.html

https://blog.csdn.net/u012707739/article/details/80170671

[对于可变参数为空情形，Visual Studio直接去掉可变参数前面的逗号，GCC需要在\_\_VA_\_ARGS__前面放上##以去除逗号。](https://blog.csdn.net/fengbingchun/article/details/78483471)


### printf 合并字符串
https://blog.csdn.net/yanxiaolx/article/details/51531633


### fflush and fsync
https://stackoverflow.com/questions/2340610/difference-between-fflush-and-fsync


### typename and class
https://stackoverflow.com/questions/2023977/difference-of-keywords-typename-and-class-in-templates



### c有重载吗
c语言是没有重载的，重载需要编译器的支持，c++编译器会对函数名加前缀和后缀来修饰函数名，可以用readelf或者nm命令来查看编译后的目标文件。

c语言的目标文件，函数名是没有修饰的，而c++的函数名在目标文件中是加了前缀和后缀修饰过的。

关于函数签名和符号修饰可以阅读《程序员的自我修养》。



### c/c++区别
c++支持面向对象，继承，封装，多态。

c++面向对象，c面向过程。

c++数据和函数被封装在对象里。c的函数和数据是分开的。



### c++如何调用c
用extern "C"，

被它大括号括住的函数声明，将不会使用c++的符号修饰。详情见《程序员的自我修养》。

举个例子：

```c
// test2.h

int add(int, int);
```

```c
// test2.c

int add(int a, int b)

{

​	return a+b;

}
```

```c++
// test1.cpp

#include <test2.h>

int main()
{
	add(10, 20);
}
/*
这个时候你如果编译链接，是会失败的，会告诉你main函数里面使用了未定义的add，但是add不是已经命名被定义过了吗？
在test1.cpp里面，add会被符号修饰，可以gcc test1.cpp -c ，然后用nm命令查看test1.o里面的符号，发现add已经被修饰过了。
而我们的add的定义是在c文件里的，是c语言，c的编译是不会有符号修饰的。
我这里试验时修饰后的符号是"_Z3addii"，单独编译test2.c后add符号就是"add"，没有任何修饰，所以链接时，寻找"_Z3addii"的定义，是找不到的。
add 在test1.cpp里只是声明。
*/
```

```c++
// test1.cpp
extern "C"{
#include <test2.h>
}
int main()
{
	add(10, 20);
}
/*
这时，test1.cpp里的add声明被extern "C"括起来了，编译器就把它当做c代码处理，不会对其进行符号修饰。test1.o里的add符号就是"add"。
test2.o里的add也还是"add"，所以函数定义就能找到了。
*/
```



### 多态-what/why/how
多态的意思就是相同的实体，在不同场合下有不同的表现。对象/函数。

编译时：函数重载，运算符重载(也是函数重载的一种)

函数重载：虽然函数名相同，但是参数列表不同。

运算符重载：同理，虽然同是一个运算符，但参数列表不同。

运行时：虚函数



### 虚函数- what/why/how 
用父类指针指向子类对象，调用虚函数，调用的版本是子类重写的版本（如果子类重写了的话）。

多态的一种实现方式。



### 虚表(virtual table)
```c++
#include <stdio.h>
#include <stdlib.h>
#include <iostream>
using namespace std;
class A
{
   public :
   A(){};
   virtual ~A(){};
   virtual void func(){cout<<"A"<<endl;};
};

class B
{
   public :
   B(){};
   virtual ~B(){};
   virtual void func(){cout<<"B"<<endl;};
};

class C : A, B
{
   public :
   C(){};
   ~C(){};
};

int main(int argc, char const* argv[])
{
   cout<<sizeof(A)<<endl;
   cout<<sizeof(B)<<endl;
   cout<<sizeof(C)<<endl;
}
```
虚表内部是虚函数的地址，每个对象都包含一个或多个虚表指针。上述类C就包含两个虚表指针。


### 虚析构函数
https://blog.csdn.net/weicao1990/article/details/81911341

只有当一个类被用来作为基类的时候，才把析构函数写成虚函数。



### 纯虚函数
```c++
class A
{
  virtual void func1() = 0;
  virtual void func2() = 0;
};
class B : A

{
  public:
  B();
  ~B();
  virtual void func1(){};
  //virtual void func2(){};
};

#include <iostream>
using namespace std;
int main()
{
  B b;
  cout<<sizeof(A)<<endl;
}
/*
只要包含纯虚函数的类就是抽象类，抽象类是不能构造的，这里编译会报错。
如果B实现函数func2，那么B就不是抽象类了。
A的大小是8，因为包含虚表指针，即使A的虚函数都是纯虚函数。
*/
```


### 栈溢出/堆溢出
堆实际上是没办法溢出的，当我们申请的空间大于堆上限时，malloc直接返回空指针。

如果用new，则会抛出异常 std::bad_alloc。

```c++
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char const* argv[])
{
     int a[999999999999999];
     printf("%x\n", a);
}
/*
栈溢出
执行结果是Segmentation fault (core dumped)
*/
```

```c++
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char const* argv[])
{
     int *a = new int[99999999999];
     printf("%x\n", a);
}
/*
执行结果是
terminate called after throwing an instance of 'std::bad_alloc'
  what():  std::bad_alloc
Aborted (core dumped)
*/
```

```c++
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char const* argv[])
{
     int *a = (int*)malloc(99999999999);
     printf("%x\n", a);
}
/*
执行结果是0，malloc失败。
*/
```



### New and malloc
1.对于申请超过限制的空间，结果不同，new是抛出异常，malloc则是返回空指针。

2.new会调用对象的构造函数，而malloc只是单纯的分配空间。

3.new返回特定对象的指针，而malloc返回空指针，就是个地址，一般用的时候要强制转换一下。

4.new是运算符，而malloc只是个函数。

5.new一个对象的时候编译器帮我们计算所需空间，而malloc需要我们自己计算所需空间大小（一般用sizeof）。

6.new从自由存储区获取空间，malloc从堆获取空间（自由存储区和堆是不能划等号的！！！难点！！！）

7.malloc得到的指针可以通过realloc重新分配一块空间，而new不行（为什么？）。



### FREE STORE vs HEAP
自由存储区可以理解为一块空闲的区域，可以是堆，也可以是别的地方。
通过重载new操作符，可以自定义，从哪里获取空间。
就比方说，一个类的静态成员变量是一个数组，我们用这个数组来当作自由存储区，然后重载这个类的new，让new从这个数组里获取空间。
具体可以参考《c++编程思想》p326-p328.

### Const cast- what/why/how
```c++
void func(char * s){}
void func2(char * s){}
int main()
{
  func(const_cast<char*>("hello\n"));
  func2("hello\n");
}
```
就是为了防止这种，把一个字符串常量传入char\*类型的参数里，如果不用cast，则编译警告。项目里暂时只有这个用法。

### Static cast- what/why/how
static cast确保所有的转换都在编译时，没有运行时开销，而dynamic cast是运行时转换。
static 也可以用作向下转型。


### Reinterpret cast - what/why/how
公司项目几乎不用


### Dynamic cast - what/why/how
用的很少，一般是这样的，函数的参数是一个父类指针，函数体内需要把这个指针转成一个子类指针。


### smart pointer- what/why/how
公司项目里几乎不使用共享指针，仅仅是使用unique。
是这样用的，一个函数内new了一块区域，直接返回指针是不太好的，一般是用这个指针初始化一个unique ptr，然后返回这个智能指针。


### iterator- what/why/how
一个类，重载了取内容操作符，指针操作符，加加操作符等操作符，行为像指针。
有了迭代器，我们可以避免使用下标操作符访问容器，而且利于代码重用。
https://www.geeksforgeeks.org/introduction-iterators-c/


### 迭代器失效
https://www.cnblogs.com/wxquare/p/4699429.html

```c++
#include <vector>
#include <stdio.h>
using namespace std;
int main()
{
  vector<int> nums;
  nums.reserve(10);
  nums.push_back(1);
  nums.push_back(2);
  nums.push_back(3);
    
  vector<int>::iterator it = nums.begin() + 1;
  printf("%d, %p\n", *it, &*it);
  nums.insert(nums.begin(), 0);
  printf("%d, %p\n", *it, &*it);
  return 0;
}
/*
对于vector插入的描述，说容量不变时，只失效插入点之后的迭代器。其实也不能说失效，只是说插入点之后的迭代器，如果再访问的话，值可能就和第一次访问时的不一样了。
代码如上，我的it一开始指向数值2，然后再begin插入一个数值0，就是说，it还是指向原来的位置，但是原来的位置是前面的数值被挤过来的数值，1被挤过来了，所以再访问it的数值就是1。
如果vector的容量变了，那么之前迭代器指向的空间全部都是非法空间，如果非要之前的迭代器，也是可以的，但是只是理论可以，实际开发不能这样做。
*/
```
```c++
#include <vector>
#include <stdio.h>
using namespace std;
int main()
{
  vector<int> nums;
  nums.reserve(10);
  nums.push_back(1);
  nums.push_back(2);
  nums.push_back(3);
  
  vector<int>::iterator it = nums.begin() + 1;
  printf("%d, %p\n", *it, &*it);
  nums.erase(nums.begin() + 1);
  printf("%d, %p\n", *it, &*it);
  return 0;
}
/*
it一开始指向的下标是1值是2，然后nums删除下标为1的数，3就被向左挪过去了。所以最后it指向的值是3，it其实还是指向下标为1的数。
*/
```

### MACRO vs INLINE

宏定义是不能访问类成员的，内联函数可以。

内联函数可以指定参数类型，宏定义不行。

```c++
inline int func(int a, int b)
{
  return a + b;
}

#include <iostream>
#define ADD(x, y) ((x)+(y))

using namespace std;
int main()
{
  cout<<ADD(1, 1)<<endl;
  return 0;
}
/*
这里的宏定义ADD是没办法指定参数类型的
*/
```

普通的函数不能把类型作为参数，但是宏定义可以。

```c++
#define MALLOC(n, type) ((type *)malloc((n)* sizeof(type)))
#include <stdlib.h>
int main()
{
  int * p = MALLOC(10, int);
  return 0;
}
/*
c++可以很轻易地用模板来实现这个自定义的函数
但这里的自定义宏malloc很巧妙地实现了所需功能，把类型作为了参数
*/
```

### C中有模板吗
如上述的malloc，c可以用宏定义实现类似于模板的功能。
