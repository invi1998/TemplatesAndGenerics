# typelist的实现（类型计算的范例）

《Modern C++ Design》-(c++设计新思维)，在这本书中通过设计了一个叫做Loki的库实现typelist（采用列表头 + 嵌套列表技术来实现，因为03年c++还不支持可变参模板）

typelist是一个类模板，是用来操作一大堆类型的c++容器，就像c++标准库中的list容器（能对各种数值提供各种基本操作）

typelist与之类似，但是他操作的不是数值，而是针对类型进行操作。所以typelist简单来说就是一个类型容器，能够对类型提供一系列操作，把类型当成数据来操作

从实现上来讲，typelist是一个类模板，中文名“类型列表”。该类模板表示一个列表，在该列表中存放一堆类型

之前对于元编程的学习，了解到，元编程很大一部分是针对类型进行计算的

## 设计和基本操作接口（算法）

可以考虑把typelist相关的定义放到一个命名空间中，避免全局命名空间被污染

### 取得typelist中的第一个元素（front）

既然typelist是一个类型列表，也就是他是保存类型的容器，那么毫无疑问，取得typelist中的第一个元素肯定也是一个类型

```c++
// ------------------------------------------------------------------------------
    // 取得typelist中的第一个元素（front）
    // 写一个类模板，他的能力就是从typelist中取得第一个元素
    // TPLT 就代表整个typelist<...>类型
    // 泛化版本，因为用不到，所以只需要声明，
    template<class TPLT>
    class front;
    
    // 特化版本
    // 这里要尤其注意这里特化版本的写法，因为泛化版本中只有一个类型参数 TPLT，
    // 所以在特化版本，front后面的尖括号中也应该要保证，只能有一个具体类型 typelist<First, Other...>
    // 然后，在根据特化版本中front后面的<>中的类型，来确定模板中应该有哪些模板参数，即 <class First, class... Other>
    template<class First, class... Other>
    class front<typelist<First, Other...>>
    {
    public:
        using type = First;
    };
```



### 取得typelist容器中元素的数量（size）

### 从typelist中移除第一个元素（pop front）

### 向typelist的开头和结尾插入一个元素（push_front、push_back）

### 替换typelist的开头元素（replace_front）

### 判断typelist是否为空（is_empty)

## 扩展操作接口（算法）

## typelist的老式设计与typelist的思考