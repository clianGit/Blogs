---
layout: post
title:  "C++中虚析构函数的调用顺序"
author: chenlian
categories: C++
tags: C++ destructor
excerpt: C++ 中，子类的在销毁时，调用析构函数的顺序。
---


* content
{:toc}
C++ 中，子类的在销毁时，调用析构函数的顺序。

## C++ 析构函数

`C++ 析构函数`(*Destructor*)是一种特殊的成员函数。析构函数与类同名，并在前面加~，它没有参数和返回值，在对象销毁的时候执行。但在C++中，对象可以分配在`自由存储区`(*free store*，*`Note：The free store is one of the dyanmic memory areas,allocated/freed by new/delete;the heap is the other memory area,allocated/freed by malloc/free and their variants.In C++,the default global new and delete might be implemented in terms of malloc and free by a particular complier`*)和`栈区`（*stack*）。因此，对象的销毁存在两种方式:

* **1、当对象分配在栈区时，并且代码执行流程离开对象的作用域时，对象会自动调用自己的析构函数进行销毁。**

* **2、当对象通过new运算符分配在自由存储区时，销毁需要程序员手动调用delete运算符进行销毁。**

## C++ 虚函数

c++虚函数就是在函数前加上`virtual`关键字，子类可以根据自己的需要完全重写父类函数内的实现，但是不能修改函数的函数名，函数参数，返回值。在有虚函数的对象中，存在着虚函数表的地址(`Note:在单继承中，父类和子类均只存在一个虚函数表的地址，而在当子类从两个或两个以上的父类进行多继承时，子类存在多个虚函数表的地址`），而在此虚函数表中存放着虚函数的地址(`函数代码编译的可执行二进制指令存放在程序结构中的代码段`)。类的对象中，虚函数表中存放着是自己实现的虚函数地址。父类的类型指针可以指向子类的对象(`因为子类一定是父类，就如同cat类继承于animal类，于是任何cat都是animal`)，而该对象需要根据具体是哪个类型的实例来调用相应的虚函数，这就实现了运行时多态。

## 程序和结果

**Talk is cheap , put the code !**



* **析构函数没有virtual，对象分配在堆区**


代码：


```c++
#include<iostream>
#include<cstdlib>

using std::cout;
using std::endl;

class Father
{
private:
	int _Int_father;		/* no use,just show the memory layout of Father class's object */
public:
	 ~Father()
	{
		cout << "father destory!!" << endl;
	}
};

class Son :public Father
{
private:
	int _Int_son;			/* no use,just show the memory layout of Son class's object */
public:
	~Son()
	{
		cout << "son destory!!" << endl;
	}
};



int main()
{
	{
		Son son;
	}						/* while leaving '}',son call the Son destructor */

	system("pause");
	return 0;
}

```


结果：


![](http://chenlian.tech/img/2017-01-15/1.jpg)



* **析构函数没有virtual，对象分配在自由存储区**

```c++
#include<iostream>
#include<cstdlib>

using std::cout;
using std::endl;

class Father
{
private:
	int _Int_father;		/* no use,just show the memory layout of Father class's object */
public:
	 ~Father()
	{
		cout << "father destory!!" << endl;
	}
};

class Son :public Father
{
private:
	int _Int_son;			/* no use,just show the memory layout of Son class's object */
public:
	~Son()
	{
		cout << "son destory!!" << endl;
	}
};



int main()
{
	
	Father * son = new Son();

	delete son;

	system("pause");
	return 0;
}

```


结果：


![](http://chenlian.tech/img/2017-01-15/2.jpg)


* **析构函数有virtual，对象分配在堆区**


代码：


```c++
#include<iostream>
#include<cstdlib>

using std::cout;
using std::endl;

class Father
{
private:
	int _Int_father;		/* no use,just show the memory layout of Father class's object */
public:
	 virtual ~Father()
	{
		cout << "father destory!!" << endl;
	}
};

class Son :public Father
{
private:
	int _Int_son;			/* no use,just show the memory layout of Son  class's object */
public:
	virtual ~Son()
	{
		cout << "son destory!!" << endl;
	}
};



int main()
{
	
	{
		Son son;
	}

	system("pause");
	return 0;
}

```


结果：


![](http://chenlian.tech/img/2017-01-15/3.jpg)


* **析构函数有virtual，对象分配在自由存储区**


```c++
#include<iostream>
#include<cstdlib>

using std::cout;
using std::endl;

class Father
{
private:
	int _Int_father;		/* no use,just show the memory layout of father class's object */
public:
	 virtual ~Father()
	{
		cout << "father destory!!" << endl;
	}
};

class Son :public Father
{
private:
	int _Int_son;			/* no use,just show the memory layout of son class's object */
public:
	virtual ~Son()
	{
		cout << "son destory!!" << endl;
	}
};



int main()
{
	
	Father * son = new Son();

	delete son;

	system("pause");
	return 0;
}

```


结果：


![](http://chenlian.tech/img/2017-01-15/4.jpg)


## 分析

当析构函数无`virtual` 关键字时，Son 类的对象内存布局

```
1>  class Son	size(8):
1>  	+---
1>   0	| +--- (base class Father)
1>   0	| | _Int_father
1>  	| +---
1>   4	| _Int_son
1>  	+---
```

当析构函数有`virtual` 关键字时，Son 类的对象内存布局

```
1>  class Son	size(12):
1>  	+---
1>   0	| +--- (base class Father)
1>   0	| | {vfptr}
1>   4	| | _Int_father
1>  	| +---
1>   8	| _Int_son
1>  	+---
1>
1>  Son::$vftable@:
1>  	| &Son_meta
1>  	|  0
1>   0	| &Son::{dtor}
```


(`Note：以下分析中son,Son,father,Father 中，开头为小写为对象，即object，开头大写为对象，即class`）


一个变量的类型是用来解释变量的，当一个son对象的类型是Father * 类型时，这个对象的指针的有效范围为Son class 对象中Father base class的范围，也就不能访问到除Father base class的son 类对象中的内容（`Note：析构函数不为virtual时，父类析构函数属于Father base class，而子类析构函数属于Son class，而不属于Father base class`）。因此，当一个子类对象的类型为父类时，其并不能调用到属于子类对象而不属于父类对象中的析构函数(此时，析构函数为非virtual)。而当一个son对象的类型是Son *　类型时，Son对象调用析构函数，可以访问到Son class 的析构函数，并且当son对象调用Son class 的析构函数时，son对象会销毁属于Son class 的而不属于Father class 的部分，也就是说当son对象调用Son class 的析构函数完成时，son对象会变成一个father对象，并且会再自动调用Father class 的析构函数。此过程会持续进行，直到对象完全销毁。（`Note：一个子对象构造时，会先调用父类的构造函数，再调用子类的构造函数，并且构造函数与析构函数不同，不能为virutal，且在对象构造时会按照固定的顺序自动调用。而当子类对象能访问子类和父类的析构函数时，顺序与构造函数相反，会先调用子类的析构函数，在调用父类的析构函数`）


而当析构函数为`virutal`类型时，子类和父类的虚析构函数会放在虚函数表中（子类和父类对象都可以访问得到），并且会根据自己属于什么类的对象而所存放的虚析构函数的地址不同（其他虚函数一样）。当当前对象是子类对象时，虚函数表中虚函数的地址为子类型的虚构函数的地址。当是父类型时，为父类型的虚构函数的地址。因此无论son对象的类型为Father * 类型还是Son * 类型，son 对象都可以访问到虚函数表中的虚析构函数，并且此时的地址为Son class 的析构函数。且当son 对象销毁属于Son class 的而不属于Father class的部分，也就是说son对象根据虚函数表中析构函数的地址调用Son class 的析构函数完成时，son对象会变成一个Father class 对象，此时，虚函数表中析构函数的地址会变成Father class 的析构函数地址。因此，该对象会自动调用该析构函数，完成对象的彻底的销毁。





