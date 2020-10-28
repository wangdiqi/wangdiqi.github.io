---
layout: post
title: "freeswitch"
subtitle: "VoIP"
date: 2020-04-17 15:29:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - freeswitch
---

> "Let's go"

## 安装

库
libsqlite3-dev
libcurl4-openssl-dev
libspeexdsp-dev
libldns-dev
libedit-dev
libavformat-dev


libav项目


命令
yasm


## 基本概念

### 工作流程

Channel状态机

~~~
NEW -->INIT-->ROUTING-->EXECUTE-->HANGUP-->REPORTING-->DESTROY
               ^              |
               |<--Transfer<--
~~~

当新建(NEW)一个Channel时，会进行初始化(INIT)，进入路由(ROUTING)阶段，也就是查找解析Dialplan阶段；  
找到合适路由后执行(EXECUTE)一系列动作，最后无论哪一方挂机，都会进入挂机(HANGUP)阶段。后面的  
报告(REPORTING)阶段一般用于进行统计、计费等，最后将Channel销毁(DESTROY)，释放系统资源。



### 常用命令

~~~
呼叫1000这个用户后，便执行echo这个程序
originate user/1000 &echo
~~~

~~~
列出某个profile的状态
sofia status profile internal

查看注册用户
sofia status profile internal reg

过滤某些符合条件的用户
sofia status profile internal reg 1000

列出某个特定用户
sofia status profile internal user 1000

列出网关状态
sofia status gateway gw
~~~


## 模块开发

~~~
初始化时加载模块
struct switch_loadable_module_container {
	switch_hash_t *module_hash;
	switch_hash_t *endpoint_hash;   //Endpoint模块，如mod_sofia
	switch_hash_t *codec_hash;
	switch_hash_t *dialplan_hash;
	switch_hash_t *timer_hash;
	switch_hash_t *application_hash;
	switch_hash_t *chat_application_hash;
	switch_hash_t *api_hash;
	switch_hash_t *json_api_hash;
	switch_hash_t *file_hash;
	switch_hash_t *speech_hash;
	switch_hash_t *asr_hash;
	switch_hash_t *directory_hash;
	switch_hash_t *chat_hash;
	switch_hash_t *say_hash;
	switch_hash_t *management_hash;
	switch_hash_t *limit_hash;
	switch_hash_t *secondary_recover_hash;
	switch_mutex_t *mutex;
	switch_memory_pool_t *pool;
};

被加载的模块中实现这些函数；不实现就是NULL
typedef struct switch_loadable_module_function_table {
	int switch_api_version;
	switch_module_load_t load;
	switch_module_shutdown_t shutdown;
	switch_module_runtime_t runtime;
	switch_module_flag_t flags;
} switch_loadable_module_function_table_t;


struct switch_loadable_module {
	char *key;
	char *filename;
	int perm;
	switch_loadable_module_interface_t *module_interface;
	switch_dso_lib_t lib;
	switch_module_load_t switch_module_load;
	switch_module_runtime_t switch_module_runtime;
	switch_module_shutdown_t switch_module_shutdown;
	switch_memory_pool_t *pool;
	switch_status_t status;
	switch_thread_t *thread;
	switch_bool_t shutting_down;
};

struct switch_core_session {
	switch_memory_pool_t *pool;
	switch_thread_t *thread;
	switch_thread_id_t thread_id;
	switch_endpoint_interface_t *endpoint_interface;
	switch_size_t id;
	switch_session_flag_t flags;
	switch_channel_t *channel;

	switch_io_event_hooks_t event_hooks;
	switch_codec_t *read_codec;
	switch_codec_t *real_read_codec;
	switch_codec_t *write_codec;
	switch_codec_t *real_write_codec;
	switch_codec_t *video_read_codec;
	switch_codec_t *video_write_codec;

	switch_codec_implementation_t read_impl;
	switch_codec_implementation_t real_read_impl;
	switch_codec_implementation_t write_impl;
	switch_codec_implementation_t video_read_impl;
	switch_codec_implementation_t video_write_impl;

	switch_audio_resampler_t *read_resampler;
	switch_audio_resampler_t *write_resampler;

	switch_mutex_t *mutex;
	switch_mutex_t *stack_count_mutex;
	switch_mutex_t *resample_mutex;
	switch_mutex_t *codec_read_mutex;
	switch_mutex_t *codec_write_mutex;
	switch_mutex_t *video_codec_read_mutex;
	switch_mutex_t *video_codec_write_mutex;
	switch_thread_cond_t *cond;
	switch_mutex_t *frame_read_mutex;

	switch_thread_rwlock_t *rwlock;
	switch_thread_rwlock_t *io_rwlock;

	void *streams[SWITCH_MAX_STREAMS];
	int stream_count;

	char uuid_str[SWITCH_UUID_FORMATTED_LENGTH + 1];
	void *private_info[SWITCH_CORE_SESSION_MAX_PRIVATES];
	switch_queue_t *event_queue;
	switch_queue_t *message_queue;
	switch_queue_t *signal_data_queue;
	switch_queue_t *private_event_queue;
	switch_queue_t *private_event_queue_pri;
	switch_thread_rwlock_t *bug_rwlock;
	switch_media_bug_t *bugs;
	switch_app_log_t *app_log;
	uint32_t stack_count;

	switch_buffer_t *raw_write_buffer;
	switch_frame_t raw_write_frame;
	switch_frame_t enc_write_frame;
	uint8_t raw_write_buf[SWITCH_RECOMMENDED_BUFFER_SIZE];
	uint8_t enc_write_buf[SWITCH_RECOMMENDED_BUFFER_SIZE];

	switch_buffer_t *raw_read_buffer;
	switch_frame_t raw_read_frame;
	switch_frame_t enc_read_frame;
	uint8_t raw_read_buf[SWITCH_RECOMMENDED_BUFFER_SIZE];
	uint8_t enc_read_buf[SWITCH_RECOMMENDED_BUFFER_SIZE];

	/* video frame.data being trated differently than audio, allocate a dynamic data buffer if necessary*/
	switch_buffer_t *video_raw_write_buffer;
	switch_frame_t video_raw_write_frame;
	// switch_frame_t video_enc_write_frame;

	switch_buffer_t *video_raw_read_buffer;
	switch_frame_t video_raw_read_frame;
	// switch_frame_t video_enc_read_frame;

	switch_codec_t bug_codec;
	uint32_t read_frame_count;
	uint32_t track_duration;
	uint32_t track_id;
	switch_log_level_t loglevel;
	uint32_t soft_lock;
	switch_ivr_dmachine_t *dmachine[2];
	plc_state_t *plc;

	switch_media_handle_t *media_handle;
	uint32_t decoder_errors;
	switch_core_video_thread_callback_func_t video_read_callback;
	void *video_read_user_data;
	switch_core_video_thread_callback_func_t text_read_callback;
	void *text_read_user_data;
	switch_io_routines_t *io_override;
	switch_slin_data_t *sdata;

	switch_buffer_t *text_buffer;
	switch_buffer_t *text_line_buffer;
	switch_mutex_t *text_mutex;
};

typedef enum {
	CS_NEW,                      // 新建
	CS_INIT,                     // 已初始化
	CS_ROUTING,                  // 路由
	CS_SOFT_EXECUTE,             // 准备好执行，可由第三方控制；
	CS_EXECUTE,                  // 执行dialplan中的app
	CS_EXCHANGE_MEDIA,           // 与另一个channel在交换媒体
	CS_PARK,                     // park，等待进一步的命令指示
	CS_CONSUME_MEDIA,            // 消费掉媒体并丢弃
	CS_HIBERNATE,                // 没事可干，sleep
	CS_RESET,                    // 重置
	CS_HANGUP,                   // 挂机，结束信令和媒体交互
	CS_REPORTING,                // 收集呼叫信息(如写CDR等)
	CS_DESTROY,                  // 待销毁，退出状态机
	CS_NONE                      // 无效
} switch_channel_state_t;

从session中获取channel
switch_core_session_get_channel(session)

创建session
switch_core_session_request_uuid(...)

大部分媒体处理逻辑都是在switch_ivr_*.c中实现的，其中多个源代码实现了不同的switch_ivr逻辑；
如switch_ivr_async.c进行异步处理，switch_ivr_bridge.c处理话路桥接等。

switch_core_io.c:
SWITCH_DECLARE(switch_status_t) switch_core_session_read_frame(switch_core_session_t *session,
                                                               switch_frame_t **frame,
                                                               switch_io_flag_t flags,
						                                       int stream_id)


switch_core_media.c:
SWITCH_DECLARE(switch_status_t) switch_core_session_write_frame(switch_core_session_t *session,
                                                                switch_frame_t *frame,
                                                                switch_io_flag_t flags,
																int stream_id)
~~~

