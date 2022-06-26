# XMPP和Tigase

## 序言
今天给大家分享一个关于即时通讯（IM：Instant Messaging）的东西。即时通讯其实在我们身边随处可见，qq、微信、钉钉等都是我们平日里接触最多的IM产品。由于即时通讯模拟了人与人之间通过电子设备达到了间接性的面对面聊天，成功实现了无距离沟通，对社会进步起到了重要作用。那么作为开发者的我们是否思考过其实现原理、内部实现等一些细节呢。

## IM协议
[即时通讯](https://baike.baidu.com/item/IM%E5%8D%8F%E8%AE%AE/15547961?fr=aladdin)：是一个实时通信系统，允许两人或多人使用网络实时的传递文字消息、文件、语音与视频交流。主要分为：

- 即时消息和表示协议IMPP（Instant Messaging And PresenceProtocol）
- 表示和即时消息协议PRIM（Presence and Instant Messaging Protocol）
- SIP即时消息和表示协议SIMPLE（SIP for Instant Messaging and Presence Leveraging Extensions）
- 可扩展消息与表示协议XMPP（Extensible Messaging and Presence Protocol）
其中XMPP是最灵活的，因此也得到了广泛使用。因为XMPP是一种基于XML的协议，它继承了XML环境中灵活的可扩展性。因此基于XMPP的应用具有超强的可扩展性。经过扩展以后的XMPP可以通过发生扩展的信息来处理用户的需求，以及在XMPP的顶端建立如内容发布系统和基于地址的服务等应用程序。而且，XMPP包含了针对服务端的软件协议，使之能与另一个进行对话，使得开发者更容易建立客户端应用程序或给一个配好系统添加功能。



## XMPP协议
### 协议核心内容
XMPP是可扩展消息和状态协议，是一套用户即时消息、在线状态、多方聊天、语音和视频呼叫、协作、轻量级中间件、内容联合的XML数据通用路由的开放式技术。他的前身是Jabber。

XMPP协议的核心是由国际互联网工程小组IETF于2004年发布的[RFC3920](https://xmpp.org/rfcs/rfc3920.html)和[RFC3921](https://xmpp.org/rfcs/rfc3921.html)规范，2011年进行了修订，新的规范为[RFC6120](https://xmpp.org/rfcs/rfc6120.html)、[RFC6121](https://xmpp.org/rfcs/rfc6121.html)、[RFC7622](https://datatracker.ietf.org/doc/rfc7622/)。基于此核心规范XMPP标准基金会继续发布了许多[XMPP扩展协议](https://xmpp.org/extensions/)。

[RFC3920](https://xmpp.org/rfcs/rfc3920.html)规定了XMPP协议框架下应用的网络架构（C/S），包含客户端、服务端、网关；引入XML Stream（XML流）与XML Stanza（XML节）；规定在通信过程使用XML标签；在安全方面，把TLS和SASL引入内核；规范还规定了错误的定义及处理、XML的使用规范、JID（Jabber Identifier）的定义、命名规范、国际化等。

![xmpp structure](C:\Users\11754\Desktop\notes\tigase\xmpp structure.png)

[RFC3921](https://xmpp.org/rfcs/rfc3921.html)规定了基本即时消息、在线状态功能、好友的管理等。所有的消息通过三种基本的XML节来完成：IQ Stanza（IQ节）、Presence Stanza（Presence节）、Message Stanza（Message节）。对阻塞也进行了定义，规定了多种阻塞方式。

[扩展协议](https://xmpp.org/extensions/)的支持丰富了基于XMPP应用的功能：MUC群聊、语音聊天、视频聊天、发布订阅功能、用户位置、文件服务、BOSH支持、WebSocket支持、UDP实现等

### 应用优势
基于XMPP实现的应用一般具有如下优势：
- 开放性：XMPP协议是免费、开放、公开的，易于理解的。可以使用多种语言实现
- 标准性：XMPP协议由IETF攥写发布
- 可靠性：长达二十几年的历史，全球使用XMPP服务的数不胜数
- 安全性：使用TLS和SASL，高度安全
- 可扩展性：基于XML的，可以构建自定义功能
- 灵活性：IM以外的XMPP应用程序包括网络管理、协作工具、文件共享、web服务、轻量级中间件、云计算等

### 不足
基于XMPP的应用都是重服务端的应用，也就是说对服务器性压力会比较大

### 实现
基于XMPP实现的有很多，其中较为知名的Tigase和Openfire
| 应用               | 支持平台                    |
| ------------------ | --------------------------- |
| AstraChat          | Linux/macOS/Solaris/Windows |
| ejabberd           | Linux/macOS/Windows         |
| IoT Broker         | Windows                     |
| Isode M-Link       | Linux/Windows               |
| MongeeseIM         | Linux/macOS                 |
| Openfire           | Linux/macOS/Solaris/Windows |
| Prosody IM         | BSD/Linux/macOS             |
| Tigase XMPP Server | Linux/macOS/Solaris/Windows |



## Tigase

Tigase XMPP Server是用Java编写的高性能、高模块化和非常灵活的XMPP/Jabber服务器，[开源地址](https://github.com/tigase/tigase-server)的。

提供的功能非常多，包括但不仅是


- AMP组件
- Bosh组件
- c2s组件
- s2s组件
- sess-man组件
- cl-comp组件
- monitor组件
- MUC组件
- PubSub发布订阅组件
- Socket5代理组件
- Push组件
- Http API组件
- Jetty Http API组件
- 消息归档组件
- Http文件上传组件

### 原理
​	Tigase主要由三部分组成：组件、插件、连接管理器，从头至尾都贯穿着这三部分。所有的组件和插件都是可插拔的，通过配置文件/etc/init.properties进行设置。
​	Tigase的入口函数是一个普通的类XMPPServer，start()中首先加载两个核心组件Configurator和MessageRouter。
​	Configurato组件负责配置文件等参数的相关操作。
​	MessageRouter负责加载和初始化Server所需的其他组件，启动服务端socket，把组件按名字和对应的类型绑定导相关的变量里，为后面的xmpp packet路由做准备。
​	ConnectionManager负责调度ConnectionOpenThread、SocketThread，这两个类是用NIO实现的，负责监听端口，建立服务端与客户端的连接，接收数据包。

### 消息流转
xmpp消息从A用户到B用户之间是不可知的，通常消息从A发出后由服务器进行处理，由服务器push给对应的B用户。
协议中规定了message节中含有from和to属性，其值是对应的jid，to是接收人的jid，from是发送者的jid。大致的[消息流程参考](https://www.cnblogs.com/eyecool/p/12621298.html)



## 视酷

### 简述

​	视酷是基于Tigase封装的一个服务，主要包含Tigase XMPP Server、文件上传下载服务和后台管理系统。
​	项目中使用视酷的地方主要是小微客户端上，小微客户端允许用户像使用钉钉、微信这样的聊天功能，单聊和群聊。
​	大致流程是：用户中心在小微客户端登录时也会通过websocket连接Tigase XMPP Server服务器，连接成功后即可使用Tigase的聊天功能，退出时同时也断开websocket连接。
​	用户可以登录到Tigase的前提是我们用户中心的用户已经通过视酷后台管理的接口导入到Tigase到中，否则在小微客户端上将显示连接中...。

### 伪代码

``` js
//fake code
const {client, xml} = require('@xmpp/client')
const debug = require('@xmpp/debug')
var xmpp = null;

//登录
function login(username, password) {
	//do login
	xmppGenerator();
}

//退出登录
function loginOut() {
	//下线
	if (!!xmpp) {
		xmpp.stop().catch(console.error);
	}
}

xmppGenerator = () => {
	if (!!xmpp) {
		return;
	}
	xmpp = client({
        service: 'ws://10.10.4.194:5290',
        domain: 'jwtech.com'
        resource: '1000',
        username: '1000',
        password: '1000'
    });

    debug(xmpp, true);

    xmpp.on('error', error => {
        console.log(error);
    });

    xmpp.on('offline'), () => {
        console.log('offline');
    }

    xmpp.on('stanza', stanza => {
        if (stanza.is('message')) {
            handleMessage(stanza);
        }
    });

    xmpp.on('online', async address => {
        await xmpp.send(xml('presence'));
        const message = xml('message', {type: 'chat', to: 'address', xml('body', {}, 'hello world'});
        await xmpp.send(message);
    });

    //登录上线
    xmpp.start().catch(console.eror);
}

//接收消息
handleMessage = stanza => {
	//let canReadData = resolveStanza(stanza)
	//showMessage(canReadData);
}

//发送消息
sendMessage = (receiver, message) => {
	//xmpp.send(xml());
}

```



## 参考
[XMPP官网](https://xmpp.org/)

[(IM)即时通讯协议](https://blog.csdn.net/qq_16519957/article/details/88787239)

[Tigase官网](https://docs.tigase.net/)

