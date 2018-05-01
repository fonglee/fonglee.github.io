---
layout: post
title: 高质量C编程指南笔记
excerpt: "高质量C编程指南笔记"
modified: 2015-01-03
tags: [程序]
comments: true
image:
  feature: sample-image-4.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---




## 目录结构
如果一个软件的头文件较多，通常将头文件和定义文件存放在不同的目录中。比如include目录和source目录。如果某些头文件是私有的，则可以和定义文件放在同一目录中。
## 长行拆分
长表达式在低优先级操作符处拆分成新行，操作符放在新行之首。拆分出的新行要进行适当的缩进。
## 注释
程序块的注释常采用/**/，行注释一般采用//。边写代码边注释
## 类的板式
将public类型的函数写在前面，将private类型的数据写在后面，即以行为为中心。
## 命名规则
windows应用程序通常采用大小写混排的方式，unix应用程序通常采用小写加下划线的方式。
变量的名字应该使用名词或者形容词+名词。
全局函数应该使用动词或者动词+名词。
windows命名：
类名和函数名采用大写字母开头的单词组合而成：
    
    class Node
    class LeafNode
    void Draw(void);
    void SetValue(int value);

变量和参数用小写字母开头的单词组合而成
    
    BOOL flag
    int  drawMode
常量全用大写的字母，用下划线分割单词。
    
    const int MAX = 100;
    const int MAX_LENGTH = 100;

静态变量前面加前缀s_
    
    void Init()
    {
        static int s_intValue;
    }
全局变量前面加前缀g_
    
    int g_howManyPeople;
    int g_howManyMoney;

类的数据成员加前缀m_

为了防止某一个软件库中的一些标示符和其它软件库中的冲突，可以为各种标示符加上能反映软件性质的前缀。如OpenGL所有库函数均以gl开头，所有常量均以GL开头。

## 浮点变量与零值的比较
    
    if ((x >= -EPSINON) && (x <= EPSINON))

## 循环语句的效率
将最长的循环放在最内层。
将逻辑判断放在循环体外面。

## const和#define的比较
用const来定义常量，c++程序只使用const常量。

## 常量
需要对外公开的常量放在头文件中，不需要对外公开的常量放在定义文件的头部。
如果某一个常量与其他常量密切相关，则应该在定义中包含这种关系。
## 类中的常量
不能在类声明中初始化const数据成员，
要在整个类中定义恒定的常量，用枚举变量来实现
    
    class A
    {
        enum  {SIZE1 = 100, SIZE2 = 200};
        int array1[SIZE1];
        int array2[SIZE2];
    }
    枚举常量不会占用对象的存储空间，他们在编译时被全部求值。

## 参数的规则
目的参数放在前面，源参数放在后面
如果源参数时指针，且仅作为输入用，则应该在类型前加上const。
    
    void StringCopy(char *strDestination, const char *strSource);

避免参数过多，控制在5个以内。

尽量不要使用类型和数目不确定的参数，如printf

## 返回值的规则
不要将正常值和错误标志混在一起返回
getchar可以改写成`BOOL GetChar(char *x);`

有时候函数原本不需要返回值，但为了增加灵活性，如增加链式表达,如字符串拷贝函数strcpy函数
## 函数内部实现的规则
return 不可以返回指向栈内存的指针或者引用

函数体的规模要小，50行代码以内
避免函数带有记忆功能，如避免使用static局部变量
不仅检查输入参数的有效性，还要检查 如全局变量、文件句柄的有效性。
## 使用断言
assert仅在debug版本起作用，在函数入口处，检查参数的有效性
在编写函数，反复考察假定，一旦确定了假定，就使用断言对假定进行检查。

## 内存
如果p是函数的参数，则在函数的入口处用asset进行检查，如果是用malloc或者new来申请的内存，则用if语句。

## 计算内存容量
    char a[] = "hello world";
    char *p = a;
    cout<< sizeof(a) << endl; // 12字节
    cout<< sizeof(p) << endl; // 4字节
    
    void Func(char a[100])
    {
        cout<< sizeof(a) << endl; // 4字节而不是100字节
    }

## 指针参数传递内存
如果要用指针参数申请内存，那么改用指向指针的指针
    
    void get_memory(char **p, int num)
    {
        *p = (char *)malloc(sizeof(char) *num);
    }
也可以用函数返回值来传递动态内存
    
    char *get_memory(int num)
    {
        char *p = (char *)malloc(sizeof(char) *num);
        return p;
    }

## 杜绝野指针
指针变量应该初始化为NULL
指针被free之后，没有置为NULL

## 内存耗尽
用return语句终止本函数，或者使用exit(1)终止程序，或者为new和malloc设置异常处理函数。

## const
对于非内部数据类型的输入参数，应该将值传递的方式改写成const 引用传递。
对于内部数据类型的输入参数，不需要。


