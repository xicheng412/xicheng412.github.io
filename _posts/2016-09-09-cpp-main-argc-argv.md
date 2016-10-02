---
layout: post
title:  "C++ main(int argc,char* argv[]) 详解"
date:   2016-09-09 00:00:00
categories: Cpp
tags: Cpp
---

* content
{:toc}

解决之前不太清楚的细节问题，同时测试一下 blog 的代码显示功能。所以写了一个关于 main() 的参数问题。





## 简介
argv 和 argc 是命令行传递给 C 和 C++ 中 main() 函数的参数

* argc 是指代的 **argv 这个 string 字符串长度** 。 这个长度是参数个数 +1 。  （argument count）
* argv 是一个包含多个字符串的数组，包含执行的文件名和所附带的参数。 (argument vector)

在 main() 中，也可以写成其他的名字例如

```c
int main(int num_args, char** arg_strings) 
```

这样也是等价的。


## 示例代码

下面是一些测试代码，可以演示这个功能：

```c
#include <iostream>

int main(int argc, char** argv) {
    std::cout << "Have " << argc << " arguments:" << std::endl;
    for (int i = 0; i < argc; ++i) {
        std::cout << argv[i] << std::endl;
    }
}
```

## 结论解析

编译之后可以运行查看结果：

```
./test a1 b2 c3
	
Have 4 arguments:
./test
a1
b2
c3
```

可以看到

变量 | 含义
--- | ---
argc | 是指字符串组数，这里是等于4
argv[0] | 是特殊的,是文件执行的命令
argv[1]-argv[3] | 是跟的参数
