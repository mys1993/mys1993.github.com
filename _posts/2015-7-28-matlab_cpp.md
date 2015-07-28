---
layout: post
title: Matlab C++ 混合编程笔记
category: 杂
description: 给朱焕同学整理的笔记o(∩_∩)o
tags: 
    - MATLAB
    - C++
---

##方法一：直接调用C/C++生成的DLL

###用VS生成dll
新建一个win32 dll工程，工程名为test，如果你的MATLAB和系统都是64位的，这里要修改平台参数到64位。
例如要导出`add()`函数，那么头文件里就要写：
```C++
//main.h
extern "C" double __declspec(dllexport)add(double x, double y);
```
注意`extern "C"`不能掉，表明它按照C的编译和连接规约来编译和连接[^1],你可以认为这里的C++函数将要给C调用，因为下面讲到的MATLAB的`loadlibrary()`函数只能识别C格式的DLL。
源文件内容为：
```C++
//main.cpp
#include "main.h"
double add(double a, double b)
{
	return a + b;
}   
```
这里的函数返回类型必须是**数值类型**。
例如编写返回字符的函数 *rep_str*  
```C++
char rep_str(char s)
{
	return s;
}
```
在Matlab中调用返回错误:

    >> calllib('test','rep_str','a')
    Error using calllib
    Array must be numeric or logical.


###在matlab中调用dll中的函数
将生成的test.dll与这里的 *main.h* 头文件放在同一个目录下,把`extern "C"`删除,因为此时相当于**在C中调用C++里的函数**。
在matlab中使用`loadlibrary()`函数载入dll，使用`libfunctions()`函数查看该dll中包含的函数。
运行结果：

    >>loadlibrary('test.dll', 'main.h')
    >>libfunctions('test')
    
    Functions in library test:
    
    add      rep_str  

使用`calllib()`函数调用dll中的函数：

    >> calllib('test', 'add', 8, 13)

    ans =
    
        21

###常见错误与注意事项
1. 参数个数必须匹配

        >> calllib('test', 'add', 8)
        Error using calllib
        No method with matching signature.
    
2. 参数必须是**数值类型**或**逻辑类型**

        >> calllib('test', 'add', 'a', 'b')
        Error using calllib
        Array must be numeric or logical.

3. 且必须是**标量**
    
        >> calllib('test', 'add', [1,2], [3,4])
        Error using calllib
        Parameter must be scalar.
 
4. dll文件不能在某个磁盘的根目录，如`"F:\"`

5. 头文件的编写最好统采用如下形式：
```C++
#ifdef __cplusplus
extern "C" {
#endif

//exported functions...

extern Type1 func1(...);
extern Type2 func2(...);
...

#ifdef __cplusplus
}
#endif

```
这样就不需要二次修改头文件了。


##方法二：使用mex编译C++代码

不细讲了，见参考文献3[^2]吧。
    
##参考文献
---

1. [c++中的 extern "C"](http://songpengfei.iteye.com/blog/1100239)
2. [C++项目中的extern "C" {}](http://www.cnblogs.com/skynet/archive/2010/07/10/1774964.html)
3. [Matlab与C/C++混合编程接口及应用](http://www.cnblogs.com/lidabo/archive/2012/08/24/2654148.html) 

[^1]: 见参考文献2

[^2]: 讲解了mexFunction方法