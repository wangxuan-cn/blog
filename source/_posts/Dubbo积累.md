---
title: Dubbo积累
date: 2018-05-25 13:54:11
tags:
    - Dubbo
categories:
    - Dubbo
---
# zookeeper注册中心
zookeeper中的数据结构查看方式：
1.通过zkCli.sh查看，前提是能登录服务器，或者从任意一台具有zkCli.sh访问另一台。
2.通过Java客户端代码访问。
```
public class ZooKeeperClientTest {
    public static String CONNSTR = "172.16.36.71:12233";
//    public static String PATH = "/";
//    public static String PATH = "/dubbo";
//    public static String PATH = "/dubbo/com.suneee.scn.system.api.provider.UserProvider";
    public static String PATH = "/dubbo/com.suneee.scn.system.api.provider.UserProvider/providers";

    public static void main(String[] args) {
        try {
            ZooKeeper zooKeeper = new ZooKeeper(CONNSTR, 5000, event -> System.out.println("已经触发了" + event.getType() + "事件！"), true);
            ZooKeeper.States state = zooKeeper.getState();
            System.out.println("state=" + state + " isConnected=" + state.isConnected());
            List<String> stringList = zooKeeper.getChildren(PATH, false);
            System.out.println("stringList=" + stringList);
            byte[] data = zooKeeper.getData(PATH, false, new Stat());
            System.out.println(data);
            if (data != null) {
                System.out.println(new String(data));
            }
        } catch (IOException | InterruptedException | KeeperException e) {
            e.printStackTrace();
        }
    }
}
```

dubbo在zookeeper注册中心的数据存储结构：
**/dubbo/[接口地址1, 接口地址2]/[consumers, configurators, routers, providers]**
例：
```
/dubbo
/dubbo/com.suneee.scn.system.api.provider.UserProvider
/dubbo/com.suneee.scn.system.api.provider.UserProvider/consumers
/dubbo/com.suneee.scn.system.api.provider.UserProvider/providers
```
其中/dubbo/com.suneee.scn.system.api.provider.UserProvider/providers下级即是提供者的注册信息，如下：
```
dubbo%3A%2F%2F172.16.36.67%3A21117%2Fcom.suneee.scn.system.api.provider.UserProvider%3Fanyhost%3Dtrue%26application%3Dsystem-provider%26dubbo%3D1.0.0-SNAPSHOT%26generic%3Dfalse%26interface%3Dcom.suneee.scn.system.api.provider.UserProvider%26methods%3DfindUserInfo%2CfindByRole%2CfindById%2CgetUserByParam%26owner%3DsunEeeSystem%26pid%3D44275%26revision%3D1.0.0-SNAPSHOT%26serialization%3Dkryo%26server%3Dnetty4%26side%3Dprovider%26timestamp%3D1524825191422%26version%3D1.0
```
将UTF-8转中文后：
```
dubbo://172.16.36.67:21117/com.suneee.scn.system.api.provider.UserProvider?anyhost=true&application=system-provider&dubbo=1.0.0-SNAPSHOT&generic=false&interface=com.suneee.scn.system.api.provider.UserProvider&methods=findUserInfo,findByRole,findById,getUserByParam&owner=sunEeeSystem&pid=44275&revision=1.0.0-SNAPSHOT&serialization=kryo&server=netty4&side=provider&timestamp=1524825191422&version=1.0
```
分析可知，这些信息包含提供者的IP地址、端口号、注册方法、系列化的实现等等
