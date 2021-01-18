
# 高权限webshell之IIS应用程序池的权限揭秘

## 前言
攻击者利用漏洞拿到windows服务器的控制权限之后，通常会考虑将改机器作为持久化攻击的跳板，本文要讲述的是如何利用应用程序池来构建一个高权限webshell的方法。


## IIS 应用程序池
IIS的应用程序池用于隔离部署在同一个服务器上的托管用户使用的web应用。每个应用程序池单独运行，一个应用程序池中的错误不会影响到其他应用程序池中的应用程序。


## 高权限应用程序池
要达到某种比较隐蔽的效果，这里都会使用.net服务安装时候使用的默认的应用程序池


### 标识
配置应用程序池以作为内置账户或特定的用户标识运行，内置账户 LocalService、LocalSystem、NetworkService、ApplicationPoolIdentity。在进程模型中的标识中选择LocalSystem, 也可以创建或使用特定的高权账号。
![avatar](/source/intranet/iis/iis1.jpg)


### .Net Level
配置web site的信任级别，信任级别 Full、High、Medium、Low、Minimal，这里配置为Full。(注意: 如果信任级别太低会导致权限配置失败，且webshell的权限被收紧)

在 `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config` 可配置全局的 `trust level`。
![avatar](/source/intranet/iis/iis2.png)


### 内嵌型webshell
配置完成应用程序池，在针对性的对网站中的某个apsx文件嵌入较为隐蔽的后门代码，这样就可以减少被检测发现的可能(一般的webshell检测工具检测不到)，由于是嵌入到正常业务代码中去所以要检测出问题需要对比md5才可以。这里为了简化操作就使用正常的大马来观察配置情况。
![avatar](/source/intranet/iis/iis3.png)


### 清除痕迹
- 情况配置应用程序池过程中产生的日志
- 利用菜刀修改webshell的创建时间


## 总结
本文的利用条件较为特殊，最初的想法是简化持久跳板二次提权的麻烦。欢迎各位老表指出文中错误和交流指教。