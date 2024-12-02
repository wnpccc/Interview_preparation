# C++11新特性

## 一、std::move相关

+ std::move 是一种将对象状态从一个地方转移到另一个地方的机制，它并不实际执行任何操作，只是将源对象转化为“右值引用”，允许你在后续的代码中使用移动构造函数或移动赋值运算符。
+ 转化为右值可以触发移动语义
+ 原对象是否可用取决于移动构造函数和移动赋值运算符

调用move就意味着承诺：除了对移后源对象进行赋值和销毁，我们将不再使用它。在调用move后，我们不能对移后源对象的值做任何假设

## 二、模版相关

### 优点

提高代码重用性、类型安全、效率高（编译时生成避免运行时类型检查）、灵活性强

### 缺点

编译时间增加、错误信息复杂、代码膨胀、可读性和维护性差

### 引用折叠规则

1. 当我们将一个左值传递给函数的右值引用参数，且此右值引用参数指向模板类型参数时，编译器推断模板类型参数为实参的左值引用类型。
2. `X& &、X& && 和 X&& &` 都会折叠为 `X&`
3. `X&& &&` 会折叠为 `X&&`

### 完美转发  

参考 `https://gukaifeng.cn/posts/c-wan-mei-zhuan-fa/index.html`

在函数之间传递参数的过程中，参数在传递后的属性保持不变（如左值仍是左值，右值仍是右值，const 修饰也会保留）  

完美转发需要使用到标准库中的 `std::forward<>()` 函数，其定义在头文件 `<utility>` ，其必须通过显式模板实参来调用。  
其有两个重载，一个接收左值引用类型参数，另一个接收右值引用类型参数，定义如下：

```c++
/**
 *  @brief  Forward an lvalue.
 *  @return The parameter cast to the specified type.
 *
 *  This function is used to implement "perfect forwarding".
 */
template<typename _Tp>
  constexpr _Tp&&
  forward(typename std::remove_reference<_Tp>::type& __t) noexcept
  { return static_cast<_Tp&&>(__t); }

/**
 *  @brief  Forward an rvalue.
 *  @return The parameter cast to the specified type.
 *
 *  This function is used to implement "perfect forwarding".
 */
template<typename _Tp>
  constexpr _Tp&&
  forward(typename std::remove_reference<_Tp>::type&& __t) noexcept
  {
    static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"
      " substituting _Tp is an lvalue reference type");
    return static_cast<_Tp&&>(__t);
  }
```

## 三、函数和可调用对象

### std::function

用于存储和调用任意可调用对象（函数指针、Lambda、函数对象等）。常用场景包括回调函数、事件处理、作为函数参数和返回值

### std::bind

用于绑定函数参数，生成函数对象，特别是当函数参数不完全时。常见于将已有函数接口适配为接口要求的回调、将成员函数与对象绑定

```C++
#include <iostream>
#include <functional>

void print_product(int a, int b) {
    std::cout << "Product: " << a * b << std::endl;
}

int main() {
    //std::bind(function, arg1, arg2, ...);
    // 创建一个新的可调用对象，绑定了第一个参数 a=5
    //auto bound_func = std::bind(print_sum, 5, std::placeholders::_1);
    // 创建一个新的可调用对象，绑定了成员函数 print_sum 和对象 calc
    //auto bound_func = std::bind(&Calculator::print_sum, &calc, 5, std::placeholders::_1);
    // 绑定参数顺序：将 b 作为第一个参数，a 作为第二个参数
    auto bound_func = std::bind(print_product, std::placeholders::_2, std::placeholders::_1);

    bound_func(5, 3);  // 输出: Product: 15  a=3,b=5

    return 0;
}

```

### Lambda表达式

用于定义匿名函数，通常在短期和局部使用函数时比如一次性回调函数、算法库中的自定义操作等

## 四、constexpr

### 优点

+ 编译时计算，提高性能
+ 提高代码可读性
+ 类型安全
+ 与模版配合使用
+ 简化代码

constexpr 变量和函数必须能够在编译时计算出其值，这意味着它们不能依赖于运行时输入或外部状态。
