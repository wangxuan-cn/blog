---
title: Dubbo异常集锦
date: 2018-04-28 09:41:35
tags:
    - Dubbo
categories:
    - Dubbo
---
# 没有引用到注册的服务提供方
服务提供方已经注册到zookeeper的注册中心，但是没有正确引用。  
错误提示如下：
```
No provider available in [invoker :interface com.suneee.scn.system.api.provider.UserProvider ->
zookeeper://zookeeper.vr.weilian.cn:12233/com.alibaba.dubbo.registry.RegistryService?application=mcs-rest&check=false&default.check=fal
se&default.reference.filter=default&default.timeout=16000&dubbo=2.8.6&interface=com.suneee.scn.system.api.provider.UserProvider&methods
=findUserInfo,findByRole,getUserInfoAndEnterpriseInfo,findById,getUserListInEnterprise&owner=sunEeemcs&pid=8296&revision=1.0.0-20180424
.015002-6&side=consumer&timestamp=1524879034209,directory: com.alibaba.dubbo.registry.integration.RegistryDirectory@70211df5, invoker
:interface com.suneee.scn.system.api.provider.UserProvider ->
zookeeper://zookeeper.vr.weilian.cn:12233/com.alibaba.dubbo.registry.RegistryService?application=mcs-rest&check=false&default.check=fal
se&default.reference.filter=default&default.timeout=16000&dubbo=2.8.6&interface=com.suneee.scn.system.api.provider.UserProvider&methods
=findUserInfo,findByRole,getUserInfoAndEnterpriseInfo,findById,getUserListInEnterprise&owner=sunEeemcs&pid=8296&revision=1.0.0-20180424
.015002-6&side=consumer&timestamp=1524879034209,directory: com.alibaba.dubbo.registry.integration.RegistryDirectory@38e7ed69]
```

服务提供方在zookeeper的注册中心已经注册了，但是服务提供方加了版本号，服务提供方代码如下：
```
@Service(version="1.0")
public class UserProviderImpl implements UserProvider {
```

服务消费方引用代码如下：
```
@Reference(check = false)
private UserProvider userProvider;
```

错误原因：没有加版本号  
修改代码如下：
```
@Reference(check = false, version = "1.0")
private UserProvider userProvider;
```

# 服务提供方的方法没有注册
服务消费方有的方法可以调用，有的不能调用出现：com.alibaba.dubbo.rpc.RpcException: Failed to invoke the method异常，由于服务提供方新加的方法并没有注册到zookeeper的注册中心导致的。  
错误提示如下：
```
com.alibaba.dubbo.rpc.RpcException: Failed to invoke the method getUserListInEnterprise in the service
com.suneee.scn.system.api.provider.UserProvider. Tried 3 times of the providers [172.16.36.67:21117] (1/1) from the registry
zookeeper.vr.weilian.cn:12233 on the consumer 172.19.24.134 using the dubbo version 2.8.6. Last error is: Failed to invoke remote
method: getUserListInEnterprise, provider:
dubbo://172.16.36.67:21117/com.suneee.scn.system.api.provider.UserProvider?anyhost=true&application=mcs-rest&check=false&default.check=
false&default.reference.filter=default&default.timeout=16000&dubbo=1.0.0-SNAPSHOT&generic=false&interface=com.suneee.scn.system.api.pro
vider.UserProvider&methods=findUserInfo,findByRole,findById,getUserByParam&owner=sunEeemcs&pid=10828&revision=1.0.0-20180424.015002-6&s
erialization=kryo&server=netty4&side=consumer&timestamp=1524825911114&version=1.0, cause: com.alibaba.dubbo.rpc.RpcException: Failed
to invoke remote proxy method getUserListInEnterprise to
registry://zookeeper.vr.weilian.cn:12233/com.alibaba.dubbo.registry.RegistryService?application=system-provider&dubbo=1.0.0-SNAPSHOT&ex
port=dubbo%3A%2F%2F172.16.36.67%3A21117%2Fcom.suneee.scn.system.api.provider.UserProvider%3Fanyhost%3Dtrue%26application%3Dsystem-provi
der%26dubbo%3D1.0.0-SNAPSHOT%26generic%3Dfalse%26interface%3Dcom.suneee.scn.system.api.provider.UserProvider%26methods%3DfindUserInfo%2
CfindByRole%2CfindById%2CgetUserByParam%26owner%3DsunEeeSystem%26pid%3D44275%26revision%3D1.0.0-SNAPSHOT%26serialization%3Dkryo%26serve
r%3Dnetty4%26side%3Dprovider%26timestamp%3D1524825191422%26version%3D1.0&owner=sunEeeSystem&pid=44275&registry=zookeeper&server=netty4&
timestamp=1524825181399, cause: Not found method "getUserListInEnterprise" in class com.suneee.scn.system.api.provider.UserProvider.
com.alibaba.dubbo.rpc.RpcException: Failed to invoke remote proxy method getUserListInEnterprise to
registry://zookeeper.vr.weilian.cn:12233/com.alibaba.dubbo.registry.RegistryService?application=system-provider&dubbo=1.0.0-SNAPSHOT&ex
port=dubb
```

如果拥有服务提供方的代码，包含了新加的代码，本机启动，向zookeeper的注册中心注册新加的方法，然后指定URL的地址。URL的地址可以是localhost、127.0.0.1或者本机IP地址。这种配置仅供开发测试使用，上测试环境或生产环境的时候一定要删除。  
引入服务消费方代码如下：  
```
@Reference(check = false, version = "1.0", url = "dubbo://172.19.24.134:20928/com.suneee.scn.system.api.provider.UserProvider")
private static UserProvider userProvider;
```
