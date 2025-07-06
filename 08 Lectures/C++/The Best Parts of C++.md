[[https://www.bilibili.com/video/BV1qt4y137X3?spm_id_from=333.788.videopod.episodes&vd_source=9d5c1f6340c538765349683259dc7c94&p=2]]

#### 1. C++ Standard

C++ 会有共用的标准，如 `C++17`、`C++20`
#### 2. `const`

#### 3. 对象的构造与析构

自动管理生命周期，实现资源的隐式释放

#### 4. `template`

#### 5. STL

#### 6. `std::array`

指示固定大小的序列

#### 7. List initialization

序列初始化，形如 `std::vector<int> v { 1, 2, 3 };`

#### 8. Variadic templates

template 带有可变的参数列表，自动识别

#### 9. `constexpr`

编译时计算，在编译层面显式地指示优化

#### 10. `auto`

自动识别复杂数据类型，并匹配

#### 11. 返回值类型识别

对于函数的返回值类型也可以用 `auto`

#### 12. lambda 匿名函数

#### 13. 通用与可变的 lambda

允许使用可变参数列表，也允许用 `auto` 捕获一个 lambda

#### 14. 容器迭代

`for (const auto &value : container) {}`

#### 15. Structured bindings

`const auto &[elem1, elem2] = someThing;`

#### 16. Concepts

#### 17. std::string_view

用于构建字符串常量，不需要 copy

#### 18. Text formatting

格式化输出文本

#### 19. Ranges(C++20)

管道流+高阶函数的思想，新特性，很多人评价不实用

#### 20. Class 模板参数的自动解码

#### 21. 右值引用

避免不恰当的 copy

#### 22. Graranteed copy elision

没懂

#### 23. Defaulted and Deleted Functions

可以显式指定函数类型 `S() = default`

#### 24. `std::unique_ptr` and `std::make_unique`

#### 25. Fold Expressions

没懂
