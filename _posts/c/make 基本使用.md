---
title: "make 基本使用"
date: 2022-04-05T22:09:07+08:00
draft: false
tags: ["makefile", "c语言"]
---

# 什么是 make ？
什么是 make: [Introduction](https://www.gnu.org/software/make/manual/make.html#Overview)

> The `make` utility automatically determines which pieces of a large program need to be recompiled, and issues commands to recompile them.
> make 指令自动化地决定一个大型程序的哪部分需要重新编译，并且发出指令重新编译。

简单理解就是，gcc 的编译指令太繁琐并且手写不方便，需要几个工具来管理。这就是 make 工具干的事儿

# 什么是 makefile ？
[Introduction](https://www.gnu.org/software/make/manual/make.html#Introduction)

> You need a file called a _makefile_ to tell `make` what to do. Most often, the makefile tells `make` how to compile and link a program.
> 需要使用 makefile 这个文件，来告诉 make 需要做什么。常见的用法是描述如何编译以及链接程序


makefile 就是编译链接的描述文件。


# makefile 基本使用


```
make makefile
```

## 一个规则
为了编译出 target， 会找 prerequisites 文件，如果找不到会在 makefile 中寻找将 prerequisites 作为目标的配置，先生成 prerequisites 文件

```
target … : prerequisites …
        recipe
        …
        …
```

- 目标（target）：需要生成的文件
- 依赖条件（prerequisites）：生成目标所需要的文件
- 配方，指令（recipe）：真正执行的命令

clean: （没有依赖）

	-rm -rf $(obj) a.out   `"-"` 作用是删除不存在的文件时，不报错，继续执行

ALL: (跟最终目标，否则将第一个目标作为最终目标)


## 两个函数
1. `wildcard` 通配符，能够匹配所有的 .c 结尾的文件，作为列表放到 src 变量中
```
src = $(wildcard ./*.c)
```
2. `patsubst` 文本替换 ，将所有 %.c 的内容替换成 %.o
```
obj = $(patsubst %.c, %.o, $(src))
```


## 三个自动变量
- $@: 表示规则中的目标（target）
- $^: 表示规则中的`所有`依赖条件
- $<: 表示规则中的第一个依赖条件，并且在模式规则（%.o:%.c）中，代表每次使用一个依赖生成一个目标

## 伪目标

```makefile
.PHONY: clean ALL
```

伪目标的作用是为了防止你的文件夹中有相应的叫 `clean` 或者 `ALL` 文件的时候，`make` 将其当作目标文件。

# 小练习

> [!note] 小练习
> 
> 编写一个 makefile 可以将其所在目录下的所有独立 .c 文件编译生成同名可执行文件


``` makefile
src=$(wildcard *.c)
targets=$(patsubst %.c, %, $(src))

ALL:$(targets)

%:%.c
	cc $< -o ./bin/$@

clean:
	-rm -rf ./bin/*

.PHONY: clean ALL
```
