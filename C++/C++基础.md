## C++项目编译流程

**预处理：**

- 处理预处理语句#，#后内容均为预处理内容
- **常见的预处理语句**#include，#define， #if #endif，#ifdef，#pragma
  - `#include` 只是简单的将后续文件内容copy到现在文件中.
  - `#define A b` 在预处理时将所有A替换成b
  - `#if 1 ` 将`#endif` 前的内容在预处理阶段添加，`#if 0 ` 预处理阶段不添加`#endif` 前的内容
  - `#pragma once` 阻止这个头文件被多次包含，并转化为单个翻译单元，防止一个文件中同一个内容声明多次
    - 函数在被多次声明无所谓，但结构体类声明本质上不是声明而是定义，不能重复声明相同名字的结构体或类（一个文件中只能定义一次）
  - `#ifndef #define #endif` 也可以避免上述重复声明s问题
- 会生成一个翻译单元，用于编译
- 虽然可以直接使用 `#include` 包含 `.cpp` 文件，但这通常不是推荐的做法。原因如下：
  - 直接包含 `.cpp` 文件会导致代码的重复编译，因为每个包含这个 `.cpp` 文件的翻译单元都会包含一份源文件代码。
  - 可能引发链接错误，比如重复定义。

**编译：**

- 编译为二进制文件（目标文件 .obj）
- 包含先编译成汇编程序，再编译成二进制程序
- 翻译单元不等于目标文件
- 编译时项目中所有cpp文件均被编译，除了头文件。因为头文件在预处理时已经copy进入项目文件中
- 编译时需要有声明，否则报错

**链接（link）：**

- 将所有obj文件合并成一个exe文件
- 找到所有函数和符号所在文件
- static控制作用域，添加static使得链接器不可见，链接只能在内部进行
- error1：找不到link的函数和符号
- error2：重复相同的函数和符号，不知道link哪一个。
  - 单个文件编译可行，而整个项目build却出现问题
- 通常要把声明和定义分开写（.h写声明.cpp写定义）的目的也就是为了防止出现link error2 防止在多个翻译单元重复定义同一函数出现link error
- 对于class，在一个文件中不能重复声明，而在多个文件中可以重复声明，但优先使用当前文件的class
- ~~根据 C++ 的链接规则，当同一个结构体在**不同的源文件**中有不同的定义时，会导致链接错误，因为链接器无法确定应该使用哪个定义。(不确定)~~
- ~~**相同定义的结构体**（结构体的成员完全一致）在多个源文件中不会导致链接错误，但推荐通过头文件来共享结构体定义，以避免重复代码。~~

## C++基础知识

### 命名方式

大驼峰

- MyString
- 本人常用于类名，函数，成员函数

小驼峰 

- myString
- 本人只用于变量，成员变量结尾添加_来据区分（也可以使用m\_前缀）

蛇形

- my_string
- 不怎么使用，但是cpp给的函数通常都是蛇形的
- mysql不区分大小写，最好用蛇形命名

串行

- my-string
- 常用于文件

### 变量

```cpp
int 
char
float
bool 0为flase 其他均为true
string
*
```

没啥可说的，记好本质就是内存中二进制数字就行，

变量本质相同，只是内存中所占内存大小不同

### 函数

主要目的防止代码重复，递归，以及分块编写内容（更好的逻辑性）

函数传参数本质是复制过去

### 头文件

主要目的是为了进行函数声明，声明的目的主要是为了通过编译。真正使用这个函数的时候，是link的功能，链接到其他文件的定义中



### 条件语句

需要进行比较，速度会降低（优化问题）

```cpp
else if 
<=>
else 
{
	if
} //else if 只是上面的方便写法而已，else if 其实不是一个关键字，而是先else 然后if
```

#### 三元运算符 ? :

我更喜欢这个方法，不用写那么长的if else 

```cpp
int a = 1;
int b = a == 1 ? 2 : 0 ; //b = 2
// 逻辑 ? 输出真时的值 : 输出假时的值
```

### 循环语句

```cpp
for( ; ; ) {}

while() {}

do{}while();
```

### 控制流语句

```cpp
//退出全部循环
break

//跳出当前迭代
continue

//跳出函数，可以在任何地方
return 
```

### 指针 `*`

保存内存地址的变量

```cpp
int * ptr;
ptr 表示地址
* ptr 地址对应的数据

所有类型的指针都是一样的，表示内存地址just
但不同类型的指针反解的数据不同
故可以通过直接强制转化为void类型的指针，来进行统一，真正使用的时候再强制转化成其他类型的指针（操作系统实验中用过）
```

### 引用 `&`

&表示引用或者取地址符号

```
int a = 5;
int& b = a; //等价于a有了一个b的别名，b和a关联等价起来了，或者是说创建了一个变量这个变量的地址和a的地址相同
//等价于int* c = &a; *c 就是b 

//引用声明后。不能再次改变关联的变量
//引用声明时，必须赋值
```

---

### 枚举 `enum`

```cpp
enum example : unsigned char //控制整数大小，默认int 4字节
{
    A=8,B,C
};
//只能整数

//在class中可以出现下面情况
class Log
{
public:
	enum Level
    {
		Error=0,Warning,Info  
    };
public:
    void SetLevel(Level level)
    {
        m_level = level;
    }
private:
    Level m_level;
};

int main()
{
    Log log;
    log.SetLevel(Log::Error); //说明的是enum只是在类中定义的一个枚举类型，而不是一个成员变量，因此是在Log这个命名空间中
}


```

给特定值命名的一种方式，增加可读性，不用再各种地方处理各种整数

枚举类型的变量，只能使用定义中的值

---

### 联合体 `union`

- 通常是匿名使用的 
- 常与类型双关关联，也可以是类型双关的另一种表示
- union中变量共享内存空间

```cpp
struct Vector2
{
	int x,y;
}；

struct Vector4
{
	union
	{ 
		struct
		{
			int x, y, z, w;
		};
		struct
		{
			Vector2 a, b;
		};
	}; //union中内容是共享内存的
};
//也可以直接类型双关，但union可读性更强
//都是为了避免复制拷贝
//建议使用匿名结构体和匿名union，否则还需要重新声明变量
```





---

### `const`常量

- 只一个承诺，承诺这个是不变的。也可以违背

```cpp
const int MAX = 10;
int* a;
a = (int*)&MAX;
//成功违反承诺 

const int* a = new int;//表示指针指向的内容不能被改变，但可以更改a本身
int const* //和上述等价

int* const a = new int; //和上述相反

const int* const a = new int; //上述综合
```

在类中的使用

- 如果对象是常量（即 `const`），则优先调用 `const` 版本的函数；如果对象是非常量，则调用非 `const` 版本。

```cpp
class Entity
{
private:
	int X，* Y;
    mutable int var; //表示这个变量即使是在const中是也是被允许修改的一个变量
public:
    int GetX() const //表示这个函数中无法修改任何的成员变量
    {
        var++;
        return X;
    }
    void SetX(int x)
    {
        X = x;
    }
    //返回了一个不能被修改且指向内容也不能修改的指针，而且这个函数又不可以对成员变量做出修改
    const int* const GetY() const 
    {
        return Y;
    }
    
};
//本质都是一种承诺
void PrintEntity(const Entity& e)
{
    std::cout<<e.GetX()<<std::endl;
}
//const的类，只能调用const标记的函数。因为其他函数可能会修改e的成员变量（除了mutable声明的变量）

```

### `mutable`

1. 和const联用，允许成员函数是常量方法，但仍然可以修改变量的声明关键字（上面例子）
2. 用在lambda表达中 （Cherno都没用过 所以等学了lambda再说吧）

---

### `new and delete`

new在堆上创建变量，delete释放在堆上申请空间

1. 对象太大
2. 需要人为控制生存期
3. 数据结构链式存储
4. 在堆上分配变量速度小于在栈上分配变量

```cpp
int* a = new int;
int* b = new int[10];
Entity* e = new Entity(); //在分配空间的同时，又调用了构造函数
delete a;
delete[] b;
delete e;

//在特定内存上实例化对象
Entity* e = new(b) Entity();
```

### `auto`

- 省事，但不能滥用，不是所有地方都能用auto
- 常用于函数返回，变量接受时，auto比较方便
- 或者是很长的类型名，太麻烦了
- 我基本只有迭代器，或者超长的函数返回类型名采用auto
- 但`auto`不会是一个引用类型，如果想是引用，得写成`auto&`，还有`const`

```cpp
auto a = 1; //int
auto b = a;  //int

auto c = "shen"; //const char* 

for(std::vector<std::string>::iterator it  = strings.begin(); it != strings.end();it++) {}

for(auto it = strings.begin(); it != strings.end(); it++) {}

```

---

### 多维数组



```cpp
//堆上分配
int** a2d = new int*[50]; //实际上分配了一个指针数组，没有分配真正的数据内存
for(int i = 0 ; i < 50 ; i++)
	a2d[i] = new int[20];
//这样分配出来的内存数组与数组之间不连续，故效率较低，cache miss，空间局部性原理
//delete时许多变量释放
for(int i = 0 ; i < 50 ; i++)
	delete[] a2d[i];
delete[] a2d;

//分配连续空间构造多维数组，把多元数组等价为一维数组
int* array = new int[5 * 5];

```







## C++面向对象

### Class 类基础

对数据和功能组合的一种方法

```cpp
//一个例子
class player
{
	player();
	~player();
public:
	void move(int xa,int ya)
    {
        this->x += xa;
        this->y += ya;
    }
private:
	int x,y;
    int speed;
};
```

类和结构体的区别

- 基本上没区别除了可见性，主要是保持对c的兼容性，基本常用就是`struct`只是用来组合数据成一个新的数据类型just，其他面向对象的用class。但本质没啥区别

- `class `默认内容均为`private`，`struct`默认均为`public`
- 

### 类的可见性

- 对运行性能没有任何影响，只是为了逻辑性，更好的组织代码
- 一般默认情况下，所有内容均是私有的

- public 在类以外的任何地方均可以访问
- private 只有这个类中可以访问的内容（除了友元）
- protected 可见度在private和public之间，可以在类中和所有子类中访问

```cpp
class Entity
{
public:

private:

protected:
    
}
```

---

### 构造函数

- 声明格式`Player（）`
- 在实例化对象的时候运行，常用于初始化类实例变量
- 只是用类的一个静态函数，静态变量时，构造函数不会运行
- 在实例化对象时，构造函数是一定要运行的，
  - `Player() = delete;`**删除构造函数**则无法实例化对象
  - 利用`private: Player() {}`将**构造函数隐藏**，则无法实例化对象
- 如果不指定构造函数，但仍然有一个默认的构造函数，等于是一个空函数
- 在c++中数据基本类型（int float），不会自动初始化为0
- 构造函数可以写多个（其实也就是说在c++中同一个名字的函数可以写多个（同名函数），来满足不同输入的需求，函数的重载）
- 常用两种来初始化，一种是如下的直接赋值和传参赋、一种是使用**成员初始化列表**

```cpp
class Player
{
public:
    int X, Y;
	std::string m_name;
    Player()
    {
        X = 0;
        Y = 0;
        m_name = std::string("Unknown"); //实际上创建了两个类，性能不高，下面方法只创建了一个
    }
	 Player(int x, int y)
    {
        X = x;
        Y = y;
    }
     
    void Print()
    {
        std::cout << X << ", " << Y << std::endl;
    }
};
int main() {
    Player player1;
    player1.Print();
	Player player2(1,2);
    player2.Print();
    return 0;
}
```

### 成员初始化列表

- 性能较高，建议使用

- 本质是在创建前对成员变量调用构造函数（不仅仅是拷贝构造函数）

```cpp
class Entity
{
priavte:
	std::string m_name;
	int m_age;
public:
	Entity()
		/*后面列出需要初始化的成员变量*/
		: m_name("Unknown"),  m_age(0){} //要确保这个顺序，和变量声明顺序相同
	Entity(const std::string& name,const int age)
		: m_name(name), m_age(age){} 
	//这样子等于是只创建了一次类
};
```

### 拷贝构造函数

-  复制是耗时较大的操作，通常是想保留原来数据，但有需要修改值的时候 使用拷贝
- 在只读，想修改本来值等操作时，最好使用引用而非赋值拷贝
- 这次学习我才突然意识到，我之前有太多多余复制的行为了

默认的拷贝构造函数（浅拷贝），只是简简单单的把值复制给另外一个类，通常往往是没有问题的，但遇到有指针的需要对分配的类时，往往只是复制一个指针过去，但是并没有真的把内容重新复制一遍，很容易出现多次delete相同内存的问题，而产生堆栈报错

故往往也需要自行的设计拷贝构造函数

```cpp
class Entity
{
public:
	Entity(const Entity& other) = delete; //表示禁用拷贝 //unique_ptr所作的行为
	Entity(const Entity& other) {}

};
```








---

### 析构函数

- 声明格式`~Entity（）`
- 在实例对象被delete释放内存时调用
- 用于销毁类使用时，申请的所有空间，可能会出现对象被摧毁，但他申请的资源未被释放
- 也可以直接调用析构函数（但很不常见）
- 析构函数和构造函数一样，必须存在时，才可以实例化对象

```cpp
class Entity
{
public:
    int *arr;
    Entity()
    {
		arr = new int[10];
    }
	~Entity()
    {
        delete arr;
    }  
};
```

---

### 继承

1. 从最初的父类中创建子类，来避免代码的重复，提高代码的逻辑性
2. 可以将类之间公共功能放在同一个父类中，然后派生出其他类
3. Player继承了Entity的public。但其实private内容也继承了，只是说private无法在类外访问
4. Player继承Entity，故Player也是一个Entity，具有多态性

```
class Entity
{
public:
	int X, Y;
	void move (int xa,int ya)
	{
		X += xa;
		Y += ya;
	}
};

class Player : public Entity //继承了Entity的public
{
public:
	const char* name;
	void PrintName()
	{
		std::cout<<name<<std::endl;
	}
	
};
```

### 虚函数

在继承中，子类覆写父类中函数时，会出现需要使用虚函数的情况

子类作为父类，调用覆写的函数，若想得到子类覆写后函数的情况，则需要使用虚函数

虚函数使用V表（虚函数表）来实现，V表包含基类中所有虚函数的映射，映射到正确的覆写（override）函数

但会产生额外开销（嵌入式需要注意，其他问题不大）

- 内存开销，需要额外内存存储V表，基类中需要有隐藏的成员指针指向V表
-  调用虚函数时，需要遍历整个表来确定映射到哪个函数

```
class father
{
public:
	virtual void GetName() {}
};

class son : public father
{
public:
	void GetName() override {}
};

```

### 纯虚函数（接口）

纯虚函数允许在基类定义一个没有实现的函数，然后强制子类中实现该函数

若子类没有实现这个方法，那么子类无法实例化

由于基类中没有实现这个方法，实际上不能实例化这个类，但可以用子类转化为基类

c++中接口也是用类来实现的

```cpp
class father
{
public:
	virtual void GetName() = 0; //纯虚函数的办法
};

class son : public father
{
public:
	void GetName() override {}
};

```

---

### 隐式转换

隐式转换指的是编译器自动将一种类型转换为另一种类型，而不需要显式的转换操作符或类型强制转换

- 通常通过**构造函数**或**类型转换运算符**来实现隐式转换。
- 当构造函数只有一个参数时，它会被视为**隐式转换构造函数**，这使得其他类型的对象能够自动转换为该类的对象。

**函数参数传递**中的隐式类型转换只允许一步转换。

```cpp
class Entity
{
private:
	std::string m_name;
	int m_age;
public:
	Entity(std::string name)
		: m_name(name), m_age(-1) {}
	Entity(int age)
		: m_name("Unknown"), m_age(age) {}
}
void Print(Entity e) {}

int main()
{
	Print("shen");
	Print(22);
	Entity e1 = "shen"; //这便是隐式转化
	Entity e2 = 22;
}


```

### explicit

添加在构造函数前，表示这个构造函数不能被隐式的调用，只能显示的调用

- `Entity a(22);`
- `Entity a = Entity(22);`
- `Entity a = (Entity)22; //强制转换`

```cpp
class Entity
{
private:
	int m_age;
public:
	explicit Entity(int age)
		: m_age(age){}
}
```

---

### 重载

**运算符重载**

- 改变或定义运算符的行为


- 运算符本质就是一种函数

- `->`运算符会递归地解析，直到找到一个可以直接调用的成员或函数，重载时不需要写多个`-> `

- `operator++()` 是前置 `++` 的重载，直接修改当前对象并返回引用。

- `operator++(int)` 是后置 `++` 的重载，参数 `int` 是占位符，用于区分前置和后置。它返回旧值，同时修改当前对象。

1. 类内运算符重载

   - 只需要传递一个参数，即符号右侧的操作数，而左侧操作数作为调用这个函数的类，会隐式的传递一个this指针

   - 可以直接访问类的私有成员。

   - ```cpp
     class Entity
     {
     public:
     	int X,Y;
     	Entity(int x, int y)
     		: X(x),Y(y){}
     	Entity Add(const Entity& other) const
     	{
     		return Entity(X + other.X, Y + other.Y);
     	}
     	
     	Entity operator+ (const Entity& other) const
     	{
     		return Add(other); //返回Entity类型主要是为了保证后续一致性
     	}
     }
     ```

   - 

2. 非成员函数的运算符重载

   - 需要传递两个参数，**左操作数**是函数的**第一个参数**，**右操作数**是函数的**第二个参数**。

   - ```cpp
     std:: ostream& operator<< (std::ostream& stream, const Entity& e)
     {
         stream << e.X << ", " << e.Y;
         return stream; //保证后续可以进行<<运算
     }
     //std::cout 是在std命名空间中定义的一个全局变量，而且也不修改这个值，直接引用效果更佳
     ```

3. 函数的重载

   - 函数重载是指同名的函数根据参数个数、类型或顺序不同，执行不同的功能。

   - **函数签名不包括返回类型**，所以函数的返回类型不能用于区分不同的重载函数。

   - ```
      void print(int i) {
          std::cout << "Integer: " << i << std::endl;
      }
      
      void print(double d) {
          std::cout << "Double: " << d << std::endl;
      }
      
      
      ```

   - 


---

### this指针

可以通过this指针访问当前的类实例，this是指向当前实例的指针

this会被初始化为当前实例的常量指针，this不可被修改 *this可以被修改 Entity\* const类型

但是在const类型的函数中，this是一个const Entity* const类型，来保护成员变量不会被修改

---

## C++高级特性

### Static 静态

- static变量初始化时，在c++11后是线程安全的，避免了多线程初始化导致出现数据重置/未定义情况。

1. 在class外部使用static

   - 链接只能在内部，只在定义的翻译单元可见，其他翻译单元无法link到
   - 有些类似与private，只对所在文件有效，其他文件无法link到
   - 用于你不想让这个全局变量是整个项目全局的，只是该文件局部全局。如果没有static，那么链接器会跨翻译单元进行link

2. 在class内部使用static

   - 表示这个属性和实例无关，是所有对象所共有的属性，就像是类的全局属性

   - ```cpp
     class Entity
     {
     	static int x,y;
     };
     
     int main()
     {
     	Entity e;
     	e.x = 1;
         e.y = 2; //这样写能编译通过，但不是恰当的写法
         
     	Entity e1 = {1 , 2}; //该行错误，因为不是该实例的类成员，无法这样初始化
     	return 0;
     }
     ```

---

### 智能指针

会自动进行new delete的一类指针

使用RAII来进行封装的指针，C++的一种管理资源避免泄露的常用方法，使用析构函数来保证释放资源

#### `std::unique_ptr`作用域指针

- 不能被复制`unique_ptr` ， 尽管是函数调用也不可以
- 使用和正常的指针没有区别，依旧是->和*
- 直接定义的对象通常都是一个栈对象，当栈分配的智能指针死亡时，会自动执行delete来释放内存

```cpp
#include <memory>

int main()
{
    std::unique_ptr<Entity> e(new Entity); //会有异常安全问题
    std::unique_ptr<Entity> e1 = new Entity; //这个是错误的，unique_ptr类的构造函数添加了explicit关键字拒绝隐式转换
	std::unique_ptr<Entity> e2 = std::make_unique<Entity>(); //最好的方式，更加安全的考虑
}
```



#### `std::shared_ptr`共享指针

- 当这个指针的引用计数为0时，自动删除这个指针
- 在shared_ptr中需要分配控制块，用来存储引用计数

```
#include <memory>

int main()
{
	std::shared_ptr<Entity> e(new Entity); //会做两次内存分配
	std::shared_ptr<Entity> e1 = std::make_shared<Entity>(); //会组合起来两次内存分配
}
```

#### `std::weak_ptr`弱指针

- 和共享指针一起使用的一个指针类
- 不会像shared_ptr一样在复制时增加引用计数

```
#include <memory>

int main()
{
	std::shared_ptr<Entity> e1 = std::make_shared<Entity>(); 
	std::weak_ptr<Entity> e2 = e1; 
}
```

---

### 模板 template

- 根据调用函数时给予的数据类型，进行函数的运行，重要的是避免相同代码的重复
- 模板不是一个实际的函数，只有当调用的时候才会真正的被创建为函数
- 模板在调用前实际上没有存在，是否报错取决于编译器
- 在使用模板是，在编译阶段会生成一个替换模板的函数或类

```cpp
template<typename T> //typename可以写为class 同义词没区别
void Print(T value)
{
	std::cout<<value<<std::endl;
}

template<typename T ,int N>
class Array
{
private: 
    T arr[N];
public:
    void GetSize() return N;
};


int main()
{
	Print(1); //隐式调用这个模板
    Print<std::string>("abc");
    Array<std::string , 20> s;
}

```

---

### 宏 `#define`

- 发生在预处理阶段
- 本质就是替换
- 可用于开发时的调试，在正式中不存在，用宏和预处理器删除特定的代码

```
#define WAIT std::cin.get() //一个简单的宏

#define LOG(x) std::cout<< x << std::endl //一个函数宏，LOG(x) x输入后也会替换后面的x

//调试中使用宏，而最后release版本不使用
#if PR_DEBUG == 1 //这个自己设置的
#define LOG(x) std::cout<< x << std::endl
#else
#define LOG(x) 
#endif

//可以利用\换行，\是Enter的转义，且\后不能有空格
#define Mian int main() \
{ \
	return 0; \
}

```

---

### 函数指针

- 存储指向函数所在空间地址的变量
- 函数的本质就是一串指令，存储在程序中，每次调用函数就是call 这个地址，最后在ret回来 （汇编内容）
- 总是会用于回调函数，异步问题上，线程这些
- 和lambda一起使用

#### 原始函数指针 c函数指针

```cpp
void HelloWorld(int a)
{
	std::cout<<"Hello World! value: "<< a <<std::endl;
}

int main()
{
	void(*functionName)(int) = HelloWorld; //括号里写的是参数，前面void是函数输出类型
	auto function = HelloWorld; //可以用auto来省写类型
	function(1);
	
	//用typedef using定义
	typedef void(* HelloWorldFunction)(int);
	HelloWorldFunction function = HelloWorld;
	function(1);
	
}
```

#### c++函数指针

std::function<>



### lambda表达式

- 本质上是定义匿名函数的一种方式，一种一次性的函数，本质是一个函数类
- 可用于 sort 里面函数指针参数
- Lambda 表达式可以自动推导返回类型，也可以显式指定返回类型。

```cpp
[ capture ]( params ){ content }

int main()
{
	int a;
    //自动推导返回类型
	//capture = 表示传递所有变量，通过值传递（复制）
	auto lambda = [=](int value) { std::cout<< a << std::endl; }
	//capture & 表示传递所有变量，通过引用传递
	auto lambda = [&](int value) { std::cout<< a << std::endl; }
	//capture a 表示传递变量a，通过值传递
	auto lambda = [a](int value) { std::cout<< a << std::endl; }
    //capture &a 表示传递变量a，通过引用传递
	auto lambda = [&a](int value) { std::cout<< a << std::endl; }
    //有capture时，无法使用原始函数指针传递这个lamdba，需要使用cpp的函数指针
    
    //显示指定返回类型
    auto lambda = [=](int value) -> int { return value }	
}
```

![image-20241110181709443](C:\Users\y\AppData\Roaming\Typora\typora-user-images\image-20241110181709443.png)

![image-20241110184253857](C:\Users\y\AppData\Roaming\Typora\typora-user-images\image-20241110184253857.png)

---

### 命名空间`namespace`

- 首先尽量不要使用`using namespace`在写项目的时候，算法题无所谓（但熟能生巧别这么搞）
- 在相同的命名空间下，不能有相同名字的符号。函数重载必须输入参数不同
- class也是一种命名空间，可以直接::调用静态函数和静态变量

```cpp
namespace shen {
    void print()
    {
        
    }
}
namespace yao {
    void print()
    {
        
    }
}
shen::print();
yao::print(); //::进入命名空间

namespace a { namespace b { 
	void function() {}
} }

a::b::function(); //a::function() b::function() 是错的

```

---

### C++线程 `std::thread`

具体详见操作系统

- 目的 优化程序

```
#include <thread>
void DoWork()
{
	std::cout<<"New Thread"<<std::endl;
	std::this_thread::get_id(); //线程id
	std::this_thread::sleep_for(2s); //this_thread类可以对当前线程下达命令
}


int main()
{
	std::thread worker(DoWork); //参数是函数指针
 	worker.jion(); //等待这个线程结束后，再继续进行。大白话 先阻塞当前，然后调度worker，worker结束后调度当前
}

```

---

### 计时

- 可以使用`chrono`库，但如果要高精度的时间，需要使用操作系统库
- 同时可以自己构造一个class/struct，来自动化计时过程，利用构造和析构函数

```cpp
#include <chrono>
#include <thread> //为了使用this_thread

class Timer
{
private:
	std::chrono::time_point<std::chrono::steady_clock> start,end;
	std::chrono::duration<float> duration;
public:
	Timer()
	{
		start = std::chrono::high_resolution_clock::now();
	}
	~Timer()
	{
		duration = end - start; //隐式转换，然后赋值
		float ms = duration.count() * 1000.0f
		std::cout<< ms << "ms" << std::endl; 
	}
	
};

void function()
{
	Timer time;
	xxxxxx
}

int main()
{
	auto start = std::chrono::high_resolution_clock::now(); //记录当前时间
	std::this_thread::sleep_for(1s);
	auto end = std::chrono::high_resolution_clock::now(); //记录当前时间
	
	std::chrono::duration<float> duration = end - start;
	std::cout<< duration.count() << std::endl; 
	
}

```

---

### 类型双关

- 同一内存数据，表示成不同的类型，可以用强制转换构造
- **类型双关**直接操纵内存和位模式，用于底层编程，它打破了类型系统的限制，带来高性能的同时也伴随风险。

```cpp
int a = 50;
float value = a; //a隐式转换为float，赋值给value
float value = (float)a; //显式
float value = *(float*)&a; //都是创建了新的变量，然后复制过去
//不想创建新的变量，而是让这个内存数据可以表示为两个类型
//类型双关
float& value = *(float*)&a; //只能这样，(float)a为右值不行


```

### 左值右值

左值：具有某种存储支持的变量

右值：临时变量

左值引用只接受左值，除非加上const，const int&

右值引用只接受右值，除非加上const，const int&&

### 移动语义



## C++小知识点
### 对象的生存周期（栈作用域

- 在c++中，每进入一个作用域中，都是在push一个栈帧
- 在这个作用域中定义的变量，都是在这个栈帧中
- 当这个作用域程序允许结束后，会释放栈帧，并**释放**其中所有基于**栈申请的变量**（堆变量不会释放）
- 所有的变量都遵循一个道理，所在的作用域消失后（or运行结束了），基于作用域申请的变量均会被释放
- 生存周期和作用域有关，堆的作用域是整个程序，栈的作用域是栈帧本身 

---

### 字符串字面量

```cpp
const char* string = "abcd";

char* string = "abcd"; //error 

char string[] = "abcd"; //并不是仅仅的吧指针给了左边，如果修改时，会发生赋值

//其中“abcd”为字符串字面量，永远在内存中保存在只读部分，“xxx”返回的是一个const char*指针

//字符串字面量不是const char*，是一个const char[]（常量字符数组） 只是说在等于时返回的是一个const char*
```

---

### 读取数量不定的输入数据

- 对于输入来说，执行`std::cin`会中断来进行键盘输入，而Enter的作用不是结束`std::cin`的状态，而是结束本次输入中断，将内容写入流
- 而`std::cin`由于是流对象，内部原理也就是流的原理，调用时直接从用户缓存中寻找是否有内容，如果有则传入对应值，如果没有则中断
- 对于`std::cin<<value;`输入运算符返回运算符左侧对象`std::cin`，while循环条件中判断的是`std::cin`
- 使用一个iostream对象作为条件时，效果为检测流对象的状态。应该是与()运算符的重载有关

```cpp
while(std::cin>>value)
{
	std::cout<<value<<std::endl;
}
```

---



## C++ 标准模板库 STL

### `std::string`

本质是一个类，有符号重构的情况下，可以把string直接默认为一个char *

string的构造函数的参数，可以是`char*` `const char*`

```cpp
string name = "abcd"; //本质是等号发生了重构
//后面的补充，其实不是重构，其实是隐式转换，string有一个只含有const char*参数的构造函数，可视为隐式构造函数
string name = string("yao") + "shen"; //此时+发生了重构，只不过这个重构时对于string这个类来说的
string name = "yao" + "shen"; //此时并没有重构所以一定是error

//与字符串结合使用的部分函数
std::isalnum(); //#include <ctype.h>，判断是否为a-z A-Z 0-9
std::tolower(): //#include <ctype.h> ,大写转为小写，其他的不变
std::to_string()； //#include <string> 将数字类型（如 int, float, double 等）转换为字符串
```

#### `MyString`

```cpp
class String
{
private:
	char* m_Buffer;
	unsigned int m_Size;
public:
	String(const char* string)
	{
		m_Size = strlen(string);
		m_Buffer = new char[m_size+1]; //往往需要重新构造拷贝函数，进行深拷贝
		memcopy(m_BUffer,string,m_Size);
		m_Buffer[m_size] = 0;
	}
    String(const String& other)
        :m_Size(other.m_Size)
	{
		m_Buffer = new char[m_size+1]; //往往需要重新构造拷贝函数，进行深拷贝
		memcopy(m_BUffer,other.m_Buffer,m_Size+1);
	}
	~String()
	{
		delete[] m_Buffer;
	}
	char& operator[] (unsigned int index)
	{
		return m_Buffer[index];
	}
	friend std::ostream& operator<<(std::ostream& stream,const String& string);
}

std::ostream& operator<<(std::ostream& stream,const String& string)
{
	stream << string.m_Buffer;
	return stream;
}
int main()
{
	String string = "shen"; //隐式转换，const char* 的构造函数
	string[1] = 'a'; //[]的重构

}
```

---

### `std::vector`

本质是一个动态数组，一个连续的空间存放，超出数组大小时，需要重新分配空间复制过去（很慢的事情） ，然后释放旧位置

- emplace_back C++11 新添内容，功能更优避免了多余复制和临时对象，但如果为了兼容以前就得用push_back，否则emplace_back更优。如果使用emplace_back()使用时空间不够，会扩充复制，通常是容量翻倍（编译器有关）
- 存储在堆中，但会自动释放内存
- 类对象在栈中，但数据在堆中

```cpp
#include<vector>


int main ()
{
	std:::vector<Entity> entities;
    entities.reserve(3); //预分配一个大小为3的动态数组
	entities.push_back(Entity(1,2)); //以Entity为参数，复制到vector最后面
    
    //emplace_back C++11 新添内容，功能更优，但如果为了兼容以前就得用push_back，否则emplace_back更优
    entities.emplace_back(3,4); //在vector实际内存上直接使用这些参数，构造一个Entity对象，没有复制
	//如果在尾部直接构造新对象emplace_back，如果已有Entity对象，可考虑push_back
    
    
	//遍历
	for(int i=0; i<entities.size();i++)
		std::cout<< entities[i] <<std::endl; //这里有重载
	//基于range的循环
    for(Entity e : entities)
    	std::cout<< e <<std::endl;
	//如果只是输出可以优化
    for(const Entity& e : entities)
    	std::cout<< e <<std::endl;
    
    entities.clear(); //清空
    entities.erase(vector_iterator); //例如 entities.erase( entities.begin() + 1 );
    //erase的过程比较奇特，他会删除一个元素，后面元素又会前移，但不是复制前移使用的不是拷贝构造函数，但使用的是移动构造函数
    
    entities.begin(); //返回第一个元素的迭代器，可以先当成一个指针对待
    entities.end();
    entities.front(); //返回第一个元素

}

```

**使用优化**

- 优化重要规则，很好的了解编写环境是啥（语言特点和环境）
- 优化策略一：直接创建足够的计划内的空间，那么就不用一直调整了
- 优化策略二：将要创建的类直接放到vector事先分配好的内存中，而不是先创建Entity然后复制到vector中
- 

```cpp
std::vector<Entity> es;
//下面一共复制了6次Entity
es.push_back(Entity(1,2)); //先生成Entity，然后复制到es中
es.push_back(Entity(3,4)); //超过动态数组大小，重新分配空间，将原来内容复制到新内存，然后复制新创建的Entity到es空间
es.push_back(Entity(5,6)); // 同上

//优化策略一：直接创建足够的计划内的空间，那么就不用一直调整了
//下面一共复制了3次
entities.reserve(3); //预分配一个大小为3的动态数组
es.push_back(Entity(1,2)); //先生成Entity，然后复制到es中
es.push_back(Entity(3,4)); //同上
es.push_back(Entity(5,6)); // 同上

//优化策略二：将要创建的类直接放到vector事先分配好的内存中，而不是先创建Entity然后复制到vector中，避免了多余复制和临时对象
//下面一共复制了0次
entities.emplace_back(1,2); //在vector实际内存上直接使用这些参数，构造一个Entity对象，没有复制
entities.emplace_back(3,4);
entities.emplace_back(5,6);
```

---

### `std::array`

- 静态数组
- `template<typename T , int N>  //两个模板参数，第一个类型，第二个大小`
- 和c数组一样存储在栈中
- 有边界检查，利用宏（虽然我不知道为啥就是了）
- 这个类其实是没有存储size这个大小的

```
#include <array>

template<int N>
void PrintArray(const std::array<int, N>& data)
{
	for(int a : data)
		std::cout<<a<<std::endl;
}


int main()
{
	std::array<int, 5> data;
	data[0] = 0;
	data[1] = 1;
	data.size();
	data.begin()
	data.end();
	
	PrintArray(data);
	PrintArray<5>(data);
	
}


```

---

### `std::tuple`元组

一个类可以包含多个类型的变量，可以用作函数多参数的返回

```cpp
#include <tuple>

std::tuple<std::string, int> Function()
{
    
    return std::make_pair(a,b);
}

int main()
{
    auto source = Function();
    std::string s = std::get<0>(source);
    int x = std::get<1>(source);
}


```



### `std::pair`元组

一个类可以包含2个类型的变量，可以用作函数多参数的返回，更好的方法还是用结构体或者函数参数

```cpp

std::pair<std::string, int> Function()
{
    
    return std::make_pair(a,b);
}

int main()
{
    auto source = Function();
    std::string s = source.first;
    int x = source.second;
}
```

---

### `std::unique_ptr`

```cpp
#include <memory>

int main()
{
    std::unique_ptr<Entity> e(new Entity); //会有异常安全问题
    std::unique_ptr<Entity> e1 = new Entity; //这个是错误的，unique_ptr类的构造函数添加了explicit关键字拒绝隐式转换
	std::unique_ptr<Entity> e2 = std::make_unique<Entity>(); //最好的方式，更加安全的考虑
}
```

##### `MyUnique_ptr`

```cpp
class UniquePtr
{
	//超出作用域时，自动释放内存的指针，且不能复制
private:
	Entity* m_obj;
public:
	UniquePtr(Entity* entity)
		: m_obj(entity) 
	{
	}

	UniquePtr(const UniquePtr& other) = delete;

	~UniquePtr()
	{
		delete m_obj;
	}
	
	Entity* operator->()
	{
		return m_obj;
	}

	const Entity* operator->() const
	{
		return m_obj;
	}
};

int main() 
{
	UniquePtr e1(new Entity());
	e1->Print();
	const UniquePtr e2(new Entity());
	e2->Print(); //不需要多写一个->来表示指向PrintXY，这是因为->运算符会递归地解析，直到找到一个可以直接调用的成员或函数。
    return 0;
}
```

### `std::shared_ptr`



### `std::weak_ptr`



---

### `std::mutex`

- 互斥锁
- 常与std::unique_lock联用来自动锁定和解锁

```
#inlcude<mutex>
std::mutex mtx;
mtx.lock();
mtx.unlock();

std::unique_lock<std::mutex> lock(mtx);
//unique_lock 也可以显示上锁 解锁 功能全面灵活
```

### `std::condition_variable`

- 条件变量

```cpp
#include<condition_variable>

std::mutex mtx;
std::condition_variable cv;
std::unique_lock<std::mutex> lock(mtx);
cv.wait(lock, cmp); //cmp为真的时候通过，为假阻塞
cv.notify_all();
cv.notify_one();
```

---

### `std::(i/o)fstream`

- 文件流，`fstream`头文件中包含输入输出流和双向流
- 流对象是将打开的文件封装成一个流，在Linux下打开的文件会以文件描述符Fd作为标识，而流对象并不是打开的文件，只是打开文件对应的流，用于IO的进行，毕竟打开的文件最重要的功能便是IO。流其实可以当成一个中间件
- 通过构造函数或者open打开文件
- `if_open()`检查是否打开文件
- `close()`关闭文件
- `getline()`逐行读取文件内容
- `outFile << "xxx" <<std::endl;` 来写入文件流
- 如果文件已经存在，默认情况下会清空文件内容。如果不想覆盖文件，可以使用 `std::ofstream` 的第二个构造参数 `std::ios::app`（追加模式）来追加内容。
- C++ 中使用 `fstream` 类时，文件在对象的生命周期结束时会自动关闭（当对象超出作用域时，析构函数会被调用）。

```cpp
#include <fstream>

int main() {
    std::fstream file("example.txt", std::ios::in | std::ios::out | std::ios::app); //模式
    if (!file) 
    {
        std::cerr << "Failed to open file!" << std::endl;
        return 1;
    }
    
    //输出流
    file << "Appending some data!" << std::endl;
	
    file.seekg(0);  // 移动到文件的开始位置
    std::string line;
    while (std::getline(file, line)) 
    {
        std::cout << line << std::endl;
    }

    file.close();
    return 0;
}
```







### `std::sort()`

- `std::sort(iterator.begin, iterator.end, cmp); //cmp为函数指针`
- 时间复杂度o(n*logn)

```cpp
#include <algorithm>

std::vector<int> values;
std::sort(values.begin(),values.end()); //默认升序

//bool cmp(const Type& a, const Type& b); //cmp官方声明
std::sort(values.begin(),values.end(), [](const int& a, const int& b){
    return a > b;  // 降序排序
})；
    
bool cmp(const int& a, const int& b)
{
    return a > b;
}
std::sort(values.begin(),values.end(), cmp);



```

---

###  `std::reverse()`

- `std::reverse(iterator.begin, iterator.end); //cmp为函数指针`

---

### `std::move()`
