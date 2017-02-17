---
layout: post
title:  "C++ constructor"
author: chenlian
categories: C++
tags: C++
excerpt: A constructor is a kind of member function that initializers an instance of its class. A constructor has the same name as the class and no return value. A constructor can have any number of parameters and a class may hava any number of overloaded constructors. Constructors may have any accessibility, public, protected or private.
---


* content
{:toc}
A constructor is a kind of member function that initializers an instance of its class. A constructor has the same name as the class and no return value. A constructor can have any number of parameters and a class may hava any number of overloaded constructors. Constructors may have any accessibility, public, protected or private.



## 简介


C++ 中有构造函数较多，有普通构造函数，拷贝构造函数(copy constructor)，赋值运算符函数(copy assignment)，移动构造函数(move constructor),移动运算符函数（move assignment）。有explicit 和 implicit 关键字来修饰构造函数。构造函数可以是default 和 delete（`C++11起才有`）。


实例化一个类，产生一个对象，可以使用(),{}（`C++11起才有`）和 new 。



## 转换构造函数


在C++中，C++11之前，implicit 的单个非默认参数的构造函数被称为转换构造函数。C++11之后，implicit 的多个包含单个非默认构造函数称为转换构造函数，因为C++11 中可以使用｛｝对多个参数来转换为对象。使用explicit 来修饰这样的构造函数则不能进行这样的转换。


```c++
#include<iostream>

class Base
{
private:
	int _int_var;
	double _double_var;
public:
	Base():_int_var(int(0)),_double_var(double(0))
	{
		std::cout<<"default constructor"<<std::endl;
	}
	
	explicit Base(int __int_var):_int_var(__int_var),_double_var(double(0))
	{
		std::cout<<"int constructor"<<std::endl;
	}
	
	explicit Base(double __double_var):_int_var(int(0)),_double_var(__double_var)
	{
		std::cout<<"double constructor"<<std::endl;
	}
	
	Base(int __int_var,double __double_var):_int_var(__int_var),_double_var(__double_var)
	{
		std::cout<<"int and double constructor"<<std::endl;
	}
	
	void show()
	{
		std::cout<<"int:"<<_int_var<<" double:"<<_double_var<<std::endl;
	}
};

Base get_base(int _int_var,double _double_var)
{
	return {_int_var,_double_var};			/* since c++11 */
}

int main()
{
	//Base b_0 = 123;				/* error, int constructor is explicit */
	//Base b_1 = 2.33;				/* error,double constructor is explict */
	Base b_2 = {1342,23.432};			/* since c++11 */
	Base b_3 = get_base(1,1.1);
	
	//b_0.show();
	//b_1.show();
	b_2.show();
	b_3.show();
	
	return 0;	
}
```


## 拷贝构造函数 赋值运算符函数


拷贝构造函数语法


**class_name(const class_name&)**


赋值运算符函数语法


**class_name& class_name:: operator=(class_name)**


**class_name& class_name:: operator=(class_name&)**


为什么参数需要const class_name&?


C++有如下的引用绑定规则：


* 非常量左值引用(X&) : 只能绑定到X类型的左值对象。
* 常量左值引用(const X&) : 可以绑定到X、const X 类型的左值对象，或X、const X类型的右值。


C++11 新增有如下的引用绑定规则：


* 非常量右值引用(X&&) ： 只能绑定到Ｘ类型的右值。
* 常量右值引用(const X&&) : 可以绑定到X、const X 类型的右值。、



```c++
#include<iostream>

class Base
{
private:
	int _int_var;
public:
	Base():_int_var(int(0))
	{
		std::cout<<"default constructor"<<std::endl;
	}
	
	Base(int __int_var):_int_var(__int_var)
	{
		std::cout<<"int constructor"<<std::endl;
	}
	
	Base(const Base& _b):_int_var(_b.get_int_var())
	{
		std::cout<<"copy constructor"<<std::endl;
	}
	
	int get_int_var() const
	{
		return this->_int_var;
	}
	
	void show()
	{
		std::cout<<this->_int_var <<std::endl;
	}
	
	Base& operator = (Base& _b)
	{
		this->_int_var = _b._int_var ;
		std::cout<<"copy assignment"<<std::endl;
		return *this;
	}
};


Base get_base(Base _b)
{
	return _b;
}

Base get_base()
{
	return Base(2239);
}

int main()
{
	Base b(1);			/* call int constructor */
	Base bb = b;		/* call copy constructor */
	Base bbb;			/* call default constructor */
	bbb = b;			/* call copy assignment */
	
	Base bbbb = get_base();		/* call function retuan a object */
	
	Base bbbbb = get_base(b);   /* give a object and get a object */
	
	b.show();
	bb.show();
	bbb.show();
	bbbb.show();
	bbbbb.show();
	return 0;
}

```


拷贝构造函数，在向一个函数以值传递的方式传递一个对象，需要调用；在以值的方式返回一个对象时，会调用拷贝构造函数产生一个临时的对象，而在将这个临时对象赋值给一个对象时，又需要调用拷贝构造函数。但是，运行上述程序，会发现拷贝构造函数的调用次数并没有像分析中那样多。这是为什么呢？在C++中，存在copy constructor elision，编译器会进行优化，尽量少产生中间临时对象和多余的对象。


## 移动构造函数 移动运算符函数


在C++11之前，能被寻址的称为左值，所有不是左值的表达式的值是右值。而在C++11之后，右值引用就是为了实现移动语义与完美转发所需要而设计出来的新的数据类型。右值引用的实例对应于将亡对象，右值引用并区别于左值引用，而作形参时能重载辨识（overload resolution）是调用拷贝语义还是移动语义的函数。


使用std::move() 能将一个对象转为右值引用，std::move 相当于 static_cast<char&&>。




```c++
#include<iostream>

class Base
{
private:
	int _int_var;
public:
	Base():_int_var(int(0))
	{
		std::cout<<"default constructor"<<std::endl;
	}
	
	Base(int __int_var):_int_var(__int_var)
	{
		std::cout<<"int constructor"<<std::endl;
	}
	
	Base(Base&& _b)
	{
		this->_int_var = _b.get_int_var();
		std::cout<<"move constructor"<<std::endl;
	}
	
	int get_int_var()
	{
		return _int_var;
	}
};

int main()
{
	Base b(2323);
	Base bb = std::move(b);						/* call move constructor */
	
	std::cout<<b.get_int_var()<<std::endl;
	std::cout<<bb.get_int_var()<<std::endl;
	
	return 0;
}
```


## 实例化类


在C++11，可以统一使用｛｝对任何类型进行赋值。


基本类型可以使用


```c++
int i = {1};
double d = {3.3}
```
 

类也可以使用{}实例化(`需要类支持，并不是所有的类使用都会得到正确的实例化`)


```c++
Class_name cn = {var1,var2,...} 
Class_name cnn{var1,var2,...}
```
