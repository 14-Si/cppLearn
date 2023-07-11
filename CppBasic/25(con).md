# STL

## string容器

string是字符容器，内部维护了一个动态的字符数组

与普通字符数组相比，string容器有三个优点：

1. 使用的时候，不必考虑内存分配和释放的问题
2. 动态管理内存（可扩展）
3. 提供了大量操作容器的API，缺点是效率略有降低

string类是std::basic_string类模板的一个具体化版本的别名

`using std::string = std::basic_string<char, std::char_traits<char>, std::allocator<char> >`

### 构造与析构

静态常量成员string::npos为字符数组的最大长度（通常为unsigned int的最大值）

NBTS（null-terminated string）:C风格字符串（以空字符0结束的字符串）

string类有七个构造函数（C++11新增了两个）：

1. `string()`创建一个长度为0的string对象（默认构造函数）
2. `string(const char* s)`将string对象初始化为s指向的NBTS（转换函数）
3. `string(const string &str)`将string对象初始化为str（拷贝构造函数）
4. `string(const char* s, size_t n)`将string对象初始化为s指向的NBTS的前n个字符，即使超过了NBTS的结尾
5. `string(const string &str, size_t pos = 0, size_t n = npos);`将string对象初始化为str从位置pos开始到结尾的字符，或从位置pos开始的n个字符
6. `template<class T> string(T begin, T end)`将string对象初始化为区间[begin, end]内的字符，其中begin和end的行为就像指针，用于指定位置，范围包括begin在内，但不包括end
7. `string(size_t m, char c)`创建一个由n个字符c组成的string对象

string类的析构函数释放内存空间

#### C++11新增的构造函数

1. `string(string &&str) noexcept`它将一个string对象初始化为string对象str，并可能修改str（移动构造函数）
2. `string(initializer_list<char> il)`它将一个string对啊ing初始化为初始化列表il中的字符，例如`string ss = {'a', 'e', 'i', 'o', 'u'};`

### 设计目标

string是一种以字节为最小存储单元的动态容器，可以用来存放字符串，用于存放数据的内存空间（缓冲区）

string内部有三个指针：

* char *start_	动态分配内存块开始的地址
* char *end_     动态分配内存块最后的地址
* char *finish_   已使用空间的最后的地址

### 操作

#### 特性操作

```
int max_size() const;	// 返回string对啊ing的最大长度string::npos,意义不大
int capacity() const;	// 返回当前容量，可以存放字符的总数
int length() const;		// 返回容器中数据的大小（字符串语义）
int size() const;		// 返回容器中数据的大小（容器语义）
bool empth() const;		// 判断容器是否为空
void clear();			// 清空容器
void reserve(size_t size = 0);		// 将容器的容量设置为至少size
void resize(int len, char c = 0);	// 把容器的实际大小置为len，如果len<实际大小，会截断多出的部分；如果len > 实际大小，就用字符c填充
```

#### 字符操作

```
char &operator[](int n);
const char &operator[](int n) const;			// 只读
char &at(int n);
const char &at(int n) const;	// 只读
// operator[] 和 at()都返回容器中的第n个元素，但at函数提供范围检查，当越界时会抛出 out_of_range异常，operator[]不提供范围检查
const char *c_str() const;		// 返回容器中动态数组的首地址，语义：寻找以null结尾的字符串
const char *data() const;		// 返回容器中动态数组的首地址，语义：只关心容器中的数据
int copy(char *s, int n, int pos = 0) const;	// 把当前容器中的内容，从pos开始的n个字节拷贝到s中，返回实际拷贝的数目
```

#### 赋值操作

```
string &operator=(const string &str);	// 把容器str赋值给当前容器
// 剩余参考构造函数，函数名为assign
```

#### 连接操作

```
string &operator+(const string &str);	// 把容器str连接到当前容器
// 剩余参考构造函数，函数名为append
```

#### 交换操作

```
void swap(string &str);	// 把当前容器与str交换
```

如果数据量很小，交换的是动态数组中的内容，如果数据量比较大，交换的是动态数组的地址

#### 截取操作

````
string substr(size_t pos = 0, size_t n = npos) const;	// 返回pos开始的n个字节组成的子容器
````

#### 比较操作

```
bool operator==(const string &str1, const string &str2) const;	// 比较两个字符串是否相等
int compare(const string &str) const;		// 比较当前字符串和str1的大小
int compare(size_t pos, size_t n, const string &str) const;		// 比较当前字符串从pos开始的n个字符组成的字符串与str的大小
int compare(size_t pos, size_t n, const string &str, size_t pos2, size_t n2) const;	// 比较当前字符串从pos开始的n个字符组成的字符串与str中pos2开始的n2个字符组成的字符串的大小
// 将compare中的string换为char *可以用于和C风格字符串比较
```

## vector

vector 容器封装了动态数组

包含头文件`#include<vector>`

vector类模板声明：

```
template<class T, class Alloc = alloctor<T> >	// T为容器存放的数据类型，Alloc为分配器，默认为STL提供的分配器
class vector{
private:
	T *start_;
	T *finish_;
	T *end_;
	...
};
```

### 分配器

各种STL容器模板都接受一个可选的模板参数，该参数指定使用哪个分配器对象来管理内存，如果省略该模板参数的值，默认使用allocator<T>，用new和delete分配和释放内存

##### 构造函数

```
vector();		// 创建一个空的vector容器
explicit vector(const size_t n);		// 创建 vector 容器，元素个数为n（容量和实际大小都是n）
vector(const size_t n, const T& value);	// 创建 vector 容器，元素个数为n，值为value
vector(const vector<T> &v);				// 拷贝构造函数
vector(Iterator first, Iterator last);	// 使用迭代器创建 vector 容器
vector(vector<T> && v);					// 移动构造函数
vector(initializer_list<T> il);			// 使用统一初始化列表
```

## 迭代器

迭代器是访问容器中元素的通用方法

如果使用迭代器，不同的容器，访问元素的方法是相同的

迭代器支持的操作：赋值=，解引用*，比较==/!=，从左向右遍历++

迭代器有五种

1. 正向迭代器

   只能使用++运算符来遍历容器，每次沿容器向右移动一个元素

   ```
   容器名<元素类型>::iterator 迭代器名;		// 正向迭代器
   容器名<元素类型>::const_iterator 迭代器名;	// 常正向迭代器
   ```

   相关函数：

   ```
   iterator begin()
   const_iterator begin();
   const_iterator cbegin();	// 配合auto使用
   iterator end();
   const_iterator end();
   const_iterator cend();
   ```

2. 双向迭代器

   具备正向迭代器的功能，还可以反向（从右到左）遍历容器（也是用++），不管是正向迭代器还是反向迭代器，都可以用--让迭代器后退一个元素

   ```
   容器名<元素类型>::reverse_iterator 迭代器名;		// 反向迭代器
   容器名<元素类型>::const_reverse_iterator 迭代器名;	// 常反向迭代器
   ```

   相关的成员函数：

   ```
   reverse_iterator rbegin();
   const_reverse_iterator crbegin();
   reverse_iterator rend();
   const_reverse_iterator crend();
   ```

3. 随机访问迭代器

   具备双向迭代器的功能，还支持以下操作

   * 用于比较两个迭代器相对位置的关系运算符（<, <=, >, >=）
   * 迭代器和一个整数值的加减法运算（+, +=, -, -=）
   * 支持下标运算符[]

   数组的指针是纯天然的随机访问迭代器

4. 输入和输出迭代器

   这两种迭代器比较特殊，它们不是把容器当作操作对象，而是把输入/输出流作为操作对象 

### 基于范围的for循环

对于一个有范围的集合来说，在程序代码中指定循环的范围有时候是多余的，还可能犯错误

C++ 11 中引入了基于范围的 for 循环

语法：

```
for(迭代的变量 : 迭代的范围)
{
	// 循环体
}
```

注意：

* 迭代的范围可以是数组名，容器名，初始化列表或者可迭代对象（支持begin(), end(), ++, ==）
* 数组名传入函数后，已退化成指针，不能作为容器名
* 如果容器中的元素是结构体和类，迭代器变量应该申明为引用，加const约束表示只读
* 注意迭代器失效问题

## list

list容器封装了双链表

包含头文件`#include<list>`

list 类模板声明：

```
templast<class T, class Alloc = allocator<T> >
class list
{
private:
	iterator head;
	iterator tail;
	...
};
```

### 构造函数

```
list();				// 创建一个空的list容器
list(initializer_list<T> il);		// 使用统一初始化列表
list(const list<T> &l);				// 拷贝构造函数
list(Iterator first, Iterator last);// 用迭代器创建list容器
list(list<T> &&l);	// 移动构造函数
explicit list(const size_t n);		// 创建list容器，元素个数为n
list(const size_t n, const T &value);	// 创建list容器，元素个数为n，值均为value
```

## pair

pair 是类模板，一般用于表示key/value数据，其实现是结构体

pair结构模板定义如下：

```
template<class T1, class T2>
struct pair
{
	T1 first;
	T2 second;
	
	pair();
	pair(const T1 &val1, const T2 &val2);
	pair(const pair<T1, T2> &p);
	void swap(pair<T1, T2> &p);
};
```

make_pair函数模板的定义如下

```
template<class T1 ,class T2>
make_pair(const T1 &first, const T2 &second)
{
	return pair<T1, T2>(first, second);
}
```

## map

map容器封装了红黑树（平衡二叉排序树）用于查找

包含头文件`#include <map>`

map 容器的元素是pair键值对

map 类模板的声明

```
template<class K, class V, class P = less<K>, class_Alloc = allocator<pair<const K, V> > >
class map : public_Tree<_Tmap_traits<K, V, P, _Alloc, false> >
{
	...
}
```

第一个模板参数K：key的数据类型（pair.first）

第二个模板参数V：value的数据类型（pair.second）

第三个模板参数P：排序方法，默认按key升序

第四个模板参数_Alloc：分配器，默认用new和delete

map提供了双向迭代器

二叉链表

```
struct BTNode
{
	pair<K, V> p;
	BTNode *parent;
	BTNode *lchild;
	BTNode *rchild;
};
```

### 构造函数

```
map();				// 创建一个空的map容器
map(initializer_list<pair<K, V> > il);	// 使用统一初始化列表
map(const map<K, V> &m);				// 拷贝构造函数
map(iterator first, iterator last);		// 用迭代器创建map容器
map(map<K, V> &&m);	// 移动构造函数
```

## unordered_map

unordered_map容器封装了哈希表，查找，插入，删除元素时，只需要比较几次key的值。

包含头文件`#include <unordered_map>`

unordered_map容器的元素是pair键值对

unorpered_map类模板声明：

```
template<class K, class V. class_Hasher = hash<K>, class_Keyeq = equal_to<K>, class_Alloc = allocator<pair<const K, V> > >
class unordered_map : public_Hash<_Umap_traits<K, V, _Uhash_compare<K, _Hasher, _Keyeq>, _Alloc, false> >
{
	...
};
```

第一个模板参数K：key的数据类型

第二个模板参数V：value的数据类型

第三个模板参数_Hasher：哈希函数，默认为`std::hash<K>`

第四个模板参数_Kereq：比较函数，用于判断两个key是否相等，默认值是`std::equal_to<K>`

第五个模板参数_Alloc：分配器，默认使用new和delete

创建std::unordered_map类模板的别名：

```
template<class K, class V>
using umap = std::inordered_map<K, V>;
```

### 构造函数

```
umap();						// 创建一个空的 umap容器
umap(size_t bucket);		// 创建一个空的umap容器，指定了桶的个数，下同
umap(initializer_list<pair<K, V> > il);					// 使用统一初始化列表
umap(initializer_list<pair<K, V> > il, size_t bucket);  // 使用统一初始化列表
umap(Iterator first, Iterator last);					// 使用迭代器创建umap容器
umap(Iterator first, Iterator last, size_t bucket);		// 使用迭代器创建umap容器
umap(const umap<K, V> &m);	// 拷贝构造函数
umap(umap<K, V> &&m);		// 移动构造函数
```



### 哈希表

哈希表长（桶的个数）：数组的长度

哈希函数：`size_t hash(const T &key){...; key % 小于哈希表长的最大质数}`

装填因子：元素总长/表长，其值越大，效率越低

## queue

## 其他容器

## for_each

## find_if

## sort

## 算法总结

