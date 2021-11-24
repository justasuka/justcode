---
title: goland导入自定义包
date: 2021-11-05 11:59:57
tags: ['go']
cover: https://s3.bmp.ovh/imgs/2021/10/52fd491754edacb4.png
---



### go的依赖管理

+ gomod

>gomod的方式管理包，解析go.mod文件查找包。mod包名就是包的前缀

+ gopath

> gopath方式，按照goroot和多个gopath目录下的src查找包

**两种方式互不兼容**

关闭gomod模式

~~~sh
go env -w GO111MODULE=off
~~~

在导入自定义包的情况下在gopath下寻找包会比较方便



### goland 2021.2.4下遇到的问题

在setting里的设置gopath下依次是 

1. Global GOPATH
2. Project GOPATH
3. Module GOPATH

问题是设置了2后无法导入自定义的包。还需要在Module GOPATH里设置一遍
