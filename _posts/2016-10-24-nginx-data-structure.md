---
layout: post
title: "Nginx 基本数据结构"
subtitle: ""
date: 2016-10-24 10:28:00
author: "seventhking"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - nginx
---

> "内容来自书本[^1]和网络"

### ngx_int_t
~~~
typedef intptr_t      ngx_int_t;
~~~

### ngx_uint_t
~~~
typedef uintptr_t     ngx_uint_t;
~~~

### ngx_str_t 字符串
~~~
typedef struct {
    //有效长度
    size_t     len;
    //字符串起始地址，并不是普通字符串，未必会以‘\0’作为结尾，必须配合len来使用
    u_char     *data;
}ngx_str_t;

比较ngx_str_t字符串，应当使用ngx_strncmp(),
#define ngx_strncmp(s1, s2, n)  strncmp((const char *)s1, (const char *)s2, n)
~~~

### ngx_list_t 链表
~~~
typedef struct ngx_list_part_s ngx_list_part_t;

struct ngx_list_part_s {
    //指向数组的起始地址
    void            *elts;
    //表示数组中已经使用了多少个元素，nelts必须小于ngx_list_t结构体中的nalloc
    ngx_uint_t      nelts;
    //下一个链表元素ngx_list_part_t的地址
    ngx_list_part_t *next;
};

typedef struct {
    //指向链表的最后一个数组元素
    ngx_list_part_t    *last;
    //链表的首个数组元素
    ngx_list_part_t    part;
    //限制ngx_list_part_s中每一个数组元素的占用空间大小，也就是用户要存储的一个数据所占用的字节数必须小于或等于size
    size_t             size;
    //链表的数组元素一旦分配后是不可更改的。nalloc表示每个ngx_list_part_t数组的容量，即最多可存储多少个数据
    ngx_uint_t         nalloc;
    //链表中管理内存分配的内存池对象。用户要存放的数据占用的内存都是由pool分配的
    ngx_pool_t         *pool;
} ngx_list_t;

ngx_list_t中的所有数据都是由ngx_pool_t类型的pool内存池分配的，它们通常都是连续的内存(在由一个pool内存池分配下)

ngx_list_t *ngx_list_create(ngx_pool_t *pool, ngx_uint_t n, size_t size);
size是每个元素的大小，n是每个链表数组可容纳元素的个数（相当于ngx_list_t结构中的nalloc成员）
static ngx_inline ngx_int_t ngx_list_init(ngx_list_t *list, ngx_pool_t *pool, ngx_uint_t n, size_t size);
void *ngx_list_push(ngx_list_t *list);
~~~

### ngx_table_elt_t
~~~
typedef struct {
    ngx_uint_t    hash;
    ngx_str_t     key;
    ngx_str_t     value;
    u_char        *lowcase_key;
} ngx_table_elt_t;                //key-value 键值对
~~~

### ngx_buf_t
~~~
typedef struct ngx_buf_s ngx_buf_t;
typedef void *           ngx_buf_tag_t;
struct ngx_buf_s {
    /*pos通常是用来告诉使用者本次应该从pos这个位置开始处理内存中的数据，这样设置是因为
    同一个ngx_buf_t可能被多次反复处理。当然，pos的含义是由使用它的模块定义的*/
    u_char            *pos;
    /*last通常表示有效的内容到此为止，注意，pos与last之间的内存是希望nginx处理的内容*/
    u_char            *last;
    /*处理文件时，file_pos与file_last的含义与处理内存时的pos与last相同，file_pos
    表示将要处理的文件位置，file_last表示截止的文件位置*/
    off_t             file_pos;
    off_t             file_last;
    /*如果ngx_buf_t缓冲区用于内存，那么start指向这段内存的起始地址*/
    u_char            *start;
    /*与start成员对应，指向缓冲区内存的末尾*/
    u_char            *end;
    /*表示当前缓冲区的类型，例如由哪个模块使用就指向这个模块ngx_module_t变量的地址*/
    ngx_buf_tag_t     tag;
    /*引用的文件*/
    ngx_file_t        *file;
    /*当前缓冲区的影子缓冲区，该成员很少用到，仅仅在使用缓冲区转发上游服务器的响应时
    才使用了shadow成员，这是因为nginx太节约内存，分配了一块内存并使用ngx_buf_t
    表示接收到的上游服务器响应后，在向下游客户端转发时可能会把这块内存存储到文件中，
    也可能直接向下游发送，此时nginx绝不会重新复制一份内存用于新的目的，而是再次建
    立一个ngx_buf_t结构体指向原内存，这样多个ngx_buf_t结构体指向了同一块内存，
    它们之间的关系就通过shadow成员来引用。这种设计过于复杂，通常不建议使用*/
    ngx_buf_t         *shadow;
    /*临时内存标志位，为1时表示数据在内存中且这段内存可以修改*/
    unsigned          temporary:1;
    /*标志位，为1时表示数据在内存中且这段内存不可以修改*/
    unsigned          memory:1;
    /*标志位，为1时表示这段内存是用mmap系统调用映射过来的，不可以被修改*/
    unsigned          mmap:1;
    /*标志位，为1时表示可回收*/
    unsigned          recycled:1;
    /*标志位，为1时表示这段缓冲区处理的时文件而不是内存*/
    unsigned          in_file:1;
    /*标志位，为1时表示需要执行flush操作*/
    unsigned          flush:1;
    /*标志位，对于操作这块缓冲区时是否使用同步方式，需谨慎考虑，这可能会阻塞nginx进程，
      nginx中所有操作几乎都是异步的，这是它支持高并发的关键。有些框架代码在sync为1时
      可能会有阻塞的方式进行I/O操作，它的意义视使用它的nginx模块而定*/
    unsigned          sync:1;
    /*标志位，表示是否是最后一块缓冲区，因为ngx_buf_t可以由ngx_chain_t链表串联起来，
      因此，当last_buf为1时，表示当前是最后一块待处理的缓冲区*/
    unsigned          last_buf:1;
    /*标志位，表示是否是ngx_chain_t中的最后一块缓冲区*/
    unsigned          last_in_chain:1;
    /*标志位，表示是否是最后一个影子缓冲区，与shadow域配合使用。通常不建议使用它*/
    unsigned          last_shadow:1;
    /*标志位，表示当前缓冲区是否属于临时文件*/
    unsigned          temp_file:1;
}
~~~

### ngx_chain_t
~~~
typedef struct ngx_chain_s      ngx_chain_t;
struct ngx_chain_s {
    ngx_buf_t    *buf;
    ngx_chain_t  *next;
}
在向用户发送HTTP包体时，就要传入ngx_chain_t链表对象，注意，如果是最后一个ngx_chain_t,
那么必须将next置为NULL，否则永远不会发送成功，而且这个请求将一直不会结束(nginx框架的要求)
~~~

### ngx_module_t
~~~
typedef struct ngx_module_s ngx_module_t;
struct ngx_module_s {
    /*下面的ctx_index, index, spare0, spare1, spare2, spare3, version变量不需
    要在定义时赋值，可以使用nginx准备好的的宏NGX_MODULE_V1来定义，它已经定义好7个值*/
    #define NGX_MODULE_V1        0, 0, 0, 0, 0, 0, 1
    /*对于一类模块(由下面的type成员决定类别)而言，ctx_index表示当前模块在这类模块中的序号。
      这个成员常常是由管理这类模块的一个nginx核心模块设置的，对于所有的HTTP模块而言，
      ctx_index是由核心模块ngx_http_module设置的。ctx_index非常重要，nginx模块
      化设计非常依赖于各个模块的顺序，它们既用于表达优先级，也用于表明每个模块的位置，
      借以帮助nginx框架快速获得某个模块的数据*/
    ngx_uint_t                   ctx_index;
    /*index表示当前模块在ngx_modules数组中的序号。注意，ctx_index表示的是当前模块
      在一类模块中的序号，而index表示当前模块在所有模块中的序号，它同样关键。nginx启
      动时会根据ngx_modules数组设置各模块的index值。例如：
      ngx_max_module = 0;
      for (i = 0; ngx_modules[i]; ++i)
      {
          ngx_modules[i]->index = ngx_max_module++;
      }*/
    ngx_uint_t                   index;
    /*spare系列的保留变量， 暂未使用*/
    ngx_uint_t                   spare0;
    ngx_uint_t                   spare1;
    ngx_uint_t                   spare2;
    ngx_uint_t                   spare3;
    /*模块的版本，便于将来的扩展。目前只有一种，默认为1*/
    ngx_uint_t                   version;
    /*ctx用于指向一类模块的上下文结构体，为什么需要ctx呢？因为nginx模块有许多种类，
      不同类模块之间的功能差别很大。例如，事件类型的模块主要处理I/O事件相关的功能，
      HTTP类型的模块主要处理HTTP应用层的功能。这样，每个模块都有了自己的特性，而
      ctx将会指向特定类型模块的公共接口。例如，在HTTP模块中，ctx需要指向
      ngx_http_module_t结构体*/
    void                         *ctx;
    /*commands将处理nginx.conf中的配置项*/
    ngx_command_t                *commands;
    /*type表示该模型的类型，它与ctx指针是紧密相关的。在官方nginx中，
      它的取值范围是以下5种：
      NGX_HTTP_MODULE, NGX_CORE_MODULE, NGX_CONF_MODULE,
      NGX_EVENT_MODULE, NGX_MAIL_MODULE，也可以自定义新模块类型*/
    ngx_uint_t                   type;
    /*在nginx的启动停止过程中，以下7个函数指针表示有7个执行点会分别调用这7种方法，
      对于任何一个方法，如果不需要nginx在某个时刻执行它，那么简单地将它设为NULL空
      指针即可*/
    /*虽然从字面上理解应当在master进程启动时回调init_master,但到目前为止，框架
      代码从来不会调用它，因此设为NULL*/
    ngx_int_t                    (*init_master)(ngx_log_t *log)
    /*init_module回调方法在初始化所有模块时被调用，在master/worker下，这个阶段
      将在启动worker子进程前完成*/
    ngx_int_t                    (*init_module)(ngx_cycle_t *cycle)
    /*init_process回调方法在正常服务前被调用。在master/worker下，多个worker子
      进程已经产生，在每个worker进程的初始化过程会调用所有模块的init_process函数*/
    ngx_int_t                    (*init_process)(ngx_cycle_t *cycle)
    /*由于nginx暂不支持多线程模式，所以init_thread在框架代码中从没有被调用郭，设为NULL*/
    ngx_int_t                    (*init_thread)(ngx_cyclet_t *cycle)
    /*同上*/
    void                         (*exit_thread)(ngx_cycle_t *cycle)
    /*exit_process回调方法在服务停止前调用，在master/worker模式下，worker进程会在退出
      前调用它*/
    void                         (*exit_process)(ngx_cycle_t *cycle)
    /*exit_master回调方法将在master进程退出前被调用*/
    void                         (*exit_master)(ngx_cycle_t *cycle)
    /*以下8个spare_hook变量也是保留字段，目前没有使用，但可用nginx提供的
      NGX_MODULE_V1_PADDING宏来填充。
      该宏定义:#define NGX_MODULE_V1_PADDING 0, 0, 0, 0, 0, 0, 0, 0*/
    uintptr_t                    spare_hook0;
    uintptr_t                    spare_hook1;
    uintptr_t                    spare_hook2;
    uintptr_t                    spare_hook3;
    uintptr_t                    spare_hook4;
    uintptr_t                    spare_hook5;
    uintptr_t                    spare_hook6;
    uintptr_t                    spare_hook7;
};
~~~

### ngx_http_module_t
~~~
typedef struct {
    /*解析配置文件前调用*/
    ngx_int_t (*preconfiguration)(ngx_conf_t *cf)
    /*完成配置文件的解析后调用*/
    ngx_int_t (*postconfiguration)(ngx_conf_t *cf)
    /*当需要创建数据结构用于存储main级别(直属于http{...}块的配置项)的全局配置项时，
      可以通过create_main_conf回调方法创建存储全局配置项的结构体*/
    void *(*create_main_conf)(ngx_conf_t *cf)
    /*常用于初始化main级别配置项*/
    char *(*init_main_conf)(ngx_conf_t *cf, void *conf)
    /*当需要创建数据结构用于存储srv级别(直属于虚拟主机server{...}块的配置项)的配
      置项时，可以通过实现create_srv_conf回调方法创建存储srv级别配置项的结构体*/
    void *(*create_srv_conf)(ngx_conf_t *cf)
    /*merge_srv_conf回调方法主要用于合并main级别和srv级别下的同名配置项*/
    char *(*merge_srv_conf)(ngx_conf_t *cf, void *prev, void *conf)
    /*当需要创建数据结构用于存储loc级别(直属于location{...}块的配置项)的配置项时，
      可以实现create_loc_conf回调方法*/
    void *(*create_loc_conf)(ngx_conf_t *cf)
    /*merge_loc_conf回调方法主要用于合并srv级别和loc级别下的同名配置项*/
    char *(*merge_loc_conf)(ngx_conf_t *cf, void *prev, void *conf)
} ngx_http_module_t;

可能调用的顺序：
1.create_main_conf,
2.create_srv_conf,
3.create_loc_conf,
4.preconfiguration,
5.init_main_conf,
6.merge_srv_conf,
7.merge_loc_conf,
8.postconfiguration
~~~

### ngx_command_t
~~~
typedef struct {
    /*配置项名称，如"gzip"*/
    ngx_str_t     name
    /*配置项类型，type将指定配置项可以出现的位置。例如，出现在server{}或
      location{}中，以及它可以携带的参数个数*/
    ngx_uint_t    type
    /*出现了name中指定的配置项后，将会调用set方法处理配置项的参数*/
    char          *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
    /*在配置文件中的偏移量*/
    ngx_uint_t    conf
    /*通常用于使用预设的解析方法解析配置项，这是配置模块的一个优秀设计。
      它需要与conf配合使用，在第4章中详细介绍*/
    ngx_uint_t    offset
    /*配置项读取后的处理方法，必须是ngx_conf_post_t结构的指针*/
    void          *post
} ngx_command_t;

ngx_null_command 是一个空的ngx_command_t,例如:
#define ngx_null_command {ngx_null_string, 0, NULL, 0, 0, NULL}
~~~

### enum ngx_http_phases
~~~
HTTP框架共定义了11个阶段(第三方HTTP模块只能介入其中的7个阶段处理请求)
typedef enum {
    /*在接收到完整的HTTP头部后处理的HTTP阶段*/
    NGX_HTTP_POST_READ_PHASE = 0,

    /*在还没有查询到URI匹配的location前，这时rewrite重写URL也作为一个独立的HTTP阶段*/
    NGX_HTTP_SERVER_REWRITE_PHASE,

    /*根据URI寻找匹配的location，这个阶段通常由ngx_http_core_module模块实现，不建议
      其他HTTP模块重新定义这一阶段的行为*/
    NGX_HTTP_FIND_CONFIG_PHASE,

    /*在NGX_HTTP_FIND_CONFIG_PHASE阶段之后重写URL的意义与
      NGX_HTTP_SERVER_REWRITE_PHASE阶段显然是不同的，
      因为这两者会导致查找到不同的location块(location是与URI进行匹配的)*/
    NGX_HTTP_REWRITE_PHSAE,

    /*这一阶段是用于在rewrite重写URL后重新调到NGX_HTTP_FIND_CONFIG_PHASE阶段，
      找到与新的URI匹配的location。所以，这一阶段是无法由第三方HTTP模块处理的，
      而仅由ngx_http_core_module模块g使用*/
    NGX_HTTP_POST_REWRITE_PHASE,

    /*处理NGX_HTTP_ACCESS_PHASE阶段前，HTTP模块可以介入的处理阶段*/
    NGX_HTTP_PREACCESS_PHASE,

    /*这个阶段用于让HTTP模块判断是否允许这个请求访问nginx服务器*/
    NGX_HTTP_ACCESS_PHASE,

    /*当NGX_HTTP_ACCESS_PHASE阶段中HTTP模块的handler处理方法返回不允许访问的
      错误码时(实际是NGX_HTTP_FORBIDDEN 或者NGX_HTTP_UNAUTHORIZED)，这个阶
      段将负责构造拒绝服务的用户响应。所以这个阶段实际上用于给NGX_HTTP_ACCESS_PHASE阶段收尾*/
    NGX_HTTP_POST_ACCESS_PHASE,

    /*这个阶段完全是为了try_files配置项而设立的。当HTTP请求访问静态文件资源时，
      try_files配置项可以使这个请求顺序地访问多个静态文件资源，如果某一次访问失败，
      则继续访问try_files中指定的下一个静态资源。另外，这个功能完全时在
      NGX_HTTP_TRY_FILES_PHASE阶段中实现的*/
    NGX_HTTP_TRY_FILES_PHASE,

    /*用于处理HTTP请求内容的阶段，这是大部分HTTP模块最喜欢介入的阶段*/
    NGX_HTTP_CONTENT_PHASE,

    /*处理完请求后记录日志的阶段。例如，ngx_http_log_module模块就在
      这个阶段中加入了一个handler处理方法，使得每个HTTP请求处理完毕后
      会记录access_log日志*/
    NGX_HTTP_LOG_PHASE
} ngx_http_phases;
~~~

### ngx_http_request_s

### ngx_http_headers_in_t
~~~
typedef struct {
    /*所有解析过的HTTP头部都在headers链表中*/
    ngx_list_t                headers;
    /*以下每个ngx_table_elt_t成员都是RFC2616规范中定义的HTTP头部，它们实际都指向
      headers链表中的相应成员。注意，当它们为NULL空指针时，表示没有解析到相应的
      HTTP头部*/
    ngx_table_elt_t           *host;
    ngx_table_elt_t           *connection;
    ngx_table_elt_t           *if_modified_since;
    ngx_table_elt_t           *if_unmodified_since;
    ngx_table_elt_t           *user_agent;
    ngx_table_elt_t           *referer;
    ngx_table_elt_t           *content_length;
    ngx_table_elt_t           *content_type;
    ngx_table_elt_t           *range;
    ngx_table_elt_t           *if_range;
    ngx_table_elt_t           *transfer_encoding;
    ngx_table_elt_t           *expect;
    ngx_table_elt_t           *accept_encoding;
    ngx_table_elt_t           *via;
    ngx_table_elt_t           *authorization;
    ngx_table_elt_t           *keep_alive;
    ngx_table_elt_t           *x_forwarded_for
    ngx_table_elt_t           *x_real_ip;
    ngx_table_elt_t           *accept;
    ngx_table_elt_t           *accept_language;
    ngx_table_elt_t           *depth;
    ngx_table_elt_t           *destination;
    ngx_table_elt_t           *overwrite;
    ngx_table_elt_t           *date;
    /*user和passwd是只有ngx_http_auth_basic_module才会用的成员，这里可以忽略*/
    ngx_str_t                 user;
    ngx_str_t                 passwd;
    /*cookie是以ngx_array_t数组存储的*/
    ngx_array_t               cookies;
    /*server名称*/
    ngx_str_t                 server;
    /*根据ngx_table_elt_t *content_length计算出的HTTP包体大小*/
    off_t                     content_length_n;
    time_t                    keep_alive_n;
    /*HTTP连接类型，它的取值范围是0， NGX_HTTP_CONNECTION_CLOSE 或者
      NGX_HTTP_CONNECTION_KEEP_ALIVE*/
    unsigned                  connection_type:2;
    /*以下7个标志位是HTTP框架根据浏览器传来的“useragent”头部，它们可用来p岸段浏览器的类型，
      值为1时表示是相应的浏览器发来的请求，值为0时则相反*/
    unsigned                  msie:1
    unsigned                  msie6:1
    unsigned                  opera:1
    unsigned                  gecko:1
    unsigned                  chromeL1
    unsigned                  safari:1
    unsigned                  konqueror:1
} ngx_http_headers_in_t;
~~~

### ngx_http_headers_out_t
~~~
typedef struct {
    /*待发送的HTTP头部链表,与headers_in中的headers成员类似*/
    ngx_list_t          headers;
    /*响应中的状态值，如200表示成功。如NGX_HTTP_OK*/
    ngx_uint_t          status;
    /*响应的状态行，“HTTP/1.1 201 CREATED”*/
    ngx_str_t           status_line;
    /*以下成员(包括ngx_table_elt_t)都是RFC1616规范中定义的HTTP头部，设置后，
      ngx_http_header_filter_module过滤模块可以把它们加到待发送的网络包中*/
    ngx_table_elt_t     *server;
    ngx_table_elt_t     *date;
    ngx_table_elt_t     *content_length;
    ngx_table_elt_t     *content_encoding;
    ngx_table_elt_t     *location;
    ngx_table_elt_t     *refresh;
    ngx_table_elt_t     *last_modified;
    ngx_table_elt_t     *content_range;
    ngx_table_elt_t     *accept_ranges;
    ngx_table_elt_t     *www_authenticate;
    ngx_table_elt_t     *expires;
    ngx_table_elt_t     *etag;

    ngx_str_t           *override_charset;
    /*可以调用ngx_http_set_content_type(r)方法帮助我们设置Content-Type头部，
      这个方法会根据URI中的文件扩展名并对应着mime.type来设置Content-Type值*/
    size_t              content_type_len;
    ngx_str_t           content_type;
    ngx_str_t           charset;
    u_char              *content_type_lowcase;
    ngx_uint_t          content_type_hash;

    ngx_array_t         cache_control;
    /*在这里指定过content_length_n后，不用再次到ngx_table_elt_t *content_length中设置响应长度*/
    off_t               content_length_n;
    time_t              date_time;
    time_t              last_modified_time;
} ngx_http_headers_out_t;
~~~

### ngx_conf_post_t
~~~
typedef struct {
    ngx_conf_post_handler_pt   post_handler;
} ngx_conf_post_t;

typedef char *(*ngx_conf_post_handler_pt) (ngx_conf_t *cf, void *data, void *conf)
~~~

### ngx_http_conf_ctx_t
~~~
typedef struct {
    /*指针数组，数组中的每个元素指向所有HTTP模块create_main_conf方法产生的结构体*/
    void **main_conf;
    /*指针数组，数组中的每个元素指向所有HTTP模块create_srv_conf方法产生的结构体*/
    void **srv_conf;
    /*指针数组，数组中的每个元素指向所有HTTP模块create_loc_conf方法产生的结构体*/
    void **loc_conf;
} ngx_http_conf_ctx_t;
~~~

### ngx_http_upstream_s
~~~
typedef struct {
    ...

    /*request_bufs决定发送什么样的请求给上游服务器，在实现create_request方法时需要设置它*/
    ngx_chain_t                   *request_bufs;
    /*upstream访问时的所有限制性参数*/
    ngx_http_upstream_conf_t      *conf;
    /*通过resolved可以直接指定上游服务器地址*/
    ngx_http_upstream_resolved_t  *resolved;
    /*buffer成员存储接收自上游服务器发来的响应内容，由于它会被复用，所以具有下列多重意义:
      a)在使用process_header方法解析上游响应的包头时，buffer中将会保存完整的响应包头；
      b)当下面的buffering成员为1，而且此时upstream是向下游转发上游的包体时，buffer没有意义
      c)当buffering标志位为0时，buffer缓冲区会被用于反复地接收上游的包体，进而向下游转发
      d)当upstream并不用于转发上游包体时，buffer会被用于反复接收上游的包体，HTTP模块实现的
        input_filter方法需要关注它*/
    ngx_buf_t                     buffer;
    /*构造发往上游服务器的请求内容*/
    ngx_int_t                     (*create_request)(ngx_http_request_t *r);
    /*收到上游服务器的响应后就会回调process_header方法。如果process_header返回NGX_AGAIN，那么是在
      告诉upstream还没有收到完整的响应包头，此时，对于本次upstream请求来说，再次接收到上游服务器发来
      的TCP流时，还会调用process_header方法处理，直到process_header函数返回非NGX_AGAIN值这一阶段才会停止*/
    ngx_int_t                     (*process_header)(ngx_http_request_t *r);
    /*销毁upstream请求时调用*/
    void                          (*finalize_request)(ngx_http_request_t *r, ngx_int_t rc);
    /*5个可选的回调方法*/
    ngx_int_t                     (*input_filter_init)(void *data);
    ngx_int_t                     (*input_filter)(void *data, ssize_t bytes);
    ngx_int_t                     (*reinit_request)(ngx_http_request_t *r);
    void                          (*abort_request)(ngx_http_request_t *r);
    ngx_ini_t                     (*rewrite_redirect)(ngx_http_request_t *r, ngx_table_elt_t *h, size_t prefix);
    /*是否基于SSL协议访问上游服务器*/
    unsigned                      ssl:1;
    /*在向客户端转发上游服务器的包体时才有用。当buffering为1时，表示使用多个缓冲区以及磁盘文件来转发上
      游的响应包体。当nginx与上游间的网速远大于nginx与下游客户端间的网速时，让nginx开辟更多的内存
      甚至使用磁盘文件来缓存上游的响应包体，这是有意义的，它可以减轻上游服务器的并发压力。当buffering
      为0时，表示只使用上面的这一个buffer缓冲区来向下游转发响应包体*/
    unsigned                      buffering:1;

    ...
} ngx_http_upstream_t;
~~~

### ngx_http_upstream_conf_t
~~~
typedef struct {
    ...

    /*连接上游服务器的超时时间，单位毫秒*/
    ngx_msec_t        connect_timeout;
    /*发送TCP包到上游服务器的超时时间，单位毫秒*/
    ngx_msec_t        send_timeout;
    /*接收TCP包到上游服务器的超时时间，单位毫秒*/
    ngx_msec_t        read_timeout;

    ...
} ngx_http_upstream_conf_t;
~~~

### ngx_http_upstream_resolved_t
~~~
typedef struct {
    ...

    //地址个数
    ngx_uint_t         naddrs;
    //上游服务器的地址
    struct sockaddr    *sockaddr;
    socklen_t          socklen;

    ...
} ngx_http_upstream_resolved_t;
~~~

### ngx_http_status_t
~~~
typedef struct {
    ngx_uint_t          code;
    ngx_uint_t          count;
    u_char              *start;
    u_char              *end;
} ngx_http_status_t;
~~~
<br/>
<br/>
<br/>

### 附录

[^1]:深入理解Nginx模块开发与架构解析(第2版) 陶辉 著
