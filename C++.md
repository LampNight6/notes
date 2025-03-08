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



#### 什么是智能指针？有哪些种类？

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





































