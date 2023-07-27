---
title: "vscode debug nginx 源码"
date: 2022-03-18T23:32:05+08:00
draft: false
tags: ['nginx', 'vscode', 'debug']
---

# 1 通过源码安装 `nginx`

## 1.1 下载源码
```bash
git clone https://github.com/nginx/nginx
```

## 1.2 编译并安装 `nginx`
修改 nginx 源码中的配置文件：`auto/cc/conf` 

将第 `12` 行修改一下

`ngx_compile_opt="-c"` 改成： `ngx_compile_opt="-c -g"`

`-g` 选项会让编译器产生调试信息

执行配置并安装：

```
cd nginx
auto/configure
```

默认被安装在 `/usr/local/nginx` ，可以通过 `--prefix` 选项修改

继续执行 `make && make install`

到这里，nginx 就安装完毕了

## 1.3 启动 nginx

```
/usr/local/nginx
```

默认监听本机 80 端口

完成设置后会出现下面的图片：

`Welcome to nginx!`

![](Pasted%20image%2020220318234805.png)

# 2 配置 vscode


## 2.1 打开项目
使用 `vscode` 打开下载下来的`源码文件夹`

## 2.2 安装调试插件

``` plain
名称: CodeLLDB
ID: vadimcn.vscode-lldb
说明: A native debugger powered by LLDB.  Debug C++, Rust and other compiled languages.
版本: 1.7.0
发布者: Vadim Chugunov
VS Marketplace 链接: https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb
```

在插件库搜索此插件并安装

## 2.3 调试配置
如图，打开调试文件：

![](Pasted%20image%2020220318235027.png)


下面是调试的配置文件，`pid` 可以设置成 `master` 或者 `worker` 的 `PID`
```json
{
	// 使用 IntelliSense 了解相关属性。 
	// 悬停以查看现有属性的描述。
	// 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
	"version": "0.2.0",
	"configurations": [
		// 监听工作进程
		{
			"name": "(lldb) Attach",
			"type": "lldb",
			"request": "attach",
			"pid": "95318", // 填写 Worker 进程 PID， 需要修改
		}
	]
}
```

我这里设置成了 `worker` 进程的 `PID`

到这里，`vscode` 中的配置就完成了

# 3 断点调试
在 `ngx_http_parse_request_line` 设置断点

再次通过浏览器请求 `localhost` ，会发现 `vscode` 上方出现了调试用的几个按钮以及停在了断点处

![](Pasted%20image%2020220319000253.png)


# 4 参考

- [Mac上用Visual Studio Code调试Nginx](https://www.jianshu.com/p/51b726b56e2f) 这个教程在 M1 芯片的 mac 上面无法使用
- [GCC 参数详解](https://www.runoob.com/w3cnote/gcc-parameter-detail.html)