# Kerberos认证
## Kerberos介绍
在古希腊神话中Kerberos['kɜːbərəs]指的是：有着一只三头犬守护在地狱之门外，禁止任何人类闯入地狱之中。

而现实中的Kerberos是一种网络身份验证协议，旨在通过密钥加密技术为客户端/服务器应用程序提供身份验证，主要用在域环境下的身份验证。

![](https://upload-images.jianshu.io/upload_images/18786052-dd6b64ef35191574.png?imageMogr2/auto-orient/strip)

## 组成部分
通过上图可以看到整个认证流程有三个重要的角色，分别为Client、Server和KDC(Key Distribution Center)密钥分发中心。

KDC 包含两部分：
- Authentication Server (AS)
	- Issues “Ticket-Granting Tickets” (TGT)
- Ticket Granting Server (TGS)
	- Issues service tickets

DC中有一个特殊用户叫做：krbtgt，它是一个无法登录的账户，是在创建域时系统自动创建的，在整个kerberos认证中会多次用到它的Hash值去做验证。

AD会维护一个Account Database(账户数据库)。它存储了域中所有用户的密码Hash和白名单。只有账户密码都在白名单中的Client才能申请到TGT。

## 交互流程
1. client向KDS发用户名和服务端名称
2. KDC回应TGT，使用用户密码加密
3. client输入密码，解密TGT
4. 到手的TGT，用来和KDC通信（我已经有TGT了，属于被你信任的列表中，给我我想要的东西吧），进一步获取service tickets
5. KDC返回用户service tickets
6. 用户拿service tickets和Server通信，请求授权
7. server端授权完成，建立session

## 命令
[官方命令](http://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/index.html)

这里列举两个常用的
- kinit # 与KDC通信，请求授权
- klist # 查看已授权列表

➜  ~ kinit somebody

➜  ~ klist

	Credentials cache: API: ******

    Principal: somebody@***.COM

  	Issued                Expires               Principal
	Apr 17 17:17:33 2022  Apr 18 17:17:31 2022  krbtgt/***.COM@***.COM

## 如何搭建一个DC
[官方教程](http://web.mit.edu/kerberos/krb5-1.12/doc/admin/index.html)

[中文教程](https://www.jianshu.com/p/7e839226a200?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)

/etc/krb5.conf 文件示例

    [libdefaults]
        default_realm = ***.COM
        dns_lookup_realm = false
        dns_lookup_kdc = false
    
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true
    
        ticket_lifetime = 24h // ticket有效期为1天，1天后需要再次续期
        renew_time = 7d // ticket最长可延续7天，7天后renew
    
        rdns = false
        ignore_acceptor_hostname = true
    
    [realms]
        ***.COM = {
            kdc = ***.org
            kdc = ****.org
            master_kdc = ***.org
            admin_server = ***.org
            default_domain = ***.org
        }
    
    [domain_realm]
        .byted.org = ***.COM
        byted.org = ***.COM
    
    [login]
        krb4_convert = true
        krb4_get_tickets = false

## 与ssh的关系
Secure Shell (SSH) 是一个允许两台电脑之间通过安全的连接进行数据交换的网络协议。通过加密保证了数据的保密性和完整性。OpenSSH (OpenBSD Secure Shell) 是一套使用ssh协议，通过计算机网络，提供加密通讯会话的计算机程序。

OpenSSH 在协议版本 1 和 2 中均支持 Kerberos 身份验证。在版本 1 中，会通过特殊协议消息传输 Kerberos 票据。版本 2 不再直接使用 Kerberos，而是依赖于 GSSAPI，即通用安全服务 API.这是一种不特定于 Kerberos 的编程接口 － 其设计目的是隐藏基础身份验证系统的特性，无论它是 Kerberos、公共密钥认证系统（如 SPKM）还是其他系统。但是，包含的 GSSAPI 库仅支持 Kerberos。

要将 Kerberos 认证与协议版本 2 一起使用，需要在客户端也启用它。此操作可在整个系统范围的配置文件 /etc/ssh/ssh_config 中执行，也可通过编辑 ~/.ssh/config 在每个用户级别上执行。在这两种情况下，均应添加选项 GSSAPIAuthentication yes。

Github/GitLab中为什么会用到SSH?

使用SSH协议，可以连接和验证远程服务器和服务；可以连接到GitHub，而无需在每次访问时提供用户名或密码。

配置示例

    Host *
        GSSAPIAuthentication yes
        GSSAPIDelegateCredentials no
    Host 10.*
        StrictHostKeyChecking no
        GSSAPIAuthentication yes
        GSSAPIDelegateCredentials no
    Host git.***.org
        Hostname git.***.org
        Port ***
        User ***
    Host *.***.org
        GSSAPIAuthentication yes


## Reference
[[简书]Kerberos认证](https://www.jianshu.com/p/23a4e8978a30)

[官方教程](http://web.mit.edu/kerberos/krb5-1.12/doc/admin/index.html)

[中文教程](https://www.jianshu.com/p/7e839226a200?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)

[ssh & kerberos](https://documentation.suse.com/zh-cn/sles/15-SP2/html/SLES-all/cha-security-kerberos.html#sec-security-kerberos-admin-sshd)

[ssh & git](https://www.jianshu.com/p/1246cfdbe460)

