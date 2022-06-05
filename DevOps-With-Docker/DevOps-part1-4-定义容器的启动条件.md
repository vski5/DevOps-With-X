# DevOps-part1-4-定义容器的启动条件

打开一个交互式会话并添加测试内容，然后再将其“存储”到我们的 Dockerfile 中。

-it 运行有交互页面的Ubuntu，然后在Ubuntu容器里面下载curl，再用curl下载youtube-dl：
```console
docker run -it ubuntu:18.04

apt-get update && apt-get install -y curl


curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
```


接下来，我们将添加权限并运行它：
```console
chmod a+rx /usr/local/bin/youtube-dl
$ youtube-dl
  /usr/bin/env: 'python': No such file or directory
```

好的 - 在下载页面的顶部，我们注意到以下消息：`youtube-dl`

请记住，youtube-dl需要Python版本2.6，2.7或3.2 +才能工作，除了Windows exe。

所以我们将添加python。

```console
$ apt-get install -y python
```

让我们再次运行它

```console
$ youtube-dl
# 然后报错
 WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this.
  Usage: youtube-dl [OPTIONS] URL [URL...]

  youtube-dl: error: You must provide at least one URL.
  Type youtube-dl --help to see a list of all options.
```
输出了有关`LC_ALL`的警告。
不同于正常的Ubuntu发行版，在此image中，它们没有设置。
解决方法：
```console
LC_ALL=C.UTF-8 youtube-dl
```
并尝试下载视频：
```console
$ export LC_ALL=C.UTF-8
$ youtube-dl https://imgur.com/JY5tHqr
```

现在根据这个做好的ubuntu的思路制造Dockerfile文件：
1. `FROM ubuntu:18.04`
2. 我们应该始终尝试将最容易发生变化的行放在底部
3. 通过将指令添加到底部，我们可以保留我们的缓存层layers- 这是一个方便的做法，当Docker文件有耗时的操作（如下载）时，可以加快创建初始版本。
4.` WORKDIR /mydir `  这将确保视频将被下载到那里
5. `ENV LC_ALL=C.UTF-8` 不用原始的Ubuntu这个image里的环境配置，自己配置一下。
6. `CMD ["/usr/local/bin/youtube-dl"] `打开youtube-dl

```dockerfile
FROM ubuntu:18.04

WORKDIR /mydir

RUN apt-get update && apt-get install -y curl python
RUN curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
RUN chmod a+x /usr/local/bin/youtube-dl

ENV LC_ALL=C.UTF-8

CMD ["/usr/local/bin/youtube-dl"]
```

youtube-dl正确的使用要命令行中的网站后缀：
`youtube-dl [OPTIONS] URL [URL...]`

所以不能用CMD启动youtube-dl，要用`ENTRYPOINT`，这样可以从命令行添加输入。
```dockerfile
FROM ubuntu:18.04

WORKDIR /mydir

RUN apt-get update && apt-get install -y curl python
RUN curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
RUN chmod a+x /usr/local/bin/youtube-dl

ENV LC_ALL=C.UTF-8

# Replacing CMD with ENTRYPOINT
ENTRYPOINT ["/usr/local/bin/youtube-dl"]
```
现在：
```console
$ docker build -t youtube-dl .
$ docker run youtube-dl https://imgur.com/JY5tHqr
```


`ENTRYPOINT`将容器内的命令与`docker run 镜像名 `后面的命令结合在一起了。

`ENTRYPOINT`在正确设置的图像中，例如我们的youtube-dl，该命令表示入口点的参数列表。
默认情况下，入口点设置为`CMD`，
如果未设置入口点，则传递此值`/bin/sh`。
这就是为什么将脚本文件的路径作为CMD提供工作的原因：您将文件作为参数提供给`/bin/sh`

**shell是运行在终端中的文本互动程序，bash（GNU Bourne-Again Shell）是最常用的一种shell。**

由于 docker 运行结束时的命令将是 CMD，因此我们希望使用 ENTRYPOINT 指定要运行的内容，并使用 CMD 指定要运行的命令（在本例中为 url）。
**大多数时候，我们可以在**构建映像时忽略 ENTRYPOINT，而只使用 CMD。例如，ubuntu 映像将 ENTRYPOINT 默认为 bash，因此我们不必担心它。它为我们提供了便利，允许我们轻松覆盖CMD，例如，使用bash进入容器内部。

还有两种方法可以设置它们：**exec** form和**shell** form。
我们一直在使用执行命令本身的exec表单。
在 shell 形式中，执行的命令是包装的 - 当您需要在命令中计算类似`/bin/sh -c`或`$MYSQL_PASSWORD`类似的环境变量时，它很有用。

在 shell 形式中，命令以不带方括号的字符串形式提供。

在exec形式中，命令及其参数以列表形式提供（带括号），请参阅下表：

exec形式的命令及其参数
![[exec形式的命令及其参数.png]]

假设要用一个Python的CMD
```console
$ docker pull python:3.8
...
$ docker run -it python:3.8
  Python 3.8.2 (default, Mar 31 2020, 15:23:55)
  [GCC 8.3.0] on linux
  Type "help", "copyright", "credits" or "license" for more information.
  >>> print("Hello, World!")
  Hello, World!
  >>> exit()

$ docker run -it python:3.8 --version
  docker: Error response from daemon: OCI runtime create failed: container_linux.go:370: starting container process caused: exec: "--version": executable file not found in $PATH: unknown.

$ docker run -it python:3.8 bash
  root@1b7b99ae2f40:/#
```

从这个实验中，我们了解到他们有ENTERPOINT作为python以外的其他东西，但CMD是python，我们可以覆盖它，在这里用`docker run -it python:3.8 bash`改成了bash。

```dockerfile
FROM python:3.8
ENTRYPOINT ["python3"]
CMD ["--help"]
```
结果是一个将python作为ENTENTISTPOINT的图像，
您可以在末尾添加命令，例如` --version`以查看版本。在不覆盖命令的情况下，它将输出帮助。

**总结：**
` ENTRYPOINT` 用于添加CMD中命令的前缀

现在我们在youtube-dl项目上有两个问题：

-   下载的文件保留在容器中
    
-   我们的容器构建过程会创建许多层，从而导致图像大小增加

# 将下载的文件保存到本地而非容器。
通过检查，我们可以看到之前的所有运行。
当我们过滤此列表时`docker container ls -a`

```console
$ docker container ls -a --last 3

  CONTAINER ID        IMAGE               COMMAND                   CREATED                  STATUS                          PORTS               NAMES
  be9fdbcafb23        youtube-dl          "/usr/local/bin/yout…"    Less than a second ago   Exited (0) About a minute ago                       determined_elion
  b61e4029f997        f2210c2591a1        "/bin/sh -c \"/usr/lo…"   Less than a second ago   Exited (2) About a minute ago                       vigorous_bardeen
  326bb4f5af1e        f2210c2591a1        "/bin/sh -c \"/usr/lo…"   About a minute ago       Exited (2) 3 minutes ago                            hardcore_carson
```

我们看到最后一个容器是`be9fdbcafb23` 名为`determined_elion`
`docker diff `查看更改：
```console
$ docker diff determined_elion

  C /mydir
  A /mydir/Imgur-JY5tHqr.mp4
```

让我们尝试命令`docker cp`来复制文件。

如果文件名有空格，我们可以使用引号。

```console
$ docker cp "determined_elion://mydir/Imgur-JY5tHqr.mp4" .
```

现在我们的文件在本地。可悲的是，这不足以解决我们的问题。在下一节中，我们将对此进行改进。

