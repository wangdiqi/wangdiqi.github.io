---
layout: post
title: "openresty基本知识"
subtitle: "openresty"
date: 2018-11-01 17:00:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - openresty
---



## 基本概要

### nginx的11个阶段
1.post-read:(ngx_realip), nginx读取并解析完请求头(request headers)之后就立即开始运行。  
支持nginx模块注册处理程序
2.server-rewrite:(ngx_rewrite配置在server配置块中时)。支持nginx模块注册处理程序  
3.find-config:由nginx核心来完成当前请求与location配置块之间的配对工作。不支持nginx模块注册处理程序  
4.rewrite:(set, set_unescape_uri),主要做一些对URI、URL参数的改写，创建并初始化一系列后续处理阶段  
可能需要的nginx变量。支持nginx模块注册处理程序  
5.post-rewrite:由nginx核心完成rewrite阶段所要求的“内部跳转”操作(如果rewrite阶段有此要求的话)。  
不支持nginx模块注册处理程序  
6.preaccess:(ngx_limit_req, ngx_limit_zone)。支持nginx模块注册处理程序  
7.access:(allow, deny, ngx_auth_request, ngx_access, ngx_auth_request, access_by_lua),主要执行  
访问控制性质的任务，比如检查用户的访问权限，检查用户的来源IP地址是否合法。支持nginx模块注册处理程序  
8.post-access:由nginx核心完成一些处理工作，主要用于配合access阶段实现标准ngx_http_core模块  
提供的配置指令satisfy的功能。不支持nginx模块注册处理程序  
9.try-files:专门用于实现标准配置指令try_files的功能(在许多FastCGI应用的配置中用到)。不支持nginx模块注册处理程序  
10.content:(echo),1个location只能有一个“内容处理程序”。支持nginx模块注册处理程序  
11.log:主要的目的就是记录访问日志，进入该阶段表明该请求的响应已经发送到系统发送缓冲区  
支持nginx模块注册处理程序

### 注意点
1.ngx_echo， ngx_lua，以及 ngx_srcache 在内的许多第三方模块都选择了禁用父子请求间的变量共享  
2.并非所有的内建变量都作用于当前请求，少数内建变量只作用于主请求，比如说：$request_method  

a.使用echo_location做子请求，自定义和内置变量不可访问  
b.使用echo_exec做跳转，内置变量不可访问，自定义变量可访问  
c.使用rewrite做跳转，都可访问  

3.$http_XXX变量在匹配请求头时会自动对请求头的名字进行归一化，即将名字的大写字母转换为小写字母，同时  
把间隔符(-)替换为下划线(_)

4.set_by_lua 可以和ngx_rewrite模块按顺序配合使用
5.rewrite_by_lua 运行在rewrite阶段的末尾


### ngx_rewrite模块
1.set配置可以对变量赋值：set $a "hello world";  
2.rewrite命令(主请求)发起location内部跳转:rewrite ^ /bar;  

### ngx_echo模块
1.echo配置指令可以将变量值作为当前请求的响应体输出: echo $foo  
2.echo_exec命令(主请求)，发起location的内部跳转，nginx变量值是共享的：echo_exec /bar;  
3.echo_location(子请求)，发起的“子请求”(内部跳转),其执行是按照配置书写的顺序串行处理的，即只有当  
第一个子请求处理完，才会接着发起第二个子请求  


### ngx_http_core模块
1.$uri,可以用来获取当前请求的URI(经过解码，并且不含请求参数)  
2.$request_uri则获取请求最原始的URI(未经解码，并且包含请求参数)  
3.$arg_XXX变量集合，参数名称及值，会将参数名调整为全部小写  
4.$cookie_XXX cookie值的  
5.$http_XXX 请求头的  
6.$sent_http_XXX响应头的  
7.$args 获取url参数字符串  

注意变量的属性(只读or可写)  
arg_XXX是可写的  


### ngx_set_misc模块
1.set_unescape_uri进行编码序列的解码:set_unescape_uri $name $arg_name;  


### ngx_proxy模块
1.proxy_pass配置方向代理,默认将当前请求的url参数也转发到远方服务: proxy_pass http://127.0.0.1:8081/args;


### ngx_index模块
作用于URI以/结尾的请求，否则直接忽略  
模块主要用于在文件系统目录中自动查找指定的首页文件，类似 index.html 和 index.htm 这样的，例如：  
~~~
location / {
        root /var/www/;
        index index.htm index.html;
}
~~~
这样，当用户请求 / 地址时，Nginx 就会自动在 root 配置指令指定的文件系统目录下依次寻找 index.htm 和 index.html 这两个文件。如果 index.htm 文件存在，则直接发起“内部跳转”到 /index.htm 这个新的地址；而如果 index.htm 文件不存在，则继续检查 index.html 是否存在。如果存在，同样发起“内部跳转”到 /index.html；如果 index.html 文件仍然不存在，则放弃处理权给 content 阶段的下一个模块


### ngx_autoindex模块
作用于URI以/结尾的请求，否则直接忽略

### ngx_static模块
作用于URI不是以/结尾的请求，否则直接忽略

### ngx_map模块
1.map配置命令;default是一个特殊的匹配条件，即当其他条件都不匹配的时候，这个条件才匹配。当这个默认条件  
匹配时，就把“因变量”$foo映射到值0。第二行的意思是说，如果"自变量"$args精确匹配了debug这个字符串，则把  
“因变量”$foo映射到值 1.  
~~~
map $args $foo {
    default 0;
    debug 1;
}
~~~

### ngx_auth_request
auth_request的子请求与父请求共用变量
~~~
location /main {
    set $var main;
    auth_request /sub;
    echo "main: $var";
}

location /sub {
    set $var sub;
    echo "sub: $var";
}
~~~

### ngx_geo模块
提供的配置指令geo来为变量$dollar赋予字符串"$":  
~~~
geo $dollar{
    default "$";
}

server {
    listen 8080;
    location /test {
        echo "this is a dollar sign: $dollar";
    }
}
~~~



## 调试分析

#### 使用openresty-gdb-utils

1. 下载openresty-gdb-utils, gdb等

~~~
yum install wget unzip gdb
cd /opt/aispeech/
wget https://github.com/openresty/openresty-gdb-utils/archive/master.zip
unzip master.zip
~~~


2. 设置~/.gdbinit (注意修改下边两处gdb-utils路径)

文件: ~/.gdbinit

~~~
directory /opt/aispeech/openresty-gdb-utils-master/

py import sys 
py sys.path.append("/opt/aispeech/openresty-gdb-utils-master/")

source luajit20.gdb 
source ngx-lua.gdb 
source luajit21.py 
source ngx-raw-req.py 
set python print-stack full
~~~


3. 开始gdb

gdb -p nginx-worker-pid

查看所有GC Objects的状态:
lgcstat



4. 使用lbt寻找lua死循环

文件: lbt.sh

~~~
#!/bin/bash
pid=$1
interval=$2
echo 'lbt
set confirm off
quit' >  .lbt.gdb
while ((1)); do
    echo ---- `date` ----
    gdb -p $pid -x .lbt.gdb 2>/dev/null | grep lua:
    sleep $interval
done
~~~

例如对pid 17的进程, 每隔1s打印一次backtrace

~~~
$bash ./lbt.sh 17 1
---- Tue Sep 25 22:41:03 CST 2018 ----
@./lualib/DFA.lua:39
@./lualib/DFA.lua:169
@./lualib/DFA.lua:160
@./lualib/aios/node/filter.lua:20
@./lualib/aios/bus/client.lua:185
@./lualib/aios/node/filter.lua:93
@/opt/aispeech/./www/boot.lua:262
~~~


使用lbt full查看变量值
~~~
# gdb -p 17
lbt full
~~~

## systemtap使用

注: 如下为千并发测试过程中的工作日志写得较随意



在宿主机上安装systamp就可以分析容器里的nginx进程了

查看内核版本号是否一致(据说不一致会引起问题)
uname -r
rpm -qa|grep kernel

下载地址 http://debuginfo.centos.org/
wget http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-$(uname -r).rpm 
wget http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-common-x86_64-$(uname -r).rpm

安装
# rpm -ivh kernel-debuginfo-common-x86_64-$(uname -r).rpm
# rpm -ivh kernel-debuginfo-$(uname -r).rpm
# yum install systemtap

测试是否安装成功
stap -vv -e 'probe vfs.read {printf("read performed\n"); exit()}’

若出现错误
ERROR: module version mismatch (#1 SMP Wed Jul 5 08:40:24 CDT 2017 vs #1 SMP Tue Jul 4 15:04:05 UTC 2017), release 3.10.0-514.26.2.el7.x86_64

则修改UTS_VERSION定义
#vim /usr/src/kernels/3.10.0-514.26.2.el7.x86_64/include/generated/compile.h

并删除cache
#rm -rf ~/.systemtap/cache/

安装openresty-systemtap-toolkit
wget https://codeload.github.com/openresty/openresty-systemtap-toolkit/zip/master -O openresty-systemtap-toolkit-master.zip
wget https://codeload.github.com/openresty/stapxx/zip/master -O stapxx-master.zip

安装FlameGraph
curl https://codeload.github.com/brendangregg/FlameGraph/zip/master -o FlameGraph-master.zip

测试一下
ps aux | grep nginx
./stap++ ./samples/lj-lua-stacks.sxx --arg time=5 --skip-badvars -x 11489 > tmp.bt

若不能执行, 可能需要宿主机里也安装openresty(直接拷贝容器里的即行)
cd shun.zhang/opt/aispeech
rm -rf olive openresty conf luaclib lualib www
docker cp 0cc:/opt/aispeech/olive . 
docker cp 0cc:/opt/aispeech/openresty .
docker cp 0cc:/opt/aispeech/lualib .
docker cp 0cc:/opt/aispeech/conf .
docker cp 0cc:/opt/aispeech/luaclib .
docker cp 0cc:/opt/aispeech/www .
rm -f /opt/aispeech
ln -sf  `pwd` /opt/aispeech

常见错误解决
ERROR: Skipped too many probes, check MAXSKIPPED or try again with stap -t for more details. 
-D MAXSKIPPED=102400

ERROR: probe overhead exceeded threshold
-D STP_NO_OVERLOAD

ERROR: Array overflow, check MAXMAPENTRIES near identifier….
-D MAXMAPENTRIES=1024000

在跳板机里下载文件
D d1-dui-045::/root/shun.zhang/opt/aispeech/output/flame.svg

再测试一下
./stap++ ./samples/lj-lua-stacks.sxx --arg time=5 --skip-badvars -x 11489 > tmp.bt 
./fix-lua-bt tmp.bt > flame.bt 
(建议直接用stap-flame-graph.sh)

监控内存占用
while((1)); do ../../stapxx-master/stap++ ../../stapxx-master/samples/lj-gc-objs.sxx -x 17708 -D MAXACTION=100000000 -D MAXSKIPPED=10000000 -D STP_NO_OVERLOAD 2>/dev/null | grep "GC total size"; sleep 10; done
(建议直接用stap-watch.sh 或 stap-watch-all.sh)

----
参考
http://www.hi-roy.com/2016/07/27/CentOS7%E5%AE%89%E8%A3%85systemtap/
https://gist.github.com/alexandrnikitin/7fd095371e8c105f6cb9ab7f87664547
https://moonbingbing.gitbooks.io/openresty-best-practices/content/flame_graph/install.html


## 常用网络连接状态分析脚本

1、统计各状态FD个数
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'


2、统计各地址FD个数
h=`ifconfig | awk '/inet / && !/127/{print $2}'`; echo h | while read h; do netstat -anop | grep -v $h:28002 | awk '/^tcp/ && !/*/ {++S[$5]} END {for(a in S) print a, S[a]}' | sort -k2 -nr; done

3、查看各进程的各种连接状态的连接统计
netstat -anop | awk '/^tcp/{print $7}' | sort -u | while read pid; do netstat -anop | grep " $pid" | awk -v pid=$pid 'BEGIN {S["ESTABLISHED"]=0; S["CLOSE_WAIT"]=0; S["TIME_WAIT"]=0; S["LISTEN"]=0;} /^tcp/ {++S[$6]} END {printf("%-24s", pid); for (a in S){printf("%-16s %-8s", a, S[a]);} print "";}'; done | sort -n -k 7


4、查看各个容器里的各状态的连接统计(在d1-k8s-master-001上执行)
kubectl get pods -n cloud | awk '/ddsserver.*smooth/{print $1}' | while read c; do echo -n $c; kubectl exec -n cloud  $c -- bash -c "netstat -n | awk 'BEGIN {S[\"ESTABLISHED\"]=0; S[\"CLOSE_WAIT\"]=0; S[\"TIME_WAIT\"]=0; S[\"LISTEN\"]=0;} /^tcp/ {++S[\$NF]} END {for(a in S) {printf(\"  %-16s%-8s\", a, S[a]);T+=S[a]} printf(\"TOTAL  %s\n\", T);}'"; done


5、查看各个容器里的各进程的各状态的连接统计(在d1-k8s-master-001上执行)
kubectl get pods -n cloud | awk '/ddsserver.*smooth/{print $1}' | while read c; do echo; echo $c; kubectl exec -n cloud  $c -- bash -c "netstat -anop | awk '/^tcp/{print \$7}' | sort -u | while read pid; do netstat -anop | grep \" \$pid\" | awk -v pid=\$pid 'BEGIN {S[\"ESTABLISHED\"]=0; S[\"CLOSE_WAIT\"]=0; S[\"TIME_WAIT\"]=0; S[\"LISTEN\"]=0;} /^tcp/ {++S[\$6]} END {printf(\"    %-24s\", pid); for (a in S){printf(\"%-16s %-8s\", a, S[a]);} print \"\";}'; done | sort -n -k 7"; done


6、查看各个容器里所有nginx worker状态, 查找CPU占用过大或内存占用过大的进程(在d1-k8s-master-001上执行)
kubectl get pods -n cloud | awk '/ddsserver.*smooth/{print $1}' | while read c; do echo; echo $c; kubectl exec -n cloud  $c -- bash -c "ps aux | awk '/[0-9] nginx.*process/'"; done


7、强杀CLOSE_WAIT大于100的nginx进程(在d1-k8s-master-001上执行)
kubectl get pods -n cloud | awk '/ddsserver.*smooth/{print $1}' | while read c; do echo; echo $c; kubectl exec -n cloud  $c -- bash -c "netstat -anop | awk '/^tcp/{print \$7}' | sort -u | while read pid; do netstat -anop | grep \"\$pid\" | awk -v pid=\$pid '/^tcp/ {++S[\$6]} END {if(S[\"CLOSE_WAIT\"]>100){n=gsub(/\/.*/, \"\", pid); system(\"kill -9 \"pid); print \"kill \", pid}};'; done | sort -n -k 7"; done


8、重启所有容器里的nginx
kubectl get pods -n cloud | awk '/ddsserver.*smooth/{print $1}' | while read c; do echo; echo $c; kubectl exec -n cloud  $c -- bash -c "kill -1 1"; done

