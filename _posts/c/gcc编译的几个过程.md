---
title: "gcc编译的几个过程"
date: 2022-03-26T22:22:18+08:00
draft: false
tags: ["gcc", "c语言"]
---

一段普通的 `c++` 代码是如何变成可执行程序的 (使用 `g++` 命令编译 `c++` 代码)

``` cpp
using namespace std;

int main() {
    cout << "hello, world" << endl;
    return 0;
}
```

会经过4个步骤
- 预处理
- 编译
- 汇编
- 链接
# 预处理
作用：`宏替换`（展开文本，include 文本替换），删除空格等`文本处理`

相关语句 -E 仅生成 .i 预处理后的文件

![](Pasted%20image%2020220327002340.png)
# 编译
-S 仅生成 .s 编译后 .s 后缀文件

将c代码解析后生成汇编语句

下面代码是一部分汇编语句

```
ubuntu@foo:~/test$ cat test.s
	.arch armv8-a
	.file	"test.c"
	.text
	.section	.rodata
	.align	3
	.type	_ZStL19piecewise_construct, %object
	.size	_ZStL19piecewise_construct, 1
_ZStL19piecewise_construct:
	.zero	1
	.local	_ZStL8__ioinit
	.comm	_ZStL8__ioinit,1,8
	.align	3
.LC0:
	.string	"hello, world"
	.text
	.align	2
	.global	main
	.type	main, %function
main:
.LFB1521:
	.cfi_startproc
	stp	x29, x30, [sp, -16]!
	.cfi_def_cfa_offset 16
	.cfi_offset 29, -16
	.cfi_offset 30, -8
	mov	x29, sp
	adrp	x0, .LC0
	add	x1, x0, :lo12:.LC0
	adrp	x0, :got:_ZSt4cout
	ldr	x0, [x0, #:got_lo12:_ZSt4cout]
	bl	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
	mov	x2, x0
	adrp	x0, :got:_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_
```


# 汇编
将汇编语句转换成 `机器码` (二进制)

-c 仅生成汇编后的 .o 后缀文件

![](Pasted%20image%2020220327002354.png)

# 链接
将静态链接库与动态链接库链接到程序中

生成的可执行文件中会多很多信息

![](Pasted%20image%2020220327002406.png)
# 总结
经过4个步骤处理之后，最终就可以执行了

![](Pasted%20image%2020220327002420.png)

OK，精通 hello world