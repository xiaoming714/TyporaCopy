# 初识Nginx

## 优点

- 高并发、高性能
- 可扩展性好
- 高可靠性
- 热部署
- BSD许可证

## 主要应用场景

- 静态资源服务
  - 通过本地文件系统提供服务
- 反向代理服务
  - Nginx的强大性能
  - 缓存
  - 负载均衡
- API服务
  - OpenResty

## Nginx为什么出现

- 互联网的数据量快速增长
- 摩尔定律：性能提升
- 低效的Apache

## Nginx的组成

- 二进制可执行文件：由各模块源码编译出的一个文件
- conf配置文件：控制Nginx的行为
- access.log访问日志：记录每一条http请求信息
- error.log错误日志：定位问题

# Nginx架构基础

## 请求处理流程

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221016104156305.png" alt="image-20221016104156305" style="zoom: 50%;" />

## 进程结构

Master进程（Worker进程的管理者，通过信号管理子进程，第三方模块通常不会加入功能到Master）--Worker进程（事件驱动，一个进程对应一个CPU核心，使用CPU核心的缓存，减少缓存失效）--Cache Manager--Cache Loader

为了保证高可用和高可靠所以使用进程而不是线程

## reload的流程

1. 向master进程发送HUP信号（reload命令）
2. master进程校验配置语法是否正确
3. master进程打开新的监听窗口
4. master进程用新配置启动新的worker子进程
5. master进程向老worker子进程发送QUIT信号
6. 老worker进程关闭监听句柄，处理完当前连接后结束进程

reload后master使用新配置启动新的worker，老配置的worker在完成已经存在的连接时优雅退出。

## 热升级的完整流程

1. 将旧的Nginx文件换成新Nginx文件（注意备份）
2. 向master进程发送USR2信号
3. master进程修改pid文件名，加后缀.oldbin
4. master进程用新的Nginx文件启动新的master进程
5. 向老的master进程发送QUIT信号，关闭老的master进程
6. 回滚：向老master发送HUP，向新master发送QUIT

![image-20221016112749258](C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221016112749258.png)

## 优雅关闭worker

1. 设置定时器-worker_shutdown_timeout
2. 关闭监听句柄
3. 关闭空闲连接
4. 在循环中等待全部连接关闭
5. 退出进程

## 网络收发与Nginx事件的对应关系

![image-20221016114355665](C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221016114355665.png)

![image-20221016114544046](C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221016114544046.png)

## epoll的优势和劣势

使用epoll的前提：高并发连接中，每次处理的活跃连接数量占比很小

实现：红黑树、链表

使用：创建、操作、获取句柄、关闭

## Nginx的请求切换

传统服务器一个进程只处理同一个请求，当一个请求条件不满足的时候切换到其他进程处理其他请求，当并发量变大时，进程间切换所消耗的时间不是线性增长的

Nginx在用户态、同一进程直接进行请求的切换

## Nginx模块的分类

每个Nginx必须具备一个ngx_module_t数据结构，其中有一个type变量，说明了这个模块是什么类型

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221031150239060.png" alt="image-20221031150239060" style="zoom: 33%;" />

通过添加coremodule来新增新增子模块

upsteam用来负载均衡向上游传递请求

## Nginx如何通过连接池处理网络请求

默认有一个512大小的数组，每个元素表示一个连接（connection数组），不止用于客户端，也用于反向代理服务器

每一个连接对应一个读事件和一个写事件

连接池64位操作系统中约占232字节

每个事件约位96字节，但是每个连接不仅对应客户端还对应这一个反向代理服务器，所以需要乘二

## 内存池对性能的影响

1. 连接内存池
   1. 通常512字节，因为连接需要保存的上下文信息比较少
2. 请求内存池
   1. http通常分配4k内存， 

## 共享内存-所有worker进程协同工作

- 基础同步工具：信号、共享内存
- 高级通讯方式：锁（自旋锁，要求Nginx模块快速使用共享内存）、Slab内存管理器

限速或流控在共享内存中操作，因为所有worker进程对一个客户端的限制应该相同-红黑树

负载均衡-单链表

lua模块（OpenResty核心模块） 

 ## Slab内存管理器

将整个共享内存切割为小块供红黑树的节点使用，每个页面切割为很多slot，以×2的方式增长

- 最多两倍内存消耗
- 适合小对象
- 避免碎片
- 避免重复初始化

伙伴分配器（buddy allocator）是以页为单位管理和分配内存，但面对字节为单位的需求就比较浪费内存，故产生了Slab内存分配器，它从伙伴分配器分配内存，随后对申请的内存进行管理。

除了分配小内存意外，slab还用作常用对象的缓存，内核中的很多结构，初始化所需要的时间可能大于等于分配内存的时间，故释放对象后，它会保持初始化状态，这样就可以快速分配对象。

SLAB分配器的最后一项任务是提高CPU硬件缓存的利用率。 如果将对象包装到SLAB中后仍有剩余空间，则将剩余空间用于为SLAB着色。 SLAB着色是一种尝试使不同SLAB中的对象使用CPU硬件缓存中不同行的方案。 通过将对象放置在SLAB中的不同起始偏移处，对象可能会在CPU缓存中使用不同的行，从而有助于确保来自同一SLAB缓存的对象不太可能相互刷新。 通过这种方案，原本被浪费掉的空间可以实现一项新功能。

出于 slab 管理的方便，每个 slab 管理的对象大小都是一致的，当我们需要分配一个处于 64-96字节中间大小的对象时，就必须从保存 96 字节的 slab 中分配。而对于专用的 slab，其管理的都是同一个结构体实例，申请一个就给一个恰好内存大小的对象，这就可以充分利用空间。

## Nginx的哈希表

哈希表通常仅用于静态不变的内容

Maxsize为最大容量，Bucketsize涉及到CPU内存对齐，为了防止取两次，所以会和CPUcachelength相同，如果不相同就会向上取整，尽量不要超过64字节

通常用来提高访问效率

## Nginx的红黑树

## Nginx的动态模块

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221105170253634.png" alt="image-20221105170253634" style="zoom:50%;" />

动态模块方便修改

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221105182512532.png" alt="image-20221105182512532" style="zoom:50%;" />

# HTTP模块

## 冲突的配置指令以谁为准

高层和底层模块的配置指令冲突

指令的合并

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221105185236105.png" alt="image-20221105185236105" style="zoom: 33%;" />

- 存储值的指令继承规则：向上覆盖：子配置不存在时，直接使用父配置，子配置存在时，直接覆盖父配置。

## 处理HTTP请求头部

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221107103547832.png" alt="image-20221107103547832" style="zoom:50%;" />

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221107104250196.png" alt="image-20221107104250196" style="zoom: 50%;" />

## Nginx的正则表达式

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221107105145141.png" alt="image-20221107105145141" style="zoom: 50%;" />

## HTTP请求的11个阶段

### 处理请求的全过程

1. Read Request Headers：解析请求头。
2. Identify Configuration Block：识别由哪一个 location 进行处理，匹配 URL。
3. Apply Rate Limits：判断是否限速。例如可能这个请求并发的连接数太多超过了限制，或者 QPS 太高。
4. Perform Authentication：连接控制，验证请求。例如可能根据 Referrer 头部做一些防盗链的设置，或者验证用户的权限。
5. Generate Content：生成返回给用户的响应。为了生成这个响应，做反向代理的时候可能会和上游服务（Upstream Services）进行通信，然后这个过程中还可能会有些子请求或者重定向，那么还会走一下这个过程（Internal redirects and subrequests）。
6. Response Filters：过滤返回给用户的响应。比如压缩响应，或者对图片进行处理。
7. Log：记录日志。 

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221111162000856.png" alt="image-20221111162000856" style="zoom: 50%;" />

### 11个阶段

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221111162145898.png" alt="image-20221111162145898" style="zoom:50%;" />

1. POST_READ：在 read 完请求的头部之后，在没有对头部做任何处理之前，想要获取到一些原始的值，就应该在这个阶段进行处理。这里面会涉及到一个 realip 模块。
2. SERVER_REWRITE：和下面的 REWRITE 阶段一样，都只有一个模块叫 rewrite 模块，一般没有第三方模块会处理这个阶段。
3. FIND_CONFIG：做 location 的匹配，暂时没有模块会用到。
4. REWRITE：对 URL 做一些处理。
5. POST_WRITE：处于 REWRITE 之后，也是暂时没有模块会在这个阶段出现。

接下来是确认用户访问权限的三个模块：

6. PREACCESS：是在 ACCESS 之前要做一些工作，例如并发连接和 QPS 需要进行限制，涉及到两个模块：limt_conn 和 limit_req

7. ACCESS：核心要解决的是用户能不能访问的问题，例如 auth_basic 是用户名和密码，access 是用户访问 IP，auth_request 根据第三方服务返回是否可以去访问。

8. POST_ACCESS：是在 ACCESS 之后会做一些事情，同样暂时没有模块会用到。

最后的三个阶段处理响应和日志：

9. PRECONTENT：在处理 CONTENT 之前会做一些事情，例如会把子请求发送给第三方的服务去处理，try_files 模块也是在这个阶段中。

10. CONTENT：这个阶段涉及到的模块就非常多了，例如 index, autoindex, concat 等都是在这个阶段生效的。

11. LOG：记录日志 access_log 模块。

以上的这些阶段都是严格按照顺序进行处理的，当然，每个阶段中各个 HTTP 模块的处理顺序也很重要，如果某个模块不把请求向下传递，后面的模块是接收不到请求的。而且每个阶段中的模块也不一定所有都要执行一遍，下面就接着讲一下各个阶段模块之间的请求顺序。

### 11个阶段的处理



#### POST_READ阶段：

在 read 完请求的头部之后，在没有对头部做任何处理之前，想要获取到一些原始的值（如拿到IP地址为后续模块提供素材，如限速、限流），就应该在这个阶段进行处理。这里面会涉及到一个 realip 模块。

我们知道，TCP 连接是由一个四元组构成的，在四元组中，包含了源 IP 地址。而在真实的互联网中，存在非常多的正向代理和反向代理。例如最终的用户有自己的内网 IP 地址，运营商会分配一个公网 IP，然后访问某个网站的时候，这个网站可能使用了 CDN 加速一些静态文件或图片，如果 CDN 没有命中，那么就会回源，回源的时候可能还要经过一个反向代理，例如阿里云的 SLB，然后才会到达 Nginx。所以我们需要拿到真实的用户IP来进行后续操作。

HTTP 协议中，有两个头部可以用来获取用户 IP：

- X-Forwardex-For 是用来传递 IP 的，这个头部会把经过的节点 IP 都记录下来
- X-Real-IP：可以记录用户真实的 IP 地址，只能有一个

拿到真实IP后通过变量来使用：例如 binary_remote_addr、remote_addr 这样的变量，其值就是真实的 IP，这样做连接限制也就是 limit_conn 模块才有意义，这也说明了，limit_conn 模块只能在 preaccess 阶段，而不能在 postread 阶段生效。

**realip模块**

- 默认不会编译进Nginx，需要通过--with-http_realip_module指令启用功能
- 如果想要使用原来TCP连接中的地址和端口，需要通过**realip_remote_addr**和**realip_remote_port**来保存
- 功能：修改客户端地址
- 指令
  - **set_real_ip_from**：指定可信的地址，只有从该地址建立的连接，获取的realip才是可信的
  - **real_ip_header**：指定从哪个头部取真实的 IP 地址，默认从 `X-Real-IP` 中取，如果设置从 `X-Forwarded-For` 中取，会先从最后一个 IP 开始取
  - **real_ip_recursive**：环回地址，默认关闭，打开的时候，如果 `X-Forwarded-For` 最后一个地址与客户端地址相同，会过滤掉该地址

#### rewtite阶段

首先 rewrite 阶段分为两个，一个是 server_rewrite 阶段，一个是 rewrite，这两个阶段都涉及到一个 rewrite 模块，而在 rewrite 模块中，有一个 return 指令，遇到该指令就不会再向下执行，直接返回响应。

**rewrite模块return指令**

语法如下

- 返回状态码，后面跟body
- 返回状态码，后面跟URL
- 直接返回URL

状态码分类

- Nginx自定义
  - 444：立即关闭连接，用户收不到响应
- HTTP1.0标准
  - 301：永久重定向
  - 302：临时重定向，禁止被缓存
- HTTP1.1标准
  - 303：临时重定向，允许改变方法，禁止被缓存
  - 307：临时重定向，不允许改变方法，禁止被缓存
  - 308：永久重定向，不允许改变方法

**return指令和error_page**

`error_page` 的作用大家肯定经常见到。当访问一个网站出现 404 的时候，一般不会直接出现一个 404 NOT FOUND，而是会有一个比较友好的页面，这就是 `error_page` 的功能。

**rewrite指令**

`rewrite` 指令用于修改用户传入 Nginx 的 URL，语法规则如下

```bash
Syntax: rewrite regex replacement [flag];
Default: —
Context: server, location, if
```

- 将 `regex` 指定的 URL 替换成 `replacement` 这个新的 URL（可以使用正则表达式及变量提取）
- 当 `replacement` 以 http:// 或者 https:// 或者 $schema 开头，则直接返回 302 重定向
- 替换后的 URL 根据 flag 指定的方式进行处理
  - last：用 `replacement` 这个 URL 进行新的 location 匹配
  - break：break 指令停止当前脚本指令的执行，等价于独立的 break 指令
  - redirect：返回 302 重定向
  - permanent：返回 301 重定向

**if指令**

if 指令也是在 rewrite 阶段生效的，语法规则如下

```bash
Syntax: if (condition) { ... }
Default: —
Context: server, location
```

规则：条件 condition 为真，则执行大括号内的指令；同时还遵循值指令的继承规则

if指令的表达式包含的内容

1. 检查变量为空或者值是否为 0
2. 将变量与字符串做匹配，使用 = 或 !=
3. 将变量与正则表达式做匹配
   - 大小写敏感，~ 或者 !~
   - 大小写不敏感，~~* 或者 !~~*
4. 检查文件是否存在，使用 -f 或者 !-f
5. 检查目录是否存在，使用 -d 或者 !-d
6. 检查文件、目录、软链接是否存在，使用 -e 或者 !-e
7. 检查是否为可执行文件，使用 -x 或者 !-x

#### find_config阶段

当经过 rewrite 模块，匹配到 URL 之后，就会进入 find_config 阶段，开始寻找 URL 对应的 location 配置。语法规则如下：

```bash
Syntax: location [ = | ~ | ~* | ^~ ] uri { ... }
        location @name { ... }
Default: —
Context: server, location

Syntax: merge_slashes on | off;#加入的url中有两个重复的/会合并为一个，默认打开，面对base64编码时才需要关闭
Default: merge_slashes on; 
Context: http, server
```

**匹配规则**

location 的匹配规则是仅匹配 URI，忽略参数，有下面三种大的情况：

- 前缀字符串
  - 常规匹配
  - =：精确匹配
  - ^~：匹配上后则不再进行正则表达式匹配
- 正则表达式
  - ~：大小写敏感的正则匹配
  - ~*：大小写不敏感
- 用户内部跳转的命名 location
  - @

全部的前缀字符串是放置在一棵二叉树中的，Nginx 会分为两部分进行匹配：

1. 先遍历所有的前缀字符串，选取最长的一个前缀字符串，如果这个字符串是 = 的精确匹配或 ^~ 的前缀匹配，会直接使用
2. 如果第一步中没有匹配上 = 或 ^~，那么会先记住最长匹配的前缀字符串 location
3. 按照 nginx.conf 文件中的配置依次匹配正则表达式
4. 如果所有的正则表达式都没有匹配上，那么会使用最长匹配的前缀字符串

#### preaccess阶段

本阶段作用为限制每个客户端的并发连接数，限制访问频率

**limit_conn模块（限制每个客户端并发连接数）**

模块名为ngx_http_limit_conn_module

- 生效阶段：`NGX_HTTP_PREACCESS_PHASE` 阶段
- 模块：`http_limit_conn_module`
- 默认编译进 Nginx，通过 `--without-http_limit_conn_module` 禁用
- 生效范围
  - 全部 worker 进程（基于共享内存）
  - 进入 preaccess 阶段前不生效
  - 限制的有效性取决于 key 的设计：依赖 postread 阶段的 realip 模块取到真实 IP（通常key为用户真实IP）

指令语法

```bash
#定义共享内存（包括大小），以及 key 关键字
Syntax: limit_conn_zone key zone=name:size;#key通常为用户的ip地址
Default: —
Context: http
#限制并发连接数
Syntax: limit_conn zone number;
Default: —
Context: http, server, location
#限制发生时的日志级别
Syntax: limit_conn_log_level info | notice | warn | error;
Default: limit_conn_log_level error; 
Context: http, server, location
#限制发生时向客户端返回的错误码
Syntax: limit_conn_status code;
Default: limit_conn_status 503; 
Context: http, server, location
```



**limit_req（限制访问频率）**

 模块名`ngx_http_limit_req_module`，它的基本特性如下：

- 生效阶段：`NGX_HTTP_PREACCESS_PHASE` 阶段
- 模块：`http_limit_req_module`
- 默认编译进 Nginx，通过 `--without-http_limit_req_module` 禁用
- 生效算法：leaky bucket 算法
- 生效范围
  - 全部 worker 进程（基于共享内存）
  - 进入 preaccess 阶段前不生效

leaky bucket 叫漏桶算法（小学接水问题，出水口固定），其他用来限制请求速率的还有令牌环算法等

指令语法

```bash
#定义共享内存（包括大小），以及 key 关键字和限制速率
Syntax: limit_req_zone key zone=name:size rate=rate ;#rate 单位为 r/s 或者 r/m（每分钟或者每秒处理多少个请求）
Default: —
Context: http
#限制并发连接数
Syntax: limit_req zone=name [burst=number] [nodelay];#burst为盆中可以容纳多少个请求，如果没有burst则不会放入盆中直接返回错误
#添加nodelay请求在没有达到burst限制之前都可以被处理并返回，超出burst限制之后才会返回503
Default: —
Context: http, server, location
#限制发生时的日志级别
Syntax: limit_req_log_level info | notice | warn | error;
Default: limit_req_log_level error; 
Context: http, server, location
#限制发生时向客户端返回的错误码
Syntax: limit_req_status code;
Default: limit_req_status 503; 
Context: http, server, location
```

limit_req 与 limit_conn 配置同时生效时，limit_req优先级高，因为req在conn之前

#### access阶段

控制请求是否可以继续向下访问

**access模块**

模块名 `ngx_http_access_module`，它的基本特性如下：

- 生效阶段：`NGX_HTTP_ACCESS_PHASE` 阶段
- 模块：`http_access_module`
- 默认编译进 Nginx，通过 `--without-http_access_module` 禁用
- 生效范围
  - 进入 access 阶段前不生效

指令语法

```bash
Syntax: allow address | CIDR | unix: | all;
Default: —
Context: http, server, location, limit_except

Syntax: deny address | CIDR | unix: | all;
Default: —
Context: http, server, location, limit_except
#如果多条指令对同一个ip生效，执行第一个，不会继续向下执行
```

**auth_basic模块**

auth_basic 模块是用作用户认证的，当开启了这个模块之后，我们通过浏览器访问网站时，就会返回一个 401 Unauthorized，当然这个 401 用户不会看见，浏览器会弹出一个对话框要求输入用户名和密码。这个模块使用的是 RFC2617 中的定义。

指令语法

- 基于 HTTP Basic Authutication 协议进行用户密码的认证
- 默认编译进 Nginx
  - –without-http_auth_basic_module
  - disable ngx_http_auth_basic_module

```bash
Syntax: auth_basic string | off;
Default: auth_basic off; 
Context: http, server, location, limit_except

Syntax: auth_basic_user_file file;
Default: —
Context: http, server, location, limit_except
#可以使用htpasswd生成密码文件，auth_basic_user_file就以来这个密码文件（htpasswd依赖httpd-tools）
#生成密码的命令htpasswd –c file –b user pass
```

**auth_request**

- 功能：向上游的服务转发请求，若上游服务返回的响应码是 2xx，则继续执行，若上游服务返回的响应码是 2xx，则继续执行，若上游服务返回的是 401 或者 403，则将响应返回给客户端
- 原理：收到请求后，生成子请求，通过反向代理技术把请求传递给上游服务
- 默认未编译进 Nginx，需要通过 –with-http_auth_request_module 编译进去

指令语法

```bash
Syntax: auth_request uri | off;
Default: auth_request off; 
Context: http, server, location

Syntax: auth_request_set $variable value;
Default: —
Context: http, server, location
#这个配置文件中，/ 路径下会将请求转发到另外一个服务中去，可以用 nginx 再搭建一个服务
#如果这个服务返回 2xx，那么鉴权成功，如果返回 401 或 403 则鉴权失败
```

**对不同access模块进行限制的satisfy指令**

指令语法

```bash
Syntax: satisfy all | any;
Default: satisfy all; 
Context: http, server, location
#如果satisfy指令的值是all的话，就表示必须所有access阶段的模块都要执行，都通过了才会放行；值是 any 的话，表示有任意一个模块得到执行即可。
```

1. 如果有return指令access阶段就不会执行，因为return指令属于rewrite阶段
2. 多个access模块的顺序有影响
3. 如果把 deny all 提到 auth_basic 之前，依然可以，因为各个模块执行顺序和指令的顺序无关。
4. 如果改为 allow all，有机会输入密码吗？

   没有机会，因为 allow all 是 access 模块，先于 auth_basic 模块执行

#### precontent阶段

**try_files模块**

指令语法

```bash
Syntax: try_files file ... uri;
        try_files file ... =code;
Default: —
Context: server, location
```

- 模块：`ngx_http_try_files_module` 模块
- 依次试图访问多个 URL 对应的文件（由 root 或者 alias 指令指定），当文件存在时，直接返回文件内容，如果所有文件都不存在，则按照最后一个 URL 结果或者 code 返回

**mirror模块**

mirror 模块可以实时拷贝流量，这对于需要同时访问多个环境的请求是非常有用的。

指令语法

```bash
Syntax: mirror uri | off;
Default: mirror off; 
Context: http, server, location

Syntax: mirror_request_body on | off;
Default: mirror_request_body on; 
Context: http, server, location
```

- 模块：

  ```
  ngx_http_mirror_module
  ```

  模块，默认编译进 Nginx

  - 通过 –without-http_mirror_module 移除模块

- 功能：处理请求时，生成子请求访问其他服务，对子请求的返回值不做处理

#### content阶段

**static模块**

root和alias指令

```bash
Syntax: alias path;
Default: —
Context: location

Syntax: root path;
Default: root html; 
Context: http, server, location, if in location
```

- 功能：将 URL 映射为文件路径，以返回静态文件内容
- 差别：root 会将完整 URL 映射进文件路径中，alias 只会将 location 后的 URL 映射到文件路径

**index模块**

- 模块：`ngx_http_index_module`

- 功能：指定 `/` 结尾的目录访问时，返回 index 文件内容

- 语法：

  ```bash
  Syntax: index file ...;
  Default: index index.html; 
  Context: http, server, location
  ```

- 先于 autoindex 模块执行

这个模块，当我们访问以 `/` 结尾的目录时，会去找 root 或 alias 指令的文件夹下的 index.html，如果有这个文件，就会把文件内容返回，也可以指定其他文件。

**autoindex 模块**

- 模块：`ngx_http_autoindex_module`，默认编译进 Nginx，使用 `--without-http_autoindex_module` 取消

- 功能：当 URL 以 `/` 结尾时，尝试以 html/xml/json/jsonp 等格式返回 root/alias 中指向目录的目录结构

- 语法：

  ```bash
  # 开启或关闭
  Syntax: autoindex on | off;
  Default: autoindex off; 
  Context: http, server, location
  # 当以 HTML 格式输出时，控制是否转换为 KB/MB/GB
  Syntax: autoindex_exact_size on | off;
  Default: autoindex_exact_size on; 
  Context: http, server, location
  # 控制以哪种格式输出
  Syntax: autoindex_format html | xml | json | jsonp;
  Default: autoindex_format html; 
  Context: http, server, location
  # 控制是否以本地时间格式显示还是 UTC 格式
  Syntax: autoindex_localtime on | off;
  Default: autoindex_localtime off; 
  Context: http, server, location
  ```

**concat模块**

- 模块：ngx_http_concat_module

- 模块开发者：Tengine(https://github.com/alibaba/nginx-http-concat) –add-module=../nginx-http-concat/

- 功能：合并多个小文件请求，可以明显提升 HTTP 请求的性能

- 指令：

  ```bash
  #在 URI 后面加上 ??，通过 ”,“ 分割文件，如果还有参数，则在最后通过 ? 添加参数
  concat on | off
  default concat off
  Context http, server, location
  
  concat_types MIME types
  Default concat_types: text/css application/x-javascript
  Context http, server, location
  
  concat_unique on | off
  Default concat_unique on
  Context http, server, location
  
  concat_max_files numberp
  Default concat_max_files 10
  Context http, server, location
  
  concat_delimiter string
  Default NONE
  Context http, server, locatione
  concat_ignore_file_error on | off
  Default off
  Context http, server, location
  ```

#### log阶段

记录请求访问日志

- 功能：将 HTTP 请求相关信息记录到日志
- 模块：`ngx_http_log_module`，无法禁用

access 日志格式

```bash
Syntax: log_format name [escape=default|json|none] string ...;
Default: log_format combined "..."; 
Context: http
```

默认的 combined 日志格式：

```
log_format combined '$remote_addr - $remote_user [$time_local] ' 
'"$request" $status $body_bytes_sent ' '"$http_referer" 
"$http_user_agent"';
```

配置日志文件路径

```
Syntax: access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
        access_log off;
Default: access_log logs/access.log combined; 
Context: http, server, location, if in location, limit_except
```

- path 路径可以包含变量：不打开 cache 时每记录一条日志都需要打开、关闭日志文件

- if 通过变量值控制请求日志是否记录

- 日志缓存

  - 功能：批量将内存中的日志写入磁盘

  - 写入磁盘的条件：

    所有待写入磁盘的日志大小超出缓存大小；

    达到 flush 指定的过期时间；

    worker 进程执行 reopen 命令，或者正在关闭。

- 日志压缩

  - 功能：批量压缩内存中的日志，再写入磁盘
  - buffer 大小默认为 64KB
  - 压缩级别默认为 1（1最快压缩率最低，9最慢压缩率最高）
  - 打开日志压缩时，默认打开日志缓存功能

对日志文件名包含变量时的优化

```
Syntax: open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];
        open_log_file_cache off;
Default: open_log_file_cache off; 
Context: http, server, location
```

- max：缓存内的最大文件句柄数，超出后用 LRU 算法淘汰
- inactive：文件访问完后在这段时间内不会被关闭。默认 10 秒
- min_uses：在 inactive 时间内使用次数超过 min_uses 才会继续存在内存中。默认 1
- valid：超出 valid 时间后，将对缓存的日志文件检查是否存在。默认 60 秒
- off：关闭缓存功能

## 过滤模块

### 过滤模块的位置

之前我们介绍了 Nginx 的 11 个阶段，在 content 阶段时，Nginx 会生成返回给用户的响应内容，对用户的响应内容，实际上还需要做再加工处理，Nginx 的过滤模块就是对响应内容进行再加工处理的。所以实际上过滤模块位于 content 阶段之后，log 阶段之前。

我们先来看一段配置指令：

```bash
limit_req zone=req_one
burst=120;
limit_conn c_zone 1;

satisfy any;
allow 192.168.1.0/32;
auth_basic_user_file access.pass;

gzip on;
image_filter resize 80 80;
```

那么在这一段配置指令之下，会遵循怎样的请求流程呢？请看一下下面这张图：
[<img src="http://ww1.sinaimg.cn/large/c552abe7ly1gf0wjcb6tsj21qx2bc4f8.jpg" alt="img" style="zoom:25%;" />](http://ww1.sinaimg.cn/large/c552abe7ly1gf0wjcb6tsj21qx2bc4f8.jpg)

上面这张图的流程大致说一下，如果对于 Nginx 的 11 个阶段不了解的去翻一下之前的文章。

我这里再简单说一下。首先由 Nginx 框架接收 HTTP 请求，经过preaccess、access、content 阶段的处理，当经过 static 模块之后生成响应的时候，很多时候需要对响应进行处理，然后才会返回给客户端。

这里我们假如响应是一张图片的话，那么需要做缩略图的时候，首先就要经过 image_filter 模块的处理。这里面还有一个 gzip 模块，这两个模块也是需要遵循严格的顺序的。因为如果先做 gzip 压缩的话，缩略图后面就没办法做了。

第二个需要关注的地方是，首先对 header 进行过滤，再对 body 进行过滤。因为我们在对用户发送响应的时候，一定是先发送 header，然后再发送 body，所以所有的过滤模块都会提供对 header 或 body 的过滤，当然 image_filter 和 gzip 模块对这两者都可以过滤。

### 返回响应

前面我们说过，Nginx 的 11 个阶段是有严格的顺序的，而这个顺序是在 Nginx 的代码中以一个数组的形式存在的，这个数组的顺序是从后往前。在给用户返回响应的时候，过滤模块也是有严格顺序的，这个顺序同样是从后往前。来看一下在代码中的定义，标红的是我下面会提到的几个过滤模块。

[<img src="https://s3plus.meituan.net/v1/mss_f32142e8d47149129e9550e929704625/yzz-test-image/20200522082225" alt="img" style="zoom:33%;" />](https://s3plus.meituan.net/v1/mss_f32142e8d47149129e9550e929704625/yzz-test-image/20200522082225)

我们需要重点关注四个过滤模块，它们分别的作用是

- copy_filter：复制包体内容

  当我们使用 sendfile 指令的时候，也就是零拷贝技术，不经过用户态内存，这里就是不经过 Nginx 直接发给用户，同时也用了 gzip 模块的时候，gzip 是必须在 copy_filter 模块之后的，因为 gzip 必须对内存中的数据做压缩，这时 copy_filter 就会让 sendfile 指令失效。有些模块不需要对内存中的数据进行处理，就需要在 copy_filter 模块之前进行处理。

- postpone_filter：处理子请求

  用来处理子请求，有些过滤模块需要关心子请求的处理结果，需要放在该模块之后。

- header_filter：构造响应头部

  用来构造最终发送给用户的响应头部，可能会添加一些 Server，Nginx 版本号等内容。

- write_filter：发送响应

  用来实际调用操作系统的 write 或 send 等系统调用，来把响应实际发送出去。

## addition 模块

下面再来看一个过滤模块，addition 模块，它可以在响应的前后添加内容。

- 功能：在相应前或者响应后增加内容，增加内容的方式，是通过新增子请求，根据子请求的响应来完成。

- 模块：`ngx_http_addition_filter_module`

  默认未编译进 Nginx，通过 –with-http_addition_module 启用

### 指令

```
Syntax: add_before_body uri;
Default: —
Context: http, server, location

Syntax: add_after_body uri;
Default: —
Context: http, server, location

Syntax: addition_types mime-type ...;
Default: addition_types text/html; 
Context: http, server, location
```

这里的三个指令都比较简单，说一下 `add_before_body` 和 `add_after_body` 后面的 `uri`，这个的意思是说，向指定 `uri` 发起子请求，根据子请求的响应来添加内容。

`addition_types` 指令是指定要添加的文件类型。

## 变量

### Nginx 变量的运行原理

围绕 Nginx 中的变量模块可以分为两类，一类是提供变量的模块，另外一类是使用变量的模块。

- 提供变量的模块
  - 在 Preconfiguration 源代码中定义变量名以及可以解析出变量的方法
- 使用变量的模块
  - 解析 nginx.conf 时定义变量的使用方式

也就是在 Nginx 启动时，已经定义了变量，而只有当真正处理请求的时候，才会根据 nginx.conf 解析出来的变量使用方式调用 Preconfiguration 中定义的方法来实际获取值。

这也是变量的两个特性：

- 惰性求值：只有使用的时候才会去调方法解析
- 变量值可以时刻变化，其值为使用的那一时刻的值。例如发送响应包体字节数，实际在发送的过程中是一直在变化的。

除了 Nginx 的模块之外，Nginx 框架也包含许多的变量，这些变量不需要通过编译模块来引入，而且，Nginx 框架所提供的变量往往反映了处理请求的细节，因此，了解 Nginx 框架所提供的变量是十分有必要的。

### HTTP 请求相关的变量

先来看一下关于 HTTP 请求的相关变量。

- arg_参数名：URL 中某个具体参数的值

- query_string：与 args 变量完全相同

- args：全部 URL 参数

- is_args：如果请求 URL 中有参数则返回 ?，否则返回空

- content_length：HTTP 请求中标识包体长度的 Content-Length 头部的值。如果请求中没有携带这个参数，那么就取不到对应的值。

- content_type：标识请求包体类型的 Content-Type 头部的值。同样需要用户请求中携带对应的参数。

- uri：请求的 URI（不同于 URL，不包括 ? 后的参数）

- document_uri：与 uri 完全相同。由于历史原因而存在的。

- request_uri：请求的 URL（包括 URI 以及完整的参数）

- scheme：协议名，例如 HTTP 或者 HTTPS

- request_method：请求方法，例如 GET 或者 POST

- request_length：所有请求内容的大小，包括请求行、头部、包体等

- remote_user：由 HTTP Basic Authentication 协议传入的用户名

- request_body_file：很多时候会将用户请求的包体存放到文件中，这个变量就是临时存放请求包体的文件

  - 如果包体非常小则不会存文件
  - client_body_in_file_only 指令强制所有包体存入文件，且可决定是否删除

- request_body：请求中的包体，这个变量当且仅当使用反向代理，且设定用内存暂存包体时才有效

- request：原始的 URL 请求，含有方法与协议版本，例如 GET /?a=1&b=22 HTTP/1.1

- host

  - 先从请求行中获取
  - 如果含有 Host 头部，则用其值替换掉请求行中的主机名
  - 如果前两者都取不到，则使用匹配上的 server_name

- http_头部名字：返回一个具体请求头部的值

  特殊变量，这些变量会做一些处理。

  - http_host
  - http_user_agent
  - http_referer
  - http_via
  - http_x_forwarded_for
  - http_cookie

  通用变量，除了以上的变量，都可以取到对应的值。

### TCP 连接相关的变量

下面是关于 TCP 连接的变量。

- binary_remote_addr：客户端地址的整形格式，对于 IPv4 是 4 字节，对于 IPv6 是 16 字节，所以**在 limit_req 和 limit_conn 中通常可以用作 key** （详见：[Nginx 处理 HTTP 请求的 11 个阶段](https://iziyang.github.io/2020/04/12/5-nginx/) 中的 preaccess 阶段）
- connection：递增的连接序号
- connection_requests：当前连接上执行过的请求数，对 keepalive 连接有意义
- remote_addr：客户端地址
- remote_port：客户端端口
- proxy_protocol_addr：若使用了 proxy_protocol 协议，则返回协议中的地址，否则返回空
- proxy_protocol_port：若使用了 proxy_protocol 协议则返回协议中的端口，否则返回空
- server_addr：服务端地址
- server_port：服务器端端口
- TCP_INFO：TCP 内核层参数，包括 $tcpinfo_rtt, $tcpinfo_rttvar,$tcpinfo_snd_cwnd, $tcpinfo_rcv_space 
- server_protocol：服务器端协议，例如 HTTP/1.1

### Nginx 处理请求过程中产生的变量

Nginx 处理 HTTP 请求的过程中也会产生很多变量。

- request_time：请求处理到现在的耗时，单位为秒，精确到毫秒
- server_name：匹配上请求的 server_name 值
- https：如果开启了 TLS/SSL 则返回 on，否则返回空
- request_completion：若请求处理完则返回 OK，否则返回空
- request_id：以 16 进制输出的请求表示 id，该 id 共含有 16 个字节，是随机生成的
- request_filename：待访问文件的完整路径
- document_root：由 URI 和 root、alias 规则生成的文件夹路径
- realpath_root：将 document_root 中的软链接等换成真实路径
- limit_rate：返回客户端响应时的速度上限，单位为每秒字节数。可以通过 set 指令修改对请求产生的效果

### 发送 HTTP 响应时相关的变量

- body_bytes_sent：响应中 body 包体的长度

- bytes_sent：全部 http 响应的长度

- status：http 响应中的返回码

- sent_trailer_名字：把响应结尾内容里的值返回

- sent_http_头部名字：响应中某个具体头部的值

  特殊处理，下面这些变量需要经过特殊处理：

  - sent_http_content_type
  - sent_http_content_length
  - sent_http_location
  - sent_http_last_modified
  - sent_http_connection
  - sent_http_keep_alive
  - sent_http_transfer_encoding
  - sent_http_cache_control
  - sent_http_link

  通用：除了上面这些头部，其他的头部都是通用型的，也就是可以直接拿来用。

### Nginx 系统变量

- time_local：以本地时间标准输出的当前时间，例如 14/Nov/2018:15:55:37 +0800
- time_iso8601：使用 ISO8601 标准输出的当前时间，例如 2018-11-14T15:55:37+08:00
- nginx_version：Nginx 版本号
- pid：所属 worker 进程的进程 id
- pipe：使用了管道则返回 p，否则返回 .
- hostname：所在服务器的主机名，与 hostname 命令输出一致
- msec：1970 年 1 月 1 日到现在的时间，单位为秒，小数点后精确到毫秒

## 防盗链模块

### 场景

说到盗链就要说一个 HTTP 协议的 头部，referer 头部。当其他网站通过 URL 引用了你的页面，用户在浏览器上点击 URL 时，HTTP 请求的头部会通过 referer 头部将该网站当前页面的 URL 带上，告诉服务器本次请求是由谁发起的。

### referer模块

- 默认编译进 Nginx，通过 `--without-http_referer_module` 禁用

referer 模块有三个指令

```
Syntax: valid_referers none | blocked | server_names | string ...;
Default: —
Context: server, location

Syntax: referer_hash_bucket_size size;
Default: referer_hash_bucket_size 64; 
Context: server, location

Syntax: referer_hash_max_size size;
Default: referer_hash_max_size 2048; 
Context: server, location
```

- `valid_referers` 指令，配置是否允许 referer 头部以及允许哪些 referer 访问。
- `referer_hash_bucket_size` 表示这些配置的值是放在哈希表中的，指定哈希表的大小。
- `referer_hash_max_size` 则表示哈希表的最大大小是多大。

这里面最重要的是 `valid_referers` 指令，需要重点来说明一下。

### valid_referers 指令

可以同时携带多个参数，表示多个 referer 头部都生效。

**参数值**

- none
  - 允许缺失 referer 头部的请求访问
- block：允许 referer 头部没有对应的值的请求访问。例如可能经过了反向代理或者防火墙
- server_names：若 referer 中站点域名与 server_name 中本机域名某个匹配，则允许该请求访问
- string：表示域名及 URL 的字符串，对域名可在前缀或者后缀中含有 * 通配符，若 referer 头部的值匹配字符串后，则允许访问
- 正则表达式：若 referer 头部的值匹配上了正则，就允许访问

**invalid_referer 变量**

- 允许访问时变量值为空
- 不允许访问时变量值为 1

## secure_link 模块

referer 模块是一种简单的防盗链手段，必须依赖浏览器发起请求才会有效，如果攻击者伪造 referer 头部的话，这种方式就失效了。

secure_link 模块是另外一种解决的方案。

它的主要原理是，通过验证 URL 中哈希值的方式防盗链。

基本过程是这个样子的：

- 由服务器（可以是 Nginx，也可以是其他 Web 服务器）生成加密的安全链接 URL，返回给客户端
- 客户端使用安全 URL 访问 Nginx（下载服务器），由 Nginx 的 secure_link 变量验证是否通过

原理如下：

- 哈希算法是不可逆的
- 客户端只能拿到执行过哈希算法的 URL
- 仅生成 URL 的服务器，验证 URL 是否安全的 Nginx，这两者才保存原始的字符串
- 原始字符串通常由以下部分有序组成：
  - 资源位置。如 HTTP 中指定资源的 URI，防止攻击者拿到一个安全 URI 后可以访问任意资源
  - 用户信息。如用户的 IP 地址，限制其他用户盗用 URL
  - 时间戳。使安全 URL 及时过期
  - 密钥。仅服务器端拥有，增加攻击者猜测出原始字符串的难度

模块：

- ngx_http_secure_link_module
  - 未编译进 Nginx，需要通过 –with-http_secure_link_module 添加
- 变量
  - secure_link
  - secure_link_expires

```
Syntax: secure_link expression;
Default: —
Context: http, server, location

Syntax: secure_link_md5 expression;
Default: —
Context: http, server, location

Syntax: secure_link_secret word;
Default: —
Context: location
```

### 变量值及带过期时间的配置示例

- secure_link
  - 值为空字符串：验证不通过
  - 值为 0：URL 过期
  - 值为 1：验证通过
- secure_link_expires
  - 时间戳的值

**命令行生成安全链接**

- 生成 md5

```
echo -n '时间戳URL客户端IP密钥' | openssl md5 -binary | openssl base64 | tr +/ - | tr -d =
```

- 构造请求 URL

```
/test1.txt?md5=md5生成值&expires=时间戳（如 2147483647）
```

**Nginx 配置**

- secure_link $arg_md5,$arg_expires;
  - secure_link 后面必须跟两个值，一个是参数中的 md5，一个是时间戳
- secure_link_md5 "$secure_link_expires$uri$remote_addr secret";
  - 按照什么样的顺序构造原始字符串

### 仅对 URI 进行哈希的简单办法

除了上面这种相对复杂的方式防盗链，还有一种相对简单的防盗链方式，就是只对 URI 进行哈希，这样当 URI 传

- 将请求 URL 分为三个部分：/prefix/hash/link
- Hash 生成方式：对 “link 密钥” 做 md5 哈希
- 用 `secure_link_secret secret;` 配置密钥

**命令行生成安全链接**

- 原请求
  - link
- 生成的安全请求
  - /prefix/md5/link
- 生成 md5
  - `echo -n 'linksecret' | openssl md5 –hex`

**Nginx 配置**

- `secure_link_secret secret;`

这个防盗链的方法比较简单，那么具体是怎么用呢？大家都在网上下载过资源对吧，不管是电子书还是软件，很多网站你点击下载的时候往往会弹出另外一个页面去下载，这个新的页面其实就是请求的 Nginx 生成的安全 URL。如果这个 URL 被拿到的话，其实还是可以用的，所以需要经常的更新密钥来确保 URL 不会被盗用。

之前的两篇文章 [Nginx 变量介绍](https://iziyang.github.io/2020/06/09/7-nginx/)以及利用 [Nginx 变量做防盗链](https://iziyang.github.io/2020/06/09/8-nginx/) 讲的是 Nginx 有哪些变量以及一个常见的应用。那么如此灵活的 Nginx 怎么能不支持自定义变量呢，今天的文章就来说一下自定义变量的几个模块以及 Nginx 的 keepalive 特性。

## 自定义变量

### 通过映射新变量提供更多的可能性：map 模块

- 功能：基于已有变量，使用类似 switch {case: … default: …} 的语法创建新变量，为其他基于变量值实现功能的模块提供更多的可能性
- 模块：`ngx_http_map_module` 默认编译进 Nginx，通过 `--without-http_map_module` 禁用

#### 指令

```bash
Syntax: map string $variable { ... }
Default: —
Context: http

Syntax: map_hash_bucket_size size;
Default: map_hash_bucket_size 32|64|128; 
Context: http

Syntax: map_hash_max_size size;
Default: map_hash_max_size 2048; 
Context: http
```

我们主要看一下 `map string $variable { ... }` 这个指令。所谓类似 switch case 的语法是指，string 的值可以有多个，可以根据 string 值的不同，来给 $variable 赋不同的值。

#### 规则

- 已有变量：string 需要是已有的变量，可以分为下面这三种情况

  - 字符串
  - 一个或者多个变量
  - 变量与字符串的组合

- case 规则：{…} 内的匹配规则需要遵循以下规则，尤其是要注意当使用 hostnames 指令时，与 server name 的匹配规则是一致的，可以看之前的文章

   

  Nginx 的配置指令

  - 字符串严格匹配
  - 使用 hostnames 指令，可以对域名使用前缀 * 泛域名匹配
  - ~ 和 ~* 正则表达式匹配，后者忽略大小写

- default 规则

  - 没有匹配到任何规则时，使用 default
  - 确实 default 时，返回空字符串给新变量

- 其他

  - 使用 include 语法提升可读性
  - 使用 volatile 禁止变量值缓存

大家看到上面这些规则可能都有些晕，废话不多说，直接来看一个实战配置文件就懂了。

#### 实战

这里我们有一个配置文件，在这个文件里面我们定义了两个 map 块，分别配置了两个变量，$name 和 $mobile，$name 中包含 hostnames 指令。

```
map $http_host $name {
    hostnames;

    default       0;

    ~map\.ziyang\w+\.org.cn 1;
    *.ziyang.org.cn   2;
    map.ziyang.com   3;
    map.ziyang.*    4;
}

map $http_user_agent $mobile {
    default       0;
    "~Opera Mini" 1;
}

server {
	listen 10001;
	default_type text/plain;
	location /{
		return 200 '$name:$mobile\n';
	}
}
```

下面看一下实际的请求：

```
➜  test_nginx curl -H "Host: map.ziyang.org.cn" 127.0.0.1:10001
2:0
```

为什么会返回 2:0 呢？我们来看一下匹配顺序。

map.ziyang.org.cn 有三个规则可以生效，分别是：

- ~map.ziyang\w+.org.cn 1;
- *.ziyang.org.cn 2;
- map.ziyang.* 4;

而泛域名是优先于正则表达式的，* 在前的泛域名优先于在后面的泛域名，因此最终匹配到的就是：

- *.ziyang.org.cn 2;

而第二个变量 $mobile 自然走的是 default 规则，不用多说。

这就是 map 模块的作用，大家可以多尝试一下。

下面再来看一个与 map 模块有点类似的 split_clients 模块，这个模块也是通过生成新的变量来完成 AB 测试功能的，它可以按照变量的值，按照百分比的方式，生成新的变量。

### 实现 AB 测试：split_clients 模块

- 功能：基于已有变量创建新变量，为其他 AB 测试提供更多的可能性
  - 对已有变量的值执行 MurmurHash2 算法，得到 32 位整形哈希数字，记为 hash
  - 32 位无符号整形的最大数字 2^32-1，记为 max
  - 哈希数字与最大数字相除，hash/max，可以得到百分比 percent
  - 配置指令中指示了各个百分比构成的范围，如 0-1%，1%-5% 等，及范围对应的值
  - 当 percent 落在哪个范围里，新变量的值就对应着其后的参数
- 模块：`ngx_http_split_clients_module`，默认编译进 Nginx，通过 `--without-http_split_clients_module` 禁用

#### 规则

- 已有变量
  - 字符串
  - 一个或者多个变量
  - 变量与字符串的组合
- case 规则：
  - xx.xx%，支持小数点后 2 位，所有项的百分比相加不能超过 100%
  - *，由它匹配剩余的百分比（100% 减去以上所有项相加的百分比）

#### 指令

```
Syntax: split_clients string $variable { ... }
Default: —
Context: http
```

split_clients 的指令与 map 是非常相似的，可以看一下前面的介绍，这里不再赘述了。

下面这个配置，来看下有没有啥问题：

```
split_clients "${http_testcli}" $variant {
    0.51% .one;
    20.0% .two;
    50.5% .three;
    40% .four;
    * "";
}
```

细心的同学可能已经发现了，所有的百分比相加已经超过了 100%，所以 Nginx 直接会抛出一个错误，禁止执行。

```
➜  test_nginx ./sbin/nginx -s reload
nginx: [emerg] percent total is greater than 100% in /Users/mtdp/myproject/nginx/test_nginx/conf/example/17.map.conf:31
```

然后将 `40% .four;` 这一行给屏蔽掉再试试看：

```
➜  test_nginx curl -H "testcli: split_clients.ziyang.com" --resolve "split_clients.ziyang.com:80:127.0.0.1" http://split_clients.ziyang.com
ABtestfile.three
```

正常执行。

### geo 模块

geo 模块与前面两个模块也很相似，不同之处在于，这个模块是基于 IP 地址或者子网掩码这样的变量值来生成新的变量的。

- 功能：根据 IP 地址创建新变量
- 模块：`ngx_http_geo_module`，默认编译进 Nginx，通过 `--without-http_geo_module` 禁用
- 指令

```
Syntax: geo [$address] $variable { ... }
Default: —
Context: http
```

#### 规则

- 如果 geo 指令后不输入 $address，那么默认使用 $remote_addr 变量作为 IP 地址

- {} 内的指令匹配：优先最长匹配

  - 通过 IP 地址及子网掩码的方式，定义 IP 范围，当 IP 地址在范围内时新变量使用其后的参数值
  - default 指定了当以上范围都未匹配上时，新变量的默认值
  - 通过 proxy 指令指定可信地址（参考 realip 模块），此时 remote_addr 的值为 X-Forwarded-For 头部值中最后一个 IP 地址
  - proxy_recursive 允许循环地址搜索
  - include，优化可读性
  - delete 删除指定网络

  ```
  geo $country {
        default ZZ;
        #include conf/geo.conf;
        #proxy 172.18.144.211; 
        127.0.0.0/24 US;
        127.0.0.1/32 RU;
        10.1.0.0/16 RU;
        192.168.1.0/24 UK;
    }
  ```

问题：以下命令执行时，变量 country 的值各为多少？（proxy 实际上为客户端地址，这里设置为本机的局域网地址即可，我这里是 172.18.144.211）

```
curl -H 'X-Forwarded-For: 10.1.0.0,127.0.0.2' geo.ziyang.com
curl -H 'X-Forwarded-For: 10.1.0.0,127.0.0.1' geo.ziyang.com
curl -H 'X-Forwarded-For: 10.1.0.0,127.0.0.1,1.2.3.4' geo.ziyang.com
```

结果如下：

```
➜  test_nginx curl -H 'X-Forwarded-For: 10.1.0.0,127.0.0.2' geo.ziyang.com
US
➜  test_nginx curl -H 'X-Forwarded-For: 10.1.0.0,127.0.0.1' geo.ziyang.com
RU
➜  test_nginx curl -H 'X-Forwarded-For: 10.1.0.0,127.0.0.1,1.2.3.4' geo.ziyang.com
ZZ
```

这里可以看出来，匹配规则实际上是遵循最长匹配的规则的。

### geoip 模块

geoip 模块可以根据 IP 地址生成对应的地址变量，用法与前面的也都类似，Nginx 是基于 MaxMind 数据库来生成对应的地址的。

- 功能：根据 IP 地址创建新变量
- 模块：`ngx_http_geoip_module`，默认未编译进 Nginx，通过 `--with-http_geoip_module` 禁用

使用这个模块是需要安装 MaxMind 库的，安装步骤如下：

- 安装 MaxMind 里 geoip 的 C 开发库（https://dev.maxmind.com/geoip/legacy/downloadable/ ）
- 编译 Nginx 时带上 `--with-http_geoip_module` 参数
- 下载 MaxMind 中的二进制地址库，这个地址库是需要在指令中指定对应的地址的
- 使用 geoip_country 或者 geoip_city 指令配置好 nginx.conf
- 运行或者升级 Nginx

#### geoip_country 指令提供的变量

**指令**

```
Syntax: geoip_country file; # 指定国家类的地址文件
Default: —
Context: http

Syntax: geoip_proxy address | CIDR;
Default: —
Context: http
```

**变量**

- $geoip_country_code：两个字母的国家代码，比如 CN 或者 US
- $geoip_country_code3：三个字母的国家代码，比如 CHN 或者 USA
- $geoip_country_name：国家名称，例如 “China”, “United States”

#### geoip_city 指令提供的变量

**指令**

```
Syntax: geoip_city file;
Default: —
Context: http
```

**变量**

- $geoip_latitude：纬度
- $geoip_longitude：经度
- $geoip_city_continent_code：位于全球哪个洲，例如 EU 或 AS
- 与 $geoip_country 指令生成的变量重叠
  - $geoip_country_code：两个字母的国家代码，比如 CN 或者 US
  - $geoip_country_code3：三个字母的国家代码，比如 CHN 或者 USA
  - $geoip_country_name：国家名称，例如 “China”, “United States”
- $geoip_region：洲或者省的编码，例如 02
- $geoip_region_name：洲或者省的名称，例如 Zhejiang 或者 Saint Petersburg
- $geoip_city：城市名
- $geoip_postal_code：邮编号
- $geoip_area_code：仅美国使用的邮编号，例如 408
- $geoip_dma_code：仅美国使用的 DMA 编号，例如 807

### keepalive 模块

前面说的都是 Nginx 的变量相关的内容，其实 Nginx 还有一个很具有特色的模块，那就是 keepalive 模块，由于内容不是很多，所以我就直接写到这篇文章里面了，单写一篇显得内容不够哈。

这里指的是 HTTP 的 keepalive，TCP 也有 keepalive，后面会说。

而且是对客户端的 keepalive，不是对上游服务器的。

- 功能：多个 HTTP 请求通过复用 TCP 连接，可以实现以下功能：
  - 减少握手次数
  - 通过减少并发连接数减少了服务器资源消耗
  - 降低 TCP 拥塞控制的影响，保证滑动窗口维持在一个最优的大小
- Connection 头部
  - close：表示请求处理完就关闭连接
  - keepalive：表示复用连接处理下一条请求
- Keepalive 头部：timeout=n，单位是秒，表示连接至少保持 n 秒

#### 指令

对客户端行为控制的指令：

```
Syntax: keepalive_disable none | browser ...;
Default: keepalive_disable msie6; 
Context: http, server, location

Syntax: keepalive_requests number;
Default: keepalive_requests 100; 
Context: http, server, location

Syntax: keepalive_timeout timeout [header_timeout];
Default: keepalive_timeout 75s; 
Context: http, server, location
```

- `keepalive_disable` 设置为 none 表示对所有浏览器启用 keepalive，msie6 表示在老版本 MSIE 上禁用 keepalive
- `keepalive_requests` 设置允许保持 keepalive 的请求的数量
- `keepalive_timeout` 表示超时时间

好了，关于 Nginx 的模块介绍就已经全部介绍完了，有兴趣的同学可以去翻我前面的系列文章。当然还有一部分重要的内容还没有介绍，那就是关于 Nginx 的反向代理和负载均衡部分，这块咱们单独抽出来说，别着急，马上干货就出来。
