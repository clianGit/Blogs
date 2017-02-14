---
layout: post
title:  "C++ parameter pack and template function in template class"
author: chenlian
categories: C++
tags: C++ Generic-programming
excerpt: C++ is not c with class, template makes C++ powerful。
---


* content
{:toc}
C++ is not c with class, template makes C++ powerful。


## 简介


A template parameter pack is a template parameter that accepts zero or more template arguments (non-types,types,or templates). A function parameter pack is a function parameter that accepts zero or more function arguments. A template with at least one paramenter pack is called a variadic template.


A function in a template class can also accepts others template arguments.




## parameter pack


早期的 C 和 C++ 处理变参的能力，但 parameter pack 给与 C++ 更强大的处理能力。


网上的例子：


```c++
#include<iostream>

void tprintf(const char * format)
{
	std::cout<<format;
}

template<typename T , typename ... Targs>
void tprintf(const char * format,T value,Targs ... Fargs)
{
	for(;*format != '\0';format++){
		if(*format == '%'){
			std::cout<<value;
			tprintf(format + 1,Fargs...);
			return;
		}
		std::cout<<*format;
	}
}

int main(void)
{
	tprintf("% % % %",1,2.2,"hello",std::string("world!"));
	return 0;
}
```


自己写的例子：


```c++
#include<iostream>

void show(int num)
{
}

template<typename T,typename ... Targs>
void show(int num,T value,Targs ... Fargs)
{
	if(num <=0 ){
		return;
	}
	else{
		std::cout<<value<<std::endl;
		show(num - 1,Fargs...);
	}
}

int main(void)
{
	show(4,1,2.2,"hello",std::string("world!"));
	return 0;
}
```


## template function in template class


在template class中，成员函数可以拥有自己的template arguments，而不予class中其它的共享。


一个简单的例子：


```c++
#include<iostream>

template<typename T>
class Base
{
private:
	T _T_var;
public:
	Base():_T_var(T(0)){}
	
	Base(T __T_var):_T_var(__T_var){}	
	
	template<typename U>
	void show(U _U_var);
};

template<typename T>
	template<typename U>
	void Base<T>::show(U _U_var)
	{
		std::cout<<_T_var<<" "<<_U_var<<std::endl;
	}
	
	
int main(void)
{
	Base<int>(1).show(2.2);
	return 0;
}
```


