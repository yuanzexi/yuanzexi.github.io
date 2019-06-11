## 型别推导
----

### 【条款1】 理解模板型别推导

```c++
// 模板函数定义
template <typename T>
void f(ParamType param);

// 调用
f(expr)
```
在编译期，编译器会通过`expr`推导两个型别，`T`和`ParamType`的型别，这两个型别往往不一样，因为`ParamType`会包含一些饰词。 型别推导有以下三种情况:    

1. `ParamType`是个指针或者引用，但不是万能引用
- 若`expr`的引用性会在推导中被忽略，忽略引用性之后，再对`expr`的型别和`ParamType`的型别执行模式匹配，由此来决定`T`的型别。 
- `expr`的指针性也会被忽略。 
2. `ParamType`是个万能引用
- 若`expr`是个右值，按情况1进行推导; 
- 若`expr`是个左值，则`T`和`ParamType`都会被推导成左值引用。 
3. `ParamType`既非指针也非引用
- 不仅`expr`的引用性会被忽略，其`const`和`volatile`性质都会被忽略。 

另外，在模板型别推导中，数组或者函数型别的实参会退化成对应的指针，除非它们被用来初始化引用 (即形参带引用修饰符)。 

----------

### 【条款2】 理解 auto 型别推导

```C++
const auto& rx = x;
```
1. 一般情况下，`auto` 型别推导与模板型别推导时一模一样的，将`ParamType`换成型别饰词即可。
2. 不同的是:
- `auto`型别推导会假定用大括号括起的初始化表达式代表一个`std::initializer_list`，但模板型别推导不会。
- **(C++ 14 特性)** 在函数返回值或`lambda`表达式的形参中使用`auto`，意思是使用模板型别推导，而不是`auto`型别推导。

----------

### 【条款3】 理解 decltype

```
// C++ 14
template <typename Container，typename Index>
decltype(auto)
authAndAccess(Container&& c，Index i)
{
    authenticateUser();
    return std::forward<Constainer>(c)[i];
}

// C++ 11
template <typename Container，typename Index>
auto authAndAccess(Container&& c，Index i)
-> decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```
`decltype(expr)`： 返回`expr`这个名字或表达式的型别。  

- 绝大多数情况下，`decltype`会得出变量或表达式的型别而不做任何修改，例如忽略引用性。
- 对于型别为`T`的左值表达式，除非该表达式仅有一个名字，`decltype`总是得出型别`T&`。
- **(C++ 14 特性)** 支持`decltype(auto)`，和`auto`一样，它会从其初始化表达式出发来推导型别，但他的型别推导遵循`decltype`的规则。 

----------

### 【条款4】 掌握查看型别推导结果的方法
- 撰写代码阶段: IDE编辑器
- 编译阶段: 编译器诊断信息
- 运行时阶段: 运行时输出，
    - 使用 `typeid(T)。name()`，引用性，常量性等会被消除
    - 使用 `Boost。TypeIndex`中的`type_id_with_cvr<T>()。pretty_name()`
- 靠自己理解型别推导是必要的

----------

## auto

----------

### 【条款5】 优先选用 auto，而非显式型别声明
- `auto`变量必须初始化，基本上对会导致兼容性和效率问题的型别不匹配现象免疫，还可以简化重构流程，以及少码一些字。
- `auto`型别的变量有条款2和6中所描述的毛病。

----------

### 【条款6】 当 auto 推导的型别不符合要求时，使用带显式型别的初始化物习惯用法
- “隐形”的代理型别可以导致`auto`根据初始化表达式推导出“错误的”型别，例如`std::vector<bool>`的`operator[]`返回的是`std::vector<bool>::reference`而不是`bool`。
- 带显式型别的初始化物习惯用法强制`auto`推导出想要的型别，如：

```c++
auto index = static_cast<int>(d * c.size())
```

----------
## 转向现代C++

----------

### 【条款 7】 在创建对象时注意区分 () 和 {} 
- 大括号初始化可以应用的语境最为宽泛，可以阻止隐式窄化型别转换，还对最令人苦恼之解析语法免疫。

```c++
double x, y, z;
int sum1{x + y + z}; // 错误！
int sum1(x + y + z); // 表达式的值被截断为int

// 最令人苦恼之解析语法
Widget w1(10); // 调用构造函数，传入形参10
Widget w2(); // 该语句会声明一个名为w2，返回一个Widget型别对象的函数
```
- 在构造函数重载决议期间，只要有任何可能，大括号初始化物就会与带有`std::initializer_list`型别的形参相匹配，即使其他重载版本有着貌似更加匹配的形参表。
- 使用小括号还是大括号，会造成结果大相径庭的一个例子是：使用两个实参来创建一个`std::vector<int>`对象。

```c++
std::vector<int> v1(10, 30); // 10个值为30的元素
std::vector<int> v2{10, 20}; // 元素：10，20
```
- 在模板内容进行对象创建时，到底应该使用小括号还是大括号会成为一个棘手问题。（更具弹性的设计，也就是允许调用者自行决定在从模板生成的函数内使用小括号还是大括号的设计的实现，可见Andrzej的2013年的文章“Intuitive interface - Part I”）

----------

### 【条款 8】 优先选用 nullptr，而非 0 或 NULL
 - `0` 和 `NULL` 都不具备指针型别
 - `nullptr` 不具备整型型别，虽然也不具备指针型别，但可以把它想成一种任意型别的指针，其实际型别为`std::nullptr_t`，该型别可隐式转换到所有的裸指针型别。
 - 避免在整型和指针型别之间重载
 - `nullptr`在模板内的表现更亮眼

----------

### 【条款 9】 优先选用别名声明，而非 typedef
- 别名声明可以模板化，但`typedef`不行

```c++
// 别名声明
template <typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

MyAllocList<Widget> lw;
// typedef
template <typename T>
struct MyAllocList {
    typedef std::list<T, MyAlloc<T>> type;
};

MyAllocList<Widget>::type lw;

```
- 别名模板可以让人免写“`::type`” 后缀，并且在模板内，对于内嵌`typedef`的引用经常要求加上`typename`前缀。

----------

### 【条款 10】 优先选用限定作用域的枚举型别，而非不限作用域的枚举型别
- 通用规则：在一对大括号内声明一个名字，该名字的可见性就被限定在括号括起来的作用域内。但是，C++98风格的枚举型别定义却不遵循该规则。

```c++
enum Color {black, white, red}; // black, white, red 的所在作用域和 Color 相同

auto white = false; // 错误， white 已在范围内被声明过。
```
- C++ 11中的限定作用域枚举型别，则不会泄露名字到外部作用域。即，**限定作用域的枚举型别带来的名字空间污染低**。

```c++
enum class Color {black, white, red}; //black, white, red 所在作用域被限定在 Color 内
auto white = false; // 正确

Color c = white; // 错误，范围内并无名为 “white” 的枚举量
Color c = Color::white; // 正确
```
- 限定作用域的枚举型别的枚举量是更强型别的（strongly typed），而不限作用域的枚举型别则可以隐式转换到整数型别（甚至进一步转换到浮点型别）。
- 限定作用域的枚举型别可以直接进行前置声明，而不限作用域的枚举型别需要经过特殊处理——指定底层型别。

```c++
enum Color; // wrong
enum class Color; // true

enum class Status; // 底层型别是 int
enum class Status: std::uint32_t; // 底层型别是 uint32_t

enum Color: std::uint8_t // true, 底层型别是 uint8_t
```
- 默认情况下，限定作用域的枚举型别的底层型别是int，而不限作用域的枚举型别的底层型别则根据定义选取合适的型别，这也是为何其不直接支持前置声明的原因。

----------

### 【条款 11】 优先选用删除函数，而非 private 未定义函数
C++ 98中为了阻止一些函数被调用，采取的做法是声明其为 `private`，并且不去定义他们。但这种做法无法阻止它们的成员函数，或类的友元去使用它们，这会导致链接阶段因缺少函数定义而宣告失败。另外，当尝试调用某个成员函数时，C++会先教研可访问性，后校验删除状态，这会让调用用户抱怨该函数不可访问。
C++ 11中，可用 “`= delete`”来阻止函数被使用。此种做法有以下几个优点：

 - 任何函数都能成为删除函数，但只有成员函数能声明成`private`。

```c++
bool isLucky(int number); 

bool isLucky(char) = delete; // 拒绝 char 型别
bool isLucky(bool) = delete; // 拒绝 bool 型别
bool isLucky(double) = delete; // 拒绝 double 和 float 型别，当 float 面临转型到 int 还是 double 时，C++ 会优先转型到 double。
```
- 删除函数能阻止不应该进行的模板具现。
```c++
template <typename T>
void processPointer(T* ptr);

template <>
void processPointer<void>(void*) = delete;

template <>
void processPointer<char>(char*) = delete;
```

----------

### 【条款 12】 为意在改写的函数添加 override 声明
通过基类接口调用派生类函数，有一系列要求必须满足：

 - 基类中的函数必须是虚函数。
 - 基类和派生类中的函数名字必须完全相同（析构函数例外）。
 - 基类和派生类中的函数形参型别必须完全相同。
 - 基类和派生类中的函数常量性必须完全相同。
 - 基类和派生类中的函数返回值和异常规格必须兼容。
 - 基类和派生类中的函数引用饰词必须完全相同。
    - 成员函数引用饰词是为了实现限制成员函数仅用于左值或右值。

```c++
class Widget{
public:
    void doWork() &; //该版本doWork仅在 *this 是左值时调用
    void doWork() &&; //该版本doWork仅在 *this 是右值时调用
}
```
C++ 11中添加 `override` 声明不仅可以让编译器帮忙检查上述要求，而且也可以让开发者自己明确改写意向。

 - `override`仅出现在成员函数声明的末尾时才有关键字意义

----------

### 【条款 13】 优先选用 const_iterator，而非 iterator
- 当你需要一个迭代器而其指涉到的内容没有修改必要，就应该使用 `const_iterator`。
- 在最通用的代码中，优先选用非成员函数版本的 `begin`、`end`和`rbegin`等，而非其成员函数版本。

----------

### 【条款 14】 只要函数不会发射异常，就为其加上 noexcept 声明

 - `noexcept`声明是函数接口的组成部分，这意味着调用方可能会对其有依赖，所以使用该声明需谨慎，必须确保该函数一定不会抛出异常。
 - 相对于不带 `noexcept` 声明的函数，带有 `noexcept` 声明的函数有更多机会得到优化。
 - `noexcept`性质对于移动操作、swap、内存释放函数和析构函数最有价值。
 - 大多数函数时异常中立的（即此类函数自身并不抛出异常，但它们调用的函数则可能会发射异常），不具备 `noexcept` 性质。
 
----------

### 【条款 15】 只要有可能使用 constexpr，就使用它

```c++
constexpr
int pow(int base, int exp) noexcept
{
    return (exp == 0 ? 1 : base * pow(base, exp - 1));
}

constexpr auto numCouds = 5;
std::array<int, pow(3, numConds)> results;
```
- `constexpr` 对象都具备`const`属性，并由编译期已知的值完成初始化。
- `constexpr`函数在调用时，若传入的实参值是编译期已知的，则会产生编译期结果；若传入的实参值有一个或多个是编译期未知的，则它的运作方式与普通函数无异，即在运行期执行结果运算。
- C++ 11中，`constexpr`函数不得包含多余一个可执行语句，即一条 `return` 语句。C++ 14中，上述限制被放宽了，允许多条语句。
- 只要有可能使用`constexpr`，就使用它。但该使用必须有个长期保证，否则日后想移除该使用，则可能会导致大规模的编译拒绝灾难。

----------

### 【条款 16】 保证 const 成员函数的线程安全性

```c++
class Polynomial {
public:
    using RootsType = std::vector<double>;
    // 返回 缓存的计算值
    RootsType roots() const{
        if (!rootsAreValid){
            
            // 计算多项式的根，存于rootVals
            
            rootsAreValid = true;
        }
        return rootVals;
    }
private:
    mutable bool rootsAreValid{false};
    mutable Rootstype rootVals {};
}
```
上述代码线程不安全，解决上述线程不安全问题有以下两种：
- 对于单个要求同步的变量或内存区域，使用`std::atomic`就足够了。`std::atomic`的操作比`mutex`的操作开销小。

```c++
// 计算调用次数
class Point{
public:
    double distanceFromOrigin() const noexcept{
        ++callcount;
        return std::sqrt((x * x) + (y * y));
    }
    
private:
    mutable std::atomic<unsigned> callcount {0};
    double x, y;
}
```
- 如果有两个或更多个变量或内存区域需要作为一整个单位进行操作时，最好使用 `mutex` 互斥量。

```c++
class Widget{
public: 
    int magicValue() const{
        std::lock_guard<std::mutex> guard(m);
        
        if (cacheValid) return cachedValue;
        else{
            auto val1 = expensiveComputation1();
            auto val2 = expensiveComputation2();
            cachedValue = val1 + val2;
            cacheValid = true;
            return cachedValue;
        }
    }
private:
    mutable std::mutex m;
    mutable int cachedValue;
    mutable bool cacheValid {false};
}
```

----------

### 【条款 17】 理解特种成员函数的生成机制
在 C++ 官方用语中，特种成员函数是指那些C++会自行生成的成员函数：

- 默认构造函数
    - 与C++ 98的机制相同，仅当类中不包含用户声明的构造函数时才生成。
- 析构函数
    - 与C++ 98的机制基本相同，仅当基类的析构函数为虚的，派生类的析构函数才是虚的。唯一的区别在于析构函数默认为`noexcept`（参见条款 14）。
- 复制构造函数
    - 运行期行为与C++ 98相同：按成员进行非静态数据成员的复制构造。仅当类中不包含用户声明的复制构造函数时才生成。
    - 如果该类声明了移动操作，则复制构造函数将被删除。
    - 在已经存在复制赋值运算符或析构函数的条件下，仍然生成复制构造函数已经成为了被废弃的行为。
- 复制赋值运算符
    - 运行期行为与C++ 98相同：按成员进行非静态数据成员的复制赋值。
    - 仅当类中不包含用户声明的复制赋值运算符时才生成。
    - 如果该类声明了移动操作，则复制赋值运算符将被删除。
    - 在已经存在复制构造函数或析构函数的条件下，仍然生成复制赋值运算符已经成为了被废弃的行为。
- 移动构造函数和移动赋值运算符 (C++ 11)
    - 都按成员进行非静态数据成员的移动操作。
    - 仅当类中不包含用户声明的复制操作、移动操作和析构函数时才生成。

这些函数仅在需要时才会生成，即在某些代码中使用了它们，而在类中并未显式声明的场合。生成的特种成员函数都具有 public 访问层级且是 inline 的，而且它们都是非虚的，除了析构函数（当基类的析构函数是虚函数时，派生类生成的析构函数也是虚函数）。

- 大三律：如果你声明了复制构造函数、复制赋值运算符，或析构函数中的任何一个，你就得同时声明所有这三个。
    - 如果有改写复制操作的需求，往往意味着该类需要执行某种资源管理。
- 成员函数模板在任何情况下都不会抑制特种成员函数的生成。

```c++
// 成员函数模板
class Widget{
    template <typename T>
    Widget(const T& rhs); // 以任意型别构造Widget
    
    template <typename T>
    Widget& operator=(const T& rhs); // 
```

----------
## 智能指针

----------

### 【条款 18】 使用 std::unique_ptr 管理具备专属所有权的资源
- 在默认情况下，`std::unique_ptr`和裸指针有着相同的尺寸，且对于大多数操作，包括提领（dereference，解析），它们二者都是精确地执行了相同的指令。即，`std::unique_ptr`基本能应用于裸指针的使用场景。
- `std::unique_ptr`是小巧、高速的，具备只移型别（无法复制）的智能指针，对托管资源实施专属所有权语义。
- 默认情况下，资源析构采用`delete`运算符来实现，但可以指定自定义删除器，有状态的删除器和采用函数指针实现的删除器会增加`std::unique_ptr`型别的对象尺寸，而采用无状态（即无捕获）的lambda表达式则不会浪费任何存储尺寸。

```c++

class Investment {
public:
    virtual ~Investment();
};

class Stock: public Investment{}

class Bond: public Investment{}

class RealEstate: public Investment{}

auto delInvmt = [](Investment* pInvestment){
    makeLogEntry(pInvestment);
    delete pInvestment;
};

// 工厂函数，可用于Pimpl习惯用法
template <typename... Ts>
std::unique_ptr<Investment, decltype(delInvmt)> makeInvestment(Ts&&... params){
    std::unique_ptr<Investment, decltype(delInvmt)> pInv(nullptr, delInvmt);
    
    if (isStock){
        pInv.reset(new Stock(std::forward<Ts>(params)...));
    }else if (isBond){
        pInv.reset(new Bond(std::forward<Ts>(params)...));
    }else if (isRealEstate){
        pInv.reset(new RealEstate(std::forward<Ts>(params)...));
    }
    return pInv;
}
```
- `std::unique_ptr`以两种形式提供
    - 单对象：`std::unique_ptr<T>`
    - 数组：`std::unique_ptr<T[]>`
- `std::unique_ptr`可以直接转换成`std::shared_ptr`。

```c++
std::shared_ptr<Investment> sp = makeInvestment(arguments);
```

----------

### 【条款 19】 使用 std::shared_ptr 管理具备共享所有权的资源
- `std::shared_ptr`提供方便的手段，实现了任意资源在共享所有权语义下进行生命周期管理的垃圾回收。
- `std::shared_ptr`的尺寸是裸指针的两倍：一个裸指针和一个指向该资源的控制块（引用计数，弱计数，其他数据，如自定义删除器，分配器等）的裸指针。引用计数的内存必须动态分配，使用`std::make_shared`可以避免动态分配的成本。
- 引用计数的递增和递减必须是原子性操作。
- 避免使用裸指针型别的变量来创建`std::shared_ptr`指针。如果用同一个裸指针去构造多个`std::shared_ptr`，会创造多重控制块，从而引发多重析构灾难。
- `std::enable_shared_from_this<T>` 可用于解决涉及`this`指针的多重控制块灾难。

```c++
// 追踪被处理过的 Widgets
std::vector<std::shared_ptr<Widget>> processedWidgets;

class Widget {
public:
    void process();
}

void Widget::process(){
    processedWidgets.emplace_back(this); // 引发多重控制块灾难
}

// 以下类定义可以避免上述灾难
class Widget : public std::enable_shared_from_this<Widget> {
    ...
}
```
- 默认的资源析构通过`delete`运算符进行，但同时也支持定制删除器，删除器的型别对`std::shared_ptr`的型别没有影响。

```c++
auto loggingDel = [](Widget* pw){
    makeLogEntry(pw);
    delete pw;
}

std::unique_ptr<Widget, decltype(loggingDel)> upw(new Widget, loggingDel); // 析构器型别是智能指针的一部分

std::shared_ptr<Widget> spw(new Widget, loggingDel); // 析构器型别不是智能指针的一部分
```
- `std::shared_ptr`只能处理单对象而不能处理数组，即无法声明`std::shared_ptr<T[]>`。对应地，可结合`std::array`、`std::vector`等替代使用。

----------

### 【条款 20】 对于类似 std::shared_ptr 但有可能空悬的指针使用 std::weak_ptr
- `std::wead_ptr` 不能提领（dereference，引用解析），也不能检查是否为空，它是`std::shared_ptr`的扩充，而不是一个独立使用的智能指针。
- `std::weak_ptr`可能的用武之地：缓存，观察者列表，以及避免`std::shared_ptr`指针环路。

----------

### 【条款 21】 优先选用 std::make_unique 和 std::make_shared，而非直接使用 new
- `std::make_shared`是C++ 11就存在的，而`std::make_unique`是C++ 14才存在的。
- 相比于直接使用 `new` 表达式，`make`系列函数消除了重复代码，改进了异常安全性，并且对于 `std::make_shared`和`std::allocated_shared`而言，生成的目标代码会尺寸更小，速度更快（`new`是先分配对象内存，再分配一次控制块内存，而`make_shared`只进行一次内存分配，分配单块内存保存对象内存机器相关联的控制块）。
- 不适用于`make`系列函数的场景：需要定制删除器，以及期望直接传递大括号初始化物。
- 对于`std::shared_ptr`，不建议使用`make`系列函数的额外场景：
    - 自定义内存管理的类
    - 内存紧张的系统、非常大的对象、以及存在比指向相同对象的`std::shared_ptr`生存期更久的`std::weak_ptr`。
    

----------

### 【条款 22】 使用 Pimpl 习惯用法时，将特殊成员函数的定义放到实现文件中
- Pimpl（pointer to implementation）：把某类的数据成员用一个指向某实现类（或结构体）的指针替代，然后把原来在主类中的数据成员放置到实现类中，并通过指针间接访问这些数据成员。

```c++
// 裸指针用法
// “widget.h” 头文件内
class Widget{
public:
    Widget();
    ~Widget();
private:
    struct Impl; 
    Impl *pImpl;
}

// “Widget.cpp” 实现文件内
struct Widget::Impl {
    std::string name;
    std::vector<double> data;
    Gadget g1,g2,g3;
};

Widget::Widget(): pImpl(new Impl) {}

Widget::~Widget() { delete pImpl;}
```
- 对于采用`std::unique_ptr`来实现的`pImpl`指针，须在类的头文件中声明特种成员函数，但在实现文件中实现它们。即使默认函数实现有着正确行为，也必须这样做。该规范仅适用于`std::unique_ptr`，而不适用于`std::shared_ptr`。

```c++
// std::unique_ptr 用法
// “widget.h” 头文件内
class Widget{
public:
    Widget();
    ~Widget();
private:
    struct Impl; 
    std::unique_ptr<Impl> pImpl;
}

// “Widget.cpp” 实现文件内
struct Widget::Impl {
    std::string name;
    std::vector<double> data;
    Gadget g1,g2,g3;
};

Widget::Widget(): pImpl(std::make_unique<Impl>()) {}

Widget::~Widget() = default; // 定义在 Impl 之后，使得析构函数内 Impl 是可见并被声明定义的
```

----------
## 右值引用、移动语义和完美转发

- 移动语义：使得编译器得以使用不那么昂贵的移动操作，来替换昂贵的复制操作。移动语义也使得创建只移型别对象成为可能，如 `std::unique_ptr`，`std::future`和`std::thread`等
- 完美转发：使得人们可以撰写接受任意实参的函数模板，并将其转发到其他函数，目标函数会接受到与转发函数所接受的完全相同的实参。
- 右值引用：将上述两种语言特性胶合起来的底层语言机制，它使得移动语义和完美转发成为了可能。
- 形参总是左值，即使其型别是右值引用，如`void f(Widget&& w);`中的形参`w`是个左值。

----------
### 【条款 23】 理解 std::move 和 std::forward
- `std::move` 不进行任何移动， `std::forward` 也不进行任何转发，二者在运行期都无所作为，不会产生任何可执行代码，连一个字节都不会生成。二者本质上是执行强制型别转换的函数模板。
- `std::move`无条件地将实参强制转换成右值。
    - 针对常量对象执行的移动操作将自动变换成复制操作。
    - `std::move`不仅不实际移动任何东西，甚至不保证经过其强制型别转换后的对象具备可移动的能力。
- `std::forward`是有条件的强制型别转换：仅当其实参是使用右值完成初始化时（即实参被绑定到右值时），它才会执行向右值型别的强制型别转换。
    - 其本质上是保持对象原始型别的左值性和右值性，方便用于对象转发。

----------

### 【条款 24】 区分万能引用和右值引用
- 万能引用（universal reference，形如`T&&`）：既可以到左值，也可以绑定到右值，甚至`const`和`volatile`对象，即可以绑定到万事万物。万能引用会在以下两种场景下现身：
    - 函数模板的形参具备`T&&`型别，且`T`的型别由推导而来。若型别声明并不精确地具备`type&&`的形式，或者型别推导未发生，则`type&&`就代表右值引用。
    - auto声明，形如`auto&&`

```c++
template <typename T>
void f(T&& param) // param 是一个万能引用

auto&& var2 = var1; // var2 是一个万能引用

template <typename T>
void f(std::vector<T>&& param) // param 是一个右值引用

template <typename T>
void f(const T&& param) // param 是一个右值引用
```

----------

### 【条款 25】 针对右值引用实施 std::move，针对万能引用实施 std::forward
- 针对右值引用的最后一次使用实施`std::move`，针对万能引用的最后一次使用实施`std::forward`。
- 作为按值返回的函数的右值引用和万能引用，也遵循上一条的规则。
- 若局部对象可能适用于返回值优化（RVO），则请勿对其实施`std::move`或`std::forward`。

----------

### 【条款 26】 避免依万能引用型别进行重载
- 形参为万能引用的函数，是C++中最贪婪的。它们会在具现过程中，和几乎任何实参型别都会产生精确匹配（条款 30 描述了几种例外情况）。
- 把万能引用作为重载候选型别，几乎总会让该重载版本在始料未及的情况下呗调用到。
- 完美转发构造函数的问题尤其严重，因为对于非常量的左值型别而言，它们一般都会行成相对于复制构造函数的更佳匹配，并且它们还会劫持派生类中对基类的复制和移动构造函数的调用。

```c++
class Person{
public:
    template <typename T>
    explicit Person(T&& n) : name(std::forward<T>(n)) {} // 完美转发构造函数
    
    explicit Person(int idx); // 形参为 int 的构造函数
    
    Person(const Person& rhs); // 复制构造函数
    
    Person(Person&& rhs); // 移动构造函数
}

Person p("Nancy");
auto cloneOfP(p); // 调用的是完美转发构造函数，而不是所预想的复制构造函数，产生错误

const Person p2("Nancy");
auto cloneOfP(p2); // 调用得是预想的复制构造函数，因为 const 的存在，使得其可以精确匹配。
```

----------

### 【条款 27】 熟悉依万能引用型别进行重载的替代方案
- 舍弃重载：使用彼此不同的函数名
- 传递 const T& 型别的形参：如条款 26
- 传值：把传递的形参从引用型别替换成值型别。
- 标签分派：创造适当的标签对象，根据重载实现函数发起的调用把工作“分派”到正确的重载版本的手法。`std::false_type`和`std::true_type`就是所谓的“标签”，这类形参没有名字，在运行期也不起任何作用。

```c++
// 原始版本
std::multiset<std::string> names;

template <typename T>
void logAndAdd(T&& name){
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAdd");
    names.emplace(std:forward<T>(name));
}

// 标签分派 版本
template <typename T>
void logAndAdd(T&& name){
    logAndAddImpl(std::forward<T>(name), std::is_integral<typename std::remove_reference<T>::type>());
}

template <typename T>
void logAndAddImpl(T&& name, std::false_type){
    auto now = std::chrono::system_clock::now();
    log(now, "logAndAd");
    names.emplace(std::forward<T>(name));
}


std::string nameFromIdx(int idx); // 根据idx查询name的函数

void logAndAddImpl(int idx, std::true_type){
    logAndAdd(nameFromIdx(idx));
}
```

- 对接受万能引用的模板施加限制
    - `std::enable_if`  可以强制编译器表现出来的行为如同特定的模板不存在一般。默认情况下，所有的模板都是启用的；但实施了`std::enable_if`的模板，只会在满足了`std::enable_if`指定的条件的前提下才会启用。（运作原理机制，参见`std::enable_if` 和 “SFINAE”）
    - `std::is_same<Person, T>::value`：判定两个型别是否同一。
    - `std::decay<T>::type`：其返回值与`T`的型别相同，但移除了`T`型别带有的所有引用，`const`和`volatile`饰词。该方法同样可用于把数组和函数型别转型成指针型别。
    - `std::is_base_of<T1, T2>::value`：判定一个型别是否是由另一个型别派生而来。注意，所有型别都可以认为是从它自身派生而来。

```c++
// c++ 11版本
class Person{
public:
    template <typename T, 
                typename = std::enable_if<
                    !std::is_base_of<Person, typename std::decay<T>::type>::value 
                    && 
                    !std::is_integral<typename std::remove_reference<T>::type>::value
                    >::type
             >
    explicit Person(T&& n): name(std::forward<T>(n)) {...} // 接受 std::string 型别以及可以强制转型到 std::string 的实参型别的构造函数
    
    explicit Person(int idx): name(nameFromIdx(idx)) {...}  // 接受整型实参的构造函数
private:
    std::string name;
}

// c++ 14 简化版本, 可用后缀 _t 来替代 ::type 和 typename
class Person{
public:
    template <typename T, 
                typename = std::enable_if_t<
                    !std::is_base_of<Person, std::decay_t<T>>::value 
                    && 
                    !std::is_integral<std::remove_reference_t<T>>::value
                    >
             >
    explicit Person(T&& n): name(std::forward<T>(n)) {...} // 接受 std::string 型别以及可以强制转型到 std::string 的实参型别的构造函数
    
    explicit Person(int idx): name(nameFromIdx(idx)) {...}  // 接受整型实参的构造函数
private:
    std::string name;
}
```

- 上述方法的比较
    - 前三种技术需要对带调用的函数形参逐一指定型别，而后两种技术则利用了完美转发，无须指定形参型别。
    - 完美转发效率更高，因为它出于和形参声明时的型别严格保持一致的目的，会避免创建临时对象。
    - 完美转发经过数层转发调用，若最终在匹配时有误，则会给出繁杂的错误信息，让人难以捉摸。可结合断言方法来检测。
    - `std::is_constructible<T1, T2>::value`：在编译器间判定具备某个型别的对象是否从另一型别（或另一组型别）的对象（或另一组对象）出发完成构造的。
    - `static_assert(condition, message)`：静态断言

----------

### 【条款 28】 理解引用折叠
- 模板形参 `T` 的推导结果型别中，会把传给`param`的实参是左值还是右值的信息给编码进去：如果传递的实参是个左值，`T`的推导结果就是个左值引用型别；如果传递的实参是个右值，`T`的推导结果就是个非引用型别。
- 开发者被禁止声明引用的引用，但编译器可以在特殊的语境中产生引用的引用，两种引用（左值引用和右值引用）会有四种引用组合，此时会对应地触发引用折叠机制，引用折叠规则如下：
    - 如果任一引用为左值引用，则结果为左值引用。
    - 否则（即两个皆为右值引用），结果为右值引用。
- 引用折叠会在四种语境中发生：模板实例化；`auto`型别生成；创建和运用`typedef`和别名声明；以及`decltype`。
- 万能引用就是在型别推导的过程会区别左值和右值，以及会发生引用折叠的语境中的右值引用。

----------

### 【条款 29】 假定移动操作不存在，成本高，未使用
- 该条款描述的是，在日常进行通用代码开发时，我们最好假定移动操作不存在，成本高，未使用。
- 仅在已知代码中会使用的型别，并肯定它们的特性不会改变，确保涉及的型别能够提供成本低廉的移动操作，且是要在这些移动操作会被调用执行的语境中使用该对象，我们才可以放心大胆地依靠移动语义来替换复制操作。
- 该条款基于以下事实前提，在以下几个场景下，C++ 11的移动语义不会带来什么好处：
    - 没有移动操作：待移动的对象未能提供移动操作。从而，移动操作变成了复制请求。
    - 移动未能更快：待移动的对象虽有移动操作，但并不比其复制操作更快。
        - `std::array`的移动，因其数据直接存储在对象内部，而不像其他容器一样是以指针指向从而可以直接替换指向指针，必须得直接移动数据，因此移动操作并不会比复制快。
        - `std::string`基本都实现了小型字符串优化（small string optimization，SSO）。SSO优化以后会将“小型”字符串（例如不超过15个字符）存储在`std::string`对象内的某个缓冲区内，而不去使用在堆上的分配的存储。
    - 移动不可用：移动本可以发生的语境下，要求移动操作不可发射异常，但该操作未加上 `noexcept` 声明。

----------

### 【条款 30】 熟悉完美转发的失败情形
```c++
template <typename... Ts>
void fwd(Ts&&... params){
    f(std::forward<Ts>(params)...);
}
```
- 模板型别推导失败
    - 大括号初始化物，因为型别推导无法推导`std::initializer_list`。但可以借用`auto`推导中转。
    - 仅有声明的整型 `static const` 成员变量，因为编译器不会为static const 成员变量保留内存，只会对其实施常数传播。即，涉及到对`static const`成员变量取址的操作会导致链接期发生错误，而引用和指针本质上是一样的。但只需要提供`static const`成员变量的定义即可使其通过编译和链接。
    - 重载的函数名字和模板名字，因为对于重载的函数名字和模板名字，无法推导其型别。
    - 位域的使用，因为不存在指向位域的指针，故同样也无法使用指向位域的引用。
- 模板型别推导结果错误
    - `0` 和 `NULL` 用作空指针
    

----------

## lambda 表达式
- 闭包：lambda表达式创建的运行期对象，根据不同的捕获模式，闭包会持有数据的副本或引用。
- 闭包类：实例化闭包的类。每个lambda表达式都会触发编译器生成一个独一无二的闭包类，而闭包中的语句会变成它的闭包类成员函数的可执行命令。
- 捕获列表只用于局部非 static 变量，lambda 可以直接使用局部 static 变量和在它所在函数之外声明的名字。

----------

### 【条款 31】 避免默认捕获模式
- C++ 11中有两种默认捕获模式：
    - 按引用捕获：
        - 可能会导致闭包包含指向局部变量的引用，或者指向定义 lambda 式的作用域内的形参的引用。此种情况下，会导致空悬指针问题。
        - 显式地列出 lambda 式所依赖的局部变量或形参是更好的软件工程实践。
    - 按值捕获：
        - 极易受空悬指针的影响（尤其是`this`），因为按值捕获指针，会使闭包中持有该指针副本，但无法保证该指针在闭包外的地方释放资源，从而导致空悬指针问题。
        - lambda 式不仅依赖于局部变量和形参（它们可以被捕获），还会依赖于静态存储期（static storage duration）对象。若不意识到这一点，可能会被误导认为 lambda 式是自洽的，即以为自己捕获了被声明为静态变量的局部变量，而实际上并未捕获，闭包内真正使用的是静态变量的内容，而非以为被捕获的那个变量。

----------

### 【条款 32】 使用初始化捕获将对象移入闭包
- C++ 14中添加了初始化捕获（又名，广义 lambda 捕获），该方式可以在捕获框内使用表达式初始化一个闭包内使用的成员变量。
```c++
auto pw = std::make_unique<Widget>();
auto func = [pw = std::move(pw)]{
    return pw->isValidated() && pw->isArchived();
}
auto func1 = [pw = std::make_unique<Widget>()]{
    return pw->isValidated() && pw->isArchived();
}
```
- C++ 11中没有初始化捕获，但可以使用`std::bind`来模仿。`std::bind`返回的函数对象称为**绑定对象**。
    - 移动构造一个对象入C++ 11闭包是不可能实现的，但移动构造一个对象入绑定对象则是可能实现的。
    - 欲在C++ 11中模拟移动捕获包括以下步骤：先移动构造一个对象入绑定对象，然后按引用把该移动构造所得的对象传递给 lambda 式。
    - 因为绑定对象的声明周期和闭包相同，所以针对绑定对象中的对象和闭包里的对象可以采用同样手法加以处置。

```c++
std::vector<double> data;
auto func = std::bind([](const std::vector<double>& data){/*操作data*/},
                        std::move(data));

auto func1 = std::bind([](const std::unique_ptr<Widget>& pw){return pw->isValidated() && pw->isArchived();},
                      std::make_unique<Widget>());
```

----------

### 【条款 33】 对 auto&& 型别的形参使用 decltype，以 std::forward 之
- C++ 14中，lambda 可以在形参中使用`auto`，即泛型 lambda 式。若在 lambda 式中需要将参数转发给另一个函数执行。
```c++
auto f = [](auto&&... params){
    return func(normalize(std::forward<decltype(params)>(params)...));
};
```

----------

### 【条款 34】 优先选用 lambda 式，而非 std::bind
- lambda 表达式比起使用 `std::bind` 而言，可读性更好，表达力更强，可能运行效率也更高。
    - 绑定对象的函数调用时通过函数指针发起的，而编译器不太会内联掉通过函数指针发起的函数调用；相对地，lambda 表达式采用的是常规函数调用，则编译器会以常规的方法将其内联。所以，使用 lambda 式就有可能会生成比使用 `std::bind` 运行得更快的代码。
    - lambda 表达式的形参是按值还是按引用传递存储是比较明确的，而`std::bind`则不明确。当然，实际上，绑定对象的所有实参都是按引用传递的，因为此种对象的函数调用运算符利用了完美转发。
- 仅在 C++ 11 中，`std::bind`在实现移动捕获，或是绑定到具备模板化的函数调用运算符的对象的场合中，可能尚有余热可以发挥。（见 条款 32）

----------

## 并发 API
（暂时不熟悉 C++ 并行线程开发，待熟悉以后再阅读补充）

----------

### 【条款 35】 优先选用基于任务而非基于线程的程序设计

### 【条款 36】 如果异步是必要的，则指定 std::launch::async

### 【条款 37】 使 std::thread 型别对象在所有路径皆不可联结

### 【条款 38】 对变化多端的线程句柄析构函数行为保持关注

### 【条款 39】 考虑针对一次性事件通信使用以 void 为模板型别实参的期值

### 【条款 40】 对并发使用 std::atomic，对特种内存使用 volatile

----------
## 微调

----------
### 【条款 41】 针对可复制的形参，在移动成本低并且一定会被复制的前提下，考虑将其按值传递
- 对于可复制的，在移动成本低廉的并且一定会被复制的形参而言，按值传递可能会和按引用传递的具备相近的效率，并可能生成更少量的代码。

```c++
// 1. 重载 ： 对于左值是一次复制，对于右值是一次移动
class Widget{
public:
    void addName(const std::string& newName){
        names.push_back(newName);
    }
    void addName(const std::string&& newName){
        names.push_back(std::move(newName));
    }
}

// 2. 使用万能引用：左值对应一次复制，右值对应一次移动
class Widget{
public:
    template <typename T>
    void addName(T&& newName){
        names.push_back(std::forward<T>(newName));
    }
}

// 3. 按值传递：左值对应一次复制加移动，右值对应两次移动，即比上述两种方式，一定多一次移动操作
class Widget{
public:
    void addName(std::string newName){
        names.push_back(std::move(newName));
    }
}
```
- 经由构造复制形参的成本可能比经由赋值复制形参高出很多
- 按值传递肯定会导致切片问题，所以基类型别特别不适用于按值传递

----------
### 【条款 42】 考虑置入而非插入
- 从原理上，置入（emplace_back）函数应该有时比对应的插入（push_back）函数更高效，而且不应该有更低效的可能。
- 从实践上，置入函数在以下几个前提成立时，极有可能会运行得更快：
    - 待添加的值是以构造而非赋值方式加入容器
    - 传递的实参型别与容器持有之物的型别不同
    - 容器不会由于存在重复值而拒绝待添加的值
- 置入函数可能会执行在插入函数中会被拒绝的型别转换
```c++
std::vector<std::regex> regexes;
regexes.push_back(nullptr); // 无法通过编译
regexes.emplace_back(nullptr); // 能通过编译
```