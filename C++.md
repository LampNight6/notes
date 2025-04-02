# C++



## 基础



### 指针和引用的区别

指针存放某个对象的地址，本身就是变量，本身就有地址，可以有指向指针的指针，指针本身可变

引用就是变量的别名，不可变，必须初始化

1、声明和使用

指针：

```c++
int x = 10;
int *p = &x;	//可以理解为*号代表这是变量是指针类型，int代表这个指针指向int类型的数据
int **pp = &p;	//指向指针的指针
cout << *p << endl;	//获取x的值
cout << p << endl;	//获取x的地址
```

引用：

```c++
int x =10;
int &ref = x;	//引用初始化
cout << ref << endl;
```

2、空对象与可变性

指针可以为空（nullptr），表示不指向任何有效地址，可改变指向的对象

引用不能为空，必须在声明时初始化，且不能改变引用的对象

3、用途

指针常用于动态内存分配、数组操作、函数参数传递

引用常用于函数参数传递、操作符重载、创建别名



### 数据类型

short至少16位

int至少与short一样长

long至少32位，至少与int一样长

long long至少64位，至少与long一样长

我的电脑：short（16位） int（32位） long（32位） long long（64位）

头文件<climits>定义了符号常量：INT_MAX 代表int的最大值，INT_MIN代表int的最小值



### 关键字



#### 指针常量与常量指针

指针常量：指针本身是常量，不能改变指针指向的地址，但是地址中保存的值可以改变

```c++
int a = 10;
int *const ptr = &a; // 指针常量：ptr 不能再指向其他变量

```

常量指针：指针指向的值是常量，不能修改指针指向的值，但是指针本身可以改变指向的地址

```c++
int a = 10;
const int *ptr = &a; // 常量指针，不能修改 *ptr 的值
int const *ptr = &a; // 作用同上，const 可以在 `int` 之前或之后
```



#### static关键字

用于控制变量和函数的生命周期，作用域、访问权限



1、静态局部变量

作用：变量只在当前函数可见，但值不会因函数的调用而销毁

生命周期：在第一次调用时创建，直达程序结束

示例：

```c++
#include <iostream>
using namespace std;

void counter() {
    static int count = 0;  // 只初始化一次，后续调用不会重新赋值
    count++;
    cout << "调用次数: " << count << endl;
}

int main() {
    counter();  // 第 1 次调用
    counter();  // 第 2 次调用
    counter();  // 第 3 次调用
    return 0;
}
```

结果：

```
调用次数: 1
调用次数: 2
调用次数: 3		//static int count = 0 只出始化一次，后续调用count仍然保留先前的值
```



2、静态全局变量

作用：使全局变量只在当前文件可见，防止命名冲突（外部extern无法访问）

生命周期：程序运行期间始终存在

示例：

```c++
// file1.cpp
#include <iostream>
using namespace std;

static int globalVar = 10;  // 仅 file1.cpp 可用

void printValue() {
    cout << "globalVar = " << globalVar << endl;
}

```

```c++
// file2.cpp
#include <iostream>
extern int globalVar;  // ❌ 错误，static 限制了作用域

int main() {
    cout << "globalVar = " << globalVar << endl;
    return 0;
}

```

结果：

```
undefined reference to `globalVar'	//globalVar 只能在 file1.cpp 访问，file2.cpp 无法访问它。
```



3、静态类成员变量

作用：静态成员变量属于整个类，而不是某个对象，所有对象共享同一份数据

生命周期：从程序开始到结束

初始化：必须在类外初始化

示例：

```c++
#include <iostream>
using namespace std;

class MyClass {
public:
    static int count;  // 静态成员变量，所有对象共享

    MyClass() { count++; }
};

int MyClass::count = 0;  // ✅ 必须在类外初始化

int main() {
    MyClass obj1, obj2;
    cout << "创建对象数量: " << MyClass::count << endl;
    return 0;
}

```

结果：

```
创建对象数量: 2	//count不属于某个对象，所有对象共享他
```



4、静态成员函数

作用：只能访问静态成员变量，不能访问非静态成员（没有this指针）

特点：可以用类名直接调用，不需要对象

示例：

```c++
#include <iostream>
using namespace std;

class MyClass {
public:
    static int count;

    static void showCount() {  // 静态函数
        cout << "当前计数: " << count << endl;
    }
};

int MyClass::count = 10;

int main() {
    MyClass::showCount();  // ✅ 直接用类名调用
    return 0;
}

```

结果：

```
当前计数: 10	//showCount() 不需要对象就能访问 count，适用于工具类、工厂模式等场景。
```



5、静态函数（文件作用域）

作用：限制函数作用域到当前文件，防止与其他文件的同名函数冲突

特点：只能在当前文件调用，不能被extern访问

示例：

```c++
// file1.cpp
#include <iostream>
using namespace std;

static void helperFunction() {  // 仅 file1.cpp 内可用
    cout << "这是一个静态函数" << endl;
}

void callHelper() {
    helperFunction();
}

```

```c++
// file2.cpp
extern void helperFunction();  // ❌ 错误，static 限制作用域

int main() {
    helperFunction();  // ❌ 错误，file2.cpp 无法访问 file1.cpp 的静态函数
    return 0;
}

```

static让heperFunction（）仅在file1.cpp中可用

总结：什么时候使用static？

| 场景               | 使用方式             |
| ------------------ | -------------------- |
| 记录函数调用次数   | static局部变量       |
| 限制全局变量作用域 | static全局变量       |
| 所有对象共享数据   | static类成员变量     |
| 不需要实例化的函数 | static类成员函数     |
| 避免函数命名冲突   | static限制函数作用域 |





#### const关键字

被const修饰的值不能改变，是只读变量。必须在定义时就给他赋初值



#### define和typedef的区别

define：

只是简单的字符串替换，没有类型检查

在编译的预处理阶段起作用

可以用来防止头文件重复引用

不分配内存，给出的是立即数，有多少次使用就进行多少次替换

typedef：

有对应的数据类型

是在编译、运行中起作用的

在静态存储区中分配空间，在程序运行过程中内存中只有一个拷贝



### 强制类型转换

关键字：static_cast、dynamic_cast、const_cast、reinterpret_cast

1、static_cast（静态转换）

适用于：

基本类型数据转换（int->double）

指针/引用类型转换（void※->int※）

向上转换（Upcasting）从子类指针转为父类指针



2、dynamic_cast（动态转换）

适用于：

只适用于多态类

用于运行时类型检查，安全的向下转换

转换失败时返回nullptr或抛出std::bad_cast异常



3、const_cast（去除const/volatile）

用于去掉const/ volatile修饰符



4、reinterpret_cast（重新解释转换）

适用于：

不同类型指针之间转换

在整数和指针之间转换

序列化、反序列化、低级别内存操作



## C++内存管理



#### 堆和栈的区别

栈是一种有限的内存区域，用于存储变量、函数调用信息等。堆是一种动态分配的内存区域，用于存储程序运行时动态分配的数据

栈上的变量生命周期与其所在函数的执行周期相同，而堆上的变量生命周期由程序员显式控制，可以使用new/malloc创建，delete/free释放

栈上的内存分配和释放是自动的，速度较快。堆上的内存分配和释放需要手动操作，速度较慢



#### C++内存分区

![image-20250308162539582](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250308162539582.png)

1、栈

栈用于存储函数的局部变量、函数参数和函数调用信息的区域。函数的调用和返回通过栈来管理

2、堆

堆用于存储动态分配的内存的区域，由程序员手动分配和释放。（new/malloc   ； delete/free）

3、全局/静态区

全局区存储全局变量和静态变量。生命周期是整个程序运行期间。在程序启动时分配，结束时释放

4、常量区

也被称为只读区。存储常量数据，如字符串常量

5、代码区

存储程序的代码



#### 内存泄漏？如何管理？

1、什么是内存泄漏？

由于疏忽或者错误导致程序未能释放掉不再使用的内存的情况。是指应用程序分配某段内存后，由于设计错误，失去对该内存的控制，造成内存浪费

可以使用valgrind或mtrace进行内存泄漏检查



2、内存泄漏的分类

（1）堆内存泄漏

使用malloc/new分配了一块内存后没有使用free/delete释放掉。那么此块内存将不会被再次使用，造成泄漏

（2）系统资源泄漏

主要指程序使用系统分配的资源如：bitmap，socket没有使用对应的函数释放掉，导致资源浪费

（3）没有将基类的析构函数定义为虚函数 

当基类指针指向⼦类对象时，如果基类的析构函数不是 virtual，那么⼦类的析构函数将不会被调⽤，⼦类的资源没
有正确是释放，因此造成内存泄露。



3、什么操作会导致内存泄漏？

指针指向改变，未释放动态分配内存



4、如何防止内存泄漏？

将内存的分配封装在类中，构造函数分配内存，析构函数释放内存；使用智能指针







#### new和malloc有什么区别？

类型安全性：

new是c++运算符，可以为对象分配内存并调用相应的构造函数

malloc是c语言库函数，只分配指定大小的内存块，不调用构造函数

返回类型：

new返回具体类型的指针，不需要类型转换

malloc返回void※，需要进行类型转换，因为他不知道分配内存的用途

内存分配失败时的行为：

new在内存分配失败时会抛出bad_alloc异常

malloc内存分配失败时返回NULL

内存块大小：

new用于动态分配数组，并知道数组大小

malloc只分配指定大小的内存块，不了解具体的用途

释放内存的方式：

delete会调用对象的析构函数并释放内存

free只会简单的释放内存，不会调用析构函数



#### delete和free有什么区别

类型安全性：

delete会调用对象的析构函数，确保资源被正确释放

free不了解对象的结构和析构，只是简单的释放内存块

内存块释放后的行为：

delete释放的内存块的指针会被设置为nullptr，避免野指针

free不会修改指针的值，可能导致野指针问题

数组的释放：

delete可以正确释放通过new[]分配的数组

free不了解数组的大小，不适用于分配malloc分配的数组



#### 什么是野指针，如何避免

野指针是指向已被释放或无效地址的指针，可能引起不可预测的行为

1、释放后没有置空指针

```c++
int* ptr = new int;
 delete ptr;
 // 此时 ptr 成为野指针，因为它仍然指向已经被释放的内存
ptr = nullptr; // 避免野指针，应该将指针置为 nullptr 或赋予新的有效地
```

2、返回局部变量的指针

```c++
int* createInt() {
 int x = 10;
 return &x; // x 是局部变量，函数结束后 x 被销毁，返回的指针成为野指针
}
 // 在使⽤返回值时可能引发未定义⾏为
```

解决方法：使用static关键字、使用new分配、使用智能指针

3、释放内存后没有调整指针

```c++
int* ptr = new int;
 // 使⽤ ptr 操作内存
delete ptr;
 // 此时 ptr 没有被置为 nullptr 或新的有效地址，成为野指针
// 避免：delete ptr; ptr = nullptr;
```

4、函数参数指针被释放

```c++
void foo(int* ptr) {
 // 操作 ptr
 delete ptr;	//在此处添加ptr = nullptr; 即可避免
 }
 int main() {
 int* ptr = new int;
 foo(ptr);
 // 在 foo 函数中 ptr 被释放，但在 main 函数中仍然可⽤，成为野指针
// 避免：在 foo 函数中不要释放调⽤⽅传递的指针
```



#### 什么是悬浮指针，如何避免

一个引用，指向了已经销毁的对象

1、返回局部变量的引用

```c++
int& getReference() {
    int x = 10;  // ❌ 局部变量
    return x;    // ⚠️ UB：x 在函数结束后被销毁
}

int main() {
    int& ref = getReference();  // ❌ ref 指向无效地址
    cout << ref << endl;  // ⚠️ UB：访问无效数据
    return 0;
}
```

解决方案：

```c++
static int x = 10;  // ✅ 使用静态变量
return x;
```

2、指向已删除对象的引用

```c++
int* p = new int(10);
int& ref = *p;  // ❌ 引用指向动态分配的对象
delete p;  // ❌ 释放内存
cout << ref << endl;  // ⚠️ UB：访问已删除的对象
```



#### 什么是内存对齐？为什么要考虑内存对齐？

1、什么是内存对齐：

内存对齐指的是CPU在访问内存时数据的起始地址应当是某个特定大小的整数倍，可以提高CPU访问速度，减少访问开销

示例：

```c++
struct A {
    char a;   // 1字节
    int b;    // 4字节
};
```

sizeof(A)并不是5字节，而是8字节，因为int需要4字节对齐

2、为什么要考虑内存对齐：

（1）CPU访问效率

CPU访问内存的方式是按块访问，如果跨过多个块则效率降低，对齐后只需要一次内存访问

（2）硬件架构需求

例如ARM必须按对齐方式访问，否则触发异常

x86允许非对齐访问，但效率下降

（3）避免数据错误

3、内存对齐规则

每个成员变量的地址必须是sizeof(类型）的整数倍

整个结构体大小必须是最大对齐数的整数倍（默认按最大成员对齐）

如果结构体内有子结构体，子结构体的地址也必须按对齐要求对齐



## 面向对象



#### 三大特性

1、继承：

子类可以使用父类的所有功能，并且在无需编写原来的类的情况下对这些功能进行扩展。

常见的三种继承方式：

实现继承：使用父类的属性和方法，无需额外编码

接口继承：仅使用父类方法属性和名称，子类需要实现这些方法



2、封装

数据和代码捆绑在一起，避免外界干扰和不确定性访问

功能：

把客观事物封装成抽象的类，并且只让可信的类访问自己的数据和方法



3、多态

允许相同的接口表现出不同的行为

（1）编译时多态（重载）

在编译时决定调用哪个函数

主要方式：

函数重载

运算符重载

模板

（2）运行时多态（重写）

在运行阶段决定调用哪个函数，通过虚函数实现

父类指针指向子类对象，调用子类重写的方法

ps：虚函数存在虚函数表中，调用父类方法时虚函数表中的指针会指向正确的子类方法



#### 访问修饰符

| 访问修饰符 | 类内部访问 | 子类访问 | 外部访问 |
| ---------- | ---------- | -------- | -------- |
| public     | √          | √        | √        |
| protected  | √          | √        | ×        |
| private    | √          | ×        | ×        |



#### 多重继承

一个子类可以同时继承多个父类的成员变量和成员函数

优点：

可以复用多个类的功能，减少代码重复

子类可以同时拥有多个父类的特性

缺点：

命名冲突：

如果A与B都有show()方法，那么C继承它们后调用show()会产生二义性，使用手动指定调用哪个父类的方法解决

菱形继承：

若B与C继承A，D同时继承B和C，产生数据冗余和二义性

使用虚继承（virtual）解决，使得A、B、C中只存在一个实例



#### 成员函数/成员变量/静态成员函数/静态成员变量的区别

1、成员函数

成员函数属于类的函数，可以访问类的成员变量和其他成员函数

成员函数可以普通成员函数和静态成员函数

普通成员函数使用对象调用，可以访问对象的成员变量

普通成员函数的声明和定义通常在类的内部，定义时需要使用类名作为限定符

2、成员变量

成员变量属于类的变量，存储在类的每一个对象中

每个对象拥有一份成员变量的副本，在创建对象时分配，对象销毁时释放

成员变量的访问权限可以是public、private、protected

```c++
class MyClass {
 public:
 int memberVariable;  // 成员变量的声明
void memberFunction() {
 // 成员函数的实现
    }
 };
```

3、静态成员函数

静态成员函数属于类而不是对象，因此可以直接通过类名调用，而不需要创建类的实例

静态成员函数不能直接访问普通成员变量，因为它们没有隐含的this指针

静态成员函数的声明在类的内部，定义时需要类名作为限定符

4、静态成员变量

静态成员变量属于类而不是对象的变量，它们在对象之间共享

静态成员变量通常在类的声明中声明，但是在类的定义外进行定义和初始化

静态成员变量可以通过类名或对象访问

```c++
class MyClass {
 public:
 static int staticMemberVariable;  // 静态成员变量的声明
static void staticMemberFunction() {
 // 静态成员函数的实现
    }
 };
 int MyClass::staticMemberVariable = 0;  // 静态成员变量的定义和初始化
```



#### 什么是构造函数和析构函数

1、构造函数

构造函数是在创建对象时自动调用的特殊成员函数。它的主要目的是初始化对象的成员变量，为对象分配资源，执行必要的初始化操作。特点：

（1）函数名与类名相同：构造函数的函数名必须与类名相同，没有返回类型，包括void

（2）可以有多个构造函数：一个类可以有多个构造函数，根据参数类型和数量重载

（3）默认构造函数：如果没有定义任何构造函数，编译器会生成一个默认构造函数，该函数没有参数，执行一些默认的初始化操作

```c++
class MyClass {
 public:
 // 默认构造函数
MyClass() {
 // 初始化操作
    }
 // 带参数的构造函数
MyClass(int value) {
 // 根据参数进⾏初始化操作
    }
 };
```

2、析构函数

析构函数是对象生命周期结束时自动调用的特殊成员函数。目的是释放资源、执行清理操作。特点：

（1）函数名与类名相同，前面加上~

（2）没有参数，不能重载

（3）默认析构函数：若没有定义析构函数，编译器自动生成一个默认析构函数，执行简单的清理操作

```c++
class MyClass {
 public:
 // 析构函数
~MyClass() {
 // 清理操作，释放资源
    }
 };
```



#### 构造函数的种类

1、默认构造函数：没有任何参数

2、带参数的构造函数：接受一个或多个参数

3、拷贝构造函数：用于通过已存在的对象创建一个新对象，新对象是原对象的副本。参数通常是同类型对象的引用

4、委托构造函数：在一个构造函数中调用同类的另一个构造函数，减少代码重复

```c++
class MyClass {
 public:
 // 委托构造函数
MyClass() : MyClass(42) {
 // 委托给带参数的构造函数
    }
 MyClass(int value) {
 // 进⾏初始化操作
    }
 };
```



#### 虚函数表

每一个类（抽象类）都包含一个虚函数表，包含了该类的虚函数地址。每个对象都包含一个指向该类的虚函数指针，称为虚指针(vptr)，调用一个虚函数时，编译器会查找虚函数表中的函数地址来执行相应的虚函数。



#### 纯虚函数

1、没有实现：纯虚函数没有函数体，只有函数声明，没有提供默认的实现

2、强制覆盖：子类必须提供纯虚函数的具体实现，否则也将称为抽象类

3、禁止实例化：包含纯虚函数的类无法被实例化，只能用于派生其他类

4、用=0声明

5、为接口提供规范：通过纯虚函数，抽象类提供一种接口规范，要求子类的相关实现

```c++
class AbstractBase {
 public:
 // 纯虚函数，没有具体实现
virtual void pureVirtualFunction() = 0;
 // 普通成员函数可以有具体实现
void commonFunction() {
 // 具体实现
    }
 };
```



#### 虚析构函数

目的：确保父类的析构函数被多态销毁

如果一个类可能被继承，并且会通过基类指针删除对象，就应当使用虚析构函数

如当我们使用 基类指针 指向 派生类对象 并 通过基类指针删除对象 时，如果基类的析构函数不是虚函数，可能会导致 内存泄漏 或 未定义行为。

示例：

```c++
#include <iostream>

class Base {
public:
    Base() { std::cout << "Base 构造函数\n"; }
    ~Base() { std::cout << "Base 析构函数\n"; }
};

class Derived : public Base {
public:
    Derived() { std::cout << "Derived 构造函数\n"; }
    ~Derived() { std::cout << "Derived 析构函数\n"; }
};

int main() {
    Base* ptr = new Derived();
    delete ptr; // 只调用了 Base 的析构函数，未调用 Derived 的析构函数！
    return 0;
}
```

输出：

```
Base 构造函数
Derived 构造函数
Base 析构函数  // 只调用了 Base 的析构函数，没有调用 Derived 的析构函数
```

正确写法：

```c++
#include <iostream>

class Base {
public:
    Base() { std::cout << "Base 构造函数\n"; }
    virtual ~Base() { std::cout << "Base 析构函数\n"; } // 虚析构函数
};

class Derived : public Base {
public:
    Derived() { std::cout << "Derived 构造函数\n"; }
    ~Derived() { std::cout << "Derived 析构函数\n"; }
};

int main() {
    Base* ptr = new Derived();
    delete ptr; // 现在可以正确调用派生类的析构函数
    return 0;
}
```

输出：

```
Base 构造函数
Derived 构造函数
Derived 析构函数
Base 析构函数
```



#### 为什么不能虚构造函数？

构造函数在对象的创建阶段就被调用，对象的类型在构造函数中已经被调用。因此，构造函数的调用不涉及多态性，也就是说，在对象的构造期间无法实现动态绑定。虚构函数没有意义，因为对象的类型在构造的过程中已经确定，不需要动态选择构造函数。

1. 从存储空间⻆度：虚函数对应⼀个vtale,这个表的地址是存储在对象的内存空间的。如果将构造函数设置为虚 函数，就需要到vtable 中调⽤，可是对象还没有实例化，没有内存空间分配，如何调⽤。（悖论） 
2. 从使⽤⻆度：虚函数主要⽤于在信息不全的情况下，能使重载的函数得到对应的调⽤。构造函数本身就是要初 始化实例，那使⽤虚函数也没有实际意义呀。所以构造函数没有必要是虚函数。虚函数的作⽤在于通过⽗类的 指针或者引⽤来调⽤它的时候能够变成调⽤⼦类的那个成员函数。⽽构造函数是在创建对象时⾃动调⽤的，不 可能通过⽗类的指针或者引⽤去调⽤，因此也就规定构造函数不能是虚函数。
3.  从实现上看，vbtl 在构造函数调⽤后才建⽴，因⽽构造函数不可能成为虚函数。从实际含义上看，在调⽤构造 函数时还不能确定对象的真实类型（因为⼦类会调⽗类的构造函数）；⽽且构造函数的作⽤是提供初始化，在 对象⽣命期只执⾏⼀次，不是对象的动态⾏为，也没有太⼤的必要成为虚函数。



#### 深拷贝与浅拷贝

浅拷贝：只复制对象的基本数据成员，对于指针和动态分配的内存，只复制指针地址而不复制实际数据（共享内存）

深拷贝：复制对象的所有数据，对于动态分配的内存，创建一个新的副本

| **特性**         | **浅拷贝（Shallow Copy）**             | **深拷贝（Deep Copy）**         |
| ---------------- | -------------------------------------- | ------------------------------- |
| **内存分配**     | 仅复制指针，多个对象共享同一块动态内存 | 每个对象都有自己的动态内存      |
| **拷贝构造函数** | 复制指针地址，不复制内容               | 重新分配新内存，并复制内容      |
| **优点**         | 速度快，内存占用小                     | 安全，不会因共享内存而崩溃      |
| **缺点**         | 可能导致 **野指针** 和 **双重释放**    | 额外的 **时间** 和 **内存开销** |



#### 运算符重载

运算符重载 是 C++ 提供的一种 让类对象也能使用运算符（如 +, -, *, = 等）的特性。它的本质是 重载运算符为类的成员函数或友元函数，让它们对自定义对象执行特定的操作。

C++ 内置的运算符 只能作用于 基本数据类型，但在自定义类中，我们可以让它们支持类对象操作。例如：

```c++
#include <iostream>

class Complex {
public:
    double real, imag;
    
    Complex(double r, double i) : real(r), imag(i) {}

    Complex operator+(const Complex& other) { // 运算符重载
        return Complex(real + other.real, imag + other.imag);
    }

    void show() { std::cout << real << " + " << imag << "i" << std::endl; }
};

int main() {
    Complex c1(1.2, 2.3);
    Complex c2(2.4, 3.6);
    Complex c3 = c1 + c2; // 通过重载的 `+` 进行对象加法
    c3.show();  // 输出 3.6 + 5.9i
}

```

为什么要重载 +？

默认情况下，+ 不能用于类对象，因为编译器不知道如何执行 两个对象的相加。运算符重载 使 c1 + c2 变成调用 operator+ 方法，返回新的 Complex 对象。



## STL



#### STL简介

STL（标准模板库）是C++标准库中的一个重要部分，提供了一系列通用容器、算法和迭代器，使C++具备强大的数据结构和算法支持

特点：

通用性：STL使用模板（template）设计，使其适用于任何数据类型

高效性：STL的实现接近底层，优化了性能，使用面向对象技和泛型编程技术

模块化：STL由六大核心组件构成，每个组件相互独立但可以互相配合

可移植性：A程序中使用STL编写的模块，可以直接移植到B上



#### STL组成

| 组件                     | 作用                           | 关键内容                                                     |
| ------------------------ | ------------------------------ | ------------------------------------------------------------ |
| **容器（Containers）**   | 存储和管理数据                 | `vector`, `list`, `deque`, `set`, `map`, `stack`, `queue` 等 |
| **算法（Algorithms）**   | 操作数据（排序、查找、修改等） | `sort`, `find`, `copy`, `for_each`, `accumulate` 等          |
| **迭代器（Iterators）**  | 连接容器和算法                 | `begin()`, `end()`, `rbegin()`, `rend()`, `advance()`, `next()` 等 |
| **函数对象（Functors）** | 作为参数传递给算法的可调用对象 | `greater<>`, `less<>`, `plus<>`, `multiplies<>` 等           |
| **适配器（Adapters）**   | 修改容器或函数对象的行为       | `stack`, `queue`, `priority_queue`, `bind`, `mem_fn`         |
| **分配器（Allocators）** | 管理底层内存分配               | `allocator<T>`                                               |





#### pair

pair 是 C++ 标准库（STL）中的一个 简单容器，用于 存储一对数据，两个数据可以是相同类型或不同类型。

定义：pair本质上是一个结构体，包含两个共有成员变量

```c++
template <class T1, class T2>
struct pair {
    T1 first;
    T2 second;
};
```

使用方法：

```c++
#include <iostream>
#include <utility>

int main() {
    std::pair<int, std::string> p1(1, "Apple"); // 使用构造函数
    std::pair<int, double> p2 = std::make_pair(2, 3.14); // 推荐用 make_pair

    std::cout << "p1: " << p1.first << ", " << p1.second << std::endl;
    std::cout << "p2: " << p2.first << ", " << p2.second << std::endl;
}
```

输出：

```
p1: 1, Apple
p2: 2, 3.14
```



#### vector容器的实现与扩充

`vector`是 C++ STL（标准模板库）中最常用的动态数组容器，支持 自动扩容、随机访问、动态大小调整，相比 `array` 和 `list`，`vector` 具有 更好的空间管理 和 访问速度。

1、vector的底层实现

vector本质上是一个动态数组，存储在连续的内存块中，具有以下成员：

_data：指向数组首地址的指针

_size：当前存储元素的个数

_capacity：当前分配的最大存储空间

2、基本操作

| **方法**            | **作用**                 |
| ------------------- | ------------------------ |
| `size()`            | 获取当前元素个数         |
| `capacity()`        | 获取当前分配的容量       |
| `push_back(x)`      | 添加元素，可能触发扩容   |
| `pop_back()`        | 删除最后一个元素         |
| `resize(n)`         | 重新设置大小，可能扩容   |
| `reserve(n)`        | 预分配内存，减少扩容次数 |
| `shrink_to_fit()`   | 释放多余容量             |
| `operator[]`        | 随机访问元素             |
| `begin()` / `end()` | 迭代器访问               |
| data()              | 动态数组首地址           |

3、扩容机制

在最开始capacity的大小为0，当插入第一个元素后将容量扩大到1，每次容量不足时，重新申请更大的空间（一般为两倍）。当初始化时规定了容量，则第一次capacity大小为规定的容量大小。

ps：数组的地址一般来说会稳定递增，但是有可能因为前面释放的大小足够支持下次扩充的容量，从而导致数组地址减小



#### list（链表）

`std::list` 是 C++ STL（标准模板库）中的一个 双向链表（Doubly Linked List） 容器，提供高效的 插入、删除 操作，但 不支持随机访问。

特点：

双向链表结构，可以双向遍历

插入/删除操作O(1)，不需要移动元素

不支持随机访问

底层实现：

`std::list` 采用 双向链表 存储数据，每个节点包含：

- 数据域（存储元素值）
- 前驱指针 `prev`（指向前一个节点）
- 后继指针 `next`（指向后一个节点）



#### deque（双端数组）

支持快速随机访问，由于deque需要处理内部跳转，速度上没有vector快

1、deque的存储结构

采用分段式存储，由多个固定大小的内存块组成。此外有一个指针数组用于管理这些块的地址，类似于索引表。另有头指针和尾指针标价首尾。

2、扩容策略

重新分配一个块，并更新指针数组（map array），调整back指针。当指针数组页满时进行两倍扩容。

3、访问速度

首先找到元素对应的块，再计算偏移量。因为多了一次间接寻址，因此deque的访问速度稍慢于vector



#### stack && queue

栈和队列以deque为底层架构，通过deque执行具体操作



#### heap && priority_queue

heap（堆）是一种数据结构，priority_queuq（优先级队列）是基于堆实现的STL容器，比heap更易用，提供push（），pop（）操作

|                | `heap`                           | `priority_queue`         |
| -------------- | -------------------------------- | ------------------------ |
| **实现**       | 适用于 `vector`                  | 独立容器                 |
| **默认类型**   | **最大堆**                       | **最大堆**               |
| **最小堆方式** | `std::greater<T>`                | `std::greater<T>`        |
| **访问堆顶**   | `vec.front()`                    | `pq.top()`               |
| **删除堆顶**   | `std::pop_heap() + pop_back()`   | `pq.pop()`               |
| **插入元素**   | `push_back() + std::push_heap()` | `pq.push()`              |
| **适用场景**   | **需要定制存储和排序的场景**     | **更易用，适合任务调度** |



#### map&&set的区别和实现原理

| 特性           | `map`（映射表）                                          | `set`（集合）                                            |
| -------------- | -------------------------------------------------------- | -------------------------------------------------------- |
| **存储结构**   | **键值对（Key-Value）**                                  | **单值集合（Key Only）**                                 |
| **键的唯一性** | **键（Key）唯一**                                        | **元素唯一**                                             |
| **值的访问**   | `map[key]` 获取 `value`                                  | 只能遍历 `set` 查找元素                                  |
| **底层实现**   | **红黑树 (`std::map`)，或哈希表 (`std::unordered_map`)** | **红黑树 (`std::set`)，或哈希表 (`std::unordered_set`)** |
| **适用场景**   | 需要**键值映射**                                         | 需要**存储唯一元素，并支持快速查找**                     |



#### 红黑树与哈希表的区别

红黑树：

一种自平衡的二叉搜索树，通过旋转和变色维持平衡。

哈希表：

一种基于数组+hash函数的数据结构

使用hash函数将key映射到数组索引

| 特性               | **红黑树 (`map/set`)**                     | **哈希表 (`unordered_map/unordered_set`)** |
| ------------------ | ------------------------------------------ | ------------------------------------------ |
| **底层数据结构**   | **红黑树（BST）**                          | **哈希表（Hash Table）**                   |
| **有序性**         | **有序（支持 `begin()` 递增遍历）**        | **无序**                                   |
| **插入时间复杂度** | **O(log n)**                               | **O(1)（平均），O(n)（最坏）**             |
| **删除时间复杂度** | **O(log n)**                               | **O(1)（平均），O(n)（最坏）**             |
| **查找时间复杂度** | **O(log n)**                               | **O(1)（平均），O(n)（最坏）**             |
| **范围查询**       | **支持（有序存储，可用 `lower_bound()`）** | **不支持（无序存储）**                     |
| **适合场景**       | **排序、范围查询**（如排名、区间统计）     | **快速查找**（如缓存、字典）               |

总结：红黑树的插入、删除、查询复杂度较稳定，适用于有序集合。哈希表的查询速度较快，适用于无序集合的快速查找



#### 哈希冲突

1、哈希表搜索的核心步骤

（1）使用哈希函数计算哈希值

（2）用哈希值%数组大小计算索引

（3）直接访问索引，如果没有哈希冲突，快速确定目标元素

（4）如果产生哈希冲突，遍历桶内的链表/数组

2、处理哈希冲突

（1）链地址法：每一个桶是一个链表，查找元素时需要遍历链表，最坏情况O(n)，一般情况O(1)

（2）开放寻址法：如果索引被占用，那么搜索下一个没有被占用的索引（线性探测/二次探测）

线性探测：(index + 1) % size

二次探测：(index + i^2) % size

（3）双哈希法：如果桶已满，那么使用另一个hash函数联合计算新的偏移量：(hash1(key) + i *hash2(key)) % size



#### push_back()和emplace_back()的区别

push_back()和emplace_back()都是在容器末尾添加元素的方法，但是底层机制和性能有一定差异

push：需要一个已经构造好的对象，再将其拷贝/移动到容器

emplace：直接在容器内构造对象

| **适用情况**                  | **`push_back()`**                       | **`emplace_back()`**                |
| ----------------------------- | --------------------------------------- | ----------------------------------- |
| **已有对象 `a` 需要存入容器** | ✅ **使用 `push_back(a)`**               | 🚫 不能 `emplace_back(a)`            |
| **直接传入值 `A(10)`**        | 🚫 先创建再拷贝/移动                     | ✅ **直接在容器内构造**              |
| **需要传递多个构造参数**      | 🚫 需先创建对象，再 `push_back(A(x, y))` | ✅ **直接传递 `emplace_back(x, y)`** |

总结：如果添加一个基础类型的元素，两种方法没有区别，如果添加复杂元素则emplace更优



## C++新特性



#### 智能指针

智能指针用于管理动态内存的对象，其主要目的是避免内存泄漏和方便资源管理

1、unique_ptr（独占智能指针）

不能复制

不能多个unique_ptr指向同一个对象

可以使用move移动所有权



2、shared_ptr（共享智能指针）

可以多个指针指向同一个对象

使用引用计数，最后一个shared_ptr被释放后，对象才被删除



3、weak_ptr（弱引用智能指针）

弱引用不会增加引用计数

用以解决shared_ptr的循环引用问题（a持有b，b也持有a）

不能直接访问对象，需要用.lock()获取shared_ptr



##### unique_ptr







#### 类型推导

一种自动推断变量或表达式类型的机制，减少手动指定类型需要，使代码更简洁、泛型更强大

1、auto（变量类型推断）

```c++
auto x = 42;            // x 被推导为 int
auto y = 3.14;          // y 被推导为 double
auto str = "hello";     // str 被推导为 const char* ，也就是字符的首字符地址，但是不能修改地址上的内容
```

auto不能用于：

```
auto a;  // ❌ 编译错误！必须初始化，否则无法推导类型
void func(auto x) {} // ❌ 错误，不能用于普通函数参数
```



2、decltype（表达式类型推导）

```c++
#include <iostream>

int main() {
    int x = 42;
    decltype(x) y = 3.14;  // y 具有和 x 一样的类型（int）

    decltype(42.5 + x) z;  // z 推导为 double

    std::cout << typeid(z).name() << std::endl; // 打印 z 的类型
    return 0;
}
```

3、auto与decltype配合

```c++
int x = 100;

auto getValue() { return x; }         // 返回 int（值传递）
decltype(auto) getValueRef() { return (x); } // 返回 int&（引用）

int main() {
    getValue() = 200;   // ❌ 修改的是拷贝，x 仍然是 100
    getValueRef() = 200; // ✅ 修改 x

    std::cout << "x: " << x << std::endl;
    return 0;
}

```



#### 右值引用



1、首先解释什么是**左右值**：

左值：在等号左侧的、生命周期较长的变量

右值：在等号右侧的、生命周期短的临时对象/表达式

简单来说就是，能使用 '&' 取地址的对象是左值，不能使用 '&' 取地址的是右值



2、左值引用

为了减少参数传递过程中的拷贝产生的花销

例子：

```c++
vector<int> grid(10, 0);
void fun(vector<int> &grid){}	//这里就是直接引用grid向量，减少传参复制grid向量的消耗
```

例外：const左值引用可以指向右值

```c++
const int &ref_a = 5;	//编译通过
```

const左值引用不会修改指向值，因此可以指向右值

实际例子：

```c++
void push_back(const value_type& val);
```

如果没有const，就不能使用vec.push_back(5);这样的代码了



3、右值引用

意义：减少右值参数传递过程中的拷贝花费，并且能够修改这些右值

例子：

```c++
int &&ref_a_right = 5;	//ok
int a = 5;
int &&ref_a_left = a;	//编译失败，右值不能指向左值
ref_a_right = 6;	//右值引用的用途，可以修改右值
```



4、右值能指向左值吗？

方法：std::move()

```c++
int a = 5; // a是个左值
int &ref_a_left = a; // 左值引用指向左值
int &&ref_a_right = std::move(a); // 通过std::move将左值转化为右值，可以被右值引用指向
 
cout << a; // 打印结果：5
```

move()的唯一功能是，将左值强制转换为右值，右值强制转换为左值



5、声明出来的左值引用、右值引用本身是左值还是右值？

被声明出来的左右值都是左值，因为被声明出来的左右值引用都是有地址的，也位于等号左侧

```c++
// 形参是个右值引用
void change(int&& right_value) {
    right_value = 8;
}
 
int main() {
    int a = 5; // a是个左值
    int &ref_a_left = a; // ref_a_left是个左值引用
    int &&ref_a_right = std::move(a); // ref_a_right是个右值引用
 
    change(a); // 编译不过，a是左值，change参数要求右值
    change(ref_a_left); // 编译不过，左值引用ref_a_left本身也是个左值
    change(ref_a_right); // 编译不过，右值引用ref_a_right本身也是个左值
     
    change(std::move(a)); // 编译通过
    change(std::move(ref_a_right)); // 编译通过
    change(std::move(ref_a_left)); // 编译通过
 
    change(5); // 当然可以直接接右值，编译通过
     
    cout << &a << ' ';
    cout << &ref_a_left << ' ';
    cout << &ref_a_right;
    // 打印这三个左值的地址，都是一样的
}
```

总结：

左右值引用没有区别，都能避免拷贝

右值引用可以直接指向右值，也可以通过std::move()指向左值，（const左值引用也能指向右值）

作为函数形参，右值引用更灵活（const左值引用无法修改值，有局限性）



#### 右值引用的应用场景

1、移动语义

主要应用在拷贝构造函数上

```c++
 Array(const Array& temp_array) {
        size_ = temp_array.size_;
        data_ = new int[size_];
        for (int i = 0; i < size_; i ++) {
            data_[i] = temp_array.data_[i];
        }
    }
     
    // 深拷贝赋值
    Array& operator=(const Array& temp_array) {
        delete[] data_;
 
        size_ = temp_array.size_;
        data_ = new int[size_];
        for (int i = 0; i < size_; i ++) {
            data_[i] = temp_array.data_[i];
        }
    }
```

正常的拷贝还是使用深拷贝。若拷贝完成后，原数组可以丢弃的话，可以使用浅拷贝，降低消耗

```c++
  Array(Array&& temp_array) {	//使用右值引用的浅拷贝
        data_ = temp_array.data_;
        size_ = temp_array.size_;
        // 为防止temp_array析构时delete data，提前置空其data_      
        temp_array.data_ = nullptr;
    }
```

实例：vector::push_back()使用std::move()提高性能

```c++
// 例2：std::vector和std::string的实际例子
int main() {
    std::string str1 = "aacasxs";
    std::vector<std::string> vec;
     
    vec.push_back(str1); // 传统方法，copy
    vec.push_back(std::move(str1)); // 调用移动语义的push_back方法，避免拷贝，str1会失去原有值，变成空字符串
    vec.emplace_back(std::move(str1)); // emplace_back效果相同，str1会失去原有值
    vec.emplace_back("axcsddcas"); // 当然可以直接接右值
}
 
// std::vector方法定义
void push_back (const value_type& val);
void push_back (value_type&& val);
 
void emplace_back (Args&&... args);
```



2、完美转发 std::forward()

forward的作用是做左右值类型转化

std::forward<T>(u)有两个参数：T与 u。 a. 当T为左值引用类型时，u将被转换为T类型的左值； b. 否则u将被转换为T类型右值。

```c++
void change2(int&& ref_r) {
    ref_r = 1;
}
 
void change3(int& ref_l) {
    ref_l = 1;
}
 
// change的入参是右值引用
// 有名字的右值引用是 左值，因此ref_r是左值
void change(int&& ref_r) {
    change2(ref_r);  // 错误，change2的入参是右值引用，需要接右值，ref_r是左值，编译失败
     
    change2(std::move(ref_r)); // ok，std::move把左值转为右值，编译通过
    change2(std::forward<int &&>(ref_r));  // ok，std::forward的T是右值引用类型(int &&)，符合条件b，因此u(ref_r)会被转换为右值，编译通过
     
    change3(ref_r); // ok，change3的入参是左值引用，需要接左值，ref_r是左值，编译通过
    change3(std::forward<int &>(ref_r)); // ok，std::forward的T是左值引用类型(int &)，符合条件a，因此u(ref_r)会被转换为左值，编译通过
    // 可见，forward可以把值转换为左值或者右值
}
 
int main() {
    int a = 5;
    change(std::move(a));
}
```



#### nullptr

nullptr相比于旧标准的NULL提供了更清晰安全的语义

传统标准中NULL的本质是0，作为空指针常量使用是可能被误认为整数，导致函数重载时产生歧义

而引入nullptr可以明确表示指针类型，nullptr能转化为任何类型的指针，而不会转化成整数



#### 范围for循环

基于范围的迭代写法，for表达式对于容器的每一个元素进行操作

```c++
for (元素类型 元素名 : 容器或范围) {
    // 循环体
}
```



#### 列表初始化

C++定义了⼏种初始化⽅式，例如对⼀个int变量 x初始化为0： 

```c++
int x = 0;      
int x = {0};    
int x{0};       
int x(0);       
```

采⽤花括号来进⾏初始化称为列表初始化，⽆论是初始化对象还是为对象赋新值。   

⽤于对内置类型变量时，如果使⽤列表初始化，且初始值存在丢失信息⻛险时，编译器会报错。

```c++
long double d = 3.1415926536; 
int a = {d};    //存在丢失信息⻛险，转换未执⾏。
int a = d;      //确实丢失信息，转换执⾏。
```



#### lambda表达式

lambda表达式是一种匿名函数，允许用户在代码中定义短小临时的匿名函数

表达式的一般形式：

```c++
[capture-list](parameters) -> return_type { function_body };
```

`[]`（捕获列表）：定义 lambda 表达式可访问的外部变量。

`()`（参数列表）：定义传递给 lambda 的参数。

`->`（可选的返回类型）：指定函数返回类型，通常自动推断。

`{}`（函数体）：函数具体执行的代码。

**1、捕获列表(可空)**

lambda在默认情况下不能访问外部变量，如果想访问，必须明确捕获

捕获列表的形式：

值捕获表示复制一个变量的值等于外部变量的值，引用捕获表示直接对外部变量进行操作

- `[=]`：值捕获所有外部变量。
- `[&]`：引用捕获所有外部变量。
- `[var]`：值捕获特定变量。
- `[&var]`：引用捕获特定变量。

混合捕获：

```c++
int a = 1, b = 2;

// 按值捕获a，引用捕获b
auto f = [a, &b]() {
    // a 是值捕获，不可修改外部a；b 是引用捕获，可修改
    std::cout << a << " " << ++b << std::endl;
};

f();  // 输出 1, b=2
std::cout << b; // b被修改，输出 2
```

mutable关键字（不常用）

用于取消捕获列表中参数的常属性，捕获列表中的参数一般默认为常数，不可改变

```c++
[]()mutable->returntype{}	//mutalbe的位置
```

所以说 `mutable` 关键字不常用，因为它取消的是值类型的常性，即使修改了，对外部也没有什么意义，如果想修改，直接使用引用捕捉 就好了 



**2、参数列表（可省略）**

定义lambda接受的参数，与普通函数的参数列表相同：

lambda调用时传递的参数

只能来自函数调用时显式传入

例如：

```c++
auto lambda = [](int x, int y) {
    return x + y;
};
lambda(3, 5); // 参数3和5传入 lambda
```



**3、lambda表达式的常见用法**

Lambda表达式在和STL标准库算法配合使用时特别方便，比如配合`std::sort`：

```c++
std::vector<int> nums = {5, 2, 8, 1, 3};

// 降序排序
std::sort(nums.begin(), nums.end(), [](int a, int b) { return a > b; });
```

作为函数对象或回调函数使用

```c++
#include <algorithm>

std::vector<int> v{1, 2, 3};
std::for_each(v.begin(), v.end(), [](int n){
    std::cout << n << " ";
});
// 输出: 1 2 3
```



### C++多线程

（简单情况下）C++多线程使用多个函数实现各自功能，然后将不同函数生成不同线程，并同时执行这些线程（不同线程可能存在一定程度的执行先后顺序，但总体上可以看做同时执行）。



#### 多线程基础

1、创建线程
（1）使用thread创建线程

```c++
#include <thread>
#include <iostream>

void worker() {
    std::cout << "线程执行中...\n";
}

int main() {
    std::thread t(worker); // 创建线程并运行worker函数
    t.join();              // 等待线程结束
}
```

（2）使用lambda

```c++
std::thread t([](){
    std::cout << "线程运行中" << std::endl;
});
t.join();
```

2、结束线程

- 当线程启动后，一定要在和线程相关联的thread销毁前，确定以何种方式等待线程执行结束。C++11有两种方式来等待线程结束：

detach方式：当前代码继续执行，线程自主在后台运行

join方式：等待线程执行完成后，当前代码再执行



#### 互斥量

互斥量是用于保证多个线程，互斥地使用临界资源(共享资源)的信号量。互斥量保证了同一时刻只被一个进程使用。

程序实例化mutex对象m,线程调用成员函数m.lock()会发生下面 3 种情况

如果该互斥量当前未上锁，则调用线程将该互斥量锁住，直到调用unlock()之前，该线程一直拥有该锁。

如果该互斥量当前被锁住，则调用线程被阻塞,直至该互斥量被解锁。

不推荐实直接去调用成员函数lock()，因为如果忘记unlock()，将导致锁无法释放，使用lock_guard或者unique_lock能避免忘记解锁这种问题。

```c++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;//实例化m对象，不要理解为定义变量
void proc1(int a)
{
    m.lock();
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
    m.unlock();
}
 
void proc2(int a)
{
    m.lock();
    cout << "proc2函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 1 << endl;
    m.unlock();
}
int main()
{
    int a = 0;
    thread proc1(proc1, a);
    thread proc2(proc2, a);
    proc1.join();
    proc2.join();
    return 0;
}
```

1、lock_guard

其原理（RAII一种编程范式）是：声明一个局部的lock_guard对象，在其构造函数中进行加锁，在其析构函数中进行解锁。加锁绑定于对象的创建，解锁绑定于对象的析构，利于栈自动回收临时变量的机制，通过使用{}来调整作用域范围（对象的生命周期），可使得互斥量m在合适的地方被解锁。

```c++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;//实例化m对象，不要理解为定义变量
void proc1(int a)
{
    m.lock();//手动锁定
    lock_guard<mutex> g1(m,adopt_lock);
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
}//自动解锁
 
void proc2(int a)
{
    lock_guard<mutex> g2(m);//自动锁定
    cout << "proc2函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 1 << endl;
}//自动解锁
int main()
{
    int a = 0;
    thread proc1(proc1, a);
    thread proc2(proc2, a);
    proc1.join();
    proc2.join();
    return 0;
}
```

2、unique_lock

unique_lock类似于lock_guard,只是unique_lock用法更加丰富，同时支持lock_guard()的原有功能。

使用lock_guard后不能手动lock()与手动unlock();使用unique_lock后可以手动lock()与手动unlock();

unique_lock的第二个参数，除了可以是adopt_lock,还可以是try_to_lock与defer_lock;

try_to_lock: 尝试去锁定，得保证锁处于unlock的状态,然后尝试现在能不能获得锁；尝试用mutx的lock()去锁定这个mutex，但如果没有锁定成功，会立即返回，不会阻塞在那里

defer_lock: 始化了一个没有加锁的mutex;

```c++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;
void proc1(int a)
{
    unique_lock<mutex> g1(m, defer_lock);//始化了一个没有加锁的mutex
    cout << "不拉不拉不拉" << endl;
    g1.lock();//手动加锁，注意，不是m.lock();注意，不是m.lock();注意，不是m.lock()
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
    g1.unlock();//临时解锁
    cout << "不拉不拉不拉"  << endl;
    g1.lock();
    cout << "不拉不拉不拉" << endl;
}//自动解锁
 
void proc2(int a)
{
    unique_lock<mutex> g2(m,try_to_lock);//尝试加锁，但如果没有锁定成功，会立即返回，不会阻塞在那里；
    cout << "proc2函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 1 << endl;
}//自动解锁
int main()
{
    int a = 0;
    thread proc1(proc1, a);
    thread proc2(proc2, a);
    proc1.join();
    proc2.join();
    return 0;
}
```



#### 原子操作

原子操作满足：

- 操作不可再拆分成更小的部分。
- 在多线程环境下，原子操作要么完整执行，要么完全不执行，不会产生“中间状态”。
- 可以避免数据竞争（data race），因此线程安全。

C++11通过atomic库实现原子操作

```c++
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> counter{0};

void increment() {
    for (int i = 0; i < 100000; ++i) {
        counter++; // 原子增加，不会造成数据竞争
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "counter = " << counter << std::endl; // 必然是200000
}
```

原子操作与互斥锁的区别：

| 区别         | `std::atomic` 原子操作 | 互斥锁 (`std::mutex`) |
| ------------ | ---------------------- | --------------------- |
| 性能         | 更高效                 | 相对较低              |
| 适用范围     | 简单数据类型、简单操作 | 较复杂的临界区操作    |
| 是否阻塞线程 | 非阻塞（无锁）         | 会阻塞线程等待锁      |
| 使用复杂性   | 简单                   | 复杂                  |

推荐在性能敏感、高并发、简单共享变量场景下优先使用原子操作。

对复杂数据结构、复杂临界区，仍需使用互斥锁（mutex）等机制。





































