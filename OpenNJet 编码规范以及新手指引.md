# **OpenNJet 编码规范以及新手指引**



# 1.简介

本文主要介绍NJET项目编码规范以及新手指引，主要是帮助开发者规范自己的代码格式以及保持风格一致，并帮助新手开发者更快的上手。



# 2.基础指引

## 2.1代码布局

```C
.
  ├── 3rd_lib         依赖的第三方动态库, 比如libmosquitto_emb.so
  ├── build/rpm       rpm包编译脚本
  ├── auto            自动检测系统环境以及编译相关的脚本
  │   ├── cc          关于编译器相关的编译选项的检测脚本
  │   ├── lib         njet编译所需要的一些库的检测脚本
  │   ├── os          与平台相关的一些系统参数与系统调用相关的检测
  │   └── types       与数据类型相关的一些辅助脚本
  ├── conf            存放默认配置文件，在make install后，会拷贝到安装目录中去
  ├── contrib         存放一些实用工具，如geo配置生成工具（geo2njet.pl）
  ├── html            存放默认的网页文件，在make install后，会拷贝到安装目录中去
  ├── repos           存放yum数据源
  ├── doc             njet的api文档
  │   ├── swagger     openapi 接口网页文档
  │   ├── gui         前端展示页面文档
  │   └── manual      njet文档手册
  ├── luajit          luajit
  ├── lualib          lualib
  ├── modules         njet动态模块以及util模块
  ├── openapi         openapi 定义文件
  └── src             存放njet的源代码
      ├── core        njet的核心源代码，包括常用数据结构的定义，以及njet初始化运行的核心代码如main函数
      ├── event       对系统事件处理机制的封装，以及定时器的实现相关代码
      │   └── modules 不同事件处理方式的模块化，如select、poll、epoll、kqueue等
      ├── http        njet作为http服务器相关的代码
      │   └── modules 包含http的各种功能模块
      ├── ext/lua     lua模块
      ├── mail        njet作为邮件代理服务器相关的代码
      ├── stream      tcp/udp四层网络代理服务器相关的代码
      ├── misc        一些辅助代码，测试c++头的兼容性，以及对google_perftools的支持
      └── os          主要是对各种不同体系统结构所提供的系统函数的封装，对外提供统一的系统调用接口
```

## 2.2include文件

以下两个语句必须出现在 每个njet文件的开头：

> ```C++
> #include <njt_config.h>
> #include <njt_core.h>
> ```

除此之外，HTTP 代码还应包括

> ```C++
> #include <njt_http.h>
> ```

Mail 代码应包括

> ```C++
> #include <njt_mail.h> 
> ```

Stream 代码应包括

> ```C++
> #include <njt_stream.h>
> ```

## 2.3整数

整数通常使用 njt_int_t 和  njt_uint_t

## 2.4常见返回码 

njet 中的大多数函数返回以下代码：

- `NJT_OK`— 操作成功。
- `NJT_ERROR`— 操作失败。
- `NJT_AGAIN`— 操作未完成;再次调用该函数。
- `NJT_DECLINED`— 操作被拒绝。
- `NJT_BUSY`— 资源不可用。
- `NJT_DONE`— 操作完成或在其他地方继续进行。 也用作替代成功代码。
- `NJT_ABORT`— 功能中止。 也用作替代错误代码。

## 2.5错误处理

njt_errno 宏返回最后一个系统错误代码,在POSIX平台它映射到errno， 在Windows中通过调用GetLastError()获取。  `njt_socket_errno`宏返回最后一个套接字错误码。 与njt_errno宏一样，在POSIX平台上它映射到errno。 在Windows 上的它映射到WSAGetLastError() 。 在同一行中多次访问`njt_errno`或者`njt_socket_errno`可能会导致 性能问题。 如果错误值可能多次使用，请将其存储在njt_err_t类型的局部变量中。 若要设置错误码，请使用 njt_set_errno(errno)和`njt_set_socket_errno(errno)` 宏。

`njt_errno` and `njt_socket_errno`也可以传递给日志函数njt_log_error() 和njt_log_debugX()。

使用njt_errno示例

```C
njt_int_t
njt_my_kill(njt_pid_t pid, njt_log_t *log, int signo)
{
    njt_err_t  err;

    if (kill(pid, signo) == -1) {
        err = njt_errno;

        njt_log_error(NJT_LOG_ALERT, log, err, "kill(%P, %d) failed", pid, signo);

        if (err == NJT_ESRCH) {
            return 2;
        }

        return 1;
    }

    return 0;
}
```



# 3.编码规范

基本上，NJet所采用的是一种类似 BSD 的 C 代码风格，很规范、也很清晰。建议我们的 NJet 模块开发也采用 该编码风格

## 3.1模块目录命名

 modules目录下模块目录名采用njet开头，单词间用中划线(-)间隔

## 3.2一般规则

- 最大文本宽度为 80 个字符
- 缩进为 4 个空格
- 没有制表符，没有尾随空格
- 同一行上的列表元素用空格分隔
- 十六进制文本为小写
- 在块和函数之间空两行
- 文件名、函数和类型名称以及全局变量具有或更具体的前缀，例如 njt_或njt_http_或`njt_mail_`

> ```C
> size_t
> njt_utf8_length(u_char *p, size_t n)
> {
>     u_char  c, *last;
>     size_t  len;
> 
>     last = p + n;
> 
>     for (len = 0; p < last; len++) {
> 
>         c = p;
> 
>         if (c < 0x80) {
>             p++;
>             continue;
>         }
> 
>         if (njt_utf8_decode(&p, last - p) > 0x10ffff) {
>             / invalid UTF-8 */
>             return n;
>         }
>     }
> 
>     return len;
> }
> ```

## 3.3文件格式

典型的源文件可能包含以下部分，分隔为两个空行：

- 版权声明
- 包括
- 预处理器定义
- 类型定义
- 功能原型
- 变量定义
- 函数定义

版权声明如下所示：在文件开头，签名空一行，后面空两行，如

```C
/*
 * Copyright (C) Igor Sysoev
 * Copyright (C) njet, Inc.
 * Copyright (C) TMLake, Inc.
 */
```

如果文件被大幅修改，则应更新作者列表， 新作者将添加到底部。

`njt_config.h`和`njt_core.h`  始终首先包含，然后是 `njt_http.h`, `njt_stream.h` 或 `njt_mail.h` 之一。 然后遵循可选的外部头文件：

> ```C
> #include <njt_config.h>
> #include <njt_core.h>
> #include <njt_http.h>
> 
> #include <libxml/parser.h>
> #include <libxml/tree.h>
> #include <libxslt/xslt.h>
> 
> #if (NJT_HAVE_EXSLT)
> #include <libexslt/exslt.h>
> #endif
> ```

头文件应包括所谓的“头保护”：

```C
#ifndef _NJT_PROCESS_CYCLE_H_INCLUDED_
#define _NJT_PROCESS_CYCLE_H_INCLUDED_
...
#endif /* _NJT_PROCESS_CYCLE_H_INCLUDED_ */
```

## 3.4注释方式

### 3.4.1多行注释

​      采用C 风格的注释，如：

```Bash
    /*
     * gcc before 3.3 compiles the broken code for
     *     if (r->uri_changes-- == 0)
     * if the r->uri_changes is defined as
     *     unsigned  uri_changes:4
     */
```

### 3.4.2单行注释

​      可使用     /* comment */

​      也可使用  // comment

#### 3.4.3函数注释

​      需要说明函数的功能

​      参数说明

​      返回值说明

```C
/** 
 * @brief  get array or object element's size 
 *   
 * @param datas           intput array datas 
 * @param contain_key     true if element has key, else false 
 * @return int64_t        return all sub element's size 
 */
 int64_t njt_calc_array_size(njt_queue_t *datas, bool contain_key);
```



## 3.5预处理

宏名称应该以前缀`njt_`或NJT`_`开头。 常量的宏名称为大写。 参数化宏和用于初始值设定的宏为小写。 宏名称和值至少用两个空格分隔：

> ```C
> #define NJT_CONF_BUFFER  4096
> 
> #define njt_buf_in_memory(b)  (b->temporary || b->memory || b->mmap)
> 
> #define njt_buf_size(b)                                                      \
>     (njt_buf_in_memory(b) ? (off_t) (b->last - b->pos):                      \
>                             (b->file_last - b->file_pos))
> 
> #define njt_null_string  { 0, NULL }
> ```

条件在括号内，否定在括号外：

> ```C
> #if (NJT_HAVE_KQUEUE)
> ...
> #elif ((NJT_HAVE_DEVPOLL && !(NJT_TEST_BUILD_DEVPOLL)) \
>        || (NJT_HAVE_EVENTPORT && !(NJT_TEST_BUILD_EVENTPORT)))
> ...
> #elif (NJT_HAVE_EPOLL && !(NJT_TEST_BUILD_EPOLL))
> ...
> #elif (NJT_HAVE_POLL)
> ...
> #else /* select */
> ...
> #endif /* NJT_HAVE_KQUEUE */
> ```

## 3.6 类型定义

类型名称以 “_t” 后缀结尾。 定义的类型名称至少由两个空格分隔：

> typedef njt_uint_t  njt_rbtree_key_t;

结构类型是使用typedef定义。 在结构内部，成员类型和名称是对齐的：

> ```C
> typedef struct {
>     size_t      len;
>     u_char     *data;
> } njt_str_t;
> ```

保持文件中不同结构之间的对齐方式相同。 指向自身结构的名称以 "_s"结尾

相邻的结构定义用两个空行分隔：

> ```C
> typedef struct njt_list_part_s  njt_list_part_t;
> 
> struct njt_list_part_s {
>     void             *elts;
>     njt_uint_t        nelts;
>     njt_list_part_t  *next;
> };
> 
> 
> typedef struct {
>     njt_list_part_t  *last;
>     njt_list_part_t   part;
>     size_t            size;
>     njt_uint_t        nalloc;
>     njt_pool_t       *pool;
> } njt_list_t;
> ```

每个结构成员都在其自己的行上声明：

> ```C
> typedef struct {
>     njt_uint_t        hash;
>     njt_str_t         key;
>     njt_str_t         value;
>     u_char           *lowcase_key;
> } njt_table_elt_t;
> ```

结构中的函数指针定义以“`_pt`”结尾：

> ```C
> typedef ssize_t (*njt_recv_pt)(njt_connection_t *c, u_char *buf, size_t size);
> typedef ssize_t (*njt_recv_chain_pt)(njt_connection_t *c, njt_chain_t *in,
>     off_t limit);
> typedef ssize_t (*njt_send_pt)(njt_connection_t *c, u_char *buf, size_t size);
> typedef njt_chain_t *(*njt_send_chain_pt)(njt_connection_t *c, njt_chain_t *in,
>     off_t limit);
> 
> typedef struct {
>     njt_recv_pt        recv;
>     njt_recv_chain_pt  recv_chain;
>     njt_recv_pt        udp_recv;
>     njt_send_pt        send;
>     njt_send_pt        udp_send;
>     njt_send_chain_pt  udp_send_chain;
>     njt_send_chain_pt  send_chain;
>     njt_uint_t         flags;
> } njt_os_io_t;
> ```

枚举类型具有以 “`_e`” 结尾：

> ```C
> typedef enum {
>     njt_http_fastcgi_st_version = 0,
>     njt_http_fastcgi_st_type,
>     ...
>     njt_http_fastcgi_st_padding
> } njt_http_fastcgi_state_e;
> ```

## 3.7变量定义

变量按基本类型的长度排序，然后按字母顺序声明。 类型名称和变量名称对齐。 类型和名称用两个空格分隔。 大数组放在声明块的末尾：

> ```C
> u_char                      |  | *rv, *p;
> njt_conf_t                  |  | *cf;
> njt_uint_t                  |  |  i, j, k;
> unsigned int                |  |  len;
> struct sockaddr             |  | *sa;
> const unsigned char         |  | *data;
> njt_peer_connection_t       |  | *pc;
> njt_http_core_srv_conf_t    |  |**cscfp;
> njt_http_upstream_srv_conf_t|  | *us, *uscf;
> u_char                      |  |  text[NJT_SOCKADDR_STRLEN];
> ```

静态变量和全局变量可以在声明时初始化：

```C
static njt_str_t  njt_http_memcached_key = njt_string("memcached_key");

static njt_uint_t  mday[] = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };

static uint32_t  njt_crc32_table16[] = {
    0x00000000, 0x1db71064, 0x3b6e20c8, 0x26d930ac,
    ...
    0x9b64c2b0, 0x86d3d2d4, 0xa00ae278, 0xbdbdf21c
};
```

一些常用的类型/名称组合：

> ```C
> u_char                        *rv;
> njt_int_t                      rc;
> njt_conf_t                    *cf;
> njt_connection_t              *c;
> njt_http_request_t            *r;
> njt_peer_connection_t         *pc;
> njt_http_upstream_srv_conf_t  *us, *uscf;
> ```

## 3.8函数定义

所有函数（包括静态函数）都应该有原型。 原型包括参数名称。 长原型在连续线上采用缩进：

> ```C
> static char *njt_http_block(njt_conf_t *cf, njt_command_t *cmd, void *conf);
> static njt_int_t njt_http_init_phases(njt_conf_t *cf,
>     njt_http_core_main_conf_t *cmcf);
> 
> static char *njt_http_merge_servers(njt_conf_t *cf,
>     njt_http_core_main_conf_t *cmcf, njt_http_module_t *module,
>     njt_uint_t ctx_index);
> ```

定义中的函数名称以换行符开头。 功能体的开始左大括号和结尾右大括号位于不同的行上。 函数的主体缩进。 函数之间有两个空行：

> ```C
> static njt_int_t
> njt_http_find_virtual_server(njt_http_request_t *r, u_char *host, size_t len)
> {
>     ...
> }
> 
> 
> static njt_int_t
> njt_http_add_addresses(njt_conf_t *cf, njt_http_core_srv_conf_t *cscf,
>     njt_http_conf_port_t *port, njt_http_listen_opt_t *lsopt)
> {
>     ...
> }
> ```

函数名称和左括号后没有空格。 长函数调用进行换行，位置从第一个函数参数的位置开始。 如果做不到（比如很长），请格式化第一个延续行，使其结束于第79个字符的位置：

> ```C
> njt_log_debug2(NJT_LOG_DEBUG_HTTP, r->connection->log, 0,
>                "http header: \"%V: %V\"",
>                &h->key, &h->value);
> 
> hc->busy = njt_palloc(r->connection->pool,
>                   cscf->large_client_header_buffers.num * sizeof(njt_buf_t *));
> ```

应使用njt_inline宏代替 inline 宏：

> ```C
> static njt_inline void njt_cpuid(uint32_t i, uint32_t *buf);
> ```

## 3.9表达式定义

除 “.” 和 “`−>`” 之外的二进制运算符应与其操作数隔一个空格。 一元运算符和下标与其操作数之间没有空格分隔：

```C
width = width * 10 + (*fmt++ - '0');

ch = (u_char) ((decoded << 4) + (ch - '0'));

r->exten.data = &r->uri.data[i + 1];
```

类型强制转换与强制转换表达式之间用一个空格分隔。 类型转换内部的星号用空格与类型名称分隔：

> ```C
> len = njt_sock_ntop((struct sockaddr *) sin6, p, len, 1)
> ```

如果表达式不适合单行，则将其换行。 换行的首选点是二元运算符。 延续行与表达式的开头对齐：

> ```C
> if (status == NJT_HTTP_MOVED_PERMANENTLY
>     || status == NJT_HTTP_MOVED_TEMPORARILY
>     || status == NJT_HTTP_SEE_OTHER
>     || status == NJT_HTTP_TEMPORARY_REDIRECT
>     || status == NJT_HTTP_PERMANENT_REDIRECT)
> {
>     ...
> }
> 
> p->temp_file->warn = "an upstream response is buffered "
>                      "to a temporary file";
> ```

作为保底的方式，要使延续线在位置 第79个字符处 结束：

> ```C
> hinit->hash = njt_pcalloc(hinit->pool, sizeof(njt_hash_wildcard_t)
>                                      + size * sizeof(njt_hash_elt_t *));
> ```

上述规则也适用于子表达式， 其中每个子表达式都有自己的缩进级别：

> ```C
> if (((u->conf->cache_use_stale & NJT_HTTP_UPSTREAM_FT_UPDATING)
>      || c->stale_updating) && !r->background
>     && u->conf->cache_background_update)
> {
>     ...
> }
> ```

指针要显式的与`NULL`比较（而不是0）：

> ```C
> if (ptr != NULL) {
>     ...
> }
> ```

## 3.10条件表达式与循环定义

“if” 关键字与条件之间用一个空格。 左大括号位于同一行上，或位于最后一个行上（如果条件需要多行）。 右大括号使用专用的一行，一般在 “`else if` / `else`” 后。 通常，在“`else if` / `else`” 前面保留一个空行

> ```C
> if (node->left == sentinel) {
>     temp = node->right;
>     subst = node;
> 
> } else if (node->right == sentinel) {
>     temp = node->left;
>     subst = node;
> 
> } else {
>     subst = njt_rbtree_min(node->right, sentinel);
> 
>     if (subst->left != sentinel) {
>         temp = subst->left;
> 
>     } else {
>         temp = subst->right;
>     }
> }
> ```

类似的格式规则应用于 “do” 和 “while” 循环：

> ```C
> while (p < last && *p == ' ') {
>     p++;
> }
> 
> do {
>     ctx->node = rn;
>     ctx = ctx->next;
> } while (ctx);
> ```

“switch” 关键字与条件之间间隔一个空格。 左大括号位于同一行上。 右大括号使用专门的一行。 “case” 关键字与"switch"对齐:

> ```C
> switch (ch) {
> case '!':
>     looked = 2;
>     state = ssi_comment0_state;
>     break;
> 
> case '<':
>     copy_end = p;
>     break;
> 
> default:
>     copy_end = p;
>     looked = 0;
>     state = ssi_start_state;
>     break;
> }
> ```

大多数 “for” 循环的格式如下：

> ```C
> for (i = 0; i < ccf->env.nelts; i++) {
>     ...
> }
> 
> for (q = njt_queue_head(locations);
>      q != njt_queue_sentinel(locations);
>      q = njt_queue_next(q))
> {
>     ...
> }
> ```

如果省略了 “for” 语句的某些部分， 请用 “`/* void */`” 注释指示：

> ```C
> for (i = 0; /* void */ ; i++) {
>     ...
> }
> ```

具有空循环体的循环也由 “`/* void */`”注释可以放在同一行：

> ```C
> for (cl = *busy; cl->next; cl = cl->next) { /* void */ }
> ```

无限循环如下所示：

> ```C
> for ( ;; ) {
>     ...
> }
> ```

## 3.11标签label的使用

标签用空行包围，并在上一级缩进：

```C
    if (i == 0) {
        u->err = "host not found";
        goto failed;
    }

    u->addrs = njt_pcalloc(pool, i * sizeof(njt_addr_t));
    if (u->addrs == NULL) {
        goto failed;
    }

    u->naddrs = i;

    ...

    return NJT_OK;

failed:

    freeaddrinfo(res);
    return NJT_ERROR;
```



# 4.String类型使用规范

## 4.1概述

对于 C 字符串，njet 使用无符号字符类型指针`u_char *`。

njet 字符串类型定义如下：`njt_str_t`

> typedef struct {    size_t      len;    u_char     *data; } njt_str_t;

该`len` 字段保存字符串长度，data字段保存字符串数据。 保存在njt_str_t 中的字符串在len字节后可能是也可能不是以 null 结尾。 在大多数情况下，并不是以null结尾。 但是，在代码的某些部分（例如在解析配置时），已知对象njt_str_t是以 null 结尾，这简化了字符串的比较，以及使字符串更容易传递到系统调用函数。

njet 中的字符串操作定义在src/core/njt_string.h，其中一些适配标准 C 函数：

- `njt_strcmp()`
- `njt_strncmp()`
- `njt_strstr()`
- `njt_strlen()`
- `njt_strchr()`
- `njt_memcmp()`
- `njt_memset()`
- `njt_memcpy()`
- `njt_memmove()`

其他字符串函数是特定于njet的

- `njt_memzero()`— 用零填充内存。
- `njt_explicit_memzero()`— 与 njt_memzero()相同，但此调用永远不会被 编译器的死存储消除优化。 此功能可用于清除敏感数据，例如密码和密钥。
- `njt_cpymem()`— 与 `njt_memcpy()`执行相同的操作，但返回最终目标地址 这对于在一行中附加多个字符串很方便。
- `njt_movemem()`— 与 `njt_memmove()`执行相同的操作，但返回最终目标地址。
- `njt_strlchr()`— 在字符串中搜索字符， 由两个指针分隔。

以下函数执行大小写转换和比较：

- `njt_tolower()`
- `njt_toupper()`
- `njt_strlow()`
- `njt_strcasecmp()`
- `njt_strncasecmp()`

以下宏简化了字符串初始化：

- `njt_string(text)`— 使用C 字符串text文本去静态初始化`njt_str_t`类型
- `njt_null_string`— 空字符串去静态初始化`njt_str_t`类型
- `njt_str_set(str, text)`— 使用 C 字符串text去初始化`strnjt_str_t`类型字符串的str字段
- `njt_str_null(str)`— 使用空字符串初始化`strnjt_str_t`类型字符串的str字段

## 4.2格式化

以下格式化函数支持 njet 特定的类型：

- `njt_sprintf(buf, fmt, ...)`
- `njt_snprintf(buf, max, fmt, ...)`
- `njt_slprintf(buf, last, fmt, ...)`
- `njt_vslprintf(buf, last, fmt, args)`
- `njt_vsnprintf(buf, max, fmt, args)`

这些功能支持的格式选项的完整列表是在`src/core/njt_string.c`文件中。其中一些是：

- `%O`—`off_t`
- `%T`—`time_t`
- `%z`—`ssize_t`
- `%i`—`njt_int_t`
- `%p`—`void *`
- `%V`—`njt_str_t *`
- `%s`— （以空结尾）`u_char *`
- `%*s`—`size_t + u_char *`

可以在大多数类型前面加上前缀，使其无符号u。 要将输出转换为十六进制，请使用 X或 x

例如：

> ```C
> u_char      buf[NJT_INT_T_LEN];
> size_t      len;
> njt_uint_t  n;
> 
> /* set n here */
> 
> len = njt_sprintf(buf, "%ui", n) — buf;
> ```

## 4.3数值转换

在njet中实现了几个用于数字转换的函数。 前四个分别将给定长度的字符串转换为正整数 指示的类型。 它们在出错时返回`NJT_ERROR`

- `njt_atoi(line, n)`—`njt_int_t`
- `njt_atosz(line, n)`—`ssize_t`
- `njt_atoof(line, n)`—`off_t`
- `njt_atotm(line, n)`—`time_t`

还有两个额外的数字转换函数。 像前四个一样，它们在错误时返回`NJT_ERROR`

- `njt_atofp(line, n, point)`— 转换定点浮点数 给定长度到njt_int_t类型的正整数。 结果左移`point` 个十进制 位置。 数字的字符串表示形式应不超过 points个小数位数。 例如，njt_atofp("10.5", 4, 2)返回`1050`
- `njt_hextoi(line, n)`— 将十六进制表示形式的正整数转换为`njt_int_t`

## 4.4正则表达式

njet 中的正则表达式接口是一个包装器 [PCRE](http://www.pcre.org/) 库。 相应的头文件为`src/core/njt_regex.h`

要使用正则表达式进行字符串匹配，首先需要 编译，通常在配置阶段完成。 请注意，由于 PCRE 支持是可选的，因此使用该接口的所有代码都必须使用NJT`_PCRE`宏的保护：

> ```C
> #if (NJT_PCRE)
> njt_regex_t          *re;
> njt_regex_compile_t   rc;
> 
> u_char                errstr[NJT_MAX_CONF_ERRSTR];
> 
> njt_str_t  value = njt_string("message (\\d\\d\\d).*Codeword is '(?<cw>\\w+)'");
> 
> njt_memzero(&rc, sizeof(njt_regex_compile_t));
> 
> rc.pattern = value;
> rc.pool = cf->pool;
> rc.err.len = NJT_MAX_CONF_ERRSTR;
> rc.err.data = errstr;
> /* rc.options can be set to NJT_REGEX_CASELESS */
> 
> if (njt_regex_compile(&rc) != NJT_OK) {
>     njt_conf_log_error(NJT_LOG_EMERG, cf, 0, "%V", &rc.err);
>     return NJT_CONF_ERROR;
> }
> 
> re = rc.regex;
> #endif
> ```

编译成功后，`njt_regex_compile_t` 结构中的`captures`  和 `named_captures` 字段包含正则表达式所有的`captures`和`named_captures` 变量的数量。

编译的正则表达式可用于匹配字符串：

> ```C
> njt_int_t  n;
> int        captures[(1 + rc.captures) * 3];
> 
> njt_str_t input = njt_string("This is message 123. Codeword is 'foobar'.");
> 
> n = njt_regex_exec(re, &input, captures, (1 + rc.captures) * 3);
> if (n >= 0) {
>     /* string matches expression /
> 
> } else if (n == NJT_REGEX_NO_MATCHED) {
>     / no match was found /
> 
> } else {
>     / some error */
>     njt_log_error(NJT_LOG_ALERT, log, 0, njt_regex_exec_n " failed: %i", n);
> }
> ```

njt_regex_exec()的参数是被编译为正则表达式re ，这个字符串去匹配input(保存在找到的`captures` 的一个可选的整数数组，以及数组的size），数组的大小必须是 3 的倍数， 根据 [PCRE API](http://www.pcre.org/original/doc/html/pcreapi.html) 的要求。 在示例中，大小是根据捕获总数加上 一个用于匹配的字符串本身。

如果存在匹配项，则可以按如下方式访问捕获：

> ```C
> u_char     p;
> size_t      size;
> njt_str_t   name, value;
> 
> / all captures /
> for (i = 0; i < n  2; i += 2) {
>     value.data = input.data + captures[i];
>     value.len = captures[i + 1] — captures[i];
> }
> 
> /* accessing named captures /
> 
> size = rc.name_size;
> p = rc.names;
> 
> for (i = 0; i < rc.named_captures; i++, p += size) {
> 
>     / capture name /
>     name.data = &p[2];
>     name.len = njt_strlen(name.data);
> 
>     n = 2  ((p[0] << 8) + p[1]);
> 
>     /* captured value */
>     value.data = &input.data[captures[n]];
>     value.len = captures[n + 1] — captures[n];
> }
> ```

njt_regex_exec_array()函数接受njt_regex_elt_t类型元素的数组（这些元素只是常规编译的 具有关联名称的表达式）、要匹配的字符串和日志。 该函数将数组中的表达式应用于字符串，直到找到匹配项，要么不再留下表达式。 存在匹配项时，返回NJT_OK，否则返回`NJT_DECLINED` ，或者在出现错误的情况下返回`NJT_ERROR`。



# 5.Time 格式

njt_time_t结构代表三个独立类型 秒、毫秒和 GMT 偏移量

> ```C
> typedef struct {
>     time_t      sec;
>     njt_uint_t  msec;
>     njt_int_t   gmtoff;
> } njt_time_t;
> ```

njt_tm_t结构类似于 UNIX 平台上的struct tm， 和 Windows 上的SYSTEMTIME

要获取当前时间，通常只需访问其中一个 可用的全局变量，表示所需 格式。

可用的字符串表示形式包括：

- `njt_cached_err_log_time`— 用于错误日志条目：`"1970/09/28 12:00:00"`
- `njt_cached_http_log_time`— 在 HTTP 访问日志条目中使用：`"28/Sep/1970:12:00:00 +0600"`
- `njt_cached_syslog_time`— 在系统日志条目中使用：`"Sep 28 12:00:00"`
- `njt_cached_http_time`— 用于 HTTP 标头：`"Mon, 28 Sep 1970 06:00:00 GMT"`
- `njt_cached_http_log_iso8601`— ISO 8601 标准格式：`"1970-09-28T12:00:00+06:00"`

njt_time()和njt_timeofday()宏 以秒为单位返回当前时间值，是访问缓存时间值的首选方式。

要显式获取时间，请使用njt_gettimeofday() ， 更新其参数（指向struct timeval 的指针）。 当njet从系统返回到事件循环时，时间总是进行更新。 在信号处理程序上下文， 如果要及时更新时间，请调用 njt_time_update()， 或者njt_time_sigsafe_update。

以下函数转换`time_t` 为细分时间表示形式。 下面每组的第一个函数将time_t 转换为`njt_tm_t`，第二个函数（带有中缀）转换为：struct tm

- `njt_gmtime(), njt_libc_gmtime()`— 时间表示为 UTC
- `njt_localtime(), njt_libc_localtime()`— 表达时间 相对于当地时区

njt_http_time(buf, time)函数返回一个字符串，适合用在HTTP headers中（例如，"Mon, 28 Sep 1970 06:00:00 GMT"）。 njt_http_cookie_time(buf, time)返回一个字符串，适合用在HTTP cookies中(例如，"Thu, 31-Dec-37 23:55:55 GMT")



# 6.Containers 

## 6.1数组

njet数组类型`njt_array_t`定义如下

> ```C
> typedef struct {
>     void        *elts;
>     njt_uint_t   nelts;
>     size_t       size;
>     njt_uint_t   nalloc;
>     njt_pool_t  *pool;
> } njt_array_t;
> ```

数组的元素保存在elts字段。 nelts字段包含元素数。 `size` 字段包含单个元素的大小并，并在初始化数组时进行设置。

在一个pool上调用njt_array_create(pool, n, size) 创建数组，调用njt_array_init(array, pool, n, size)去初始化已分配的数组对象。

> ```C
> njt_array_t  a, b;
> 
> / create an array of strings with preallocated memory for 10 elements /
> a = njt_array_create(pool, 10, sizeof(njt_str_t));
> 
> / initialize string array for 10 elements */
> njt_array_init(&b, pool, 10, sizeof(njt_str_t));
> ```

使用以下函数将元素添加到数组：

- `njt_array_push(a)`添加一个尾部元素并返回指针指向它
- `njt_array_push_n(a, n)`添加尾部元素 并返回指向第一个的指针`n`

如果当前分配的内存量不够大，无法容纳 新元素，分配新的内存块和现有元素 被复制到其中。 新内存块通常是现有内存块的两倍。

> ```C
> s = njt_array_push(a);
> ss = njt_array_push_n(&b, 3);
> ```

## 6.2列表

在njet中，列表是数组序列，针对数组插入大量元素进行了优化。 列表njt_list_t类型定义如下：

> ```C
> typedef struct {
>     njt_list_part_t  *last;
>     njt_list_part_t   part;
>     size_t            size;
>     njt_uint_t        nalloc;
>     njt_pool_t       *pool;
> } njt_list_t;
> ```

列表中实际存储项，定义如下：

> ```C
> typedef struct njt_list_part_s  njt_list_part_t;
> 
> struct njt_list_part_s {
>     void             *elts;
>     njt_uint_t        nelts;
>     njt_list_part_t  *next;
> };
> ```

使用前，必须通过调用`njt_list_init(list, pool, n, size)` 初始化列表或通过调用njt_list_create(pool, n, size)创建列表。 这两个函数都需要传入每一项的大小和数量。 若要将项添加到列表，请使用 `njt_list_push(list)`函数。 要循环访问项目，请直接访问列表字段，例如：

> ```C
> njt_str_t        *v;
> njt_uint_t        i;
> njt_list_t       *list;
> njt_list_part_t  part;
> 
> list = njt_list_create(pool, 100, sizeof(njt_str_t));
> if (list == NULL) { / error / }
> 
> / add items to the list /
> 
> v = njt_list_push(list);
> if (v == NULL) { / error / }
> njt_str_set(v, "foo");
> 
> v = njt_list_push(list);
> if (v == NULL) { / error / }
> njt_str_set(v, "bar");
> 
> / iterate over the list /
> 
> part = &list->part;
> v = part->elts;
> 
> for (i = 0; / void */; i++) {
> 
>     if (i >= part->nelts) {
>         if (part->next == NULL) {
>             break;
>         }
> 
>         part = part->next;
>         v = part->elts;
>         i = 0;
>     }
> 
>     njt_do_smth(&v[i]);
> }
> ```

列表主要用于 HTTP 输入和输出header。

列表不支持删除项目。 但是，在需要时，可以在内部将项目标记为缺失，而实际上 已从列表中删除。 例如，要将 HTTP 输出header（存储为njt_table_elt_t对象）标记为缺失，请将hash字段设置为 零。 以这种方式标记的项在迭代标头时显式跳过

## 6.3队列

在njet中，队列是采用的是双向链表，每个节点定义 遵循：

> ```C
> typedef struct njt_queue_s  njt_queue_t;
> 
> struct njt_queue_s {
>     njt_queue_t  *prev;
>     njt_queue_t  *next;
> };
> ```

头队列节点未与任何数据链接。 使用前，使用njt_queue_init(q)去初始化。 队列支持以下操作：

- `njt_queue_insert_head(h, x)`，`njt_queue_insert_tail(h, x)` — 插入新节点
- `njt_queue_remove(x)`— 删除队列节点
- `njt_queue_split(h, q, n)`— 在节点上拆分队列， 在单独的队列中返回队列尾部
- `njt_queue_add(h, n)`— 将第二个队列添加到第一个队列
- `njt_queue_head(h)`，`njt_queue_last(h)` — 获取第一个或最后一个队列节点
- `njt_queue_sentinel(h)`- 获取队列哨兵对象
- `njt_queue_data(q, type, link)`—基于队列字段偏移量， 获取对队列节点数据结构的开头，

举个例子：

> ```C
> typedef struct {
>     njt_str_t    value;
>     njt_queue_t  queue;
> } njt_foo_t;
> 
> njt_foo_t    *f;
> njt_queue_t   values, *q;
> 
> njt_queue_init(&values);
> 
> f = njt_palloc(pool, sizeof(njt_foo_t));
> if (f == NULL) { /* error / }
> njt_str_set(&f->value, "foo");
> 
> njt_queue_insert_tail(&values, &f->queue);
> 
> / insert more nodes here */
> 
> for (q = njt_queue_head(&values);
>      q != njt_queue_sentinel(&values);
>      q = njt_queue_next(q))
> {
>     f = njt_queue_data(q, njt_foo_t, queue);
> 
>     njt_do_smth(&f->value);
> }
> ```

## 6.4红黑树

在src/core/njt_rbtree.h头文件提供了对红黑树的使用。

> ```C
> typedef struct {
>     njt_rbtree_t       rbtree;
>     njt_rbtree_node_t  sentinel;
> 
>     /* custom per-tree data here /
> } my_tree_t;
> 
> typedef struct {
>     njt_rbtree_node_t  rbnode;
> 
>     / custom per-node data */
>     foo_t              val;
> } my_node_t;
> ```

要处理整个树，您需要两个节点：根节点和哨兵节点。 通常，它们被添加到自定义结构中，允许您将数据组织到树中，树叶中包含指向或嵌入的链接指向您的数据。

初始化树：

> ```C
> my_tree_t  root;
> 
> njt_rbtree_init(&root.rbtree, &root.sentinel, insert_value_function);
> ```

要遍历树并插入新值，请使用 “insert_value” 函数。 例如，njt_str_rbtree_insert_value函数处理njt_str_t类型。 它的参数是指向插入的根节点的指针，新创建的要添加的节点，以及树哨兵。

> void njt_str_rbtree_insert_value(njt_rbtree_node_t *temp,                                 njt_rbtree_node_t *node,                                 njt_rbtree_node_t *sentinel)

遍历非常简单，可以用 以下查找函数模式：

> ```C
> my_node_t *
> my_rbtree_lookup(njt_rbtree_t *rbtree, foo_t *val, uint32_t hash)
> {
>     njt_int_t           rc;
>     my_node_t          *n;
>     njt_rbtree_node_t  *node, *sentinel;
> 
>     node = rbtree->root;
>     sentinel = rbtree->sentinel;
> 
>     while (node != sentinel) {
> 
>         n = (my_node_t *) node;
> 
>         if (hash != node->key) {
>             node = (hash < node->key) ? node->left : node->right;
>             continue;
>         }
> 
>         rc = compare(val, node->val);
> 
>         if (rc < 0) {
>             node = node->left;
>             continue;
>         }
> 
>         if (rc > 0) {
>             node = node->right;
>             continue;
>         }
> 
>         return n;
>     }
> 
>     return NULL;
> }
> ```

compare()功能是一个经典的比较器函数， 返回小于、等于或大于零的值。 若要加快查找速度并避免比较可能较大的对象，请使用整数哈希字段。

要将节点添加到树中，请分配一个新节点，对其进行初始化并调用`njt_rbtree_insert()`：

> ​    my_node_t          *my_node;    njt_rbtree_node_t  *node;     my_node = njt_palloc(...);    init_custom_data(&my_node->val);     node = &my_node->rbnode;    node->key = create_key(my_node->val);     njt_rbtree_insert(&root->rbtree, node);

若要删除节点，请调用 `njt_rbtree_delete()`函数：

> njt_rbtree_delete(&root->rbtree, node);

## 6.5哈希表

哈希表函数在`src/core/njt_hash.h` 中声明。 支持精确匹配和通配符匹配。 后者需要额外的设置，将在下面的单独部分中进行描述。

在初始化哈希之前，您需要知道它将要处理的元素数量 保持，以便njet可以最佳地构建它。 需要配置的两个参数max_size 和 bucket_size，详见单独的[文档](http://nginx.org/en/docs/hash.html)。 它们通常可由用户配置。 哈希初始化设置为 `njt_hash_init_t`类型以及哈希本身njt_hash_t

> njt_hash_t       foo_hash; njt_hash_init_t  hash; hash.hash = &foo_hash; hash.key = njt_hash_key; hash.max_size = 512; hash.bucket_size = njt_align(64, njt_cacheline_size); hash.name = "foo_hash"; hash.pool = cf->pool; hash.temp_pool = cf->temp_pool;

key是一个指向创建哈希函数的指针， 该函数使用字符串中去创建一个hash整数值。 有两个通用的key创建函数：njt_hash_key(data, len)和 `njt_hash_key_lc(data, len)`.。 后者将字符串转换为所有小写字符，因此传递的字符串 必须是可写的。 如果不是这样，请传递njt_HASH_READONLY_KEY标志到函数，去初始化可以数组（见下文）。

哈希键存储在 njt_hash_keys_arrays_t和 使用njt_hash_keys_array_init(arr, type)去初始化 ： 第二个参数 （type） 控制资源量 为哈希预分配，可以是 NJT_HASH_SMALL或 NJT_HASH_LARGE。 如果您希望哈希包含数千个 元素，可参考如下代码

> njt_hash_keys_arrays_t  foo_keys; foo_keys.pool = cf->pool; foo_keys.temp_pool = cf->temp_pool; njt_hash_keys_array_init(&foo_keys, NJT_HASH_SMALL);

要将键插入哈希键数组，请使用以下函数：`njt_hash_add_key(keys_array, key, value, flags)`

> njt_str_t k1 = njt_string("key1"); njt_str_t k2 = njt_string("key2"); njt_hash_add_key(&foo_keys, &k1, &my_data_ptr_1, NJT_HASH_READONLY_KEY); njt_hash_add_key(&foo_keys, &k2, &my_data_ptr_2, NJT_HASH_READONLY_KEY);

要构建哈希表，请调用该函数：`njt_hash_init(hinit, key_names, nelts)`

> njt_hash_init(&hash, foo_keys.keys.elts, foo_keys.keys.nelts);

如果`max_size`或者`bucket_size`参数不够大，函数将失败。

构建哈希后，使用该函数查找元素：`njt_hash_find(hash, key, name, len)`

> my_data_t   *data;* *njt_uint_t   key;* *key = njt_hash_key(k1.data, k1.len);* *data = njt_hash_find(&foo_hash, key, k1.data, k1.len);* *if (data ==* *NULL**) {*    */* key not found */ }

## 6.6通配符匹配

若要创建使用通配符的哈希，请使用 `njt_hash_combined_t`类型。 它包括上述哈希类型，并具有两个附加键数组：dns_wc_head和dns_wc_tail 。 基本属性的初始化类似于常规哈希：

> njt_hash_init_t      hash njt_hash_combined_t  foo_hash; hash.hash = &foo_hash.hash; hash.key = ...;

可以使用标志添加通配符键：NJT`_HASH_WILDCARD_KEY`

> /* k1 = ".example.org"; */* */* k2 = "foo.*";        */ njt_hash_add_key(&foo_keys, &k1, &data1, NJT_HASH_WILDCARD_KEY); njt_hash_add_key(&foo_keys, &k2, &data2, NJT_HASH_WILDCARD_KEY);

该函数识别通配符并将键添加到相应的数组中。 请参考map模块 通配符语法说明的文档和 匹配算法。

根据添加的密钥的内容，您可能需要初始化最多三个 键数组：一个用于精确匹配（如上所述），另外两个用于启用 从字符串的头部或尾部开始匹配：

> if (foo_keys.dns_wc_head.nelts) {     njt_qsort(foo_keys.dns_wc_head.elts,              (size_t) foo_keys.dns_wc_head.nelts,              sizeof(njt_hash_key_t),              cmp_dns_wildcards);     hash.hash = NULL;    hash.temp_pool = pool;     if (njt_hash_wildcard_init(&hash, foo_keys.dns_wc_head.elts,                               foo_keys.dns_wc_head.nelts)        != NJT_OK)    {        return NJT_ERROR;    }     foo_hash.wc_head = (njt_hash_wildcard_t *) hash.hash; }

键数组需要排序，初始化结果必须添加 到组合哈希。 dns_wc_tail数组的初始化以类似的方式完成。

组合哈希中的查找使用：`njt_hash_find_combined(chash, key, name, len)`

> /* key = "bar.example.org"; — will match ".example.org" */* */* key = "foo.example.com"; — will match "foo.*"        */ hkey = njt_hash_key(key.data, key.len); res = njt_hash_find_combined(&foo_hash, hkey, key.data, key.len);



## 7.内存管理

## 7.1堆

要从系统堆分配内存，请使用以下函数：

- `njt_alloc(size, log)`— 从系统堆分配内存。 这是一个使用`malloc()` 并具有日志记录支持的包装器。 分配错误和调试信息将记录到`log`
- `njt_calloc(size, log)`— 从系统堆分配内存比如njt_alloc()，但在后面用零填充内存 分配。`njt_alloc()`
- `njt_memalign(alignment, size, log)`— 从系统堆分配对齐的内存。 这是提供posix_memalign()该功能的平台上的包装器。 否则，实现将回退到njt_alloc()。
- `njt_free(p)`— 释放分配的内存。 这是一个`free()`包装器

## 7.2池

大多数njet分配都是在池中完成的。 njet 池中分配的内存在池中自动释放 摧毁。 这提供了良好的分配性能，并使内存控制变得容易。

池在内部分配连续内存块中的对象。 一旦块已满，就会分配一个新的块并将其添加到池内存中block列表。 当请求的分配太大而无法放入块中时，请求 被转发到系统分配器，并且 返回的指针存储在池中，以便进一步解除分配。

njet 池的类型是`njt_pool_t`. 。 支持以下操作：

- `njt_create_pool(size, log)`— 创建具有指定 块大小。 返回的池对象也会在池中分配。size 应至少为NJT_MIN_POOL_SIZE 和NJT_POOL_ALIGNMENT 的倍数。
- `njt_destroy_pool(pool)`— 释放所有池内存，包括 池对象本身。
- `njt_palloc(pool, size)`— 从 指定的池分配对齐的内存。
- `njt_pcalloc(pool, size)`—  从指定的池中分配对齐的内存，并用零填充它。
- `njt_pnalloc(pool, size)`— 从 指定的池分配未对齐的内存。 主要用于分配字符串。
- `njt_pfree(pool, p)`— 释放以前从指定的池中分配的内存。 仅可以释放由转发到系统分配器的请求产生的内存。

> u_char      *p; njt_str_t   *s; njt_pool_t  *pool;* *pool = njt_create_pool(1024, log);* *if (pool ==* *NULL**) { /* error */ }* *s = njt_palloc(pool, sizeof(njt_str_t));* *if (s == NULL) { /* error */ }* *njt_str_set(s, "foo");* *p = njt_pnalloc(pool, 3);* *if (p == NULL) { /* error */ } njt_memcpy(p, "foo", 3);

Chain links（njt_chain_t）在njet中被积极使用， 因此，njet pool实现提供了一种重用它们的方法。 njt_pool_t中的`chain` 地段保持一个以前分配的链接列表，可供重用。 为了在池中有效分配链节，请使用`njt_alloc_chain_link(pool)` 函数。 此函数在池列表中查找空闲链链接并分配新的 链链接（如果池列表为空）。 若要释放链接，请调用该函数`njt_free_chain(pool, cl)` 。

可以在池中注册清理处理程序。当池销毁时回回调带有参数的清理处理程序。 池通常绑定到特定的njet对象（如HTTP请求），并且 当对象达到其生存期结束时销毁。 注册池清理是释放资源、关闭文件描述符或对主要对象关联的共享数据进行最终的调整。

要注册池清理，请调用njt_pool_cleanup_add(pool, size) ，这将返回指向 `njt_pool_cleanup_t` 的指针，该指针 由调用者填写。 使用size参数为清理handler分配上下文。

> njt_pool_cleanup_t  *cln;* *cln = njt_pool_cleanup_add(pool, 0);* *if (cln ==* *NULL**) { /* error */ } cln->handler = njt_my_cleanup; cln->data = "foo"; ... static void njt_my_cleanup(void *data) {    u_char  *msg = data;     njt_do_smth(msg); }

## 7.3共享内存

njet使用共享内存在进程之间共享公共数据。 njt_shared_memory_add(cf, name, size, tag)函数添加一个新的njt_shm_zone_t 共享内存item到cycle。 该函数接收zone的 name和size。 每个共享区域必须具有唯一的名称。 如果提供`name` 和tag的共享区域item已存在，则重用现有区域item。 如果具有name的现有区域具有 不同的tag， 该函数会调用并产生一个错误。模块结构的地址被传递为tag ，使得可以在一个njet模块通过name重用该共享内存。

共享内存entry结构njt_shm_zone_t具有 以下字段：

- `init`— 初始化回调，在共享区域之后调用 映射到实际内存
- `data`— 数据上下文，用于将任意数据传递给回调`init`
- `noreuse`— flag， 禁止从old cycle重用共享的zone
- `tag`— 共享区域标签
- `shm`— 特定于平台的njt_shm_t对象类型，至少具有以下字段：
  - `addr`— 映射的共享内存地址，最初为空
  - `size`— 共享内存大小
  - `name`— 共享内存名称
  - `log`— 共享内存日志
  - `exists`— flag，指示从主进程继承的共享内存（特定于 Windows）

njt_init_cycle()解析配置后，共享区域条目将映射到实际内存。 在 POSIX 系统上，mmap()系统调用用于创建 共享匿名映射。 在 Windows 上，使用`CreateFileMapping()`/ `MapViewOfFileEx()`。

为了在共享内存中分配，njet提供了slab pool类型 `njt_slab_pool_t` 。 用于分配内存的slab pool在njet每个共享的中自动创建内存。 池位于共享区域的开头，可通过(njt_slab_pool_t *) shm_zone->shm.addr访问 . 要在共享区域中分配内存，请调用 njt_slab_alloc(pool, size)或 njt_slab_calloc(pool, size)。 要释放内存，请调用  `njt_slab_free(pool, p)`。

Slab pool将所有共享区域划分为多个页面。 每个页面用于分配相同大小的对象。 指定的大小必须是 2 的幂，并且大于 8 字节。 不符合项的值将向上舍入。 每个页面的位掩码跟踪哪些块正在使用中，哪些块是空闲的 分配。 对于大于半页（通常为 2048 字节）的大小，分配为 一次完成一整页

要保护共享内存中的数据免受并发访问，请使用互斥锁 在`njt_slab_pool_t`的`mutex` 字段中可用。 slab pool在分配和释放时最常使用互斥锁，它可用于保护在共享区域中分配的任何其他用户数据结构 。 要锁定或解锁互斥锁，请分别调用 `njt_shmtx_lock(&shpool->mutex)` 或者 `njt_shmtx_unlock(&shpool->mutex)`

> njt_str_t        name; njt_foo_ctx_t   *ctx; njt_shm_zone_t  *shm_zone; njt_str_set(&name, "foo"); /* allocate shared zone context */* *ctx = njt_pcalloc(cf->pool, sizeof(njt_foo_ctx_t));* *if (ctx ==* *NULL**) {*    */* error */* *}* */* add an entry for 64k shared zone */* *shm_zone = njt_shared_memory_add(**cf**, &name, 65536, &njt_foo_module);* *if (shm_zone == NULL) {*    */* error */* *}* */* register init callback and context */ shm_zone->init = njt_foo_init_zone; shm_zone->data = ctx;  ...  static njt_int_t njt_foo_init_zone(njt_shm_zone_t *shm_zone, void *data) {    njt_foo_ctx_t  *octx = data;     size_t            len;    njt_foo_ctx_t    *ctx;    njt_slab_pool_t  *shpool;     value = shm_zone->data;     if (octx) {        /* reusing a shared zone from old cycle */        ctx->value = octx->value;        return NJT_OK;    }     shpool = (njt_slab_pool_t *) shm_zone->shm.addr;     if (shm_zone->shm.exists) {        /* initialize shared zone context in Windows njet worker */*        *ctx->value = shpool->data;*        *return NJT_OK;*    *}*     */* initialize shared zone */     ctx->value = njt_slab_alloc(shpool, sizeof(njt_uint_t));    if (ctx->value == NULL) {        return NJT_ERROR;    }     shpool->data = ctx->value;     return NJT_OK; }



# 8.LOG日志规范

使用njt_log_t对象用于njet日志。 njet记录器支持多种类型的输出：

- stderr — 记录到标准错误 （stderr）
- 文件 — 记录到文件
- 系统日志 — 记录到系统日志
- 内存 — 记录到内部内存存储以用于开发目的;以后可以使用调试器访问

logger实例可以是loggers链，使用next字段进行链接。 在这种情况下，每条消息都会写入链中的所有logger。

对于每个logger，通过日志严重性级别控制将哪些消息写入日志（仅记录分配了该级别或更高级别的事件）。 支持以下严重性级别：

- `NJT_LOG_EMERG`
- `NJT_LOG_ALERT`
- `NJT_LOG_CRIT`
- `NJT_LOG_ERR`
- `NJT_LOG_WARN`
- `NJT_LOG_NOTICE`
- `NJT_LOG_INFO`
- `NJT_LOG_DEBUG`

对于调试日志记录，还会检查调试掩码。 调试掩码包括：

- `NJT_LOG_DEBUG_CORE`
- `NJT_LOG_DEBUG_ALLOC`
- `NJT_LOG_DEBUG_MUTEX`
- `NJT_LOG_DEBUG_EVENT`
- `NJT_LOG_DEBUG_HTTP`
- `NJT_LOG_DEBUG_MAIL`
- `NJT_LOG_DEBUG_STREAM`

通常，loggers 是由现有njet代码中的error_log指令创建的，几乎在cycle处理、配置、客户端连接和其他对象处理的每个阶段都可使用

njet 提供以下日志记录宏：

- `njt_log_error(level, log, err, fmt, ...)`— 错误日志记录
- `njt_log_debug0(level, log, err, fmt)`，`njt_log_debug1(level, log, err, fmt, arg1)`等 — 调试 使用多达八个受支持的格式参数进行日志记录

日志消息在堆栈上的NJT_MAX_ERROR_STR大小（当前为 2048 字节）的缓冲区中格式化。 消息前面带有严重性级别、进程 ID （PID）、连接 ID（存储在log->connection 中）和系统错误文本。 对于非调试消息，调用 log->handler在日志消息前面附加更具体的信息。 HTTP 模块将功能设置为njt_http_log_error()日志 用于记录客户端和服务器地址、当前操作（存储在log->action）、客户端请求行、服务器名称等的处理程序。

> /* specify what is currently done */* *log->action = "sending mp4 to client";* */* error and debug log */ njt_log_error(NJT_LOG_INFO, c->log, 0, "client prematurely              closed connection"); njt_log_debug2(NJT_LOG_DEBUG_HTTP, mp4->file.log, 0,               "mp4 start:%ui, length:%ui", mp4->start, mp4->length);

上面的示例会产生如下日志条目：

> 2016/09/16 22:08:52 [info] 17445#0: *1 client prematurely closed connection while sending mp4 to client, client: 127.0.0.1, server: , request: "GET /file.mp4 HTTP/1.1" 2016/09/16 23:28:33 [debug] 22140#0: *1 mp4 start:0, length:10000



# 9.Cycle

cycle对象存储在特定配置创建的 njet 运行时上下文中。 其类型为 njt_cycle_t。 当前的cycle又worker进程启动时继承，并存在在njt_cycle全局变量中。 每次重新加载 njet 配置时，都会从 新的njet配置创建一个新的cycle，旧cycle通常在新cycle成功创建之后删除。

一个cycle由函数njt_init_cycle()创建，该函数 将前一个cycle作为其参数。 该函数使用前一个cycle的配置文件并继承为 上一个cycle尽可能多的资源。 一个称为“init cycle”的占位符循环被创建为 njet start，然后是 替换为从配置构建的实际cycle。

该cycle的成员包括：

- `pool`— 循环池。 为每个新cycle创建。
- `log`— 循环日志。 最初继承自旧cycle，设置为在读取配置后指向`new_log`。
- `new_log`— cycle日志，由配置创建。 它受根作用域指令error_log的影响。
- `connections`,`connection_n` —njt_connection_t 类型的连接数组，由 创建 初始化每个 njet 工作线程时的事件模块。 njet 配置中的指令 worker_connections设置连接数connection_n。
- `free_connections`，`free_connection_n` — 当前可用的列表和数量 连接。 如果没有可用的连接，njet工作线程拒绝接受新客户端 或连接到上游服务器。
- `files`，`files_n` — 映射文件的数组 njet 连接的描述符。 此映射由事件模块使用，具有NJT_USE_FD_EVENT标志（当前为`poll` 和 `devpoll`）。
- `conf_ctx`— 核心模块配置阵列。 配置是在读取njet配置期间创建和填充的 文件。
- `modules`，`modules_n` —  `njt_module_t`类型的模块数组，静态和动态，加载者 当前配置。
- `listening`— njt_listening_t类型的侦听对象数组。 侦听对象通常由调用`njt_create_listening()` 函数的不同模块的listen指令添加。 侦听套接字是基于侦听对象创建的。
- `paths`— `njt_path_t`.类型的路径数组。 路径是通过从调用njt_add_path()函数来添加的 将在某些目录上运行的模块。 如果缺失，这些目录将在读取配置后由njet创建的，。 此外，可以为每个路径添加两个处理程序：
  - path loader — 启动或重新加载 njet后 60 秒仅执行一次。 通常，加载器读取目录并将数据存储在njet共享内存中。 处理程序是从专用的njet进程“njet缓存加载器”调用的。
  - 路径管理器 — 定期执行。 通常，管理器会从目录中删除旧文件并更新njet 内存以反映更改。 处理程序是从专用的“njet 缓存管理器”进程中调用的。
- `open_files`— 通过调用`njt_conf_open_file()`函数创建的njt_open_file_t类型的文件对象列表。 目前，njet使用这种打开的文件进行日志记录。 阅读配置后，njet打开列表中的所有文件，并将每个文件描述符存储在open_files 对象的fd字段。 这些文件以追加模式打开，如果缺少，则会创建这些文件。 列表中的文件由njet工作人员在收到 重新打开信号（最常见的USR1）。 在这种情况下，fd字段中的描述符将更改为新value。
- `shared_memory`— 共享内存区域列表，每个区域由 调用njt_shared_memory_add()函数。 共享区域映射到所有njet进程中的相同地址范围，并且 用于共享公共数据，例如 HTTP 缓存内存中树。



# 10.buffer

对于输入/输出操作，njet 提供了缓冲区类型njt_buf_t。 通常，它用于保存要写入目标或从目标读取的数据 源。 缓冲区可以引用内存或文件中的数据，从技术上讲，它是 缓冲区可以同时引用两者。 缓冲区的内存是单独分配的，与缓冲区njt_buf_t结构无关。

njt_buf_t结构具有以下字段：

- `start`，`end` —  为缓冲区分配的内存块的边界。
- `pos, last ` — buffer内存的边界;通常是`start` .. `end` 的子范围
- `file_pos, file_last` — 文件buffer的边界，表示为文件开头的偏移量。
- `tag`— 用于区分缓冲区的唯一值;不同的njet模块创建，通常用于缓冲区重用的目的。
- `file`— 文件对象。
- `temporary`— 指示缓冲区引用的标志 可写内存。
- `memory`— 指示缓冲区引用只读的标志。
- `in_file`— 指示缓冲区引用数据的在文件中的标志。
- `flush`— 指示缓冲区之前的所有数据的需要清洗的标志 。
- `recycled`— 指示缓冲区可以重用的标志，并且需要尽快消费。
- `sync`— 指示缓冲区不携带任何数据的标志或 特殊信号如`flush` 或last_buf . 默认情况下，njet 认为这样的缓冲区是一个错误条件，但这个标志告诉 njet 跳过错误检查。
- `last_buf`— 指示缓冲区是最后一个的输出的标志 。
- `last_in_chain`— 指示请求或子请求中的缓冲区中没有更多数据的标志 。
- `shadow`— 引用与 当前缓冲区有关的另一个 ("shadow") buffer。 当该buffer已经消费后，对应的shadow buffer通常也标记为已消费。
- `last_shadow`— 指示缓冲区是一个特定shadow buffer的最后一个的标志。
- `temp_file`— 指示缓冲区处于临时状态的标志 文件。

对于输入和输出操作，缓冲区以链形式链接。 链是链njt_chain_t类型的链链序列， 定义如下：

> typedef struct njt_chain_s  njt_chain_t; struct njt_chain_s {    njt_buf_t    *buf;    njt_chain_t  *next; };

每个链条链接都保留对其缓冲区的引用和对下一个缓冲区的引用 链环。

使用缓冲区和链的示例：

> njt_chain_t * njt_get_my_chain(njt_pool_t *pool) {    njt_buf_t    *b;    njt_chain_t  *out, *cl, **ll;     /* first buf */*    *cl = njt_alloc_chain_link(pool);*    *if (cl ==* *NULL**) { /* error */ }*     *b = njt_calloc_buf(pool);*    *if (b == NULL) { /* error */ }     b->start = (u_char *) "foo";    b->pos = b->start;    b->end = b->start + 3;    b->last = b->end;    b->memory = 1; /* read-only memory */*     *cl->buf = b;*    *out = cl;*    *ll = &cl->next;*     */* second buf */*    *cl = njt_alloc_chain_link(pool);*    *if (cl == NULL) { /* error */ }*     *b = njt_create_temp_buf(pool, 3);*    *if (b == NULL) { /* error */ }     b->last = njt_cpymem(b->last, "foo", 3);     cl->buf = b;    cl->next = NULL;    *ll = cl;     return out; }

# 11.Networking

## 11.1Connection

连接类型njt_connection_t主要是围绕 套接字描述符。 它包括以下字段：

- `fd`— 套接字描述符
- `data`— 任意连接上下文。 通常，它是指向构建在连接之上的高层的对象，例如 HTTP 请求或STREAM会话。
- `read, write ` — 读取和写入事件。
- `recv, send, recv_chain, send_chain`、— I/O 操作。
- `pool`— 连接池。
- `log`— 连接日志。
- `sockaddr, socklen, addr_text`， — 二进制和文本形式的远程套接字地址。
- `local_sockaddr, local_socklen ` — 本地 二进制形式的套接字地址。 最初，这些字段为空。 使用 `njt_connection_local_sockaddr()`函数获取 本地套接字地址。
- `proxy_protocol_addr, proxy_protocol_port` - 代理协议客户端地址和端口（如果启用了代理协议） 连接。
- `ssl`— 连接的 SSL 上下文。
- `reusable`— 指示连接使其有资格重复使用的标志。
- `close`— 指示连接已经重用并且需要关闭的标志。

njet连接可以透明地封装SSL层。 在这种情况下，连接的ssl字段包含指向`njt_ssl_connection_t` 结构的指针，保留所有与 SSL 相关的数据 对于连接，包括 `SSL_CTX` 和SSL 。`recv`, `send`, `recv_chain`, 和`send_chain` 处理程序也设置为启用 SSL 的函数。

njet 配置中的`worker_connections` 指令 限制每个 njet 工作线程的连接数。 所有连接结构都是在工作线程启动时预先创建的，并存储在cycle对象的`connections` 字段。 若要检索连接结构，请使用njt_get_connection(s, log)函数。 它采用套接字描述符作为s其参数，它需要 包裹在连接结构中。

由于每个工作线程的连接数有限，njet 提供了一个 获取当前正在使用的连接的方法。 若要启用或禁用重用的连接，请调用该函数。 调用在连接结构中设置标志并插入 连接到循环。 每当发现没有 周期列表中的可用连接， 它调用发布 可重用连接的特定数量。 对于每个此类连接，设置标志并读取 handler 被调用，它应该通过调用来释放连接并使其可供重用。 调用可以重用连接时退出状态。 HTTP客户端连接是njet中可重用连接的一个例子;他们 标记为可重用，直到从客户端收到第一个请求字节。



# 12.Events

## 12.1事件

njet 中的事件`njt_event_t`对象提供了一种机制 用于通知已发生特定事件。

`njt_event_t`中的字段包括：

- `data`— 事件处理程序中使用的任意事件上下文， 通常作为指向与事件相关的连接的指针。
- `handler`— 事件发生时要调用的回调函数。
- `write`— 指示写入事件的标志。 缺少该标志表示读取事件。
- `active`— 指示事件已注册的标志 接收 I/O 通知，通常来自`epoll`, `kqueue`, `poll`等通知机制。
- `ready`— 指示事件已收到 I/O 通知。
- `delayed`— 指示 I/O 因速率而延迟的限制标志。
- `timer`— 用于将事件插入的红黑树节点计时器树。
- `timer_set`— 指示事件计时器已设置的标志，并且 尚未过期。
- `timedout`— 指示事件计时器已过期的标志。
- `eof`— 指示读取数据时发生 EOF 的标志。
- `pending_eof`— 指示 EOF 在 套接字，即使之前可能有一些数据可用。 标志通过`EPOLLRDHUP` `epoll`事件或`EV_EOF` `kqueue` 标志传递。
- `error`— 指示在以下期间发生错误的标志 读取（对于读取事件）或写入（对于写入事件）。
- `cancelable`— 计时器事件标志，指示事件 在关闭工作线程时应忽略。 正常工作器关闭延迟，直到没有不可取消的计时器 已安排的活动。
- `posted`— 指示事件已发布到队列的标志。
- `queue`— 用于将事件发布到队列的队列节点。

## 12.2 I/O 事件

通过调用njt_get_connection()函数获得的每个连接都有两个附加事件，c->read以及c->write ，用于接收以下通知： 套接字已准备好读取或写入。 所有此类事件都在边沿触发模式下运行，这意味着它们仅触发 套接字状态更改时的通知。 例如，对套接字进行部分读取不会使njet提供 重复读取通知，直到更多数据到达套接字。 即使底层 I/O 通知机制本质上是水平触发（`poll`, `select`等）、njet 将通知转换为边沿触发。 使 njet 事件通知在所有通知系统中保持一致 在不同的平台上，处理 I/O 套接字通知之后必须调用njt_handle_read_event(rev, flags)函数和njt_handle_write_event(wev, lowat)或调用该套接字上的任何 I/O 函数。 通常，函数在每次读取或写入结束时调用一次 事件处理程序。

## 12.3计时器事件

可以将事件设置为在超时到期时发送通知。 事件使用的计时器从某个未指定的点开始计算毫秒数 过去被截断为njt_msec_t类型。 它的当前值可以从变量`njt_current_msec` 中获得。

njt_add_timer(ev, timer)函数为一个事件设置超时，njt_del_timer(ev)删除先前设置的超时。 全局超时红黑树njt_event_timer_rbtree存储当前设置的所有超时。 树中的njt_msec_t类型的键为事件发生时的时间。 树结构可实现快速插入和删除操作，以及 访问最近的超时，njet使用它来找出等待多长时间 用于 I/O 事件和即将过期的超时事件。

## 12.4Posted 事件

可以Posted一个事件，这意味着它的处理程序将在稍后当前事件循环迭代中某个时候被调用。 发布事件是简化代码和转义堆栈的好做法 溢出。 已发布的事件保存在帖子队列中。 njt_post_event(ev, q)将ev事件发布到发布队列。 njt_delete_posted_event(ev)宏从当前发布的队列中删除ev事件。 通常，事件会发布到`njt_posted_events` 队列中， 在事件循环的后期处理 — 在所有 I/O 和计时器之后 事件已处理。 调用`njt_event_process_posted()` 函数以处理 事件队列。 它调用事件处理程序，直到队列不为空。 这意味着已发布的事件处理程序可以发布更多要处理的事件 在当前事件循环迭代中。

举个例子：

> ```C
> void
> njt_my_connection_read(njt_connection_t *c)
> {
>     njt_event_t  *rev;
> 
>     rev = c->read;
> 
>     njt_add_timer(rev, 1000);
> 
>     rev->handler = njt_my_read_handler;
> 
>     njt_my_read(rev);
> }
> 
> 
> void
> njt_my_read_handler(njt_event_t *rev)
> {
>     ssize_t            n;
>     njt_connection_t  *c;
>     u_char             buf[256];
> 
>     if (rev->timedout) { /* timeout expired */ }
> 
>     c = rev->data;
> 
>     while (rev->ready) {
>         n = c->recv(c, buf, sizeof(buf));
> 
>         if (n == NJT_AGAIN) {
>             break;
>         }
> 
>         if (n == NJT_ERROR) { /* error */ }
> 
>         /* process buf */
>     }
> 
>     if (njt_handle_read_event(rev, 0) != NJT_OK) { /* error */ }
> }
> ```

## 12.5事件循环

除了 njet master 进程之外，所有 njet 进程都执行 I/O，因此具有 事件循环。 （njet master 进程将大部分时间花在sigsuspend()调用中等待信号到达。 njet事件循环在njt_process_events_and_timers()函数中实现，一直重复调用，直到进程退出。

事件循环具有以下阶段：

- 通过调用 `njt_event_find_timer()`查找最接近到期的超时。 此函数查找计时器树中最左侧的节点，并返回 节点过期前的毫秒数。
- 通过调用特定于事件通知的处理程序来处理 I/O 事件 机制，由njet配置选择。 此处理程序等待至少一个 I/O 事件发生，但只等待下一个 I/O 事件发生 超时过期。 发生读取或写入事件时，将设置该标志并调用事件的处理程序。 对于 Linux，通常使用njt_epoll_process_events()处理程序 ，调用epoll_wait()等待 I/O 事件。
- 通过调用njt_event_expire_timers() 使计时器过期。 计时器树从最左边的元素向右迭代，直到 发现未过期的超时。 对于每个过期`timedout` 的节点，设置timer_set事件标志， 将重置标志，并调用事件处理程序
- 通过调用 njt_event_process_posted()处理已发布的事件。 该函数从发布的事件中重复删除第一个元素 队列并调用元素的处理程序，直到队列为空。

所有njet进程也处理信号。 信号处理程序仅设置在调用njt_process`_events_and_timers()` 后检查的全局变量。



# 13.进程

njet中有几种类型的进程。 进程的类型保留在`NJT_PROCESS`全局变量中，并且是以下类型之一：

- `NJT_PROCESS_MASTER`— 主进程，读取 njet配置，创建周期，启动和控制子进程。 它不执行任何 I/O，仅响应信号。 它的循环函数是 `njt_master_process_cycle()`
- `NJT_PROCESS_WORKER`— 处理客户端 连接的工作进程。 它由主进程启动并响应其信号和通道 命令也是如此。 它的循环函数是njt_worker_process_cycle() 。 可以有多个工作进程，由`worker_processes`指令配置。
- `NJT_PROCESS_SINGLE`— 单个进程，它只存在于master_process off模式中，并且是唯一在 这种模式。 它创建循环（如主进程）并处理客户端连接 （就像工作进程一样）。 它的循环函数是`njt_single_process_cycle()`
- `NJT_PROCESS_HELPER`— 帮助程序进程，目前 有两种类型：缓存管理器和缓存加载程序。 两者的循环函数为 `njt_cache_manager_process_cycle()`

njet 进程处理以下信号：

- `NJT_SHUTDOWN_SIGNAL` (`SIGQUIT`在大多数 系统） — 正常关闭。 收到此信号后，主进程向所有 子进程。 当没有子进程离开时，主进程将销毁循环池并退出。 当工作进程收到此信号时，它会关闭所有侦听套接字，并 等到没有计划不可取消的事件，然后销毁 循环池和出口。 当缓存管理器或缓存加载程序进程收到此信号时，它会 立即退出。njt_quit变量设置为1当 进程接收到此信号，并在处理后立即复位。 工作进程处于关闭状态，`njt_exiting` 变量设置为1 。
- `NJT_TERMINATE_SIGNAL` (`SIGTERM`在大多数 系统） — 终止。 收到此信号后，主进程向所有 子进程。 如果子进程在 1 秒内未退出，则主进程发送`SIGKILL` 信号以终止它。 当没有子进程离开时，主进程将销毁循环池，并且 出口。 当工作进程、缓存管理器进程或缓存加载程序进程时 接收到此信号后，它会破坏循环池并退出。当接收此信号时 njt_terminate变量设置为1。
- `NJT_NOACCEPT_SIGNAL` (`SIGWINCH`在大多数 系统） - 关闭所有工作进程和帮助程序进程。 收到此信号后，主进程将关闭其子进程。 如果以前启动的新 njet 二进制文件退出，则旧的 njet 的子进程 再次启动主控。 当工作进程收到此信号时，它将在调试模式下关闭 由`debug_points`指令设置。
- `NJT_RECONFIGURE_SIGNAL` (`SIGHUP`在大多数 系统） - 重新配置。 收到此信号后，主进程重新读取配置并 基于它创建一个新循环。 如果新周期创建成功，则删除旧周期并新建 子进程已启动。 同时，旧的子进程接收NJT_SHUTDOWN_SIGNAL信号。 在单进程模式下，njet创建一个新循环，但保留旧循环直到 不再有绑定活动连接的客户端。 工作进程和帮助程序进程忽略此信号。
- `NJT_REOPEN_SIGNAL` (`SIGUSR1`在大多数 系统） — 重新打开文件。 主进程将此信号发送给工作线程，工作线程重新打开与循环相关的所有open_files。
- `NJT_CHANGEBIN_SIGNAL` (`SIGUSR2`在大多数 systems） — 更改 njet 二进制文件。 主进程启动一个新的njet二进制文件，并传入所有侦听的socket列表。 在环境中传递的文本格式列表“njet” 变量，由用分号分隔的描述符数字组成。 新的 njet 二进制文件读取“njet”变量并添加 套接字到其初始化周期。 其他进程忽略此信号。

虽然所有njet工作进程都能够接收并正确处理POSIX 信号，主进程不使用标准`kill()`系统调用将信号传递给工作线程和帮助程序。 相反，njet使用进程间套接字对，允许发送消息 在所有 njet 进程之间。 但是，目前消息仅从主服务器发送到其子级。 消息携带标准信号。



# 14.线程

可以卸载到单独的线程任务中，否则这些任务会 阻止 njet 工作进程。 例如，njet可以配置为使用线程来执行[文件 I/O](http://nginx.org/en/docs/http/ngx_http_core_module.html#aio)。 另一个用例是没有异步接口的库 因此不能与njet正常一起使用。 请记住，线程接口是现有 处理客户端连接的异步方法，绝不是 旨在作为替代品。

为了处理同步，可以使用以下基元包装器：`pthreads`

- `typedef pthread_mutex_t njt_thread_mutex_t;`
  - `njt_int_t njt_thread_mutex_create(njt_thread_mutex_t *mtx, njt_log_t *log);`
  - `njt_int_t njt_thread_mutex_destroy(njt_thread_mutex_t *mtx, njt_log_t *log);`
  - `njt_int_t njt_thread_mutex_lock(njt_thread_mutex_t *mtx, njt_log_t *log);`
  - `njt_int_t njt_thread_mutex_unlock(njt_thread_mutex_t *mtx, njt_log_t *log);`
- `typedef pthread_cond_t njt_thread_cond_t;`
  - `njt_int_t njt_thread_cond_create(njt_thread_cond_t *cond, njt_log_t *log);`
  - `njt_int_t njt_thread_cond_destroy(njt_thread_cond_t *cond, njt_log_t *log);`
  - `njt_int_t njt_thread_cond_signal(njt_thread_cond_t *cond, njt_log_t *log);`
  - `njt_int_t njt_thread_cond_wait(njt_thread_cond_t *cond, njt_thread_mutex_t *mtx, njt_log_t *log);`

njet不是为每个任务创建一个新线程，而是实现 [一个thread_pool](http://nginx.org/en/docs/ngx_core_module.html#thread_pool)战略。 可以为不同的目的配置多个线程池 （例如，在不同的磁盘集上执行 I/O）。 每个线程池在启动时创建，包含有限数量的线程 处理任务队列。 任务完成后，将调用预定义的完成处理程序。

头文件`src/core/njt_thread_pool.h` 包含 相关定义：

> ```C
> struct njt_thread_task_s {
>     njt_thread_task_t   *next;
>     njt_uint_t           id;
>     void                *ctx;
>     void               (*handler)(void *data, njt_log_t *log);
>     njt_event_t          event;
> };
> 
> typedef struct njt_thread_pool_s  njt_thread_pool_t;
> 
> njt_thread_pool_t *njt_thread_pool_add(njt_conf_t *cf, njt_str_t *name);
> njt_thread_pool_t *njt_thread_pool_get(njt_cycle_t *cycle, njt_str_t *name);
> 
> njt_thread_task_t *njt_thread_task_alloc(njt_pool_t *pool, size_t size);
> njt_int_t njt_thread_task_post(njt_thread_pool_t *tp, njt_thread_task_t *task);
> ```

在配置时，愿意使用线程的模块必须获得 通过调用 njt_thread_pool_add(cf, name)引用线程池，这将创建一个 具有给定`name`或返回引用的新线程池 到具有该名称的池（如果已存在）。

要在运行时将 添加task到指定线程池的队列中，请使用njt_thread_task_post(tp, task)函数。 要在线程中执行函数，请传递参数并设置完成 使用njt_thread_task_t结构的处理程序：

> ```C
> typedef struct {
>     int    foo;
> } my_thread_ctx_t;
> 
> 
> static void
> my_thread_func(void *data, njt_log_t *log)
> {
>     my_thread_ctx_t *ctx = data;
> 
>     /* this function is executed in a separate thread */
> }
> 
> 
> static void
> my_thread_completion(njt_event_t *ev)
> {
>     my_thread_ctx_t *ctx = ev->data;
> 
>     /* executed in njet event loop */
> }
> 
> 
> njt_int_t
> my_task_offload(my_conf_t *conf)
> {
>     my_thread_ctx_t    *ctx;
>     njt_thread_task_t  *task;
> 
>     task = njt_thread_task_alloc(conf->pool, sizeof(my_thread_ctx_t));
>     if (task == NULL) {
>         return NJT_ERROR;
>     }
> 
>     ctx = task->ctx;
> 
>     ctx->foo = 42;
> 
>     task->handler = my_thread_func;
>     task->event.handler = my_thread_completion;
>     task->event.data = ctx;
> 
>     if (njt_thread_task_post(conf->thread_pool, task) != NJT_OK) {
>         return NJT_ERROR;
>     }
> 
>     return NJT_OK;
> }
> ```



# 15.添加模块

## 15.1添加新模块

每个独立的njet模块使用一个单独的目录，该目录包含至少两个文件：`config` 文件和一个包含模块源代码的文件。 该`config` 文件包含集成到njet所需的所有信息， 例如：

> ```C
> njt_module_type=CORE
> njt_module_name=njt_foo_module
> njt_module_srcs="$njt_addon_dir/njt_foo_module.c"
> 
> . auto/module
> 
> njt_addon_name=$njt_module_name
> ```

config 文件是一个 POSIX shell脚本，可以设置 并访问以下变量：

- `njt_module_type`— 要构建的模块类型。 可能的值为`CORE`, `HTTP`, `HTTP_FILTER`, `HTTP_INIT_FILTER`, `HTTP_AUX_FILTER`, `MAIL`, `STREAM`, 或 `MISC` 。
- `njt_module_name`— 模块名称。 要从一组源文件构建多个模块，请指定 以空格分隔的名称列表。 名字指示动态模块的输出二进制文件的名称。 列表中的名称必须与源代码中使用的名称匹配。
- `njt_addon_name`— 配置脚本输出到控制台上显示的模块名。
- `njt_module_srcs`— 用于编译模块的以空格分隔的源文件列表 。 $njt_addon_dir 变量可用于表示模块目录路径。
- `njt_module_incs`— 包括构建模块所需的路径
- `njt_module_deps`— 空格分隔的模块依赖列表。 通常，它是头文件的列表。
- `njt_module_libs`— 以空格分隔的模块链接库列表。 例如，njt_module_libs=-lpthread 用于链接 libpthread 库。 以下`LIBXSLT`, `LIBGD`, `GEOIP`, `PCRE`, `OPENSSL`, `MD5`, `SHA1`, `ZLIB`, and `PERL`宏可用于链接到与 njet一样的库 。
- `njt_module_link`— 构建系统为动态模块设置`DYNAMIC` 或静态模块设置ADDON，依赖链接类型用于确定要执行的不同操作。
- `njt_module_order`— 模块的加载顺序; 对 `HTTP_FILTER`  和 `HTTP_AUX_FILTER`  模块类型很有用。 此选项的格式是以空格分隔的模块列表。 列表中当前模块名称后面的所有模块都以 模块的全局列表，用于设置模块初始化的顺序。 对于过滤器模块，较晚的初始化意味着较早的执行。
- 以下模块通常用作参考。 njt_http_copy_filter_module 读取其他数据给 过滤器模块，并放置在列表底部附近，以便它是 第一个要执行的。 njt_http_write_filter_module 将数据写入 客户端套接字，位于列表顶部附近，并且是最后一个 执行。
- 默认情况下，过滤器模块放置在模块列表中的 njt_http_copy_filter之前，以便过滤器 处理程序在复制筛选器处理程序之后执行。 对于其他模块类型，默认值为空字符串。

要将模块静态编译为 njet，请使用参数 --add-module=/path/to/module 来配置 脚本。 要编译模块以便以后动态加载到 njet 中，请使用 --add-dynamic-module=/path/to/module 参数。

## 15.2核心模块

模块是njet的构建块，它的大部分功能是 作为模块实现。 模块源文件必须包含 njt_module_t 类型的全局变量，其定义如下：

> ```C
> struct njt_module_s {
> 
>     /* private part is omitted */
> 
>     void                 *ctx;
>     njt_command_t        *commands;
>     njt_uint_t            type;
> 
>     njt_int_t           (*init_master)(njt_log_t *log);
> 
>     njt_int_t           (*init_module)(njt_cycle_t *cycle);
> 
>     njt_int_t           (*init_process)(njt_cycle_t *cycle);
>     njt_int_t           (*init_thread)(njt_cycle_t *cycle);
>     void                (*exit_thread)(njt_cycle_t *cycle);
>     void                (*exit_process)(njt_cycle_t *cycle);
> 
>     void                (*exit_master)(njt_cycle_t *cycle);
> 
>     /* stubs for future extensions are omitted */
> };
> ```

省略的私有部分包括模块版本和签名，并且 使用预定义的 NJT_MODULE_V1 宏填充。

每个模块将其私有数据保存在 ctx 字段， 识别存放在 `commands` 数组中指定的配置指令，并且嵌入在 njet 特定的生命周期中。 模块生命周期由以下事件组成：

- 配置指令处理程序在出现时被调用 在主进程上下文中的配置文件中。
- 成功解析配置后，将在主进程的上下文中调用 `init_module` 处理程序。
- 主进程创建一个或多个工作进程，并在每个工作进程中调用`init_process` 处理程序。
- 当工作进程从主进程接收到 shutdown 或 terminate 命令时，它调用 `exit_process`  处理程序。
- 主进程在退出之前调用 exit_master 处理程序。

因为线程在njet中仅用作补充I / O工具，其拥有自己的 API，当前不会调用 `init_thread` 和`exit_thread`  处理程序。 也没有 `init_master` 处理程序，因为它是不必要的开销。

模块类型 type 字段 准确定义存储在 ctx 字段中的内容。 其值为以下类型之一：

- `NJT_CORE_MODULE`
- `NJT_EVENT_MODULE`
- `NJT_HTTP_MODULE`
- `NJT_MAIL_MODULE`
- `NJT_STREAM_MODULE`

`NJT_CORE_MODULE` 是最基本的，因此也是最通用和最底层类型的模块。 其他模块类型在其之上实现，并提供更多 处理相应域的便捷方式，例如处理事件或 HTTP 请求。

核心模块集包括 `njt_core_module`, `njt_errlog_module`, `njt_regex_module`, `njt_thread_pool_module`  和`njt_openssl_module`模块。 HTTP 模块、Stream 模块、Mail 模块和Event模块也是核心模块。 核心模块的上下文定义为：

> ```C
> typedef struct {
>     njt_str_t             name;
>     void               *(*create_conf)(njt_cycle_t *cycle);
>     char               *(*init_conf)(njt_cycle_t *cycle, void *conf);
> } njt_core_module_t;
> ```

其中 `name` 是模块名称字符串，`create_conf` 和`init_conf`是分别指向创建和初始化模块配置的函数的指针 。 对于核心模块，njet 在解析之前调用`create_conf` ，成功解析之后调用`init_conf`。 典型函数`create_conf` 分配内存并为配置设置默认值。

例如，一个名为 `njt_foo_module`的简单模块应该像这样：

```C
/*
 * Copyright (C) Author.
 */


#include <njt_config.h>
#include <njt_core.h>


typedef struct {
    njt_flag_t  enable;
} njt_foo_conf_t;


static void *njt_foo_create_conf(njt_cycle_t *cycle);
static char *njt_foo_init_conf(njt_cycle_t *cycle, void *conf);

static char *njt_foo_enable(njt_conf_t *cf, void *post, void *data);
static njt_conf_post_t  njt_foo_enable_post = { njt_foo_enable };


static njt_command_t  njt_foo_commands[] = {

    { njt_string("foo_enabled"),
      NJT_MAIN_CONF|NJT_DIRECT_CONF|NJT_CONF_FLAG,
      njt_conf_set_flag_slot,
      0,
      offsetof(njt_foo_conf_t, enable),
      &njt_foo_enable_post },

      njt_null_command
};


static njt_core_module_t  njt_foo_module_ctx = {
    njt_string("foo"),
    njt_foo_create_conf,
    njt_foo_init_conf
};


njt_module_t  njt_foo_module = {
    NJT_MODULE_V1,
    &njt_foo_module_ctx,                   /* module context */
    njt_foo_commands,                      /* module directives */
    njt_CORE_MODULE,                       /* module type */
    NULL,                                  /* init master */
    NULL,                                  /* init module */
    NULL,                                  /* init process */
    NULL,                                  /* init thread */
    NULL,                                  /* exit thread */
    NULL,                                  /* exit process */
    NULL,                                  /* exit master */
    NJT_MODULE_V1_PADDING
};


static void *
njt_foo_create_conf(njt_cycle_t *cycle)
{
    njt_foo_conf_t  *fcf;

    fcf = njt_pcalloc(cycle->pool, sizeof(njt_foo_conf_t));
    if (fcf == NULL) {
        return NULL;
    }

    fcf->enable = NJT_CONF_UNSET;

    return fcf;
}


static char *
njt_foo_init_conf(njt_cycle_t *cycle, void *conf)
{
    njt_foo_conf_t *fcf = conf;

    njt_conf_init_value(fcf->enable, 0);

    return NJT_CONF_OK;
}


static char *
njt_foo_enable(njt_conf_t *cf, void *post, void *data)
{
    njt_flag_t  *fp = data;

    if (*fp == 0) {
        return NJT_CONF_OK;
    }

    njt_log_error(NJT_LOG_NOTICE, cf->log, 0, "Foo Module is enabled");

    return NJT_CONF_OK;
}
```

## 15.3指令配置

`njt_command_t` 类型定义单个配置。 每个支持配置的模块都提供此类结构的数组 描述如何处理参数以及要调用哪些处理程序：

> ```C
> typedef struct njt_command_s  njt_command_t;
> 
> struct njt_command_s {
>     njt_str_t             name;
>     njt_uint_t            type;
>     char               *(*set)(njt_conf_t *cf, njt_command_t *cmd, void *conf);
>     njt_uint_t            conf;
>     njt_uint_t            offset;
>     void                 *post;
> };
> ```

数组末尾使用 njt_null_command 结束。 Name 是指令的名称，它出现在配置文件中，例如“worker_processes”或“listen”。 `type` 是标志字段，用于指定 指令采用的参数、其类型以及它出现的上下文。 这些标志是：

- `NJT_CONF_NOARGS`— 指令不带任何参数。
- `NJT_CONF_1MORE`— 指令接受一个或多个参数。
- `NJT_CONF_2MORE`— 指令接受两个或多个参数。
- `NJT_CONF_TAKE1..NJT_CONF_TAKE7`— 指令完全采用指示数量的参数。
- `NJT_CONF_TAKE12, NJT_CONF_TAKE13, NJT_CONF_TAKE23, NJT_CONF_TAKE123, NJT_CONF_TAKE1234`— 指令可能需要不同数量的 参数。 选项仅限于给定的数字。 例如，`NJT_CONF_TAKE12`  意味着它需要一两个 参数。

指令类型的标志包括：

- `NJT_CONF_BLOCK`— 指令是一个块，也就是说，它可以 在其左大括号和右大括号内包含其他指令，甚至 实现自己的解析器来处理内部内容。
- `NJT_CONF_FLAG`— 指令采用布尔值，on 或者 off 。

指令的上下文定义了它可能出现在配置中的位置：

- `NJT_MAIN_CONF`— 在顶层上下文中。
- `NJT_HTTP_MAIN_CONF`— 在`http` 块中。
- `NJT_HTTP_SRV_CONF`— 在http块的一个 server 块中。
- `NJT_HTTP_LOC_CONF`— 在http块的一个 location 块中。
- `NJT_HTTP_UPS_CONF`—  在http块的一个 upstream 块中。
- `NJT_HTTP_SIF_CONF`— 在 http块中server块的if块中。
- `NJT_HTTP_LIF_CONF`— 在 http块中locaion块的if块中。
- `NJT_HTTP_LMT_CONF`— 在http块的块`limit_except`中。
- `NJT_STREAM_MAIN_CONF`— 在`stream` 块中。
- `NJT_STREAM_SRV_CONF`— 在stream 块的server 块中。
- `NJT_STREAM_UPS_CONF`— 在stream 块的upstream 块中。
- `NJT_MAIL_MAIN_CONF`— 在`mail` 块中。
- `NJT_MAIL_SRV_CONF`— 在mail 块的server 块中。
- `NJT_EVENT_CONF`— 在event 块中。`event`
- `NJT_DIRECT_CONF`— 由不创建上下文层次结构的模块中使用，并且只有一个全局配置。 此配置作为`conf` 参数传递给处理程序。

配置分析器使用这些标志在遇到错误位置的指令时引发错误 和并通过提供适当的配置指针来调用指令处理程序，以便不同位置的相同指令可以将其值存储在不同的位置。

set 字段定义处理指令的处理程序 并将解析的值存储到相应的配置中。 有许多函数可以执行常见转换：

- `njt_conf_set_flag_slot`— 将文字字符串on和off值分别转换为 `njt_flag_t` 类型的 为 1 或 0 的值。
- `njt_conf_set_str_slot`— 将字符串存储为`njt_str_t` 类型的值。
- `njt_conf_set_str_array_slot`— 将值附加到字符串njt_str_t 类型的`njt_array_t`数组。 如果 不存在，则创建数组。
- `njt_conf_set_keyval_slot`— 将键值对附加到 njt_keyval_t 键值对`njt_array_t`  数组。 第一个字符串成为键，第二个字符串成为值。 如果数组尚不存在，则创建该数组。
- `njt_conf_set_num_slot`— 转换指令的参数 到一个`njt_int_t` 值。
- `njt_conf_set_size_slot`— 将[大小](http://nginx.org/en/docs/syntax.html)转换为`size_t`值 以字节表示。
- `njt_conf_set_off_slot`— 将[偏移](http://nginx.org/en/docs/syntax.html)量转换为`size_t` 值 以字节表示。
- `njt_conf_set_msec_slot`— 将[时间](http://nginx.org/en/docs/syntax.html)转换为`njt_msec_t` 值 以毫秒表示。
- `njt_conf_set_sec_slot`— 将[时间](http://nginx.org/en/docs/syntax.html)转换为`time_t`值 以秒为单位表示。
- `njt_conf_set_bufs_slot`— 转换为支持两个参数的`njt_bufs_t` ， 保存缓冲区数量和[大小](http://nginx.org/en/docs/syntax.html)。
- `njt_conf_set_enum_slot`— 转换提供的参数 成一个`njt_uint_t` 值。 字段中传递的以 null 结尾的`njt_conf_enum_t` 数组， `post`字段定义了可接受的字符串和相应的 整数值。
- `njt_conf_set_bitmask_slot`— 转换提供的参数 成一个`njt_uint_t` 值。 每个参数的掩码值都经过 OR 运算生成结果。 字段中传递的以 null 结尾的`njt_conf_bitmask_t` 数组，`post` 字段定义了可接受的字符串和相应的 掩码值。
- `set_path_slot`— 将提供的参数转换为`njt_path_t` 值并执行所有必需的初始化。 有关详细信息，请参阅 [proxy_temp_path](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_temp_path) 指令的文档。
- `set_access_slot`— 将提供的参数转换为文件 权限掩码。 有关详细信息，请参阅 [proxy_store_access](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_store_access) 指令的文档。

`conf` 字段定义哪个配置结构是 传递给目录处理程序。 核心模块只有全局配置和设置NJT`_DIRECT_CONF` 标志来访问它。 HTTP、Stream 或 Mail 等模块创建配置层次结构。 例如，为`server`, `location` 和`if`作用域创建模块的配置。

- `NJT_HTTP_MAIN_CONF_OFFSET`— `http`块的配置。
- `NJT_HTTP_SRV_CONF_OFFSET`— http 块内 server 块的配置。
- `NJT_HTTP_LOC_CONF_OFFSET`— http块内 location 块的配置。
- `NJT_STREAM_MAIN_CONF_OFFSET`— stream块的配置。
- `NJT_STREAM_SRV_CONF_OFFSET`— stream块内server块的配置。
- `NJT_MAIL_MAIN_CONF_OFFSET`— mail块的配置。
- `NJT_MAIL_SRV_CONF_OFFSET`— mail块内server块的配置。

`offset` 字段定义了特定指令在对应配置结构中的偏移量， 定义模块中字段的偏移量 保存此特定指令的值的配置结构。 典型用途是使用offsetof()宏。

`post` 字段有两个用途：它可用于定义在主处理程序完成后调用或传递其他数据给主处理程序。 在第一种情况下，`njt_conf_post_t` 结构需要 使用指向处理程序的指针进行初始化，例如：

> static char *njt_do_foo(njt_conf_t *cf, void *post, void *data); static njt_conf_post_t  njt_foo_post = { njt_do_foo };

`post` 参数是`njt_conf_post_t` 对象本身，`data` 参数是指向data的指针， 由主处理程序转换成具有合适的类型。



# 16.调试内存问题

要调试内存问题（如缓冲区溢出或释放后使用错误），请使用一些现代编译器支持的[地址清理器](https://en.wikipedia.org/wiki/AddressSanitizer) （ASan）。 要在 gcc和 `clang`启用 ASan，请执行以下操作： 使用编译器和链接器选项 -fsanitize=address 。 在构建njet时，可以通过在 configure 脚本添加 --with-cc-opt 和 --with-ld-opt 参数来完成。

由于njet中的大多数分配都是从njet内部[池](http://nginx.org/en/docs/dev/development_guide.html#pool)中进行的，因此启用ASan可能并不总是足以进行调试。 内部池从系统中分配一大块内存并切分为小块内存。 但是，可以通过将 NJT_DEBUG_PALLOC 宏设置为1去禁用它 . 在这种情况下，将直接传递给系统分配。

以下配置行总结了上面提供的信息。 建议在开发第三方模块和在不同的平台测试 njet 时使用 。

> ```C
> auto/configure --with-cc-opt='-fsanitize=address -DNJT_DEBUG_PALLOC=1'
>                --with-ld-opt=-fsanitize=address
> ```