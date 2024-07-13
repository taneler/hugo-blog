+++
title = 'RustDesk 自托管服务部署'
date = 2024-07-14T02:57:21+08:00
draft = false
slug = '4edc8217-a595-43b9-af04-b860cc82a7c1'
+++

## 说明

部署 RustDesk 自托管服务，首先需要有一台拥有公网 IP 的服务器，并部署了 Docker 运行环境。

## 网络放通

| 服务 | 端口  | 协议 | 说明                                                   |
| ---- | ----- | ---- | ------------------------------------------------------ |
| hbbs | 21114 | TCP  | Web 控制台，只有在专业版中可用，如果不是专业版请关闭。 |
| hbbs | 21115 | TCP  | NAT 类型测试。                                         |
| hbbs | 21116 | UDP  | ID 注册服务器及心跳检测服务。                          |
| hbbs | 21116 | TCP  | 用于 TCP 打孔及连接服务。                              |
| hbbs | 21118 | TCP  | 用于支持 Web 客户端，如不需 Web 客户端请关闭。         |
| hbbr | 21117 | TCP  | 中继服务。                                             |
| hbbr | 21119 | TCP  | 用于支持 Web 客户端，如不需 Web 客户端请关闭。         |

RustDesk 服务端使用的端口请查看上表，如果部署社区版，则可以忽略 21114、21118、21119 端口的放通，所以仅需到安全组开放以下端口的访问即可。

TCP(21115, 21116, 21117)

UDP(21116)

## 部署服务端

创建一个目录用于保存 Docker 持久化文件。

```bash
mkdir -p /opt/rustDesk && cd /opt/rustDesk
```

分别启动 `hbbs` 服务和 `hbbr` 服务。

```bash
docker run --name hbbs \\
	-p 21115:21115 \\
	-p 21116:21116 \\
	-p 21116:21116/udp \\
	-v `pwd`:/root \\
	-td \\
	--net=host \\
	rustdesk/rustdesk-server:latest \\
	hbbs -r <hbbr地址>:21117
	
docker run --name hbbr \\
	-p 21117:21117 \\
	-v `pwd`:/root \\
	-td \\
	--net=host \\
	rustdesk/rustdesk-server \\
	hbbr
```

在容器启动完成后，执行以下命令。

```bash
cat id_ed25519.pub
```

会输出一串字符串，那就是我们的 `Key`。

![image-20240714025818878](https://s3.kubedevops.cn/s3/image-20240714025818878.png)

## 客户端配置

打开 RustDesk 客户端，进入到 ID/中继服务器，填写上自己的服务器信息以及前面获取的 `Key` 即可。

![image-20240714030010626](https://s3.kubedevops.cn/s3/image-20240714030010626.png)
