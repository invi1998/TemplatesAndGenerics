# 元组（tuple）实现

## 重要基础知识回顾

### 左值、右值、左值引用、右值引用

```c++
namespace _nmsp1
{
    void tsFunc1(int& j)
    {}
    
    void tsFunc2(const int& j)
    {}
    
    void tsFunc3(int&& j)
    {}
    
    void tsFunc4(const int&& j)
    {}
    
    void tsFunc5(int j)
    {}
    
    void func()
    {
        // 左值。右值、左值引用，右值引用
        
        int i = 10;
        // i 是左值，10是右值
        
        // 然后左值引用，谈到引用，就必须有一个地址符（&）
        int& j = i;
        // j 就是一个左值引用，那么给这个左值引用赋值的时候（也就是等号右侧），必须也得是一个左值
        // int& j = 10; // 这个就不行，因为左值引用只能接左值，而10是右值，不能用一个右值给左值引用j赋值
        
        // 同理，当左值引用作为一个函数形参的时候，那么你传递的实参就得是一个左值
        tsFunc1(i);
        // tsFunc1(10); // 语法错，10 是右值，不可以作为实参传递给左值引用形参
        
        // 所以记住：左值引用只能绑定到左值上，编译器会且只会为const引用 （const &）开绿灯，可以让其绑定左值，也可以让其绑定右值
        // const例外情况（加上const，如下就没有语法错）
        
        const int& k = 10;
        tsFunc2(100);
        
        // 右值引用
        int&& l = 10;
        // l就是一个右值引用，给右值引用赋值的时候（也就是等号右侧），必须也得是一个右值
        // int&& l = i; // 这个就不行，因为右值引用只能接右值，i是左值，不能用一个左值给右值引用l赋值
        
        // 同理，当右值引用作为一个函数形参的时候，那么你传递的实参就得是一个右值
        tsFunc3(100);
        // tsFunc3(i);  // 语法错，i是左值，不可以作为实参传递给右值引用形参
        
        // 所以记住：右值引用只能绑定到右值上
        
        
        int m = i;
        // 这里m是一个左值引用还是右值引用？
        // 既不是左值引用，也不是右值引用，引用必须得有(&)地址符
        // 那既然他这里既不是左值，又不是右值，那么这里你无论给一个左值还是右值进行赋值，都不会报错。都是可以的
        // 展开了讲，如果你把这个m作为函数形参，那么你调用这个函数的时候，无论传递的是左值还是右值，都没有问题
        tsFunc5(12);
        tsFunc5(i);
        
    }
    
}
```

**左值引用**

先看一下传统的左值引用。

```c++
int a = 10;
int &b = a;  // 定义一个左值引用变量
b = 20;      // 通过左值引用修改引用内存的值
```

*左值引用在汇编层面其实和普通的指针是一样的；*定义引用变量必须初始化，因为引用其实就是一个别名，需要告诉编译器定义的是谁的引用。

```c++
int &var = 10;
```

上述代码是无法编译通过的，因为10无法进行取地址操作，无法对一个立即数取地址，因为立即数并没有在内存中存储，而是存储在寄存器中，可以通过下述方法解决：

```c++
const int &var = 10;
```

使用常引用来引用常量数字10，因为此刻内存上产生了临时变量保存了10，这个临时变量是可以进行取地址操作的，因此var引用的其实是这个临时变量，相当于下面的操作：

```c++
const int temp = 10; 
const int &var = temp;
```

根据上述分析，得出如下结论：

- 左值引用要求右边的值必须能够取地址，如果无法取地址，可以用常引用；
  但使用常引用后，我们只能通过引用来读取数据，无法去修改数据，因为其被const修饰成常量引用了。

那么C++11 引入了右值引用的概念，使用右值引用能够很好的解决这个问题。

**右值引用**

C++对于左值和右值没有标准定义，但是有一个被广泛认同的说法：

- 可以取地址的，有名字的，非临时的就是左值；
- 不能取地址的，没有名字的，临时的就是右值；

可见立即数，函数返回的值等都是右值；而非匿名对象(包括变量)，函数返回的引用，const对象等都是左值。

从本质上理解，创建和销毁由编译器幕后控制，程序员只能确保在本行代码有效的，就是右值(包括立即数)；而用户创建的，通过作用域规则可知其生存期的，就是左值(包括函数返回的局部变量的引用以及const对象)。

定义右值引用的格式如下：

```c++
类型 && 引用名 = 右值表达式;
```

右值引用是C++ 11新增的特性，所以C++ 98的引用为左值引用。右值引用用来绑定到右值，绑定到右值以后本来会被销毁的右值的生存期会延长至与绑定到它的右值引用的生存期。

```c++
int &&var = 10;
```

在汇编层面右值引用做的事情和常引用是相同的，即产生临时量来存储常量。但是，唯一 一点的区别是，右值引用可以进行读写操作，而常引用只能进行读操作。

右值引用的存在并不是为了取代左值引用，而是充分利用右值(特别是临时对象)的构造来减少对象构造和析构操作以达到提高效率的目的。

用C++实现一个简单的顺序栈：

```c++
class Stack
{
public:
    // 构造
    Stack(int size = 1000) 
	:msize(size), mtop(0)
    {
	cout << "Stack(int)" << endl;
	mpstack = new int[size];
    }
	
    // 析构
    ~Stack()
    {
	cout << "~Stack()" << endl;
	delete[]mpstack;
	mpstack = nullptr;
    }
	
    // 拷贝构造
    Stack(const Stack &src)
	:msize(src.msize), mtop(src.mtop)
    {
	cout << "Stack(const Stack&)" << endl;
	mpstack = new int[src.msize];
	for (int i = 0; i < mtop; ++i) {
	    mpstack[i] = src.mpstack[i];
	}
    }
	
    // 赋值重载
    Stack& operator=(const Stack &src)
    {
	cout << "operator=" << endl;
	if (this == &src)
     	    return *this;

	delete[]mpstack;

	msize = src.msize;
	mtop = src.mtop;
	mpstack = new int[src.msize];
	for (int i = 0; i < mtop; ++i) {
	    mpstack[i] = src.mpstack[i];
	}
	return *this;
    }

    int getSize() 
    {
	return msize;
    }
private:
    int *mpstack;
    int mtop;
    int msize;
};

Stack GetStack(Stack &stack)
{
    Stack tmp(stack.getSize());
    return tmp;
}

int main()
{
    Stack s;
    s = GetStack(s);
    return 0;
}
```

运行结果如下：

```c++
Stack(int)             // 构造s
Stack(int)             // 构造tmp
Stack(const Stack&)    // tmp拷贝构造main函数栈帧上的临时对象
~Stack()               // tmp析构
operator=              // 临时对象赋值给s
~Stack()               // 临时对象析构
~Stack()               // s析构
```

为了解决浅拷贝问题，为类提供了自定义的拷贝构造函数和赋值运算符重载函数，并且这两个函数内部实现都是非常的耗费时间和资源(首先开辟较大的空间，然后将数据逐个复制)，我们通过上述运行结果发现了两处使用了拷贝构造和赋值重载，分别是tmp拷贝构造main函数栈帧上的临时对象、临时对象赋值给s，其中tmp和临时对象都在各自的操作结束后便销毁了，使得程序效率非常低下。

那么我们为了提高效率，是否可以把tmp持有的内存资源直接给临时对象？是否可以把临时对象的资源直接给s？

在C++11中，我们可以解决上述问题，方式是提供带右值引用参数的拷贝构造函数和赋值运算符重载函数.

```c++
// 带右值引用参数的拷贝构造函数
Stack(Stack &&src)
    :msize(src.msize), mtop(src.mtop)
{
    cout << "Stack(Stack&&)" << endl;

    /*此处没有重新开辟内存拷贝数据，把src的资源直接给当前对象，再把src置空*/
    mpstack = src.mpstack;  
    src.mpstack = nullptr;
}

// 带右值引用参数的赋值运算符重载函数
Stack& operator=(Stack &&src)
{
    cout << "operator=(Stack&&)" << endl;

    if(this == &src)
        return *this;
	    
    delete[]mpstack;

    msize = src.msize;
    mtop = src.mtop;

    /*此处没有重新开辟内存拷贝数据，把src的资源直接给当前对象，再把src置空*/
    mpstack = src.mpstack;
    src.mpstack = nullptr;

    return *this;
}
```

运行结果如下：

```c++
Stack(int)             // 构造s
Stack(int)             // 构造tmp
Stack(Stack&&)         // 调用带右值引用的拷贝构造函数，直接将tmp的资源给临时对象
~Stack()               // tmp析构
operator=(Stack&&)     // 调用带右值引用的赋值运算符重载函数，直接将临时对象资源给s
~Stack()               // 临时对象析构
~Stack()               // s析构
```

程序自动调用了带右值引用的拷贝构造函数和赋值运算符重载函数，使得程序的效率得到了很大的提升，因为并没有重新开辟内存拷贝数据。

```c++
mpstack = src.mpstack;  
```

可以直接赋值的原因是临时对象即将销毁，不会出现浅拷贝的问题，我们直接把临时对象持有的资源赋给新对象就可以了。

所以，临时量都会自动匹配右值引用版本的成员方法，旨在提高内存资源使用效率。

带右值引用参数的拷贝构造和赋值重载函数，又叫移动构造函数和移动赋值函数，这里的移动指的是把临时量的资源移动给了当前对象，临时对象就不持有资源，为nullptr了，实际上没有进行任何的数据移动，没发生任何的内存开辟和数据拷贝。

### std::move究竟干了什么

### 万能引用（转发引用）

### 完美转发

## 元组基本概念、基础代码的设计与实现

## 操作接口（算法）