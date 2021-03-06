# Spring4ShellScan
一个Spring4Shell【CVE-2022-22965】 被动式检测的Burp插件。

为什么需要造这个轮子？？因为这个漏洞黑盒较难发现，没有具体的业务路径，有了路径没有其他的参数都有可能难以触发到。
同时Burp也是我们常用的工具，抓着包做安全测试时顺带覆盖这种漏洞。

安利yakit的MITM也支持这个漏洞检测：https://github.com/yaklang/yakit

## 检测理论依据

在参数的KEY中插入以下2个POC时，会触发业务行为不一致。
通常来说：
class.module.classLoader.URLs[a0]=  ,抛出500异常
class.module.classLoader.URLs[0]= ，不会抛出500异常，和上面的请求返回行为极有可能是不一致的。

## 实现思路：

给所有Body，URL的参数中以这2个POC，多插入一个KEY，判断两个POC插入后返回的请求响应码 和 返回内容是不是一致。如果不一致，则抛出告警。【todo: 还可以在返回500时，对内容进行关键字匹配】

## 靶机测试
感谢FofaX官方交流群的@官方提醒 的靶机

自己搭建源码：`https://github.com/spring-petclinic/spring-framework-petclinic`

![avatar](20220408004036.png)

## 关于误报漏报

目前测试是发现有误报的，因为判断条件是这样的：
- 条件1 满足两次响应码 和响应内容不一致就报错
- 条件2 两次中某次返回内容出现关键字：org.springframework.validation.DataBinder
```
// 判断是否存在漏洞

                    if (status_code != req1_statuscode && !responseBody.equals(req1_body)){
                        hasIssue = true;
                    }

                    if(req1_body.contains(FoundErrorKey) || responseBody.contains(FoundErrorKey)){
                        hasIssue = true;
                    }
```

漏报情况：
    触发了被动式测试，基于这个判断条件，在Spring + JDK 9.0以上 + Tomcat 应该是没有漏报

## 关于确认发出去的请求

在插件Logger++里面，通过Comment找req1 和req2 如下: 
![avatar](20220408114012.png)
