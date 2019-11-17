+++
title = "环境需求"
toc = true
type = "docs"
draft = false
date = "2018-09-19"
lastmod = "2018-09-20"
weight = 103

[menu.v1]
  parent = "base-info"
  weight = 3
+++
## 必须安装的

- 安装PHP并且版本至少 `>7.0`，推荐 `7.1+`
- 安装php包管理器 `composer`
- 安装redis的异步客户端 `hiredis`
- 连接迭代器依赖 `pcre` 库
- 安装php扩展swoole, 并且版本至少 `>=2.1`
  - swoole开启协程和异步redis
- 其他需要安装和启用的php扩展有：`PDO`

## 有冲突的

下面列出一些已知的和swoole有冲突的php扩展，请使用swoft时不要安装或禁用它们：

- `xdebug`
- `xhprof`
- `blackfire`
- `zend`
- `trace`
- `uopz`

## 推荐环境配置

可以查看 [swoft](https://github.com/swoft-cloud/swoft) 下 [Dockerfile](https://github.com/swoft-cloud/swoft/blob/master/Dockerfile) 文件
