---
title: Linux工具之make/Makefile
date: 2023-12-29 21:50:01
tags: [Linux,makefile]
categories: [Linux]
---

## make/Makefile

makefile实际上是一个自动化构建项目的工具，他是对大型项目的编译工作的集成化处理，他可以处理文件的编译顺序，是否编译，以及对于代码的更复杂的操作

make是一个命令工具，大多数的ide都有这个命令，比如说Delphi的make等

make是一条命令，而makefile是一个文件，我们需要搭配使用完成项目的自动化构建

这里我们演示一下，具体的使用

```c
#include<stdio.h>
int main()
{
    printf("hello makefile\n");
    return 0;
}
```

然后我们创建一个makefile文件

内容如下

```makefile
test: test.c
	gcc -o test test.c
	
.PHONY:clean
clean:
     rm -f *.o test 
```

我们依次来说说这几个命令的意思，第一个test就是会生成的目标的名称，test.c是指的是依赖

目标指的是要编译的目标，也可以是一个动作，依赖可以理解为是项目的源文件

第二行指的是目标下要执行的具体命令，可以没有，也可以有很多条，需要注意的是，如果有很多条，就只写一行

这里的phony意味着伪目标，他的特性就是总是被可执行的

当我们调用这个make命令时，只使用make时，会自动调用第一个目标，也可以把项目名称作为命令之一，调用clean时，需要写clean了

例如

```bash
make
make test
make clean
```

当然如果clean是第一条就不用了

为什么我们说伪目标总是可执行的呢，因为如果编译过了一次源文件，再次编译时会自动检查，如果源文件没有被修改的话，则不会被再次编译，这样做有利于在大量工程的情况下节省系统的开销
