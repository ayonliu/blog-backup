---
title: 利用 XHProf 对 PHP 代码进行性能分析
date: 2018-11-23 19:38:15
tags:
    - php
    - xhprof
---

遇到程序执行速度慢的情况，除了看服务器系统状态、mysql slow log外，还可以对代码进行性能分析来定位问题。最简单常用的办法是代码不同的地方记录日志来分析程序各个节点的执行时间。[XHProf](http://pecl.php.net/package/xhprof)可以高效地对php代码进行性能分析。

## 介绍

XHProf 是一个轻量级的分层性能测量分析器的PHP扩展。 可以跟踪调用次数、运行时间、CPU 计算时间和内存开销，展示程序动态调用的路径。XHProf 提供一个基于 HTML 的简单用户界面来展示分析数据，也支持图片的形式查看调用路径。

## 安装

Centos

```shell
pecl install xhprof
```
这是会报如下错误：
```
Failed to download pecl/xhprof within preferred state “stable”, 
latest release is version 0.9.4, stability “beta”, 
use “channel://pecl.php.net/xhprof-0.9.4” to install
```
因为 xhprof 是 beta 版本，不是 stable 版，要用下面命令安装：
```shell
pecl install channel://pecl.php.net/xhprof-0.9.4
```

为了后面可以用图片的形式展示，需要安装 graphviz:
```shell
yum install graphviz # 支持图形化展示结果
```

在 php.ini 中加载扩展，配置生成 profile 数据的存放目录：

```
extension=xhprof.so
xhprof.output_dir=生成数据的存放目录
```

要通过 web 形式查看分析结果，需要[下载xhprof源码](http://pecl.php.net/get/xhprof-0.9.4.tgz)(里面包括图形化展示分析结果所需的代码和相关类库)放到 web 可以访问到的目录。xhprof 目录结构如下所示：

```
bin
CHANGELOG
CREDITS
examples
extension
LICENSE
package.xml
README
scripts
support
xhprof_html
xhprof_lib
```

## 使用

以一段简单的代码举例：
```php
<?php
include(realpath(dirname(__FILE__)."/vendor/autoload.php"));

// 在开始分析的地方加上 xhprof_enable() 方法
// XHPROF_FLAGS_NO_BUILTINS 表示不分析 PHP内置函数
xhprof_enable(XHPROF_FLAGS_NO_BUILTINS);


// 需要分析的代码开始
// 具体逻辑不用关心
use Simplelog\Simplelog;

$logFile = "/tmp/simple.log";
$log = new Simplelog($logFile);
$log->debug('skimlinks', array('affid'=>'223'));
// 需要分析的代码结束

// 结束分析
// 分析结果存在数组 $xhprof_data 里
$xhprof_data = xhprof_disable();

// 将分析结果存到 php.ini 中配置的 xhprof.output_dir 目录中
// 下面代码可以封装，这里只做简单演示用
// xhprof/xhprof_lib/ 是下载源码中的 xhprof_lib，即上图标2的文件夹
include_once "xhprof/xhprof_lib/utils/xhprof_lib.php";
include_once "xhprof/xhprof_lib/utils/xhprof_runs.php";

$xhprof_runs = new XHProfRuns_Default();

// "xhprof_foo" 是命名空间，可自定义.
// run_id 生成的唯一id，供下面通过 url 的形式访问查看
$run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_foo");

// xhprof/xhprof_html/ 是下载源码中的 xhprof_html，即上图标1的文件夹
// xhprof_html 须放到 web 可以访问到的目录
// link 为可以查看分析结果的地址
$link = "http://yourdomain.com/xhprof_html/index.php?run=$run_id&source=xhprof_foo";

echo "<a href='{$link}' target='_blank'>{$link}</a>";
```

## 结果分析

上一步中生成的分析结果，可以通过下面地址查看：
```
http://yourdomain.com/xhprof_html/index.php?run=5be28a5486b02&source=xhprof_foo
```

通过上面地址，可以看到类似下面的列表展示，可以从不同的维度来看各方法的性能：执行时长，CPU占用等信息
![xhprof report list](https://raw.githubusercontent.com/ayonliu/exercise/master/images/xhprof_report.png)

### 主要参数说明
```
Calls        调用次数
Calls%   调用百分比
Incl. Wall Time   调用的包括子函数所有花费时间，以微秒算
IWall%                调用的包括子函数所有花费时间的百分比
Excl. Wall Time   函数执行本身花费的时间，不包括子树执行时间,以微秒算
EWall%               函数执行本身花费的时间的百分比不包括子树执行时间
Incl. CPU(microsecs)   方法执行花费的CPU时间，包括子方法的执行时间
ICPU%                          方法执行花费的CPU时间百分比
Excl. CPU(microsec)    方法本身执行花费的CPU时间，不包括子方法的执行时间
ECPU%                         方法本身执行花费的CPU时间百分比
Incl.MemUse(bytes)    方法执行占用的内存，包括子方法执行占用的内存。（单位：字节）
IMemUse%                  方法执行占用的内存百分比。
Excl.MemUse(bytes)    方法本身执行占用的内存，不包括子方法执行占用的内存。（单位：字节）
EMemUse%                  方法本身执行占用的内存百分比。
```

## 图片展示

点击上面列表展示中的“View Full Callgraph”，就可以跳转到下图的的图片展示的界面。
通过下图可以看到各方法的调用路径以及执行所花费的时间，红色块是执行时间最长的，黄色块是其次。这样就能快速找到对性能影响较大的节点，然后到相应的节点里分析到底是代码还是 mysql 的问题就很方便了。

![xhprof report graph](https://raw.githubusercontent.com/ayonliu/exercise/master/images/xhprof_graph.png)

参考：
[XHProf](http://pecl.php.net/package/xhprof)
[层次式性能分析器](http://php.net/manual/zh/book.xhprof.php)
[https://www.kam24.ru/xhprof/docs/index.html#ui_setup](https://www.kam24.ru/xhprof/docs/index.html#ui_setup)