# 使用了SFINEA特性的信息萃取

## 用成员函数重载实现is_default_constructible

c++标准库中： std::is_default_constructible 类模板

该类模板的能力是用于判断一个对象是否能被默认构造（也就是构造一个类的时候，不需要给该类传递任何参数, 只要该类没有构造函数或者有一个不带参数的构造函数就都能默认构造）

```c++
namespace _nmsp1
{
    class A
    {
        
    };
    
    class B
    {
        int m_i;
    public:
        B(int a):m_i(a){}
    };
    
    void func()
    {
        // A可以默认构造
        A a;
        // B不能被默认构造
        // B b;
        B b(10);
        // **----------------------------------**
        std::cout << std::is_default_constructible<A>() << std::endl;       // 1
        std::cout << std::is_default_constructible<B>::value << std::endl;  // 0
    }
}
```



## 用成员函数重载实现is_convertible

## 用成员函数重载实现is_class

## 用成员函数重载实现is_base_of

## 用类模板特化实现is_default_constructible