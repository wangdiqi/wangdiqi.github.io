---
layout: post
title: "docker学习"
subtitle: "docker"
date: 2019-02-13 15:29:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - docker
---

> "Let's go"

# 概述
docker 容器 =cgroup+namespace+secomp+capability+selinux

cgexec 是 cgroup 提供的一个工具，可以在启动时就将程序运行到某个 cgroup 中

nsenter 是一个 namespace 相关的工具，通过它可以进入某个进程所在的 namespace

# 原理基础

docker项目来说，它最核心的原理实际上就是为待创建的用户进程：
1. 启用Linux Namespace配置
2. 设置指定的Cgroups参数
3. 切换进程的根目录(Change Root)

Docker项目在最后一步的切换上会优先使用pivot_root系统调用，如果系统不支持，才会使用chroot。这两个系统调用虽然功能类似，但是也有细微的区别。

rootfs只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在Linux操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。
所以说，rootfs只包括了操作系统的“躯壳”，并没有包括操作系统的“灵魂”

## namespace

隔离作用，所有的namespace：cgroup/ipc/network/mount/pid/time/user/uts

~~~
man namespaces

具体看某个进程的namespace可以看
/proc/33090/ns/ 下的文件
~~~

## cgroups

~~~
man cgroups

具体看cgroup的配置
/sys/fs/cgroup/memory/docker/
~~~

## lxcfs
可解决容器内top显示宿主机的信息

## 容器启动前挂载整个根目录"/"

chroot 命令
将使用$HOME/test目录作为/bin/bash进程的根目录
```
chroot $HOME/test /bin/bash
```

```
T=$HOME/test
list="$(ldd /bin/ls | egrep -o '/lib.*\.[0-9]')"
for i in $list; do cp -v "$i" "${T}${i}"; done
```

## rootfs (根文件系统)

### 只读层(ro+wh : readonly+whiteout)

以增量的方式分别包含了ubuntu的操作系统的一部分
```
docker inspect image <docker_image>
```

### Init层(ro+wh : readonly+whiteout)

Init层是Docker项目单独生成的一个内部层，专门用来存放/etc/hosts、/etc/resolv.conf等信息

需要这样一层的原因是，这些文件本来属于只读的 Ubuntu 镜像的一部分，但是用户往往需要在启动容器时写入一些指定的值比如 hostname，所以就需要在可读写层对它们进行修改。
可是，这些修改往往只对当前的容器有效，我们并不希望执行 docker commit 时，把这些信息连同可读写层一起提交掉。

### 可读写层(rw : read write)

这个容器的riitfs最上面的一层，专门用来存放你修改rootfs后产生的增量

## Union File System (新版docker已经使用overlay2)
也叫UnionFS，最主要的功能是将多个不同位置的目录联合挂载到同一个目录下

AuFS的全称是Another UnionFS，后改名为Alternative UnionFS，再后来干脆改名叫作Advance UnionFS：
1. 它是对Linux原生UnionFS的重写和改进；
2. 只能在Ubuntu和Debian这些发行版上使用它。Linus Torvalds一直不让AuFS进入Linux内核主干

```
# 挂载点，挂载的目录下应该有完整的Ubuntu操作系统
/var/lib/docker/overlay2/<id>/merged下

docker inspect <contianer_id> 查到，df -h 也可以查到
```

## mount 命令

mount分为share方式和private方式挂载

```
mount -l | egrep tmpfs
```

## dockerinit

这个容器进程，是Docker创建的一个容器初始化进程(dockerinit)，而不是应用进程(ENTRYPOINT + CMD)。dockerinit会负责完成根目录的准备、挂载设备和目录、配置hostname等一系列
需要在容器内进行的初始化操作。最后，它通过execv()系统调用，让应用进程取代自己，成为容器里的PID=1的进程

## 容器运行时

CRI(Container Runtime Interface)远程调用接口

docker项目一般通过OCI这个容器运行时规范同底层的Linux操作系统进行交互：把CRI请求翻译成对Linux操作系统的调用(操作Linux Namespace和Cgroups等)

kubelet主要负责同容器运行时打交道
kubelet还通过gRPC协议同一个叫作Device Plugin的插件进行交互。这个插件是k8s项目用来管理GPU等宿主机物理设备的主要组件，也是基于 Kubernetes 项目进行机器学习训练、高性能作业支持等工作必须关注的功能。
而 kubelet 的另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储。这两个插件与 kubelet 进行交互的接口，分别是 CNI（Container Networking Interface）和 CSI（Container Storage Interface）。

# docker基础知识

## 常用命令

```
docker inspect --format '{{.State.Pid}}'  <container_id>

ll /proc/<pid>/ns
```


# 附录

## 模拟实现docker的代码

使用clone创建一个新的子进程container_main,并且声明要为它启用Mount Namespace(CLONE_NEWNS)
子进程执行了"/bin/bash"程序。这时在“容器内”执行ls，发现和宿主机内容一样。因为，我们没有指定挂载点。
```
#define _GNU_SOURCE
#include <sys/mount.h> 
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <sched.h>
#include <signal.h>
#include <unistd.h>
#define STACK_SIZE (1024 * 1024)
static char container_stack[STACK_SIZE];
char* const container_args[] = {
  "/bin/bash",
  NULL
};

int container_main(void* arg)
{
  printf("Container - inside the container!\n");
  // 如果机器的根目录挂载类型是shared，那必须先重新挂载根目录
  /*
    有读者反映，咱们重新挂载/tmp目录的实验执行完成后，在宿主机上居然可以看到这个挂载信息。
    这是怎么回事呢？实际上，大家自己装的虚拟机，或者云上的虚拟机的根目录，很多都是以share方式的挂载的。这时候，你在容器里做mount也会继承share方式。这样就会把容器内挂载传播到宿主机上。
    解决这个问题，你可以在重新挂载/tmp之前，在容器内先执行一句：mount(“”, “/“, NULL, MS_PRIVATE, “”) 这样，容器内的根目录就是private挂载的了
  */
  // mount("", "/", NULL, MS_PRIVATE, "");
  // 启动“容器”进程前先挂载
  mount("none", "/tmp", "tmpfs", 0, "");
  execv(container_args[0], container_args);
  printf("Something's wrong!\n");
  return 1;
}

int main()
{
  printf("Parent - start a container!\n");
  int container_pid = clone(container_main, container_stack+STACK_SIZE, CLONE_NEWNS | SIGCHLD , NULL);
  waitpid(container_pid, NULL, 0);
  printf("Parent - container stopped!\n");
  return 0;
}
```

用setns()linux系统调用进入已有的namespace
```

#define _GNU_SOURCE
#include <fcntl.h>
#include <sched.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define errExit(msg) do { perror(msg); exit(EXIT_FAILURE);} while (0)

int main(int argc, char *argv[]) {
    int fd;

    fd = open(argv[1], O_RDONLY);
    if (setns(fd, 0) == -1) {
        errExit("setns");
    }
    execvp(argv[2], &argv[2]); 
    errExit("execvp");
}
```

## 原始的部署两个实例
如果是自己 DIY 的话，可能需要启动两台虚拟机，分别安装两个 Nginx，然后使用 keepalived 为这两个虚拟机做一个虚拟 IP。

Keepalived起初是为LVS设计的，专门用来监控集群系统中各个服务节点的状态，
它根据TCP/IP参考模型的第三、第四层、第五层交换机制检测每个服务节点的状态，如果某个服务器节点出现异常，或者工作出现故障，Keepalived将检测到，
并将出现的故障的服务器节点从集群系统中剔除，这些工作全部是自动完成的，不需要人工干涉，需要人工完成的只是修复出现故障的服务节点。


# QA

## 你是否知道如何修复容器中的 top 指令以及 /proc 文件系统中的信息呢？（提示：lxcfs）

之前遇到过，但是没有考虑如何解决，临时抱佛脚，查了 lxcfs，尝试回答一下。top 是从 /prof/stats 目录下获取数据，所以道理上来讲，容器不挂载宿主机的该目录就可以了。lxcfs就是来实现这个功能的，做法是把宿主机的 /var/lib/lxcfs/proc/memoinfo 文件挂载到Docker容器的/proc/meminfo位置后。容器中进程读取相应文件内容时，LXCFS的FUSE实现会从容器对应的Cgroup中读取正确的内存限制。从而使得应用获得正确的资源约束设定。kubernetes环境下，也能用，以ds 方式运行 lxcfs ，自动给容器注入争取的 proc 信息。

## 在从虚拟机向容器环境迁移应用的过程中，你还遇到哪些容器与虚拟机的不一致问题？

用的是vanilla kubernetes，遇到的主要挑战就是性能损失和多租户隔离问题，性能损失目前没想到好办法，可能的方案是用ipvs 替换iptables ，以及用 RPC 替换 rest。多租户隔离也没有很好的方法，现在是让不同的namespace调度到不同的物理机上。也许 rancher和openshift已经多租户隔离。

## 既然容器的 rootfs（比如，Ubuntu 镜像），是以只读方式挂载的，那么又如何在容器里修改 Ubuntu 镜像的内容呢？（提示：Copy-on-Write）

上面的读写层通常也称为容器层，下面的只读层称为镜像层，所有的增删查改操作都只会作用在容器层，相同的文件上层会覆盖掉下层。
知道这一点，就不难理解镜像文件的修改，比如修改一个文件的时候，首先会从上到下查找有没有这个文件，找到，就复制到容器层中，修改，修改的结果就会作用到下层的文件，这种方式也被称为copy-on-write。

## 除了 AuFS，你知道 Docker 项目还支持哪些 UnionFS 实现吗？你能说出不同宿主机环境下推荐使用哪种实现吗？

包括但不限于以下这几种：aufs, device mapper, btrfs, overlayfs, vfs, zfs。
aufs是ubuntu 常用的，
device mapper 是 centos，
btrfs 是 SUSE，
overlayfs ubuntu 和 centos 都会使用，
现在最新的 docker 版本中默认两个系统都是使用的 overlayfs，
vfs 和 zfs 常用在 solaris 系统。

## 你在查看 Docker 容器的 Namespace 时，是否注意到有一个叫 cgroup 的 Namespace？它是 Linux 4.6 之后新增加的一个 Namespace，你知道它的作用吗？

## 如果你执行 docker run -v /home:/test 的时候，容器镜像里的 /test 目录下本来就有内容的话，你会发现，在宿主机的 /home 目录下，也会出现这些内容。这是怎么回事？为什么它们没有被绑定挂载隐藏起来呢？（提示：Docker 的“copyData”功能）

docker的copyData功能可以使挂载点文件和挂载目录下的文件同时存在

## 请尝试给这个 Python 应用加上 CPU 和 Memory 限制，然后启动它。根据我们前面介绍的 Cgroups 的知识，请你查看一下这个容器的 Cgroups 文件系统的设置，是不是跟我前面的讲解一致。
