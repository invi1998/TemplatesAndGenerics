# 策略（policy）技术中的算法策略

## 常规范例：普通策略类

```c++
namespace _nmsp1
{
    // SumFixedTraits类模板
    template<typename T>
    struct SumFixedTraits;
    
    template<>
    struct SumFixedTraits<char>
    {
        using sumT = int;
        static sumT initValue() {return 0; }
    };
    
    template<>
    struct SumFixedTraits<int>
    {
        using sumT = int64_t;
        static sumT initValue() {return 0; }
    };
    
    template<>
    struct SumFixedTraits<double>
    {
        using sumT = double;
        static sumT initValue() {return 0.0; }
    };
    // ----------------------------------------
    // 写一个适合求最小值的固定萃取类模板
    template<typename T>
    struct MinFixedTraits;
    
    template<>
    struct MinFixedTraits<int>
    {
        using sumT = int;   // 求最小值，结果类型与元素类型相同即可，为了名字统一调用（因为下面有用到），所以都叫sumT
        static sumT initValue() {return INT_MAX; }
        // 要求数组的最小值，这里可以返回一个最大值做初始值
        // INT_MAX 是系统定义的 整形最大值
    };
    
    // ----------------------------------------
    // 将求和操作（也就是算法）写成一个策略类
    struct sumPolicy
    {
        // 算法一般实现成静态成员函数模板
        // sumT 和值类型
        // T是元素类型
        template<typename sumT, typename T>
        static void algorithm(sumT &sum, const T& value)    // 策略类核心算法
        {
            sum += value;   // 求和
        }
    };
    // ----------------------------------------
    // 实现一个求最小值的策略类
    struct MinPolicy
    {
        // minT:最小值类型
        // T：数组元素类型
        template<typename minT, typename T>
        static void algorithm(minT &min, const T& value)
        {
            if (min > value)
            {
                min = value;
            }
        }
    };
    
    // ----------------------------------------
    // 给funcsum增加一个模板参数以进一步funcsum的灵活性
    template<typename T, typename U = SumFixedTraits<T>, typename Plicy = sumPolicy>
    auto funcsum(const T* begin, const T* end)
    {
        typename U::sumT sum = U::initValue();
        
        for(;;)
        {
            Plicy::algorithm(sum, *begin);
            if (begin == end)
                break;
            ++begin;
        }
        return sum;
    }
    // ----------------------------------------
    
    
    void func()
    {
        char mc[] = "abc";
        std::cout << (int)funcsum(&mc[0], &mc[2]) << std::endl;
        // 294
        
        
        int mi[] = {10, 34, 2, 6, 12, 5, 1, 90, 2, 52};
        std::cout << funcsum<int, SumFixedTraits<int>, MinPolicy>(&mi[0], &mi[9]) << std::endl;
        // 0
        // 看到结果是 0 ，并不是1，因为这里我们给sum符了一个初值0，所以它判断不到比0大的真正的最小值
        
        // 有两个解决方式，写一个新的固定萃取类模板来取代SumFixedTraits
        // 或者给 funcsum 这个函数模板新增一个非类型参数类接收 初值
        
        // MinFixedTraits
        std::cout << funcsum<int, MinFixedTraits<int>, MinPolicy>(&mi[0], &mi[9]) << std::endl;
        // 1
        
    }
}
```



## 常规范例：策略类模板

## 萃取（trait）技术与策略（policy）技术的比较