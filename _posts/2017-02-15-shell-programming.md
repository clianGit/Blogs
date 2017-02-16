---
layout: post
title:  "shell programming"
author: chenlian
categories: shell
tags: shell Linux
excerpt: The shell is an application that allows users to communicate with the computer. It is a text based application that allows programs to be started and tasks to be run. 
---


* content
{:toc}
The shell is an application that allows users to communicate with the computer. It is a text based application that allows programs to be started and tasks to be run. 


## shell 变量


* $$ 当前shell的pid号
* $? shell执行的结果
* $0 shell运行的命令名
* $1 第一个参数
* $2 第二个参数
* $* 所有的参数
* $# 参数的个数


$(a) = $a


$(A)B != $AB


其中，$() 是用来做命令替换的。


* ${var#*c}   拿掉变量var第一个c代表的字符及其左边的字符串
* $(var##*c}  拿掉变量var最后一个c代表的字符及其左边的字符串
* $(var%c*}   拿掉变量var最后一个c代表的字符及其右边的字符串
* $(var%%c*}  拿掉变量var第一个c代表的字符及其右边的字符串
* $(var:n:m}  提取变量var第n个字节右边连续m个字节
* $(#var}     获取变量var的长度


shell 数组


varname=(var1 var2 var3 ...)


* ${varname[@]} 或 ${varname[*]}  得到全部的数组值
* ${varname[0]} 得到varname数组的第1个值
* ${#varname[@]} 或 ${#varname[*]} 得到数组的数量
* ${#varname[0]} 第一个数组元素值的长度


## shell 数字运算


$(()) 可用来整数运算，整数运算算符有：


* \+ 加
* \- 减
* \* 乘
* / 除
* % 取余
* & 与
* \| 或
* ^ 异或
* ！非

expr 命令计算运算


> echo $(expr 1 + 2)  或  var=\` expr 1 + 2 \`



let 命令计算


> a=5;let a=a+5;echo $a



## shell 语法


### if


```shell
if [ 判断条件 ]
then
	...
fi

if [ 判断条件 ] && 或 || [ 判断条件 ]
then
	...
fi

#嵌套
if [ 判断条件 ]
then
	...
elif [ 判断条件 ]
then
	...
else
	...
fi
	
```


判断使用


* -eq		相等
* -ge		大于或等于
* -gt		大于
* -le		小于或等于
* -lt		小于
* -ne		不相等


或者使用<,<=,>,>=,==


```shell
if((判读条件))
then
	...
fi
```


### for


```shell
for var in var1 var2 var3 ...
do
	...
done


for((初始值；限制值；执行步长))
do
	...
done
```


### while until


```shell
while [ 条件 ]
do
	...
done


until [ 条件 ]
do
	...
done
```


### case


```shell
case $变量名 in 
	"匹配内容1")
		...
	;;
	"匹配内容2")
		...
	;;
	*)	#匹配其他情况
		...
	;;
esac
```



### function


```shell
function function_name{
	...
}

function_name() {
	...
}
```


## 文件重定向


* command > filename  2>&1  把标准输出和标准错误一起重定向到一个文件中。
* command >> filename 2>&1   把标准输出和标准错误一起重定向到一个文件中（追加）
* command < filename > filename2  command 命令以filename文件作为标准输入，以filename2文件作为标准输出
* command < filename   command 命令以filename文件作为标准输入。
* command  << delimiter  以标准输入中读入，直至遇到delimiter分界符。
* command <&m   以文件描述符m作为输入
* command >&m   以标准输出重定向到文件描述符m中
* command >&-    关闭标准输入


## 通配符


* \* 匹配0或多个字符
* ？ 匹配任意单个字符
* [list] 匹配list中的任意单个字符
* [!list] 匹配不在list中的任意单个字符
* ｛string1，string2，...｝ 匹配string1 或 string2 (或更多）中的一个字符串


## 例子


测试网段能ping 通的ip，并保存到文件中


```shell
#!/usr/bin/bash

for i in {1..100}
do
        ping -c 1  172.16.1.$i >&/dev/null
        if [ $? -eq 0 ];  then
                echo "succeed ping 172.16.1.$i"
                echo 172.16.1.$i >> ip_succeed_ping.txt
        fi
done

```