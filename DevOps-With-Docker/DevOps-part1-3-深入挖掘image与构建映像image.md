# DevOps-part1-3-深入挖掘image

使用`docker search`在 Docker Hub 中搜索映像。
尝试运行 `docker search hello-world`
：
搜索找到大量结果，并打印每个图像的名称，简短描述，星数以及“官方”和“自动”状态。
```console
$ docker search hello-world

  NAME                         DESCRIPTION    STARS   OFFICIAL   AUTOMATED
  hello-world                  Hello World!…  699     [OK]
  kitematic/hello-world-nginx  A light-weig…  112
  tutum/hello-world            Image to tes…  56                 [OK]
```

第一个结果是 ，官方image。官方图像由 Docker Inc. 策划和审核，通常由作者积极维护。它们是从 docker 库中的存储库构建的。

浏览CLI的搜索结果时，您可以从“OFFICIAL”列中的“[OK]”以及图像名称没有前缀（也称为组织/用户）的事实中识别官方图像。浏览 Docker Hub 时，该页面将显示“Docker 官方镜像”作为存储库，而不是用户或组织。

第三个结果 ， 标记为“自动”。这意味着映像是从源存储库自动构建的。其 Docker Hub 页面显示其之前的“生成”，以及指向映像的“源存储库”（在本例中为 GitHub）的链接，Docker Hub 从该源生成映像。

第二个结果， 既不是官方图像，也不是自动图像。我们无法知道映像是从什么构建的，因为它的 Docker 中心页面没有指向任何存储库的链接。其Docker Hub页面显示的唯一一件事是该图像已有6年的历史。即使映像的“概述”部分具有指向存储库的链接，我们也不能保证已发布的映像是从该源构建的。

还有其他 Docker 注册表与 Docker Hub 竞争，例如 quay（红帽的库）。
从其他源拉取image也是`docker pull`。
例如：`docker pull quay.io/nordstrom/hello-world `
要写上正确的版本号。

# 详细查看image
ubuntu镜像，它是最常见的Docker镜像之一，可以用作你自己镜像的基础。

让我们拉动 Ubuntu 并查看第一行：

```console
$ docker pull ubuntu
  Using default tag: latest
  latest: Pulling from library/ubuntu
```

由于我们没有指定标记，Docker 默认为`latest`，这通常是构建并推送到注册表的最新映像。
可以标记图像以保存同一图像的不同版本。您可以通过在图像名称后添加来定义图像的标记。`:<tag>`
例如：
```console
docker pull ubuntu:18.04
```

image由并行下载的不同图层组成，以加快下载速度。由image组成的图像还有其他方面。

标记也是“重命名”image的一种方法。
先拉取`docker pull ubuntu:18.04`
再标记`docker tag ubuntu:18.04 fav_distro:bionic`
最后查看效果：`docker image ls`

```console
REPOSITORY            TAG         IMAGE ID       CREATED        SIZE
nginx                 latest      0e901e68141f   4 days ago     142MB
postgres              12-alpine   e2b44ba9a01f   6 days ago     184MB
ubuntu                latest      d2e4e1f51132   4 weeks ago    77.8MB
fav_distro            bionic      c6ad7e71ba7d   4 weeks ago    63.2MB
ubuntu                18.04       c6ad7e71ba7d   4 weeks ago    63.2MB
```
fav_distro取代了ubuntu:18.04
TAG不再显示latest或者具体的数字，或者某些组织的特供版（例如12-alpine），而是bionic。

ubuntu:18.04也可以用来表示ubuntu:18.04，
fav_distro:bionic也可以用来表示ubuntu:18.04。

只是多了一个方法，而非取代。

# 构建映像image

Dockerfile 只是一个包含映像构建说明的文件。
可以使用不同的说明定义映像中应包含的内容。

先用一个最简单的应用程序并将其进行容器化。
touch（创建文件） 、mkdir（创建目录）、rm （删除文件、目录）
`touch hello.sh`
用vim写进去。
**hello.sh**
 #!/**bin/sh**是vim 指此脚本使用/**bin/sh**来解释执行，#!是特殊的表示符，其后面根的是此解释此脚本的shell的路径。
```sh
#!/bin/sh 

echo "Hello, docker!"
```

首先，我们将测试它是否有效。创建文件，添加执行权限并运行它：
Linux chmod（英文全拼：change mode）命令是控制用户对文件的权限的命令
```console
$ chmod +x hello.sh

$ ./hello.sh
  Hello, docker!
```

**现在从中创建一个图像。**
我们必须创建声明 包含了**所有必需依赖项的 Dockerfile**。
这里是用一个小的linux发行版来作为依赖项。
至少它依赖于可以运行shell脚本的东西。所以我会选择alpine，它是一个小型Linux发行版，经常用于创建小image。
默认情况下，Ubuntu 映像包含更多工具，用于在出现问题时调试错误。
要准确选择要使用的给定图像的版本。这样，我们就不会意外地通过重大更改进行更新，并且当旧映像中存在已知的安全漏洞时，我们知道哪些映像需要更新。

**现在创建一个文件并将其命名为“Dockerfile”，让我们将以下说明放入其中：**

**Dockerfile**

```dockerfile
# 从alpine:3.13的image开始，它比较小，但没有花哨的工具
FROM alpine:3.13

# 使用/usr/src/app作为我们的工作目录。下面的指令将在这个位置执行。
# WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在， WORKDIR 会帮你建立目录。
# 这些文件目录指的是alpine:3.13的文件路径。
WORKDIR /usr/src/app

# 将hello.sh文件复制到这个位置/usr/src/app/ 创建/usr/src/app/hello.sh
COPY hello.sh .

# 另外，如果我们先前在创建原文件的时候跳过了chmod没去赋权，我们可以在构建过程中添加执行权限。
# RUN chmod +x hello.sh

# 当运行docker run时，相当于在alpine的命令行输入./hello.sh
CMD ./hello.sh
```

如果您现在收到“/bin/sh： ./hello.sh： 权限被拒绝”，那是因为之前跳过了。您只需在 COPY 和 CMD 指令之间取消注释（#） RUN 的指令即可`chmod +x hello.sh`


默认情况下，`docker build`将查找名为 Dockerfile 的文件。现在，我们可以使用说明在何处构建（`docker build`）并为其命名（`.`）：`-t <name>`

```console
docker build . -t hello-docker
```
**别在根目录下面这么搞，另起一个文件夹，还要把`CPOY`里的文件放到这个文件夹里（如果不标路径的话）。**
报错：
```console
Sending build context to Docker daemon   2.56kB
Step 1/4 : FROM alpine:3.13
3.13: Pulling from library/alpine
6097bfa160c1: Pull complete 
Digest: sha256:ccf92aa53bc6c3b25be2ad0cce80baec1778f007f7e076b0ffbd1b225d0b3a9b
Status: Downloaded newer image for alpine:3.13
 ---> 20e452a0a81a
Step 2/4 : WORKDIR /usr/src/app
 ---> Running in 999bd2264f28
Removing intermediate container 999bd2264f28
 ---> c4f81905508b
Step 3/4 : COPY hello.sh .
COPY failed: file not found in build context or excluded by .dockerignore: stat hello.sh: file does not exist
```

在构建过程中，我们看到哈希和中间容器有多个Step(步骤)。此处的Step表示图层，以便每个Step都是iamge的新图层。**每个Step都是上面Dockerfile的命令。**
```console
 Sending build context to Docker daemon  54.78kB
  Step 1/4 : FROM alpine:3.13
   ---> d6e46aa2470d
  Step 2/4 : WORKDIR /usr/src/app
   ---> Running in bd0b4e349cb4
  Removing intermediate container bd0b4e349cb4
   ---> b382ca27c182
  Step 3/4 : COPY hello.sh .
   ---> 7fbc1b6e45ab
  Step 4/4 : CMD ./hello.sh
   ---> Running in 24f28f026b3f
  Removing intermediate container 24f28f026b3f
   ---> 444f21cf7bd5
  Successfully built 444f21cf7bd5
  Successfully tagged hello-docker:latest
```
这些**层**具有多种功能。我们经常尝试限制层数以节省存储空间，但层可以在构建时用作缓存。
假设有一个修改需求，只需要编辑 Dockerfile 的最后几行，构建命令可以从上一层开始，然后直接跳到已更改的部分。COPY 会自动检测文件中的更改，因此，如果我们更改 hello.sh 它将从步骤 3/4 开始运行，跳过 1 和 2。这可用于创建更快的生成传递。

中间容器(The intermediate containers)是由执行命令的image创建的容器。
然后，改变的状态被存储到一个image中。
我们可以手动做类似的任务和一个新层 ：
创建一个名为`additional.txt`的新文件，让我们把它复制到容器内，并在此过程中学习新的技巧 我们将需要两个终端，所以我将用1和2标记，分别表示代表两个不同的终端。

命令行1：
```console
docker run -it hello-docker sh

 /usr/src/app #
```

现在我们进入了容器内部。我们用前面定义的 CMD 替换了 CMD，并使用 -i 和 -t 来启动容器，`sh`以便我们可以与它进行交互。

在第二个终端中，我们将在此处复制文件。
命令行2：

```consloe
docker ps

CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                                       NAMES
d02e9573dfa7   hello-docker             "sh"                     8 minutes ago   Up 8 minutes                                               stoic_gates
```
可以看见默认赋予的名字是stoic_gates。

**更改容器：**
```console
docker cp ./additional.txt stoic_gates:/usr/src/app/
```
从`./additional.txt`拷贝到`stoic_gates:/usr/src/app/`

已对容器进行了更改。
可以用`diff`来检查发生了什么变化
```console
docker diff stoic_gates
```

```console
# docker diff stoic_gates

C /usr
C /usr/src
C /usr/src/app
A /usr/src/app/additional.txt
C /root
A /root/.ash_history
```

文件名前面的字符表示容器文件系统中更改的类型：A = 已添加，D = 已删除，C = 已更改。额外的.txt已经创建，我们创建的.ash_history。
接下来，我们将更改另存为新图层！
```console
docker commit 容器名 别名注释
```
类似git的commit，但会创造一个新分支。
```console
docker commit stoic_gates hello-docker-additional
```

```console
docker images

REPOSITORY                          TAG         IMAGE ID       CREATED          
hello-docker-additional             latest      2b0dccddacbe   49 seconds ago   
hello-docker                        latest      ec6fdd543685   25 minutes ago   
```

实际上永远不会再使用 docker 提交（docker commit）。
这是因为定义对 Dockerfile 的更改是管理更改的更可持续的方法。没有神奇的操作或脚本，只有一个可以版本控制的Dockerfile。

让我们这样做，并使用包含附加additional.txt的 V2（第二版本）创建 hello-docker。

**Dockerfile**

```dockerfile
# Start from the alpine image
FROM alpine:3.13

# Use /usr/src/app as our workdir. The following instructions will be executed in this location.
WORKDIR /usr/src/app

# Copy the hello.sh file from this location to /usr/src/app/ creating /usr/src/app/hello.sh.
COPY hello.sh .

# 执行一个带有`/bin/sh -c`前缀的命令。
RUN touch additional.txt

# When running docker run the command will be ./hello.sh
CMD ./hello.sh
```


`docker build . -t hello-docker:v2`
然后
比较 ls 的输出:
```text
$ docker run hello-docker-additional ls
  additional.txt
  hello.sh

$ docker run hello-docker:v2 ls
  additional.txt
  hello.sh
```
**Dockerfile 中除** CMD（以及我们即将了解的另一条指令）之外的所有指令都是在构建时执行的。当我们调用 docker run 时，**CMD** 会执行，除非我们覆盖它。