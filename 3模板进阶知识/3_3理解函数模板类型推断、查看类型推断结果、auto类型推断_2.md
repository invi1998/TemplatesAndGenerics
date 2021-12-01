# 理解函数模板类型推断

## 万能引用类型

```c++
namespace _nmsp3
{
    
    // 函数模板
    template<typename T>
    void getTypeByBoost(T&& tmprv) // 万能引用类型
    {
        cout << "------------------------------begin----------------------------" << endl;
        
        using boost::typeindex::type_id_with_cvr;
        cout << " T 类型 = " << type_id_with_cvr<T>().pretty_name() << endl;      // 显示T的类型
        cout << " tmprv 类型 = " << type_id_with_cvr<decltype(tmprv)>().pretty_name() << endl; // 显示tmprv的类型
        
        cout << "------------------------------endl-----------------------------" << endl;
    }
    
    void func()
    {
        int i = 100;        // i的类型int
        const int j = i;    // j的类型const int
        const int &k = i;   // k的类型const int&
        
        getTypeByBoost(i);
        // ------------------------------begin----------------------------
        //  T 类型 = int&
        //  tmprv 类型 = int&
        
        
        getTypeByBoost(j);
        // ------------------------------endl-----------------------------
        // ------------------------------begin----------------------------
        //  T 类型 = int const&
        //  tmprv 类型 = int const&
        // ------------------------------endl-----------------------------
        
        
        getTypeByBoost(k);
        // ------------------------------begin----------------------------
        //  T 类型 = int const&
        //  tmprv 类型 = int const&
        // ------------------------------endl-----------------------------
        
        getTypeByBoost(120);
        // ------------------------------begin----------------------------
        //  T 类型 = int
        //  tmprv 类型 = int &&
        // ------------------------------endl-----------------------------
        
    }

}
```



## 传值方式

```c++
namespace _nmsp4
{
    
    // 函数模板
    template<typename T>
    void getTypeByBoost(T tmprv) // 传值方式
    {
        cout << "------------------------------begin----------------------------" << endl;
        
        using boost::typeindex::type_id_with_cvr;
        cout << " T 类型 = " << type_id_with_cvr<T>().pretty_name() << endl;      // 显示T的类型
        cout << " tmprv 类型 = " << type_id_with_cvr<decltype(tmprv)>().pretty_name() << endl; // 显示tmprv的类型
        
        cout << "------------------------------endl-----------------------------" << endl;
        
        
    }
    
    void func()
    {
        int i = 100;        // i的类型int
        const int j = i;    // j的类型const int
        const int &k = i;   // k的类型const int&
        
        getTypeByBoost(i);
        // ------------------------------begin----------------------------
        //  T 类型 = int
        //  tmprv 类型 = int
        
        
        getTypeByBoost(j);
        // ------------------------------endl-----------------------------
        // ------------------------------begin----------------------------
        //  T 类型 = int
        //  tmprv 类型 = int
        // ------------------------------endl-----------------------------
        
        
        getTypeByBoost(k);
        // ------------------------------begin----------------------------
        //  T 类型 = int
        //  tmprv 类型 = int
        // ------------------------------endl-----------------------------
        
        getTypeByBoost(120);
        // ------------------------------begin----------------------------
        //  T 类型 = int
        //  tmprv 类型 = int
        // ------------------------------endl-----------------------------
        
        
        // 手动指定（不建议这样写代码）
        int& m = i;
        getTypeByBoost<int&>(m);
        // ------------------------------begin----------------------------
        //  T 类型 = int&
        //  tmprv 类型 = int&
        // ------------------------------endl-----------------------------
        
        // 传递指针进去
        char mystr[] = "LiKe";
        const char* const p = mystr;    // 第一个const表示p指向的目标中的内容不能通过p改变，第二个cosnt表示p指向mystr以后，不能再指向其他内容（p的指向不能变）
        getTypeByBoost(p);
        // ------------------------------begin----------------------------
        //  T 类型 = char const*
        //  tmprv 类型 = char const*      
        // ------------------------------endl-----------------------------
        // 传递给函数模板后，第二个const没有了，第一个const被保留
        // 那这也就说明p指向的内容不能通过p改变，但是p的指向却可以改变
        // 也就是通过传值方式进入到函数模板内部后，tmprv指向的内容不能通过tmprv改变，但是tmprv可以指向其他内容
        // tmprv(也就是实参p）的常量性被忽略，tmprv(也就是实参p）所指向内容的常量性会被保留
        // 即在模板函数内：tmprv = nullptr; 这个代码是可以的
        // tmprv = "ppppp"; 这个代码是不行的，编译报错，不能给常量赋值
    }

}
```



## 传值方式的引申---std::ref与std::cref

## 数组做实参

## 函数名做实参

## 初始化列表做实参



# auto类型常规推断

## 传值方式（非指针，非引用）

## 指针或者引用类型但不是万能引用

## 万能引用类型