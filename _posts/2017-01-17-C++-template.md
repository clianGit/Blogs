---
layout: post
title:  "C++ 模板"
author: chenlian
categories: C++
tags: C++ Generic-programming
excerpt: C++ is not c with class, template makes C++ powerful。
---


* content
{:toc}
C++ is not c with class, template makes C++ powerful。

# C++ 模板
C++模板是为类或函数提供一种在编译期进行参数转换的能力。template中的参数可以是基本数据类型和自定义数据类型，参数也可以为兼容int类型(char，bool等）的变量，当然template也可以是参数和兼容int类型变量的组合。`(个人理解)`

## C++ 泛型编程
C++泛型编程为不同数据类型提供统一的类或函数。比如，一个通用的数据结构队列，可以为基本数据类型int、double或者自定义类型string等产生各自的队列；也可以为一个比较大小的通用算法产生各种不同数据类型的比较大小算法。

### 模板函数
```c++
#include<iostream>

using std::cout;
using std::endl;

class my_class
{
private:
	int _Int_num;
public:
	my_class(int __Int_num):_Int_num(__Int_num){}
	
	/* 重载==运算符 */
	friend bool operator ==(my_class & __my_class_one,my_class & __my_class_two){
		if(__my_class_one._Int_num == __my_class_two._Int_num){
			return true;
		}else{
			return false;
		}	
	}
	
	/* 重载 > 运算符  */ 
	friend bool operator >(my_class & __my_class_one,my_class & __my_class_two){
		if(__my_class_one._Int_num > __my_class_two._Int_num){
			return true;
		}else{
			return false;
		}	
	}
	/* 重载 < 运算符 */
	friend bool operator <(my_class & __my_class_one,my_class & __my_class_two){
		if(__my_class_one._Int_num < __my_class_two._Int_num){
			return true;
		}else{
			return false;
		}	
	}
};
/* 模板函数 */
template<typename T>
int compare(T & __T_var_a, T & __T_var_b)
{
	if(__T_var_a == __T_var_b){
		return 0;
	}
	
	if(__T_var_a > __T_var_b){
		return 1;
	}
	
	if(__T_var_a < __T_var_b){
		return -1;
	}
}

int main(void)
{
	/* 基本类型int比较 */ 
	int Int_one = 3;
	int Int_two = 2;
	cout<<compare<int>(Int_one,Int_two)<<endl;
	
	/* 基本类型double比较 */
	double Double_one = 2.2;
	double Double_two = 2.2;
	cout<<compare<double>(Double_one,Double_two)<<endl;
	
	/* 自定义类型比较，需重载==,>,<运算符 */
	my_class my_class_one(1);
	my_class my_class_two(2);
	cout<<compare(my_class_one,my_class_two)<<endl;
	
	return 0;
}

```

### 模板类


```c++
#include<iostream>
#include<string>

using std::cout;
using std::endl;
using std::string;

/* 模板类 */
template<typename T>
class Print
{
private:
	T _T_var;
public:
	Print(T __T_var):_T_var(__T_var){}
	
	void show(){
		cout<<_T_var<<endl;
	}
};

int main()
{
	/* 基本类型int */
	Print<int> Int_print(1);
	Int_print.show();
	
	/* 基本类型double */
	Print<double> Double_print(2.2);
	Double_print.show();
	
	/* 自定义类型string */ 
	Print<string> String_print("hello world");
	String_print.show();
	
	return 0;
}
```
### template 为自定义类型
```c++
#include<iostream>

using std::cout;
using std::endl;

class Father{};

class Son_1:public Father{};


class Son_2:public Father{};

template<typename Father>
void print()
{
	cout<<"Father"<<endl;
}
template<>
void print<Son_1>()
{
	cout<<"Son_1"<<endl;
}
template<>
void print<Son_2>()
{
	cout<<"Son_2"<<endl;
}

int main()
{
	print<Father>();
	print<Son_1>();
	print<Son_2>();
	
	return 0;
}
```


### template为兼容int类型的变量


```c++
#include<iostream>

using std::cout;
using std::endl;

template<int N>
void display()
{
	cout<<N<<endl;
}

int main()
{
	display<1>();
	display<2>();
	display<3>();
	
	return 0;
}
```


char类型兼容int类型，将其中的int换成char和bool类型也可。但将其中int 改为double，会报错的。


### template 为数据类型和兼容int类型的组合
```c++
#include<iostream>
#include<string>

using std::cout;
using std::endl;
using std::string;

template<typename T ,int N>
void display(T __T_var)
{
	cout<<__T_var<<" "<<N<<endl;
}

int main()
{
	display<int,1>(1);
	display<double,2>(2.0);
	display<string,3>("hello world!");
	
	return 0;
}
```


## 特化
特化是将template的数据类型指定为具体的数据类型，比如typename T 中可以指定T 为int或将父类的类型指定为子类类型。代码如下：


```c++
#include<iostream>
#include<string>

using std::cout;
using std::endl;
using std::string;

/* 模板类 */
template<typename T>
class Print
{
private:
	T _T_var;
public:
	Print(T __T_var):_T_var(__T_var){}
	
	void show(){
		cout<<"T "<<_T_var<<endl;
	}
};

template<>
class Print<int>
{
private:
	int _Int_var;
public:
	Print(int __Int_var):_Int_var(__Int_var){}
	
	void show(){
		cout<<"int "<<_Int_var<<endl;
	}
};

int main()
{
	/* 基本类型int */
	Print<int> Int_print(1);
	Int_print.show();
	
	/* 基本类型double */
	Print<double> Double_print(2.2);
	Double_print.show();
	
	/* 自定义类型string */ 
	Print<string> String_print("hello world");
	String_print.show();
	
	return 0;
}
```

template为兼容int类型的变量时，其实就是对int类型变量的一种特化。
```c++
#include<iostream>

using std::cout;
using std::endl;

template<int N>
struct sum
{
	enum{value = N + sum<N-1>::value};
};

template<>
struct sum<1>
{
	enum{value = 1};
};

int main()
{
	cout<<sum<10>::value<<endl;
	return 0;
}
```


### 偏特化
偏特化就是对多个template类型中的内容，只部分特化。

### 全特化
全特化就是能列出来所有的情况

## Traits
Traits 是一种在程序编译期间根据型别作出判断的泛型技术。这能写出容易理解和更易维护的代码。
```c++
#include<iostream>

using std::cout;
using std::endl;

class Fish{};

class Cat{};

class Dog{};


template<typename A>
class Traits{};
	
template<>
class Traits<Fish>{
public:
	void print()
	{
		cout<<"fish"<<endl;
	}
};
	
template<>
class  Traits<Cat>{
public:
	void print()
	{
		cout<<"cat"<<endl;
	}
};
	
template<>
class  Traits<Dog>{
public:
	void print()
	{
		cout<<"dog"<<endl;
	}
};

template<typename T>
class Animal{
public:
	void print()
	{
		Traits<T>().print();
	}
};

int main()
{
	Animal<Fish> fish;
	fish.print();
	
	Animal<Cat> cat;
	cat.print();
	
	Animal<Dog> dog;
	dog.print();
	
	return 0;
}
```


