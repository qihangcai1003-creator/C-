## 1. 头文件防卫 (Header Guard)

* **知识点**：防止头文件被重复包含导致编译错误。
* **结论**：优先使用现代写法。

### 代码对比

```cpp
// [推荐] 现代写法
// 特点：编译器直接支持，编译速度快，无需担心宏名冲突
#pragma once 

// [老派] 传统写法
// 特点：兼容性最强（标准支持），但编译稍慢，需要人为保证宏名唯一
#ifndef __MY_CLASS_H__
#define __MY_CLASS_H__

// ... 代码内容 ...

#endif
````

-----

## 2\. 构造函数 (Constructor)

  * **知识点**：初始化列表 (Initialization List) vs 函数体赋值 (Assignment)。
  * **结论**：一定要用初始化列表（冒号后面写），效率最高。

### 代码对比

```cpp
// [推荐] 初始化列表
// 特点：变量诞生即赋值 (直接初始化)，对于类成员对象效率提升显著
Complex(double r, double i) : re(r), im(i) {} 

// [不推荐] 函数体赋值
// 特点：变量先诞生(默认构造)，然后再被修改(赋值)，多了一次操作
Complex(double r, double i) { 
    re = r; 
    im = i; 
}
```

> **⚠️ 避坑指南 (Ambiguity)**
>
> 不要同时定义 `Complex()` 和 `Complex(double r=0)`。
> 前者是无参构造，后者是所有参数都有默认值的构造（也可以无参调用）。编译器会报 **二义性错误 (Ambiguity)**，因为它不知道该调用哪一个。

-----

## 3\. 析构函数 (Destructor)

  * **知识点**：对象生命周期结束时的清理工作。
  * **结论**：根据“是否有指针成员”来决定。

### 判断标准

1.  **无指针** (只有 `int`, `double`, `std::vector` 等)：不用写析构函数，使用编译器默认生成的即可。
2.  **有指针** (new 出来的资源)：**必须写**，需要在析构函数中 `delete`，否则会导致内存泄漏。

<!-- end list -->

```cpp
// 有指针的例子
~String() { 
    delete[] data; // 拿了系统的就要还
} 
```

-----

## 4\. 内联函数 (Inline)

  * **知识点**：用空间换时间（编译器将函数代码直接复制到调用处，省去函数跳转开销）。
  * **结论**：
      * **类内部定义**：写在 `class { ... }` 内部的函数，编译器会自动将其视为隐式 `inline`。
      * **类外部定义**：写在 `class` 外部的函数，如果想要内联，必须手动添加 `inline` 关键字。

-----

## 5\. 参数传递 (Pass by...) —— 性能关键

  * **知识点**：如何传参效率最高？
  * **结论**：除了基础类型（`int`, `bool` 等小类型），其他对象全部传 **Const Reference**。

### 代码对比

```cpp
// [推荐] 传常引用 (Pass by Reference to Const)
// 特点：只传地址（快），不拷贝对象，const 保证原始数据不被修改
double func(const Complex& param);

// [不推荐] 传值 (Pass by Value)
// 特点：会拷贝整个对象，如果对象很大，性能开销巨大
double func(Complex param); 
```

-----

## 6\. 返回值传递 (Return by...) —— 难点

  * **知识点**：什么时候可以返回引用？
  * **结论**：取决于对象存在哪里（生命周期）。

### 场景分析

#### ❌ 情况一：局部变量（不能返回引用）

函数内部创建的变量，函数结束时会被销毁。如果返回引用，指向的是一个“尸体”（悬空引用）。

```cpp
// [危险] 错误示范
Complex& bad_func() {
    Complex c; 
    return c; // 函数结束 c 就销毁了，外部拿到的是非法内存！
}
```

#### ✅ 情况二：已有对象（可以返回引用）

对象在函数调用前就已经存在（例如通过指针传进来的，或者 `*this` 对象）。

```cpp
// [安全] 正确示范：支持链式调用 (c1 += c2 += c3)
Complex& operator+=(const Complex& r) { 
    this->re += r.re; 
    return *this; // *this 指向当前对象，它一直存在，返回引用安全
}
```

-----

## 7\. 友元逻辑 (Friendship)

  * **知识点**：私有成员 (`private`) 的访问权限控制。
  * **结论**：相同类 (`class`) 的不同对象 (`objects`) 互为友元。
  * **现象**：在 `Complex` 类的方法里，可以直接访问另一个 `Complex` 对象实例的私有成员。

<!-- end list -->

```cpp
class Complex {
public:
    int func(const Complex& param) {
        // param 是另一个对象，但因为同属 Complex 类
        // 所以可以直接访问 param.re (private)
        return param.re + param.im; 
    }

private:
    double re, im;
};
```

```
```
