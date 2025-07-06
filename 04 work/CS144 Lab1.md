字节重排

处理：
1. 收到可用的下一个字节，立刻调用 write（）
2. 收到失序字节，缓存到本地 
3. 丢弃超出容量的字节（从 writer 字节流中未 read 的字节为起点）
4. 遇到 EOF 标记，当读入 EOF 时关闭流


数据结构设计：
- 字节碎片采用 `struct Fragment`
	1. `head_index` : `uint64_t`
	2. `tail_index` : `uint64_t`
	3. `data` : `string`
	4. `merge(A, B)` : 合并 `Fragment`
	5. 重载运算符 `<` ，`head_index` 为第一关键字
- 缓存采用 STL `set<Fragment>`
- 双指针
	- ByteStream buffer 头指针 `buf_base`
	- 缓存 头指针 j`

算法设计：
- 缓存容量查询：
	- 遍历 set，累加`data.size()`
- Fragment 合并 ：
	- 判断是否为子集
	- 判断是否重叠
	- 合并左右区间
	- 返回新区间
	- 更新 set，删除旧的，插入新的
- 检查新的 Fragment 是否溢出缓存，并丢弃数据
	- 比较 data 尾指针和 `buf_base + capacity`
- 维护 set 两两不相交：
	- 假设每次调用 insert 前 set 有序
	- 直接插入新的 Fragment
	- 遍历 set，检查相邻两项是否可合并
- 检查 set 首元素是否有序
	- 有序则 write，踢出首元素



Another Version：
数据结构设计：
- STL `deque<char>` 模拟缓存中的乱序字节
	- 用 对应的 `deque<bool>` 表示未收到字节

算法设计：
- 查询缓存空间，就遍历 `deque<bool>` 统计
- 插入字块，就遍历 `deque` 下标进行直接更新
- 溢出处理，写入

1.  frag_base + aviliable_capacity() 判断缓存溢出