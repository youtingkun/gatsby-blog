---
title: 从URL到页面显示发生了什么
date: 2018-11-22 15:49:37
template: "post"
categories: "计算机网络"
socialImage: "/media/image-4.jpg"
---

# 一、在浏览器地址栏输入URL #
浏览器查看缓存，如果请求资源在缓存中并且新鲜，跳转到转码步骤

1. 如果资源未缓存，发起新请求
2. 如果已缓存，检验是否足够新鲜，足够新鲜直接提供给客户端，否则与服务器进行验证。
3. 检验新鲜通常有两个HTTP头进行控制Expires和Cache-Control：
	- HTTP1.0提供Expires，值为一个绝对时间表示缓存新鲜日期
	- HTTP1.1增加了Cache-Control: max-age=,值为以秒为单位的最大新鲜时间

<!-- more -->
# 二、进行DNS域名解析 #
**浏览器解析URL获取协议，主机，端口，PATH。把这些请求组装一个HTTP（GET）请求报文:**

	//首行是Request-Line包括：请求方法，请求URI，协议版本，CRLF
	//首行之后是若干行请求头，包括general-header，request-header或者entity-header，每个一行以CRLF结束
	//请求头和消息实体之间有一个CRLF分隔
	//根据实际请求需要可能包含一个消息实体
	GET /Protocols/rfc2616/rfc2616-sec5.html HTTP/1.1
	Host: www.w3.org
	Connection: keep-alive
	Cache-Control: max-age=0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
	User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36
	Referer: https://www.google.com.hk/
	Accept-Encoding: gzip,deflate,sdch
	Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
	Cookie: authorstyle=yes
	If-None-Match: "2cc8-3e3073913b100"
	If-Modified-Since: Wed, 01 Sep 2004 13:24:52 GMT

	name=qiu&age=25

**浏览器获取主机ip地址，解析的查找顺序依次为：**

1.浏览器自身的DNS缓存

2.本机DNS缓存    //存放在内存中，关机就消失

3.hosts文件    //存放在系统文件夹中

4.路由器缓存

5.DNS域名服务器

6.根域名服务器。

**DNS服务器的分级查询：**

主机名.次级域名.顶级域名.根域名 = www.google.com.root   //一般root会被省略，因为对于所有的域名都是一样的。
全世界有13台根服务器，域名为a.root-servers.net 到m.root-servers.net。

所谓"分级查询"，就是从根域名开始，依次查询每一级域名的NS记录，直到查到最终的IP地址，过程大致如下：

	1.从"根域名服务器"查到"顶级域名服务器"的NS记录和A记录（IP地址）
    2.从"顶级域名服务器"查到"次级域名服务器"的NS记录和A记录（IP地址）
    3.从"次级域名服务器"查出"主机名"的IP地址
	//NS记录（Name Server的缩写），即哪些服务器负责管理google.com的DNS记录。


根域名服务器分布：

![](https://i.imgur.com/IW1A84s.png)

**查找方式分为两种：**

迭代和递归。简单来说，迭代查询就是需要本地服务器自己去找，而递归查询是其它其它服务器直接把查询好的结果告诉它。
![](https://i.imgur.com/RVQ3Uxv.png)

# 三、根据目标IP地址，端口建立TCP链接 #
**进行TCP协议三次握手：**

1.主机向服务器发送一个建立连接的请求（您好，我想认识您）；
2.服务器接到请求后发送同意连接的信号（好的，很高兴认识您）；
3.主机接到同意连接的信号后，再次向服务器发送了确认信号（我也很高兴认识您），自此，主机与服务器两者建立了连接。
//结束断开的时候为四次挥手

    客户端发送一个TCP的SYN=1，Seq=X的包到服务器端口
    服务器发回SYN=1， ACK=X+1， Seq=Y的响应包
    客户端发送ACK=Y+1， Seq=Z

# 四、服务器将响应报文，返回HTTP报文给浏览器 #
**服务器接受请求并解析，将请求转发到服务程序，如虚拟主机使用HTTP Host头部判断请求的服务程序**

**服务器检查HTTP请求头是否包含缓存验证信息如果验证缓存新鲜，返回304等对应状态码**

**处理程序读取完整请求并准备HTTP响应，可能需要查询数据库等操作**

**服务器将响应报文通过TCP连接发送回浏览器**

返回的HTTP报文如下：

    //首行是Request-Line包括：请求方法，请求URI，协议版本，CRLF
    //首行之后是若干行请求头，包括general-header，request-header或者entity-header，每个一行以CRLF结束
    //请求头和消息实体之间有一个CRLF分隔
    //根据实际请求需要可能包含一个消息实体 一个请求报文例子如下：

	GET /Protocols/rfc2616/rfc2616-sec5.html HTTP/1.1
	Host: www.w3.org
	Connection: keep-alive
	Cache-Control: max-age=0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
	User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36
	Referer: https://www.google.com.hk/
	Accept-Encoding: gzip,deflate,sdch
	Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
	Cookie: authorstyle=yes
	If-None-Match: "2cc8-3e3073913b100"
	If-Modified-Since: Wed, 01 Sep 2004 13:24:52 GMT

	<html>
	      <head></head>
	      <body>
	            <!--body goes here-->
	      </body>
	</html>

# 五、浏览器解析渲染页面 #
**浏览器接收HTTP响应，然后根据情况选择关闭TCP连接或者保留重用，关闭TCP连接的四次握手如下： **

    主动方发送Fin=1， Ack=Z， Seq= X报文
    被动方发送ACK=X+1， Seq=Z报文
    被动方发送Fin=1， ACK=X， Seq=Y报文
    主动方发送ACK=Y， Seq=X报文


浏览器检查响应状态吗：是否为1XX，3XX， 4XX， 5XX，这些情况处理与2XX不同

如果资源可缓存，进行缓存

对响应进行解码（例如gzip压缩）

根据资源类型决定如何处理（假设资源为HTML文档）

**解析HTML文档，构件DOM树，下载资源，构造CSSOM树，执行js脚本：**

构建DOM树：

    Tokenizing：根据HTML规范将字符流解析为标记
    Lexing：词法分析将标记转换为对象并定义属性和规则
    DOM construction：根据HTML标记关系将对象组成DOM树

解析过程中遇到图片、样式表、js文件，启动下载

构建CSSOM树：

    Tokenizing：字符流转换为标记流
    Node：根据标记创建节点
    CSSOM：节点创建CSSOM树

根据DOM树和CSSOM树构建渲染树:

    从DOM树的根节点遍历所有可见节点，不可见节点包括：1）script,meta这样本身不可见的标签。2)被css隐藏的节点，如display: none
    对每一个可见节点，找到恰当的CSSOM规则并应用
    发布可视节点的内容和计算样式

js解析：

    浏览器创建Document对象并解析HTML，将解析到的元素和文本节点添加到文档中，此时document.readystate为loading
    HTML解析器遇到没有async和defer的script时，将他们添加到文档中，然后执行行内或外部脚本。这些脚本会同步执行，并且在脚本下载和执行时解析器会暂停。这样就可以用document.write()把文本插入到输入流中。同步脚本经常简单定义函数和注册事件处理程序，他们可以遍历和操作script和他们之前的文档内容
    当解析器遇到设置了async属性的script时，开始下载脚本并继续解析文档。脚本会在它下载完成后尽快执行，但是解析器不会停下来等它下载。异步脚本禁止使用document.write()，它们可以访问自己script和之前的文档元素
    当文档完成解析，document.readState变成interactive
    所有defer脚本会按照在文档出现的顺序执行，延迟脚本能访问完整文档树，禁止使用document.write()
    浏览器在Document对象上触发DOMContentLoaded事件
    此时文档完全解析完成，浏览器可能还在等待如图片等内容加载，等这些内容完成载入并且所有异步脚本完成载入和执行，document.readState变为complete,window触发load事件

显示页面（HTML解析过程中会逐步显示页面）
