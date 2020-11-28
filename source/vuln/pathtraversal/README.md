
# 目录穿越


## 简介

目录穿越漏洞在国内外有许多不同的叫法，也可以叫做信息泄露漏洞、非授权文件包含漏洞等。它可能让攻击者访问受限制的目录，来访问存储在文件系统上的任意文件和目录，特别是应用程序源代码、配置文件、重要的系统文件等。

目录遍历漏洞原理就是程序在实现上没有充分过滤用户输入的../之类的目录跳转符，导致恶意用户可以通过提交目录跳转来遍历服务器上的任意文件。

## 攻击载荷


### URL参数

- ``../``
- ``..\``
- ``..;/``

### Nginx Off by Slash

- ``https://vuln.site.com/files../``

### UNC Bypass

- ``\\localhost\c$\windows\win.ini``

## 过滤绕过

- 单次替换
    - ``...//``
- URL编码
- 16位Unicode编码
    - ``\u002e``
- 超长UTF-8编码
    - ``\%e0%40%ae``

## 防御

- 在进行文件操作相关的API前，应该对用户输入做过滤。较强的规则下可以使用白名单，仅允许纯字母或数字字符等。

- 规范化或限制用户访问的资源, 较强规则下可认证用户访问资源的权限。

## 参考链接

- `Directory traversal by portswigger <https://portswigger.net/web-security/file-path-traversal>`_
- `Path Traversal by OWASP <https://www.owasp.org/index.php/Path_Traversal>`_
- `path normalization <https://blogs.msdn.microsoft.com/jeremykuhne/2016/04/21/path-normalization/>`_
- `Breaking Parser Logic: Take Your Path Normalization Off and Pop 0days Out defcon <https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf>`_
