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
列出所有容器`docker container ls -a`
如果没有`-a`标志，它只会打印正在运行的容器。

没有运行的容器就是垃圾容器，如果您有数百个已停止的容器，并且希望将它们全部删除，则应使用`docker container prune`。

# Docker CLI 基础知识
使用命令行与“Docker 引擎”进行交互，该引擎由 3 部分组成：CLI、REST API 和 Docker 守护程序。

在删除镜像之前，应先删除 引用容器。

列出所有容器`docker container ls -a`
容器具有_容器 ID_ 和_名称_。这些名称当前是自动生成的。当我们有很多不同的容器时，我们可以使用grep（或其他类似的实用程序）来过滤列表：
```console
docker container ls -a | grep hello-world
```
该命令也适用于 ID 的前几个字符。

例如
`docker container rm 3d`
可以多个筛选。
`docker container rm id1 id2 id3`

使用 ID 的简写不会删除多个容器，因此，如果有两个以 3d 开头的 ID，则会打印警告，并且不会删除任何一个 ID。

如果您有数百个已停止的容器，并且希望将它们全部删除，则应使用`docker container prune`。

prune还可用于删除带有`docker image prune`的“悬垂”image。
悬空image是没有名称且未使用的image。它们可以手动创建，并在构建期间自动生成。删除它们只会节省一些空间。

最后，您可以使用`docker system prune`来清除几乎所有内容。
我们还不熟悉`docker system prune`不会删除的异常。

删除所有 _hello-world_ 容器后，运行`docker image rm hello-world`以删除image。
您可以使用`docker image ls`来确认未列出该图像。

您还可以使用`image pull`命令下载映像，而无需运行它们：
例如:`docker image pull hello-world`

让我们尝试启动一个新容器：

```console
$ docker container run nginx
```

注意在拉动和启动容器后，命令行似乎冻结了。这是因为Nginx现在正在当前终端中运行，阻止了输入。
您可以从另一个终端`docker container ls`观察到这一点。
让我们通过按`control + c`退出，然后使用标志`-d`重试。

```console
$ docker container run -d nginx
  c7749cf989f61353c1d433466d9ed6c45458291106e8131391af972c287fb0e5
```

标志`-d`启动独立的容器，在后台运行。
容器可以看到

```console
$ docker container ls
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
  c7749cf989f6        nginx               "nginx -g 'daemon of…"   35 seconds ago      Up 34 seconds       80/tcp              blissful_wright
```

现在，如果我们尝试删除它，它将失败：

```console
$ docker container rm blissful_wright
  Error response from daemon: You cannot remove a running container c7749cf989f61353c1d433466d9ed6c45458291106e8131391af972c287fb0e5. Stop the container before attempting removal or force remove
```

我们应该首先`docker container stop blissful_wright`停止使用容器，然后使用`rm`。

强制也是一种可能性，在这种情况下我们可以安全地使用。同样，对于它们两个而不是名称，我们可以使用ID或其中的一部分，例如c77。`docker container rm --force blissful_wright`

随着时间的推移，Docker 守护程序会被旧映像和容器堵塞，这是很常见的。



![[最常用的命令.png]]

命令

解释

速记

`docker image ls`

列出所有图像

`docker images`

`docker image rm <image>`

删除图像

`docker rmi`

`docker image pull <image>`

从 Docker 注册表中拉取映像

`docker pull`

`docker container ls -a`

列出所有容器

`docker ps -a`

`docker container run <image>`

从映像运行容器

`docker run`

`docker container rm <container>`

删除容器

`docker rm`

`docker container stop <container>`

停止容器

`docker stop`

`docker container exec <container>`

在容器内执行命令

`docker exec`