# Spring4Shell
一个Spring4Shell 被动式检测的Burp插件

## 检测理论依据

在参数的KEY中插入以下2个POC时，会触发业务行为不一致。
通常来说：
class.module.classLoader.URLs[a0]=  ,抛出500异常
class.module.classLoader.URLs[0]= ，不会抛出500异常，和上面的请求返回行为极有可能是不一致的。

## 实现思路：

给所有Body，URL的参数中以这2个POC，多插入一个KEY，判断两个POC插入后返回的请求响应码 和 返回内容是不是一致。如果不一致，则抛出告警。

