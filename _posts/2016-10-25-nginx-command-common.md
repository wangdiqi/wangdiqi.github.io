---
layout: post
title: "Nginx 配置说明"
subtitle: ""
date: 2016-10-25 10:00:00
author: seventhking
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - nginx
---

> "内容来自书本[^1]和网络"

### 配置说明
~~~
1. location [ = \| ~ \| ~* \| ^~ ] uri {...}
   1) 概述
       a. 不含正则表达式uri称为“标准uri”，使用正则表达式的uri称为“正则uri”.
       b. 不添加选项时，首先在server块的多个location块中搜索是否有标准uri
          和请求字符串匹配，如果有多个可以匹配，就记录匹配度最高的一个。然后，
          服务器再用location块中的正则uri和请求字符串匹配，当第一个正则uri
          匹配成功，结束搜索，并使用这个location块处理此请求；
          如果正则匹配全部失败，就使用刚才记录的匹配度最高的location块处理此请求。
   2) 选项解释
      a. “=”，用于标准uri前，要求请求字符串与uri严格匹配。如果已经匹配成功，就停止继续向下搜索
         并立即处理此请求。
      b. “～”，用于表示uri包含正则表达式，并且区分大小写。
      c. “～*”，用于表示uri包含正则表达式，并且不区分大小写。
      d. “^～”，用于标准uri前，要求nginx服务器找到标识uri和请求字符串匹配度最高的location后,
         立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配

2. rewrite regex replacement [flag]
     1) 概述
        replacement, 匹配成功后用于替换URI中被戳取内容的字符串。默认情况下，如果该字符串是
        由“http://”或者“https://”开头的，则不会继续向下对URI进行其他处理，而直接将重写后
        的URI返回给客户端

     2) flag选项解释
        a. last，终止继续在本location块重处理接收到的URI，并将此处重写的URI作为一个新的URI，
           使用各location块进行处理。该标志将重写后的URI重新在server块中执行，为重写后的URI
           提供了转入到其他location块的机会。
        b. break，将此处重写的URI作为一个新的URI，在本块中继续进行处理。该标志将重写后的地址
           在当前的location块中执行，不会将新的URI转向到其他location块。
        c. redirect，将重写后的URI返回给客户端，状态代码为302,指明是临时重定向URI，主要用
           在replacement变量不是以“http://”或者"https://"开头的情况下
        d. permanent，将重写后的URI返回给客户端，状态代码为301,指明是永久重定向URI

3. proxy_pass
     1) 概述
        note:proxy_pass会用新的URI替代旧的URI，尤其需要注意‘\’（根目录会自动添加\）

4. 缓存
     1) 概述
        a. 基于proxy_store缓存机制，通常将proxy_store的缓存目录配置到/dev/shm中提高缓存
           数据的处理速度。note：只能缓存200状态下的响应数据，不支持动态链接请求，即会忽略get请求的参数
        b. 基于memcached缓存机制
        c. proxy_cached缓存机制
        d. 基于第三方模块ncache的缓存机制

5. nginx与Squid组合
     1) 概述
        Squid Cache（简称Squid）是目前在大访问量的网站建设中应用非常广泛的Web缓存服务器。
        但是，Squid服务本身不支持在单台服务器同一端口下运行多个进程（列入要反向代理Web必须指定端口80）

6. nginx服务器的邮件服务

7. nginx源码结构
     1) 目录结构
        * core
          该目录中存放了nginx使用到的关键数据结构和nginx内核实现的源码，目录中大致的重要文件如下：
          a. nginx.* 文件，包含了nginx程序入口函数main()的文件，实现了对nginx各模块的整体控制
          b. ngx_connection.* 文件，实现与网络连接管理相关的功能
          c. ngx_inet.* 文件，实现与socket网络套接字相关的功能
          d. ngx_cycle.* 文件，实现对系统整个运行过程中参数、资源的统一管理和调配
          e. ngx_log.* 文件，实现日志输出、管理的相关功能
          f. ngx_file.* 文件，实现文件读写相关功能
          g. ngx_regex.* 实现nginx服务器对正则表达式的支持
          h. ngx_string.* 实现对字符串处理的基本功能
          i. ngx_times.* 实现对时间的获取和处理操作

          该目录中还包含了一些重要数据结构的定义和操作，比如ngx_array.*, ngx_list.*, 
          ngx_queue.*, ngx_hash.*, ngx_*tree.*, ngx_output_chain.*;
          一些与内存管理相关的源码，比如ngx_palloc.*, ngx_shmtx.*, ngx_open_file_cache.*等

        * event
          该目录实现了nginx服务器的事件驱动模型，实现了nginx的消息机制

        * http
          该目录提供了对web服务主要的支持

        * mail
          mail目录存放了实现nginx服务器邮件服务的源码

        * misc
          ngx_cpp_test_module.cpp:文件实现的功能是测试程序中引用的头文件是否与C＋＋兼容
          ngx_google_perftools_module.c:文件用来支持Google PerfTools的使用，
          Google Perftools包含四个工具，用于优化内存分配的效率和速度，帮助高并发的
          情况下很好地控制内存的使用

        * os
          默认只包含一个unix目录，其中存放的源代码是针对“类Unix系统”，如Solaris、FreeBSD
          等的特殊情况，进行实现

~~~

~~~
8. nginx内置变量
~~~

| name                         |explain|
|:------------------------------|:--------------------------------------------------------------------------------------|
| $arg_PARAMETER               | 客户端GET请求中PARAMETER字段的值                                                     |
| $args                        | 客户端请求中的参数                                                                   |
| $binary_remote_addr          | 远程地址的二进制表示                                                                 |
| $body_bytes_sent             | 已发送的消息体字节数                                                                 |
| $content_length              | HTTP请求信息里的Content－Length                                                      |
| $content_type                | 请求信息里的Content－Type                                                            |
| $cookie_COOKIE               | 客户端请求中COOKIE头域的值                                                           |
| $document_root               | 针对当前请求的根路径设置值                                                           |
| $document_uri                | 与$uri相同                                                                           |
| $host                        | 请求信息中的Host头域值，如果请求中没有Host行，则等于设置的服务器名                   |
| $http_HEADER                 | HTTP请求信息里的HEARER字段                                                           |
| $http_host                   | 与$host相同，但如果请求信息中没有Host行，则可能不同                                  |
| $http_cookie                 | 客户端的cookie信息                                                                   |
| $http_referer                | 引用地址                                                                             |
| $http_user_agent             | 客户端代理信息                                                                       |
| $http_via                    | 最后一个访问服务器的IP地址                                                           |
| $http_x_forwarded_for        | 相当于网络访问路径                                                                   |
| $is_args                     | 如果$args有值，则等于“？”；否则等于空                                                |
| $limit_rate                  | 对连接速率的限制                                                                     |
| $nginx_version               | 当前nginx服务器的版本                                                                |
| $pid                         | 当前nginx服务器主进程的进程ID                                                        |
| $query_string                | 与$args相同                                                                          |
| $remote_addr                 | 客户端IP地址                                                                         |
| $remote_port                 | 客户端端口号                                                                         |
| $remote_user                 | 客户端用户名，用于Auth Basic Module验证                                              |
| $request                     | 客户端请求                                                                           |
| $request_body                | 客户端请求的报文体                                                                   |
| $request_body_file           | 发往后端服务器的本地临时缓存文件的名称                                               |
| $request_filename            | 当前请求的文件路径名，由root或alias指令与URI请求生成                                 |
| $request_method              | 请求的方法，比如GET，POST                                                            |
| $request_uri                 | 请求的URI，带参数，不包含主机名                                                      |
| $scheme                      | 所用的协议，如http或者https                                                          |
| $sent_http_cache_control     |                                                                                      |
| $sent_http_connection        |                                                                                      |
| $sent_http_content_length    |                                                                                      |
| $sent_http_content_type      |                                                                                      |
| $sent_http_keep_alive        |                                                                                      |
| $sent_http_last_modified     |                                                                                      |
| $sent_http_location          |                                                                                      |
| $sent_http_transfer_encoding |                                                                                      |
| $server_addr                 | 服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址 |
| $server_name                 | 请求到达的服务器名                                                                   |
| $server_port                 | 请求到达的服务器端口号                                                               |
| $server_protocol             | 请求的协议版本，HTTP/1.0或者HTTP/1.1                                                 |
| $uri                         | 请求的不带请求参数的URI，可能和最初的值有不同，比如经过重定向之类的                  |


### 编译第三方模块
* config文件的写法
    a.HTTP模块，那么config文件中需要定义以下3个变量:
      ngx_addon_name:仅在configure执行时使用，一般设置为模块名称
      HTTP_MODULES  :保存所有的HTTP模块名称，每个HTTP模块间由空格符相连。在重新设置HTTP_MODULES变量时，
      不要直接覆盖它，因为configure调用到自定义的config脚本前，已经将各个HTTP模块设置到
      HTTP_MODULES变量中了，因此，要像如下这样设置:"$HTTP_MODULES ngx_http_mytest_module"
      NGX_ADDON_SRCS:用于指定新增模块的源代码，多个待编译的源代码间以空格符相连。注意，在设置NGX_ADDON_SRCS
      时可以使用$ngx_addon_dir变量，它等价于configure执行时--add-module=PATH的参数

        例子：
        ngx_addon_name=ngx_http_mytest_module
        HTTP_MODULES="$HTTP_MODULES ngx_http_mytest_module"
        NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_mytest_module.c"

        除了上述三个变量名，还有许多变量可以设置，HTTP_FILTER_MODULES:HTTP过滤模块，包括$CORE_MODULES,
        $EVENT_MODULES,$HTTP_MODULES,$HTTP_FILTER_MODULES,$HTTP_HEADERS_FILTER_MODULE等模块变
        量都可以重定义，它们分别对应着nginx的核心模块，事件模块，HTTP模块，HTTP过滤模块，HTTP头部过滤模块。
        除了NGX_ADDON_SRCS变量，还有一个变量经常用到：$NGX_ADDON_DEPS变量，它指定了模块依赖的路径，同样
        可在config中设置

<br/>
<br/>
<br/>

### 附录

[^1]:深入理解Nginx模块开发与架构解析(第2版) 陶辉 著
