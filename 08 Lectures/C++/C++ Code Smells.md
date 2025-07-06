[[https://youtu.be/maDhQ57-gr8]]
## 构造与赋值分离

尽量别这么写，会增加编译长度
如果这个值从未使用过，那么编译器会优化为 `nop`
但是如果你分离着写，就需要分别编译构造和赋值，意味着更多的 cost


## 不要使用输出参数

`const int get_value(int &out)`
别这么写，不要通过参数来传递引用或者输出结果

Instead，`const auto value = get_value()`
查看编译结果会发现，这个函数有一个隐含的参数，就是`value` 的返回地址

## Raw Loops

为提高可读性，最好使用 algorithms 库函数，让代码的意图更明显

lambda 很 cool

尽量不要用 全局capture `[&]`

## 重载运算符

必须对返回值和操作对象加 `const`
## 类型转换

避免一切隐式的类型转换
注意类型的数据范围

## `std::move`

强制 cast 为 rvalue reference

避免将 `std::move()` 作为返回值，编译器可能会优化掉 move

## `const_cast`

对一个 `const` 类型作强制转换，并修改内容，不可以

一个 `const` 的 lifetime 是 UB 时，编译器可能会假定它不会再变化，因此一切修改操作都会被省略


## 类型命名

当你使用一个具有实际含义的数据类型时，相当**有必要**给它一个具体的名字 e.g. `size_t`，如果你没有这么做，那么面对 一堆`int`，代码的可读性将变得极差

## `static` is not free

当你在函数中使用 `static` 变量时，每次进入函数，编译器会检查一个 guard variable，检查该变量是否被初始化

因此增加性能开销

更应该用 `constexpr` 而不是 `static const`
前者会在编译时就完成

## 别用 `extern const` 

这么写，会强迫编译器去检查相应的数据类型，增加开销，同样应该用 `constexpr`

## 也别直接用 `new` 和 `delete`（Raw）

内存管理，就用智能指针，别手动操作

---

## Compiler is your best friend

![[Pasted image 20250630232038.png]]

## 避免使用临时变量

尽量用字面量直接作为返回值

[[NVRO]] 不希望对返回一个临时变量的`move`