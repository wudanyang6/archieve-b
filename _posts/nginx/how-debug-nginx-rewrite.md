---
title: "如何调试nginx的rewrite规则?"
date: 2021-04-23T14:58:47+08:00
draft: false
tags: ['nginx', 'debug', 'rewrite log', '配置']
---

# 如何调试nginx的rewrite规则？



设置 `rewrite_log on;`

并且将`error_log` 的报错等级改成 `notice`;



官网对 `rewrite_log` 的解释：

```plain
Syntax:	rewrite_log on | off;
Default:	
rewrite_log off;
Context:	http, server, location, if
Enables or disables logging of ngx_http_rewrite_module module directives processing results into the error_log at the notice level.
```

可以看到，这个指令只能在 `http` `server` `location` `if` 配置指令中使用，日志等级是 `notice` 写入到 `error_log` 配置的文件中。



# 示例

url： `/q?someparams`

```sh
2021/04/22 18:21:28 [notice] 18131#0: *102982 "^/+m/question(.*)?qid=\d+(.*)?$" does not match ...
... 中间省略一些日志
2021/04/22 18:21:28 [notice] 18131#0: *102982 "^/+question/(\w+.*?)$" does not match ...
2021/04/22 18:21:28 [notice] 18131#0: *102982 "^/+q\?(.*)?tn=nsatom_qb_main(.*)?$" matches ...
2021/04/22 18:21:28 [notice] 18131#0: *102982 "^/+q" matches ...
2021/04/22 18:21:28 [notice] 18131#0: *102982 rewritten data: "/q/q/q/q/q"
```

从上面的日志中可以看出，当触发了重写规则时，会有关键字 `matches` 否则会有 `does not match` 。

最终重写完成之后，访问的地址会有标记：`rewritten data`

这里就是 `/q/q/q/q/q`



# 参考

[best way to debug nginx rewrite rules in config file?](https://serverfault.com/questions/333048/best-way-to-debug-nginx-rewrite-rules-in-config-file)

[nginx官方文档：Module ngx_http_rewrite_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html)
