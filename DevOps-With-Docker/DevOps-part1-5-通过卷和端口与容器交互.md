# DevOps-part1-5-通过卷和端口与容器交互

bind mount 是一项内核的特性，可以**将一个目录挂载到另一个目录**，可以理解是 ln 对目录的硬链接，ln 对文件有硬链接和软链接，但是 ln 对目录只有软链接。

我们可以使用卷来使它更容易在容器的短暂存储之外存储下载。
通过bind mount，我们可以从我们自己的机器上挂载一个文件或目录到容器中。
让我们用-v选项启动另一个容器，这需要一个绝对路径。
我们把我们的当前文件夹挂载到我们的容器中，作为/mydir，覆盖我们在Docker文件中放在该文件夹中的所有内容。
```console
 docker run -v "$(pwd):/mydir" youtube-dl https://imgur.com/JY5tHqr
```

$(pwd) 是当前工作目录。

所以卷只是主机和容器之间共享的一个文件夹（或一个文件）。
如果卷中的文件被运行在容器中的程序修改，那么当容器关闭时，这些修改也会被保存下来，因为文件存在于主机上。
这是卷的主要用途，否则在重新启动容器时，所有的文件都无法访问。
卷也可以用来在容器之间共享文件，并运行能够加载改变的文件的程序。

在我们的youtube-dl中，我们想挂载整个目录，因为文件的命名是相当随机的。
如果我们希望创建一个只有一个文件的卷，我们也可以通过指向它来实现。
例如`-v $(pwd)/material.md:/mydir/material.md `这样我们就可以在本地编辑 material.md 并在容器中改变它（反之亦然）。
还要注意，如果文件不存在，`-v`会创建一个目录。

因为网路问题最后用的是：
```console
docker run youtube-dl https://www.bilibili.com/video/BV1q44y1G75W?spm_id_from=333.851.b_7265636f6d6d656e64.5

docker run -v "$(pwd):/mydir" youtube-dl https://www.bilibili.com/video/BV1q44y1G75W?spm_id_from=333.851.b_7265636f6d6d656e64.5
```
输出：
```console
root@VM-16-4-ubuntu:/home/ubuntu/ytb# tree
.
├── 【A-SOUL游戏室】向晚：“真没有剧本”， 嘉然：“有剧本的，你给钱了的”-1q44y1G75W.flv
├── Dockerfile
└── mydir

```


# 允许外部连接进入容器

- 发送消息：程序可以将消息发送到 URL 地址，如下所示：http://127.0.0.1:3000 其中 http 是协议，127.0.0.1 是 IP 地址，3000 是端口。请注意，ip部分也可以是主机名：127.0.0.1也称为localhost，因此您可以改用 http://localhost:3000。

- 接收消息：可以分配程序以侦听任何可用端口。如果程序正在侦听端口 3000 上的流量，并且向该端口发送了一条消息，它将接收该消息（并可能对其进行处理）。

地址127.0.0.1和hostname localhost是特殊的，它们指的是机器或容器本身.

所以如果你在一个容器上并向localhost发送消息，目标是同一个容器。
同样，如果要将请求从容器外部发送到 localhost，则目标是您的计算机。

## 可以将主机端口映射到容器端口。

打开从外部世界到 Docker 容器的连接分两步进行：

-   公开端口：公开容器端口意味着告诉 Docker 容器侦听某个端口。这并没有多大作用，除了它可以帮助人类进行配置。
	要公开端口，请在 Docker 文件中添加该行`EXPOSE <port>`
    
-   发布端口：发布端口意味着 Docker 会将主机端口映射到容器端口。
    若要发布端口，请使用以下命令运行容器：`-p <host-port>:<container-port>`


如果您省略主机端口而仅指定容器端口，docker 将自动选择一个空闲端口作为主机端口：
主机的端口在前面，容器的端口在后面。
```console
$ docker run -p 4567 app-in-port
```

我们还可以仅将连接限制为某些协议，
例如，通过在末尾添加协议来 udp：
`EXPOSE <port>/udp`
和
`-p <host-port>:<container-port>/udp`。