# DevOps-part1-2-运行和停止容器

添加一些标志来与命令进行交互。
`-t` 将创建一个 tty。
```console
docker run -t ubuntu
```

在容器内，如果我们输入`ls`并按回车键...没有任何反应。
因为我们的终端没有将消息发送到容器中。
标志`-i`将指示将 STDIN （标准输入）传递到容器。
如果您被困在另一个终端，您可以停止容器。

```console
docker run -it ubuntu
```

3个有用的标志，可以-it叠在一起。 
1. 交互式传递标准输入-i、
2. tty是-t、
3. 独立容器是-d

后台运行容器：
如果您是命令提示符（Windows）用户，则必须在脚本周围使用双引号
```console
docker run -d -it --name looper ubuntu sh -c 'while true; do date; sleep 1; done'
```
1. 运行分离容器。`docker run -d`
2. 允许你使用命令行与容器交互。`-it`
3. 使用`--name looper`运行容器，所以现在可以轻松引用它，名字是looper。
4. 镜像是`ubuntu`，它后面的是提供给容器的命令。
5. 这个命令会导致一个不断运行的循环器，打印当前的时间。

要检查它是否正在运行，请运行`docker container ls`

按照日志的输出`-f`
```console
$ docker logs -f looper
  Thu Feb  4 15:51:29 UTC 2021
  Thu Feb  4 15:51:30 UTC 2021
  Thu Feb  4 15:51:31 UTC 2021
  ...
```

**让我们测试一下暂停循环器而不退出或停止它。**
在另一个终端运行`docker pause looper` ,注意在第一个终端中，日志输出已经暂停了。**整个container都被暂停了。**
要取消暂停，运行`docker unpause looper`

pause就是暂停的意思。

保持日志开放，用`attach`从第二个终端连接到运行中的容器。
```console
docker attach looper
You cannot attach to a paused container, unpause it first
```

先要取消暂停，`docker unpause looper`

现在，在两个终端中运行了进程日志 （STDOUT）。

现在按连接窗口中的 control+c。然后因为进程不再运行，容器已停止。

如果我们想连接到容器，同时确保不从其他终端关闭它，我们可以指定不附加STDIN与选项`--no-stdin`。
让我们启动已停止的容器`docker start looper`，并使用`--no-stdin`连接到它。
```console
docker attach --no-stdin looper
```
按ctrl+C只会断开你与 STDOUT 的连接。
容器将继续运行。

要进入容器，我们可以在其中启动一个新进程。

```console
$ docker exec -it looper bash
```

exec 是 bash 的内置命令。shell 的内件命令exec执行命令时，不启用新的shell进程。exec是用被执行的命令行替换掉当前的shell进程，且exec命令后的其他命令将不再执行。

再运行`ps aux`
```console
root@2a49df3ba735:/# ps aux

  USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root         1  0.2  0.0   2612  1512 pts/0    Ss+  12:36   0:00 sh -c while true; do date; sleep 1; done
  root        64  1.5  0.0   4112  3460 pts/1    Ss   12:36   0:00 bash
  root        79  0.0  0.0   2512   584 pts/0    S+   12:36   0:00 sleep 1
  root        80  0.0  0.0   5900  2844 pts/1    R+   12:36   0:00 ps aux
```

从`ps aux`列表中，我们可以看到我们的`bash`流程的PID（流程ID）为64。

现在我们已经在容器中，它的行为就像你对ubuntu的期望一样，我们可以`exit`退出容器，然后杀死或停止容器。
我们的looper进程不会因为stop命令发出的SIGTERM信号而停止。
（SIGTERM：用于终止程序的通用信号。）

为了终止进程，stop会在一个宽限期后用SIGKILL跟随SIGTERM。在这种情况下，使用kill会更快。
```console
$ docker kill looper
$ docker rm looper
```

运行前两个命令基本上等同于运行`docker rm --force looper`

让我们启动另一个进程 并添加`-it`和`--rm`，以便在退出后自动删除它。
`--rm`确保没有留下垃圾容器。
这也意味着`docker start`不能用于在容器退出后启动容器。

```console
$ docker run -d --rm -it --name looper-it ubuntu sh -c 'while true; do date; sleep 1; done'
```

让我们attach加到容器并按 control+p、control+q 将我们从 STDOUT 中分离出来。

```console
$ docker attach looper-it

  Mon Jan 15 19:50:42 UTC 2018
  Mon Jan 15 19:50:43 UTC 2018
  ^P^Qread escape sequence
```

相反，如果我们使用ctrl+c，它将发送一个终止信号，然后按照我们在命令中指定的那样删除容器。