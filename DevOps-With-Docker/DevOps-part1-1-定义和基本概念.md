# Dev和Ops
Dev是指软件开发和Ops到运营。DevOps的简单定义是，这意味着软件的发布，配置和监控掌握在开发软件的人手中。

# Docker
1.  Docker 是一组在容器中交付软件的工具。
2.  容器是软件包。

容器是隔离的，因此它们不会相互干扰，也不会干扰在容器外运行的软件。如果您需要与它们交互或启用它们之间的交互，Docker 提供了执行此操作的工具。

避免”在我的计算机上工作“的问题。

隔离的环境

只需一个命令，您就可以在机器中运行一个独立的应用程序，如postgres或mongo。

启动和停止 Docker 容器的开销很小。

通过高级工具，我们可以立即启动多个容器，并对这些容器之间的流量进行负载平衡。

在容器上运行软件几乎与在容器外部“本机”运行软件一样有效，至少与虚拟机相比是这样。

因此，容器可以直接访问您自己的操作系统内核和资源。使用容器的资源使用开销已降至最低，因为应用程序的行为就像没有额外的层一样。

# 第一行docker命令

`docker container run hello-world`

```console
docker container run hello-world
  Unable to find image 'hello-world:latest' locally
  latest: Pulling from library/hello-world
  b8dfde127a29: Pull complete
  Digest: sha256:308866a43596e83578c7dfa15e27a73011bdd402185a84c5cd7f32a88b501a24
  Status: Downloaded newer image for hello-world:latest

  Hello from Docker!
  This message shows that your installation appears to be working correctly.

  To generate this message, Docker took the following steps:
   1. The Docker client contacted the Docker daemon.
   2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
      (amd64)
   3. The Docker daemon created a new container from that image which runs the
      executable that produces the output you are currently reading.
   4. The Docker daemon streamed that output to the Docker client, which sent it
      to your terminal.

  To try something more ambitious, you can run an Ubuntu container with:
   $ docker run -it ubuntu bash

  Share images, automate workflows, and more with a free Docker ID:
   https://hub.docker.com/

  For more examples and ideas, visit:
   https://docs.docker.com/get-started/
```

# inage & container
镜像与容器

一个Docker镜像是一个文件，镜像永远不会改变，无法编辑现有镜像文件。创建新镜像是通过从基础镜像开始并向其添加**新图层（layers）** 来实现的。

列出所有镜像`docker image ls`

```console
$ docker image ls
  REPOSITORY      TAG      IMAGE ID       CREATED         SIZE
  hello-world     latest   d1165f221234   9 days ago      13.3kB
```

容器是从镜像创建的，因此当我们运行 hello-world 两次时，我们下载了一个镜像，并从单个镜像创建了其中两个镜像。

烹饪比喻：

-   镜像是预先煮熟的，冷冻的点心。
-   容器是美味的食物。

使用烹饪比喻，困难的任务是创造冷冻的零食，而加热相对容易。

创建镜像需要一些工作和知识，但是当您知道如何创建镜像时，您可以在自己的项目中利用容器的强大功能。

#  **Dockerfile** 
如果图像用于创建容器，那么图像来自哪里？

此映像文件是从名为 **Dockerfile** 的指令文件构建的，该文件在运行`docker image build`时进行分析。

列出您的所有图像`docker image ls`

Dockerfile是一个名为Dockerfile的文件，看起来像这样

**Dockerfile**

```dockerfile
FROM <image>:<tag>

RUN <install some dependencies>

CMD <command that is executed on `docker container run`>
```

并且是用于构建镜像的指令集。稍后，当我们构建自己的镜像时，我们将研究 Dockerfile。

如果我们回到烹饪的比喻，Dockerfile就是配方。

# 容器
是主机中的**隔离**环境，能够通过定义的方法（TCP/UDP）相互交互，并与主机本身进行交互。

列出所有容器`docker container ls`

如果没有`-a`标志，它只会打印正在运行的容器。

# Docker CLI 基础知识
使用命令行与“Docker 引擎”进行交互，该引擎由 3 部分组成：CLI、REST API 和 Docker 守护程序。

