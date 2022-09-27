

### C++11新特性

- 自动类型推导auto/decltype
- 智能指针
- 右值引用和引用折叠
- 列表初始化
- 多线程与锁
- lamda表达式
- 空指针nullptr
- 范围for循环
- constexpr关键字，编译阶段确认变量结果
#### 类型推导

auto在编译阶段推断变量的类型

1. 必须初始化，否则无法推导
2. 一次定义多个变量不能有二义性
3. 不能做函数参数
4. 类中不能做非静态成员变量
5. 不能定义数组，可以定义指针
6. 不能推导模板
7. 不声明为引用或指针时，auto会忽略引用和cv
8. 引用需要显示声明

decltype推断表达式类型表达式不会进行运算，编译阶段处理

1. decltype会保留引用和cv属性

```cpp
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) {
return t + u;
}
```

#### 右值引用

**左值**：可以在赋值运算左边也可以在右边

**右值**：只能出现在赋值运算符右边

1. 将亡值
   即将要销毁的值，可以让其他变量接管这个值的资源，避免内存的释放和拷贝
2. 左值引用，变量别名
3. std::move可以将左值转换为右值
4. 移动语义，转让所有权，对实现了移动构造和移动拷贝的类的对象，基本类型没有意义

**浅拷贝**：a和b指向同一块内存，只是数据的简单复制

**深拷贝**：拷贝独享是对对象内部的指针引用的资源进行拷贝，开批新的空间存放拷贝的资源

==完美转发==std::forward():一个接收任意实参的函数模板，准发到其他函数，目标函数会收到与转发函数完全相同的实参，通过std::farrow_forward

std::ref解决std::bind的传参问题，因为std::bind调用的是值拷贝，需要调用引用的时候需要显式的使用std::ref进行绑定

#### nullptr

nullptr是用来替代NULL的，C++把NULL和0视为统一同喜，因为有的编译器将NULL定义为0

C++不允许把void*隐式的转换到其他类型，否则重载时会发生混乱

```cpp
void foo(char *);
void foo(int );
```

foo(NULL)会调用foo(int)

nullptr的类型是nullptr_t，能够转换为任何指针或成员指针类型

#### 范围for循环

#### 列表初始化

用花括号进行初始化，实现方式是使用`initializer_list<T>`接收初始化数据，使用的时候用花括号匹配，类似于数组的初始化

没有使用`initializer_list<T>`的时候会和构造函数匹配

#### lamda表达式

lamda表示一个可调用的可调用的代码单元，C++primer的表述是创建一个临时的类型和这个类型的一个对象，这个对象可以想函数一样被调用。

1. 语法

   ```cpp
   [capture list] (parameter list)[mutable]-> return type {function body }
   // [ഔ឴ڜᤒ] (݇හڜᤒ) -> ᬬࢧᔄڍ{ ࣳහ֛ }
   // ݝ有 [capture list] ഔ឴ڜᤒ{ ޾function body } ڍහ֛ฎ஠ᭌጱ
   auto lam =[]() -> int { cout << "Hello, World!"; return 88; };
   auto ret = lam();
   cout<<ret<<endl; // ᬌڊ88
   ```

2. 表达式特点

   1. []不捕获任何变量
   2. [&]捕获所有变量引用
   3. [=]值捕获所有变量，但是const
   4. [=, &foo] 以引⽤捕获变ᰁfoo, 但其余变ᰁ都靠值捕获
   5.  [&, foo] 以值捕获foo, 但其余变量都靠引⽤捕获
   6.  [bar] 以值⽅式捕获bar; 不捕获其它变量
   7. mutable可以转换const捕获
   8. 支持算法库

#### 并发

##### std::thread

1. 默认构造函数创建空的thread

2. 构造函数传递一个函数，thread对象创建的时候就会开始执行，参数在后面给出，也可以直接用std::bind传递无参的函数

3. 不能拷贝

4. 可以移动，移动后的右值不代表任何thread对象

5. 注：可以joinable的对象不许在他们销毁之前被主线程join或者设置为detached
   例如：

   ```cpp
   #include <iostream>
   #include <thread>
   void threadproc() {
   while(true) {
   	std::cout << "I am New Thread!" << std::endl;
   }
   }
   void func() {
   	std::thread t(threadproc);
   }
   int main() {
       func();
       while(true) {} //防止主线程退出
       return 0;
   }
   ```

   崩溃原因

   func结束后，局部变量t被释放，此时线程函数仍在运行，使用thread类的时候，必须保证线程函数运行期间气象成对象有效

   使用detached方方让线程对象和线程函数脱离关系，即使线程对象被释放也不会影响线程函数的运行。

   修改如下

   ```cpp
   void func() {
       std::thread t(threadproc);
       t.detach();
   }
   ```

##### 两种锁lock_guard与unique_lock

> `lock_guard`
>
> 1. 创建时加锁，作用域结束自动析构并解锁
> 2. 不能中途解锁
> 3. 不能复制

> `unique_lock`
>
> 1. 创建时不加锁，需要时锁定
> 2. 随时加锁
> 3. 脱离作用域解锁
> 4. 不可复制可以移动
> 5. 可以使用条件变量

### C++14/17新特性
#### C++14
##### 类型推断更新
允许推断函数返回值，允许对auto进行decltype

[mongrel2](https://github.com/mongrel2/mongrel2)



```cpp
auto func(){
    return 10;
}

string getStr(){ return string("123");}
string& getStrRef(){ 
   string* pStr = new string("456");
   return *pStr;
}

auto funcStrA(){
   return getStr();
}

auto funcStrB(){ //这里返回的不是引用类型
   return getStrRef();
}


decltype(auto) funcStrC(){
   return getStrRef();
}
decltype(auto) funcStrD(string&& str){ //这里返回的是引用类型
	return std::forward<string>(str);
}

```
##### 允许模板lamada
```cpp
auto add = [](auto a, auto b){
	return a + b;
};
cout<< add(1, 2)<< add(string("abc"), string("def"))<<endl;
```
##### 允许变量模板
```cpp
template<typename T>
T var

var<int> a = 5;
var<string> b = 2.0;
....
```
##### std::enable_if

#### C++17
##### 结构化绑定，解包tuple、数组以及结构体
```cpp
//c++17之前
auto tup = make_tuple("123", 12, 7.0);
string str;
int i;
double d;
std::tie(str, i, d) = tup;

//C++17之后
auto [x,y,z] = tup;
//对数组结构体
double myArray[3] = { 1.0, 2.0, 3.0 };  
auto [a, b, c] = myArray;
auto& [ra, rb, rc] = myArray;
struct S { int x1 : 2; double y1; };
S f();
const auto [ x, y ] = f; //备注1：
```
 备注1：gcc版本的编译器对此项特性的支持有bug，  
##### 可变参数模板表达式折叠
```cpp
//17之前
template<typename T>
auto myAdd(const T& a,const T& b){
    return a + b;
}
template<typename T, typename... RestT>
auto myAdd(const T& a, const RestT&... restArgs){
    return a + myAdd(restArgs...);
}
//17之后
template<typename ...Args> 
auto myAddEx(const Args& ...args) { 
    return (args + ...); //编译器会这样干：1+(2+(3+(4)))
    或者
    return (... + args); //编译器会这样干：((1+2)+3)+4
    对于加法上述两种表达是等效的，但是减法就不是了。另外，括号是不能省略的
}
cout<<myAddEx(1,2,3,4)<<endl;
```
##### if constexpr
```cpp
template<int  N>
constexpr int fibonacci() {return fibonacci<N-1>() + fibonacci<N-2>(); }
template<>
constexpr int fibonacci<1>() { return 1; }
template<>
constexpr int fibonacci<0>() { return 0; }
/***************************************************/
template<int N>
constexpr int fibonacci(){
	if constexpr(N <= 1)
		return N;
	else
		return fibonacci<N-1>() + fibonacci<N-2>;
}
```
##### 类模板的实参推演
在之前版本的C++中，函数模板可以有显式实例化和隐式实例化两种实例化方式，隐式实例化，编译器会根据实参的类型，推导，然后自动实例化函数模板。而显式的则需程序员指定类型，来让编译器实例化。在类模板中，则只有显式实例化——类模板的构造函数不支持实参推演，例如：
```cpp
std::pair<int, int> p(12,3);
为了方便，于是STL提供了下面这样的函数模板：
auto p = std::make_pair(12,3);
实际上make_pair只是做了这样一件事：
template<typename _T1, typename _T2>
inline pair<_T1, _T2> make_pair(_T1 __x, _T2 __y){ 
	return pair<_T1, _T2>(__x, __y); 
}
利用函数模板的实参推演，确定pair的类型，C++17之后，类模板也支持实参推演了，上述代码就可以这样写了：
std::pair p(10, 0.0);
或者
auto p = std::pair(1,1);
```
##### 嵌套的namespace定义
```cpp
namespace X{
	namespace Y{
		namespace X{

		}
	}
}
-----------------------------------
//C++17:
namespace X::Y::Z{

}
```



### 指针与引用

- 指针
  存放对象地址，本身就是变量，本身有地址，可以改变指针的指向，通过指针指向的地址改变存放的数据
- 引用
  引用是变量的别名，必须初始化，不能改变引用的目标

不存在指向空值的引用，存在指向空值的指针

### C++从代码到二进制文件

1. 预编译：`#deinfe`宏展开，处理预编译指令`#ifdef #include`，过滤注释，`__FUNC__,__LINE__,__FILE__`也在这一步展开
2. 编译：词法分析，语法分析，语义分析，代码优化，生成汇编代码
3. 汇编：把汇编代码转变成机器指令
4. 链接：把不同源文件生成的目标文件进行连链接，形成可执行的程序
   --静态链接
   --动态链接
   ----装入时动态链接
   ----运行时动态链接

### const关键字

const关键字修饰的值不能改变，是只读的（常量不好说）。必须在定义的时候赋值

1. 常量指针
   指针指向一个只读的对象，不能通过指针改变这个对象的值。
   
   ```cpp
   const 数据类型 * 指针变量 = 变量名
   数据类型 const * 指针变量 = 变量名
   ```
   
2. 指针常量
   指针常量指定义的指针是只读，定义时初始化，它的指向的地址不可变，但地址上的数据没有要求

   ```cpp
   数据类型 * const 指针变量 = 变量名
   ```

### define和typedef

> **define**
>
> 1. 字符串替换，没有类型检查
> 2. 预处理阶段展开宏
> 3. 可以防止头文件重复引用
> 4. 不分配内存，给出的是立即数，有多少次就进行多少次替换

> **typedef**
>
> 1. 会进行数据类型的判断
> 2. 编译运行的时候起作用
> 3. 静态存储区中分配控件，运行过程中只有一个拷贝
> 4. 对函数指针类型的支持，原本的函数指针类型是从内向外看，用了typedef之后可以像变量名一样使用函数指针

### define和inline的区别

define是预编译时处理的宏，字符串替换无类型检查，不安全，不受命名空间的影响

inline是将内联函数编译生成的函数体插入被调用的地方，减少了函数调用的开销（函数相关变量的压栈，跳转和返回），inline是一种请求，编译器可以拒绝按照inline的方法

> **inline的限制**
>
> 不能有循环语句
> 不能有条件判断语句
> 函数体不能太大
> 先声明再调用

### 重载与重写/overload override overwrite

|                          | 作用域       | 函数名·参数·返回值                                   | 效果                                                         |
| ------------------------ | ------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
| **overload/重载**        | 同一作用域   | 函数名相同<br />参数不同<br />返回值没有限制         | 编译期间根据调用的参数自动选择使用的哪个函数                 |
| **overwite/重定义/隐藏** | 基类与派生类 | 函数名相同<br />参数和返回值没有要求                 | 子类的函数会隐藏父类的函数，类似于内部的命名空间中的名字有隐藏了上层的同名函数<br />父类函数设置为虚函数，参数相同时遵循==override==规则 |
| **overlaod/覆盖**        | 基类与派生类 | 函数名相同<br />参数相同<br />返回值相同（协变除外） | 参数/返回值相同的父类函数被声明为虚函数<br />调用时遵循动态绑定规则，根据虚函数表进行调用 |



### 协变与逆变

协变是覆盖的特例，返回值不同，但必须是子类或者父类的指针或引用

```cpp
#include <iostream>
using namespace std;

class Person
{
public:
	virtual Person& print()
	{
		cout << "Person" << endl;
		return *this;
	}
protected:
	string _str;

};
class Student :public Person
{
public:
	Student& print()
	{
		cout << "Student" << endl;
		return *this;
	}
protected:
	string _num;
};
void func(Person& p)
{
	p.print();
}
int main()
{
    Person p;
	Student s;
	func(p);//Pserson
	func(s);//Student
    return 0;
}
```

逆变：可以将子类的指针和引用赋给需要父类的变量，STL的智能指针遵循逆变规则，但容器不允许，不能将子类的容器赋给父类容器

关于智能指针的逆变，是因为拷贝构造函数和赋值运算符的设计（这里省略了引用计数），拷贝构造和赋值运算符重载了模板

```cpp
template <typename _Tp>
class shared_ptr
{
public:
  // 将赋值构造函数设置为模板，此时shared_ptr具有了隐式类型转换的能力
  // 第二个模板参数的意义是，检测这种隐式转换是否合理
  // 需要注意的是 _Tp1 和 _Tp 一定是不同的类型，即标准库此外必须另提供正常的复制构造函数
  template <typename _Tp1,
            typename = typename std::enable_if<std::is_convertiable<_Tp1*,_Tp*>::value>::type> 
  shared_ptr(const shared_ptr<_Tp1>& __r) noexcept
            // 注意 __r 与本实例是不同的类型，访问数据需要 friend
      : _M_ptr(__r._M_ptr) { }

  // 赋值操作符
  template <typename _Tp1>
  shared_ptr& operator=(const shared_ptr<_Tp1> __r) noexcept {
    _M_ptr = __r._M_ptr;
    return *this;
  }

private:
  // 尽管因模板参数不同而产生不同的模板实例，彼此可以相互访问 _M_ptr
  template <typename _Tp1> friend class shared_ptr;

  _Tp*  _M_ptr;
};
```

> 协变与逆变用来描述类型转换后的继承关系
>
> ==定义==：如果A、B标识类型f(·)表示类型转换，&le;表示继承关系，A&le;B表示A是由B派生的子类
>
> f(·`)是逆变(contravariant)的时候，当A&le;B时有f(B)&le;f(B)成立
>
> f(·`)是协变(covariant)的时候，当A&le;B时有f(A)&le;f(B)成立
>
> f(·`)是不变(invariant)的时候，当A&le;B时有f(A)与(B)没有继承关系，上述转换均不成立

### new和malloc的区别

|           `new`           |       `malloc`       |             |
| :-----------------------: | :------------------: | ----------: |
| 分配失败抛出bac_alloc异常 |       返回NULL       |    失败返回 |
|  不需要指定大小指定类型   |    指定大小再转换    |  大小和类型 |
|  `new`和`delete`可以重载  | `malloc`和`free`不能 |        重载 |
|   调用了构造和析构函数    |       没有调用       |  构造和析构 |
|          运算符           |         函数         | 运算符/函数 |

### constexpr

#### constexpr和const

const表示“只读”的语义，constexpr表示常量语义。

constexpr只能定义编译期的常量，const可以定义编译期的常量，也可以定义运行期的常量。

#### constexpr

复杂的系统中很难分辨一个初始值是不是常量表达式，可以将变量声明为constexpr类型，有编译期来验证变量的值是否是一个常量表达式。

**必须使用常量初始化**

```cpp
constexpr int n = 20;
constexpr int m =  n + 1;
static constexpr int MOD = 100000007
```

如果constexpr声明了指针，只对指针有效，和所指对象无关

```cpp
constexpr int *p = nullptr; //常量指针，顶层const
const int * q = nullptr;	//指向常量的指针，底层const
int * const q = nullptr		//顶层const
```

#### constexpr函数

constexpr函数是指能用于常量表达式的函数。

函数的返类型和所有的形参类型都是字面值类型，函数体只有一个return语句

```cpp
constexpr int new() {return 24;}
```

#### constexpr构造函数

字面值常量必须提供一个constexpr构造函数

```cpp
class Debug
{
public:
	constexpr Debug(bool b = true) :hw(b), io(b), other(b) {}
	constexpr Debug(bool h, bool i, bool o) : hw(h), io(i), other(o) {}
	constexpr  bool any() const //注意记得在尾部加上const，不然编译错误
	{
		return(hw || io || other);
	}
	void set_io(bool b)
	{
		io = b;
	}
	void set_hw(bool b)
	{
		hw = b;
	}
	void set_other(bool b)
	{
		other = b;
	}
private:
	bool hw;
	bool io;
	bool other;
};
int main()
{
    // constexpr构造函数用于生成constexpr对象以及constexpr函数的参数或返回类型
	constexpr Debug io_sub(false, true, false);
	if (io_sub.any())
	{
		//一些语句
	}

	constexpr Debug prod(false);
	if (prod.any())
	{
		//一些语句
	}
	system("pause");
	return  0;
 
}
```

#### constexpr的好处

1. 为不能修改的数据提供保障，防止意外修改
2. 有些场景编译器可以在编译期对constexpr代码进行优化
3. 相比宏来说，没有额外的开销，更安全可靠

### volatile

类型修饰符，该关键字声明的变量随时可能发生变化，与改变量有关的运算，不要进行编译优化，会从内存中重新装载内容，而不是从寄存器拷贝内容。

不保证原子性

volatile指针与const类似，区分指向volatile变量的指针和volatile的指针

```cpp
const char * cpch;		//指向const数据
volatile char * vpch; 	//指向volatile数据

char* const pchc;		//const的指针
char* volatile pchv;	//volatile的指针	
```

### extern

声明外部变量，在函数或者文件外部定义的全局变量

### static

实现多个对象之间的数据共享和隐藏（类的static成员，注：static成员不能访问`this`指针），使用静态成员不会破坏隐藏原则（单个文件的static变量不会被其他的文件）；默认初始化为0；

### 前置++与后置++

- 后置保留临时对象返回一个右值
- 前置返回自身的引用

```cpp
T& operator++(){		//前置++
	*this+=1;
	return *this;
}
const T operator++(int){	//后置
	T tmp = *this;
	*this += 1;
	return tmp;
}
```

### std::atomic

问：a++和 int a=b在c++中是否线程安全？

答：不安全



### 虚函数与虚继承的size

> https://blog.csdn.net/chczy1/article/details/99896747
>
> 有的文章说虚基类会有虚基类表，gdb调试MinGM的效果如下

多重继承新的虚函数会添加在第一个父类的虚函数表后

正常继承vptr指针不会增加

虚继承的时候还会有新增的vptr指针，（==MinGM的测试结果中，虚函数和虚继承共用这个vptr，不会新增两个，但是补充了父类的虚函数表又有虚继承的话，还是会新增vptr==）

vptr指针在添加vptr指针的那个类的地址的最前面

### C++面向对象

继承和访问权限：`public`、`protected`、`private`

#### 面向对象三大基本特性

1. 封装：数据和代码捆绑在一起，避免外接干扰和不缺确定访问，（目标的数据和行为封装为一个类）
2. 继承：让某种类型的对象获得另一个对象的属性和方法
3. 多态：同一事务表现出不同事务的能力，即向不同对象发送同一消息，不同对象接收时产生不同的行为。

### 虚函数

基类希望派生类定义适合自己的版本，将函数声明为虚函数`virutal`

**多态**：虚函数实现==动态的多态==，使用引用和指针调用虚函数的时候，根据==动态绑定==的类型调用对应的函数。

**关于构造函数**：构造函数不能是虚函数，构造函数中调用虚函数，说是父类的虚函数，实际上是进入函数体之前，对象完成了初始化，子类的构造函数体内虚函数是子类的，父类的构造函数体内的虚函数是父类的虚函数，构建子类的时候，必定构建父类。

### 虚继承

1. 解决多继承的命名冲突和冗余数据
2. 让某个类做出声明，承诺愿意共享它的基类



### 空类

空类的大小至少为1，不为0，为了让空类对象有独一无二的地址，多重继承也是1

含有虚函数表的类会有虚函数表指针。



### 抽象类与接口的实现

C++接口是使用抽象类来实现的

至少一个函数是纯虚函数`=0`，抽象类不能实例化，但是抽象类可以声明成员

### 智能指针

> `share_ptr`共享指针

1. 实现机制
   1. 一个模板指针`T* ptr`指向实际对象
   2. 一个引用计数，通过new创建
   3. 重载`operator*`和`operator->`
   4. 重载copy constructor，引用加一
   5. 重载=运算符，赋值运算符左边的`shared_ptr`对象需要减一，并判断是否需要析构
   6. 重载析构函数，引用计数减一，减一后判断是否为0
2. 线程安全
   1. 同一个shared_ptr被多个线程读是安全的
   2. 同一个shared_ptr被多个线程写是不安全的
   3. 共享引用计数的不同shared_ptr被多个线程写是安全的（前提是不操作指向的内容）

> `unique_ptr`唯一指针

1. unique_ptr“唯一”拥有其指定对象
2. 定义时就绑定在一个new返回的指针上
3. 不支持普通的拷贝和赋值
   可以用release将指针所有权转移到另一个unique上

> `weak_ptr`虚指针

1. 配合shared_ptr引入的指针，不会影响指针的引用计数
2. 和shared_ptr指向相同内存
   shared_ptr析构后内存释放，可以使用lock()检查weak_ptr是否为空指针
3. 成员函数：
   1. `expired()`检测管理对象是否已经释放
   2. `lock()`返回对应的shared_ptr；如果已经释放则返回nullptr
   3. `use_count()`返回引用计数
   4. `reset()`

### C++强制类型转换

> `static_cast`没有运行时类型检查来保证转换的安全性

上行转换（派生类的指针或引用转换为基类）安全，下行转换不安全，基本类型的转换也用这个方法

> `dynamicc_cast`进行下行转时，具有类型检查功能，比`static_cast`安全

类型检查信息在虚函数中，转换后必须是指针、引用或者void*，基类要有虚函数，可以交叉转换，转换失败返回空指针，引用转换失败会抛出异常。

> `reinterpret_cast`，对指针所指向的地址重新解释，甚至可以将数据指针转换为函数指针，这种情况下调用时会报错

> `const_cast`常量指针转换为非常亮指针，常量引用转换为非常亮引用，去掉==const==和==volatile==属性

### 内存模型

>  字符串操作函数

1. `strcpy(dest,src)`，把src复制到dest，碰到‘\0’停止（会拷贝结束符）
2. `strlen(str)`计算长度，断言非空，不算结束符
3. `strcat(dest, src)`src拼接到dest结尾处
4. `strcmp(str1, str2)`比较str1和str2，执行str1-str2的第一个不同字符

### 内存泄露

程序未释放掉不在使用的内存，这些内存被分配给了程序，但失去了对这部分内存的控制，造成了内存的浪费

追踪内存的工具==mtrace==和==valgrind==

#### 内存泄露的分类

1. 堆内存泄露 malloc，realloc和new分配之后没有释放
2. 系统资源泄露，handle,Socket等没有释放掉
3.  基类的析构函数定义为虚函数

### 构造函数、析构函数声明为虚函数

构造函数不能声明成虚函数

析构函数在派生时必须声明成虚函数

### 指针和数组的sizeof

```cpp
char str[] = "hello";	//6数组大小，包括结束符
char*p = str;			//指针长度
int n = 10;				//数据类型大小
void Func(char str[1000]){
	sizeof(str);		//数组转到是数组名，是地址的大小
}
void * p = malloc(100);	//指针大小
```

### C++内存空间

![](E:\短暂视界\面经\八股git\mianshibagu\img\C++内存.jpg)

这里注明了命令行参数和环境变量在栈区和内核空间之间

### `new` `delete` `malloc` `free`

```cpp
int p = new int[10];
```

之后，数组长度会保存在 p所指位置之前的空间中，delete[]以此判断如何删除

重载delete和new

```cpp
void* operator new(size_t sz);
void* operator new[](size_t sz);

void operator delete(void* m);
void operator delete[](void* m);

//访问原始的new和delete
::new
::delete
```

### ~~计算机乱序执行~~