---
layout: post
title:  "new features of C++11"
author: chenlian
categories: C++
tags: C++
excerpt: C++11 will become more and more important in the C++ ecosytem, eventually becoming the most prevalent version. C++11 brings a large range of new features that makes developments safer, faster, easier and more fun.
---


* content
{:toc}
C++11 will become more and more important in the C++ ecosytem, eventually becoming the most prevalent version. C++11 brings a large range of new features that makes developments safer, faster, easier and more fun.


## 简介


C++11 is a version of the standard for the programming language C++. It was approved by International Organization for Standardization (ISO) on 12 August 2011, replacing C++03, superseded by C++14 on 18 August 2014 and later, by C++17, which is still under development. the name follows the tradition of naming  language version by the publication year of the specification, though it was formerly named C++0x because it was expected to be pulished before 2010.(`from wikipedia`)


## auto 自动类型推导


auto -- deduction of a type from an initializer


auto 的自动类型推导，用于从初始化表达式中推断出变量的数据类型。


格式： `auto x = expression;`

```c++
/* error,auto 通过初始化表达式进行类型推导。如果没有初始化表达式，就无法确定类型。*/
auto var;	
```

```c++
auto i = 0;				// 推导为 int
auto d = 0.0;				// 推导为 double
auto c = 'c';				// 推导为 char
auto str = "hello world!";		// 推导为 char*
auto b = Base()				// 推导为 Base
auto bp = new Base();			// 推导为Base *
auto& ir = i;				// 推导为 int ，dr 为 i 的引用
auto& dr = d;				// 推导为 double ，dr 为 d 的引用
auto& br = b;				// 推导为 Base ，dr 为 b 的引用
```


## decltype 类型获取


decltype -- the type of an expression


decltype(E) is the type("declared type") of the name or expression E and can be used in declarations.



decltype（E）是名称或表达式E的类型（“声明类型”），可以在声明中使用。

```c++
int a;
decltype(a) b;
```


## nullptr -- 空指针字面量


nullptr is a literal denoting the null pointer; it is not an interger.


nullptr 是一个字面量，表示空指针，它不是一个整数。


而 NULL 在 C 和 C++ 中是一个宏定义，`#define NULL 0`。下面程序可以说明：


```c
#include<stdio.h>

int main(void)
{
	int n = NULL;
	printf("n is %d\n",NULL)
	if(NULL == 0)
		printf("NULL is %d\n",NULL);
	return 0;
}
```

而 `int i = nullptr; ` 会报错。 `if(nullptr == 0)` 不会报错，但这不能说明 nullptr = 0，而是因为 0 可以转为值为0的指针类型。

nullptr 使用格式： 


`Type * var = nullptr;` 


`if(nullptr == point_type)`


## range-for statement 序列for循环


A range for statement allows you to iterate through a "range", which you can iterate through like an STL-sequence defined by a begin() and end(). All standard containers can be used as a range, as can a std::string, an initializer list, an array, and anything for which you define begin() and end(), e.g. istream.


序列for循环可以遍历序列。


```c++
#include<iostream>
#include<map>
#include<vector>
#include<string>

using std::cout;
using std::endl;
using std::map;
using std::vector;
using std::string;

int main(void)
{
    map<string,vector<int>> my_map;
    vector<int> my_vector;

    my_vector.push_back(1);
    my_vector.push_back(2);
    my_vector.push_back(3);

    my_map["one"] = my_vector;

    for(const auto& kvp : my_map){
        cout<<kvp.first<< ":";
        for(auto value : kvp.second ){
            cout<<value<<" ";
        }
        cout<<endl;
    }

    int int_array[] = {10,20,30,40};
    for(auto int_value : int_array){
        cout<<int_value<<" ";
    }
    cout<<endl;

    return 0;
}

```


## lambda 表达式


A lambda expression is a mechanism for specifying a funciton object. The primary use for a lambda is to specify a simple action to be performed by some function.


lambda 表达式是指定函数对象的机制。lambda 的主要用途是指定一些函数要执行的简单的动作。


格式：


`[capture](parameters)->return-type{body}`


* \[capture\] : lambda 可以使用的在定义lambda 出能访问的变量列表。[&] 说明通过引用传递来使用变量，可读可写。[=] 说明通过值传递来使用变量，可读。[] 不使用变量。使用特定的变量可以在 = 和 & 后添加变量名。e.g. [&i]
* (parameters) : lambda 表达式的的形参类型和形参名。e.g. (int a,int b)
* ->return-type : lambda 表达式的返回值。可以不写，可通过编译器判断出返回值类型。
* ｛body} : 程序块。


```c++
#include<iostream>

using std::cout;
using std::endl;

int out = 2;

int main()
{
	int a = 1;
	int b = 2;
	auto add = [&](int c)->int{
		a = 10;
		b = 20;
		return a + b + c;
	};
	cout<<add(30)<<endl;
}
```

## control of defaults: defaults and delete


C++ 中类存在着构造函数，拷贝构造函数，赋值构造函数。在编程中，当没有定义这些构造函数时，编译器会自己产生，但编译器产生的构造函数的行为有可能与程序的意图不合或者根本不需要。因此，需要通过C++的关键字来明确程序的意图。

以下是其语法：


```c++
class Base
{
	Base() = default;				//构造函数
	Base & operator = (const Base& ) = delete;	//赋值构造函数
	Base (const Base&) = default;			//拷贝构造函数
}
```

