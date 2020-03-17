# Linux 常见命令

## 1. Linux介绍

### 1.1 各个文件夹的作用

> [yanliang的博客](https://www.cnblogs.com/yoke/p/7217019.html)  
> [玉米疯收的博客](https://www.cnblogs.com/amboyna/archive/2008/02/16/1070474.html)

Linux号称一切皆文件，首先让我们来看看Linux文件，能有什么特殊的吧。

- `/bin`

普通用户的二进制可执行文件。/bin/usr/bin下，可以看到有ls，cp等可执行文件。

- `/boot`

存放引导加载器（bootstrap loader）使用的文件

- `/dev`（英文全称：device）

设备文件，Linux下一切皆文件，设备也是如此。

/input/mouse0来访问鼠标的输入，就像访问其他文件一样。这些文件都可以通过mkdir创建，为了将对这些`设备文件的访问`转化为`对设备的访问`，需要向响应的设备提供设备驱动模块。

- `/etc`（英文全称：editable text configuration）

/etc/passwd: 用户信息文件，给出了用户名、真实姓名、用户起始目录、加密口令和用户的其他信息。

/etc/group: 类似/passwd，但说明的不是用户信息而是组的信息。包括组的各种数据。

/etc/rc: 系统初始化文件，rc.d启动的配置文件和脚本

- `/home`

用户家目录的起点

- `/lib`

根文件系统上的程序所需要的共享库，避免每个程序都拥有相同子程序的副本，节省空间。

/usr/lib/modules: 包含系统核心可加载各个模块，如网络和文件系统驱动

- `/media`

挂载的媒体设备目录

- `/mnt（mount）`

让用户临时挂载其他的文件系统

- `/opt（optional）`

可选择的文件目录，一些软件包可以自定义安装在这里

- `/proc（process）`

虚拟的目录，是系统内存的映射。可直接访问这个目录来获取系统信息。

/proc/数字n: 关于进程n的信息目录，这里的n是这一进程的标识符

/proc/cpuinfo: 存放CPU的信息，如CPU的类型、制造商、型号和性能

/proc/devices: 当前运行的核心配置的驱动设备的列表

- `/root`

超级用户的目录

- `/run`

- `/sbin`（system binary）

系统管理命令。这个文件夹下是超级权限用户root的可执行命令存放地，普通用户无权执行这个目录下的命令（但是有时普通用户也需要用到，可以通过其他途径获取这个权限），我们要记住，凡是目录sbin中包含的都是root权限才能执行的。

/usr/sbin  例如有引导系统的init程序

- `/srv`（service）

- `/sys`

- `/tmp`（temporary）

临时文件目录

- `/usr`（unix share resource）

最庞大的目录，要用到的应用程序和文件几乎都在这个目录，这个目录中包含了命令库文件和通常操作中不会修改的文件。

/usr/bin: 集中了所有用户的命令，是系统的软件库

/usr/sbin: 包括根文件系统不必要的命令管理命令，例如多数服务程序

/usr/include: Linux下开发和编译应用程序的头文件

/usr/lib: 常用的动态链接库和软件包的配置文件

/usr/src: 源代码

/usr/local: 本地安装的程序都在 /usr/local下，这样子可以在系统升级新版本时无需重新安装程序。 /usr/local/bin：本地增加的命令；/usr/local/lib: 本地增加的库

/usr/share: 存放共享文件的目录

- `/var`

包含系统运行时要改变的数据。这些数据所在的目录的大小是要经常变化或扩充的。

### 1.2 常见的Linux发行版本

> [内核版本](https://www.kernel.org/)

- red hat: 软件经过专业的测试，很稳定
- fedora: 软件比较新，但是稳定性不好
- centos: 免费安全。其中版本号有规律，如7.6.1810代表centos7,经过6次设计，在18年10月发行
- debian: 界面良好
- ubuntu: 界面良好

Linux用命令行，目的是为了保证服务端的稳定性。图形化界面可能会导致不稳定。Linux主要的应用为：服务端的运维和开发。

## 2. Linux常用操作

### 2.1 修改时区

```bash
cp  /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```

### 2.2 init的可选择操作

- `init 3`: 进入全命令行模式
- `init 5`: 图形化界面
- `init 0`: 关机

### 2.3 帮助命令

- `man`(manual)

```
篇章7，因为可能有重名的情况
man 7 man 

查看所有的命令
man -a 命令
```

- `help` 帮助

```
help cd

ls --help
```

shell（命令解释器）自带的命令称为内部命令，其他命令是外部命令

- `info` 帮助

info帮助比help更详细，是help的补充

- 如果还看不懂，则善用搜索引擎和文档

- 命令选项是：拓展命令的功能

### 2.3 最常用的操作

