# OpenNJet使用手册

# v2.0.1



# 1.关于本手册

 

本手册面向于OpenNJet应用引擎的非管理员用户，即属于普通角色的用户，主要包括各个功能模块的操作指南。

 

# 2.OpenNJet应用引擎介绍



## 2.1 应用引擎技术介绍

应用引擎是面向互联网和云原生应用提供的运行时组态服务程序。具备环境感知、安全控制、加速优化等能力，一般呈现为Web服务、流媒体服务、代理(Proxy)、应用中间件、API网关、消息队列等产品形态。

应用引擎在云原生架构中，除了提供南北向通信网关的功能以外，因为提供了服务网格中东西向通信、透明流量劫持、熔断、遥测与故障注入等新功能特性，其地位和作用在云原生架构中变得愈发重要。

OpenNJet最早是基于NGINX1.19基础，fork并独立演进的开源应用引擎，并随着NGINX版本迭代，吸收上游NGINX的更新，已经同步更新到NGINX1.23.1版本。OpenNJet的目标在于适应国内特定的技术规范及标准，如国密算法套件支持，并构建安全可控的云原生数据面，支撑我国云原生产业生态。作为底层引擎，OpenNJet利用动态加载机制可以实现不同的产品形态，如API网关、消息代理、出入向代理，负载均衡，WAF等等



## 2.2 **系统架构介绍**



![image-20230612134609759](https://gitee.com/gebona/picture/raw/master/image-20230612134609759.png)

相比市面其他类型的API网关，高性能是NGINX主要的优点，但动态配置能力的缺乏一直受到业界的诟病。OpenNJet在NGINX的架构上进行了扩充，对其框架进行了改写，增加了 C 及可持久化的动态存储能力，解决了指令配置变更动态生效的关键问题，扩展了OpenNJet的应用场景。此外，业界对应用引擎可观测性的需求，需要应用引擎持续不断的采集性能指标、日志数据以及注入跟踪信息，但这对应用引擎的性能造成了不可忽视的影响，OpenNJet利用Copilot framwork隔离了业务处理及配置变更和指标采集，避免了遥测对性能的影响。作为云原生的应用引擎，OpenNJet需要支持业界流行的Ingress及Sidecar的api规范，基于动态配置+ Copilot framework架构，NJet可以通过不断更新独立的相关Copilot module，实现对响应标准规范的及时支持。



## 2.3 OpenNJet 编译与安装

###         2.3.1   **编译环境：**

​                  i.     准备一台能够访问外网的

​                  ii.     CentOS Linux release 7.9.2009 (Core)

###        2.3.2  **编译环境配置**

####             2.3.2.1 配置yum源：

1. 执行以下指令：sudo yum --enablerepo=extras install -q -y epel-release centos-release-scl-rh https://repo.ius.io/ius-release-el7.rpm 
2. sudo curl -o /etc/yum.repos.d/mercurial.repo https://www.mercurial-scm.org/release/centos7/mercurial.repo 
3. 上面步骤完成后，文件系统的目录/etc/yum.repos.d 将生成对应的repo文件。 
4. [root@CDN102 home]# ls -al /etc/yum.repos.d/mercurial.repo 
5. -rw-r--r--. 1 root root 267 3月  7 11:26 /etc/yum.repos.d/mercurial.repo


#### 2.3.2.2 yum 安装软件包：

1. sudo yum install -y devtoolset-8-make devtoolset-8-toolchain ca-certificates mercurial zlib-devel         cmake3 ninja-build libunwind-devel pcre-devel openssl-devel libtool libtool-ltdl 

####           2.3.2.3 **创建符号连接：**

1. sudo ln -s /opt/rh/devtoolset-8/root/usr/bin/gcc /usr/local/bin/gcc
2. sudo ln -s /opt/rh/devtoolset-8/root/usr/bin/c++ /usr/local/bin/c++
3. sudo ln -s /opt/rh/devtoolset-8/root/usr/bin/cc /usr/local/bin/cc
4. sudo ln -s /opt/rh/devtoolset-8/root/usr/bin/make /usr/local/bin/make

####           2.3.2.4 **修改** **ld.so.conf** **配置** 

 执行以下命令：

```
 sudo bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf'

 sudo ldconfig
```

###      2.3.3 **编译代码**

 把OpenNJet 1.0.zip 包上传到 /home 目录下。 并解压 unzip njet1.0.zip，如图：![image-20230612135634822](https://gitee.com/gebona/picture/raw/master/image-20230612135634822.png)

####           2.3.3.1 **编译：**OpenNJet

#####              2.3.3.1.1 **执行：** **sh build_cc.sh conf**

![image-20230612140953550](https://gitee.com/gebona/picture/raw/master/image-20230612140953550.png)

#####            2.3.3.1.2 **执行：**make

​    如果make 后，有如下提示：则继续 执行 make 指令。

![image-20230612141031022](https://gitee.com/gebona/picture/raw/master/image-20230612141031022.png)

#####          2.3.3.1.3 **正确编译完成：如图：**

![image-20230612141122763](https://gitee.com/gebona/picture/raw/master/image-20230612141122763.png)

#####       2.3.3.1.4 **最后执行：**make install



# 3.**系统功能**

1.0目标是实现基本动态框架，为动态配置打下基础，全面改造SSL支持，从而支持国密，DTLS.初步实现sidecar的xDS接口，可作为边车应用在服务网格中。共涉及引擎内核，边车、出入口网关、web引擎、应用代理、安全、监控、可编程控制、api网关等4大类38项功能。在下述功能清单中，着重描述扩展、定制的功能，原生NGINX的能力可以参考 https://nginx.org/en/docs



## 3.1 **基本使用**

###         3.1.1  **目录结构及功能说明**

```
Bash
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
      ├── stream      tcp/  udp   四层网络代理服务器相关的代码
      ├── misc        一些辅助代码，测试  c++  头的兼容性，以及对google_perftools的支持
      └── os          主要是对各种不同体系统结构所提供的系统函数的封装，对外提供统一的系统调用接口
```

###       3.1.2 **使用命令**

####             3.1.2.1 **显示帮助信息** 

​           njet -h 

####             3.1.2.2 **启动**

​             njet -p /tmpr/njet/   -c conf/njet.conf

​             常见启动参数：

​              -p 指定prefix配置文件路径，不指定，默认/etc/njet

​              -c 指定配置文件，不指定，默认njet.conf

​              -e 指定error 日志文件 

####             3.1.2.3 **测试配置信息是否有错误**

​             njet -t

####            3.1.2.4 **显示版本** 

​             njet -v

####            3.1.2.5 **显示编译阶段的参数** 

​              njet -V

####            3.1.2.6 **快速停止**   

​              njet -s stop 或者 kill -TERM {进程id}

####            3.1.2.7 **优雅停止服务**

​                njet -s quit  或者 kill -QUIT {进程id}

####            3.1.2.8 **重新加载配置**

​                 njet -s reload 或者 kill -HUP {进程id}



## 3.2 **CoPilot**

copilot类似于NGINX中的helper，但通过配置helper指令，实现装载不同的helper实现。从而实现不同的功能。目前的OpenNJet的事件总线、控制平面集成接口都是通过其设置。

copilot框架依赖mqconf提供的helper指令:

指令说明：

<table class="md-table">
<thead>
<tr class="md-end-block">
<th><span class="td-span"><span class="md-plain"> Syntax </span></span></th>
<th><span class="td-span"><span class="md-plain"> helper tag so_file conf_file; </span></span></th>
</tr>
</thead>
<tbody>
<tr class="md-end-block">
<td><span class="td-span"><span class="md-plain"> Default </span></span></td>
<td><span class="td-span"><span class="md-plain"> --- </span></span></td>
</tr>
<tr class="md-end-block md-focus-container">
<td><span class="td-span"><span class="md-plain"> Context </span></span></td>
<td><span class="td-span md-focus"><span class="md-plain"> main </span></span></td>
</tr>
</tbody>
</table>

参数说明：

<table>
<tbody>
<tr>
<td width="100">
<p>参数名称</p>
</td>
<td width="100">
<p>是否必须</p>
</td>
<td width="247">
<p>参数说明</p>
</td>
</tr>
<tr>
<td width="100">
<p>tag</p>
</td>
<td width="100">
<p>是</p>
</td>
<td width="247">
<p>指定一个名字标签</p>
</td>
</tr>
<tr>
<td width="100">
<p>so_file</p>
</td>
<td width="100">
<p>是</p>
</td>
<td width="247">
<p>so文件</p>
</td>
</tr>
<tr>
<td width="100">
<p>conf_file</p>
</td>
<td width="100">
<p>是</p>
</td>
<td width="247">
<p>配置文件</p>
</td>
</tr>
</tbody>
</table>
**Copilot so** **规范**

**so实现以下接口：**

1）unsigned int njt_helper_check_version(void)
    当前版本是1
    \#define NJT_HELPER_VER     1


 2）void njt_helper_run(helper_param param)

 \#define NJT_HELPER_CMD_NO    0
 \#define NJT_HELPER_CMD_STOP   1
 \#define NJT_HELPER_CMD_RESTART 2

 typedef unsigned int (*helper_check_cmd_fp)(void* ctx);
 typedef struct {
   size_t   conf_fn_len;
   u_char   *conf_fn_data;
   helper_check_cmd_fp check_cmd_fp;
   void*    ctx;
 } helper_param;

 

3）unsigned int njt_helper_ignore_reload(void)

返回1，表示该so的copilot进程，不会在reload的时候重启。

返回0，表示该so的copilot进程，会在reload的时候重启。

注1：so可以不实现该接口。若不实现，则等同于返回0。

注2：如果so实现该接口并且返回1，那么在reload的时候该so的copilot进程不会重启，但是有一点需要注意：reload的时候配置文件中需保留原helper指令，这是配置上的强制要求，不满足此要求会导致reload失败。

**so 需要实现以下操作：**

1）在njt_helper_run的事件循环中，调用param.check_cmd_fp()
    例如：
 unsigned int cmd;
 cmd = param.check_cmd_fp(param.ctx);

 2）根据接收到的命令进行处理，
    接收到NJT_HELPER_CMD_STOP命令，要进行停止操作；
    接收到NJT_HELPER_CMD_RESTART命令，要进行停止操作。

注意：NJT_HELPER_CMD_RESTART 为预留命令

###     3.2.1 **copilot:ctrl**

该模块是一个基础模块，给 http-sendmsg 和 http_health_check 等模块提供运行环境。

为了运行该模块，需要在主配置文件中进行配置：

下面为示例配置：

helper ctrl objs/njt_helper_ctrl_module.so njet_ctrl.conf;

该模块自身的配置，即上面主配置文件中helper指令中指定的配置文件的内容。



配置的指令分为两类：

1）OpenNJet中标准的指令；

**需要注意**：用 error_log 和 access_log 指令指定的log文件要不同于主配置文件中的log文件。

2）通过 load_module 指令加载的动态模块中扩展出来的指令。



配置示例如下：

```
load_module /home/njet/modules/njt_http_sendmsg_module.so;
load_module /home/njet/modules/njt_http_health_check_helper.so;
user njet njet;

error_log logs/error_ctrl.log info;

events {
    worker_connections  1024;
}

http {
    dyn_sendmsg_conf conf/iot_msg.conf;
    access_log logs/access_ctrl.log combined;

    server {
        listen       8081;
        
       

        location / {
            return 200 "NJet control_plane stub\n";
        }
        location /hc {
                health_check_api;
        }
    }
}
```

###      3.2.2 **copilot:broker**

该模块提供消息服务端功能，要开启该功能在njet.conf 的 main block 中添加如下指令：

*helper broker /home/njet/modules/njt_helper_broker_module.so conf/mqtt.conf;* 

**mqtt.conf**    **Message Broker** **配置文件**

<table style="width: 664px;">
<tbody>
<tr>
<td style="width: 210.438px;">
<p>配置项</p>
<p>&nbsp;</p>
</td>
<td style="width: 118.234px;">
<p>必须修改</p>
</td>
<td style="width: 313.328px;">
<p>配置说明</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>log_dest</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
</td>
<td style="width: 313.328px;">
<p>日志输出方法，一般配置为文件输出，配置值为file 文件名（绝对路径）</p>
<p>默认： $PREFIX/logs/mosquitto.log</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>log_type</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
<p>&nbsp;</p>
</td>
<td style="width: 313.328px;">
<p>日志级别： debug, error, warning, notice, information，mqtt库中的日志输出方式和java 中的不一样，配置了某个日志级别，只有该日志级别的消息才会输出到日志中， 因此需要为需要的日志级别单独配置一行。默认： error</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>listener</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
</td>
<td style="width: 313.328px;">
<p>监听端口及地址 , 默认： 0 $PREFIX/data/mosquitto.sock</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>allow_anonymous</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
</td>
<td style="width: 313.328px;">
<p>是否允许匿名连接，目前配置为true</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>persistence</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
</td>
<td style="width: 313.328px;">
<p>是否开启消息持久化 （true, false),&nbsp; 默认： true</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>autosave_on_changes</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
</td>
<td style="width: 313.328px;">
<p>是否有消息时进行自动保存&nbsp;&nbsp; 默认： true</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>autosave_interval</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
</td>
<td style="width: 313.328px;">
<p>当自动保存开启时，当新消息个数大于这个配置值时，将触发保存操作 默认： 1</p>
</td>
</tr>
<tr>
<td style="width: 210.438px;">
<p>persistence_location</p>
</td>
<td style="width: 118.234px;">
<p>否</p>
</td>
<td style="width: 313.328px;">
<p>持久化文件保存路径，将在该路径下创建 mosquitto.db , 默认： $PREFIX/data</p>
</td>
</tr>
</tbody>
</table>


配置都会有默认值，建议只配置日志级别。

```
C++
  log_type error
```

### 3.2.3 **copilot:ha**

 该模块通过vrrp协议提供多个njet实例的高可用功能。

 VRRP 的相关介绍可以参见网页：

https://baike.baidu.com/item/%E8%99%9A%E6%8B%9F%E8%B7%AF%E7%94%B1%E5%99%A8%E5%86%97%E4%BD%99%E5%8D%8F%E8%AE%AE/2991482

**配置说明：**

▪   要开启该功能在njet.conf 的 main block 中添加如下指令：

```
Bash
njet配置文件增加：
helper ha /usr/lib/njet/modules/njt_helper_ha_module.so vrrp.conf;
```

▪   该模块需要依赖外部的so动态库，将 libha_emb.* 拷贝到 /usr/local/lib 目录下，git 的流水线自动编译后下载的zip 包中包含需要的so库，确认 /usr/loca/lib 目录已经配置到 /etc/ld.so.conf 中

```
Bash
cp -a work/usr/local/lib/libha_emb.* /usr/local/lib
```

▪   由于设置网卡vip , 需要一定的权限，opennjet 在非root 用户启动情况下，需要预先设置对应的权限

```
Bash
 sudo setcap cap_net_bind_service,cap_net_admin,cap_net_raw+eip njet 
```

▪   生效配置

```
Bash
sudo ldconfig
```

 **增加vrrp.conf 配置文件说明：**

<table>
<tbody>
<tr>
<td style="width: 166px;">
<p>配置项</p>
<p>&nbsp;</p>
</td>
<td style="width: 94px;">
<p>必须修改</p>
</td>
<td style="width: 305px;">
<p>配置说明</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 166px;">
<p>vrrp_instance</p>
</td>
<td style="width: 94px;">
<p>是</p>
</td>
<td style="width: 305px;">
<p>设置的VRRP的实例名，string类型</p>
</td>
</tr>
<tr>
<td style="width: 166px;">
<p>state</p>
</td>
<td style="width: 94px;">
<p>是</p>
</td>
<td style="width: 305px;">
<p>设置当前节点的初始化状态，状态为MASTER或者BACKUP。</p>
</td>
</tr>
<tr>
<td style="width: 166px;">
<p>interface</p>
</td>
<td style="width: 94px;">
<p>是</p>
</td>
<td style="width: 305px;">
<p>可以绑定vip的接口名称，比如eth0,bond0,br0。</p>
</td>
</tr>
<tr>
<td style="width: 166px;">
<p>virtual_router_id</p>
<p>&nbsp;</p>
</td>
<td style="width: 94px;">
<p>是</p>
</td>
<td style="width: 305px;">
<p>设置虚拟路由器惟一标识，范围：0-255，同属一个集群的多个njet节点该id相同, 不同的njet集群该值必须不同，务必要确认在同一网络中此值必须唯一。</p>
</td>
</tr>
<tr>
<td style="width: 166px;">
<p>priority</p>
</td>
<td style="width: 94px;">
<p>是</p>
</td>
<td style="width: 305px;">
<p>设置当前njet节点的优先级，范围：1-254，值越大优先级越高</p>
</td>
</tr>
<tr>
<td style="width: 166px;">
<p>virtual_ipaddress</p>
</td>
<td style="width: 94px;">
<p>是</p>
</td>
<td style="width: 305px;">
<p>设置虚拟IP对应的子网掩码。</p>
</td>
</tr>
</tbody>
</table>

配置样例：

```
Bash
vrrp_instance m{
        state MASTER
        interface eth0 
        virtual_router_id 32 
        priority 100 
        virtual_ipaddress {
             10.1.0.22/24 
        }
}
```

HA copilot 模块的对应日志是 $PREFIX/logs/njet_vrrp.log, 日志中将显示 ha 模块设置VIP时的一些相关信息, 配置文件中有错误的配置项，日志中将显示对应的错误位置及加载后的默认值。 

```
Bash
Thu Jun 01 17:18:11.192916235 2023: (/usr/local/njet/conf/vrrp.conf: Line 2) (m) unknown state 'Back', defaulting to BACKUP
Thu Jun 01 17:18:11.193003054 2023: (/usr/local/njet/conf/vrrp.conf: Line 5) number '1000' outside range [1, 255]
Thu Jun 01 17:18:11.193027778 2023: (/usr/local/njet/conf/vrrp.conf: Line 5) (m) Priority not valid! must be between 1 & 255. Using default 100
Thu Jun 01 17:18:11.193230339 2023: gratuitous_arp_init end
Thu Jun 01 17:18:11.193271876 2023: start up end
Thu Jun 01 17:18:11.193536733 2023: (m) Entering BACKUP STATE (init)
Thu Jun 01 17:18:14.803010859 2023: (m) Entering MASTER STATE
```

**配置的动态加载**

Copilot 配置的动态设置在后续的版本中，将进行统一的考虑及设计。

目前 HA Copilot 已经屏蔽NJet 的 Reload 信号， vrrp 相关的配置，如果进行了手工编辑， 需要给 HA Copilot 进程发送 HUP 信号。 (kill -HUP $PID, 其中 $PID 为 HA Copilot 的进程ID )



## 3.3 **动态配置框架**

---

OpenNJet 提供基于eventbus构建的消息总线，其包括copilot：broker，动态配置 stub 模块 njet_http_kv,sendmsg 模块 njet_http_sendmsg 模块。

![image-20230612152526773](https://gitee.com/gebona/picture/raw/master/image-20230612152526773.png)

如上图所示意， OpenNJet的动态配置一般由控制面发送配置消息，不同的 worker 通过加载动态配置 stub，监听不同的消息，并回调不同的处理函数。

![image-20230612152611529](https://gitee.com/gebona/picture/raw/master/image-20230612152611529.png)

###     3.3.1 kv模块配置

在 http block 下，指定该模块的配置文件：

```
C++
  http {
   dyn_kv_conf conf/iot.conf;
  }
```

**iot.conf  MQTT** **客户端配置**

<table style="width: 660px;">
<tbody>
<tr>
<td style="width: 203.312px;">
<p>配置项</p>
</td>
<td style="width: 104.156px;">
<p>必须修改</p>
</td>
<td style="width: 330.531px;">
<p>配置说明</p>
</td>
</tr>
<tr>
<td style="width: 203.312px;">
<p>broker_addr</p>
</td>
<td style="width: 104.156px;">
<p>否</p>
</td>
<td style="width: 330.531px;">
<p>MQ broker 地址。默认： unix:$PREFIX/data/mosquitto.sock</p>
</td>
</tr>
<tr>
<td style="width: 203.312px;">
<p>log_type</p>
<p>&nbsp;</p>
</td>
<td style="width: 104.156px;">
<p>否</p>
</td>
<td style="width: 330.531px;">
<p>日志级别： debug, error, warning, notice, information，不同的日志级别需要单独配置一行。默认： error</p>
</td>
</tr>
<tr>
<td style="width: 203.312px;">
<p>topic</p>
</td>
<td style="width: 104.156px;">
<p>否</p>
</td>
<td style="width: 330.531px;">
<p>默认订阅的主题</p>
<p>&nbsp;/cluster/+/kv_set/#&nbsp;</p>
<p>&nbsp;用于kvstore 值的设置</p>
<p>/dyn/#&nbsp;&nbsp;&nbsp;</p>
<p>用于动态 location 更新</p>
<p>/rpc/#&nbsp;&nbsp;&nbsp;</p>
<p>用于发送rpc消息</p>
</td>
</tr>
<tr>
<td style="width: 203.312px;">
<p>keepalive</p>
</td>
<td style="width: 104.156px;">
<p>否</p>
</td>
<td style="width: 330.531px;">
<p>给服务端发送PING命令的间隔时间。 默认：30</p>
</td>
</tr>
<tr>
<td style="width: 203.312px;">
<p>kv_store_dir</p>
</td>
<td style="width: 104.156px;">
<p>否</p>
</td>
<td style="width: 330.531px;">
<p>Key value store 持久化文件路径。 默认： $PREFIX/data</p>
</td>
</tr>
<tr>
<td style="width: 203.312px;">
<p>protocol_version</p>
<p>&nbsp;</p>
</td>
<td style="width: 104.156px;">
<p>否</p>
</td>
<td style="width: 330.531px;">
<p>使用的协议版本，要支持 request response消息，</p>
<p>必须填 5 。 默认：5</p>
</td>
</tr>
</tbody>
</table>


配置都会有默认值，建议只配置日志级别

```
C++
 log_type error
```

###       3.3.2 **Sendmsg**

要开启该功能, 需在njet_ctrl.conf 的 main block 中加载该模块：

*load_module /home/njet/modules/njt_http_sendmsg_module.so;* 

 

并在 http block 下，指定该模块的配置文件：

```
  http {
      dyn_sendmsg_conf conf/iot_ctrl.conf;
      
     
      
       server {
         ...
         location /kv {
             dyn_sendmsg_kv;
         }
         ...
     }
  }
```

dyn_sendmsg_kv 配置不是必须的， 这个是对外提供了 kv 值设置及查询的 http API 接口。

**iot_ctrl.conf  MQTT** **客户端配置**

<table style="height: 380px; width: 674px;">
<tbody>
<tr style="height: 46px;">
<td style="height: 46px; width: 178.062px;">
<p>配置项</p>
</td>
<td style="height: 46px; width: 147.312px;">
<p>必须修改</p>
</td>
<td style="height: 46px; width: 326.625px;">
<p>配置说明</p>
</td>
</tr>
<tr style="height: 64px;">
<td style="height: 64px; width: 178.062px;">
<p>broker_addr</p>
</td>
<td style="height: 64px; width: 147.312px;">
<p>否</p>
</td>
<td style="height: 64px; width: 326.625px;">
<p>MQ broker 地址. 默认：unix:$PREFIX/data/mosquitto.sock</p>
</td>
</tr>
<tr style="height: 82px;">
<td style="height: 82px; width: 178.062px;">
<p>log_type</p>
<p>&nbsp;</p>
</td>
<td style="height: 82px; width: 147.312px;">
<p>否</p>
<p>&nbsp;</p>
</td>
<td style="height: 82px; width: 326.625px;">
<p>日志级别： debug, error, warning, notice, information，不同的日志级别需要单独配置一行。 默认：error</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 178.062px;">
<p>keepalive</p>
</td>
<td style="height: 46px; width: 147.312px;">
<p>否</p>
</td>
<td style="height: 46px; width: 326.625px;">
<p>给服务端发送PING命令的间隔时间. 默认：30</p>
</td>
</tr>
<tr style="height: 64px;">
<td style="height: 64px; width: 178.062px;">
<p>kv_store_dir</p>
</td>
<td style="height: 64px; width: 147.312px;">
<p>否</p>
</td>
<td style="height: 64px; width: 326.625px;">
<p>Key value store 持久化文件路径. 默认：$PREFIX/data</p>
</td>
</tr>
<tr style="height: 78px;">
<td style="height: 78px; width: 178.062px;">
<p>protocol_version</p>
<p>&nbsp;</p>
</td>
<td style="height: 78px; width: 147.312px;">
<p>否</p>
</td>
<td style="height: 78px; width: 326.625px;">
<p>使用的协议版本，要支持 request response消息，</p>
<p>必须填 5 . 默认：5</p>
</td>
</tr>
</tbody>
</table>


配置都会有默认值，建议只配置日志级别

```
log_type error
```



## 3.4 **国密支持**

###        3.4.1 **功能说明**

使用国密版 OpenSSL 库来使用 SM2、SM3、SM4 算法（以下简称为SM234算法）。

SM234 算法使用双证书，包括签名证书和加密证书，这是国密支持在证书上的一个特点。

同时，还要求使用的国密版 OpenSSL 兼容标准 OpenSSL。

根据 OpenNJet 的使用场景，国密支持的场景也细分为 Server 和反向代理两种场景。

###        3.4.2 **配置说明**

OpenNJet 作为 Server 时国密支持需要增加双证书的配置：

```
Plain Text
      server {
        listen       443 ssl;
        server_name  localhost;

        # 原RSA证书
        ssl_certificate     /home/certs/cert/sslserver.crt;
        ssl_certificate_key /home/certs/cert/sslserver.key;
        ssl_ntls  on;
        # 国密支持添加的证书
        ssl_certificate      certs/SS.crt     certs/SE.crt;
        ssl_certificate_key  certs/SS.key     certs/SE.key;
        
        location / {
            root   html;
            index  index.html index.htm;
        }
      }
```

OpenNJet 作为反向代理时，增加以下三个指令：

proxy_ssl_gm指令，用于OpenNJet连接后端服务器时，指定是否使用国密。

```
Syntax:        proxy_ssl_gm on | off;
Default:        proxy_ssl_gm off
Context:        http, server, location
```

proxy_ssl_enc_certificate指令，用于OpenNJet连接后端服务器时，指定加密证书文件。

```
Syntax:        proxy_ssl_enc_certificate file;
Default:        —
Context:        http, server, location
```

proxy_ssl_enc_certificate_key指令，用于 OpenNJet 连接后端服务器时，指定加密密钥文件。

```
Syntax:        proxy_ssl_enc_certificate_key file;
Default:        —
Context:        http, server, location
```

配置示例：

```
      server {
        listen       8000;
        server_name  localhost;

        proxy_ssl on;
        proxy_ssl_gm on;
        proxy_ssl_ciphers ECDHE-SM4-SM3; #也可以指定ECC-SM4-SM3国密套件
        
        ssl_certificate      certs/SS.crt     certs/SE.crt;
        ssl_certificate_key  certs/SS.key     certs/SE.key;

        # 其他配置，例如：
        location / {
           proxy_pass https://192.168.183.111:443;
        }        
      }
```

### 3.4.3 **功能验证**

####     3.4.3.1 **验证服务器对国密的支持**

验证工具为分别为密信浏览器和gmcurl。

密信浏览器：

![image-20230612160751057](https://gitee.com/gebona/picture/raw/master/image-20230612160751057.png)

使用密信浏览器访问server：

![image-20230612160843325](https://gitee.com/gebona/picture/raw/master/image-20230612160843325.png)

使用gmcurl访问server：

![image-20230612160908138](https://gitee.com/gebona/picture/raw/master/image-20230612160908138.png)

####       3.4.3.2 验证服务器对RSA证书的支持

上节的server的配置中分别指定了RSA证书和国密证书，再验证一下服务器对RSA证书的支持。验证工具为分别为Chrome浏览器和curl。

使用Chrome浏览器访问server：

![image-20230612160956474](https://gitee.com/gebona/picture/raw/master/image-20230612160956474.png)

使用curl访问server:

![image-20230612164313289](https://gitee.com/gebona/picture/raw/master/image-20230612164313289.png)

####     3.4.3.3 后端upstream使用国密验证

OpenNJet 作反向代理访问后端国密 upstream 服务器。验证工具为分别为 Chrome 浏览器和 curl 。

![image-20230612164434481](https://gitee.com/gebona/picture/raw/master/image-20230612164434481.png)

![image-20230612164451379](https://gitee.com/gebona/picture/raw/master/image-20230612164451379.png)



## 3.5 **动态Location**

###     3.5.1 **功能说明**

本系统支持对location模块的动态添加、删除，可以对locaiton进行便捷的配置和所需指令功能的添加使用。

 该模块支持ACL控制，详见章节3.26。

###     3.5.2 **配置说明**

请参考标准配置文件章节配置，全量配置即可配置上此功能，但请注意一定要包含以下指令和块：

njet.conf 配置文件中务必有此模块

```
load_module modules/njt_http_location_module.so;
```

ctrl.conf配置文件中务必包含这些模块

```
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_http_location_api_module.so;

http {
     server {
        listen       8081;
        location /dyn_location {
        dyn_location_api;
        }
     }
}
```

 动态location，配置ACL控制

```Bash
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_http_location_api_module.so;

 server {
        listen       8081;
       
        location /dyn_location {
            dyn_location_api;
            limit_except GET {
               auth_basic "NGINX plus API";
               auth_basic_user_file /etc/njet/htpasswd;
            }
        }
  }

```

###     3.5.3 **API** **说明**

动态location增加说明：

<table style="width: 687px;">
<tbody>
<tr>
<td style="width: 153.5px;">
<p>&nbsp;</p>
<p>配置项</p>
<p>&nbsp;</p>
</td>
<td style="width: 102.922px;">
<p>必填</p>
</td>
<td style="width: 408.578px;">
<p>&nbsp;</p>
<p>配置说明</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 153.5px;">
<p>type</p>
</td>
<td style="width: 102.922px;">
<p>是</p>
</td>
<td style="width: 408.578px;">
<p>&nbsp;&ldquo;add&rdquo; &nbsp;添加location</p>
</td>
</tr>
<tr>
<td style="width: 153.5px;">
<p>addr_port</p>
</td>
<td style="width: 102.922px;">
<p>是</p>
</td>
<td style="width: 408.578px;">
<p>添加的主机的，port 端口。&nbsp; 例如："192.168.40.203：8000"， 或 &ldquo;0.0.0.0:8000&rdquo;</p>
</td>
</tr>
<tr>
<td style="width: 153.5px;">
<p>server_name</p>
</td>
<td style="width: 102.922px;">
<p>是</p>
</td>
<td style="width: 408.578px;">
<p>主机的server_name, 例如："cluster.tmlake.com"</p>
</td>
</tr>
<tr>
<td style="width: 153.5px;">
<p>locations</p>
</td>
<td style="width: 102.922px;">
<p>是</p>
</td>
<td style="width: 408.578px;">
<p>List 对象列表。</p>
<p>对象字段：</p>
<p>location_rule，//可以为空。</p>
<p>location_name，&nbsp; //不能为空。</p>
<p>location_body， //location_body 或 proxy_pass 必须有一个不为空。结尾不要带;</p>
<p>proxy_pass&nbsp;&nbsp;&nbsp; //location_body 或 proxy_pass 必须有一个不为空。结尾不要带;</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //proxy_pass 后面只能跟配置文件预定义的upstream 名字。 其他的不支持</p>
<p>&nbsp;</p>
</td>
</tr>
</tbody>
</table>


说明：

 location_body 或 proxy_pass 字段均可以为空，结尾不要带;

 proxy_pass 现在支持http，https，ip地址，unix socket，域名，变量等。（例如：http://backend1、https://backend1、http://127.0.0.1:443、http://$upstream_name、http://unix:/var/lib/njet/njet-502-server.sock等）

 

动态location删除说明：

<table style="width: 683px; height: 376px;">
<tbody>
<tr style="height: 110px;">
<td style="width: 155.469px; height: 110px;">
<p>&nbsp;</p>
<p>配置项</p>
<p>&nbsp;</p>
</td>
<td style="width: 128.391px; height: 110px;">
<p>必填</p>
</td>
<td style="width: 377.141px; height: 110px;">
<p>&nbsp;</p>
<p>配置说明</p>
<p>&nbsp;</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="width: 155.469px; height: 46px;">
<p>type</p>
</td>
<td style="width: 128.391px; height: 46px;">
<p>是</p>
</td>
<td style="width: 377.141px; height: 46px;">
<p>&nbsp;&ldquo;del&rdquo;&nbsp; 删除location</p>
</td>
</tr>
<tr style="height: 64px;">
<td style="width: 155.469px; height: 64px;">
<p>addr_port</p>
</td>
<td style="width: 128.391px; height: 64px;">
<p>是</p>
</td>
<td style="width: 377.141px; height: 64px;">
<p>添加的主机的，port 端口。&nbsp; 例如："192.168.40.203：8000"， 或 &ldquo;0.0.0.0:8000&rdquo;</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="width: 155.469px; height: 46px;">
<p>server_name</p>
</td>
<td style="width: 128.391px; height: 46px;">
<p>是</p>
</td>
<td style="width: 377.141px; height: 46px;">
<p>主机的server_name, 例如："cluster.tmlake.com"</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="width: 155.469px; height: 46px;">
<p>location_rule</p>
</td>
<td style="width: 128.391px; height: 46px;">
<p>否</p>
</td>
<td style="width: 377.141px; height: 46px;">
<p>Location 的表达式， 例如：&ldquo;=&rdquo;&nbsp; 或 其他的正则式</p>
</td>
</tr>
<tr style="height: 64px;">
<td style="width: 155.469px; height: 64px;">
<p>location_name</p>
</td>
<td style="width: 128.391px; height: 64px;">
<p>是</p>
</td>
<td style="width: 377.141px; height: 64px;">
<p>location的名字，也就是表达式后面的。 例如： &ldquo;/&rdquo;&nbsp; 或 &ldquo;/test&rdquo;</p>
</td>
</tr>
</tbody>
</table>


说明：

添加的嵌套location， 删除时只能通过根location一并删除，不支持直接删除子location。

### 3.5.4 **调用样例**

####    3.5.4.1 **post** **添加**

回显{"code":0,"msg":"success."}即为添加成功。

```
Bash
curl -v -s -X POST http://127.0.0.1:8081/dyn_location -d '{"type":"add","addr_port":"0.0.0.0:8899","server_name":"cluster1","locations":[{"location_rule":"","location_name":"/","location_body":"","proxy_pass":"http://http1"}]}'
> POST /dyn_location HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 127.0.0.1:8081
> Accept: */*
> Content-Length: 149
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 149 out of 149 bytes
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Mon, 22 May 2023 07:22:30 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
<
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

####   3.5.4.2 **查询**

添加成功后，可以对所添加的 location 对应的上游资源进行请求，已验证是否可以正常对资源进行请求查询。

```
Bash
curl -v -s http://127.0.0.1:8899/1.html
> GET /1.html HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 127.0.0.1:8899
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Fri, 31 Mar 2023 03:34:00 GMT
< Content-Type: text/html
< Content-Length: 49
< Connection: keep-alive
< ETag: W/"49-1641534167000"
< Last-Modified: Fri, 07 Jan 2022 05:42:47 GMT
< Vary: Accept-Encoding
< X-Cache-Status: STALE
< Accept-Ranges: bytes
<
<html>
<head></head>
* Connection #0 to host 127.0.0.1 left intact
<body>111111</body></html>
```

#### 3.5.4.3 **put** **禁用删除**

动态location功能也支持动态进行删除操作，以下将展示如何进行验证：

进行动态删除location操作：

```
Bash
curl -v -s -X PUT  http://127.0.0.1:8081/dyn_location -d '{"type":"del","addr_port":"0.0.0.0:8899","server_name":"cluster1","location_rule":"","location_name":"/"}'
> PUT /dyn_location HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 127.0.0.1:8081
> Accept: */*
> Content-Length: 105
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 105 out of 105 bytes
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Mon, 22 May 2023 07:38:36 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
<
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

再次请求之前的上游资源，可以发现已经请求失败，删除成功：

```
Bash
curl -v -s http://127.0.0.1:8899/1.html
> GET /1.html HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 127.0.0.1:8899
> Accept: */*
>
< HTTP/1.1 404 Not Found
< Server: njet/1.23.1
< Date: Mon, 22 May 2023 07:39:55 GMT
< Content-Type: text/html
< Content-Length: 152
< Connection: keep-alive
<
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>njet/1.23.1</center>
</body>
</html>
* Connection #0 to host 127.0.0.1 left intact
```

### 3.5.5**location功能增强**

#### 3.5.5.1**功能说明**

1）支持所有变量匹配

2）支持逻辑与和逻辑或运算

3）顺序匹配多添加location，都不满足走正常location 路径匹配。

4）操作符：=，!= ,正则。

#### 3.5.5.2 **静态配置文件示例**

```
JSON
server {
   listen   80;
   location ( $http_host = www.test.com && $http_user_agent ~= windows) {
        return 200 " www.test.com && http_user_agent ~= window ";
   }

   location (  $uri = "/delete" || $request_method = DELETE) {
        return 200 $remote_addr":delete";
   }
   location ( $remote_addr = "192.168.40.203" && $request_method = PUT && $uri  ~* .png$) {
        proxy_pass 

  }
  location ( $remote_addr = "192.168.40.203" && $request_method = PUT && $uri  ~* .png$) {
        proxy_pass http://backend1;
 }

}
```

#### 3.5.5.3 **动态新增方法示例**

1）多个与（&&）变量locaiton添加

```
JSON
   {
        "type": "add",
        "addr_port": "0.0.0.0:8888",
        "server_name": "cluster1",
        "locations": [{
                "location_rule": "",
                "location_name": "( $remote_addr = \"192.168.40.203\" && $request_method = PUT && $uri  ~* .png$)",
                "location_body": "return 200 $remote_addr\":PUT*.png\""
        }]
}
```

2）多个与（&&）否（!=）变量添加

```
JSON
{
        "type": "add",
        "addr_port": "0.0.0.0:8888",
        "server_name": "cluster1",
        "locations": [{
                "location_rule": "",
                "location_name": "( $remote_addr = \"192.168.40.203\" && $request_method = PUT && $uri != \"/1.png\")",
                "location_body": "return 200 $remote_addr\":PUT*.png\""
        }]
}
```

3）多个或（||）变量添加

```
JSON
{
        "type": "add",
        "addr_port": "0.0.0.0:8888",
        "server_name": "cluster1",
        "locations": [{
                "location_rule": "",
                "location_name": "(  $uri = \"/delete\" || $request_method = DELETE || $remote_addr = \"192.168.40.203\")",
                "location_body": "return 200 $remote_addr\":delete\""
        }]
}
```

4）正则方式添加

```
JSON
{
        "type": "add",
        "addr_port": "0.0.0.0:8888",
        "server_name": "cluster1",
        "locations": [{
                "location_rule": "",
                "location_name": "($uri ~* .(mp4|jpg|gif)$ && $server_port ~ \\d)",
                "location_body": "",
                "proxy_pass":"http://http1"
        }]
}
```

5)与（||）或（&&）多变量添加

```
JSON
{
        "type": "add",
        "addr_port": "0.0.0.0:8888",
        "server_name": "cluster1",
        "locations": [{
                "location_rule": "",
                "location_name": "( $remote_addr = \"192.168.40.203\" || $uri ~* .(mp4|jpg|gif)$ && $request_method = PUT)",
                "location_body": "",
                "proxy_pass":"http://http1"
        }]
}
```

#### 3.5.5.4注意事项

1）mirror 指令：

 在使用mirror指令时location 条件一定要包含uri 的条件，避免重新路由时，条件总是满足类似（$remote_addr = "192.168.40.104" && $server_port = "6969"）这种不包含uri路径的条件，造成mirror 失败。

```
JSON
   location （$uri = "/" && $remote_addr = "192.168.40.104" && $server_port = "6969"）{
             mirror  /mirror1;
             mirror  /mirror2;
            proxy_pass  http://backend;
   }
```

2）表达式location的执行顺序为由上到下、由先到后依次执行，在添加表达式location时需要注意先后顺序。

3）location 名称：因在增强功能中需要对表达式进行处理，所以NJet会将"("作为关键词进行处理，在locaiton使用中，无法单独将location_name设置为"("起始进行使用，如若检测到location_name是以"("起始，而没有")"时，njet即会出现报错，不予处理。

 例如：

```
location ( {

  return 200 ok;

}
```

  在nginx 下能，正常使用。 而在NJet 里 会被识别不完整的表达式，并提示错误。



## 3.6 **主动健康检查**

###     3.6.1 **功能说明**

 主动健康检查与被动健康检查：主动健康检查由Njet主动发起请求测试上游服务是否正常，如不正常则及时剔除对应服务；被动健康检查由Njet检测上游服务的异常返回发现服务异常，不能主动感知上游服务状态。

 helper进程主动健康检查：可以实现健康检查由独立进程完成，对work进程无影响，完成后修改共享内存完成upstream状态回写。

 该模块支持ACL控制，详见章节3.26。

###     3.6.2 **控制面指令说明**

####        3.6.2.1 **语法说明**

health_check_api

```
语法:        health_check_api ;
默认值:        —
允许配置位置:         location
```

在当前location开启健康检查API。

####       3.6.2.2 **配置示例**

```
load_module modules/njt_helper_health_check_module.so;
load_module modules/njt_http_sendmsg_module.so;

http{

    dyn_sendmsg_conf  conf/iot-ctrl.conf;
    include           mime.types;
    
    server {
        listen       8081;
        
        location /hc {
           health_check_api;
        }
     }
}
```

 主动健康检查，配置ACL控制

```Bash
load_module modules/njt_helper_health_check_module.so;
load_module modules/njt_http_sendmsg_module.so;

 server {
        listen       8081;
        
        
        location /hc {
            health_check_api;
            limit_except GET {
               auth_basic "NGINX plus API";
               auth_basic_user_file /etc/njet/htpasswd;
            }
        }
  }
```

### 3.6.3 **数据面指令说明**

####     3.6.3.1 **语法说明**

health_check

```
语法:        health_check mandatory | persistent;
默认值:        —
允许配置位置:         location
```

<table>
<tbody>
<tr>
<td style="width: 145px;">
<p>&nbsp;</p>
<p>配置项</p>
<p>&nbsp;</p>
</td>
<td style="width: 142px;">
<p>必填</p>
</td>
<td style="width: 277px;">
<p>&nbsp;</p>
<p>配置说明</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 145px;">
<p>health_check</p>
</td>
<td style="width: 142px;">
<p>是</p>
</td>
<td style="width: 277px;">
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 145px;">
<p>mandatory</p>
</td>
<td style="width: 142px;">
<p>是</p>
</td>
<td style="width: 277px;">
<p>配置后，静态文件server，api，域名解析出的server 初始状态都是checking 状态（不做业务，等待健康检查），会立即进行健康检查，然后更新结果为 health&nbsp; 或 unhealth</p>
</td>
</tr>
<tr>
<td style="width: 145px;">
<p>persistent</p>
</td>
<td style="width: 142px;">
<p>否</p>
</td>
<td style="width: 277px;">
<p>配置之后reload 后，会保留上次的健康状态。 直到下次健康检查。</p>
</td>
</tr>
</tbody>
</table>


####    3.6.3.2 **配置示例**

```
Nginx
    upstream backend1 {
    
         zone backend1 128k;
         
         server 192.168.40.144:5555;

         health_check mandatory persistent;

    }
```



### 3.6.4 API说明

格式说明健康检查配置项参数说明

```
JSON
{
    "interval": "3s",
    "jitter": "1s",
    "timeout": "10s",
    "passes": 2,
    "fails": 1,
    "port": 13470
}

# interval 主动健康检查频率 
# timeout 超时时间
# jitter 设置健康检查项定时器最大偏差。防止所有检查项同时触发。默认1秒。
# passes 连续通过passes次检测，更新peer为up状态
# fails 连续不通过fails次检测，更新peer为unhealthy状态
# port 指定peer端口，如果不指定，使用upstream中peer设置的端口
```

#### 3.6.4.1 **HTTP**健康检查

```
JSON
{
    "interval": "3s",
    "jitter": "1s",
    "timeout": "10s",
    "passes": 2,
    "fails": 1,
    "port": 13470
    "http": {
        "header":["host"],
        "uri": "/body",
        "body": "~ ok",
        "status": "200 204"
  }
}
```

部分API字段补充

健康检查 HTTP > status 配置方式

![img](https://gitee.com/gebona/picture/raw/master/202307101038671.jpg)

​                                                                                                      **点击图片可查看完整电子表格**

```
Plain Text
配置内容判定为健康的描述
200http返回状态码为200
! 500http返回状态码不是500
200 204http返回状态码是200 或者 204
! 301 302http返回状态码不是301且不是302
 200-399http返回状态码是200到399的，包含200和399
 ！400-599http返回状态码不是400到599的，包含400和599
301-303 307http返回状态码是301到303的，包含301和303，或者307
```

健康检查 http > header 配置方式

![img](https://gitee.com/gebona/picture/raw/master/202307101039294.jpg)

​                                                                                                      **点击图片可查看完整电子表格**

健康检查 http > body配置方式

![img](https://gitee.com/gebona/picture/raw/master/202307101041107.jpg)

​                                                                                                     **点击图片可查看完整电子表格**

#### 3.6.4.2 **Stream健康检查**

##### 3.6.4.2.1 **健康检查**TCP配置方式

```
JSON
请求BODY
{
    "interval": "3s",
    "jitter": "1s",
    "timeout": "10s",
    "passes": 2,
    "fails": 1,
    "stream": {                    /* 开启四层健康检查 */
        "send": "zhao\\x6B\\x61\\x6E\\x67",        /* 期望发送的文本 */
        "expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"      /* 期望收到的文本 */
    }
}

请求命令
curl -s http://127.0.0.1:7071/hc/1/hc/stcp/demo -XPOST -d '{"interval": "3s","jitter": "1s","timeout": "10s","passes": 2, "fails": 1,"stream": {"send": "zhao\\x6B\\x61\\x6E\\x67","expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"}}'

返回
{
    "code": 0,
    "msg": "success"
}
参数说明：
stcp为四层健康检查配置的关键字,表示使用TCP协议
demo为对应下发的upstream的name
stream 为stream类型的上游健康检查指定参数。
stream.send为期望发送的文本，对于不可见字符，可使用16进制方式表示，格式为\\x[a-f0-9]{1,2},配置时可与普通文本串混合使用。
stream.expect 为期望收到的文件内容串，对于不可见字符，可使用16进制方式表示，\\x[a-f0-9]{1,2},配置时可与普通文本串混合使用
```

##### **3.6.4.2.2 健康检查TCP + TLS配置方式**

```
JSON
请求BODY
{
    "interval": "3s",
    "jitter": "1s",
    "timeout": "10s",
    "passes": 2,
    "fails": 1,
    "stream": {                    /* 开启四层健康检查 */
        "send": "zhao\\x6B\\x61\\x6E\\x67",        /* 期望发送的文本 */
        "expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"      /* 期望收到的文本 */
    },
  "ssl": {
    "enable": true,  /* 是否启用TLS */
     "ntls": true, /* 是否是国密算法 */
     "ciphers":"ECC-SM2-SM4-CBC-SM3:ECDHE-SM2-WITH-SM4-SM3:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:ECDHE-RSA-AES128-SHA256:!aNULL:!eNULL:!RC4:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS" /* 支持算法 */
  }
}
请求命令
curl -s http://127.0.0.1:7071/hc/1/hc/stcp/demo -XPOST -d '{"interval": "3s","jitter": "1s","timeout": "10s","passes": 2, "fails": 1,"stream": {"send": "zhao\\x6B\\x61\\x6E\\x67","expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"},"ssl": {"enable": true,"ntls": true,"ciphers":"ECC-SM2-SM4-CBC-SM3:ECDHE-SM2-WITH-SM4-SM3:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:ECDHE-RSA-AES128-SHA256:!aNULL:!eNULL:!RC4:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS" }}'
返回
{
    "code": 0,
    "msg": "success"
}
参数说明：
stcp 为四层健康检查配置的关键字,表示使用TCP协议
demo 为对应下发的upstream的name
stream 意义同 “健康检查TCP配置方式”
ssl TLS相关配置
ssl.enable 是否启tls，默认 false
ssl.ntls 是否是国密算法.
ssl.ciphers 支持的算法 默认"DEFAULT"
```

##### 3.6.4.2.3 **健康检查**UDP配置方式

```
JSON
请求BODY
{
    "interval": "3s",
    "jitter": "1s",
    "timeout": "10s",
    "passes": 2,
    "fails": 1,
    "stream": {                    /* 开启四层健康检查 */
        "send": "zhao\\x6B\\x61\\x6E\\x67",        /* 期望发送的文本 */
        "expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"      /* 期望收到的文本 */
    }
}
请求命令
curl -s http://127.0.0.1:7071/hc/1/hc/sudp/tmux -XPOST -d '{"interval": "3s","jitter": "1s","timeout": "10s","passes": 2, "fails": 1, "stream": {"send": "zhao\\x6B\\x61\\x6E\\x67","expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"}}'
返回
{
    "code": 0,
    "msg": "success"
}

参数说明：
sudp为四层健康检查配置的关键字
tmux为对应下发的upstream的name
stream 意义同 “健康检查TCP配置方式”

UDP方式不支持TLS
请求命令
curl -s http://127.0.0.1:7071/hc/1/hc/sudp/demo -XPOST -d '{"interval": "3s","jitter": "1s","timeout": "10s","passes": 2, "fails": 1,"stream": {"send": "zhao\\x6B\\x61\\x6E\\x67","expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"},"ssl": {"enable": true,"ntls": true,"ciphers":"ECC-SM2-SM4-CBC-SM3:ECDHE-SM2-WITH-SM4-SM3:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:ECDHE-RSA-AES128-SHA256:!aNULL:!eNULL:!RC4:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS" }}'
返回
{
    "code": 14,
    "msg": "UDP does not support tls"
}
```



### 3.6.5 **调用样例**

```
以下请求请注意替换ip 、端口 、uri
```

####  3.6.5.1 **增加新的健康检查**

请求

```
HTTP
POST http://127.0.0.1:8081/hc/1/hc/http/demo
Content-Type: application/json

 {
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
  "http": {
    "header": [
      "Host"
    ],
    "uri": "/test",
    "body": "~ ok",
    "status": "200 204"
  },
  "ssl": {
    "enable": false
  }
 }
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:06:24 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```

#### 3.6.5.2 **删除健康检查**

请求

```
HTTP
DELETE http://127.0.0.1:8081/hc/1/hc/http/demo
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:06:21 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```

#### 3.6.5.3 **查看健康检查列表**

请求

```
Nginx
GET http://127.0.0.1:8081/hc/1/hc
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:07:12 GMT
Content-Type: application/json
Content-Length: 44
Connection: keep-alive

[
  {
    "upstream": "demo",
    "type": "http"
  }
]
```

#### 3.6.5.4 **查看健康检查配置详情**

请求

```
HTTP
GET http://127.0.0.1:8081/hc/1/hc/http/demo
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:07:14 GMT
Content-Type: application/json
Content-Length: 157
Connection: keep-alive

{
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
  "http": {
    "uri": "/test",
    "status": "200",
    "body": "~ ok",
    "header": [
      "Host"
    ]
  }
}
```

#### 3.6.5.5 http**健康检查标密**SSL配置

请求

```
HTTP
POST http://127.0.0.1:8081/hc/1/hc/http/demos
Content-Type: application/json

{
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
  "http": {
    "uri": "/ssl",
    "status": "200"
  },
  "ssl": {
    "enable": true,
    "ntls": false 
}
```

返回

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Mon, 13 Feb 2023 07:35:27 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```

#### 3.6.5.6 增加新的**stream**，TCP健康检查

请求

```
HTTP
POST http://127.0.0.1:8081/hc/1/hc/stcp/demo
Content-Type: application/json

 {
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
  "stream": {                   
       "send": "zhao\\x6B\\x61\\x6E\\x67",        /* 期望发送的文本 */
       "expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"      /* 期望收到的文本 */
  },
  "ssl": {
    "enable": false
  }
 }
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:06:24 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```

#### 3.6.5.7 增加新的**stream**，UDP健康检查

请求

```
HTTP
POST http://127.0.0.1:8081/hc/1/hc/sudp/demo
Content-Type: application/json

 {
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
  "stream": {                   
       "send": "zhao\\x6B\\x61\\x6E\\x67",        /* 期望发送的文本 */
       "expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"      /* 期望收到的文本 */
  },
  "ssl": {
    "enable": false
  }
 }
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:06:24 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```

#### 3.6.5.8 http健康检查国密SSL配置

请求

```
HTTP
POST http://127.0.0.1:8081/hc/1/hc/http/demos
Content-Type: application/json

{
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
  "http": {
    "uri": "/gmssl",
    "status": "200"
  },
  "ssl": {
    "enable": true,
    "ntls": true,
    "ciphers":"ECC-SM2-SM4-CBC-SM3:ECDHE-SM2-WITH-SM4-SM3:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:ECDHE-RSA-AES128-SHA256:!aNULL:!eNULL:!RC4:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS"
}
```

返回

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Mon, 13 Feb 2023 07:35:27 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```

####  3.6.5.9 stream健康检查标密SSL配置

请求

```
HTTP
POST http://127.0.0.1:8081/hc/1/hc/stcp/demos
Content-Type: application/json

{
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
   "stream": {                   
       "send": "zhao\\x6B\\x61\\x6E\\x67",        /* 期望发送的文本 */
       "expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"      /* 期望收到的文本 */
  },
  "ssl": {
    "enable": true,
    "ntls": false 
}
```

返回

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Mon, 13 Feb 2023 07:35:27 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```

####  3.6.5.10 stream健康检查国密SSL配置

请求

```
HTTP
POST http://127.0.0.1:8081/hc/1/hc/stcp/demos
Content-Type: application/json

{
  "interval": "3s",
  "jitter": "1s",
  "timeout": "10s",
  "passes": 2,
  "fails": 1,
   "stream": {                   
       "send": "zhao\\x6B\\x61\\x6E\\x67",        /* 期望发送的文本 */
       "expect": "\\x74\\x68\\x61\\x6E\\x6B\\x20you"      /* 期望收到的文本 */
  },
  "ssl": {
    "enable": true,
    "ntls": true，
    "ciphers":"ECC-SM2-SM4-CBC-SM3:ECDHE-SM2-WITH-SM4-SM3:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:ECDHE-RSA-AES128-SHA256:!aNULL:!eNULL:!RC4:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS"
  }
}
```

返回

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Mon, 13 Feb 2023 07:35:27 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive

{
  "code": 0,
  "msg": "success"
}
```



## 3.7 **metrics** 

### 3.7.1  **功能介绍**

提供对虚拟主机状态信息的访问。包含当前状态、服务器、上游、缓存等信息。

首先，指令 vhost_traffic_status_zone 是必需的， 然后如果指令 vhost_traffic_status_display 设置后，可按如下方式访问：

/metrics/format/json

- 如果您要求 /metrics/format/json，将使用包含当前活动数据的 JSON 文档进行响应，以便在实时仪表板和第三方监视工具中使用。
- /metrics/format/html
	- 如果您要求 /metrics/format/html，将使用 HTML 中的内置实时仪表板进行响应，该仪表板在内部请求 /metrics/format/json.
- /metrics/format/jsonp
	- 如果您要求 /status/format/jsonp，将使用包含当前活动数据的 JSONP 回调函数进行响应，以便在实时仪表板和第三方监视工具中使用。
- /metrics/format/prometheus
	- 如果您要求 /status/format/prometheus，将响应 prometheus 格式，包含当前活动数据的文档。

- vts新指标   统计从收到client请求第一个字节到proxy送给上游服务器前这段时间的百分位数，包括50%, 99%, 99.9%, 99.99%, 99.999%这些百分位的请求延时。具体含义如下：
	-    "p50reqdelayMsecr": 0,  50%的请求时间延迟应该是0ms(50%的请求应该比给定的延迟更快.换句话说,只允许1%的请求更慢.)

	-   "p99reqdelayMsecr": 1,   99%的请求时间延迟应该是1ms

	-   "p999reqdelayMsecr": 1,  99.9%的请求时间延迟应该是1ms

	-   "p9999reqdelayMsecr": 1,  99.99%的请求时间延迟应该是1ms "p99999reqdelayMsecr": 1,  99.999%的请求时间延迟应该是1ms

- 统计上游服务器的超时次数 ：依赖于反向代理的业务处理，在反向代理的过程中，会选择一个上游服务器处理client的请求。
	-  当超时发生后反向代理会记录这种情况，并尝试选择下一个上游服务器进行处理并记录处理情况。vts新增的     该指标，是把反向代理过程中尝试过的上游服务器的超时次数进行统计，展现给用户。在上图所示中，会记录两个超时次数。对应于error.log中的upstream timed out 。

![image-20230717103936492](https://gitee.com/gebona/picture/raw/master/202307171039846.png)

#### 3.7.1.1  **新指标时间延迟百分位数设计到的知识**

  计算时间延迟设计到日志时间变量：

$request_time:从发起请求的客户端获取到第一个字节开始，到返回给客户端最后一个字节后，日志写入文件所经过的时间。单位为秒。

$upstream_connect_time:从与上游服务开始建立连接，到连接建立成功，所经过的时间。单位为秒。

$upstream_header_time:从与上游服务开始建立连接，到接收到响应返回头的第一个字节，所经过的时间。单位为秒。

$upstream_response_time:从与上游服务开始建立连接，到接收完响应返回的最后一个字节，所经过的时间。单位为秒。

结合以上释义，便可得到时序图：

![image-20230717140228243](https://gitee.com/gebona/picture/raw/master/202307171402618.png)

从上图中可以得出时间延迟的计算方法为：

时间延迟=$request_time -$upstream_response_time + $upstream_connect_time';

测试时需要设置log_format aaaa '$request_time   $upstream_response_time   $upstream_connect_time'和

access_log logs/access.log aaaa;两个指令获得$request_time 、$upstream_response_time和   $upstream_connect_time。从access.log 文件中获取这上述提到的3个时间：$request_time，$upstream_response_time，$upstream_connect_time。

测试步骤：

①njet.conf

```
Bash
helper broker modules/njt_helper_broker_module.so
conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so
conf/ctrl.conf;

load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 
load_module modules/njt_http_lua_module.so;
load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

cluster_name helper;
node_name node1;

worker_processes auto;  

error_log logs/error.log info;    
pid logs/njet.pid;   

events {
        worker_connections 1024; 
}

http {
        dyn_kv_conf conf/iot-work.conf;
        include mime.types;   
        include conf.d/*.conf;
  log_format aaaa  '$request_time     $upstream_response_time     $upstream_connect_time';
 # log_format aaaa  '"request_time:$request_time upstream_response_time:$upstream_response_time upstream_connect_time:$upstream_connect_time"'
 # '$remote_addr - $remote_user [$time_local] "$request" '
 # '$status $body_bytes_sent "$http_referer" '
 # '"$http_user_agent" "$http_x_forwarded_for"';
  access_log logs/access.log aaaa;
 
  vhost_traffic_status_zone; 
  vhost_traffic_status_filter_by_set_key $request_uri "$realip_remote_addr to $server_name";
  variables_hash_max_size 2048;
  proxy_read_timeout 1;

  proxy_cache_path cache1 levels=1:2  keys_zone=cache1:20m  max_size=2g  inactive=1m  use_temp_path=off; 
  
  proxy_cache_valid any 10m ;
  proxy_cache_revalidate on;
  proxy_cache_lock on;
  proxy_cache_key $scheme$host$request_uri$slice_range;
  
  map $request_method  $purge_method{
      PURGE 1;
      default 0;
  }
   
  upstream staticResource {
      server 192.168.40.144;
      server 192.168.40.103;
      server 192.168.40.150;
  }
  
  
  server {
     
      listen 7111;
      server_name server7111;
      location ~ .*\.(gif|jpg|jpeg|png|rar|html|txt|mp3|mp4)$ {
       #$upstream_cache_status表示资源缓存的状态，有HIT MISS EXPIRED三种状态
       add_header X-Cache $upstream_cache_status;
       #proxy_cache_bypass $uri !='txt1.txt';
      # proxy_cache_bypass $uri !='png1.png';
       proxy_pass http://staticResource;
       #proxy_cache cache1;
       
      }
  }
  
 upstream apiResource {
    server 192.168.40.144:8082 fail_timeout=6s;
    server 192.168.40.103:8082 weight=5;
    server 192.168.40.150:8082 max_fails=10;
  }
  
  upstream backend {
        server 192.168.40.158:8082 ;
 }
 
 server {
                listen 8081;
                server_name server8081;

                location / {
      
       proxy_pass http://backend;
                
                }
  
        }

        server {
                listen 8082;
                server_name server8082;
                    access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;

                
                }
  
        }
 
 server {
                listen 8093;
                server_name server8093;

                location / {
      
       return 200 text8093;
                
                }
  
        }
 
 server {
                listen 8094;
                server_name server8094;

                location / {
     
       return 200 text8094;
                
                }
  
        }
 server {
                listen 8095;
                server_name server8095;

                location / {
     
       return 200 text8095;
                
                }
  
        }
 
  server {
    listen 8084;
    server_name server8084;
    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
       
        proxy_pass http://apiResource/;
    }
  }

  server {
                listen 8086;
                server_name server8086;

                location /a {
                      
                        proxy_pass http://192.168.40.246:9008;
                }
   location / {
                       
                    return 200 '<h1>server:8086</h1>';
                }
        }
}
```

②通过postman 发送1000 个请求-采集到的时间放在access.log 里；编写计算$request_time-$upstream_response_time+$upstream_connect_time的时间差值，并统计时间值的数量，判断是否和vts 采集指标一致

```
Bash
#!/bin/bash

cd /home/njet/test_vtsnew2/logs/test
filename="/home/njet/test_vtsnew2/logs/test/access.log"
awk '{ sum = $1 - $2 + $3; print sum }' $filename >out.log
awk '{a[$1]++}END{for(i in a) print i, a[i]}' out.log
```

新指标timeout 次数统计：

①njet.conf

```
Bash
helper broker modules/njt_helper_broker_module.so
conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so
conf/ctrl.conf;

load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 
load_module modules/njt_http_lua_module.so;

load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

cluster_name helper;
node_name node1;

worker_processes auto;   
 
error_log logs/error.log info;    
pid logs/njet.pid;   
events {
        worker_connections 1024;  
}

http {
        dyn_kv_conf conf/iot-work.conf;
        include mime.types;  
        include conf.d/*.conf;
  vhost_traffic_status_zone; 
  vhost_traffic_status_filter_by_set_key $request_uri "$realip_remote_addr to $server_name";
  variables_hash_max_size 2048;
  
  proxy_cache_path cache1 levels=1:2  keys_zone=cache1:20m  max_size=2g  inactive=1m  use_temp_path=off; 
  
  proxy_cache_valid any 10m ;
  proxy_cache_revalidate on;
  proxy_cache_lock on;
  proxy_cache_key $scheme$host$request_uri$slice_range;
  
  map $request_method  $purge_method{
      PURGE 1;
      default 0;
  }
  
  
  upstream staticResource {
      server 192.168.40.144;
      server 192.168.40.103;
      server 192.168.40.150;
  }
  
 
  upstream backend {
        server 192.168.40.158:8082;
        server 192.168.40.158:8083;
        server 192.168.40.158:8084;
        server 192.168.40.158:8085;
 }
 
 server {
                listen 8081;
                server_name server8081;

                location / {
       proxy_pass http://backend;
       proxy_next_upstream_timeout 6;
       proxy_read_timeout 1;
                
                }
  
        }

        server {
                listen 8082;
                server_name server8082;
                    access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;                
                } 
        }
 
 
 
 server {
                listen 8083;
                server_name server8083;
                access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;
                }
  
        }
 
 
 
  server {
                listen 8084;
                server_name server8083;
                    access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;
                }
  
        }
 
 server {
                listen 8085;
                server_name server8083;
                access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;
                }
        }
 

}         
```

main.lua 中配置：

```
Bash
njt.sleep(2)
--local args,err = njt.req.get_uri_args()
--local k= args["k"]
njt.say("hello world:")
```

②启动njet 包，每秒发送一次curl 命令，判断timeout 次数是否在增加

```
Bash
watch -n1  'curl -k http://192.168.40.158:8081/'
```

#### 3.7.1.2  **JSON** **示例**

```
{
    "hostName": ...,
    "moduleVersion": ...,
    "nginxVersion": ...,
    "loadMsec": ...,
    "nowMsec": ...,
    "connections": {
        "active":...,
        "reading":...,
        "writing":...,
        "waiting":...,
        "accepted":...,
        "handled":...,
        "requests":...
    },
    "sharedZones": {
        "name":...,
        "maxSize":...,
        "usedSize":...,
        "usedNode":...
    },
    "serverZones": {
        "...":{
            "requestCounter":...,
            "inBytes":...,
            "outBytes":...,
            "responses":{
                "1xx":...,
                "2xx":...,
                "3xx":...,
                "4xx":...,
                "5xx":...,
                "miss":...,
                "bypass":...,
                "expired":...,
                "stale":...,
                "updating":...,
                "revalidated":...,
                "hit":...,
                "scarce":...
            },
            "requestMsecCounter":...,
            "requestMsec":...,
            "requestMsecs":{
                "times":[...],
                "msecs":[...]
            },
            "requestBuckets":{
                "msecs":[...],
                "counters":[...]
            },
        }
        ...
    },
    "filterZones": {
        "...":{
            "...":{
                "requestCounter":...,
                "inBytes":...,
                "outBytes":...,
                "responses":{
                    "1xx":...,
                    "2xx":...,
                    "3xx":...,
                    "4xx":...,
                    "5xx":...,
                    "miss":...,
                    "bypass":...,
                    "expired":...,
                    "stale":...,
                    "updating":...,
                    "revalidated":...,
                    "hit":...,
                    "scarce":...
                },
                "requestMsecCounter":...,
                "requestMsec":...,
                "requestMsecs":{
                    "times":[...],
                    "msecs":[...]
                },
                "requestBuckets":{
                    "msecs":[...],
                    "counters":[...]
                },
            },
            ...
        },
        ...
    },
    "upstreamZones": {
        "...":[
            {
                "server":...,
                "requestCounter":...,
                "inBytes":...,
                "outBytes":...,
                "responses":{
                    "1xx":...,
                    "2xx":...,
                    "3xx":...,
                    "4xx":...,
                    "5xx":...
                },
                "requestMsecCounter":...,
                "requestMsec":...,
                "requestMsecs":{
                    "times":[...],
                    "msecs":[...]
                },
                "requestBuckets":{
                    "msecs":[...],
                    "counters":[...]
                },
                "responseMsecCounter":...,
                "responseMsec":...,
                "responseMsecs":{
                    "times":[...],
                    "msecs":[...]
                },
                "responseBuckets":{
                    "msecs":[...],
                    "counters":[...]
                },
                "weight":...,
                "maxFails":...,
                "failTimeout":...,
                "backup":...,
                "down":...
            }
            ...
        ],
        ...
    }
    "cacheZones": {
        "...":{
            "maxSize":...,
            "usedSize":...,
            "inBytes":...,
            "outBytes":...,
            "responses":{
                "miss":...,
                "bypass":...,
                "expired":...,
                "stale":...,
                "updating":...,
                "revalidated":...,
                "hit":...,
                "scarce":...
            }
        },
        ...
    }
}
 
```

#### 3.7.1.3 **JSON** **字段含义**

以下含义以 JSON 示例顺序提供：

```
hostName
主机名
moduleVersion
{Version}(|.dev.{commit})格式的模块版本。
njetVersion
njet版本。
loadMsec
加载过程时间（以毫秒为单位）。
nowMsec
当前时间（毫秒）
 
connections
active
当前活动客户端连接数。
reading
读取客户端连接的总数。
writing
写入客户端连接的总数。
waiting
等待客户端连接的总数。
accepted
接受的客户端连接总数。
handled
已处理的客户端连接总数。
requests
请求的客户端连接总数。
 
sharedZones
name
在配置中指定的共享内存的名称。（默认值： vhost_traffic_status)
maxSize
在配置中指定的共享内存的最大大小的限制。
usedSize
共享内存的当前大小。
usedNode
共享内存中使用的当前节点数。它可以通过以下公式获得一个节点的大致大小:(usedSize / usedNode)
 
serverZones
requestCounter
从客户端收到的客户端请求总数。
inBytes
从客户端接收的总字节数。
outBytes
发送到客户端的总字节数。
 
responses
1xx， 2xx， 3xx， 4xx， 5xx
状态代码为 1xx、2xx、3xx、4xx 和 5xx 的响应数。
miss
cache miss的个数。
bypass
缓存绕过的数量。
expired
缓存过期的数量。
stale
缓存过时的数量。
updating
缓存更新的次数。
revalidated
重新验证的缓存数。
hit
缓存命中数。
scarce
缓存不足的数量。
 
requestMsecCounter
累积请求处理时间的次数（以毫秒为单位）。
requestMsec
请求处理时间的平均时间（以毫秒为单位）。
requestMsecs
times
请求处理时间以毫秒为单位。
msecs
请求处理时间（包括上游），以毫秒为单位。
requestBuckets
msecs
由vhost_traffic_status_histogram_buckets指令设置的柱状图的桶值。
counters
每个存储桶值大于或等于请求处理时间的累积值。
 
filterZones
它提供了与serverzone相同的字段，只是包含了组名。
upstreamZones
server
服务器的地址。
requestCounter
转发到此服务器的客户端连接总数。
inBytes
从此服务器接收的总字节数。
outBytes
发送到此服务器的总字节数。
responses
1xx， 2xx， 3xx， 4xx， 5xx
状态代码为 1xx、2xx、3xx、4xx 和 5xx 的响应数。
requestMsecCounter
包括上游在内的累计请求处理时间数（以毫秒为单位）。
requestMsec
请求处理时间（包括上游）的平均时间（以毫秒为单位）。
requestMsecs
times
请求处理时间以毫秒为单位。
msecs
请求处理时间（包括上游），以毫秒为单位。
requestBuckets
msecs
由vhost_traffic_status_histogram_buckets指令设置的柱状图的桶值。
counters
每个存储桶值大于或等于请求处理时间（包括上游）的累积值。
responseMsecCounter
累积的仅上游响应处理时间的数量，以毫秒为单位。
responseMsec
仅上游响应处理时间的平均值（以毫秒为单位）。
responseMsecs
times
请求处理时间以毫秒为单位。
msecs
唯一的上游响应处理时间（以毫秒为单位）。
responseBuckets
msecs
由vhost_traffic_status_histogram_buckets指令设置的柱状图的桶值。
counters
每个存储桶值大于或等于唯一上游响应处理时间的累积值。
weight
当前服务器设置的权重
maxFails
服务器当前max_fails设置。
failTimeout
服务器当前fail_timeout设置。
backup
服务器当前备份设置。
down
服务器当前down设置。基本上，这只是一个标记ngx_http_upstream_module的服务器关闭(例如。服务器backend3.example.com关闭)，而不是实际的上游服务器状态。如果你启用了upstream zone指令，它将改变为实际状态。
cacheZones
maxSize
在配置中指定的缓存的最大大小的限制。如果proxy_cache_path指令中的max_size未指定，则默认分配系统依赖值NGX_MAX_OFF_T_VALUE
usedSize
缓存的当前大小。这个值取自njet，就像上面的maxSize值一样。
inBytes
从缓存接收的总字节数。
outBytes
从缓存发送的总字节数。
responses
miss
缓存未命中数。
bypass
缓存绕过的数量。
expired
缓存过期的数量。
stale
缓存过时的数量。
updating
缓存更新的次数。
revalidated
重新验证的缓存数。
hit
缓存命中数。
scarce
缓存不足的数量。
```

### 3.7.2 **功能展示**

 vts监控页面：

![image-20230616171426065](https://gitee.com/gebona/picture/raw/master/202306161714300.png)

 swagger页面：

![image-20230616171453077](https://gitee.com/gebona/picture/raw/master/202306161714421.png)

### 3.7.3  **配置说明**

#### 3.7.3.1  **vhost_traffic_status**

![img](https://gitee.com/gebona/picture/raw/master/202306161715474.jpg)

​                                                                                                  **点击图片可查看完整电子表格**

描述: 启用或禁用数据面统计指定上下文内性能指标数据。如果设置 了vhost_traffic_status_zone 指令，将自动启用。

#### 3.7.3.2  **vhost_traffic_status_zone**

![img](https://gitee.com/gebona/picture/raw/master/202306161716215.jpg)

​                                                                                             **点击图片可查看完整电子表格**

描述: 为数据面共享内存区域设置参数，该区域将保留各种键的状态。缓存在所有工作进程之间共享。

在大多数情况下，njet-module-vts使用的共享内存大小不会增加太多。当使用vhost_traffic_status_filter_by_set_key指令时，共享内存大小会增加很多，但如果过滤器的键是固定的(例如。国家代码的总数约为240)，则不会持续增加。

如果使用vhost_traffic_status_filter_by_set_key指令，设置如下:

￮    默认设置为上限32M的共享内存大小。(vhost_traffic_status_zone shared:vhost_traffic_status:32 m) 

￮    如果在error_log中打印出：(“ngx_slab_alloc() failed: no memory in vhost_traffic_status_zone”)，则增加到大于(usedSize * 2)。

#### 3.7.3.3  **vhost_traffic_status_filter_by_set_key**

![img](https://gitee.com/gebona/picture/raw/master/202306161717351.jpg)

​                                                                                            **点击图片可查看完整电子表格**

描述: 通过用户定义的变量作为展示的维度。key和name可以使用变量，如$host、$server_name等

示例如下：

```
http{
  vhost_traffic_status_filter_by_set_key $request_uri "$realip_remote_addr to $server_name";
  ...
}
```

#### 3.7.3.4 **vhost_traffic_status_display**

<table>
<tbody>
<tr>
<td width="92">
<p>-</p>
</td>
<td width="277">
<p>-</p>
</td>
</tr>
<tr>
<td width="92">
<p>语法</p>
</td>
<td width="277">
<p>vhost_traffic_status_display</p>
</td>
</tr>
<tr>
<td width="92">
<p>默认</p>
</td>
<td width="277">
<p>-</p>
</td>
</tr>
<tr>
<td width="92">
<p>上下文</p>
</td>
<td width="277">
<p>location</p>
</td>
</tr>
<tr>
<td width="92">
<p>配置</p>
</td>
<td width="277">
<p>控制面</p>
</td>
</tr>
</tbody>
</table>

描述: 启用或禁用控制面显示处理程序。

#### 3.7.3.5  **vhost_traffic_status_display_format**

<table>
<tbody>
<tr>
<td width="192">
<p>-</p>
</td>
<td width="320">
<p>-</p>
</td>
</tr>
<tr>
<td width="192">
<p>语法</p>
</td>
<td width="320">
<p>vhost_traffic_status_display_format &lt;json|html|jsonp|prometheus&gt;</p>
</td>
</tr>
<tr>
<td width="192">
<p>默认</p>
</td>
<td width="320">
<p>json</p>
</td>
</tr>
<tr>
<td width="192">
<p>上下文</p>
</td>
<td width="320">
<p>location</p>
</td>
</tr>
<tr>
<td width="192">
<p>配置</p>
</td>
<td width="320">
<p>控制面</p>
</td>
</tr>
</tbody>
</table>

描述: 设置控制面数据输出的格式。

￮    如果设置 json，将使用 JSON 格式进行响应；

￮    如果设置 html，将使用 HTML 中的内置实时仪表板进行响应；

￮    如果设置 jsonp，将使用 JSONP 回调函数（默认： ngx_http_vhost_traffic_status_jsonp_callback)；

￮    如果设置 prometheus，将响应 prometheus格式的数据。

#### 3.7.3.6  **配置示例**

##### 3.7.3.6.1 **控制面**

```
load_module modules/njt_http_vtsc_module.so;
error_log logs/error-ctrl.log info;
events {
....
}
http {
    access_log logs/access-ctrl.log combined;
    server {
        listen       8081;
       
        location / {
            return 200 "welcome to sub http server\n";
        }
        location /status {
            vhost_traffic_status_display;
            vhost_traffic_status_display_format html;
        }
    }
}
```

##### 3.7.3.6.2 **数据面**

```
load_module modules/njt_http_vtsd_module.so;
helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;

error_log logs/error.log info;
events {
....
}
http {
    vhost_traffic_status_zone;
    vhost_traffic_status_filter_by_set_key $request_uri "$realip_remote_addr to $server_name";
....
....
}
```

### 3.7.4 **动态开关**

vts模块可以通过动态配置接口实现对是否统计及统计指标的动态配置，详见：动态开关功能

### 3.7.5 **接口消息返回提示**

#### 3.7.5.1 **查询**

第一种方法：

```
Bash
curl -X 'GET' \
  'http://127.0.0.1:8089/config/2/config/http_vts' \
  -H 'accept: application/json'
```

#### 3.7.5.2 **配置修改**

```
Bash
curl -X PUT -d @input.json http://127.0.0.1:8089/config/2/config/http_vts >out.json
```

#### 3.7.5.3 **结果**

get 返回结果示例：

```
C
{
    "vhost_traffic_status_filter_by_set_key":"\"$request_uri\" \"$server_name\"",
    "servers":[
        {
            "listens":[
                "0.0.0.0:8082"
            ],
            "serverNames":[
                "localhost"
            ],
            "locations":[
                {
                    "location":"/",
                    "vhost_traffic_status":false
                }
            ]
        },
        {
            "listens":[
                "0.0.0.0:9001"
            ],
            "serverNames":[
                "localhost"
            ],
            "locations":[
                {
                    "location":"/",
                    "vhost_traffic_status":true
                },
                {
                    "location":"= /notexist",
                    "vhost_traffic_status":true
                }
            ]
        },
        {
            "listens":[
                "0.0.0.0:9002"
            ],
            "serverNames":[
                "localhost"
            ],
            "locations":[
                {
                    "location":"/",
                    "vhost_traffic_status":true
                },
                {
                    "location":"= /50x.html",
                    "vhost_traffic_status":true
                }
            ]
        },
        {
            "listens":[
                "0.0.0.0:9003"
            ],
            "serverNames":[
                "localhost"
            ],
            "locations":[
                {
                    "location":"/",
                    "vhost_traffic_status":true
                },
                {
                    "location":"= /50x.html",
                    "vhost_traffic_status":true
                }
            ]
        }
    ]
}
```

Post 返回结果示例：

```
C

{
  "code": 0,
  "msg": "success"
}
```

#### 3.7.5.4 **新指标配置时间延迟百分位数（**njet.conf)

```
Bash
helper broker modules/njt_helper_broker_module.so
conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so
conf/ctrl.conf;

load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 
load_module modules/njt_http_lua_module.so;
load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

cluster_name helper;
node_name node1;

worker_processes auto;  

error_log logs/error.log info;    
pid logs/njet.pid;   

events {
        worker_connections 1024; 
}

http {
        dyn_kv_conf conf/iot-work.conf;
        include mime.types;   
        include conf.d/*.conf;
  log_format aaaa  '$request_time     $upstream_response_time     $upstream_connect_time';
 # log_format aaaa  '"request_time:$request_time upstream_response_time:$upstream_response_time upstream_connect_time:$upstream_connect_time"'
 # '$remote_addr - $remote_user [$time_local] "$request" '
 # '$status $body_bytes_sent "$http_referer" '
 # '"$http_user_agent" "$http_x_forwarded_for"';
  access_log logs/access.log aaaa;
 
  vhost_traffic_status_zone; 
  vhost_traffic_status_filter_by_set_key $request_uri "$realip_remote_addr to $server_name";
  variables_hash_max_size 2048;
  proxy_read_timeout 1;

  proxy_cache_path cache1 levels=1:2  keys_zone=cache1:20m  max_size=2g  inactive=1m  use_temp_path=off; 
  
  proxy_cache_valid any 10m ;
  proxy_cache_revalidate on;
  proxy_cache_lock on;
  proxy_cache_key $scheme$host$request_uri$slice_range;
  
  map $request_method  $purge_method{
      PURGE 1;
      default 0;
  }
   
  upstream staticResource {
      server 192.168.40.144;
      server 192.168.40.103;
      server 192.168.40.150;
  }
  
  
  server {
     
      listen 7111;
      server_name server7111;
      location ~ .*\.(gif|jpg|jpeg|png|rar|html|txt|mp3|mp4)$ {
       #$upstream_cache_status表示资源缓存的状态，有HIT MISS EXPIRED三种状态
       add_header X-Cache $upstream_cache_status;
       #proxy_cache_bypass $uri !='txt1.txt';
      # proxy_cache_bypass $uri !='png1.png';
       proxy_pass http://staticResource;
       #proxy_cache cache1;
       
      }
  }
  
 upstream apiResource {
    server 192.168.40.144:8082 fail_timeout=6s;
    server 192.168.40.103:8082 weight=5;
    server 192.168.40.150:8082 max_fails=10;
  }
  
  upstream backend {
        server 192.168.40.158:8082 ;
 }
 
 server {
                listen 8081;
                server_name server8081;

                location / {
      
       proxy_pass http://backend;
                
                }
  
        }

        server {
                listen 8082;
                server_name server8082;
                    access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;

                
                }
  
        }
 
 server {
                listen 8093;
                server_name server8093;

                location / {
      
       return 200 text8093;
                
                }
  
        }
 
 server {
                listen 8094;
                server_name server8094;

                location / {
     
       return 200 text8094;
                
                }
  
        }
 server {
                listen 8095;
                server_name server8095;

                location / {
     
       return 200 text8095;
                
                }
  
        }
 
  server {
    listen 8084;
    server_name server8084;
    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
       
        proxy_pass http://apiResource/;
    }
  }

  server {
                listen 8086;
                server_name server8086;

                location /a {
                      
                        proxy_pass http://192.168.40.246:9008;
                }
   location / {
                       
                    return 200 '<h1>server:8086</h1>';
                }
        }
}
```

#### 3.7.5.5 **结果**

通过postman 发送1000个请求后统计的时间延迟结果如下：

```
Bash

cd /home/njet/test_vtsnew2/logs/test
filename="/home/njet/test_vtsnew2/logs/test/access.log"
awk '{ sum = $1 - $2 + $3; print sum }' $filename >out.log
awk '{a[$1]++}END{for(i in a) print i, a[i]}' out.log
```

结果截图：

![image-20230717141039642](https://gitee.com/gebona/picture/raw/master/202307171410112.png)

Vts 采集页面展示：

![image-20230717141104720](https://gitee.com/gebona/picture/raw/master/202307171411530.png)

#### 3.6.5.6 **新指标配置**timeout**次数统计（**njet.conf)

```
Bash
helper broker modules/njt_helper_broker_module.so
conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so
conf/ctrl.conf;

load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 
load_module modules/njt_http_lua_module.so;

load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

cluster_name helper;
node_name node1;

worker_processes auto;   
 
error_log logs/error.log info;    
pid logs/njet.pid;   
events {
        worker_connections 1024;  
}

http {
        dyn_kv_conf conf/iot-work.conf;
        include mime.types;  
        include conf.d/*.conf;
  vhost_traffic_status_zone; 
  vhost_traffic_status_filter_by_set_key $request_uri "$realip_remote_addr to $server_name";
  variables_hash_max_size 2048;
  
  proxy_cache_path cache1 levels=1:2  keys_zone=cache1:20m  max_size=2g  inactive=1m  use_temp_path=off; 
  
  proxy_cache_valid any 10m ;
  proxy_cache_revalidate on;
  proxy_cache_lock on;
  proxy_cache_key $scheme$host$request_uri$slice_range;
  
  map $request_method  $purge_method{
      PURGE 1;
      default 0;
  }
  
  
  upstream staticResource {
      server 192.168.40.144;
      server 192.168.40.103;
      server 192.168.40.150;
  }
  
 
  upstream backend {
        server 192.168.40.158:8082;
        server 192.168.40.158:8083;
        server 192.168.40.158:8084;
        server 192.168.40.158:8085;
 }
 
 server {
                listen 8081;
                server_name server8081;

                location / {
       proxy_pass http://backend;
       proxy_next_upstream_timeout 6;
       proxy_read_timeout 1;
                
                }
  
        }

        server {
                listen 8082;
                server_name server8082;
                    access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;                
                } 
        }
 
 
 
 server {
                listen 8083;
                server_name server8083;
                access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;
                }
  
        }
 
 
 
  server {
                listen 8084;
                server_name server8083;
                    access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;
                }
  
        }
 
 server {
                listen 8085;
                server_name server8083;
                access_log  /home/njet/test_vtsnew2/logs/8082.log;

                location / {
       content_by_lua_file main.lua;
                }
        }
 

}
```

#### 3.7.5.7 **结果**

```
Bash
   通过 watch -n1  'curl -k http://192.168.40.158:8081/'   这个指令每秒访问一次，查看vts界面中timeout 次数统计是否正确
```

![image-20230717141232121](https://gitee.com/gebona/picture/raw/master/202307171412091.png)

### 3.7.6 API接口ACL控制

该模块可以设置acl控制，具体设置方法参考3.25章节描述。

## 3.8 **telemetry**功能

### 3.8.1 **介绍**

#### 3.8.1.1 什么是OpenTelemetry？

OpenTelemetry合并了OpenTracing和OpenCensus项目，提供了一组API和库来标准化遥测数据的采集和传输。OpenTelemetry提供了一个安全，厂商中立的工具，这样就可以按照需要将数据发往不同的后端。

OpenTelemetry项目由如下组件构成：

￮    推动在所有项目中使用一致的规范

￮    基于规范的，包含接口和实现的APIs

￮    不同语言的SDK(APIs的实现)，如 Java, Python, Go, Erlang等

￮    Exporters：可以将数据发往一个选择的后端

￮    Collectors：厂商中立的实现，用于处理和导出遥测数据

#### 3.8.1.2 **术语**

￮    Traces：记录经过分布式系统的请求活动，一个trace是spans的有向无环图

￮    Spans：一个trace中表示一个命名的，基于时间的操作。Spans嵌套形成trace树。每个trace包含一个根span，描述了端到端的延迟，其子操作也可能拥有一个或多个子spans。

￮    Metrics：在运行时捕获的关于服务的原始度量数据。Opentelemetry定义的metric instruments(指标工具)如下。Observer支持通过异步API来采集数据，每个采集间隔采集一个数据。

￮    Context：一个span包含一个span context，它是一个全局唯一的标识，表示每个span所属的唯一的请求，以及跨服务边界转移trace信息所需的数据。OpenTelemetry 也支持correlation context，它可以包含用户定义的属性。correlation context不是必要的，组件可以选择不携带和存储该信息。

￮    Context propagation：表示在不同的服务之间传递上下文信息，通常通过HTTP首部。Context propagation 是 OpenTelemetry 系统的关键功能之一。除了tracing之外，还有一些有趣的用法，如，执行A/B测试。OpenTelemetry支持通过多个协议的Context propagation来避免可能发生的问题，但需要注意的是，在自己的应用中最好使用单一的方法。

#### 3.8.1.3 **OpenTelemetry**架构

![image-20230616172608506](https://gitee.com/gebona/picture/raw/master/202306161726941.png)

opentelemetry也是个插件式的架构，针对不同的开发语言会有相应的Client组件，叫**Instrumenttation**，也就是在代码中埋点调用的api/sdk采集telemetry数据

OpenNJet主要集成了分布式服务 telemetry 模块：

分布式服务追踪模块:  njt_otel_module.so

### 3.8.2 **分布式服务追踪模块**: njt_otel_module.so

#### 3.8.2.1 **指令介绍**

**3.8.2.1.1 opentelemetry**

启用或禁用OpenTelemetry (默认为启用)。

￮    **required**: false

￮    **syntax**: opentelemetry on|off

￮    **block**: http, server, location



**3.8.2.1.2 opentelemetry_trust_incoming_spans**

启用或禁用使用传入请求的spans作为创建请求的父级。(默认值: 启用)。

￮    **required**: false

￮    **syntax**: opentelemetry_trust_incoming_spans on|off

￮    **block**: http, server, location



**3.8.2.1.3 opentelemetry_attribute**

向 span 添加自定义属性，可以访问 nginx 变量, 如： opentelemetry_attribute "my.user.agent" "$http_user_agent".

￮    **required**: false

￮    **syntax**: opentelemetry_attribute <key> <value>

￮    **block**: http, server, location



**3.8.2.1.4 opentelemetry_config**

Exporters, processors

￮    **required**: true

￮    **syntax**: opentelemetry_config /path/to/config.toml

￮    **block**: http



**3.8.2.1.5 opentelemetry_operation_name**

在启动新的 span 时设置操作名称。

￮    **required**: false

￮    **syntax**: opentelemetry_operation_name <name>

￮    **block**: http, server, location



**3.8.2.1.6 opentelemetry_propagate**

启用分布式跟踪头的传播, e.g. traceparent。

当没有给出父跟踪时，将启动新的跟踪。默认的传播器是 W3C。

应用的继承规则与[proxy_set_header](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)相同，这意味着当且仅当在较低的配置级别上没有定义[proxy_set_header](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) 指令时，才在当前配置级别应用该指令。

￮    **required**: false

￮    **syntax**: opentelemetry_propagate or opentelemetry_propagate b3

￮    **block**: http, server, location



**3.8.2.1.7 opentelemetry_capture_headers**

允许捕获请求和响应头(默认值: 禁用)。

￮    **required**: false

￮    **syntax**: opentelemetry_capture_headers on|off

￮    **block**: http, server, location



**3.8.2.1.8 opentelemetry_sensitive_header_names**

对于名称匹配给定正则表达式(不区分大小写)的所有头，将捕获的头值设置为[REDACTED]。

￮    **required**: false

￮    **syntax**: opentelemetry_sensitive_header_names <regex>

￮    **block**: http, server, location



**3.8.2.1.9 opentelemetry_sensitive_header_values**

将捕获的头值设置为[REDACTED]，用于所有与给定正则表达式匹配的头(不区分大小写)。

￮    **required**: false

￮    **syntax**: opentelemetry_sensitive_header_values <regex>

￮    **block**: http, server, location



**3.8.2.1.10 opentelemetry_ignore_paths**

不会为匹配给定正则表达式的 URI 创建 span (不区分大小写)。

￮    **required**: false

￮    **syntax**: opentelemetry_ignore_paths <regex>

￮    **block**: http, server, location



#### 3.8.2.2 **NJet.conf** **配置**

```
Bash
#user  root;
worker_processes  1;
daemon off;
#master_process on;
error_log  /home/njet/clbtest/logs/error.log debug;
pid  /home/njet/clbtest/njet.pid;

#load_module /root/project/telemetry/opentelemetry-cpp-contrib-main/instrumentation/nginx/build/otel_ngx_module.so;
load_module /home/njet/modules/njt_otel_module.so;

events {
    worker_connections  1024;
}


http {
    opentelemetry_config /home/njet/clbtest/conf/otel-njet.toml;
    opentelemetry off;

    upstream http1{
        server 192.168.40.139:9002;
        #server 192.168.40.136:8082;
    }

  server {
    listen 8081;
    server_name otel_example;

    location = / {
      opentelemetry on;
      opentelemetry_operation_name my_example_backend;
      opentelemetry_propagate;
      proxy_pass http://http1;
    }


    location = /b3 {
      opentelemetry on;
      opentelemetry_operation_name my_other_backend;
      opentelemetry_propagate b3;
      # Adds a custom attribute to the span
      opentelemetry_attribute "req.time" "$msec";
      proxy_pass http://http1;
    }
  }
}
```



#### 3.8.2.3 **Otel module** **配置**

```
Bash
exporter = "otlp"
processor = "batch"

[exporters.otlp]
#collector server address
# Alternatively the OTEL_EXPORTER_OTLP_ENDPOINT environment variable can also be used.
host = "192.168.40.136"
port = 4317
# Optional: enable SSL, for endpoints that support it
# use_ssl = true
# Optional: set a filesystem path to a pem file to be used for SSL encryption
# (when use_ssl = true)
# ssl_cert_path = "/path/to/cert.pem"

[processors.batch]
max_queue_size = 2048
schedule_delay_millis = 5000
max_export_batch_size = 512

[service]
# Can also be set by the OTEL_SERVICE_NAME environment variable.
name = "njet-proxy" # Opentelemetry resource name

[sampler]
name = "AlwaysOn" # Also: AlwaysOff, TraceIdRatioBased
ratio = 0.1
parent_based = false
```

### 3.8.3 **启动**collector以及Jaeger服务

通过docker-compose 启动collector 以及jaeger服务

若没有docker-compose，需先下载：

```Bash
yum install docker-compose
```

然后执行：

```Bash
docker-compose  -f docker-compose.yaml up  -d
docker-compose -f docker-compose.yaml down  #停掉jager
```

docker-compose.yaml

```
version: "2"
services:

  # Jaeger
  jaeger-all-in-one:
    image: jaegertracing/all-in-one:latest
    restart: always
    ports:
      - "16686:16686"
      - "14268"
      - "14250"

  # Zipkin
  zipkin-all-in-one:
    image: openzipkin/zipkin:latest
    restart: always
    ports:
      - "9411:9411"

  # Collector
  otel-collector:
    image: otel/opentelemetry-collector:0.67.0
    restart: always
    command: ["--config=/etc/otel-collector-config.yaml", "${OTELCOL_ARGS}"]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - "1888:1888"   # pprof extension
      - "8888:8888"   # Prometheus metrics exposed by the collector
      - "8889:8889"   # Prometheus exporter metrics
      - "13133:13133" # health_check extension
      - "4317:4317"   # OTLP gRPC receiver
      - "4318:4318"   # OTLP http receiver
      - "55679:55679" # zpages extension
    depends_on:
      - jaeger-all-in-one
      - zipkin-all-in-one

  #demo-client:
  #  build:
  #    dockerfile: Dockerfile
  #    context: ./client
  #  restart: always
  #  environment:
  #    - OTEL_EXPORTER_OTLP_ENDPOINT=otel-collector:4317
  #    - DEMO_SERVER_ENDPOINT=http://demo-server:7080/hello
  #  depends_on:
  #    - demo-server

  #demo-server:
  #  build:
  #    dockerfile: Dockerfile
  #    context: ./server
  #  restart: always
  #  environment:
  #    - OTEL_EXPORTER_OTLP_ENDPOINT=otel-collector:4317
  #  ports:
  #    - "7080"
  #  depends_on:
  #    - otel-collector

  #prometheus:
  #  container_name: prometheus
  #  image: prom/prometheus:latest
  #  restart: always
  #  volumes:
  #    - ./prometheus.yaml:/etc/prometheus/prometheus.yml
  #  ports:
  #    - "9090:9090"
```

otel-collector-config.yaml

```
receivers:
  otlp:
    protocols:
      grpc:
      http:

exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
    const_labels:
      label1: value1

  logging:

  zipkin:
    endpoint: "http://zipkin-all-in-one:9411/api/v2/spans"
    format: proto

  jaeger:
    endpoint: jaeger-all-in-one:14250
    tls:
      insecure: true

processors:
  batch:

extensions:
  health_check:
  pprof:
    endpoint: :1888
  zpages:
    endpoint: :55679

service:
  extensions: [pprof, zpages, health_check]
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging, zipkin, jaeger]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging, prometheus]
```

### 3.8.4 opentelemetry_sdk_log4cxx.xml(log 配置文件）

同其他njet 配置文件路径一致

```
<?xml version="1.0" encoding="UTF-8" ?>
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/" debug="false">

<appender name="main" class="org.apache.log4j.ConsoleAppender">
 <layout class="org.apache.log4j.PatternLayout">
  <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS z} %-5p %X{pid}[%t] [%c{2}] %m%n" />
  <param name="HeaderPattern" value="Opentelemetry Webserver %X{version} %X{pid}%n" />
 </layout>
</appender>

<appender name="api" class="org.apache.log4j.ConsoleAppender">
 <layout class="org.apache.log4j.PatternLayout">
  <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS z} %-5p %X{pid} [%t] [%c{2}] %m%n" />
  <param name="HeaderPattern" value="Opentelemetry Webserver %X{version} %X{pid}%n" />
 </layout>
</appender>

<appender name="api_user" class="org.apache.log4j.ConsoleAppender">
 <layout class="org.apache.log4j.PatternLayout">
  <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS z} %-5p %X{pid} [%t] [%c{2}] %m%n" />
  <param name="HeaderPattern" value="Opentelemetry Webserver %X{version} %X{pid}%n" />
 </layout>
</appender>

<logger name="api" additivity="false">
    <level value="info"/>
    <appender-ref ref="api"/>
</logger>

<logger name="api_user" additivity="false">
    <level value="info"/>
    <appender-ref ref="api_user"/>
</logger>

<root>
  <priority value="info" />
  <appender-ref ref="main"/>
</root>

</log4j:configuration>
```

### 3.8.5**效果**

#### 3.8.4.1 **分布式服务追踪模块**: njt_otel_module.so

Jaeger ui：http://{jaeger server ip}:16686

通过jaeger ui 查看trace 

http://192.168.40.136:16686

![image-20230616173103901](https://gitee.com/gebona/picture/raw/master/202306161731257.png)



## 3.9 Upstream API

### 3.9.1 **简介**

开源nginx提供了一个流量配比的模块split_clients，但是该模块是根据指定的一个变量值（如客户端ip, url 中的参数等），然后计算Hash 值, 然后根据计算的值在 0 - 2^32-1 的位置来进行流量配比， 无法做到根据请求的数量做概率分布。 

Split Clients 2 模块提供流量配比，是根据概率进行分布， 当访问样本足够大的情况下，与设置的比例基本达到一致, 通过该模块的动态配置，可以实现蓝绿发布（配置比例为 0%， 及 100%）

### 3.9.2 **配置说明**

请参考标准配置文件章节配置，全量配置即可配置上此功能，但请注意一定要包含以下指令和块：

要开启该功能, 需在 ctrl.conf 配置文件的 main 块中加载该模块：

```
load_module modules/njt_http_upstream_api_module.so; 
```

并在 http 块下，指定该模块的配置文件：

```
Bash
http {
     access_log logs/access_ctrl.log combined;
     server {
         listen         8081  ;
         location /  api   {
            api   write=on;
         }
     }
}
```

![image-20230616173223505](https://gitee.com/gebona/picture/raw/master/202306161732256.png)

Upstream api，配置ACL控制

```
load_module modules/njt_http_upstream_api_module.so; 

 server {
        listen       8081;
        
        
         location /api {
             api write=on;
             limit_except GET {
               auth_basic "NGINX plus API";
               auth_basic_user_file /etc/njet/htpasswd;
            }
        }
  }
```



### 3.9.3 **参考说明**

参考swagger-ui：

使用时亦可参考swagger的使用说明，所有upstream_api的接口和使用方法皆有介绍。（下方swagger地址的server即为所部属njet机器的ip）

'server':8081/doc/swagger/

![image-20230616173253680](https://gitee.com/gebona/picture/raw/master/202306161732954.png)

![image-20230616173306645](https://gitee.com/gebona/picture/raw/master/202306161733130.png)

### 3.9.4 **API **实例

#### 3.9.4.1 **http**模块

http模块可以使用upstream_api功能进行增删改查的操作，以下将介绍具体使用方法和使用效果展示。

##### 3.9.4.1.1 **添加**

添加功能支持IP和域名的添加：

IP添加

```
Bash
curl -X POST -s http://127.0.0.1:8081/api/7/http/upstreams/backend1/servers/ -d '{"server":"192.168.40.101:8080","weight":2,"max_conns":2,"max_fails":"1","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "id": 0,
  "server": "192.168.40.101:8080",
  "weight": 2,
  "max_conns": 2,
  "max_fails": 1,
  "fail_timeout": "5s",
  "slow_start": "5s",
  "route": "",
  "backup": true,
  "down": false
}
```

域名添加

```
Bash
 curl -X POST -s http://127.0.0.1:8081/api/7/http/upstreams/beckend2/servers/ -d '{"server":"www.dev.test.com:8080","weight":1,"max_conns":1,"max_fails":"1","fail_timeout":"1s","slow_start":"1s","backup":false,"down":true}'|jq .
{
  "id": 0,
  "server": "www.dev.test.com:8080",
  "weight": 1,
  "max_conns": 1,
  "max_fails": 1,
  "fail_timeout": "1s",
  "slow_start": "1s",
  "route": "",
  "backup": false,
  "down": true
}
```

##### 3.9.4.1.2 **查询**

查询功能支持多层级查询，首先是upstreams级，能够查找出所有的upstream信息，且无法查询出域名server的父级节点

```
Bash
curl -s http://127.0.0.1:8081/api/7/http/upstreams/ |jq .                                                               {
  "backend1": {
    "peers": [
      {
        "id": 0,
        "server": "192.168.40.101:8080",
        "name": "192.168.40.101:8080",
        "backup": true,
        "weight": 2,
        "state": "up",
        "active": 0,
        "max_conns": 2,
        "requests": 0,
        "responses": {
          "1xx": 0,
          "2xx": 0,
          "3xx": 0,
          "4xx": 0,
          "5xx": 0,
          "codes": {},
          "total": 0
        },
        "sent": 0,
        "received": 0,
        "fails": 0,
        "unavail": 0,
        "health_checks": {
          "checks": 0,
          "fails": 0,
          "unhealthy": 0
        },
        "downtime": 0
      }
    ],
    "keepalive": 0,
    "zombies": 0,
    "zone": "cluster1"
  },
  "backend2": {
    "peers": [
      {
        "id": 1,
        "server": "192.168.40.101:8080",
        "name": "www.dev.test.com:8080",
        "backup": false,
        "weight": 1,
        "state": "down",
        "active": 0,
        "max_conns": 1,
        "requests": 0,
        "responses": {
          "1xx": 0,
          "2xx": 0,
          "3xx": 0,
          "4xx": 0,
          "5xx": 0,
          "codes": {},
          "total": 0
        },
        "sent": 0,
        "received": 0,
        "fails": 0,
        "unavail": 0,
        "health_checks": {
          "checks": 0,
          "fails": 0,
          "unhealthy": 0
        },
        "downtime": 0
      }
    ],
    "keepalive": 0,
    "zombies": 0,
    "zone": "cluster2"
  }
}
```

upsteam级，可以指定查看单个upstream的信息

```
Bash
curl -s http://127.0.0.1:8081/api/7/http/upstreams/backend1 |jq .
{
  "peers": [
    {
      "id": 0,
      "server": "192.168.40.101:8080",
      "name": "192.168.40.101:8080",
      "backup": true,
      "weight": 2,
      "state": "up",
      "active": 0,
      "max_conns": 2,
      "requests": 0,
      "responses": {
        "1xx": 0,
        "2xx": 0,
        "3xx": 0,
        "4xx": 0,
        "5xx": 0,
        "codes": {},
        "total": 0
      },
      "sent": 0,
      "received": 0,
      "fails": 0,
      "unavail": 0,
      "health_checks": {
        "checks": 0,
        "fails": 0,
        "unhealthy": 0
      },
      "downtime": 0
    }
  ],
  "keepalive": 0,
  "zombies": 0,
  "zone": "cluster1"
}
```

servers级，可以查看单个upstream的所有server级节点

```
Bash
curl -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers |jq .
[
  {
    "id": 1,
    "server": "192.168.40.101:8080",
    "weight": 1,
    "max_conns": 1,
    "max_fails": 1,
    "fail_timeout": "1s",
    "slow_start": "1s",
    "route": "",
    "backup": false,
    "down": true,
    "parent": 0,
    "host": "www.dev.test.com:8080"
  },
  {
    "id": 0,
    "server": "www.dev.test.com:8080",
    "weight": 1,
    "max_conns": 1,
    "max_fails": 1,
    "fail_timeout": "1s",
    "slow_start": "1s",
    "route": "",
    "backup": false,
    "down": true
  }
]
```

id级，可以查看单个upstream的单个id级节点

```
Bash
curl -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers/1 |jq .
{
  "id": 1,
  "server": "192.168.40.101:8080",
  "weight": 1,
  "max_conns": 1,
  "max_fails": 1,
  "fail_timeout": "1s",
  "slow_start": "1s",
  "route": "",
  "backup": false,
  "down": true,
  "parent": 0,
  "host": "www.dev.test.com:8080"
}
```

##### 3.9.4.1.3 **修改**

修改功能可以对server节点的参数进行修改，其中IP格式server节点可以修改server参数，域名server及其子节点server无法修改，且backup参数无法修改

修改IP格式server

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/http/upstreams/backend1/servers/0 -d '{"server":"192.168.40.114:8080","weight":1,"max_conns":1,"max_fails":"2","fail_timeout":"1s","slow_start":"1s","backup":false,"down":true}'|jq .
{
  "id": 0,
  "server": "192.168.40.114:8080",
  "weight": 1,
  "max_conns": 1,
  "max_fails": 2,
  "fail_timeout": "1s",
  "slow_start": "1s",
  "route": "",
  "backup": true,
  "down": true
}
```

修改域名格式server，父节点server参数修改

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers/0 -d '{"server":"www.dev.test2.com:8080","weight":2,"max_conns":2,"max_fails":"2","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "error": {
    "status": 400,
    "text": "server address is immutable",
    "code": "UpstramServerImmutable"
  },
  "request_id": "730e18137d711d23bb93b1c867c21b89",
  "href": "https://njet.org/en/docs/http/njt_http_api_module.html"
}
```

修改域名格式server，子节点server参数修改

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers/1 -d '{"server":"192.168.40.114:8080","weight":2,"max_conns":2,"max_fails":"2","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "error": {
    "status": 400,
    "text": "server address is immutable",
    "code": "UpstramServerImmutable"
  },
  "request_id": "bcda20dbc9276a1e46229e620e5c5813",
  "href": "https://njet.org/en/docs/http/njt_http_api_module.html"
}
```

修改域名格式server，只修改父节点基本参数

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers/0 -d '{"weight":2,"max_conns":2,"max_fails":"2","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "id": 0,
  "server": "www.dev.test.com:8080",
  "weight": 2,
  "max_conns": 2,
  "max_fails": 2,
  "fail_timeout": "5s",
  "slow_start": "5s",
  "route": "",
  "backup": false,
  "down": false
}
```

修改域名格式server，只修改子节点基本参数

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers/1 -d '{"weight":1,"max_conns":3,"max_fails":"3","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "id": 1,
  "server": "192.168.40.101:8080",
  "weight": 1,
  "max_conns": 3,
  "max_fails": 3,
  "fail_timeout": "5s",
  "slow_start": "5s",
  "route": "",
  "backup": false,
  "down": false,
  "parent": 0,
  "host": "www.dev.test.com:8080"
}
```

##### 3.9.4.1.4 **删除**

curl -X DELETE http://127.0.0.1:8081/api/7/http/upstreams/http1/servers/0 |jq .  

删除功能可以针对单个server进行删除，且删除域名格式server父节点时会兼带子节点全部删除，但是删除域名格式server子节点时将无法删除

删除IP格式server

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/http/upstreams/backend1/servers/0  |jq .
[]
```

删除域名格式server子节点

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers/1  |jq .
{
  "error": {
    "status": 400,
    "text": "server not removeable",
    "code": "UpstreamServerImmutable"
  },
  "request_id": "5399a28db3d4598b821c1c65cb7efa86",
  "href": "https://njet.org/en/docs/http/njt_http_api_module.html"
}
```

删除域名格式server父节点

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/http/upstreams/backend2/servers/0  |jq .
[]
```

##### 3.9.4.1.5 **重置**

针对upstream，清空健康检查等数据

使用查询命令查看upstream的数据，可以观察到requests参数有一些数据信息

```
Bash
curl -s http://127.0.0.1:8081/api/7/http/upstreams/backend1 |jq .
{
  "peers": [
    {
      "id": 0,
      "server": "192.168.40.101:8080",
      "name": "192.168.40.101:8080",
      "backup": true,
      "weight": 2,
      "state": "up",
      "active": 0,
      "max_conns": 2,
      "requests": 2,
      "responses": {
        "1xx": 0,
        "2xx": 0,
        "3xx": 0,
        "4xx": 0,
        "5xx": 0,
        "codes": {},
        "total": 0
      },
      "sent": 0,
      "received": 0,
      "fails": 0,
      "unavail": 0,
      "health_checks": {
        "checks": 0,
        "fails": 0,
        "unhealthy": 0
      },
      "downtime": 0
    }
  ],
  "keepalive": 0,
  "zombies": 0,
  "zone": "cluster1"
}
```

使用清除数据命令进行数据清空

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/http/upstreams/backend1  |jq .
```

再次查询upstream的数据，可以看到已经被清空

```
Bash
curl -s http://127.0.0.1:8081/api/7/http/upstreams/backend1 |jq .
{
  "peers": [
    {
      "id": 0,
      "server": "192.168.40.101:8080",
      "name": "192.168.40.101:8080",
      "backup": true,
      "weight": 2,
      "state": "up",
      "active": 0,
      "max_conns": 2,
      "requests": 0,
      "responses": {
        "1xx": 0,
        "2xx": 0,
        "3xx": 0,
        "4xx": 0,
        "5xx": 0,
        "codes": {},
        "total": 0
      },
      "sent": 0,
      "received": 0,
      "fails": 0,
      "unavail": 0,
      "health_checks": {
        "checks": 0,
        "fails": 0,
        "unhealthy": 0
      },
      "downtime": 0
    }
  ],
  "keepalive": 0,
  "zombies": 0,
  "zone": "cluster1"
}
```

#### 3.9.4.2 **stream**模块

stream模块可以使用upstream_api功能进行增删改查的操作，以下将介绍具体使用方法和使用效果展示。

##### 3.9.4.2.1 **添加**

添加功能支持IP和域名的添加：

IP添加

```
Bash
curl -X POST -s http://127.0.0.1:8081/api/7/stream/upstreams/backend3/servers/ -d '{"server":"192.168.40.101:8080","weight":2,"max_conns":2,"max_fails":"1","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "id": 0,
  "server": "192.168.40.101:8080",
  "weight": 2,
  "max_conns": 2,
  "max_fails": 1,
  "fail_timeout": "5s",
  "slow_start": "5s",
  "route": "",
  "backup": true,
  "down": false
}
```

域名添加

```
Bash
 curl -X POST -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/ -d '{"server":"www.dev.test.com:8080","weight":1,"max_conns":1,"max_fails":"1","fail_timeout":"1s","slow_start":"1s","backup":false,"down":true}'|jq .
{
  "id": 0,
  "server": "www.dev.test.com:8080",
  "weight": 1,
  "max_conns": 1,
  "max_fails": 1,
  "fail_timeout": "1s",
  "slow_start": "1s",
  "route": "",
  "backup": false,
  "down": true
}
```

##### 3.9.4.2.2 **查询**

查询功能支持多层级查询，首先是upstreams级，能够查找出所有的upstream信息，且无法查询出域名server的父级节点

```
Bash
curl -s http://127.0.0.1:8081/api/7/stream/upstreams/ |jq .                                                               {
  "backend3": {
    "peers": [
      {
        "id": 0,
        "server": "192.168.40.101:8080",
        "name": "192.168.40.101:8080",
        "backup": true,
        "weight": 2,
        "state": "up",
        "active": 0,
        "max_conns": 2,
        "connecions": 0,
        "sent": 0,
        "received": 0,
        "fails": 0,
        "unavail": 0,
        "health_checks": {
          "checks": 0,
          "fails": 0,
          "unhealthy": 0
        },
        "downtime": 0
      }
    ],
    "zombies": 0,
    "zone": "cluster3"
  },
  "backend4": {
    "peers": [
      {
        "id": 1,
        "server": "192.168.40.101:8080",
        "name": "www.dev.test.com:8080",
        "backup": false,
        "weight": 1,
        "state": "down",
        "active": 0,
        "max_conns": 1,
        "connecions": 0,
        "sent": 0,
        "received": 0,
        "fails": 0,
        "unavail": 0,
        "health_checks": {
          "checks": 0,
          "fails": 0,
          "unhealthy": 0
        },
        "downtime": 0
      }
    ],
    "zombies": 0,
    "zone": "cluster4"
  }
}
```

upsteam级，可以指定查看单个upstream的信息

```
Bash
curl -s http://127.0.0.1:8081/api/7/stream/upstreams/backend3 |jq .
{
  "peers": [
    {
      "id": 0,
      "server": "192.168.40.101:8080",
      "name": "192.168.40.101:8080",
      "backup": true,
      "weight": 2,
      "state": "up",
      "active": 0,
      "max_conns": 2,
      "connecions": 0,
      "sent": 0,
      "received": 0,
      "fails": 0,
      "unavail": 0,
      "health_checks": {
        "checks": 0,
        "fails": 0,
        "unhealthy": 0
      },
      "downtime": 0
    }
  ],
  "zombies": 0,
  "zone": "cluster3"
}
```

servers级，可以查看单个upstream的所有server级节点

```
Bash
curl -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers |jq .
[
  {
    "id": 1,
    "server": "192.168.40.101:8080",
    "weight": 1,
    "max_conns": 1,
    "max_fails": 1,
    "fail_timeout": "1s",
    "slow_start": "1s",
    "backup": false,
    "down": true,
    "parent": 0,
    "host": "www.dev.test.com:8080"
  },
  {
    "id": 0,
    "server": "www.dev.test.com:8080",
    "weight": 1,
    "max_conns": 1,
    "max_fails": 1,
    "fail_timeout": "1s",
    "slow_start": "1s",
    "backup": false,
    "down": true
  }
]
```

id级，可以查看单个upstream的单个id级节点

```

Bash
curl -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/1 |jq .
{
  "id": 1,
  "server": "192.168.40.101:8080",
  "weight": 1,
  "max_conns": 1,
  "max_fails": 1,
  "fail_timeout": "1s",
  "slow_start": "1s",
  "backup": false,
  "down": true,
  "parent": 0,
  "host": "www.dev.test.com:8080"
}
```

##### 3.9.4.2.3 修改

修改功能可以对server节点的参数进行修改，其中IP格式server节点可以修改server参数，域名server及其子节点server无法修改，且backup参数无法修改
修改IP格式server

修改IP格式server

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/stream/upstreams/backend3/servers/0 -d '{"server":"192.168.40.114:8080","weight":1,"max_conns":1,"max_fails":"2","fail_timeout":"1s","slow_start":"1s","backup":false,"down":true}'|jq .
{
  "id": 0,
  "server": "192.168.40.114:8080",
  "weight": 1,
  "max_conns": 1,
  "max_fails": 2,
  "fail_timeout": "1s",
  "slow_start": "1s",
  "backup": true,
  "down": true
}
```

修改域名格式server，父节点server参数修改

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/0 -d '{"server":"www.dev.test2.com:8080","weight":2,"max_conns":2,"max_fails":"2","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "error": {
    "status": 400,
    "text": "server address is immutable",
    "code": "UpstramServerImmutable"
  },
  "request_id": "730e18137d711d23bb93b1c867c21b89",
  "href": "https://njet.org/en/docs/http/njt_http_api_module.html"
}
```

修改域名格式server，子节点server参数修改

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/1 -d '{"server":"192.168.40.114:8080","weight":2,"max_conns":2,"max_fails":"2","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "error": {
    "status": 400,
    "text": "server address is immutable",
    "code": "UpstramServerImmutable"
  },
  "request_id": "bcda20dbc9276a1e46229e620e5c5813",
  "href": "https://njet.org/en/docs/http/njt_http_api_module.html"
}
```

修改域名格式server，只修改父节点基本参数

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/0 -d '{"weight":2,"max_conns":2,"max_fails":"2","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "id": 0,
  "server": "www.dev.test.com:8080",
  "weight": 2,
  "max_conns": 2,
  "max_fails": 2,
  "fail_timeout": "5s",
  "slow_start": "5s",
  "backup": false,
  "down": false
}
```

修改域名格式server，只修改子节点基本参数

```
Bash
curl -X PATCH -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/1 -d '{"weight":1,"max_conns":3,"max_fails":"3","fail_timeout":"5s","slow_start":"5s","backup":true,"down":false}'|jq .
{
  "id": 1,
  "server": "192.168.40.101:8080",
  "weight": 1,
  "max_conns": 3,
  "max_fails": 3,
  "fail_timeout": "5s",
  "slow_start": "5s",
  "backup": false,
  "down": false,
  "parent": 0,
  "host": "www.dev.test.com:8080"
}
```

##### 3.9.4.2.4 **删除**

删除功能可以针对单个server进行删除，且删除域名格式server父节点时会兼带子节点全部删除，但是删除域名格式server子节点时将无法删除

删除IP格式server

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/stream/upstreams/backend3/servers/0  |jq .
[]
```

删除域名格式server子节点

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/1  |jq .
{
  "error": {
    "status": 400,
    "text": "server not removeable",
    "code": "UpstreamServerImmutable"
  },
  "request_id": "5399a28db3d4598b821c1c65cb7efa86",
  "href": "https://njet.org/en/docs/http/njt_http_api_module.html"
}
```

删除域名格式server父节点

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/stream/upstreams/backend4/servers/0  |jq .
[]
```

##### 3.9.4.2.5 **重置**

针对upstream，清空健康检查等数据

使用查询命令查看upstream的数据，可以观察到requests参数有一些数据信息

```
Bash
curl -s http://127.0.0.1:8081/api/7/stream/upstreams/backend3 |jq .
{
  "peers": [
    {
      "id": 0,
      "server": "192.168.40.101:8080",
      "name": "192.168.40.101:8080",
      "backup": true,
      "weight": 2,
      "state": "up",
      "active": 0,
      "max_conns": 2,
      "connecions": 4,
      "sent": 0,
      "received": 0,
      "fails": 0,
      "unavail": 0,
      "health_checks": {
        "checks": 0,
        "fails": 0,
        "unhealthy": 0
      },
      "downtime": 0
    }
  ],
  "keepalive": 0,
  "zombies": 0,
  "zone": "cluster3"
}
```

使用清除数据命令进行数据清空

```
Bash
curl -X DELETE -s http://127.0.0.1:8081/api/7/stream/upstreams/backend3 |jq .
```

再次查询upstream的数据，可以看到已经被清空

```
Bash
curl -s http://127.0.0.1:8081/api/7/stream/upstreams/backend3 |jq .
{
  "peers": [
    {
      "id": 0,
      "server": "192.168.40.101:8080",
      "name": "192.168.40.101:8080",
      "backup": true,
      "weight": 2,
      "state": "up",
      "active": 0,
      "max_conns": 2,
      "connecions": 0,
      "sent": 0,    
      "received": 0,
      "fails": 0,
      "unavail": 0,
      "health_checks": {
        "checks": 0,
        "fails": 0,
        "unhealthy": 0
      },
      "downtime": 0
    }
  ],
  "keepalive": 0,
  "zombies": 0,
  "zone": "cluster3"
}
```

#### 3.9.4.3 **持久化**

upstream_api功能可以使用持久化配置来保证增加、删除、修改后数据的持久性，例如重启服务后，之前的配置依然可以使用，且当某个upstream配置了state持久化后，upstream就无法再静态添加server指令想要使用持久化功能，需要添加以下配置。

```
Bash
upstream http1{
           zone cluster 64k;
           state /home/njet/state/njet_state.file; #state持久化保存文件可以自定义名称
}
```



## 3.10 **Split Clients_2**

### 3.10.1 **功能说明**

开源nginx提供了一个流量配比的模块split_clients，但是该模块是根据指定的一个变量值（如客户端ip, url 中的参数等），然后计算Hash 值, 然后根据计算的值在 0 - 2^32-1 的位置来进行流量配比， 无法做到根据请求的数量做概率分布。 

Split Clients 2 模块提供流量配比，是根据概率进行分布， 当访问样本足够大的情况下，与设置的比例基本达到一致, 通过该模块的动态配置，可以实现蓝绿发布（配置比例为 0%， 及 100%）

### 3.10.2 **流量配置实现方式**

Split Clients 2 模块提供了两种方式来进行流量配比：

￮    静态配置方式：通过静态配置文件实现不同流量配比，分配的百分比，已经配置好，可以在配置文件中进行修改，但需要重新reload才能生效

￮    动态配置方式：通过声明式API实现动态修改流量配比

 **静态配置**

配置指令：

```
语法:        split_clients_2 $variable { ... }；
默认值:        
允许配置位置:    http
```

配置样例：  

```
Nginx
split_clients_2 $backend {
  10%       backend1;
  *         backend2;
}
```

配置补充说明：

￮    静态配置百分比修改后，需要reload才能生效。

￮    百分比一行结束后，如果缺少分号，会导致下一行配比不生效。注意不能缺少分号。

￮    配比分流upstream的数量，限制只能配置两个。

**动态配置**

参考声明式API配置splitclient章节

 **验证分流是否生效**

可以通过如下脚本实现模拟大样本的流量配比验证。如配比为50%， 请求100次，会在40多到50多区间进行配比，如下。

![image-20230616175735101](https://gitee.com/gebona/picture/raw/master/202306161757179.png)

### 3.10.3 **接口消息返回提示**

#### 3.10.3.1 **查询**

使用GET方法获取当前split_clients_2的静态配置。

```
Bash
curl -X 'GET' 'http://127.0.0.1:8081/config/2/config/http_split_clients_2'
```

```
Bash
示例返回：
{
  "http": {
    "split_clients_2": {
      "backend1": 10,
      "backend2": 90
    }
}
```

#### 3.10.3.2 **配置修改**

修改PUT方法的配置，对应返回消息提示。

```
Bash
curl -X 'PUT'  'http://127.0.0.1:8081/config/2/config/http_split_clients_2'  -d '{
  "http": {
    "split_clients_2": {
      "backend1": 20,
      "backend2": 80
    }
  }
}'
```

#### 3.10.3.3 **结果**

```
Bash
{
  "code": 0,
  "msg": "success."
}
```



## 3.11 **动态**Access Log

### 3.11.1 **功能说明**

通过动态access_log，无需对配置文件进行编辑及重载，就可以实现对access log开关状态的动态控制功能，现提供了动态查询接口和动态修改接口。

动态access_log开关accessLogOn缺省值为true。该动态开关配置在http，server，location的上下文配置中修改没有继承关系。

静态access log开关指令为access_log off，再把动态acesss log开关同时配置情况下，以动态acceess log开关配置为优先。

### 3.11.2 **动态查询接口** 

可以查看HTTP 下的所有server及location，以及各个location 块内access log 当前的开关状态；

 发起请求：

```
PUT http://127.0.0.1:8081/config/2/config/http_log
Content-Type: application/json

{
  "servers": [
    {
      "listens": [
        "0.0.0.0:90"
      ],
      "serverNames": [
        "localhost"
      ],
      "locations": [
        {
          "location": "/",
          "accessLogOn": true,
          "accessLogs": [
            {
              "path": "logs/access.log",
              "formatName": "combined"
            }
          ]
        },
        {
          "location": "/test_accesslog",
          "accessLogOn": true,
          "accessLogs": [
            {
              "path": "logs/access.log",
              "formatName": "combined"
            }
          ]
        }
      ]
    }
  ],
  "accessLogFormats": [
    {
      "name": "combined",
      "escape": "default",
      "format": "$remote_addr - $remote_user [$time_local] \"$request\" $status $body_bytes_sent \"$http_referer\" \"$http_user_agent\""
    }
  ]
}

```

 结果：

```
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Mon, 22 May 2023 09:49:20 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
{
  "code": 0,
  "msg": "success."
}
```

#### 3.11.2.1  **验证方式**

##### 3.11.2.1.1 **配置**

```
Nginx
helper broker modules/njt_helper_broker_module.so conf/mqtt.conf; 
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;

load_module modules/njt_http_split_clients_2_module.so;  
load_module modules/njt_agent_dynlog_module.so;  
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 

load_module modules/njt_otel_module.so;
load_module modules/njt_agent_dyn_otel_module.so;
load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

#user  root;
worker_processes  2;

cluster_name helper;
node_name node1;

error_log  logs/error.log info;
pid        logs/njet.pid;

events {
    worker_connections  1024;
}

http {
    
    dyn_kv_conf conf/iot-work.conf;
    include       mime.types;
    default_type  application/octet-stream;
    
    access_log  logs/access.log;
    
    sendfile        on;

    keepalive_timeout  65;

    upstream backend1 { 
         
         zone backend1 128k;
         
         server 192.168.40.150:5678;

         
    }
    
    upstream backend2 {
    
         zone backend2 128k;
         
         server 192.168.40.144:5555;

    }

    
    server {
    
        listen 90;
        server_name localhost;
        
         location / {
         
             proxy_pass http://backend2;

             
             
             
         }

        location /test_accesslog {

            proxy_pass http://backend1;

        }
    }
}
```

##### 3.11.2.1.2 **查询接口调用**

![image-20230616180059101](https://gitee.com/gebona/picture/raw/master/202306161801224.png)

 调用查询接口，可见所有监听的server，及各个server下的location，并能看到上方server子location 访问日志是的开启与关闭状态。

### 3.11.3 **动态修改接口**

可以对查询结果中的access log的开关状态进行动态修改。

#### 3.11.3.1  **请求**

```
Bash
 PUT http://127.0.0.1:8081/config/2/config/http_log
Content-Type: application/json

{
  "servers": [
    {
      "listens": [
        "0.0.0.0:90"
      ],
      "serverNames": [
        "localhost"
      ],
      "locations": [
        {
          "location": "/",
          "accessLogOn": true,
          "accessLogs": [
            {
              "path": "logs/access.log",
              "formatName": "combined"
            }
          ]
        },
        {
          "location": "/test_accesslog",
          "accessLogOn": true,
          "accessLogs": [
            {
              "path": "logs/access.log",
              "formatName": "combined"
            }
          ]
        }
      ]
    }
  ],
  "accessLogFormats": [
    {
      "name": "combined",
      "escape": "default",
      "format": "$remote_addr - $remote_user [$time_local] \"$request\" $status $body_bytes_sent \"$http_referer\" \"$http_user_agent\""
    }
  ]
}
```

#### 3.11.3.2  **结果**

```
Bash
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Mon, 22 May 2023 09:49:20 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
{
  "code": 0,
  "msg": "success."
}
```

#### 3.11.3.3  **验证方式**

##### 3.11.3.3.1 **修改前查询**

![image-20230616180222130](https://gitee.com/gebona/picture/raw/master/202306161802687.png)

##### 3.11.3.3.2 **修改配置**

编辑需要改动的字段，将location /test_accesslog 路径下的accessLogOn值从false改为true，并填写accessLogs字段里path和formatName的值。

![image-20230616180255803](https://gitee.com/gebona/picture/raw/master/202306161803214.png)

##### 3.11.3.3.3 **修改接口调用**

![image-20230616180327790](https://gitee.com/gebona/picture/raw/master/202306161803220.png)

##### 3.11.3.3.4 **修改后再查询验证修改是否成功**

![image-20230616180353189](https://gitee.com/gebona/picture/raw/master/202306161803820.png)



## 3.12 **缓存功能**

### 3.12.1 **简介**

OpenNJet会缓存来自代理 Web 或应用程序服务器的静态和动态内容，以加快向客户端的响应速度并减少上游服务器上的负载。启用缓存后，OpenNJet将响应保存在OpenNJet磁盘缓存中，并使用它们来响应客户端，而不必每次都对相同内容向上游服务器发起请请求。

### 3.12.2 **启用缓存**

若要启用缓存，请在Http上下文中包含 proxy_cache_path指令。第一个参数是缓存内容的本地文件系统路径，第二个参数定义用于存储有关缓存项的元数据的共享内存区域的名称和大小。

```
http {
    # ...
    proxy_cache_path /data/nginx/cache keys_zone=mycache:10m;
}
```

然后在要缓存server响应的上下文（http、server或location）中包含proxy_cache指令，并指定由上述指令参数定义的共享内存区域名称。

```
http {
    # ...
    proxy_cache_path /data/nginx/cache keys_zone=mycache:10m;
    server {
        proxy_cache mycache;
        location / {
            proxy_pass http://localhost:8000;
        }
    }
}
```

请注意，参数定义的大小不会限制缓存响应数据的总量。缓存的响应本身与元数据的副本一起存储在文件系统上的特定文件中。要限制缓存的响应数据量，请将参数max_size 配置在 proxy_cache_path指令中。（但请注意，缓存的数据量可能会暂时超过此限制)。

### 3.12.3 **指定要缓存的请求**

默认情况下，OpenNJet会在第一次从代理服务器收到此类响应时缓存对使用HTTP GET和HEAD方法发出的请求的所有响应。OpenNJet使用请求字符串作为请求的标识符。如果请求与缓存的响应具有相同的键，OpenNJet会将缓存的响应发送到客户端。您可以在 http{} 、server{} 或location{}中包含各种指令，以控制缓存哪些响应。

要更改用于计算缓存文件标识，请使用proxy_cache_key指令。

```
proxy_cache_key "$host$request_uri$cookie_user";
```

要定义在缓存响应之前必须发出具有相同键的请求的最小次数，请使用 proxy_cache_min_uses指令：

```
proxy_cache_min_uses 5;
```

要使用 GET和 HEAD以外的方法缓存对请求的响应，请使用 proxy_cache_methods指令并配置要缓存的方法。

```
proxy_cache_methods GET HEAD POST;
```

### 3.12.4 **限制或禁用缓存**

默认情况下，响应无限期地保留在缓存中。仅当缓存超过最大配置大小时，才会删除它们，然后按自上次请求它们以来的时间排序最长最少使用的缓存进行删除。您可以通过在http{} 、server{} 或location{}上下文中包含指令来设置缓存响应被视为有效的时间和是否使用缓存：

要限制具有特定状态代码的缓存响应被视为有效的时间，请使用 proxy_cache_valid 指令：

```
proxy_cache_valid 200 302 10m;
proxy_cache_valid 404      1m;
```

在此示例中，包含代码200或302的响应被视为有效 10 分钟，包含代码的响应的有效期为 1 分钟。要定义具有所有状态代码的响应的有效期，请指定为第一个参数为any：

```
proxy_cache_valid any 5m;
```

要定义OpenNJet不向客户端发送缓存响应的条件，请使用proxy_cache_bpass指令。每个参数定义一个条件，并由许多变量组成。如果至少有一个参数不为空且不等于"0"，OpenNJet不会在缓存中查找响应，而是立即将请求转发到后端服务器。

注意：

关于缓存清理还有一个很重要的参数需要配置

proxy_cache_path 后面的inactive参数，该参数表示资源不活跃时间，在该时间内如果资源没有被访问，也会被清理掉，尽管proxy_cache_valid 后面设置的有效期还没到，该资源也会被删除掉

补充：（后面可以跟很多字符串（可以为变量），只要有一个字符串不为空并且不等于"0", 则都会走pass， 所以bypass数字都会加1.）

```
proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
```

要定义OpenNJet根本不缓存响应的条件，请使用proxy_no_cache指令，并以与proxy_cache_bypass指令相同的方式定义参数。（ 如果字符串参数中至少有一个值不为空且不等于“0”，则不会保存响应）

```
proxy_no_cache $http_pragma $http_authorization;
```

### 3.12.5 **从缓存中清除内容**

OpenNJet可以从缓存中删除过时的缓存文件。这对于删除过时的缓存内容以防止同时提供旧版和新版本的网页是必需的。缓存在收到包含自定义 HTTP 标头或 HTTP 方法的特殊“PURGE”请求时被清除。

#### 3.12.5.1 **配置缓存清除功能**

让我们设置一个配置来标识使用 HTTP PURGE 方法的请求并删除匹配的 URL。

a.   在上下文中，使用map创建一个新变量，例如：

```
Nginx
http {# ...
    map $request_method $purge_method {
        PURGE 1;
        default 0;
    }
}
```

b.   在配置缓存的location{}块中，配置proxy_cache_purge 指令以指定缓存清除请求的条件。在我们的示例中，它是在上一步中配置的：

```
Nginx
server {
    listen      80;
    server_name www.example.com;
    location / {
        proxy_pass  https://localhost:8002;
        proxy_cache mycache;
        proxy_cache_purge $purge_method;
    }
}
```

#### 3.12.5.2 **发送**PURGE请求

配置proxy_cache_purge指令后，需要发送特殊的缓存清除请求来清除缓存。您可以使用一系列工具发出清除请求，包括以下示例中的curl命令：

```
Bash
$ curl -X PURGE -D – "https://www.example.com/*"
HTTP/1.1 204 No Content
Server: njet/1.23.2
Date: Sat, 19 May 2018 16:33:04 GMT
Connection: keep-alive
```

在此示例中，将清除具有公共 URL 部分（由星号通配符指定）的资源。但是，此类缓存条目不会立即从缓存中完全删除：它们将保留在磁盘上，直到它们因不活动（由 proxy_cache_path 指令的参数确定）或缓存清除器（使用 purge 参数启用）而被删除，或者客户端尝试访问它们。

#### 3.12.5.3 **限制对清除命令的访问**

我们建议您限制允许发送缓存清除请求的 IP 地址：

```
Nginx
geo $purge_allowed {
    default         0;  # deny from other
   10.0.0.1        1;  # allow from 10.0.0.1 address
   192.168.0.0/24  1;  # allow from 192.168.0.0/24
}
map $request_method $purge_method {
    PURGE   $purge_allowed;
    default 0;
}
```

在此示例中，OpenNJet 检查请求中是否使用了PURGE方法，如果是则分析客户端 IP 地址。如果 IP 地址已列入白名单，则 设置为 ： 允许清除，否则拒绝清除。

#### 3.12.5.4 **从缓存中完全删除文件**

要完全删除与星号匹配的缓存文件，请激活purge功能，该功能将循环访问所有缓存条目并删除与通配符键匹配的条目。在配置中包含 purge 参数到proxy_cache_path指令：

```
Nginx
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m purger=on;
```

#### 3.12.5.5 **缓存清除配置示例**

```
Nginx
http {# ...
    proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m purger=on;
    map $request_method $purge_method {
        PURGE 1;
        default 0;
    }
    server {
        listen      80;
        server_name www.example.com;
        location / {
            proxy_pass        https://localhost:8002;
            proxy_cache       mycache;
            proxy_cache_purge $purge_method;
        }
    }
    geo $purge_allowed {
        default         0;
        10.0.0.1        1;
        192.168.0.0/24  1;
    }
    map $request_method $purge_method {
        PURGE   $purge_allowed;
        default 0;
    }
}
```

### 3.12.6 **文件分片缓存**

初始缓存填充操作有时需要相当长的时间，尤其是对于大文件。例如，当视频文件开始下载以满足对文件一部分的初始请求时，后续请求必须等待整个文件下载并放入缓存中。

OpenNJet可以缓存此类分片请求，并使用缓存切片模块逐渐填充缓存，该模块将文件划分为较小的“分片”。每个Range请求都会选择覆盖所请求范围的特定切片，如果此Range仍未缓存，则将其放入缓存中。对这些切片的所有其他请求都会从缓存中获取数据。

#### 3.12.6.1 **开启分片缓存**

a.   使用slice指令指定切片的大小：

```
Nginx
location / {
    slice  1m;
}
```

选择可加快切片下载速度的切片大小。如果大小太小，则内存使用量可能过多，并且在处理请求时打开了大量文件描述符，而过大可能会导致下载生成缓存缓慢。

b.   将 $slice_range变量包含在缓存键中：

```
Nginx
proxy_cache_key $uri$is_args$args$slice_range;
```

c.   状态代码206启用响应缓存：

```
Nginx
proxy_cache_valid 200 206 1h;
```

d.   通过在标头字段中设置 $slice_range变量，允许将range请求传递到代理服务器

```
Nginx
proxy_set_header  Range $slice_range;
```

全部配置如下：

```
Nginx
location / {
    slice             1m;
    proxy_cache       cache;
    proxy_cache_key   $uri$is_args$args$slice_range;
    proxy_set_header  Range $slice_range;
    proxy_cache_valid 200 206 1h;
    proxy_pass        http://localhost:8000;
}
```

### 3.12.7 **组合配置示例**

以下示例配置结合了上述一些缓存选项。

```
Nginx
http {
    # ...
    proxy_cache_path /data/nginx/cache keys_zone=mycache:10m loader_threshold=300loader_files=200 max_size=200m;
    server {
        listen 8080;
        proxy_cache mycache;
        location / {
            proxy_pass http://backend1;
        }
        location /some/path {
            proxy_pass http://backend2;
            proxy_cache_valid any 1m;
            proxy_cache_min_uses 3;
            proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
        }
    }
}
```

在此示例中，两个location使用相同的缓存，但方式不同。

由于backend1响应很少更改，因此不包括缓存控制指令。响应在首次发出请求时缓存，并无限期有效。

相比之下，对backend2请求的响应经常更改，因此它们被认为仅在 1 分钟内有效，并且在发出 3 次相同的请求之前不会缓存。此外，如果请求符合proxy_cache_bypass指令定义的条件，OpenNJet会立即将请求传递给backend2，而无需在缓存中查找相应的响应。



## 3.13 **key value**功能

### **3.13.1** **简介**

控制面的Sendmsg模块提供了可选的配置指令dyn_sendmsg_kv， 这个指令将对外提供KV 值设置及查询的 HTTP API 接口。

### **3.13.2** **启用**KV API

通过在控制面的配置文件中，添加一个location, 并在location 中使用dyn_sendmsg_kv指令开启KV HTTP API。以下的配置示例中的端口及URL 可以根据需要进行更改。

```
Nginx
   server {
         listen 8081;
         location /kv {
             dyn_sendmsg_kv;
         }
     }
```

### **3.13.3 API **说明

通过配置的location 的url，使用POST方法，设置key value 值， 提交的报文为JSON格式。

```
以下请求请注意替换IP 、端口 、URL，与实际的配置需要一致
```

设置KV值

请求：

```
HTTP
POST http://127.0.0.1:8081/kv
Content-Type: application/json

 {
    "key":"sc_test_key", 
    "value":"here is the test msg"
 }
```

返回：

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Tue, 07 Mar 2023 08:14:38 GMT
Content-Type: text/plain
Content-Length: 20
Connection: keep-alive
 

here is the test msg
```

### 3.13.4 API 说明ACL控制

该模块可以设置acl控制，具体设置方法参考3.25章节描述。

## 3.14 **fastcgi**模块

### 3.14.1  **模块介绍以及原理**

通用网关接口（Common Gateway Interface、CGI）描述了客户端和服务器程序之间传输数据的一种标准，可以让一个客户端，从网页浏览器向执行在网络服务器上的程序请求数据。 CGI 独立于任何语言的，CGI 程序可以用任何脚本语言或者是完全独立编程语言实现，只 要这个语言可以在这个系统上运行。Unix shell script、Python、Ruby、PHP、perl、Tcl、C/C++ 和 Visual Basic 都可以用来编写 CGI 程序

快速通用网关接口（Fast Common Gateway Interface／FastCGI）是通用网关接口（CGI）的改进，描述了客户端和服务器程序之间传输数据的一种标准。FastCGI 致力于减少 Web 服务器 与 CGI 程式之间互动的开销，从而使服务器可以同时处理更多的 Web 请求。与为每个请求 创建一个新的进程不同，FastCGI 使用持续的进程来处理一连串的请求。这些进程由 FastCGI 进程管理器管理（例如下面我们要介绍的spawn-fcgi），而不是 web 服务器

#### 3.14.1.1 **fastcgi**处理流程

![WeChatea9e910c29b6ac9f42578474855405db](https://gitee.com/gebona/picture/raw/master/202306191006307.png)

![](https://gitee.com/gebona/picture/raw/master/202306191000525.png

![image-20230717152720270](https://gitee.com/gebona/picture/raw/master/202307171527978.png)

#### 3.14.1.2 **常用环境变量**

1. SCRIPT_FILENAME $document_root$fastcgi_script_name;#脚本文件请求的路径
2. QUERY_STRING $query_string; #请求的参数;如?app=123
3. REQUEST_METHOD $request_method; #请求的动作(GET,POST)
4. CONTENT_TYPE $content_type; #请求头中的 Content-Type 字段
5. CONTENT_LENGTH $content_length; #请求头中的 Content-length 字段。
6. SCRIPT_NAME $fastcgi_script_name; #脚本名称
7. REQUEST_URI $request_uri; #请求的地址不带参数
8. DOCUMENT_URI $document_uri; #与$uri 相同。
9. DOCUMENT_ROOT $document_root; #网站的根目录。在 server 配置中 root 指令中指定的值
10. SERVER_PROTOCOL $server_protocol; #请求使用的协议，通常是 HTTP/1.0 或 HTTP/1.1。
11. GATEWAY_INTERFACE CGI/1.1;#cgi 版本
12. SERVER_SOFTWARE nginx/$nginx_version;#nginx 版本号，可修改、隐藏

13. REMOTE_ADDR $remote_addr; #客户端 IP
14. REMOTE_PORT $remote_port; #客户端端口
15. SERVER_ADDR $server_addr; #服务器 IP 地址
16. SERVER_PORT $server_port; #服务器端口
17. SERVER_NAME $server_name; #服务器名，域名在 server 配置中指定的 server_name
18. PATH_INFO $path_info;#可自定义变量

### 3.14.2 **功能列表**

官方详细指令介绍请查看 https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html

指令列表(50个，其中15个指令是关于缓存cache，其他是一些buffer设置、超时设置、header设置等基础功能)

 [fastcgi_bind](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_bind)
        [fastcgi_buffer_size](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_buffer_size)
        [fastcgi_buffering](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_buffering)
        [fastcgi_buffers](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_buffers)
        [fastcgi_busy_buffers_size](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_busy_buffers_size)

\#cache 功能

 [fastcgi_cache](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache)                  
        [fastcgi_cache_background_update](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_background_update)
        [fastcgi_cache_bypass](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_bypass)
        [fastcgi_cache_key](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_key)
        [fastcgi_cache_lock](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_lock)
     [   fastcgi_cache_lock_age](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_lock_age)
     [  fastcgi_cache_lock_timeout](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_lock_timeout)
     [  fastcgi_cache_max_range_offset](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_max_range_offset)
       [fastcgi_cache_methods](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_methods)
       [fastcgi_cache_min_uses](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_min_uses)
       [fastcgi_cache_path](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_path)
       [fastcgi_cache_revalidate](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_revalidate)
       [fastcgi_cache_use_stale](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_use_stale)
       [fastcgi_cache_valid](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_valid)
       [fastcgi_catch_stderr](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_catch_stderr)  

 [fastcgi_connect_timeout](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_connect_timeout)
        [fastcgi_force_ranges](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_force_ranges)
        [fastcgi_hide_header](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_hide_header)
        [fastcgi_ignore_client_abort](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_ignore_client_abort)
        [fastcgi_ignore_headers](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_ignore_headers)
        [fastcgi_index](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_index)
        [fastcgi_intercept_errors](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_intercept_errors)
        [fastcgi_keep_conn](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_keep_conn)
        [fastcgi_limit_rate](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_limit_rate)
        [fastcgi_max_temp_file_size](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_max_temp_file_size)
        [fastcgi_next_upstream](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_next_upstream)
        [fastcgi_next_upstream_timeout](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_next_upstream_timeout)
        [fastcgi_next_upstream_tries](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_next_upstream_tries)
        [fastcgi_no_cache](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_no_cache)
        [fastcgi_param](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_param)

\#pass功能， 注意pass 地址支持两种，unix socket以及IP端口形式

   [fastcgi_pass](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass)
           [fastcgi_pass_header](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass_header)
          [fastcgi_pass_request_body](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass_request_body)
          [fastcgi_pass_request_headers](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass_request_headers)
          [fastcgi_read_timeout](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_read_timeout)
          [fastcgi_request_buffering](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_request_buffering)
          [fastcgi_send_lowat](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_send_lowat)
          [fastcgi_send_timeout](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_send_timeout)
          [fastcgi_socket_keepalive](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_socket_keepalive)
          [fastcgi_split_path_info](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_split_path_info)
          [fastcgi_store](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_store)
          [fastcgi_store_access](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_store_access)
          [fastcgi_temp_file_write_size](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_temp_file_write_size)
          [fastcgi_temp_path](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_temp_path)

### 3.14.3 **测试样例**

#### 3.14.3.1 **环境搭建**

**FasiCGI**进程管理器（有多种，此处使用Php-fpm测试）

spawn-fcgi是一个通用的 FastCGI 进程管理器，简单小巧，原先是属于 lighttpd 的一部分， 管理C语言编写的FastCGI程序

php-fpm 即 PHP FastCGI 进程管理器(管理PHP的)，并且提供进程 管理的功能。进程包括master进程和worker进程。master进程只有一个，负责监听端口，接受来自 web server的请求。worker进程一般会有多个，每个进程中会嵌入一个PHP解析器，进行PHP代码的处 理

进程管理器，只是创建一些进程池进行处理，真正的具体业务需要由对应的业务处理程序处理

![image-20230619101605634](https://gitee.com/gebona/picture/raw/master/202306191016777.png)

本次测试环境（可根据测试需要自行安装, 推荐OpenNJet代理与php-fpm部署在不同机器测试， unix地址测试可采用本机测试）：

安装： centos下可通过命令安装 php-fpm：    yum install php-fpm.x86_64

配置： 配置文件位于 /etc/php-fpm.conf  以及     /etc/php-fpm.d/www.conf

 vim /etc/php-fpm.d/www.conf   编辑该文件可设置监听IP以及端口或者unix地址，此处设置为ip: 127.0.0.1:9000

![image-20230619101641410](https://gitee.com/gebona/picture/raw/master/202306191016683.png)

启动: service php-fpm retart      也支持 (stop/restart)

具体可参考 https://zhuanlan.zhihu.com/p/134029988

### 3.14.4 OpenNJet fastcgi配置示例 

njet.conf

```
C
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    log_format scripts '$document_root$fastcgi_script_name > $request';
    access_log logs/scripts.log scripts;


    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root /usr/share/njet/html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            include        fastcgi.conf;
        }
    }
```

fastcgi.conf (官方提供的一个默认的配置文件，可根据需要改动)

```
C
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    njet/$njet_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

index.php

```
PHP
<?php
echo "This is my first PHP page!";
?>
```



## 3.15 **正向代理**

配置在客户端，客户端所有流量都通过正向代理去访问

### 3.15.1 **指令介绍**

#### 3.15.1.1 **proxy_connect**

Syntax: **proxy_connect**
        Default: none
        Context: server

启用“ CONNECT”HTTP 方法支持。

#### 3.15.1.2 **proxy_connect_allow**

Syntax: **proxy_connect_allow** **all | [port ...] | [port-range ...]**
 Default: 443 563
 Context: server

此指令指定代理 CONNECT 方法可以连接到的端口号或范围的列表。
 默认情况下，只启用默认的 https 端口(443)和默认的 snews 端口(563)。
 使用此指令将覆盖此默认值，并仅允许连接到列出的端口。

All 值将允许所有端口进行代理。

给定值将允许指定的端口代理。

port-range 将允许指定的端口到代理的范围，例如:

```
Plaintext
proxy_connect_allow 1000-2000 3000-4000; # allow range of port from 1000 to 2000, from 3000 to 4000.
```

#### 3.15.1.3 **proxy_connect_connect_timeout**

Syntax: **proxy_connect_connect_timeout** **time**
        Default: none
        Context: server

定义与代理服务器建立连接的超时。

### 3.15.2 **代理配置示例**

Example

```
Bash
server {
        listen 80;    //端口设定
        resolver 114.114.114.114; # dns解析服务器
        
        proxy_connect;
        proxy_connect_allow            443, 456;
        proxy_connect_connect_timeout  10s;
        
        location /{
              proxy_pass $scheme://$host$request_uri; #proxy_pass 用来要代理的网站，
              #$scheme是客户端请求的协议（如http，https）；
              #$host是客户端请求的域名（如baidu.com）；
              #$request_uri是客户端访问的url地址（如/baidu?s=12345）。
              #他们拼接成就是http://baidu.com/baidu?s=12345
        }
    }
}
```

### 3.15.3 **客户端配置（**win10系统为例）

![image-20230619102106220](https://gitee.com/gebona/picture/raw/master/202306191021429.png)



## 3.16 **反向代理**

反向代理（Reverse Proxy）是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

### 3.16.1 **指令介绍**

#### 3.16.1.1 **proxy_pass**

设置连接被代理服务器的协议、IP 地址或套接字，也可以是域名或 upstream 定义的服务器组。

```
SQL
Syntax:proxy_pass URL;
Default:—
Context:location, , if in locationlimit_except
```

#### 3.16.1.2 **proxy_hide_header**

指定被代理服务器响应数据中不向客户端传递的 HTTP 头字段名称。

```
SQL
Syntax:proxy_hide_header field;
Default:—
Context:http, server, location
```

#### 3.16.1.3 **proxy_http_version**

设置用于代理的 HTTP 协议版本，若使用 keepalive 或 NTLM 认证，建议指令值设置为 1.1。

```
SQL
Syntax:proxy_http_version 1.0 | 1.1;
Default:proxy_http_version 1.0;
Context:http, server, location
```

#### 3.16.1.4 **proxy_redirect**

替换被代理服务器返回的响应头中属性字段 location 或 Refresh 的值，并返回给客户端。指令值为 default 时，使用 proxy_pass 指令值的内容进行替换。

#### 3.16.1.5 **proxy_pass_header**

默认配置下 NGINX不会将头属性字段 Status 和 X-Accel-... 传递给客户端，可通过该指令开放传递。

```
SQL
Syntax:proxy_pass_header field;
Default:—
Context:http, server, location
```

#### 3.16.1.6 **proxy_set_header**

在转发给被代理服务器前，修改或添加客户端的请求头属性字段

```
SQL
Syntax:proxy_set_header field value;
Default:proxy_set_header Host $proxy_host;
        proxy_set_header Connection close;
Context:http, server, location
```

#### 3.16.1.7 **proxy_socket_keepalive**

设置NGINX与被代理服务器的 TCP keepalive 行为的心跳检测机制，默认使用操作系统的 socket 配置。若指令值为 on，则开启 SO_KEEPALIVE 选项进行心跳检测

```
SQL
Syntax:proxy_socket_keepalive on | off;
Default:proxy_socket_keepalive off;
Context:http, server, location
```

### 3.16.2 **配置示例**

```
Bash
upstream backend1{
        zone upstream_backend1 64k;
        server 127.0.0.1:8001;
        server 127.0.0.1:8002;
    }
upstream backend2{
        zone upstream_backend2 64k;
        server 127.0.0.1:8003;
    }
    
    server{
        location /nginx {
           proxy_http_version 1.1;
           proxy_pass http://http1;
           proxy_hide_header Vary;
           proxy_pass_header server;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_redirect http://http1 http://http2;
           proxy_socket_keepalive on;
        }
    }
```



## **3.17 JWT认证**

### 3.17.1 **简介**

该模块用于检查JWT-token 是否有效。

### 3.17.2 **指令说明**

```
配置语法:         auth_jwt $variable | on | off;
默认值:            auth_jwt off;
允许配置位置:  http、http>server、 http>location
```

启用 JWT 的验证，仅当变量值为on时或者配置为on时开启验证。

```
配置语法:         auth_jwt_key value [encoding];
默认值:            无
允许配置位置: http、http>server、 http>location
```

指定用于验证 JWT 签名的密钥。
       *编码* 可以是 hex | utf8 | base64 | file（默认值为 utf8 ）。
       如果是文件该选项要求*该值*为有效的文件路径（指向 PEM 编码的公钥）。

```
配置语法:  auth_jwt_alg any | HS256 | HS384 | HS512 | RS256 | RS384 | RS512 | ES256 | ES384 | ES512;
默认值:     auth_jwt_alg any;
允许配置位置: http、http>server、 http>location
```

指定服务器期望在接收的 JWT 的算法。

### 3.17.3 **配置示例**

```
Nginx
server {    
    auth_jwt_key "0123456789abcdef" hex; # Your key as hex string    
    auth_jwt     off;    
    location /secured-by-cookie/ {
        auth_jwt $cookie_MyCookieName;
    }    
    location /secured-by-auth-header/ {
        auth_jwt on;    
    }  
    location /secured-by-auth-header-too/ {
          auth_jwt_key "another-secret"; # Your key as utf8 string  
          auth_jwt on;    
    }   
    location /secured-by-rsa-key/ { 
        auth_jwt_key /etc/keys/rsa-public.pem file; # Your key from a PEM file   
        auth_jwt on;
    } 
}
```



## 3.18 **Mirror**

 镜像原始请求 通过创建后台镜像子请求。 对镜像子请求的响应将被忽略。

### 3.18.1 **指令说明**

```
配置语法: mirror  uri | off;
默认值:     mirror off;
允许配置位置: http、http>server、 http>location
```

设置原始请求将镜像到的 URI。 可以在同一配置级别上指定多个镜像。

```
配置语法: mirror_request_body on| off;
默认值:     mirror_request_body on;
允许配置位置: http、http>server、 http>location
```

指示是否镜像客户端请求正文。 启用后，将读取客户端请求正文 在创建镜像子请求之前。 在这种情况下，无缓冲的客户端请求正文代理 ，proxy_request_buffering、fastcgi_request_buffering、scgi_request_buffering、uwsgi_request_buffering指令将被禁用。

### 3.18.2 **配置示例**

```
Nginx
location / {
    mirror /mirror;
    proxy_pass http://backend;
}

location = /mirror {
    internal;
    proxy_pass http://test_backend$request_uri;
}
```



## 3.19 **服务发现**

### 3.19.1  **简介**

http，stream upstream 配置域名格式的server，定期查询dns server进行域名解析，实现对upstream server 的添加，删除，修改。

### 3.19.2 **配置说明**

支持http 和 stream 的upstream 配置。

要开启该功能, 需在njet.conf 的 http 或 stream 的upstream block 修改配置：

```
user root root;
worker_processes 1;
error_log /home/njet/logs/error.log debug;
events {
    worker_connections  1024;
}

http {
    dyn_kv_conf conf/iot.conf;
    error_log /home/njet/logs/error.log debug;
    access_log /home/njet/logs/access.log combined;

    upstream http1{
            zone upstream_http 64k;
            resolver 192.168.40.101 valid=10s;
            state  zyg11.conf;
    }
   upstream http2{
            zone upstream_http 64k;
            resolver 192.168.40.101 valid=10s;
            server  www.dev.test.com resolve;
    }
    server {
        listen        8000;
        server_name  cluster1;
        location / {
            proxy_pass http://http1;
        }
    }
    server {
        listen 8899;
        server_name cluster2;
    }
}
stream{

   upstream stream_upstream{
            zone upstream_stream 64k;
            resolver 192.168.40.101 valid=10s;
            state  zyg_stream.conf;
            }
   server {
    listen        9000;
    proxy_pass  stream_upstream;
   }
}
helper ctrl /home/njet/modules/njt_helper_ctrl_module.so /home/njet/conf/njet_ctrl.conf;
helper broker /home/njet/modules/njt_helper_broker_module.so conf/mqtt.conf;
cluster_name helper;
node_name node1;
```

<table>
<tbody>
<tr>
<td width="153">
<p>配置项</p>
</td>
<td width="153">
<p> 必须修改 </p>
</td>
<td width="331">
<p>配置说明</p>
</td>
</tr>
<tr>
<td width="153">
<p>zone</p>
</td>
<td width="58">
<p>是</p>
</td>
<td width="420">
<p>upstream 的zone 的名字，及大小。 例如：zone upstream_http 64k;&nbsp; zone upstream_stream 64k;</p>
</td>
</tr>
<tr>
<td width="163">
<p>resolver</p>
<p>&nbsp;</p>
</td>
<td width="58">
<p>是</p>
<p>&nbsp;</p>
</td>
<td width="420">
<p>dns 服务器的地址。resolver 192.168.40.101 valid=10s;</p>
</td>
</tr>
<tr>
<td width="163">
<p>valid</p>
</td>
<td width="58">
<p>&nbsp;否</p>
</td>
<td width="420">
<p>域名失效的周期，也就是查询dns 的周期。支持标准的OpenNJet时间字段。resolver 192.168.40.101 valid=10s;</p>
</td>
</tr>
<tr>
<td width="163">
<p>resolver_timeout</p>
</td>
<td width="58">
<p>否</p>
</td>
<td width="420">
<p>查询dns 超时时间。支持标准的OpenNJet时间字段。</p>
</td>
</tr>
<tr>
<td width="163">
<p>state</p>
</td>
<td width="58">
<p>否</p>
</td>
<td width="420">
<p>state&nbsp; zyg_stream.conf;&nbsp; 通过upstream api 接口添加的upstream server 信息的保存路径。reload 之后不会丢失。不配置的话，不保存。 reload 之后丢失。</p>
</td>
</tr>
<tr>
<td width="163">
<p>resolve</p>
</td>
<td width="58">
<p>否</p>
</td>
<td width="420">
<p>配置静态的动态域名。删除后reload 之后，还会加载。 例如：&nbsp;server&nbsp; www.dev.test.com resolve;注意：server 字段和state 字段冲突。&nbsp; 不能静态动态混用。</p>
</td>
</tr>
</tbody>
</table>

### 3.19.3 **依赖：**

a.   Upstream API 接口， 动态解析的 IP 可以通过upstream api 查看信息。

b.   需要搭建dns 服务器，进行域名解析 IP 的添加，删除，修改



## 3.20 **动态开关功能**

### 3.20.1 **简介**

该模块使用声明式API实现access log 等模块的部分功能动态开关。其通过控制面Copilot:njet_ctrl实现接口，借助Copilot：broker，sendmsg，kv等模块构建的消息总线，传递到worker中，worker即数据面通过实现特定模块的动态配置接口从而实现整体的动态控制。

### 3.20.2 **控制面配置说明**

依赖模块

```
njt_http_sendmsg_module.so
njt_ctrl_config_api_module.so
```

指令说明

config_req_pool_size

```
语法:        config_req_pool_size  number;
默认值:        500
允许配置位置:         http->main
```

配置查询请求（GET）等待池的大小，要求大于查询接口请求最大qps。

config_api

```
语法:        config_api;
默认值:        无
允许配置位置:         http->location
```

配置开启动态config api

配置示例

控制面配置

```
Nginx
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_ctrl_config_api_module.so;
error_log stderr info;# 注意修改为具体的文件路径，此配置为输出到控制台

events {
    worker_connections  1024;
}

http {
    access_log logs/access_ctrl.log combined;
    config_req_pool_size 1000;

    server {
        listen       8081;

        location /api {
            config_api;
        }
    }
}
```

### 3.20.3 **数据面配置说明**

#### 3.20.3.1 **access log** **数据面配置说明**

依赖模块

控制面参见上文控制面配置说明

```
njt_agent_dynlog_module.so
```

#### 3.20.3.2 **配置指令**

无

#### 3.20.3.3 **调用样例**

##### 3.20.3.3.1 **查询**access log**当前状态**

该接口可以查看所有HTTP location 块内access log 当前的开关状态

请求

```
HTTP
GET http://127.0.0.1:8443/config/1/config/http_log
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 24 Feb 2023 06:52:19 GMT
Content-Type: application/json
Content-Length: 480
Connection: keep-alive

{
  "servers": [
    {
      "listens": [
        "0.0.0.0:1081"
      ],
      "serverNames": [
        ""
      ],
      "locations": [
        {
          "location": "/",
          "accessLogOn": true,
          "locations": [
            {
              "location": "/demo",
              "accessLogOn": false
            }
          ]
        },
        {
          "location": "= /test/demo",
          "accessLogOn": true
        }
      ]
    },
    {
      "listens": [
        "0.0.0.0:1080"
      ],
      "serverNames": [
        ""
      ],
      "locations": [
        {
          "location": "/",
          "accessLogOn": true
        },
        {
          "location": "= /test/demo",
          "accessLogOn": true
        },
        {
          "location": "/demo",
          "accessLogOn": true
        },
        {
          "location": "/demos",
          "accessLogOn": true
        },
        {
          "location": "/named",
          "accessLogOn": true
        }
      ]
    }
  ]
}
```

##### 3.20.3.3.2 **修改**access log**开关状态**

```
此处请求body可以支持发送仅局部json
报错提示在error log中，http不做参数校验
```

请求

```
HTTP
PUT http://127.0.0.1:8443/config/1/config/http_log
Content-Type: application/json

{
  "servers": [
    {
      "listens": [
        "0.0.0.0:1081"
      ],
      "serverNames": [
        ""
      ],
      "locations": [
        {
          "location": "/",
          "accessLogOn": true,
          "locations": [
            {
              "location": "/demo",
              "accessLogOn": false
            }
          ]
        }
      ]
    }
  ]
}
```

返回值

```
HTTP
HTTP/1.1 204 No Content
Server: njet/1.23.1
Date: Thu, 23 Feb 2023 11:33:08 GMT
Connection: keep-alive
```

完成效果

logOff字段为true的location块处理请求时不再打印accesslog。

logOff字段为false的location块处理请求时打印accesslog。

#### 3.20.3.4 **动态黑白名单配置**

##### **3.20.3.4.1** **依赖模块**

控制面参见上文控制面配置说明

数据面需要加载如下模块

```
njt_helper_broker_module
njt_http_dyn_bwlist_module.so
```

##### **3.20.3.4.2** **配置指令**

动态配置无特殊的指令， 动态黑白名单的配置指令及格式与静态配置一致，使用 allow, deny 指定允许或拒绝的IP地址。

```
语法:        allow address | CIDR | all；
默认值:     --   
允许配置位置:    http, server, location
```

```
语法:        deny address | CIDR | all；
默认值:     --
允许配置位置:    http, server, location
```

##### **3.20.3.4.3** **配置示例**

```
Nginx
 worker_processes 1;
error_log logs/error.log info;
load_module /home/njet/modules/njt_http_dyn_bwlist_module.so;
helper broker /home/njet/modules/njt_helper_broker_module.so conf/mqtt.conf;
helper broker /home/njet/modules/njt_helper_ctrl_module.so conf/njet_ctrl.conf;

events {
    worker_connections  1024;
}
 upstream backend {
  server 10.10.12.1:8080;
 }
 
 http{ 
   dyn_kv_conf conf/iot.conf;
    server {
        listen       18888;
        location / {
           proxy_pass http://${backend};
        }
        location /test_bwlist {
          allow 192.168.1.0/24;
          deny all; 
          proxy_pass http://${backend};
       }
    }
 }
cluster_name helper;
node_name node1;
```

##### **3.20.3.4.4** **接口示例**

使用GET方法获取当前各location 的黑白名单配置，使用PUT方法更新黑白名单配置。PUT 提交时报文格式与GET获取到的报文格式一致，可以只提交需要改动的 location。

```
HTTP
GET http://127.0.0.1:8443/api/1/config/http_dyn_bwlist

{
        "servers": [{
                "listens": ["0.0.0.0:18888"],
                "serverNames": [""],
                "locations": [{
                        "location": "/"
                }, {
                        "location": "/test_bwlist",
                        "accessIpv4": [{
                                "rule": "allow",
                                "addr": "192.168.1.0",
                                "mask": "255.255.255.0"
                        }, {
                                "rule": "deny",
                                "addr": "0.0.0.0",
                                "mask": "0.0.0.0"
                        }]
                }]
              }]
}
```

##### **3.20.3.4.5** **接口消息返回提示**

###### **3.20.3.4.5.1** **查询**

使用GET方法获取当前黑白名单静态配置信息。

```
Bash
curl -X 'GET' 'http://127.0.0.1:8081/config/2/config/http_dyn_bwlist'
```

```
Bash
示例返回：
{
  "servers": [
    {
      "listens": [
        "0.0.0.0:23"
      ],
      "serverNames": [
        "testy"
      ],
      "locations": [
        {
          "location": "/",
          "accessIpv4": [
            {
              "rule": "deny",
              "addr": "0.0.0.0",
              "mask": "0.0.0.0"
            }
          ]
        }
      ]
    }
  ]
}
```

###### **3.20.3.4.5.2** **配置修改**

修改PUT方法的配置，对应返回消息提示

```
Bash
curl -X 'PUT' 'http://127.0.0.1:8081/config/2/config/http_dyn_bwlist' -d '{
  "servers": [
    {
      "listens": [
        "0.0.0.0:23"
      ],
      "serverNames": [
        "testy"
      ],
      "locations": [
        {
          "location": "/",
          "accessIpv4": [
            {
              "rule": "deny",
              "addr": "192.168.40.157",
              "mask": "255.255.255.255"
            }
          ]
        }
      ]
    }
  ]
}'
```

###### **3.20.3.4.5.3** **结果**

```
Bash
{
  "code": 0,
  "msg": "success."
}
```

#### 3.20.3.5 **动态分流**

##### **3.20.3.5.1** **依赖模块**

控制面参见上文控制面配置说明

数据面需要加载如下模块

```
njt_helper_broker_module
njt_http_split_clients_2_module.so
```

##### **3.20.3.5.2** **配置指令**

动态配置无特殊的指令， split_clients_2 的配置指令及格式与静态配置一致。

##### **3.20.3.5.3** **配置示例**

```
Nginx
worker_processes 1;
error_log logs/error.log info;
load_module /home/njet/modules/njt_http_split_clients_2_module.so;
helper broker /home/njet/modules/njt_helper_broker_module.so conf/mqtt.conf;
helper broker /home/njet/modules/njt_helper_ctrl_module.so conf/njet_ctrl.conf;

events {
    worker_connections  1024;
}
 
 http{ 
   dyn_kv_conf conf/iot.conf;
   split_clients_2 $backend {
      10%  backend1;
      *             backend2;
   }
   upstream backend1 {
     server 127.0.0.1:18080;
   }
   upstream backend2 {
     server 127.0.0.1:18081;
   }
   server {
        listen     18888;
        location / {
           proxy_pass http://${backend};
        }
   }
    server {
        listen     18080;
        location / {
            return 200 "welcome to 18080 http server\n";
        }
    }
    server {
        listen    18081;
        location / {
            return 200 "welcome to 18081 http server\n";
        }
    }
}
cluster_name helper;
node_name node1;
```

**3.20.3.5.4** **接口示例**

使用GET方法获取当前split clients 2 模块的流量配比设置，使用PUT 方法更新流量配比设置。 PUT时使用的报文格式与GET获取到的报文格式一致。

```
HTTP
GET http://127.0.0.1:8443/api/1/config/http_split_clients_2

{
        "http": {
                "split_clients_2": {
                        "backend1": 10,
                        "backend2": 90
                }
        }
}
```

#### 3.20.3.6 **动态**metric

##### 3.20.3.6.1 **依赖模块**

控制面参见上文控制面配置说明。

数据面需要加载如下模块

```
njt_helper_broker_module
njt_http_vtsc_module.so
```

##### 3.20.3.6.2 **配置指令**

动态配置无特殊的指令。

##### 3.20.3.6.3 **配置示例**

主要配置示例

```
Nginx
load_module objs/njt_http_vtsc_module.so;

helper ctrl objs/njt_helper_ctrl_module.so conf/njet_ctrl.conf;
helper broker objs/njt_helper_broker_module.so conf/mqtt.conf;

worker_processes  1;
error_log logs/error.log info;

events {
    worker_connections  1024;
}

http {
    dyn_kv_conf conf/iot.conf;

    access_log logs/access.log combined;
    variables_hash_max_size 2048;

    proxy_cache_path cache levels=1:2  keys_zone=cache:20m  max_size=1g  inactive=30d  use_temp_path=off;
    proxy_cache_valid any 10m ;
    proxy_cache_revalidate on;
    proxy_cache_lock on;
    proxy_cache_key $scheme$host$request_uri$slice_range;

    vhost_traffic_status_zone;
    vhost_traffic_status_filter_by_set_key "$request_uri" "$server_name";

    upstream backgroup {
        zone backcenter 128k;
        server  192.168.30.222:80;
    }

    upstream errgroup {
        zone backcenter 128k;
        server  192.168.40.108:28000;
    }

    server {
        listen       8081;
        server_name  server1;
        server_name  server11;

        location / {
            add_header x-Cache-Status $upstream_cache_status;
            add_header x-Cache-Key $scheme$host$request_uri$slice_range;
            proxy_cache cache;

            proxy_pass     http://backgroup;
        }

        location  /a {
            return 200 "welcome to 8081 http server a\n";
        }        
    }

    server {
        listen       8082;
        server_name  server2;

        location / {
            proxy_pass     http://errgroup;
        }

        location /c {
            root html;
            index index.html;
        }

        location /d {
            root html;
            index index.html;
        }
      
    }

    server {
        listen       8083;
        server_name  server3;

        location / {
            root html;
            index index.html;
        }

        location /a {
            root html;
            index index.html;
        }

        location /b {
            root html;
            index index.html;
        }        
    }
}
```

##### 3.20.3.6.4 **接口示例**

使用GET方法获取当前VTS模块的设置，使用PUT 方法更新VTS模块的设置。 PUT时使用的报文格式与GET获取到的报文格式一致。 

查询当前VTS模块的设置：

```
HTTP
curl -X GET http://127.0.0.1:8443/api/1/config/http_vts

返回：
{
  "vhost_traffic_status_filter_by_set_key": "\"$request_uri\" \"$server_name\"",
  "servers": [
    {
      "listens": [
        "0.0.0.0:8081"
      ],
      "serverNames": [
        "server1",
        "server11"
      ],
      "locations": [
        {
          "location": "/",
          "vhost_traffic_status": true
        },
        {
          "location": "/a",
          "vhost_traffic_status": true
        }
      ]
    },
    {
      "listens": [
        "0.0.0.0:8082"
      ],
      "serverNames": [
        "server2"
      ],
      "locations": [
        {
          "location": "/",
          "vhost_traffic_status": true
        },
        {
          "location": "/c",
          "vhost_traffic_status": true
        },
        {
          "location": "/d",
          "vhost_traffic_status": true
        }
      ]
    },
    {
      "listens": [
        "0.0.0.0:8083"
      ],
      "serverNames": [
        "server3"
      ],
      "locations": [
        {
          "location": "/",
          "vhost_traffic_status": true
        },
        {
          "location": "/a",
          "vhost_traffic_status": true
        },
        {
          "location": "/b",
          "vhost_traffic_status": true
        }
      ]
    }
  ]
}
```

动态配置：

```
Nginx
curl -X GET http://127.0.0.1:8443/api/1/config/http_vts

返回：
{
  "vhost_traffic_status_filter_by_set_key": "\"$status\" \"$server_name::$request_uri\"",
  "servers": [
    {
      "listens": [
        "0.0.0.0:8081"
      ],
      "serverNames": [
        "server1"
      ],
      "locations": [
        {
          "location": "/a",
          "vhost_traffic_status": false
        }
      ]
    }
  ]
}
```

再次查询当前的配置：

```
Nginx
PUT http://127.0.0.1/api/1/config/http_vts HTTP/1.1
content-type: application/json

{
  "vhost_traffic_status_filter_by_set_key": "\"$status\" \"$server_name::$request_uri\"",
  "servers": [
    {
      "listens": [
        "0.0.0.0:8081"
      ],
      "serverNames": [
        "server1",
        "server11"
      ],
      "locations": [
        {
          "location": "/",
          "vhost_traffic_status": true
        },
        {
          "location": "/a",
          "vhost_traffic_status": false
        }
      ]
    },
    {
      "listens": [
        "0.0.0.0:8082"
      ],
      "serverNames": [
        "server2"
      ],
      "locations": [
        {
          "location": "/",
          "vhost_traffic_status": true
        },
        {
          "location": "/c",
          "vhost_traffic_status": true
        },
        {
          "location": "/d",
          "vhost_traffic_status": true
        }
      ]
    },
    {
      "listens": [
        "0.0.0.0:8083"
      ],
      "serverNames": [
        "server3"
      ],
      "locations": [
        {
          "location": "/",
          "vhost_traffic_status": true
        },
        {
          "location": "/a",
          "vhost_traffic_status": true
        },
        {
          "location": "/b",
          "vhost_traffic_status": true
        }
      ]
    }
  ]
}
```

#### 3.20.3.7 **动态**telemetry

##### 3.20.3.7.1 **依赖模块**

上文控制面配置说明

```
njt_helper_broker_module
njt_ctrl_config_api_module.so;
```

本功能需要模块，因为本功能采用外部build，所以版本发布的时候提供对应的so

服务间追踪模块(Otel)依赖: 

​      njt_otel_module.so

​      njt_agent_dyn_otel_module.so

服务间追踪模块(Otel-webserver)依赖: 

​      njt_otel_webserver_module.so

​      njt_agent_dyn_otel_webserver_module.so

##### 3.20.3.7.2 **配置指令**

无

##### 3.20.3.7.3 **配置示例**

主要配置示例

```
Nginx

# 分布式telemetry 动态开关需要加载这两个模块
load_module modules/njt_otel_module.so;
load_module modules/njt_agent_dyn_otel_module.so;


# telemetry-webserver 动态开关需要加载这两个模块
load_module modules/njt_otel_webserver_module.so;
load_module modules/njt_agent_dyn_otel_webserver_module.so;
worker_processes  1;

error_log  stderr info; # 注意修改为具体的文件路径，此配置为输出到控制台

helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl /modules/njt_helper_ctrl_module.so conf/ctrl_demo.conf;

events {
    worker_connections  1024;
}
cluster_name helper;
node_name node1;

http {
    dyn_kv_conf conf/iot.conf;
    access_log logs/access.log combined; ## 此处为必要配置，日志格式可以修改
    
    # 分布式telemetry模块配置文件
    opentelemetry_config conf/otel.toml;
    # telemetry-webserver模块配置文件
    include conf/opentelemetry_module.conf;

    upstream demo {
       zone demo 128k;
        server 192.168.40.141:8080;
    }

    upstream demos {
        zone demos 128k;
        server 192.168.40.141:443;
    }

    server {
        listen       1081;

        location / {
            # telemetry-webserver 配置
            NjetModuleEnabled OFF;
            # 分布式telemetry 配置
            opentelemetry on;
            opentelemetry_operation_name my_example_backend;
            opentelemetry_propagate;
            
            root html;
            index index.html;
            location /demo {
                proxy_pass http://demo;
            }
        }

        location = /test/demo {
           # telemetry-webserver 配置
            NjetModuleEnabled ON;
            # 分布式telemetry 配置
            opentelemetry on;
            opentelemetry_operation_name my_example_backend;
            opentelemetry_propagate;
            
            proxy_pass http://demo;
        }
    }
    server {
        listen       1080;

        location / {
            root html;
            index index.html;
        }

        location = /test/demo {
            limit_conn addr 100;
            proxy_pass http://demo;
        }

        location /demo {
            add_header test api;
            proxy_pass http://demo;
        }
        location /demos {
            limit_conn addr 100;
            proxy_ssl_certificate nginx.pem;
            proxy_ssl_certificate_key nginx-key.pem;
            proxy_pass https://demos;
        }
    }

}
```

##### 3.20.3.7.4 **返回值说明：**

![img](https://gitee.com/gebona/picture/raw/master/202306191135839.jpg)

​                                                                                                   **点击图片可查看完整电子表格**

##### 3.20.3.7.5 **接口示例**

###### 3.20.3.7.5.1 **查询**telemetry **当前状态**

该接口可以查看所有HTTP location 块内telemetry 当前的开关状态

请求

```
HTTP
GET http://127.0.0.1:8081/config/2/config/otel
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Fri, 24 Feb 2023 06:52:19 GMT
Content-Type: application/json
Content-Length: 480
Connection: keep-alive

* Connection #0 to host 127.0.0.1 left intact
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:1081"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "opentelemetry": true,
                    "locations": [
                        {
                            "location": "/demo",
                            "opentelemetry": true
                        }
                    ]
                },
                {
                    "location": "/test/demo",
                    "opentelemetry": true
                }
            ]
        },
        {
            "listens": [
                "0.0.0.0:1080"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "opentelemetry": true
                },
                {
                    "location": "= /test/demo",
                    "opentelemetry": true
                },
                {
                    "location": "/demo",
                    "opentelemetry": true
                },
                {
                    "location": "/demos",
                    "opentelemetry": true
                },
                {
                    "location": "/named",
                    "opentelemetry": true
                }
            ]
        }
    ]
}
```

###### 3.20.3.7.5.2 **修改**telemetry **开关状态**

```
此处请求body可以支持发送仅局部json
报错提示在error log中，http不做参数校验
```

请求

```
HTTP
PUT http://127.0.0.1:8081/config/2/config/otel
Content-Type: application/json

{
    "servers": [
        {
            "listens": [
                "0.0.0.0:1081"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "opentelemetry": true,
                    "locations": [
                        {
                            "location": "/demo",
                            "opentelemetry": true
                        }
                    ]
                },
                {
                    "location": "/test/demo",
                    "opentelemetry": false
                }
            ]
        },
        {
            "listens": [
                "0.0.0.0:1080"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "opentelemetry": true
                },
                {
                    "location": "= /test/demo",
                    "opentelemetry": true
                },
                {
                    "location": "/demo",
                    "opentelemetry": true
                },
                {
                    "location": "/demos",
                    "opentelemetry": true
                },
                {
                    "location": "/named",
                    "opentelemetry": true
                }
            ]
        }
    ]
}
```

返回

```
Bash
HTTP/1.1 204 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:06:21 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive
```

###### 3.20.3.7.5.3 **查询**telemetry webserver **当前状态**

该接口可以查看所有HTTP location 块内telemetry webserver当前的开关状态

请求

```
HTTP
GET http://127.0.0.1:8081/config/2/config/otel_webserver
```

返回值

```
HTTP
HTTP/1.1 200 OK
Server: njet/1.23.1
Date: Tue, 28 Feb 2023 08:52:12 GMT
Content-Type: application/json
Content-Length: 538
Connection: keep-alive

{
    "servers": [
        {
            "listens": [
                "0.0.0.0:1081"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "NjetModuleEnabled": true,
                    "locations": [
                        {
                            "location": "/demo",
                            "NjetModuleEnabled": true
                        }
                    ]
                },
                {
                    "location": "/test/demo",
                    "NjetModuleEnabled": true
                }
            ]
        }
    ]
}
```

###### 3.20.3.7.5.4 **修改**telemetry webserver **开关状态**

```
此处请求body可以支持发送仅局部json
报错提示在error log中，http不做参数校验
```

请求

```
HTTP
PUT http://127.0.0.1:8081/config/2/config/otel_webserver
Content-Type: application/json

{
    "servers": [
        {
            "listens": [
                "0.0.0.0:1081"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "NjetModuleEnabled": true,
                    "locations": [
                        {
                            "location": "/demo",
                            "NjetModuleEnabled": true
                        }
                    ]
                },
                {
                    "location": "/test/demo",
                    "NjetModuleEnabled": false
                }
            ]
        }
    ]
}
```

返回

```
Bash
HTTP/1.1 204 OK
Server: njet/1.23.1
Date: Fri, 10 Feb 2023 13:06:21 GMT
Content-Type: application/json
Content-Length: 37
Connection: keep-alive
```

#### 3.20.3.8 **动态证书**

##### 3.20.3.8.1 **简介**

该模块支持在运行中替换http server的服务端证书。

##### 3.20.3.8.2 **依赖模块**

```
njt_helper_broker_module
njt_http_kv_module.so
njt_dyn_ssl_module.so
```

##### 3.20.3.8.3 **配置指令**

无

##### 3.20.3.8.4 **配置示例**

主要配置示例

```
注意主配置文件中需要修改so路径，log路径，替换ssl证书
```

```
load_module /modules/njt_http_kv_module.so;
load_module /mnt/f/workspace/c/njet_main/objs/njt_dyn_ssl_module.so;
worker_processes  1;
daemon off;
master_process on;
error_log  stderr info;
helper broker /modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl /modules/njt_helper_ctrl_module.so conf/ctrl_ssl.conf;

events {
    worker_connections  1024;
}
cluster_name helper;
node_name node1;
http {
    access_log logs/access.log combined;
    dyn_kv_conf conf/iot-work.cf;
    upstream demo {
       zone demo 128k;
        server 192.168.40.141:8080;
        keepalive 10240;
    }
    server {
        listen 443 ssl;

        #开启国密功能
        ssl_ntls     on;

        #国际 ECC 证书（可选）
        ssl_certificate                 certs/server.crt;
        ssl_certificate_key             certs/server.key;

        #ssl_certificate            certs/SS.crt certs/SE.crt;
        #ssl_certificate_key        certs/SS.key  certs/SE.key;
        #国密套件
        #ssl_ciphers "ECC-SM2-SM4-CBC-SM3:ECDHE-SM2-WITH-SM4-SM3:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:ECDHE-RSA-AES128-SHA256:!aNULL:!eNULL:!RC4:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS";

        #ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

        default_type            text/plain;

        add_header  "Content-Type" "text/html;charset=utf-8";

        location / {
            return 200 "njet ntls test OK, ssl_protocol is $ssl_protocol (NTLSv1.1 表示国密，其他表示国际)";
        }
    }
}
```

mqtt.conf

```
log_type error
```

iot.conf

```
log_type error
```

##### 3.20.3.8.5 **接口示例**

###### 3.20.3.8.5.1 **查询**http server ssl **当前配置**

该接口可以查看HTTP 块下所有server的证书配置情况

请求

```
HTTP
GET http://127.0.0.1:8081/ssl
```

返回值

```
HTTP
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Sat, 06 May 2023 06:52:52 GMT
< Content-Type: application/json
< Content-Length: 674
< Connection: keep-alive

{
    "servers": [
        {
            "listens": [
                "0.0.0.0:92"
            ],
            "serverNames": [
                ""
            ]
        },
        {
            "listens": [
                "0.0.0.0:443"
            ],
            "serverNames": [
                ""
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "certs/server.crt",
                    "certificateKey": "certs/server.key"
                }
            ]
        },
        {
            "listens": [
                "0.0.0.0:90"
            ],
            "serverNames": [
                "localhost"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "/root/bug/njet1.0/ssl/client.pem",
                    "certificateKey": "/root/bug/njet1.0/ssl/client.key"
                }
            ]
        }
    ]
}
```

###### 3.20.3.8.5.2 新增**http server ssl**当前证书 

更新国密证书

前提需要静态配置文件配置指令，ssl_ntls   on;

```
Bash
PUT  http://127.0.0.1:8081/ssl
Content-Type: application/json

{
    "listens": [
        "0.0.0.0:443"
    ],
    "serverNames": [
        "aaaa"
    ],
    "type": "add",
    "cert_info": {
        "cert_type": "ntls",
        "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+GMAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABOgF2jiXSu+sWssR6wDiiozJXTep2gZgDHdkYMn8JRYu3r/JnD79hdaj\r\nys4KMyYecZ7vxx7Za8XDCHqLtMCYUaejgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQrE10lVWLeOLx0wUJ0qvw9DAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgDAMAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANMXEYW4FZrKpQxedELM7xn461/lHePexG7U7qRJeTdjAiEAuISM\r\nWZunhb0P1gUnMoSjBwKLIr2f7YpmO7xxUO63qdU=\r\n-----END CERTIFICATE-----\r\n",
        "certificateKey": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQgrGLqbyNS5aTuc7Y6\r\nlH3lspDv+ipwL+JEVw6eJnp3CPSgCgYIKoEcz1UBgi2hRANCAAToBdo4l0rvrFrL\r\nEesA4oqMyV03qdoGYAx3ZGDJ/CUWLt6/yZw+/YXWo8rOCjMmHnGe78ce2WvFwwh6\r\ni7TAmFGn\r\n-----END PRIVATE KEY-----\r\n",
        "certificateEnc": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+5MAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABJ434ZA5OJuDfYmPhGuYqcEnth0qwTrr0Nr8/e81X10F+AwBCwt/7iI/\r\nVr8Wus3KpmidSryPfQUrtU2QaY600tOjgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQpWdTiQuwA1LLaCKQHutzKjAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgA4MAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANqpcb/4ZIUVTePBNvJDHb6OPyl3Elr6Z9Cig5DQ97EaAiEA58Lx\r\nLM7OuX/4nxSS+n8LyxToV6uRGOdQKPyAsaOhTHU=\r\n-----END CERTIFICATE-----\r\n",
        "certificateKeyEnc": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQg2zdpRZvEtA5XRXbc\r\nHc/AI3DFLwFAF6s0mUysPE2ZSIOgCgYIKoEcz1UBgi2hRANCAASeN+GQOTibg32J\r\nj4RrmKnBJ7YdKsE669Da/P3vNV9dBfgMAQsLf+4iP1a/FrrNyqZonUq8j30FK7VN\r\nkGmOtNLT\r\n-----END PRIVATE KEY-----\r\n"
    }
}
```

使用支持国密的gmcurl 访问

```
Bash
[root@localhost njet1.0]# ./gmcurl --gmssl https://aaaa:443/ -kv
GM Version: 1.0.1 Ported by www.gmssl.cn
GM options:
--gmssl, use TLCP protocol
--cert,  use sm2 sig pem cert
--key,   use sm2 sig pem key
--cert2, use sm2 enc pem cert
--key2,  use sm2 enc pem key
*   Trying 127.0.0.1:443...
* Connected to aaaa (127.0.0.1) port 443 (#0)
* ALPN, offering http/1.1
* (101) (OUT), , Unknown (1):
* (101) (IN), , Unknown (2):
* (101) (IN), , Unknown (11):
* (101) (IN), , Unknown (12):
* (101) (IN), , Unknown (14):
* (101) (OUT), , Unknown (16):
* (101) (OUT), , Change cipher spec (1):
* (101) (OUT), , Unknown (20):
* (101) (IN), , Unknown (20):
* SSL connection using GMSSLv1.1 / ECC-SM4-GCM-SM3
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=CN; ST=Beijing; L=Beijing1; O=aa; OU=aaa; CN=aaaa; emailAddress=aa@aa.com
*  start date: Jan 30 16:00:00 2023 GMT
*  expire date: Jan 30 16:00:00 2024 GMT
*  issuer: C=CN; O=GMSSL; OU=PKI/SM2; CN=MiddleCA for Test
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
> GET / HTTP/1.1
> Host: aaaa
> User-Agent: curl/7.82.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Wed, 31 May 2023 03:27:24 GMT
< Content-Type: text/plain
< Content-Length: 88
< Connection: keep-alive
< Content-Type: text/html;charset=utf-8
< 
* Connection #0 to host aaaa left intact
njet ntls test OK, ssl_protocol is NTLSv1.1 (NTLSv1.1 表示国密，其他表示国际)
```

get查询：

```
JSON
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:92"
            ],
            "serverNames": [
                ""
            ]
        },
        {
            "listens": [
                "0.0.0.0:443"
            ],
            "serverNames": [
                "aaaa"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "certs/test_rsa.crt",
                    "certificateKey": "certs/test_rsa.key"
                },
                {
                    "cert_type": "regular",
                    "certificate": "certs/test_rsa.crt",
                    "certificateKey": "certs/test_rsa.key"
                },
                {
                    "cert_type": "regular",
                    "certificate": "certs/server.crt",
                    "certificateKey": "certs/server.key"
                },
                {
                    "cert_type": "ntls",
                    "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIICJjCCAcygAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAcwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYYxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRowGAYDVQQDDBFzZXJ2\r\nZXIgc2lnbiAoU00yKTBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABKWvwarEtpHK\r\nckfCaWTcqhzO8e3z02Q2+NOyUr94WgvuQFuJjdLMoHCDxDuMrhXaxUXhXE/rZsG7\r\nZ4XWyd7K/n+jGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgbAMAoGCCqBHM9VAYN1\r\nA0gAMEUCIQDXyNeZN0iqy2cxDYytWtWLayYdSmdnuPg8VOhXUWWb+QIgEwj6Y2xg\r\n38xlatqML8aO89O4movtgOxPZG2YYTaDon8=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKey": "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIPunrzGM/F+w/64DaLg5RLn2ZUoqYEzwsgxUkeKWF6jmoAoGCCqBHM9V\r\nAYItoUQDQgAEpa/BqsS2kcpyR8JpZNyqHM7x7fPTZDb407JSv3haC+5AW4mN0syg\r\ncIPEO4yuFdrFReFcT+tmwbtnhdbJ3sr+fw==\r\n-----END EC PRIVATE KEY-----\r\n",
                    "certificateEnc": "data:-----BEGIN CERTIFICATE-----\r\nMIICJTCCAcugAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAgwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYUxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRkwFwYDVQQDDBBzZXJ2\r\nZXIgZW5jIChTTTIpMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEdGUnMh317JO6\r\nePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaAwMC16wze1n3JjSIqIMf7ielE\r\nePXi4G1dsKMaMBgwCQYDVR0TBAIwADALBgNVHQ8EBAMCAzgwCgYIKoEcz1UBg3UD\r\nSAAwRQIhANC8B9M4Mo0GDWDgSeJ5M4wlF0QgWMlY29GYB1/bGxq6AiAb4IlVTwnZ\r\nV0/PP3zuNjJjp7blkvsaCGsXtb9pk2rQYw==\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKeyEnc": "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIJH/qG2bKhybB1uUSnQbKiNueeLneQVioS+0kLMygYCVoAoGCCqBHM9V\r\nAYItoUQDQgAEdGUnMh317JO6ePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaA\r\nwMC16wze1n3JjSIqIMf7ielEePXi4G1dsA==\r\n-----END EC PRIVATE KEY-----\r\n"
                }
            ]
        },
        {
            "listens": [
                "0.0.0.0:90"
            ],
            "serverNames": [
                "localhost"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "/root/bug/njet1.0/ssl/client.pem",
                    "certificateKey": "/root/bug/njet1.0/ssl/client.key"
                }
            ]
        }
    ]
}
```

更新普通证书

```
HTTP
PUT http://127.0.0.1:8081/ssl
Content-Type: application/json

{
    "listens": [
        "0.0.0.0:443"
    ],
    "serverNames": [
        "aaaa"
    ],
    "type": "add",
    "cert_info": {
         "cert_type": "regular",
         "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIIEITCCAwmgAwIBAgIUK6l+HK4KR/hLosx2XqNTaHLUkO0wDQYJKoZIhvcNAQEL\r\nBQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl\r\naUppbmcxDjAMBgNVBAoTBW5naW54MQ8wDQYDVQQLEwZ0bWxha2UxETAPBgNVBAMT\r\nCG5naW54LWNhMCAXDTIyMDgwOTA5NDEwMFoYDzIxMjIwNzE2MDk0MTAwWjBiMQsw\r\nCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpSmluZzEQMA4GA1UEBxMHQmVpSmluZzEO\r\nMAwGA1UEChMFbmdpbngxDzANBgNVBAsTBnRtbGFrZTEOMAwGA1UEAxMFbmdpbngw\r\nggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC0hs5E8ZZT+ZU/DvvIRrSL\r\nokk6OE5EA4URo6zawqguKRlTc1USgfMdZFwR2znCxjFcsk7W/FSWqA5k5zyePYL8\r\n/0lhn3i/GQGczfa9dMZZEtN9EMD6TPOiDI/CaWAFc3U7qE+SYckUHLu/lMX8LshR\r\ndNkGecq41Ck5Sftm3xwy4cmpcCIyMpNumHrjJyxkp3S/QGewgZRQypntG9R34EKj\r\nQNFnovsn39TeeG+DBjS5m1+roth0JJ6DbemB47hy/uaEwAQpobN1etx20wCwxRO7\r\n7Huuh4kmWXjUol0nvzXM3ZZX2QnHn4jIg/ALS7O6p86z+m4vEHK4hRfWf52xGjMv\r\nAgMBAAGjgckwgcYwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMB\r\nBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBQz6z5AIFRrP+KD1gBt\r\nW6j5NEMzwDAfBgNVHSMEGDAWgBQ64+C++PsR6pDZoQ7EORozSJ8d9DBHBgNVHREE\r\nQDA+ggkxOTIuMTY4LiqCDiouY2x1c3RlcjEuY29tgg0qLmNsdXN0ZXIuY29tggwq\r\nLnRtbGFrZS5jb22HBH8AAAEwDQYJKoZIhvcNAQELBQADggEBADouKA9fFflNAs2u\r\nu7yj4cGNG+sC8r1wDWyGiXGDzvaB7C/sPTZiCgh8hI5gxzOSC+Se51kNFYymN9g4\r\nBPfB//SZ+CxO60BOt5eFF4PxIW7+d8gGFJr7Yx55QGA9DrvGi/LEiwjV3bQaezjN\r\noXUkfFgyKv79AfNFShkUPiHCklnfyqcCF+GEKxNC1q4wyClUt2EGtwfiO4YR85s0\r\n7jQ2BKJOsw3LLd8pZSvOXYu/I9DoO5w3gM5TgjnJlQPq+bmgqJ7bfWbSpFo70cdF\r\nFLfE9MLpwhDjVQEJ6DOmZqU1dVdNtwLrUrm0/8xkkmac0/8BaEYyPy1SNMGJTMGE\r\nW8psxto=\r\n-----END CERTIFICATE-----\r\n",
         "certificateKey": "data:-----BEGIN RSA PRIVATE KEY-----\r\nMIIEogIBAAKCAQEAtIbORPGWU/mVPw77yEa0i6JJOjhORAOFEaOs2sKoLikZU3NV\r\nEoHzHWRcEds5wsYxXLJO1vxUlqgOZOc8nj2C/P9JYZ94vxkBnM32vXTGWRLTfRDA\r\n+kzzogyPwmlgBXN1O6hPkmHJFBy7v5TF/C7IUXTZBnnKuNQpOUn7Zt8cMuHJqXAi\r\nMjKTbph64ycsZKd0v0BnsIGUUMqZ7RvUd+BCo0DRZ6L7J9/U3nhvgwY0uZtfq6LY\r\ndCSeg23pgeO4cv7mhMAEKaGzdXrcdtMAsMUTu+x7roeJJll41KJdJ781zN2WV9kJ\r\nx5+IyIPwC0uzuqfOs/puLxByuIUX1n+dsRozLwIDAQABAoIBAFh+VJLbUnOrvwuA\r\nTtBoSIzCat8NRuB0UUDKWSuLjGHEZ9POj39ZEFHyJmfibTgba4sjJR6h5t1LWHMC\r\nH2b6hEF86v3d7JTQr0esdy18FtcHMYD3O4H3Qt7HBZmpihZh+K/b29XH9YfUZfyN\r\n81ehnzS+8LwJ6+QarHKW35QX/ny5+pCw+G97cjdE7sClZnryd1KseWubAXDLSgmQ\r\nJcjXMMkwjtd2x/w23ZT97WMEcToL1wMw4hF/tHtu9u3UHMhzLHgrfrkW4QmACbLF\r\n8MJR94wzscJrUEviz5fV+3ZTmK+lujGmO/KkOpNAPGf8QAeBfa/qyBTQCQgCdxRo\r\n8hrDT4ECgYEA5vLsDDWiAROvvPaRkCv0y0UWkdgyBxv2T6pjDJZzMxb0yKZ48E6l\r\nh0PI9bfn0KbI1ZZviiZg20oZ/Keo2wLYa+W00rZhWljWUh0Ivd/5vTPHBmpsXl6+\r\nRK49aNVkfqlleherhfrYUkHKKDeg4lKVY3fwAlAQeg/y4Ds/8k3HUuECgYEAyBu8\r\ns7KuO7sAaJVj+QFfRrR6u2eJD48HPB4UoeDYMaKAuPTiM5/AUjH/YdD4E2lGAyJJ\r\nJVN413yu3Dk9UIAivgqCROepAcnnZ1BwE2qI/gfr2GD95o8JQSphze0pC/6WS/jG\r\niBR+j7MGWIkHJHEIUjOrf8eqtVLAywjxz9rPWA8CgYBTkuzgrjfl893Qn9mlNoLr\r\nXCECvh28fN3xjlMxpvAhONl0EuoI7CzyehEq+lYlJ3Xd9QaAE8tRD8u/plxwhOMU\r\niJea+OzZ6PQF2wPi0j5pvWb0Z2a378kiyXrniPFI9LwIJrCnV1MY0T36t8a8n+33\r\nhNuRuq97vHHDuy003fiXgQKBgGKzM6cauc+iU/hBvzbBk4HnYSXwUm1HKdVgLOMP\r\naPNKaN1RhATchdrE6GcR0FqasTq4fYWYn2ECEalz3idHnFtKCaj87qKAOM//n9gj\r\n0wAhXhWy+WjwIitvQSB2Gqnc37sHML1MBoTQU4/1vn0d93G8JJn5HN0kvQ0oE0Vn\r\ncp/HAoGAGCxpM24rHBK0HmlABZd8ZfLW/sW4Y7bMsQmrOVuPZcTWcO5HzkkiGzTc\r\n8DCDDeGdfJUC9NaM0DnVuMwp8/f5uZv4dTS7Hzf8Ie5vR9oQx/z73iakiKUIpDQc\r\n0/sUJSKijOQRT1xZQmkw5+QClmbjLLS3ZXn9kMt0bbxZRPkcV2A=\r\n-----END RSA PRIVATE KEY-----\r\n"
    }
}
```

查询更新结果

```
HTTP
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:92"
            ],
            "serverNames": [
                ""
            ]
        },
        {
            "listens": [
                "0.0.0.0:443"
            ],
            "serverNames": [
                "aaaa"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "certs/test_rsa.crt",
                    "certificateKey": "certs/test_rsa.key"
                },
                {
                    "cert_type": "regular",
                    "certificate": "certs/test_rsa.crt",
                    "certificateKey": "certs/test_rsa.key"
                },
                {
                    "cert_type": "regular",
                    "certificate": "certs/server.crt",
                    "certificateKey": "certs/server.key"
                },
                {
                    "cert_type": "ntls",
                    "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+GMAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABOgF2jiXSu+sWssR6wDiiozJXTep2gZgDHdkYMn8JRYu3r/JnD79hdaj\r\nys4KMyYecZ7vxx7Za8XDCHqLtMCYUaejgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQrE10lVWLeOLx0wUJ0qvw9DAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgDAMAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANMXEYW4FZrKpQxedELM7xn461/lHePexG7U7qRJeTdjAiEAuISM\r\nWZunhb0P1gUnMoSjBwKLIr2f7YpmO7xxUO63qdU=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKey": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQgrGLqbyNS5aTuc7Y6\r\nlH3lspDv+ipwL+JEVw6eJnp3CPSgCgYIKoEcz1UBgi2hRANCAAToBdo4l0rvrFrL\r\nEesA4oqMyV03qdoGYAx3ZGDJ/CUWLt6/yZw+/YXWo8rOCjMmHnGe78ce2WvFwwh6\r\ni7TAmFGn\r\n-----END PRIVATE KEY-----\r\n",
                    "certificateEnc": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+5MAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABJ434ZA5OJuDfYmPhGuYqcEnth0qwTrr0Nr8/e81X10F+AwBCwt/7iI/\r\nVr8Wus3KpmidSryPfQUrtU2QaY600tOjgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQpWdTiQuwA1LLaCKQHutzKjAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgA4MAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANqpcb/4ZIUVTePBNvJDHb6OPyl3Elr6Z9Cig5DQ97EaAiEA58Lx\r\nLM7OuX/4nxSS+n8LyxToV6uRGOdQKPyAsaOhTHU=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKeyEnc": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQg2zdpRZvEtA5XRXbc\r\nHc/AI3DFLwFAF6s0mUysPE2ZSIOgCgYIKoEcz1UBgi2hRANCAASeN+GQOTibg32J\r\nj4RrmKnBJ7YdKsE669Da/P3vNV9dBfgMAQsLf+4iP1a/FrrNyqZonUq8j30FK7VN\r\nkGmOtNLT\r\n-----END PRIVATE KEY-----\r\n"
                },
                {
                    "cert_type": "regular",
                    "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIIEITCCAwmgAwIBAgIUK6l+HK4KR/hLosx2XqNTaHLUkO0wDQYJKoZIhvcNAQEL\r\nBQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl\r\naUppbmcxDjAMBgNVBAoTBW5naW54MQ8wDQYDVQQLEwZ0bWxha2UxETAPBgNVBAMT\r\nCG5naW54LWNhMCAXDTIyMDgwOTA5NDEwMFoYDzIxMjIwNzE2MDk0MTAwWjBiMQsw\r\nCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpSmluZzEQMA4GA1UEBxMHQmVpSmluZzEO\r\nMAwGA1UEChMFbmdpbngxDzANBgNVBAsTBnRtbGFrZTEOMAwGA1UEAxMFbmdpbngw\r\nggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC0hs5E8ZZT+ZU/DvvIRrSL\r\nokk6OE5EA4URo6zawqguKRlTc1USgfMdZFwR2znCxjFcsk7W/FSWqA5k5zyePYL8\r\n/0lhn3i/GQGczfa9dMZZEtN9EMD6TPOiDI/CaWAFc3U7qE+SYckUHLu/lMX8LshR\r\ndNkGecq41Ck5Sftm3xwy4cmpcCIyMpNumHrjJyxkp3S/QGewgZRQypntG9R34EKj\r\nQNFnovsn39TeeG+DBjS5m1+roth0JJ6DbemB47hy/uaEwAQpobN1etx20wCwxRO7\r\n7Huuh4kmWXjUol0nvzXM3ZZX2QnHn4jIg/ALS7O6p86z+m4vEHK4hRfWf52xGjMv\r\nAgMBAAGjgckwgcYwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMB\r\nBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBQz6z5AIFRrP+KD1gBt\r\nW6j5NEMzwDAfBgNVHSMEGDAWgBQ64+C++PsR6pDZoQ7EORozSJ8d9DBHBgNVHREE\r\nQDA+ggkxOTIuMTY4LiqCDiouY2x1c3RlcjEuY29tgg0qLmNsdXN0ZXIuY29tggwq\r\nLnRtbGFrZS5jb22HBH8AAAEwDQYJKoZIhvcNAQELBQADggEBADouKA9fFflNAs2u\r\nu7yj4cGNG+sC8r1wDWyGiXGDzvaB7C/sPTZiCgh8hI5gxzOSC+Se51kNFYymN9g4\r\nBPfB//SZ+CxO60BOt5eFF4PxIW7+d8gGFJr7Yx55QGA9DrvGi/LEiwjV3bQaezjN\r\noXUkfFgyKv79AfNFShkUPiHCklnfyqcCF+GEKxNC1q4wyClUt2EGtwfiO4YR85s0\r\n7jQ2BKJOsw3LLd8pZSvOXYu/I9DoO5w3gM5TgjnJlQPq+bmgqJ7bfWbSpFo70cdF\r\nFLfE9MLpwhDjVQEJ6DOmZqU1dVdNtwLrUrm0/8xkkmac0/8BaEYyPy1SNMGJTMGE\r\nW8psxto=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKey": "data:-----BEGIN RSA PRIVATE KEY-----\r\nMIIEogIBAAKCAQEAtIbORPGWU/mVPw77yEa0i6JJOjhORAOFEaOs2sKoLikZU3NV\r\nEoHzHWRcEds5wsYxXLJO1vxUlqgOZOc8nj2C/P9JYZ94vxkBnM32vXTGWRLTfRDA\r\n+kzzogyPwmlgBXN1O6hPkmHJFBy7v5TF/C7IUXTZBnnKuNQpOUn7Zt8cMuHJqXAi\r\nMjKTbph64ycsZKd0v0BnsIGUUMqZ7RvUd+BCo0DRZ6L7J9/U3nhvgwY0uZtfq6LY\r\ndCSeg23pgeO4cv7mhMAEKaGzdXrcdtMAsMUTu+x7roeJJll41KJdJ781zN2WV9kJ\r\nx5+IyIPwC0uzuqfOs/puLxByuIUX1n+dsRozLwIDAQABAoIBAFh+VJLbUnOrvwuA\r\nTtBoSIzCat8NRuB0UUDKWSuLjGHEZ9POj39ZEFHyJmfibTgba4sjJR6h5t1LWHMC\r\nH2b6hEF86v3d7JTQr0esdy18FtcHMYD3O4H3Qt7HBZmpihZh+K/b29XH9YfUZfyN\r\n81ehnzS+8LwJ6+QarHKW35QX/ny5+pCw+G97cjdE7sClZnryd1KseWubAXDLSgmQ\r\nJcjXMMkwjtd2x/w23ZT97WMEcToL1wMw4hF/tHtu9u3UHMhzLHgrfrkW4QmACbLF\r\n8MJR94wzscJrUEviz5fV+3ZTmK+lujGmO/KkOpNAPGf8QAeBfa/qyBTQCQgCdxRo\r\n8hrDT4ECgYEA5vLsDDWiAROvvPaRkCv0y0UWkdgyBxv2T6pjDJZzMxb0yKZ48E6l\r\nh0PI9bfn0KbI1ZZviiZg20oZ/Keo2wLYa+W00rZhWljWUh0Ivd/5vTPHBmpsXl6+\r\nRK49aNVkfqlleherhfrYUkHKKDeg4lKVY3fwAlAQeg/y4Ds/8k3HUuECgYEAyBu8\r\ns7KuO7sAaJVj+QFfRrR6u2eJD48HPB4UoeDYMaKAuPTiM5/AUjH/YdD4E2lGAyJJ\r\nJVN413yu3Dk9UIAivgqCROepAcnnZ1BwE2qI/gfr2GD95o8JQSphze0pC/6WS/jG\r\niBR+j7MGWIkHJHEIUjOrf8eqtVLAywjxz9rPWA8CgYBTkuzgrjfl893Qn9mlNoLr\r\nXCECvh28fN3xjlMxpvAhONl0EuoI7CzyehEq+lYlJ3Xd9QaAE8tRD8u/plxwhOMU\r\niJea+OzZ6PQF2wPi0j5pvWb0Z2a378kiyXrniPFI9LwIJrCnV1MY0T36t8a8n+33\r\nhNuRuq97vHHDuy003fiXgQKBgGKzM6cauc+iU/hBvzbBk4HnYSXwUm1HKdVgLOMP\r\naPNKaN1RhATchdrE6GcR0FqasTq4fYWYn2ECEalz3idHnFtKCaj87qKAOM//n9gj\r\n0wAhXhWy+WjwIitvQSB2Gqnc37sHML1MBoTQU4/1vn0d93G8JJn5HN0kvQ0oE0Vn\r\ncp/HAoGAGCxpM24rHBK0HmlABZd8ZfLW/sW4Y7bMsQmrOVuPZcTWcO5HzkkiGzTc\r\n8DCDDeGdfJUC9NaM0DnVuMwp8/f5uZv4dTS7Hzf8Ie5vR9oQx/z73iakiKUIpDQc\r\n0/sUJSKijOQRT1xZQmkw5+QClmbjLLS3ZXn9kMt0bbxZRPkcV2A=\r\n-----END RSA PRIVATE KEY-----\r\n"
                }
            ]
        },
        {
            "listens": [
                "0.0.0.0:90"
            ],
            "serverNames": [
                "localhost"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "/root/bug/njet1.0/ssl/client.pem",
                    "certificateKey": "/root/bug/njet1.0/ssl/client.key"
                }
            ]
        }
    ]
}
```

查看证书

```
Bash
[root@localhost njet1.0]# curl https://localhost:443/ -kv
*   Trying 127.0.0.1:443...
* Connected to localhost (127.0.0.1) port 443 (#0)
* ALPN: offers http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN: server accepted http/1.1
* Server certificate:
*  subject: C=CN; ST=BeiJing; L=BeiJing; O=nginx; OU=tmlake; CN=nginx
*  start date: Aug  9 09:41:00 2022 GMT
*  expire date: Jul 16 09:41:00 2122 GMT
*  issuer: C=CN; ST=BeiJing; L=BeiJing; O=nginx; OU=tmlake; CN=nginx-ca
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* using HTTP/1.1
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.29.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Wed, 31 May 2023 03:36:41 GMT
< Content-Type: text/plain
< Content-Length: 87
< Connection: keep-alive
< Content-Type: text/html;charset=utf-8
< 
* Connection #0 to host localhost left intact
njet ntls test OK, ssl_protocol is TLSv1.2 (NTLSv1.1 表示国密，其他表示国际)
```

###### 3.20.3.8.5.3 删除http server ssl证书

删除只会在reload后生效并且有效果

```
JSON
PUT http://127.0.0.1:8081/ssl
Content-Type: application/json

{
    "listens": [
        "0.0.0.0:443"
    ],
    "serverNames": [
        "aaaa"
    ],
    "type": "del",
    "cert_info": {
         "cert_type": "regular",
         "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIIEITCCAwmgAwIBAgIUK6l+HK4KR/hLosx2XqNTaHLUkO0wDQYJKoZIhvcNAQEL\r\nBQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl\r\naUppbmcxDjAMBgNVBAoTBW5naW54MQ8wDQYDVQQLEwZ0bWxha2UxETAPBgNVBAMT\r\nCG5naW54LWNhMCAXDTIyMDgwOTA5NDEwMFoYDzIxMjIwNzE2MDk0MTAwWjBiMQsw\r\nCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpSmluZzEQMA4GA1UEBxMHQmVpSmluZzEO\r\nMAwGA1UEChMFbmdpbngxDzANBgNVBAsTBnRtbGFrZTEOMAwGA1UEAxMFbmdpbngw\r\nggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC0hs5E8ZZT+ZU/DvvIRrSL\r\nokk6OE5EA4URo6zawqguKRlTc1USgfMdZFwR2znCxjFcsk7W/FSWqA5k5zyePYL8\r\n/0lhn3i/GQGczfa9dMZZEtN9EMD6TPOiDI/CaWAFc3U7qE+SYckUHLu/lMX8LshR\r\ndNkGecq41Ck5Sftm3xwy4cmpcCIyMpNumHrjJyxkp3S/QGewgZRQypntG9R34EKj\r\nQNFnovsn39TeeG+DBjS5m1+roth0JJ6DbemB47hy/uaEwAQpobN1etx20wCwxRO7\r\n7Huuh4kmWXjUol0nvzXM3ZZX2QnHn4jIg/ALS7O6p86z+m4vEHK4hRfWf52xGjMv\r\nAgMBAAGjgckwgcYwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMB\r\nBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBQz6z5AIFRrP+KD1gBt\r\nW6j5NEMzwDAfBgNVHSMEGDAWgBQ64+C++PsR6pDZoQ7EORozSJ8d9DBHBgNVHREE\r\nQDA+ggkxOTIuMTY4LiqCDiouY2x1c3RlcjEuY29tgg0qLmNsdXN0ZXIuY29tggwq\r\nLnRtbGFrZS5jb22HBH8AAAEwDQYJKoZIhvcNAQELBQADggEBADouKA9fFflNAs2u\r\nu7yj4cGNG+sC8r1wDWyGiXGDzvaB7C/sPTZiCgh8hI5gxzOSC+Se51kNFYymN9g4\r\nBPfB//SZ+CxO60BOt5eFF4PxIW7+d8gGFJr7Yx55QGA9DrvGi/LEiwjV3bQaezjN\r\noXUkfFgyKv79AfNFShkUPiHCklnfyqcCF+GEKxNC1q4wyClUt2EGtwfiO4YR85s0\r\n7jQ2BKJOsw3LLd8pZSvOXYu/I9DoO5w3gM5TgjnJlQPq+bmgqJ7bfWbSpFo70cdF\r\nFLfE9MLpwhDjVQEJ6DOmZqU1dVdNtwLrUrm0/8xkkmac0/8BaEYyPy1SNMGJTMGE\r\nW8psxto=\r\n-----END CERTIFICATE-----\r\n",
         "certificateKey": "data:-----BEGIN RSA PRIVATE KEY-----\r\nMIIEogIBAAKCAQEAtIbORPGWU/mVPw77yEa0i6JJOjhORAOFEaOs2sKoLikZU3NV\r\nEoHzHWRcEds5wsYxXLJO1vxUlqgOZOc8nj2C/P9JYZ94vxkBnM32vXTGWRLTfRDA\r\n+kzzogyPwmlgBXN1O6hPkmHJFBy7v5TF/C7IUXTZBnnKuNQpOUn7Zt8cMuHJqXAi\r\nMjKTbph64ycsZKd0v0BnsIGUUMqZ7RvUd+BCo0DRZ6L7J9/U3nhvgwY0uZtfq6LY\r\ndCSeg23pgeO4cv7mhMAEKaGzdXrcdtMAsMUTu+x7roeJJll41KJdJ781zN2WV9kJ\r\nx5+IyIPwC0uzuqfOs/puLxByuIUX1n+dsRozLwIDAQABAoIBAFh+VJLbUnOrvwuA\r\nTtBoSIzCat8NRuB0UUDKWSuLjGHEZ9POj39ZEFHyJmfibTgba4sjJR6h5t1LWHMC\r\nH2b6hEF86v3d7JTQr0esdy18FtcHMYD3O4H3Qt7HBZmpihZh+K/b29XH9YfUZfyN\r\n81ehnzS+8LwJ6+QarHKW35QX/ny5+pCw+G97cjdE7sClZnryd1KseWubAXDLSgmQ\r\nJcjXMMkwjtd2x/w23ZT97WMEcToL1wMw4hF/tHtu9u3UHMhzLHgrfrkW4QmACbLF\r\n8MJR94wzscJrUEviz5fV+3ZTmK+lujGmO/KkOpNAPGf8QAeBfa/qyBTQCQgCdxRo\r\n8hrDT4ECgYEA5vLsDDWiAROvvPaRkCv0y0UWkdgyBxv2T6pjDJZzMxb0yKZ48E6l\r\nh0PI9bfn0KbI1ZZviiZg20oZ/Keo2wLYa+W00rZhWljWUh0Ivd/5vTPHBmpsXl6+\r\nRK49aNVkfqlleherhfrYUkHKKDeg4lKVY3fwAlAQeg/y4Ds/8k3HUuECgYEAyBu8\r\ns7KuO7sAaJVj+QFfRrR6u2eJD48HPB4UoeDYMaKAuPTiM5/AUjH/YdD4E2lGAyJJ\r\nJVN413yu3Dk9UIAivgqCROepAcnnZ1BwE2qI/gfr2GD95o8JQSphze0pC/6WS/jG\r\niBR+j7MGWIkHJHEIUjOrf8eqtVLAywjxz9rPWA8CgYBTkuzgrjfl893Qn9mlNoLr\r\nXCECvh28fN3xjlMxpvAhONl0EuoI7CzyehEq+lYlJ3Xd9QaAE8tRD8u/plxwhOMU\r\niJea+OzZ6PQF2wPi0j5pvWb0Z2a378kiyXrniPFI9LwIJrCnV1MY0T36t8a8n+33\r\nhNuRuq97vHHDuy003fiXgQKBgGKzM6cauc+iU/hBvzbBk4HnYSXwUm1HKdVgLOMP\r\naPNKaN1RhATchdrE6GcR0FqasTq4fYWYn2ECEalz3idHnFtKCaj87qKAOM//n9gj\r\n0wAhXhWy+WjwIitvQSB2Gqnc37sHML1MBoTQU4/1vn0d93G8JJn5HN0kvQ0oE0Vn\r\ncp/HAoGAGCxpM24rHBK0HmlABZd8ZfLW/sW4Y7bMsQmrOVuPZcTWcO5HzkkiGzTc\r\n8DCDDeGdfJUC9NaM0DnVuMwp8/f5uZv4dTS7Hzf8Ie5vR9oQx/z73iakiKUIpDQc\r\n0/sUJSKijOQRT1xZQmkw5+QClmbjLLS3ZXn9kMt0bbxZRPkcV2A=\r\n-----END RSA PRIVATE KEY-----\r\n"
    }
}
```

#### 3.20.3.9 **动态limit**

##### 3.20.3.9.1 **依赖模块**

```
Bash
njt_http_dyn_limit_module.so
```

##### 3.20.3.9.2 **配置指定**

无

##### 3.20.3.9.3 **配置示例**

```
Bash
load_module modules/njt_http_dyn_limit_module.so;
 
   
http {    
    limit_conn_zone $binary_remote_addr zone=perip:10m;
    limit_conn_zone $server_name zone=perserver:10m;

    limit_req_zone $binary_remote_addr zone=req_perip:10m rate=1r/s;
    limit_req_zone $server_name zone=req_perserver:10m rate=10r/s;


    server {
        listen       92;

        location / {
            limit_rate 1k;
            limit_rate_after 5k;

            set $aaa 50k;

            limit_conn perip 10;
            limit_conn perserver 100;

            limit_req zone=req_perip burst=5 ;

            root html;
            index large.html;
        }
   }
}
```

##### 3.20.3.9.4 **返回值说明**

put接口使用rpc调用，返回值新增返回值code，以及错误描述信息，详细信息如下：

![img](https://gitee.com/gebona/picture/raw/master/202307101119683.jpg)

​                                                                                                        **点击图片可查看完整电子表格**

示例如下：

```
JSON
{
  "code": 1,            #错误码
  "msg": "partial success.",       #错误码描述信息
  "data": [                        #详细错误列表
    " update rps zone:req_perip  rate invalid",
    " update rps zone:req_perserver2  is not exist",
    "listen[0.0.0.0:92] server_name[].locations[/] zone:perip conn number is invalid, should >0 and <= 65535",
    "listen[0.0.0.0:92] server_name[].locations[/] zone \"perserver2\" is not exist",
    "listen[0.0.0.0:92] server_name[].locations[/] dyn limit req not location level, so not update",
    "listen[0.0.0.0:92] server_name[].locations[/] dyn limit rate complex set error",
    "listen[0.0.0.0:92] server_name[].locations[/] dyn limit rate_after set complex errorc",
    "listen[0.0.0.0:92] server_name[].locations[/] dyn limit conn_log_level level invalid",
    "listen[0.0.0.0:92] server_name[].locations[/] dyn limit conn_status invalid, shoudld [400, 599]",
    "listen[0.0.0.0:92] server_name[].locations[/] dyn limit req_status invalid, shoudld [400, 599]",
    "listen[0.0.0.0:92] server_name[].locations[/a] location[/ab] not found",
    " can`t find server by listen[0.0.0.0:444] server_name[]",
    " can`t find server by listen[0.0.0.0:90] server_name[localhost1]"
  ]
}
```

##### 3.20.3.9.5 **接口示例**

###### 3.20.3.9.5.1 **查询**limit 当前配置

该接口可以查看HTTP 块下所有location的limit当前配置

请求

```
HTTP
GET http://127.0.0.1:8081/config/2/config/http_dyn_limit
```

返回值

```
HTTP
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:92"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "limit_rate": "1k",
                    "limit_rate_after": "5k",
                    "limit_conns": [
                        {
                            "zone": "perip",
                            "conn": 10
                        },
                        {
                            "zone": "perserver",
                            "conn": 100
                        }
                    ],
                    "limit_conn_dry_run": "off",
                    "limit_conn_log_level": "error",
                    "limit_conn_status": 503,
                    "limit_reqs": [
                        {
                            "zone": "req_perip",
                            "burst": 5,
                            "delay": "0"
                        }
                    ],
                    "limit_req_dry_run": "off",
                    "limit_req_log_level": "error",
                    "limit_req_status": 503
                }
            ]
        },
        {
            "listens": [
                "0.0.0.0:90"
            ],
            "serverNames": [
                "localhost"
            ],
            "locations": [
                {
                    "location": "/",
                    "limit_rate": "",
                    "limit_rate_after": "",
                    "limit_conns": [],
                    "limit_conn_dry_run": "off",
                    "limit_conn_log_level": "error",
                    "limit_conn_status": 503,
                    "limit_reqs": [],
                    "limit_req_dry_run": "off",
                    "limit_req_log_level": "error",
                    "limit_req_status": 503,
                    "locations": [
                        {
                            "location": "/abc/",
                            "limit_rate": "",
                            "limit_rate_after": "",
                            "limit_conns": [],
                            "limit_conn_dry_run": "off",
                            "limit_conn_log_level": "error",
                            "limit_conn_status": 503,
                            "limit_reqs": [],
                            "limit_req_dry_run": "off",
                            "limit_req_log_level": "error",
                            "limit_req_status": 503
                        }
                    ]
                },
                {
                    "location": "/test_bwlist",
                    "limit_rate": "",
                    "limit_rate_after": "",
                    "limit_conns": [],
                    "limit_conn_dry_run": "off",
                    "limit_conn_log_level": "error",
                    "limit_conn_status": 503,
                    "limit_reqs": [],
                    "limit_req_dry_run": "off",
                    "limit_req_log_level": "error",
                    "limit_req_status": 503
                }
            ]
        }
    ],
         #limit_rps 专门修改limit_req_zone 指定zone name的rate值的动态修改，该指令可配置多个，故是数组形式
    "limit_rps": [
        {
            "zone": "req_perip",     #zone name
            "rate": "1r/m"          #rate值修改
        },
        {
            "zone": "req_perserver",
            "rate": "10r/s"
        }
    ]
}
```

###### 3.20.3.9.5.2 **修改limit **配置

请求

```
HTTP
PUT http://127.0.0.1:8081/config/2/config/http_dyn_list
Content-Type: application/json

{
    "servers": [
        {
            "listens": [
                "0.0.0.0:92"
            ],
            "serverNames": [
                ""
            ],
            "locations": [
                {
                    "location": "/",
                    "limit_rate": "1k",
                    "limit_rate_after": "5k",
                    "limit_conns": [
                        {
                            "zone": "perip",
                            "conn": 10
                        },
                        {
                            "zone": "perserver",
                            "conn": 100
                        }
                    ],
                    "limit_conn_dry_run": "off",
                    "limit_conn_log_level": "error",
                    "limit_conn_status": 503,
                    "limit_reqs": [
                        {
                            "zone": "req_perip",
                            "burst": 5,
                            "delay": "nodelay"
                        },
                        {
                            "zone": "req_perserver",
                            "burst": 10,
                            "delay": "1000"
                        }
                    ],
                    "limit_req_dry_run": "off",
                    "limit_req_log_level": "error",
                    "limit_req_status": 503
                }
            ]
        },
        {
            "listens": [
                "0.0.0.0:90"
            ],
            "serverNames": [
                "localhost"
            ],
            "locations": [
                {
                    "location": "/",
                    "limit_rate": "",
                    "limit_rate_after": "",
                    "limit_conns": [],
                    "limit_conn_dry_run": "off",
                    "limit_conn_log_level": "error",
                    "limit_conn_status": 503,
                    "limit_reqs": [],
                    "limit_req_dry_run": "off",
                    "limit_req_log_level": "error",
                    "limit_req_status": 503,
                    "locations": [
                        {
                            "location": "/abc/",
                            "limit_rate": "",
                            "limit_rate_after": "",
                            "limit_conns": [],
                            "limit_conn_dry_run": "off",
                            "limit_conn_log_level": "error",
                            "limit_conn_status": 503,
                            "limit_reqs": [],
                            "limit_req_dry_run": "off",
                            "limit_req_log_level": "error",
                            "limit_req_status": 503
                        }
                    ]
                },
                {
                    "location": "/test_bwlist",
                    "limit_rate": "",
                    "limit_rate_after": "",
                    "limit_conns": [],
                    "limit_conn_dry_run": "off",
                    "limit_conn_log_level": "error",
                    "limit_conn_status": 503,
                    "limit_reqs": [],
                    "limit_req_dry_run": "off",
                    "limit_req_log_level": "error",
                    "limit_req_status": 503
                }
            ]
        }
    ],
    "limit_rps": [
        {
            "zone": "req_perip",
            "rate": "100r/m"
        },
        {
            "zone": "req_perserver",
            "rate": "20r/s"
        }
    ]
}
```

#### 3.20.3.10 **动态**worker**调整**

##### 3.20.3.10.1 **简介**

NJet 静态配置文件中，设置了 worker_processes 的数目, 设置为auto 将根据系统的CPU 核数自动创建worker 进程。

**！！静态配置只有在NJet首次启动时有效，之后将只能通过动态API进行调整。NJet 首次启动后，将在 data 目录下生成一些数据文件，除非停止NJet后删除该目录下的数据文件，才能恢复初始状态，并且在启动后再次使用静态配置的worker数目进行初始化。建议直接使用动态API进行worker数目调整。**

通过控制面提供的 sendmsg 模块, 可以通过设置key: "__master_worker_count"的值动态调整worker 数目。

设置后，master 进程将根据对应的值进行worker 数目的调整，配置的值需要在这个区间：（0，512], 其它的值，worker数目将不做调整。

##### 3.20.3.10.2 **配置指令**

动态worker调整配置无特殊的指令。

##### 3.20.3.10.3 **配置示例**

```
helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf; 

load_module modules/njt_agent_dynlog_module.so; 
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so;

load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

cluster_name helper;
node_name node1;

worker_processes  1;
error_log logs/error.log info;
pid        logs/njet.pid;
events {
    worker_connections  1024;
}

http {

    dyn_kv_conf conf/iot-work.conf;
    include mime.types;
     
    upstream test{
      zone customer 64k;
      server 192.168.40.249:8090;
    }
   
     server {
      server_name testName;
      listen 7777;
     
       location / {
        proxy_pass http://test;
      }
    }
   
}
```

##### 3.20.3.10.4 **接口示例**

```
curl -X 'POST' \
  'http://127.0.0.1:8081/kv' \
  -d '{
  "key": "__master_worker_count",
  "value":  "2"
}'
```



## 3.21 **边车**

### **3.21.1** **概述**

Sidecar NJet 作为 Istio 架构中的数据面，旨在替换原生数据面 Sidecar envoy。本文档基于 Istio 1.13.3 版本编写，主要针对 Sidecar NJet 对 Istio proxy 功能特性的支持进行了说明。

#### **3.21.1.1 Istio**架构

Istio服务网格在逻辑上分为数据平面和控制平面。

￮    数据平面由一组作为sidecars部署的智能代理(Envoy)组成。这些代理调解和控制微服务之间的所有网络通信。他们还收集并报告所有网格流量的遥测数据。

￮    控制平面对代理进行路由管理和配置。

下图显示了组成每个平面的不同组件:

![image-20230619114848905](https://gitee.com/gebona/picture/raw/master/202306191148309.png)

#### **3.21.1.2 Sidecar NJet**架构

![image-20230619114934619](https://gitee.com/gebona/picture/raw/master/202306191149203.png)

api-gateway有两个主要模块：

￮    xds 客户端
 主要与istiod xds服务端通信，进行xds资源的订阅，基于grpc-go xds实现，grpc支持SotW ADS

￮    NJet控制器

▪   管理NJet进程，包括启动，reload

▪   xds资源类型数据进行解析、处理，生成NJet server、upstream、loction元数据，通过go tmpl技术生成NJet.conf文件

▪   集成lua

### **3.21.2** **镜像构建**

通过git submodule，下载njet_main和lua_module源码，这两个仓库是njet proxy源码仓库。

```
Shell
cd api-gateway
git submodule init
git submodule update 
```

执行成功后，源码被下载到istio-agent/docker/njet_lua_src/

```
cd 到istio-agent/docker/njet_lua_src/目录，目录下为lua源码与njet源码子目录，进入到子目录，切换到对应的分支，确保njet源码是您期望的。
```

```
Shell
cd api-gateway/istio-agent/docker  
```

执行

```
Shell
make docker_api_gateway
```

执行上述命令后，sidecar image开始构建，等待片刻，image会被构建生成。出现如下提示代表成功。

![image-20230619115127552](https://gitee.com/gebona/picture/raw/master/202306191151579.png)

### **3.21.3** **部署**sidecar

使用Sidecar NJet需要定制化部署Istio，自动注入NJet Sidecar。定制化部署Istio需要使用Istio-install仓库中的文件，此仓库是基于Istio-1.13.3 release修改的，需要修改Sidecar镜像名称，修改Sidecar 容器模板。

#### **3.21.3.1** **安装**Istio

拉取Istio-install仓库，修改default-istio.yaml文件values.global.proxy.image字段，设置为NJet的镜像名称，必须为包含“/”。 执行如下命令安装istiod：

```
Shell
./istio-1.13.3/bin/istioctl install --manifests=istio-1.13.3/manifests/  -f default-istio.yaml
```

#### **3.21.3.2**注入**sidecar njet**

##### 3.21.3.2.1**Pod level**

给pod打标贴：sidecar.istio.io/inject: "true"，以productpage为例，修改deploy，增加标贴。

```
Shell
kubectl edit deploy productpage-v1
```

![image-20230619115924667](https://gitee.com/gebona/picture/raw/master/202306191159522.png)

等待一会

![image-20230619115939093](https://gitee.com/gebona/picture/raw/master/202306191159222.png)

##### 3.21.3.2.2**Namespace level**

添加namespace label,以便后续在此ns部署应用时，通知Istio自动注入nginx sidecar，下面为default ns添加标贴。

```
Shell
kubectl label namespace default istio-injection=enabled
```

查看ns标贴

```
Shell
kubectl get ns --show-labels
```

![image-20230619133132696](https://gitee.com/gebona/picture/raw/master/202306191331379.png)

查看default ns pod状态

![image-20230619133144887](https://gitee.com/gebona/picture/raw/master/202306191331406.png)

删除details pod，触发sidecar注入

```
Shell
kubectl delete pod details-v1-5498c86cf5-6kcpw  
```

查看default ns pod状态，details pod注入NJet sidecar

![image-20230619133337371](https://gitee.com/gebona/picture/raw/master/202306191333540.png)

#### **3.21.3.3** **卸载**Istio

官方使用如下命令卸载 Istio ,但是部分cm和sa没有被删除

```
Shell
istioctl x uninstall --purge
```

![23](https://gitee.com/gebona/picture/raw/master/202306191557551.png)

删除Istio-system ns，可以删除istio-system ns下的所有资源

```
Shell
kubectl delete namespace istio-system
```

#### **3.21.3.4** **更新**Sidecar NJet镜像

安装Istio，Sidecar成功注入后，当修改了代码重新构建了不同名称的镜像后，需要替换镜像，Istio 通过ConfigMap去更新Sidecar镜像，操作如下：

```
Shell
[root@node151 ~]# kubectl get cm -n istio-system
NAME                                  DATA   AGE
istio                                 2      121d
istio-ca-root-cert                    1      121d
istio-gateway-deployment-leader       0      121d
istio-gateway-status-leader           0      121d
istio-leader                          0      121d
istio-namespace-controller-election   0      121d
istio-sidecar-injector                2      121d
kube-root-ca.crt                      1      121d
[root@node151 ~]#
```

通过修改名为Istio-sidecar-injector的 ConfigMap资源中的配置项，来修改sidecar镜像。修改的配置项为data.values.proxy.image字段，格式如下：

通过如下指令进行编辑，保存即可。

```
Shell
kubectl edit cm -n istio-system istio-sidecar-injector
```

![image-20230619133800829](https://gitee.com/gebona/picture/raw/master/202306191338627.png)

### **3.21.4** **流量管理**

#### **3.21.4.1** **协议支持**

目前支持代理HTTP1。

<table>
<tbody>
<tr>
<td width="100">
<p>协议类型</p>
</td>
<td width="100">
<p>支持情况</p>
</td>
</tr>
<tr>
<td width="100">
<p>HTTP1</p>
</td>
<td width="100">
<p>支持</p>
</td>
</tr>
<tr>
<td width="100">
<p>HTTP2</p>
</td>
<td width="100">
<p>不支持</p>
</td>
</tr>
<tr>
<td width="100">
<p>HTTPS</p>
</td>
<td width="100">
<p>不支持</p>
</td>
</tr>
<tr>
<td width="100">
<p>TCP</p>
</td>
<td width="100">
<p>不支持</p>
</td>
</tr>
<tr>
<td width="100">
<p>TLS</p>
</td>
<td width="100">
<p>不支持</p>
</td>
</tr>
</tbody>
</table>

#### **3.21.4.2 HTTP**路由

##### 3.21.4.2.1**功能描述**

通过丰富的路由规则对HTTP流量实施细粒度的流量控制。使用路由规则告诉NJet如何将虚拟服务的流量发送到适当的目的地，路由目的地可以是相同服务的版本，也可以是完全不同的服务。路由目的地支持按权重分流。

高级的路由可以实现灰度发布，发送到灰度版本的流量是基于请求的属性或者是随机选择的，按照请求的百分比进行。

Istio Sidecar NJet支持 Istio 控制面 HTTP 路由匹配规则，目前支持的路由匹配规则如下表所述：

<table style="height: 500px; width: 699px;">
<tbody>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>字段名称</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>字段类型</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>Istio njet sidecar支持情况</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>uri</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>支持exact、prefix、regex</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>支持exact、prefix、regex</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>scheme</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>支持exact、prefix、regex</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>支持exact</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>method</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>支持exact、prefix、regex</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>支持exact</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>authority</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>支持exact、prefix、regex</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>不支持</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>headers</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>支持exact、prefix、regex</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>支持exact</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>queryParams</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>exact、regex</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>支持exact、regex</p>
</td>
</tr>
<tr style="height: 64px;">
<td style="height: 64px; width: 169.531px;">
<p>ignoreUriCase</p>
</td>
<td style="height: 64px; width: 299.688px;">
<p>uri大小写忽略flag(bool)，针对于exact、prefix</p>
</td>
<td style="height: 64px; width: 207.781px;">
<p>不支持</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 169.531px;">
<p>withoutHeaders</p>
</td>
<td style="height: 46px; width: 299.688px;">
<p>支持exact、prefix、regex</p>
</td>
<td style="height: 46px; width: 207.781px;">
<p>支持exact</p>
</td>
</tr>
</tbody>
</table>


Istio 控制面HTTP路由匹配接口定义参照:

https://istio.io/v1.13/docs/reference/config/networking/virtual-service/#HTTPMatchRequest

##### **3.21.4.2.2 NJet HTTP路由实现**

NJet 通过map+rewrite+split_clients实现基于请求内容的高级路由，比如基于请求的:header、method、queryParams。

##### 3.21.4.2.3**创建一个基于header的灰度路由**

```
Shell
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - uri:
        exact: /reviews/0
    route:
    - destination:
        host: reviews
        subset: v3
  - match:
    - headers:
        end-user:
          exact: jason
      method:
        exact: get
      uri:
        exact: /reviews/0
    route:
    - destination:
        host: reviews
        subset: v2
```

￮    请求中不携带header->end-user：jason，请求被路由到reviews|v3| upstream，即reviews v3版本，v3会返回红色的星星，在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0
```

```
curl请求服务时，主机域名根据实际情况填写，查看njet VirtualHOST 配置的域名列表，即server 块的server_name指令。本测试中名称如下：
server_name reviews.default.svc.cluster.local reviews.default.svc.cluster.local:9080 reviews reviews:9080 reviews.default.svc reviews.default.svc:9080 reviews.default reviews.default:9080 10.233.213.122 10.233.213.122:9080 ;
```

返回结果如下图所示，可知，与期望一致。

![image-20230619134733330](https://gitee.com/gebona/picture/raw/master/202306191347638.png)

￮    请求中携带header->end-user：jason，请求被路由到reviews|v2| upstream，即reviews v2版本，v2会返回5ke黑色的星星，在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0 -H "end-user:jason"
```

返回结果如下图所示，可知，与期望一致。

##### **3.21.4.2.4分流**

```
Shell
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - method:
        exact: GET
      uri:
        prefix: /productpage
    route:
    - destination:
        host: productpage
        subset: v1
      weight: 80
    - destination:
        host: reviews
        subset: v3
      weight: 20
```

match指明，当请求方法为GET，且uri有/productpage前缀时，请求按权重被路由到productpage|v1|和reviews|v3| upstream。

在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/productpage -v
```

返回结果如下图所示，可知，与期望一致。

![image-20230619134842286](https://gitee.com/gebona/picture/raw/master/202306191348651.png)

由图可知，成功返回productpage页面。

反复执行如下命令，会出现404响应。

```
Shell
curl http://reviews:9080/productpage -v
```

![image-20230619134903154](https://gitee.com/gebona/picture/raw/master/202306191349358.png)

![image-20230619134912134](https://gitee.com/gebona/picture/raw/master/202306191349745.png)

由图可知，请求被路由到reviews v3版本，因为reviews服务没有实现/productpage，所以404属于正常。多次请求中，大部分会返回productpage首页，从而说明按权重分流已生效。

### **3.21.5** **安全**

#### **3.21.5.1** **证书管理**

##### 3.21.5.1.1 功能描述

Istio 使用 X.509 证书安全地为每个工作负载提供强身份。与每个 NJet (Envoy)  proxy一起运行的 Istio agent 与 Istiod 一起工作，实现大规模自动化密钥和证书轮换。下图显示了身份供应流程。

![image-20230619135006849](https://gitee.com/gebona/picture/raw/master/202306191350293.png)

Istio通过以下流程提供密钥和证书:

￮    Istiod提供gRPC服务来接收证书签名请求(csr)。

￮    启动时，Istio agent创建私钥和CSR，然后将带有凭证的CSR发送给Istiod进行签名。

￮    Istiod中的CA验证CSR中携带的凭据。在成功验证之后，它签署CSR以生成证书。

￮    这一步，njet与Envoy proxy不同，Envoy支持SDS动态获取，Envoy通过Envoy SDS API从同一个容器中的Istio agent 请求证书和密钥。Istio agent通过Envoy SDS API将从Istio接收到的证书和私钥发送给Envoy。njet的策略是直接在njet.conf中通过指令配置证书文件及私钥文件，证书文件及私钥文件会被Istio agent写到 /var/lib/istio/data/。

￮    Istio agent监视证书的过期。对于证书和密钥轮换，定期重复上述过程。

```
NJet Sidecar目前不支持动态证书加载，即证书轮换后，njet不会动态加载，此功能目前未实现
```

##### **3.21.5.1.2 看证书定期轮换**

Istio 默认签发的证书为 24h 有效期，生成的证书链和根证书及 key 文件被存放到 sidecar 容器的 /var/lib/istio/data 目录下，如下图所示：

![image-20230619135138210](https://gitee.com/gebona/picture/raw/master/202306191351765.png)

a.   可以通过查看文件生成时间，来确认证书已经轮换，如下所示：

![image-20230619135152609](https://gitee.com/gebona/picture/raw/master/202306191351296.png)

b.   或者到期后，通过openssl 命令查看证书过期时间是否修改，如下所示：

```
Shell
openssl x509 -noout -text -in /var/lib/istio/data/cert-chain.pem
```

![image-20230619135229241](https://gitee.com/gebona/picture/raw/master/202306191352584.png)

##### **3.21.5.1.3修改签发证书的有效期时间**

通过修改 Istio-sidecar-injector k8s ConfigMap 资源中的配置项，来改变证书的有效期日期。配置项为data.config.templates.sidecar.spec.containers 里的istio-proxy容器中的env变量SECRET_TTL的value，格式为1h（小时）、10m(分钟)。修改后delete pod，重新注入Sidecar，即可生效。执行下面指令修改配置(记得修改后保存)：

```
Shell
kubectl edit cm -n istio-system istio-sidecar-injector
```

![image-20230619135438185](https://gitee.com/gebona/picture/raw/master/202306191354885.png)

#### **3.21.5.2** **服务到服务的双向认证**(service-to-service mTLS)

##### 3.21.5.2.1**功能描述**

用于service-to-service身份验证，客户端与服务端服务彼此认证对端身份信息，验证通过后，按照双方配置的安全通信策略进行数据交互。对于传输层身份认证(transport authentication)，Istio 提供mTLS作为全栈解决方案，无需更改服务代码即可启用。mTLS方案提供以下特性：
 ● 为每个服务提供表示其角色的强标识，以支持跨集群和云的互操作性。
 ● service-to-service的安全通信。
 ● 提供一个key管理系统，使key和证书的生成、分发、轮换自动化。

##### 3.21.5.2.2**客户端TLS策略**

客户端服务，那些发送请求的服务，是有责任的遵循需要的身份认证机制。你可以通过DestinationRule CR中的ClientTLSSettings去设置与上游服务连接的安全策略。客户端服务根据用户创建的DestinationRule CR策略，给予与上游服务通信的约束，例如明文通信或者密文通信，如果密文通信，验证上游服务身份，来判断其是否可信。

**在Istio mTLS方案中，客户端TLS策略支持如下模式：**

![image-20230619144625587](https://gitee.com/gebona/picture/raw/master/202306191446673.png)

Istio通过k8s DestinationRule CR配置客户端TLS策略。

**NJet客户端TLS支持情况如下：**

NJet客户端TLS配置使用proxy_ssl相关指令进行配置。

```
njet客户端TLS默认配置为ISTIO_MUTUAL模式。
```

<table style="height: 230px; width: 447px;">
<tbody>
<tr style="height: 46px;">
<td style="height: 46px; width: 148.688px;">
<p>TLS mode</p>
</td>
<td style="height: 46px; width: 142.656px;">
<p>NJet Sidecar</p>
</td>
<td style="height: 46px; width: 133.656px;">
<p>描述</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 148.688px;">
<p>DISABLE</p>
</td>
<td style="height: 46px; width: 142.656px;">
<p>支持</p>
</td>
<td style="height: 46px; width: 133.656px;">
<p>明文传输</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 148.688px;">
<p>SIMPLE</p>
</td>
<td style="height: 46px; width: 142.656px;">
<p>非sidecar特性</p>
</td>
<td style="height: 46px; width: 133.656px;">
<p>密文传输</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 148.688px;">
<p>MUTUAL</p>
</td>
<td style="height: 46px; width: 142.656px;">
<p>非sidecar特性</p>
</td>
<td style="height: 46px; width: 133.656px;">
<p>密文传输</p>
</td>
</tr>
<tr style="height: 46px;">
<td style="height: 46px; width: 148.688px;">
<p>ISTIO_MUTUAL</p>
</td>
<td style="height: 46px; width: 142.656px;">
<p>支持</p>
</td>
<td style="height: 46px; width: 133.656px;">
<p>密文传输</p>
</td>
</tr>
</tbody>
</table>

详细介绍参照https://istio.io/v1.13/docs/reference/config/networking/destination-rule/#ClientTLSSettings

##### **3.21.5.2.3 服务端TLS策略**

Istio 通过 PeerAuthentication CR 实现服务端对客户端请求的认证策略配置。

PeerAuthentication 定义流量怎样通往 Sidecar，支持服务端口级别的安全认证。在 Sidecar 场景，接受请求的服务端为15006 listener(根据原始目的地port，转发流量，并且给予端口级别的认证策略)，因为 Istio iptables rule 将流量重定向到了15006，所以所有策略会被应用到此 listener。

**在Istio mTLS方案中，服务端TLS策略支持如下模式：**

![image-20230619145022177](https://gitee.com/gebona/picture/raw/master/202306191450545.png)

Istio 通过 k8s PeerAuthentication CR 配置服务端 TLS 策略。

**NJet 服务端 TLS 支持情况如下：**

NJet 服务端 TLS 配置在 stream 块中的 server 块使用 njtmesh_port_mode 相关指令进行配置。示例如下所示：

![image-20230619145134149](https://gitee.com/gebona/picture/raw/master/202306191451010.png)

```
njet服务端TLS默认配置为PERMISSIVE模式。
```

<table style="width: 678px;">
<tbody>
<tr>
<td style="width: 168.25px;">
<p>TLS mode</p>
</td>
<td style="width: 151.219px;">
<p>njet sidecar</p>
</td>
<td style="width: 336.531px;">
<p>描述</p>
</td>
</tr>
<tr>
<td style="width: 168.25px;">
<p>UNSET</p>
<p>&nbsp;</p>
</td>
<td style="width: 151.219px;">
<p>支持</p>
</td>
<td style="width: 336.531px;">
<p>继承父作用域的模式，如果最高级别(mesh级别)没有设置，默认使用PERMISSIVE</p>
</td>
</tr>
<tr>
<td style="width: 168.25px;">
<p>DISABLE</p>
</td>
<td style="width: 151.219px;">
<p>支持</p>
</td>
<td style="width: 336.531px;">
<p>只接受明文</p>
</td>
</tr>
<tr>
<td style="width: 168.25px;">
<p>PERMISSIVE</p>
</td>
<td style="width: 151.219px;">
<p>支持</p>
</td>
<td style="width: 336.531px;">
<p>可以接受密文和明文</p>
</td>
</tr>
<tr>
<td style="width: 168.25px;">
<p>STRICT</p>
</td>
<td style="width: 151.219px;">
<p>支持</p>
</td>
<td style="width: 336.531px;">
<p>严格的mTLS，不接受明文</p>
</td>
</tr>
</tbody>
</table>

详细介绍参照https://istio.io/v1.13/docs/reference/config/security/peer_authentication/

##### 3.21.5.2.4 **灰度发布：不同upstream不同安全策略**

使用“3.18.4.2 HTTP路由””创建一个基于header的灰度路由“章节的路由规则。我们修改reviews|v2| upstream 为明文传输。

通过下面指令修改：

```
Shell
kubectl edit dr reviews
```

![image-20230619145301546](https://gitee.com/gebona/picture/raw/master/202306191453859.png)

查看 NJet 配置

reviews|v2| upstream为明文传输

![image-20230619145321697](https://gitee.com/gebona/picture/raw/master/202306191453117.png)

reviews|v3| upstream为密文传输

![image-20230619145339823](https://gitee.com/gebona/picture/raw/master/202306191453243.png)

NJet location path实现如下：

![image-20230619145401471](https://gitee.com/gebona/picture/raw/master/202306191454991.png)

![image-20230619145409511](https://gitee.com/gebona/picture/raw/master/202306191454084.png)

￮    请求中不携带header->end-user：jason，请求被路由到reviews|v3| upstream，即reviews v3版本，v3会返回红色的星星，在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0
```

返回结果如下图所示，可知，与期望一致。

![image-20230619145438333](https://gitee.com/gebona/picture/raw/master/202306191454865.png)

在reviews v3 pod中Sidecar容器抓包

![image-20230619145458225](https://gitee.com/gebona/picture/raw/master/202306191454616.png)

![image-20230619145506110](https://gitee.com/gebona/picture/raw/master/202306191455647.png)

![image-20230619145517485](https://gitee.com/gebona/picture/raw/master/202306191455739.png)

由图可知，路由到reviews v3服务的请求发送的是密文。

￮    请求中携带header->end-user：jason，请求被路由到reviews|v2| upstream，即reviews v2版本，v2会返回黑色的星星，在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0 -H "end-user:jason"
```

返回结果如下图所示，可知，与期望一致。

![image-20230619145557812](https://gitee.com/gebona/picture/raw/master/202306191456377.png)

在reviews v2 pod中中sidecar容器抓包

![image-20230619145617283](https://gitee.com/gebona/picture/raw/master/202306191456634.png)

![image-20230619145711192](https://gitee.com/gebona/picture/raw/master/202306191457388.png)

![image-20230619145721180](https://gitee.com/gebona/picture/raw/master/202306191457743.png)

由图可知，路由到reviews v2服务的请求发送的是明文。

### 3.21.6日志

#### 3.21.6.1 **功能描述**

istio控制面(istiod)支持对agent及proxy的日志控制，不同的日志级别输出不同的日志，istio通过在注入sidecar容器时指定日志级别。sidecar-njet兼容istio 日志配置，因此，sidecar-njet支持对agent和njet进程的日志级别控制。

#### 3.21.6.2 **Agent** **日志**

istio控制面针对agent不同的功能，可以给与不同的日志级别配置，这样方便查看关注功能的log信息。当然前提是程序agent针对某一个功能需要先注册一个logger。目前agent注册了如下logger：

<table style="width: 462px;">
<tbody>
<tr>
<td style="width: 146.812px;">
<p>logger类型</p>
</td>
<td style="width: 299.188px;">
<p>logger描述</p>
</td>
</tr>
<tr>
<td style="width: 146.812px;">
<p>ca</p>
</td>
<td style="width: 299.188px;">
<p>ca 客户端功能相关</p>
</td>
</tr>
<tr>
<td style="width: 146.812px;">
<p>cache</p>
</td>
<td style="width: 299.188px;">
<p>Secret 缓冲，即证书缓冲</p>
</td>
</tr>
<tr>
<td style="width: 146.812px;">
<p>citadelclient</p>
</td>
<td style="width: 299.188px;">
<p>ca 客户端功能相关</p>
</td>
</tr>
<tr>
<td style="width: 146.812px;">
<p>njet-controller</p>
</td>
<td style="width: 299.188px;">
<p>管理NJet及生成NJet配置</p>
</td>
</tr>
<tr>
<td style="width: 146.812px;">
<p>sds</p>
</td>
<td style="width: 299.188px;">
<p>生成secret</p>
</td>
</tr>
<tr>
<td style="width: 146.812px;">
<p>spiffe</p>
</td>
<td style="width: 299.188px;">
<p>spiffe相关处理(spiffe是一种标准)</p>
</td>
</tr>
</tbody>
</table>

all 与defalut为程序引入的日志包内部存在的logger，程序员可直接使用。all logger会重写所有logger的日志级别。agent中针对不同的logger都注册了日志级别，即默认级别，比如njet-controller logger默认为error级别。

日志级别如下：

<table style="width: 538px;">
<tbody>
<tr>
<td style="width: 173.062px;">
<p>日志级别</p>
</td>
<td style="width: 348.938px;">
<p>描述</p>
</td>
</tr>
<tr>
<td style="width: 173.062px;">
<p>debug</p>
</td>
<td style="width: 348.938px;">
<p>会打印所有级别的信息</p>
</td>
</tr>
<tr>
<td style="width: 173.062px;">
<p>info</p>
</td>
<td style="width: 348.938px;">
<p>打印除debug的日志</p>
</td>
</tr>
<tr>
<td style="width: 173.062px;">
<p>warn</p>
</td>
<td style="width: 348.938px;">
<p>打印除debug、info的日志</p>
</td>
</tr>
<tr>
<td style="width: 173.062px;">
<p>error</p>
</td>
<td style="width: 348.938px;">
<p>打印error、fatal级别的日志</p>
</td>
</tr>
<tr>
<td style="width: 173.062px;">
<p>fatal</p>
</td>
<td style="width: 348.938px;">
<p>只打印fatal级别的日志</p>
</td>
</tr>
</tbody>
</table>

##### 3.21.6.2.1 **Istio** **控制面配置**

需要修改Istio 配置参数，配置参数需要在Istio 创建的k8s ConfigMap资源中进行修改，具体ConfigMap为istio-system ns下的Istio-sidecar-Injector ConfigMap，可以通过下面指令查询。

```
Shell
[root@node151 ~]# kubectl get cm -n istio-system
NAME                                  DATA   AGE
istio                                 2      121d
istio-ca-root-cert                    1      121d
istio-gateway-deployment-leader       0      121d
istio-gateway-status-leader           0      121d
istio-leader                          0      121d
istio-namespace-controller-election   0      121d
istio-sidecar-injector                2      121d
kube-root-ca.crt                      1      121d
[root@node151 ~]#
```

通过修改istio-sidecar-injector中的values 对应的值，来修改agent日志级别。修改的配置项为values.global.logging.level，格式如下：

<scope>:<level>,<scope>:<level>

#### 3.21.6.3 **NJet日志**

Istio 控制面可以为proxy设置日志级别。proxy的日志级别取决于proxy本身的支持情况。NJet proxy支持如下日志级别：

<table>
<tbody>
<tr>
<td width="100">
<p>日志级别</p>
</td>
</tr>
<tr>
<td width="100">
<p>debug</p>
</td>
</tr>
<tr>
<td width="100">
<p>info</p>
</td>
</tr>
<tr>
<td width="100">
<p>notice</p>
</td>
</tr>
<tr>
<td width="100">
<p>warn</p>
</td>
</tr>
<tr>
<td width="100">
<p>error</p>
</td>
</tr>
<tr>
<td width="100">
<p>crit</p>
</td>
</tr>
<tr>
<td width="100">
<p>alert</p>
</td>
</tr>
<tr>
<td width="100">
<p>emerg</p>
</td>
</tr>
</tbody>
</table>

##### 3.21.6.3.1 **Istio** **控制面配置**

通过修改Istio-sidecar-Injector中的values 对应的值，来修改proxy日志级别。修改的配置项为values.global.proxy.logLevel，格式为NJet支持的日志级别字符串。

### 3.21.7 **HTTP**重写

#### 3.21.7.1 **功能描述**

HTTPRewrite可用于在将请求转发到目的地之前重写HTTP请求的特定部分。

Istio sidecar njet支持istio 控制面HTTP重写规则，目前支持的重写规则情况如下表所述：

<table style="width: 632px;">
<tbody>
<tr>
<td style="width: 154.609px;">
<p>字段名称</p>
</td>
<td style="width: 191.688px;">
<p>字段类型</p>
</td>
<td style="width: 263.703px;">
<p>Istio njet sidecar支持情况</p>
</td>
</tr>
<tr>
<td style="width: 154.609px;">
<p>uri</p>
</td>
<td style="width: 191.688px;">
<p>string</p>
</td>
<td style="width: 263.703px;">
<p>支持</p>
</td>
</tr>
<tr>
<td style="width: 154.609px;">
<p>authority</p>
</td>
<td style="width: 191.688px;">
<p>string</p>
</td>
<td style="width: 263.703px;">
<p>支持</p>
</td>
</tr>
</tbody>
</table>

Istio 控制面HTTP重写接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“HTTP重写”章节。

#### 3.21.7.2 **NJet sidecar HTTP**重写实现

##### 3.21.7.2.1 **URI**重写

NJet通过rewrite指令实现。支持exact、prefix、regex类型的uri重写。支持exact、prefix、regex类型uri普通路由匹配，及基于header等高级路由匹配下的HTTP重写。

<table style="width: 305px;">
<tbody>
<tr>
<td style="width: 99.9219px;">
<p>uri类型</p>
</td>
<td style="width: 189.078px;">
<p>重写结果</p>
</td>
</tr>
<tr>
<td style="width: 99.9219px;">
<p>exact</p>
</td>
<td style="width: 189.078px;">
<p>完全替换</p>
</td>
</tr>
<tr>
<td style="width: 99.9219px;">
<p>prefix</p>
</td>
<td style="width: 189.078px;">
<p>替换对应的匹配前缀</p>
</td>
</tr>
<tr>
<td style="width: 99.9219px;">
<p>regex</p>
</td>
<td style="width: 189.078px;">
<p>完全替换</p>
</td>
</tr>
</tbody>
</table>

##### 3.21.7.2.2 **host**重写

NJet通过proxy_set_header指令实现，设置http HOST头。

#### 3.21.7.3 **Istio HTTP**重写示例

##### 3.21.7.3.1 **uri**重写

###### 3.21.7.3.1.1 **exact uri**重写

本测试使用uri为/test 进行测试，创建如下VirtualService配置：

```
Shell
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - uri:    
        exact: /test
    rewrite:
      uri: /details/0
    route:
    - destination:
        host: details
        subset: v1
```

match指明，当请求uri为/test时，请求path被重写为/details/0，最终路由到details|v1| upstream。

在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/test
```

返回结果如下图所示，可知，与期望一致(返回/details/0接口的内容)。

root@productpage-v1-64884d7c8d-c9cgr:/opt/microservices# curl http://reviews:9080/test

{"id":0,"author":"William Shakespeare","year":1595,"type":"paperback","pages":200,"publisher":"PublisherA","language":"English","ISBN-10":"1234567890","ISBN-13":"123-1234567890"}

**查看details v1服务日志：**

[root@node151 ~]# kubectl logs details-v1-dd6c5d9c9-bf5tj -f

[2023-04-26 07:00:41] INFO WEBrick 1.6.0

[2023-04-26 07:00:41] INFO ruby 2.7.1 (2020-03-31) [x86_64-linux]

[2023-04-26 07:00:41] INFO WEBrick::HTTPServer#start: pid=1 port=9080

 

127.0.0.6 - - [26/Apr/2023:07:53:51 UTC] "GET /details/0 HTTP/1.1" 200 178

•     -> /details/0

###### 3.21.7.3.1.2 **prefix uri**重写

本测试使用uri前缀为/huidu 进行测试，创建如下VirtualService配置：

```
Shell
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
      - method:
          exact: post
        uri:
          prefix: /huidu
      route:
      - destination:
          host: reviews
          subset: v1
    - match:
      - uri:
          prefix: /huidu
      rewrite:
        uri: /reviews/0
      route:
      - destination:
          host: reviews
          subset: v3
```

match指明，当请求uri有/huidu前缀时，请求path被重写为/reviews/0，最终路由到reviews|v3| upstream，即reviews v3版本。

在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/huidu666?key=value -v
```

返回结果如下图所示，可知，与期望一致。

root@productpage-v1-64884d7c8d-c9cgr:/opt/microservices# curl http://reviews:9080/huidu666?key=value

{"id": "666","reviews": [{ "reviewer": "Reviewer1", "text": "An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!", "rating": {"stars": 5, "color": "red"}},{ "reviewer": "Reviewer2", "text": "Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare.", "rating": {"stars": 4, "color": "red"}}]}

**我们在reviews-v3 sidecar容器中抓包看一下，重写后的请求uri：**

08:13:28.131554 IP 10.234.190.191.49856 > 10.234.190.193.9080: Flags [P.], seq 1:283, ack 1, win 225, options [nop,nop,TS val 2004186302 ecr 2004186302], length 282

​    0x0000: 4500 014e 348c 4000 3f06 72c9 0aea bebf E..N4.@.?.r.....

​    0x0010:  0aea bec1 c2c0 2378 69c8 4035 e736 0085 ......#xi.@5.6..

​    0x0020: 8018 00e1 9495 0000 0101 080a 7775 74be ............wut.

​    0x0030: 7775 74be 4745 5420 2f72 6576 6965 7773 wut.GET./reviews

​    0x0040: 2f30 3636 363f 6b65 793d 7661 6c75 6520 /0666?key=value.

​    0x0050: 4854 5450 2f31 2e31 0d0a 486f 7374 3a20 HTTP/1.1..Host:.

​    0x0060: 7265 7669 6577 733a 3930 3830 0d0a 582d reviews:9080

###### 3.21.7.3.1.3 **regex uri**重写

创建如下VirtualService配置：

```
Shell
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - uri:
        regex: \d+$
    rewrite:
      uri: /details/0
    route:
    - destination:
        host: details
        subset: v1  
```

match指明，当请求uri有连续数字且以数字结尾时，请求path被重写为/details/0，最终路由到details|v1| upstream，即details v1版本。

在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/lsssss/123 -v
```

返回结果如下图所示，可知，与期望一致。

root@productpage-v1-64884d7c8d-c9cgr:/opt/microservices# curl http://reviews:9080/lsssss/123

{"id":0,"author":"William Shakespeare","year":1595,"type":"paperback","pages":200,"publisher":"PublisherA","language":"English","ISBN-10":"1234567890","ISBN-13":"123-1234567890"}

##### 3.21.7.3.2 **host**重写

创建如下VirtualService配置：

```
Shell
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
      - method:
          exact: post
        uri:
          prefix: /huidu
      route:
      - destination:
          host: reviews
          subset: v1
    - match:
      - uri:
          prefix: /huidu
      rewrite:
        uri: /reviews/0
        authority: testrewrite.com
      route:
      - destination:
          host: reviews
          subset: v3 
```

match指明，当请求uri有/huidu前缀时，请求path被重写为/reviews/0，HOST被重写为testrewrite.com，最终路由到reviews|v3| upstream，即reviews v3版本。

在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/huidu
```

返回结果如下图所示，可知，与期望一致。

root@productpage-v1-64884d7c8d-c9cgr:/opt/microservices# curl http://reviews:9080/huidu

{"id": "0","reviews": [{ "reviewer": "Reviewer1", "text": "An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!", "rating": {"stars": 5, "color": "red"}},{ "reviewer": "Reviewer2", "text": "Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare.", "rating": {"stars": 4, "color": "red"}}]}

**我们在****reviews-v3 sidecar****容器中抓包看一下，重写后的请求****uri****及****HOST****：**

0x0030: 777f f1d4 4745 5420 2f72 6576 6965 7773 w...GET./reviews

​    0x0040: 2f30 2048 5454 502f 312e 310d 0a48 6f73 /0.HTTP/1.1..Hos

​    0x0050: 743a 2074 6573 7472 6577 7269 7465 2e63 t:.testrewrite.c

​    0x0060: 6f6d 0d0a 582d 466f 7277 6172 6465 642d om..X-Forwarded-

​    0x0070: 486f 7374 3a20 7265 7669 6577 733a 3930 Host:.reviews:90

​    0x0080: 3830 0d0a 582d 466f 7277 6172 6465 642d 80.

### 3.21.8 **HTTP**重定向

#### 3.21.8.1 **功能描述**

HTTPRedirect可用于向调用方发送301、302、303、307、308重定向响应，其中响应中的scheme、Authority/Host和URI可以与指定的值交换。vs redirect与vs route(rewrite)二选一，两者都为目的地，选一即可，如果都配置，校验失败，创建vs失败。

Istio sidecar njet支持istio 控制面HTTP重定向规则，目前支持的重定向规则情况如下表所述：

<table style="width: 679px;">
<tbody>
<tr>
<td style="width: 116.219px;">
<p>字段名称</p>
</td>
<td style="width: 162.219px;">
<p>字段类型</p>
</td>
<td style="width: 175.375px;">
<p>Istio njet sidecar支持情况</p>
</td>
<td style="width: 197.188px;">
<p>备注</p>
</td>
</tr>
<tr>
<td style="width: 116.219px;">
<p>uri</p>
</td>
<td style="width: 162.219px;">
<p>string</p>
</td>
<td style="width: 175.375px;">
<p>支持</p>
</td>
<td style="width: 197.188px;">
<p>注意，整个路径将被替换,不管请求URI匹配的是exact、prefix、regex。</p>
</td>
</tr>
<tr>
<td style="width: 116.219px;">
<p>authority</p>
</td>
<td style="width: 162.219px;">
<p>string</p>
</td>
<td style="width: 175.375px;">
<p>支持</p>
</td>
<td style="width: 197.188px;">
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 116.219px;">
<p>port</p>
</td>
<td style="width: 162.219px;">
<p>uint32 (oneof)</p>
</td>
<td style="width: 175.375px;">
<p>支持</p>
</td>
<td style="width: 197.188px;">
<p>port与derivePort，二选一，如果两个都配置时，会报错，导致vs创建失败</p>
</td>
</tr>
<tr>
<td style="width: 116.219px;">
<p>derivePort</p>
<p>&nbsp;</p>
</td>
<td style="width: 162.219px;">
<p>RedirectPortSelection (oneof)</p>
</td>
<td style="width: 175.375px;">
<p>支持</p>
<p>&nbsp;</p>
</td>
<td style="width: 197.188px;">
<p>FROM_PROTOCOL_DEFAULT时，istio控制面不会下发port值</p>
</td>
</tr>
<tr>
<td style="width: 116.219px;">
<p>scheme</p>
</td>
<td style="width: 162.219px;">
<p>string</p>
</td>
<td style="width: 175.375px;">
<p>支持</p>
</td>
<td style="width: 197.188px;">
<p>如果未设置，将使用原值</p>
</td>
</tr>
<tr>
<td style="width: 116.219px;">
<p>redirectCode</p>
</td>
<td style="width: 162.219px;">
<p>uint32</p>
</td>
<td style="width: 175.375px;">
<p>支持</p>
</td>
<td style="width: 197.188px;">
<p>默认301</p>
</td>
</tr>
</tbody>
</table>

Istio 控制面HTTP重定向接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“HTTP重定向”章节。

#### 3.21.8.2 **NJet sidecar HTTP**重定向实现

NJet 通过return指令实现。支持301、302、303、307、308重定向，非这些值的无符号3xx整数，Istio忽略(导致目的地为nil，既不会重定向也不会路由到其他地方)。其他值(除0外，0处理为301)vs校验失败，导致vs创建失败。

Istio authority、port规则与重定向后的URL中的host、port域的对应关系如下：

<table style="width: 614px;">
<tbody>
<tr>
<td style="width: 96.3281px;" rowspan="2">
<p>&nbsp;</p>
</td>
<td style="width: 354.172px;" colspan="2">
<p>重定向后的URL域</p>
</td>
<td style="width: 141.5px;" colspan="2">
<p>istio规则</p>
</td>
</tr>
<tr>
<td style="width: 251.844px;">
<p>host</p>
</td>
<td style="width: 96.3281px;">
<p>port</p>
</td>
<td style="width: 66.2188px;">
<p>authority</p>
</td>
<td style="width: 69.2812px;">
<p>port</p>
</td>
</tr>
<tr>
<td style="width: 96.3281px;" rowspan="4">
<p>值</p>
</td>
<td style="width: 251.844px;">
<p>http HOST 头</p>
</td>
<td style="width: 96.3281px;">
<p>空</p>
<p>&nbsp;</p>
</td>
<td style="width: 66.2188px;">
<p>空</p>
<p>&nbsp;</p>
</td>
<td style="width: 69.2812px;">
<p>空</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 251.844px;">
<p>http HOST 头中有端口，则剔除</p>
</td>
<td style="width: 96.3281px;">
<p>Istio port值</p>
<p>&nbsp;</p>
</td>
<td style="width: 66.2188px;">
<p>空</p>
<p>&nbsp;</p>
</td>
<td style="width: 69.2812px;">
<p>非空</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 251.844px;">
<p>Istio authority值</p>
</td>
<td style="width: 96.3281px;">
<p>空</p>
</td>
<td style="width: 66.2188px;">
<p>非空</p>
</td>
<td style="width: 69.2812px;">
<p>空</p>
</td>
</tr>
<tr>
<td style="width: 251.844px;">
<p>Istio authority值</p>
</td>
<td style="width: 96.3281px;">
<p>Istio port值</p>
</td>
<td style="width: 66.2188px;">
<p>非空</p>
</td>
<td style="width: 69.2812px;">
<p>非空</p>
</td>
</tr>
</tbody>
</table>

Istio uri规则与重定向后的URL中的uri域的对应关系如下：

<table style="width: 630px;">
<tbody>
<tr>
<td style="width: 110.172px;" rowspan="2">
<p>&nbsp;</p>
</td>
<td style="width: 295.469px;">
<p>重定向后的URL域</p>
</td>
<td style="width: 202.359px;">
<p>istio规则</p>
</td>
</tr>
<tr>
<td style="width: 295.469px;">
<p>uri</p>
</td>
<td style="width: 202.359px;">
<p>uri</p>
</td>
</tr>
<tr>
<td style="width: 110.172px;" rowspan="3">
<p>值</p>
</td>
<td style="width: 295.469px;">
<p>原始请求uri</p>
</td>
<td style="width: 202.359px;">
<p>空</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 295.469px;">
<p>Istio uri值</p>
</td>
<td style="width: 202.359px;">
<p>非空，有参数</p>
</td>
</tr>
<tr>
<td style="width: 295.469px;">
<p>Istio uri值，追加请求原始参数</p>
</td>
<td style="width: 202.359px;">
<p>非空，无参数</p>
</td>
</tr>
</tbody>
</table>

#### 3.21.8.3 **Istio HTTP**重定向示例

创建如下VirtualService配置：

```
Shell
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - uri:
        exact: /backend
    redirect:
      port: 666
      uri: /productpage?test=123
```

match指明，当请求uri为/backend时，请求被重定向。

在productpage pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/backend -v
```

返回结果如下图所示，可知，与期望一致(port为666，host剔除http HOST中的9080，scheme为请求中的http，uri为Istio uri值)。

\*  Trying 10.233.137.235...

\* TCP_NODELAY set

\* Expire in 200 ms for 4 (transfer 0x55990c115f50)

\* Connected to reviews (10.233.137.235) port 9080 (#0)

\> GET /backend HTTP/1.1

\> Host: reviews:9080

\> User-Agent: curl/7.64.0

\> Accept: */*

\> 

< HTTP/1.1 301 Moved Permanently

< Server: njet/1.23.1

< Date: Wed, 26 Apr 2023 09:44:59 GMT

< Content-Type: text/html

< Content-Length: 168

< Connection: keep-alive

< Location: http://reviews:666/productpage?test=123

< 

### 3.21.9 **HTTP**流量镜像

#### 3.21.9.1 **功能描述**

除了将请求转发到预定目的地外，还将HTTP流量镜像到另一个目的地。在从原始目的地返回响应之前，sidecar不会等待镜像集群的响应。

Istio sidecar njet支持istio 控制面HTTP流量镜像规则，目前支持的流量镜像规则情况如下表所述：

<table style="width: 694px;">
<tbody>
<tr>
<td style="width: 310.453px;">
<p>字段名称</p>
</td>
<td style="width: 162.234px;">
<p>字段类型</p>
</td>
<td style="width: 199.312px;">
<p>Istio njet sidecar支持情况</p>
</td>
</tr>
<tr>
<td style="width: 310.453px;">
<p>mirror</p>
</td>
<td style="width: 162.234px;">
<p><a href="https://istio.io/v1.13/docs/reference/config/networking/virtual-service/#Destination">Destination</a></p>
</td>
<td style="width: 199.312px;">
<p>支持</p>
</td>
</tr>
<tr>
<td style="width: 310.453px;">
<p>mirrorPercentage</p>
</td>
<td style="width: 162.234px;">
<p><a href="https://istio.io/v1.13/docs/reference/config/networking/virtual-service/#Percent">Percent</a></p>
</td>
<td style="width: 199.312px;">
<p>不支持</p>
</td>
</tr>
</tbody>
</table>

Istio 控制面HTTP流量镜像接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“HTTP流量镜像”章节。

#### 3.21.9.2 **Sidecar NJet HTTP**流量镜像实现

NJet通过mirror指令实现HTTP流量镜像。支持exact、prefix、regex类型uri普通路由匹配，及基于header等高级路由匹配下的HTTP流量镜像。且支持请求rewite uri后镜像。

100%的流量镜像到镜像集群。

#### 3.21.9.3 **Istio HTTP**镜像测试案例

这里给出regex uri普通路由匹配下的流量镜像，及基于header的高级路由匹配下的流量镜像测试案例。

##### 3.21.9.3.1 **regex uri+rewrite** **下的流量镜像**

创建如下VirtualService配置：

```
YAML
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - uri:
        regex: \d+$
    mirror:
      host: reviews
      port:
        number: 9080
      subset: v2
    mirrorPercentage:
      value: 86
    rewrite:
      uri: /reviews/abc
    route:
    - destination:
        host: details
        subset: v1
```

match指明，当请求uri以数字结尾时，请求path被重写为/reviews/abc，并且镜像到reviews|v2| upstream，路由到details|v1| upstream。

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/123
```

返回结果如下图所示，可知，与期望一致(path被重写，details|v1| upstream返回错误，且reviews|v2| 收到镜像请求并且host增加-shadow后缀)。

```
root@reviews-v3-674898dfb-sgsht:/# curl http://reviews:9080/123
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0//EN">
<HTML>
  <HEAD><TITLE>Not Found</TITLE></HEAD>
  <BODY>
    <H1>Not Found</H1>
    `/reviews/abc' not found.
    <HR>
    <ADDRESS>
     WEBrick/1.6.0 (Ruby/2.7.1/2020-03-31) at
     reviews:9080
    </ADDRESS>
  </BODY>
</HTML>
root@reviews-v3-674898dfb-sgsht:/#
details|v1| 服务日志如下：
[root@node151 ~]# kubectl logs details-v1-dd6c5d9c9-5flgw -f
[2023-05-06 02:58:27] INFO  WEBrick 1.6.0
[2023-05-06 02:58:27] INFO  ruby 2.7.1 (2020-03-31) [x86_64-linux]
[2023-05-06 02:58:27] INFO  WEBrick::HTTPServer#start: pid=1 port=9080
[2023-05-06 05:59:50] ERROR `/reviews/abc' not found.
127.0.0.6 - - [06/May/2023:05:59:50 UTC] "GET /reviews/abc HTTP/1.1" 404 279
•	-> /reviews/abc
Details 没有实现/reviews/abc 接口，故log显示not found
```

**details|v1|** **服务日志如下：**

[root@node151 ~]# kubectl logs details-v1-dd6c5d9c9-5flgw -f

[2023-05-06 02:58:27] INFO WEBrick 1.6.0

[2023-05-06 02:58:27] INFO ruby 2.7.1 (2020-03-31) [x86_64-linux]

[2023-05-06 02:58:27] INFO WEBrick:HTTPServer#start: pid=1 port=9080

[2023-05-06 05:59:50] ERROR `/reviews/abc' not found.

127.0.0.6 - - [06/May/2023:05:59:50 UTC] "GET /reviews/abc HTTP/1.1" 404 279

•     -> /reviews/abc

Details 没有实现/reviews/abc 接口，故log显示not found

**reviews|v2|** **服务日志如下：**

[WARNING ] No operation matching request path "/reviews/abc" is found, Relative Path: /reviews/abc, HTTP Method: GET, ContentType: */*, Accept: */*,. Please enable FINE/TRACE log level for more details.

Reviews v2没有实现/reviews/abc 接口，故log显示No operation matching

**reviews|v2| sidecar容器中抓包，查看host：**

```
Shell
 sudo tcpdump -i eth0 port 9080 -n -nn -X
```

GET./reviews

​    0x0040: 2f61 6263 2048 5454 502f 312e 300d 0a48 /abc.HTTP/1.0..H

​    0x0050: 6f73 743a 2072 6576 6965 7773 3a39 3038 ost:.reviews:908

​    0x0060: 302d 7368 6164 6f77 0d0a 436f 6e6e 6563 0-shadow..Connec

​    0x0070: 7469 6f6e 3a20 636c 6f73 650d 0a55 7365 tion:.close..Use

​    0x0080: 722d 4167 656e 743a 2063 7572 6c2f 372e r-Agent:.curl/7.

​    0x0090: 3538 2e30 0d0a 4163 6365 7074 3a20 2a2f 58.0.

##### 3.21.9.3.2 **regex uri** **下的流量镜像**

创建如下VirtualService配置：

```
YAML
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - uri:
        regex: \d+$
    mirror:
      host: reviews
      port:
        number: 9080
      subset: v2
    mirrorPercentage:
      value: 86
    route:
    - destination:
        host: details
        subset: v1
```

match指明，当请求uri以数字结尾时，镜像到reviews|v2| upstream，路由到details|v1| upstream。

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/123
```

返回结果如下图所示，可知，与期望一致(details|v1| upstream返回错误，且reviews|v2| 收到镜像请求并且host增加-shadow后缀)。

```
root@reviews-v3-674898dfb-sgsht:/# curl http://reviews:9080/123

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0//EN">

<HTML>

 <HEAD><TITLE>Not Found</TITLE></HEAD>

 <BODY>

  <H1>Not Found</H1>

  `/123' not found.

  <HR>

  <ADDRESS>

   WEBrick/1.6.0 (Ruby/2.7.1/2020-03-31) at

   reviews:9080

  </ADDRESS>

 </BODY>

</HTML>

root@reviews-v3-674898dfb-sgsht:/#
```

**details|v1|** **服务日志如下：**

[2023-05-06 06:44:34] ERROR `/123' not found.

127.0.0.6 - - [06/May/2023:06:44:34 UTC] "GET /123 HTTP/1.1" 404 271

•     -> /123

Details 没有实现/123 接口，故log显示not found

**reviews|v2|** **服务日志如下：**

[WARNING ] No operation matching request path "/123" is found, Relative Path: /123, HTTP Method: GET, ContentType: */*, Accept: */*,. Please enable FINE/TRACE log level for more details.

Reviews v2没有实现/123 接口，故log显示No operation matching

**reviews|v2| sidecar**容器中抓包，查看**host**：

```
Shell
 sudo tcpdump -i eth0 port 9080 -n -nn -X
```

 3233 2048 5454 ....GET./123.HTT

​    0x0040: 502f 312e 300d 0a48 6f73 743a 2072 6576 P/1.0..Host:.rev

​    0x0050: 6965 7773 3a39 3038 302d 7368 6164 6f77 iews:9080-shadow

​    0x0060: 0d0a 436f 6e6e 6563 7469 6f6e 3a20 636c ..Connection:.cl

​    0x0070: 6f73 650d 0a55 7365 722d 4167 656e 743a ose..User-Agent:

​    0x0080: 2063 7572 6c2f 372e 3538 2e30 0d0a 4163 .curl/7.58.0..Ac

​    0x0090: 6365 7074 3a20 2a2f 2a0d 0a0d 0a     cept:.*/*....

##### 3.21.9.3.3 **高级路由下的流量镜像**

创建匹配header的高级路由，实现灰度发布(携带end-user:jason被路由到reviews v1，且镜像到reviews v2，path被重写，否则被路由到reviews v3，且镜像到details v1，path不被重写)。创建如下VirtualService配置：

```
YAML
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - match:
    - uri:
        exact: /reviews/0
    mirror:
      host: details
      port:
        number: 9080
      subset: v1
    mirrorPercentage:
      value: 86
    route:
    - destination:
        host: reviews
        subset: v3
  - match:
    - headers:
        end-user:
          exact: jason
      method:
        exact: get
      uri:
        exact: /reviews/0
    mirror:
      host: reviews
      port:
        number: 9080
      subset: v2
    mirrorPercentage:
      value: 86
    rewrite:
      uri: /hello
    route:
    - destination:
        host: reviews
        subset: v1
```

###### 3.21.9.3.3.1 **携带**end-user:jason header

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0 -H "end-user:jason" -v
```

返回结果如下图所示，可知，与期望一致(path被重写，reviews|v1| upstream返回错误，且reviews|v2| 收到镜像请求并且host增加-shadow后缀)。

root@reviews-v3-674898dfb-sgsht:/# curl http://reviews:9080/reviews/0 -H "end-user:jason" -v

•      Trying 10.233.164.213...

•     TCP_NODELAY set

•     Connected to reviews (10.233.164.213) port 9080 (#0)

*GET /reviews/0 HTTP/1.1*

*Host: reviews:9080*

*User-Agent: curl/7.58.0*

*Accept: /*

*end-user:jason*

\> 

< HTTP/1.1 404 Not Found

< Server: njet/1.23.1

< Date: Sat, 06 May 2023 06:56:15 GMT

< Content-Length: 0

< Connection: keep-alive

< X-Powered-By: Servlet/3.1

< Allow: GET,OPTIONS,HEAD

< Content-Language: en-US

< 

•     Connection #0 to host reviews left intact

404是由于reviews v1没有实现/hello接口

**reviews|v1|** **服务日志如下：**

[WARNING ] No operation matching request path "/hello" is found, Relative Path: /hello, HTTP Method: GET, ContentType: */*, Accept: */*,. Please enable FINE/TRACE log level for more details.

Reviews v1没有实现/hello 接口，故log显示No operation matching

**reviews|v2|** **服务日志如下：**

[WARNING ] No operation matching request path "/hello" is found, Relative Path: /hello, HTTP Method: GET, ContentType: */*, Accept: */*,. Please enable FINE/TRACE log level for more details.

Reviews v2没有实现/hello 接口，故log显示No operation matching

**reviews|v2| sidecar****容器中抓包，查看****host****：**

 0x0030: 00de 69b8 4745 5420 2f68 656c 6c6f 2048 ..i.GET./hello.H

​    0x0040: 5454 502f 312e 300d 0a48 6f73 743a 2072 TTP/1.0..Host:.r

​    0x0050: 6576 6965 7773 3a39 3038 302d 7368 6164  eviews:9080-shad

​    0x0060: 6f77 0d0a 436f 6e6e 6563 7469 6f6e 3a20 ow..Connection:.

​    0x0070: 636c 6f73 650d 0a55 7365 722d 4167 656e close..User-Agen

​    0x0080: 743a 2063 7572 6c2f 372e 3538 2e30 0d0a t:.curl/7.58.0..

​    0x0090: 4163 6365 7074 3a20 2a2f 2a0d 0a65 6e64 Accept:.*/*..end

​    0x00a0: 2d75 7365 723a 206a 6173 6f6e 0d0a 0d0a -user:.jason....

###### 3.21.9.3.3.2 **不携带**end-user:jason header

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0  -v
```

返回结果如下图所示，可知，与期望一致成功路由到reviews|v3|

root@reviews-v3-674898dfb-sgsht:/# curl http://reviews:9080/reviews/0 -v

•      Trying 10.233.164.213...

•     TCP_NODELAY set

•     Connected to reviews (10.233.164.213) port 9080 (#0)

*GET /reviews/0 HTTP/1.1*

*Host: reviews:9080* 

*User-Agent: curl/7.58.0*

*Accept: /*

\> 

< HTTP/1.1 200 OK

< Server: njet/1.23.1

< Date: Sat, 06 May 2023 07:03:12 GMT

< Content-Type: application/json

< Content-Length: 375

< Connection: keep-alive

< X-Powered-By: Servlet/3.1

< Content-Language: en-US

< 

•     Connection #0 to host reviews left intact

{"id": "0","reviews": [{ "reviewer": "Reviewer1", "text": "An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!", "rating": {"stars": 5, "color": "red"}},{ "reviewer": "Reviewer2", "text": "Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare.", "rating": {"stars": 4, "color": "red"}}]}

返回red，请求到达reviews v3.

 

**details|v1|** **服务日志如下：**

[2023-05-06 07:03:12] ERROR `/reviews/0' not found.

127.0.0.6 - - [06/May/2023:07:03:12 UTC] "GET /reviews/0 HTTP/1.0" 404 268

\- -> /reviews/0

 

Details v2没有实现/reviews/0接口，故log显示not found

**details|v1| sidecar**容器中抓包，查看host：

   0x0030: 00e1 b81a 4745 5420 2f72 6576 6965 7773 ....GET./reviews

​    0x0040: 2f30 2048 5454 502f 312e 300d 0a48 6f73 /0.HTTP/1.0..Hos

​    0x0050: 743a 2072 6576 6965 7773 3a39 3038 302d t:.reviews:9080-

​    0x0060: 7368 6164 6f77 0d0a 436f 6e6e 6563 7469 shadow..Connecti

​    0x0070: 6f6e 3a20 636c 6f73 650d 0a55 7365 722d on:.close..User-

​    0x0080: 4167 656e 743a 2063 7572 6c2f 372e 3538 Agent:.curl/7.58

​    0x0090: 2e30 0d0a 4163 6365 7074 3a20 2a2f 2a0d .0..Accept:.*/*

### 3.21.10 **HTTP**头操作

#### 3.21.10.1 **功能描述**

HTTP headers操作，当sidecar向目的地服务转发请求或从目的地服务转发响应时，可以操纵消息头。可以为特定路由目的地(基于权重的分流场景会有多个目的地，每一个称为特定目的地，可以有专属的头操作规则)或所有目的地指定header操作规则(应用于所有目的地)，前者称作目的地级别头操作，后者称作路由级别头操作。头操作支持重写(set)、追加(add)、移除(remove)。

Istio sidecar njet支持istio 控制面HTTP头操作规则，目前支持的头操作规则情况如下表所述：

<table style="width: 674px;">
<tbody>
<tr>
<td style="width: 114.953px;">
<p>字段名称</p>
</td>
<td style="width: 151.078px;">
<p>字段类型</p>
</td>
<td style="width: 134.375px;">
<p>Istio njet sidecar支持情况</p>
</td>
<td style="width: 245.594px;">
<p>备注</p>
</td>
</tr>
<tr>
<td style="width: 114.953px;">
<p>request</p>
<p>&nbsp;</p>
</td>
<td style="width: 151.078px;">
<p><a href="https://istio.io/v1.13/docs/reference/config/networking/virtual-service/#Headers-HeaderOperations">HeaderOperations</a></p>
</td>
<td style="width: 134.375px;">
<p>支持</p>
<p>&nbsp;</p>
</td>
<td style="width: 245.594px;">
<p>add操作与set操作一致，即用给定的值覆盖由key指定的header</p>
</td>
</tr>
<tr>
<td style="width: 114.953px;">
<p>response</p>
</td>
<td style="width: 151.078px;">
<p><a href="https://istio.io/v1.13/docs/reference/config/networking/virtual-service/#Headers-HeaderOperations">HeaderOperations</a></p>
</td>
<td style="width: 134.375px;">
<p>支持</p>
</td>
<td style="width: 245.594px;">
<p>add操作与set操作一致</p>
</td>
</tr>
</tbody>
</table>

Istio 控制面HTTP头操作接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“HTTP头操作”章节。

  注：如果目的地级别和路由级别头操作同时配置，执行顺序遵循如下定义： 

 1.先执行目的地级别，后执行路由级别(这个顺序影响同一个header的不同操作)  

 2.在1中的每一个阶段内先执行remove，后执行set、add  

#### 3.21.10.2 **NJet HTTP**头操作实现

请求头操作实现：

<table style="width: 607px;">
<tbody>
<tr>
<td style="width: 179.297px;">
<p>操作类型</p>
</td>
<td style="width: 332.562px;">
<p>指令</p>
</td>
<td style="width: 73.1406px;">
<p>备注</p>
</td>
</tr>
<tr>
<td style="width: 179.297px;">
<p>删除请求头</p>
</td>
<td style="width: 332.562px;">
<p>proxy_set_header key ""</p>
</td>
<td style="width: 73.1406px;">
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 179.297px;">
<p>新增或重写请求头</p>
</td>
<td style="width: 332.562px;">
<p>proxy_set_header key value</p>
</td>
<td style="width: 73.1406px;">
<p>&nbsp;</p>
</td>
</tr>
</tbody>
</table>

响应头操作实现：删除请求头proxy_set_header key "",新增或重写头proxy_set_header key value

<table style="width: 693px;">
<tbody>
<tr>
<td style="width: 185.266px;">
<p>操作类型</p>
</td>
<td style="width: 267.391px;">
<p>指令</p>
</td>
<td style="width: 218.344px;">
<p>备注</p>
</td>
</tr>
<tr>
<td style="width: 185.266px;">
<p>删除响应头</p>
</td>
<td style="width: 267.391px;">
<p>proxy_hide_header key</p>
</td>
<td style="width: 218.344px;">
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 185.266px;">
<p>新增或重写响应头</p>
</td>
<td style="width: 267.391px;">
<p>proxy_hide_header key</p>
<p>add_header key value</p>
<p>&nbsp;</p>
</td>
<td style="width: 218.344px;">
<p>add_header 不会删除上游服务中的响应头，所以先删除头</p>
</td>
</tr>
</tbody>
</table>

NJet添加响应头后，NJet不会再删除此头(通过Istio策略使njet添加头后，再删除此头的场景(目的地级别添加了头，路由级别又删除此头)，当然实际应该不会这样做，只是Istio支持这样的配置)，NJet只会删除上游服务返回的响应头。请求头同理，NJet只会删除客户端携带的请求头。

如果set、add配置了相同的key，最终会有两个相同的key。NJet只会重写请求和响应中的头，自己添加的请求头和响应头不会重写。

#### 3.21.10.3 **Istio HTTP**头操作示例

##### 3.21.10.3.1 **环境准备**

测试前需要部署bookinfo应用和service-b服务(nginx)，注入sidecar。service-b服务是为了修改上游真实服务的响应头。

在service-b服务容器中修改nginx配置/etc/nginx/nginx.conf，增加响应头，然后reload。

![image-20230710143634753](https://gitee.com/gebona/picture/raw/master/202307101436591.png)

本测试案例为基于一个高级路由的header操作，此场景较复杂(携带end-user:jason且为GET方法被路由到reviews v2和details v1，携带end-user:jason且为POST方法被路由到reviews v3和service-b，否则请求被路由到service-b)，创建如下VirtualService配置：

```
YAML
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - headers:
      request:
        add:
          huidu2: huidu2
    match:
    - headers:
        end-user:
          exact: jason
      method:
        exact: GET
      uri:
        exact: /reviews/0
    route:
    - destination:
        host: reviews
        subset: v2
      headers:
        request:
          add:
            test: $host
          remove:
          - end-user
      weight: 50
    - destination:
        host: details
        subset: v1
      weight: 50
  - headers:
      response:
        add:
          test: testnew
        set:
          huidu3: huidu3
    match:
    - method:
        exact: POST
      uri:
        exact: /reviews/0
    route:
    - destination:
        host: service-b
        port:
          number: 6080
      headers:
        response:
          add:
            backend: service-b 
      weight: 60
    - destination:
        host: reviews
        subset: v3
      weight: 40
      headers:
        response:
          add:
            backend: reviewsv3
  - headers:
      response:
        remove:
        - hello
    match:
    - uri:
        exact: /reviews/0
    route:
    - destination:
        host: service-b
        port:
          number: 6080
```

##### 3.21.10.3.2 **Request** **头测试**

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
 curl http://reviews:9080/reviews/0 -H "end-user:jason" -H "test:666" -v
```

返回结果如下图所示，可知，与期望一致(流量被路由到reviews v2，有时候会跑到details，多执行几遍)。

![image-20230710143747080](https://gitee.com/gebona/picture/raw/master/202307101437787.png)

![image-20230710143806400](https://gitee.com/gebona/picture/raw/master/202307101438115.png)

**在reviews v2 sidecar容器中抓包，观察请求头修改情况**

![image-20230710143903229](https://gitee.com/gebona/picture/raw/master/202307101439109.png)

抓包可知，客户端的test头被njet重写为$host，新增huidu2头，删除end-user头。

**在details v1 sidecar容器中抓包，观察请求头修改情况**

![image-20230710143951237](https://gitee.com/gebona/picture/raw/master/202307101440938.png)

抓包可知，客户端的test头和end-user头被没被原样传到后端，新增huidu2头。

##### 3.21.10.3.3 **Response** **头测试**

**携带end-user:jason header，方法为POST**

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
  curl http://reviews:9080/reviews/0 -H "end-user:jason" -X POST -v
```

返回结果如下图所示，可知，与期望一致。

![image-20230710144125867](https://gitee.com/gebona/picture/raw/master/202307101441546.png)

service-b服务返回的test头被重写为testnew，且新增了backend和huidu3头，hello头保持原样传输给curl客户端。

![image-20230710144150022](https://gitee.com/gebona/picture/raw/master/202307101441042.png)

![image-20230710144157887](https://gitee.com/gebona/picture/raw/master/202307101441209.png)

由截图可知流量到达reviews v3，只是接口不支持POST。

Reviews v3新增了backend和huidu3、test头，reviews v3服务本身不会返回响应头。

**不携带end-user:jason header**

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
  curl http://reviews:9080/reviews/0 -v
```

返回结果如下图所示，可知，与期望一致。

![image-20230710144306530](https://gitee.com/gebona/picture/raw/master/202307101443776.png)

由图可知，请求被路由到service-b，service-b服务返回的test头保持原样传输给curl客户端，hello头被移除。

### 3.21.11 **HTTP**跨域资源共享

#### 3.20.11.1 **功能描述**

跨域资源共享(Cross-Origin Resource Sharing, CORS)是一种基于http头的机制，允许跨域共享响应(sharing responses cross-origin)，它允许服务器说明除自己以外的任何源(domain, scheme, or port)，这些源可以加载此服务器的资源。CORS协议由一组报头组成，这些报头表示响应是否可以跨源共享，响应头中填充各种header来指示资源的跨域约束。CORS依赖于一种机制(浏览器向托管跨源资源的服务器发出“preflight”请求)，通过该机制，以检查服务器是否允许实际请求。在preflight中，浏览器发送指示HTTP方法的头和将在实际请求中使用的头，服务端确定它是否可以在这些条件下接受请求。preflight请求由http OPTIONS方法触发。

Istio sidecar njet支持istio 控制面HTTP CORS规则，目前支持的CORS规则情况如下表所述：

<table style="width: 678px;">
<tbody>
<tr>
<td style="width: 140.422px;">
<p>字段名称</p>
</td>
<td style="width: 129.391px;">
<p>字段类型</p>
</td>
<td style="width: 152.469px;">
<p>Istio njet sidecar支持情况</p>
</td>
<td style="width: 227.719px;">
<p>备注</p>
</td>
</tr>
<tr>
<td style="width: 140.422px;">
<p>allowOrigins</p>
<p>&nbsp;</p>
</td>
<td style="width: 129.391px;">
<p><a href="https://istio.io/v1.13/docs/reference/config/networking/virtual-service/#StringMatch">StringMatch[]</a></p>
</td>
<td style="width: 152.469px;">
<p>支持exact、prefix(大小写不敏感)、regex(大小写不敏感)</p>
<p>&nbsp;</p>
</td>
<td style="width: 227.719px;">
<p>匹配源(跨域请求对应的源)规则，匹配成功后，启用CORS，服务会应用限制策略到跨域的请求，否则跨域请求不受跨域限制</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 140.422px;">
<p>allowMethods</p>
</td>
<td style="width: 129.391px;">
<p>string[]</p>
</td>
<td style="width: 152.469px;">
<p>支持</p>
</td>
<td style="width: 227.719px;">
<p>允许访问资源的http 方法，启用cors后，server会返回 Access-Control-Allow-Methods 头，内容由allowMethods填充</p>
</td>
</tr>
<tr>
<td style="width: 140.422px;">
<p>allowHeaders</p>
</td>
<td style="width: 129.391px;">
<p>string[]</p>
</td>
<td style="width: 152.469px;">
<p>支持</p>
</td>
<td style="width: 227.719px;">
<p>指示在发出实际请求时可以使用哪些HTTP头</p>
</td>
</tr>
<tr>
<td style="width: 140.422px;">
<p>exposeHeaders</p>
</td>
<td style="width: 129.391px;">
<p>string[]</p>
</td>
<td style="width: 152.469px;">
<p>支持</p>
</td>
<td style="width: 227.719px;">
<p>允许浏览器访问的HTTP标头列表。</p>
</td>
</tr>
<tr>
<td style="width: 140.422px;">
<p>maxAge</p>
</td>
<td style="width: 129.391px;">
<p><a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#duration">Duration</a></p>
</td>
<td style="width: 152.469px;">
<p>支持</p>
</td>
<td style="width: 227.719px;">
<p>指定preflight请求的结果可以缓存多长时间。</p>
</td>
</tr>
<tr>
<td style="width: 140.422px;">
<p>allowCredentials</p>
</td>
<td style="width: 129.391px;">
<p>bool</p>
</td>
<td style="width: 152.469px;">
<p>支持</p>
</td>
<td style="width: 227.719px;">
<p>指明调用者是否需要使用凭证发送实际请求</p>
</td>
</tr>
</tbody>
</table>

Istio 控制面HTTP CORS接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“http跨域资源共享”章节。

CORS详细介绍参考如下链接：https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

#### 3.21.11.2 **NJet HTTP CORS**实现

通过获取origin头，及获取HTTP 方法为OPTIOINS的请求，来判定是否启用CORS。匹配成功后，添加istio配置的响应头的值，并返回204。CORS定义的响应头如下：

Access-Control-Allow-Origin

Access-Control-Allow-Headers

Access-Control-Allow-Methods

Access-Control-Max-Age

Access-Control-Allow-Credentials

Access-Control-Expose-Headers

 Access-Control-Allow-Methods  默认值为"GET, PUT, POST, DELETE, PATCH,  OPTIONS"，Access-Control-Max-Age默认值为"86400"。  

#### 3.21.11.3 **Istio HTTP CORS** **示例**

本测试案例为基于一个高级路由的CORS，携带end-user:jason的请求被路由到reviews v3和details v1，否则请求被路由到reviews v2，创建如下VirtualService配置：

```
YAML
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: default
spec:
  hosts:
  - reviews
  http:
  - corsPolicy:
      allowCredentials: true
      allowHeaders:
      - nihao
      - nihaollll
      allowMethods:
      - POST
      - PUT
      allowOrigins:
      - prefix: hello
      exposeHeaders:
      - export
      maxAge: 2h
    match:
    - headers:
        end-user:
          exact: jason
      uri:
        exact: /reviews/0
    route:
    - destination:
        host: details
        subset: v1
      weight: 60
    - destination:
        host: reviews
        subset: v3
      weight: 40
  - corsPolicy:
      allowCredentials: false
      allowOrigins:
      - regex: http://baidu.com*
      - regex: z+
      maxAge: 24h
    match:
    - uri:
        exact: /reviews/0
    route:
    - destination:
        host: reviews
        subset: v2
```

**不携带end-user:jason header**

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
 curl http://reviews:9080/reviews/0 -H "Origin: http://baidu.comlllll"    -X OPTIONS -v
 curl http://reviews:9080/reviews/0 -H "Origin: zzzz"    -X OPTIONS -v
```

返回结果如下图所示，可知，与期望一致。

![image-20230710145314828](https://gitee.com/gebona/picture/raw/master/202307101453980.png)

Access-Control-Allow-Methods istio vs中未配置，给与默认值。

**携带end-user:jason header**

在reviews v3 pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
  curl http://reviews:9080/reviews/0 -H "Origin: hello123"  -H "end-user:jason"  -X OPTIONS -v
```

返回结果如下图所示，可知，与期望一致。

![image-20230710145414596](https://gitee.com/gebona/picture/raw/master/202307101454289.png)

由图可知，返回CORS 响应头与vs中的一致。

### 3.21.12 **请求身份认证**(JWT)

#### 3.20.12.1 **功能描述**

验证附加到请求(所有请求)上的凭据，如果请求包含无效的(基于配置的认证规则)认证信息将被拒绝，不包含任何身份验证凭据的请求将被接受，但意味着没有任何经过身份验证的标识。istio支持使用JWT进行请求级别的身份认证，JWT认证规则应用于处理劫持的入口流量的Listener(listen 15006端口)。

Istio sidecar NJet支持istio 控制面JWT规则，目前支持的JWT规则情况如下表所述：

<table style="width: 700px;">
<tbody>
<tr>
<td style="width: 198.578px;">
<p>字段名称</p>
</td>
<td style="width: 136.406px;">
<p>字段类型</p>
</td>
<td style="width: 156.453px;">
<p>Istio njet sidecar支持情况</p>
</td>
<td style="width: 180.562px;">
<p>备注</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>issuer</p>
</td>
<td style="width: 136.406px;">
<p>string</p>
<p>&nbsp;</p>
</td>
<td style="width: 156.453px;">
<p>不支持</p>
</td>
<td style="width: 180.562px;">
<p>不校验JWT中的iss信息</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>audiences</p>
</td>
<td style="width: 136.406px;">
<p>string[]</p>
</td>
<td style="width: 156.453px;">
<p>不支持</p>
</td>
<td style="width: 180.562px;">
<p>不校验JWT中的aud信息</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>jwksUri</p>
</td>
<td style="width: 136.406px;">
<p>string</p>
</td>
<td style="width: 156.453px;">
<p>支持</p>
</td>
<td style="width: 180.562px;" rowspan="2">
<p>二选一，istiod会通过jwksUri获取jwks</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>jwks</p>
</td>
<td style="width: 136.406px;">
<p>string</p>
</td>
<td style="width: 156.453px;">
<p>支持</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>fromHeaders</p>
</td>
<td style="width: 136.406px;">
<p><a href="https://istio.io/v1.13/docs/reference/config/security/jwt/#JWTHeader">JWTHeader[]</a></p>
</td>
<td style="width: 156.453px;">
<p>不支持</p>
</td>
<td style="width: 180.562px;" rowspan="2">
<p>token从标准的Authorization头中获取，以 Bearer 为前缀</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>fromParams</p>
</td>
<td style="width: 136.406px;">
<p>string[]</p>
</td>
<td style="width: 156.453px;">
<p>不支持</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>outputPayloadToHeader</p>
</td>
<td style="width: 136.406px;">
<p>string</p>
</td>
<td style="width: 156.453px;">
<p>不支持</p>
</td>
<td style="width: 180.562px;">
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td style="width: 198.578px;">
<p>forwardOriginalToken</p>
</td>
<td style="width: 136.406px;">
<p>bool</p>
</td>
<td style="width: 156.453px;">
<p>支持</p>
</td>
<td style="width: 180.562px;">
<p>原始token始终被转发到上游真实服务</p>
</td>
</tr>
</tbody>
</table>

Istio支持多条jwt规则配置，他们是OR的关系，如果其中任何一个通过，则结果通过。不包含JWT的请求将被接受。  

Istio 控制面请求身份认证接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“请求身份认证(Request authentication(JWT))”章节。

#### 3.21.12.2 **NJet sidecar Request authentication**实现

NJet sidecar 15006处理器通过识别原始目的地端口，来匹配是否需要针对请求进行jwt校验，如果此端口需要进行jwt校验，则此请求会由内部的http proxy(监听8086端口)处理，否则15006处理器自己处理。

Http proxy(8086)在处理请求时，原始目的地是通过x-njet-original-dst-host http头获取的。x-njet-original-dst-host由客户端sidecar执行upstream 负载均衡，最终选择的upstream_addr赋值。

![image-20230710145746364](https://gitee.com/gebona/picture/raw/master/202307101457881.png)

![image-20230710145801859](https://gitee.com/gebona/picture/raw/master/202307101458419.png)

 

  启用了jwt认证，当请求没有携带JWT，会认证失败。  

  NJet  sidecar 会校验JWT的签名和有效期。仅支持RSA算法。  

  如果Istio 配置了多条规则，NJet sidecar 只会使用第一条规则进行jwt校验。不支持多条规则OR。  

#### 3.21.12.3 **NJet sidecar Request authentication** **示例**

解析JWT的工具如下：

[该类型的内容暂不支持下载]

创建如下VirtualService：

```
YAML
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
      uri:
        exact: /reviews/0
    route:
    - destination:
        host: reviews
        subset: v3
  - match:
    - uri:
        exact: /reviews/0
    route:
    - destination:
        host: reviews
        subset: v2
```

##### 3.21.12.3.1 **jwksUri** **测试**

参照istio官方测试用例：

[该类型的内容暂不支持下载]

创建如下RequestAuthentication配置：

```
YAML
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: reviews
spec:
  jwtRules:
  - issuer: testing@secure.istio.io
    jwksUri: https://raw.githubusercontent.com/istio/istio/release-1.13/security/tools/jwt/samples/jwks.json
  selector:
    matchLabels:
      app: reviews
```

上面配置中指定的jwksUri为istio官方提供的，与之匹配的jwt为https://raw.githubusercontent.com/istio/istio/release-1.13/security/tools/jwt/samples/demo.jwt 

在details pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
//有效的token
curl   http://reviews:9080/reviews/0 -H "end-user:jason" -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjQ2ODU5ODk3MDAsImZvbyI6ImJhciIsImlhdCI6MTUzMjM4OTcwMCwiaXNzIjoidGVzdGluZ0BzZWN1cmUuaXN0aW8uaW8iLCJzdWIiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyJ9.CfNnxWP2tcnR9q0vxyxweaF3ovQYHYZl82hAUsn21bwQd9zP7c-LS9qd_vpdLG4Tn1A15NxfCjp5f7QNBUo-KC9PJqYpgGbaXhaGx7bEdFWjcwv3nZzvc7M__ZpaCERdwU7igUmJqYGBYQ51vr2njU9ZimyKkfDe3axcyiBZde7G6dabliUosJvvKOPcKIWPccCgefSj_GNfwIip3-SsFdlR7BtbVUcqR-yv-XOxJ3Uc1MI0tz3uMiiZcyPV7sNCU4KRnemRIMHVOfuvHsU60_GhGbiSFzgPTAa9WTltbnarTbxudb_YEOx12JiwYToeX0DCPb43W1tzIBxgm8NxUg"
```

返回结果如下图所示，可知，与期望一致(返回/reviews/0接口的内容)。

![image-20230710150125755](https://gitee.com/gebona/picture/raw/master/202307101501459.png)

```
Shell
//无效的token
curl   http://reviews:9080/reviews/0 -H "end-user:jason" -H "Authorization: Bearer asdasdad"
```

返回结果如下图所示，可知，与期望一致(返回401)。

![image-20230710150154896](https://gitee.com/gebona/picture/raw/master/202307101501392.png)

```
Shell
//没有token
curl   http://reviews:9080/reviews/0 -H "end-user:jason" 
```

返回结果如下图所示，可知，与期望一致(返回401)。

![image-20230710150221607](https://gitee.com/gebona/picture/raw/master/202307101502989.png)

##### 3.21.12.3.2 **jwks** **测试**

创建如下RequestAuthentication配置：

```
YAML
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: reviews
spec:
  jwtRules:
  - issuer: testing@secure.istio.io
    jwks: '{ "keys":[ {"e":"AQAB","kid":"DHFbpoIUqrY8t2zpA2qXfCmr5VO5ZEr4RzHU_-envvQ","kty":"RSA","n":"xAE7eB6qugXyCAG3yhh7pkDkT65pHymX-P7KfIupjf59vsdo91bSP9C8H07pSAGQO1MV_xFj9VswgsCg4R6otmg5PV2He95lZdHtOcU5DXIg_pbhLdKXbi66GlVeK6ABZOUW3WYtnNHD-91gVuoeJT_DwtGGcp4ignkgXfkiEm4sw-4sfb4qdt5oLbyVpmW6x9cfa7vs2WTfURiCrBoUqgBo_-4WTiULmmHSGZHOjzwa8WtrtOQGsAFjIbno85jp6MnGGGZPYZbDAa_b3y5u-YpW7ypZrvD8BgtKVjgtQgZhLAGezMt0ua3DRrWnKqTZ0BJ_EyxOGuHJrLsn00fnMQ"}]}'
  selector:
    matchLabels:
      app: reviews
```

上面配置中指定的jwks为istio官方提供的，链接如下：https://raw.githubusercontent.com/istio/istio/release-1.13/security/tools/jwt/samples/jwks.json

与之匹配的jwt为

https://raw.githubusercontent.com/istio/istio/release-1.13/security/tools/jwt/samples/demo.jwt

在details pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
//有效的token
curl   http://reviews:9080/reviews/0 -H "end-user:jason" -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjQ2ODU5ODk3MDAsImZvbyI6ImJhciIsImlhdCI6MTUzMjM4OTcwMCwiaXNzIjoidGVzdGluZ0BzZWN1cmUuaXN0aW8uaW8iLCJzdWIiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyJ9.CfNnxWP2tcnR9q0vxyxweaF3ovQYHYZl82hAUsn21bwQd9zP7c-LS9qd_vpdLG4Tn1A15NxfCjp5f7QNBUo-KC9PJqYpgGbaXhaGx7bEdFWjcwv3nZzvc7M__ZpaCERdwU7igUmJqYGBYQ51vr2njU9ZimyKkfDe3axcyiBZde7G6dabliUosJvvKOPcKIWPccCgefSj_GNfwIip3-SsFdlR7BtbVUcqR-yv-XOxJ3Uc1MI0tz3uMiiZcyPV7sNCU4KRnemRIMHVOfuvHsU60_GhGbiSFzgPTAa9WTltbnarTbxudb_YEOx12JiwYToeX0DCPb43W1tzIBxgm8NxUg"
```

返回结果如下图所示，可知，与期望一致(返回/reviews/0接口的内容)。

![image-20230710150504924](https://gitee.com/gebona/picture/raw/master/202307101505508.png)

```
Shell
//无效的token
curl   http://reviews:9080/reviews/0 -H "end-user:jason" -H "Authorization: Bearer 123456"
```

返回结果如下图所示，可知，与期望一致(返回401)。

![image-20230710150542696](https://gitee.com/gebona/picture/raw/master/202307101505619.png)

```
Shell
//没有token
curl   http://reviews:9080/reviews/0 -H "end-user:jason" 
```

返回结果如下图所示，可知，与期望一致(返回401)。

![image-20230710150623562](https://gitee.com/gebona/picture/raw/master/202307101506122.png)

##### 3.21.12.3.3 **Jwt** **有效期测试**

此测试案例中，我们需要自己生成一个JWT，上面使用的JWT有效期到2118年。RSA密钥依然使用官方的，所以jwks是一样的。利用官方提供的python脚本，生成一个有效期为3分钟的JWT。

![image-20230710150704232](https://gitee.com/gebona/picture/raw/master/202307101507929.png)

创建如下RequestAuthentication配置：

```
YAML
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: reviews
spec:
  jwtRules:
  - issuer: testing@secure.istio.io
    jwks: '{ "keys":[ {"e":"AQAB","kid":"DHFbpoIUqrY8t2zpA2qXfCmr5VO5ZEr4RzHU_-envvQ","kty":"RSA","n":"xAE7eB6qugXyCAG3yhh7pkDkT65pHymX-P7KfIupjf59vsdo91bSP9C8H07pSAGQO1MV_xFj9VswgsCg4R6otmg5PV2He95lZdHtOcU5DXIg_pbhLdKXbi66GlVeK6ABZOUW3WYtnNHD-91gVuoeJT_DwtGGcp4ignkgXfkiEm4sw-4sfb4qdt5oLbyVpmW6x9cfa7vs2WTfURiCrBoUqgBo_-4WTiULmmHSGZHOjzwa8WtrtOQGsAFjIbno85jp6MnGGGZPYZbDAa_b3y5u-YpW7ypZrvD8BgtKVjgtQgZhLAGezMt0ua3DRrWnKqTZ0BJ_EyxOGuHJrLsn00fnMQ"}]}'
  selector:
    matchLabels:
      app: reviews
```

在details pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl   http://reviews:9080/reviews/0 -H "end-user:jason" -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODUwMDEwMjQsImlhdCI6MTY4NTAwMDg0NCwiaXNzIjoibmV3LWlzc3VlckBzZWN1cmUuaXN0aW8uaW8iLCJzdWIiOiJuZXctaXNzdWVyQHNlY3VyZS5pc3Rpby5pbyJ9.qi6R-vN_NqssKEihYxDGxYAGWbRSg0j6iVE1YqJB5clG2rSuk_xnc093wxxyAYsjM_qOnfeZN1LETCKa94xrXDxQfPG5YJTnuKBLqx4PRx5YH6PlLaZ8XH2O67qhN73CXDNh1PQgQVzko-d-RmkxkIVCzjoiVu01cZ1bvOIh7cc-wBR9-XZ6Fn-mSI9zB1J-eZ6tDGCYYxHN7pDvi5REPU00hhXIkX06MSLWYQqmWCGuMxxs5eVOScHC_0aHd0OK-b-csfFe1AHJuP23G_N8M_vYs9d81_Mdif57bhZZ8zrghDgtr3Gv2CKvFhSIc3UlxdajYIvuosbHVWTVU3jFOw"
```

返回结果如下图所示，可知，与期望一致(返回/reviews/0接口的内容)。

![image-20230710150803560](https://gitee.com/gebona/picture/raw/master/202307101508468.png)

等待3分钟后，再次请求

```
Shell
curl   http://reviews:9080/reviews/0 -H "end-user:jason" -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODUwMDEwMjQsImlhdCI6MTY4NTAwMDg0NCwiaXNzIjoibmV3LWlzc3VlckBzZWN1cmUuaXN0aW8uaW8iLCJzdWIiOiJuZXctaXNzdWVyQHNlY3VyZS5pc3Rpby5pbyJ9.qi6R-vN_NqssKEihYxDGxYAGWbRSg0j6iVE1YqJB5clG2rSuk_xnc093wxxyAYsjM_qOnfeZN1LETCKa94xrXDxQfPG5YJTnuKBLqx4PRx5YH6PlLaZ8XH2O67qhN73CXDNh1PQgQVzko-d-RmkxkIVCzjoiVu01cZ1bvOIh7cc-wBR9-XZ6Fn-mSI9zB1J-eZ6tDGCYYxHN7pDvi5REPU00hhXIkX06MSLWYQqmWCGuMxxs5eVOScHC_0aHd0OK-b-csfFe1AHJuP23G_N8M_vYs9d81_Mdif57bhZZ8zrghDgtr3Gv2CKvFhSIc3UlxdajYIvuosbHVWTVU3jFOw"
```

返回结果如下图所示，可知，与期望一致(返回401)。

![image-20230710150904298](https://gitee.com/gebona/picture/raw/master/202307101509562.png)

查看NJet日志，401是由于JWT过期导致

![image-20230710150922676](https://gitee.com/gebona/picture/raw/master/202307101509301.png)

### 3.21.13 **Access Log**

#### 3.21.13.1 **功能描述**

Istio可以生成每个请求的完整记录，包括源和目标元数据。这些信息使操作人员能够审计服务行为，粒度为单个工作负载实例。可以以一组可配置的格式为服务流量生成访问日志。支持全局配置(mesh级别)，支持针对单个工作负载、整个namespace范围配置。istio提供了两种方式使能access log，如下：

•     通过Telemetry CR配置

属性字段为Telemetry.accessLogging(AccessLogging[]类型)，定义文档：https://istio.io/v1.13/docs/reference/config/telemetry/

•     通过Mesh config配置

由istio-system namespace下的istio ConfigMap资源对象进行配置。此方式为全局配置，整个mesh中的工作负载应用相同配置策略。定义文档：https://istio.io/v1.13/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig

建议使用Telemetry 。

#### 3.21.13.2 **Telemetry CR**配置

Telemetry CR方式需要在istio ConfigMap创建extensionProviders，然后在Telemetry 资源对象中引用Provider。istio支持的accesslog extensionProviders有如下几种类型：

![image-20230710151153306](https://gitee.com/gebona/picture/raw/master/202307101511922.png)

**[该类型的内容暂不支持下载]**

  NJet  sidecar支持[EnvoyFileAccessLogProvider](https://istio.io/v1.13/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-ExtensionProvider-EnvoyFileAccessLogProvider)类型。  Istio 默认创建了名为envoy的[EnvoyFileAccessLogProvider ](https://istio.io/v1.13/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-ExtensionProvider-EnvoyFileAccessLogProvider)类型的ExtensionProvider，在Teltmetry可以直接使用。envoy ExtensionProvider的path为/dev/out，logFormat为Istio 默认LogFormat，MeshConfig中会介绍。  

**extensionProviders** **创建示例**

![image-20230710152306895](https://gitee.com/gebona/picture/raw/master/202307101523957.png)

**Telemetry 资源对象中引用Provider示例**

![image-20230710152351460](https://gitee.com/gebona/picture/raw/master/202307101523942.png)

Istio sidecar NJet支持Istio 控制面通过Telemetry 配置的accesslog规则，目前支持的规则情况如下表所述：

<table>
<tbody>
<tr>
<td width="155">
<p>字段名称</p>
</td>
<td width="195">
<p>字段类型</p>
</td>
<td width="186">
<p>Istio njet sidecar支持情况</p>
</td>
</tr>
<tr>
<td width="155">
<p>providers</p>
</td>
<td width="195">
<p><a href="https://istio.io/v1.13/docs/reference/config/telemetry/#ProviderRef">ProviderRef[]</a></p>
</td>
<td width="186">
<p>支持</p>
</td>
</tr>
<tr>
<td width="155">
<p>disabled</p>
</td>
<td width="195">
<p><br /><a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#boolvalue">BoolValue</a></p>
</td>
<td width="186">
<p>支持</p>
</td>
</tr>
<tr>
<td width="155">
<p>filter</p>
</td>
<td width="195">
<p><br /><a href="https://istio.io/v1.13/docs/reference/config/telemetry/#AccessLogging-Filter">Filter</a></p>
</td>
<td width="186">
<p>不支持</p>
</td>
</tr>
</tbody>
</table>

Istio 控制面AccessLog接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“访问日志(Access Logs)”章节。

NJet也支持多条provider

#### 3.21.13.3 **Mesh config**配置

支持accessLogFile(文件位置)、accessLogFormat(日志格式)、accessLogEncoding(编码格式)的配置。

accessLogFile为空，意味着禁用accessLog。istio部署后accessLogFile为空。

默认accessLogFormat如下：

[%START_TIME%] \"%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%\" %RESPONSE_CODE% %RESPONSE_FLAGS% %RESPONSE_CODE_DETAILS% %CONNECTION_TERMINATION_DETAILS%

\"%UPSTREAM_TRANSPORT_FAILURE_REASON%\" %BYTES_RECEIVED% %BYTES_SENT% %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% \"%REQ(X-FORWARDED-FOR)%\" \"%REQ(USER-AGENT)%\" \"%REQ(X-REQUEST-ID)%\"

\"%REQ(:AUTHORITY)%\" \"%UPSTREAM_HOST%\" %UPSTREAM_CLUSTER% %UPSTREAM_LOCAL_ADDRESS% %DOWNSTREAM_LOCAL_ADDRESS% %DOWNSTREAM_REMOTE_ADDRESS% %REQUESTED_SERVER_NAME% %ROUTE_NAME%\n

![image-20230710152629788](https://gitee.com/gebona/picture/raw/master/202307101526216.png)

accessLogEncoding支持TEXT、JSON两种，默认为TEXT，LogFormat为默认accessLogFormat。JSON编码，LogFormat如下：

![image-20230710152652900](https://gitee.com/gebona/picture/raw/master/202307101526059.png)

Istio sidecar NJet支持Istio 控制面通过MeshConfig配置的accesslog规则，目前支持的规则情况如下表所述：

<table style="width: 693px;">
<tbody>
<tr>
<td style="width: 219.641px;">
<p>字段名称</p>
</td>
<td style="width: 218.641px;">
<p>字段类型</p>
</td>
<td style="width: 232.719px;">
<p>Istio njet sidecar支持情况</p>
</td>
</tr>
<tr>
<td style="width: 219.641px;">
<p>accessLogFormat</p>
</td>
<td style="width: 218.641px;">
<p>string</p>
</td>
<td style="width: 232.719px;">
<p>支持</p>
</td>
</tr>
<tr>
<td style="width: 219.641px;">
<p>accessLogEncoding</p>
</td>
<td style="width: 218.641px;">
<p><br /><a href="https://istio.io/v1.13/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-AccessLogEncoding">AccessLogEncoding</a>&nbsp; 枚举</p>
</td>
<td style="width: 232.719px;">
<p>支持</p>
</td>
</tr>
<tr>
<td style="width: 219.641px;">
<p>accessLogFile</p>
</td>
<td style="width: 218.641px;">
<p><br />string</p>
</td>
<td style="width: 232.719px;">
<p>支持</p>
</td>
</tr>
</tbody>
</table>

Istio 控制面AccessLog接口定义参照[Istio sidecar proxy功能定义梳理](https://xw1ei7mxto.feishu.cn/wiki/wikcnYIgJKrERhrs71T135g85tx) 的“访问日志(Access Logs)”章节。

#### 3.21.13.4 Open**NJet sidecar AccessLog**实现

OpenNJet 使用access_log 与 log_format 指令实现。OpenNJet sidecar支持的istio logFormat如下：

<table>
<tbody>
<tr>
<td width="262">
<p>Envoy Log operator</p>
</td>
<td colspan="2" width="121">
<p>njet accessLog</p>
</td>
<td width="169">
<p>log示例</p>
</td>
</tr>
<tr>
<td width="262">
<p>&nbsp;</p>
</td>
<td width="58">
<p>http</p>
</td>
<td width="63">
<p>stream</p>
</td>
<td width="169">
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td width="262">
<p>%START_TIME%</p>
<p>&nbsp;</p>
</td>
<td width="58">
<p>$time_local</p>
</td>
<td width="63">
<p>$time_local</p>
</td>
<td width="169">
<p>[2020-11-25T21:26:18.409Z]</p>
<p>&nbsp;</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%</p>
</td>
<td width="58">
<p>$request</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>GET /status/418 HTTP/1.1</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(:METHOD)%</p>
</td>
<td width="58">
<p>$request_method</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>GET</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(X-ENVOY-ORIGINAL-PATH?:PATH)%</p>
</td>
<td width="58">
<p>$request_uri</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>/status/418</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(X-ENVOY-ORIGINAL-PATH)%</p>
</td>
<td width="58">
<p>$request_uri</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>/status/418</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(:PATH)%</p>
</td>
<td width="58">
<p>$request_uri</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>/status/418</p>
</td>
</tr>
<tr>
<td width="262">
<p>%PROTOCOL%</p>
</td>
<td width="58">
<p>$server_protocol</p>
</td>
<td width="63">
<p>$protocol</p>
</td>
<td width="169">
<p>HTTP/1.1</p>
</td>
</tr>
<tr>
<td width="262">
<p>%RESPONSE_CODE%</p>
</td>
<td width="58">
<p>$status</p>
</td>
<td width="63">
<p>$status</p>
</td>
<td width="169">
<p>418</p>
</td>
</tr>
<tr>
<td width="262">
<p>%RESPONSE_FLAGS</p>
</td>
<td width="58">
<p>-</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>-</p>
</td>
</tr>
<tr>
<td width="262">
<p>%RESPONSE_CODE_DETAILS%</p>
</td>
<td width="58">
<p>-</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>via_upstream</p>
</td>
</tr>
<tr>
<td width="262">
<p>%CONNECTION_TERMINATION_DETAILS%</p>
</td>
<td width="58">
<p>-</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>-</p>
</td>
</tr>
<tr>
<td width="262">
<p>%UPSTREAM_TRANSPORT_FAILURE_REASON%%</p>
</td>
<td width="58">
<p>$upstream_status</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>-</p>
</td>
</tr>
<tr>
<td width="262">
<p>%BYTES_RECEIVED%</p>
</td>
<td width="58">
<p>$body_bytes_sent</p>
</td>
<td width="63">
<p>$bytes_sent</p>
</td>
<td width="169">
<p>0</p>
</td>
</tr>
<tr>
<td width="262">
<p>%BYTES_SENT%</p>
</td>
<td width="58">
<p>-</p>
</td>
<td width="63">
<p>$bytes_received</p>
</td>
<td width="169">
<p>135</p>
</td>
</tr>
<tr>
<td width="262">
<p>%DURATION%</p>
</td>
<td width="58">
<p>$request_time</p>
</td>
<td width="63">
<p>$session_time</p>
</td>
<td width="169">
<p>4</p>
</td>
</tr>
<tr>
<td width="262">
<p>%RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)%</p>
</td>
<td width="58">
<p>-</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>4</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(X-FORWARDED-FOR)%</p>
</td>
<td width="58">
<p>$proxy_add_x_forwarded_for</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>-</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(USER-AGENT)%</p>
</td>
<td width="58">
<p>$http_user_agent</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>curl/7.73.0-DEV</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(X-REQUEST-ID)%</p>
</td>
<td width="58">
<p>$request_id</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>84961386-6d84-929d-98bd-c5aee93b5c88</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQ(:AUTHORITY)%</p>
</td>
<td width="58">
<p>$http_host</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>httpbin:8000</p>
</td>
</tr>
<tr>
<td width="262">
<p>%UPSTREAM_HOST%</p>
</td>
<td width="58">
<p>$upstream_addr</p>
</td>
<td width="63">
<p>$upstream_addr</p>
</td>
<td width="169">
<p>10.44.1.27:80</p>
</td>
</tr>
<tr>
<td width="262">
<p>%UPSTREAM_CLUSTER%</p>
</td>
<td width="58">
<p>$proxy_host</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>outbound|8000||httpbin.foo.svc.cluster.local</p>
</td>
</tr>
<tr>
<td width="262">
<p>%UPSTREAM_LOCAL_ADDRESS%</p>
</td>
<td width="58">
<p>-</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>10.44.1.23:37652</p>
</td>
</tr>
<tr>
<td width="262">
<p>%DOWNSTREAM_LOCAL_ADDRESS%</p>
</td>
<td width="58">
<p>$server_addr:$server_port</p>
</td>
<td width="63">
<p>$njtmesh_dest</p>
<p>&nbsp;</p>
</td>
<td width="169">
<p>10.0.45.184:8000</p>
</td>
</tr>
<tr>
<td width="262">
<p>%DOWNSTREAM_REMOTE_ADDRESS%</p>
</td>
<td width="58">
<p>$remote_addr:$remote_port</p>
</td>
<td width="63">
<p>$remote_addr:$remote_port</p>
</td>
<td width="169">
<p>10.44.1.23:46520</p>
</td>
</tr>
<tr>
<td width="262">
<p>%REQUESTED_SERVER_NAME%</p>
</td>
<td width="58">
<p>$ssl_server_name</p>
</td>
<td width="63">
<p>$ssl_server_name</p>
</td>
<td width="169">
<p>-</p>
</td>
</tr>
<tr>
<td width="262">
<p>%ROUTE_NAME%</p>
</td>
<td width="58">
<p>-</p>
</td>
<td width="63">
<p>-</p>
</td>
<td width="169">
<p>default</p>
</td>
</tr>
</tbody>
</table>

Envoy sidecar Log operator定义：

**[该类型的内容暂不支持下载]**

Njet sidecar Log operator定义：

•     Http vs：

**[该类型的内容暂不支持下载]**

**[该类型的内容暂不支持下载]**

**[该类型的内容暂不支持下载]**

**[该类型的内容暂不支持下载]**

•     stream：

**[该类型的内容暂不支持下载]**

**[该类型的内容暂不支持下载]**

**[该类型的内容暂不支持下载]**

  Istio  envoy sidecar 出口流量处理器15001 中http vs accesslog与入口流量处理器 15006的accesslog使用相同的accesslog。njet sidecar 15001与15006的不同，前者使用的是http accesslog模块，后者是stream accesslog模块，两模块虽然指令一样，但是支持的变量不同。njet 目前支持的accessLogFormat是istio的默认accessLogFormat，envoy其他Log operator njet不支持。   

  accessLogFile为accesslog 存放的文件路径，由于istio sidecar中的进程用户id 为1337，1337目前可操作的路径为/var/log/njet/，所以accessLogFile可配置为“/var/log/njet/日志文件名称”，其他accessLogFile 1337没权限操作，配置不会生效。  

#### 3.21.13.5 OpenNJet sidecar AccessLog 示例

可以参照Istio官方测试用例：

**[该类型的内容暂不支持下载]**

  Using  Mesh Config测试中，不需要重新安装istio，直接修改Istio ConfigMap即可。重新安装需要注意，NJet Sidecar是定制化安装。  

##### 3.21.13.5.1 **Mesh config** 配置

###### 3.21.13.5.1.1 accessLogEncoding为TEXT

执行如下指令，修改Istio ConfigMap：

```
Shell
kubectl edit cm istio -n istio-system
```

![image-20230710153426211](https://gitee.com/gebona/picture/raw/master/202307101534388.png)

查看details pod中的NJet配置文件：

Http vs accesslog

![image-20230710153456042](https://gitee.com/gebona/picture/raw/master/202307101534112.png)

![image-20230710153504956](https://gitee.com/gebona/picture/raw/master/202307101535415.png)

15006 15001 Stream accesslog

![image-20230710153522399](https://gitee.com/gebona/picture/raw/master/202307101535374.png)

reviews、productpage中的accesslog配置与details一致。

创建如下VirtualService：

![image-20230710153543022](https://gitee.com/gebona/picture/raw/master/202307101535713.png)

上面match指明，携带end-user:jason请求头，且uri为/reviews/0的请求被路由到上游reviews v3服务。

在details pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0 -H "end-user:jason"
```

返回结果如下图所示：

![image-20230710153625464](https://gitee.com/gebona/picture/raw/master/202307101536389.png)

查看details pod 内sidecar容器的accesslog

![image-20230710153640173](https://gitee.com/gebona/picture/raw/master/202307101536178.png)

查看reviews v3 pod内sidecar容器的accesslog

![image-20230710153655930](https://gitee.com/gebona/picture/raw/master/202307101536457.png)

删除istio ConfigMap 中的accessLogFormat: "%PROTOCOL%"，使用istio 默认accessLogFormat，details与reviews v3的accesslog日志分别如下：

![image-20230710153711505](https://gitee.com/gebona/picture/raw/master/202307101537796.png)

![image-20230710153717953](https://gitee.com/gebona/picture/raw/master/202307101537326.png)

###### 3.21.13.5.1.2 accessLogEncoding 为 JSON

修改Istio ConfigMap：

![image-20230710153821929](https://gitee.com/gebona/picture/raw/master/202307101538556.png)

在details pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0 -H "end-user:jason"
```

返回结果如下图所示：

![image-20230710153900872](https://gitee.com/gebona/picture/raw/master/202307101539419.png)

看details pod 内sidecar容器的accesslog

![image-20230710154017751](https://gitee.com/gebona/picture/raw/master/202307101540030.png)

查看reviews v3 pod内sidecar容器的accesslog

![image-20230710154030655](https://gitee.com/gebona/picture/raw/master/202307101540527.png)

##### 3.21.13.5.2 **Telemetry CR**配置

###### 3.21.13.5.2.1 **workerload-level**

Telemetry CR为workerload级别，且[EnvoyFileAccessLogProvider](https://istio.io/v1.13/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-ExtensionProvider-EnvoyFileAccessLogProvider) logFormat为text和lables

修改Istio ConfigMap创建extensionProviders：

```
YAML
    extensionProviders:
    - name: file-accesslog-1
      envoyFileAccessLog:
        path: /dev/stdout
        logFormat:
          text: "protocol=%PROTOCOL% response-code=%RESPONSE_CODE% path=%REQ(:PATH)% upstream-host=%UPSTREAM_HOST% upstream-cluster=%UPSTREAM_CLUSTER%"
    - name: file-accesslog-2
      envoyFileAccessLog:
        path: /var/log/njet/access-test.log
        logFormat:
          labels:
            protocol: "%PROTOCOL%"
            response-code: "%RESPONSE_CODE%"
            path: "%REQ(:PATH)%"
            upstream-host: "%UPSTREAM_HOST%"
            upstream-cluster: "%UPSTREAM_CLUSTER%"
```

![image-20230710154140200](https://gitee.com/gebona/picture/raw/master/202307101541808.png)、

创建如下Telemetry ：

引用file-accesslog-1和file-accesslog-2 Provider

```
YAML
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: namespace-no-log
spec:
  accessLogging:
  - disabled: false
    providers:
    - name: file-accesslog-1
    - name: file-accesslog-2
  selector:
    matchLabels:
      app: reviews
```

查看review v3 njet配置http vs accesslog：

![image-20230710154235243](https://gitee.com/gebona/picture/raw/master/202307101542809.png)

![image-20230710154251631](https://gitee.com/gebona/picture/raw/master/202307101542102.png)

查看review v3 njet配置15001 15006 stream accesslog：

![image-20230710154311283](https://gitee.com/gebona/picture/raw/master/202307101543398.png)

查看review v2 v1 njet配置与v3配置一致，这里不进行截图展示了。 

查看details pod 内sidecar njet配置，可见依然是mesh config指定的JSON配置。

![image-20230710154332130](https://gitee.com/gebona/picture/raw/master/202307101543631.png)

**由上面配置可知，只有reviews pod中的njet 配置是namespace-no-log Telemetry中的配置。说明workerload-level级别生效。**

在details pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0 -H "end-user:jason"
```

返回结果如下图所示：

![image-20230710155017074](https://gitee.com/gebona/picture/raw/master/202307101550589.png)

查看details pod 内sidecar容器的accesslog

![image-20230710155031254](https://gitee.com/gebona/picture/raw/master/202307101550727.png)

查看reviews v3 pod内sidecar容器的accesslog

写到/var/log/njet/access-test.log的日志如下：

![image-20230710155046641](https://gitee.com/gebona/picture/raw/master/202307101550644.png)

同时，/dev/stdout也会有日志输出，且为自定义TEXT格式，通过如下命令可查看：

```
Shell
kubectl logs  reviews-v3-674898dfb-6d2rs -c istio-proxy  -f
```

![image-20230710155116335](https://gitee.com/gebona/picture/raw/master/202307101551009.png)

###### 3.21.13.5.2.2 **ns-level**

删除 workerload-level章节中，Telemetry 的selector。

在details pod中的业务容器中通过curl 访问reviews:9080 host，指令如下：

```
Shell
curl http://reviews:9080/reviews/0 -H "end-user:jason"
```

返回结果如下图所示：

![image-20230710155206203](https://gitee.com/gebona/picture/raw/master/202307101552852.png)

查看details pod内sidecar容器日志，写到/var/log/njet/access-test.log的日志如下：

![image-20230710155226559](https://gitee.com/gebona/picture/raw/master/202307101552776.png)

同时，/dev/stdout也会有日志输出，且为自定义TEXT格式，通过如下命令可查看：

```
Shell
kubectl logs details-v1-6cfd454d8-xfml4 -c istio-proxy -f
```

![image-20230710155249204](https://gitee.com/gebona/picture/raw/master/202307101552184.png)

### 3.21.14 OpenNJet vts**指标**

#### 3.21.14.1 **功能描述**

每个代理都会生成一组关于所有通过代理(入站和出站)的流量的丰富指标。代理还提供关于代理本身的管理功能的详细统计信息，包括配置和运行状况信息。

#### 3.21.14.2 **指标类型**

**Proxy** **级别指标**

目前NJet支持vts(virtual host traffic status )指标采集，prometheus通过访问njet 15020端口拉取vts指标，uri为/stats/prometheus。vts指标可查看文档[vts指标](https://xw1ei7mxto.feishu.cn/wiki/TSR4wSm3WilzyVkjONccPByWnng)。vts官方资料可查看下面所示网站。

**[该类型的内容暂不支持下载**]

**Service** **级别指标** **（暂不支持）**

```
不支持
```

#### 3.21.14.3 **Istio** **指标测试**

##### 3.21.14.3.1 **Proxy** **级别指标**

###### 3.21.14.3.1.1 **通过**prometheus查看**vts**指标

首先需要部署prometheus，Istio安装包提供了prometheus 部署文件。部署文件在Istio-install项目中，路径如下：

```
Shell
istio-1.13.3/samples/addons/prometheus.yaml
```

执行下面指令部署prometheus

```
Shell
kubectl apply -f istio-1.13.3/samples/addons/prometheus.yaml
```

执行如下命令，启用端口转发，这样可以在与k8s 节点可以互通的机器上通过浏览器查看指标

```
Shell
kubectl port-forward -n istio-system prometheus-7cc96d969f-j4775 --address 0.0.0.0 9090:9090
```

访问prometheus时，目的ip为执行上述指令的机器的IP。比如我的测试环境为192.168.40.151，访问prometheus的完整URL为：

```
Shell
http://192.168.40.151:9090/
```

出现如下页面，说明成功

![image-20230710160632334](https://gitee.com/gebona/picture/raw/master/202307101606099.png)

在productpage pod中的业务容器中通过curl 访问details:9080 host，指令如下：

```
Shell
curl http://details:9080/details/0
```

返回结果如下图所示

![image-20230710160701260](https://gitee.com/gebona/picture/raw/master/202307101607750.png)

这样会触发一次upstream的统计。在prometheus 页面中输入njet_vts_upstream_requests_total指标，进行查询，可以看到4xx统计值显示1。

![image-20230710160717504](https://gitee.com/gebona/picture/raw/master/202307101607605.png)

![image-20230710160724625](https://gitee.com/gebona/picture/raw/master/202307101607856.png)

prometheus 拉取指标后，会自动打上pod、namespace、app等标贴。

###### 3.21.14.3.1.2 **通过访问**sidecar 15020端口查看vts指标

在productpage pod中的业务容器中通过curl 访问，指令如下：

```
Shell
//返回prometheus格式的指标连表
curl http://127.0.0.1:15020/stats/prometheus
```

返回结果如下图所示：

![image-20230710160833200](https://gitee.com/gebona/picture/raw/master/202307101608685.png)

```
Shell
//返回html页面用来展示指标
http://127.0.0.1:15020/stats/format/html
//返回json格式用来展示指标
http://127.0.0.1:15020/stats/format/json
```

也可以使用kubectl port-forward 暴露productpage pod的15020端口，通过浏览器查看

```
Shell
 kubectl port-forward productpage-v1-64884d7c8d-qd78r  --address 0.0.0.0 15020:15020
```

![image-20230710160911213](https://gitee.com/gebona/picture/raw/master/202307101609727.png)



## 3.22 **慢启动**

### 3.22.1 **简介**

慢启动（Slow Start）功能是一种用于优化负载均衡和请求处理的机制。它旨在在系统启动或发生重启后逐渐增加服务的负载，以避免因同时接收大量请求而导致系统过载或崩溃的情况。

慢启动功能的原理是，在服务启动后，NGINX 会逐步增加对服务的流量引导，而不是立即将所有请求发送到后端服务器。这样可以有效地平滑过渡和控制负载，避免由于突然大量请求的到来而引起的性能问题。

### 3.22.2 **配置示例**

```
Bash
upstream test {
        server 127.0.0.1:7001 slow_start=30s;
        server 127.0.0.1:7002;
}
```

在上述示例中，slow_start参数设置为30s，表示在启动后的前30秒内逐步增加对服务器的流量引导。

￮    slow_start：设置慢启动的时间长度。可以使用s表示秒，m表示分钟，h表示小时。例如：slow_start=1m30s表示1分钟30秒的慢启动时间。



## 3.23 **组播集群**

### 3.23.1 Open**NJet** **集群通用配置**

#### 3.22.1.1 **Listen** 配置

需要配置d类组播ip， eg: listen 238.255.253.254:5555 udp;

#### 3.22.1.2 **指令配置：**

```
Bash
gossip zone=test:1m heartbeat_timeout=20s nodeclean_timeout=30s;
zone必须配置
heartbeat_timeout 选配，不配，默认是10s；配置小于1s,则设置为默认值10s        超时心跳时间
nodeclean_timeout 选配，不配, 默认是20s； 配置小于1s,则设置为默认值20s        节点清理时间，如果该时间间隔内没收到心跳的节点会从本集群删除
```

heartbeat_timeout与nodeclean_timeout 规则：

```
Bash
nodeclean_timeout >= 2 *  heartbeat_timeout
两个timeout都必须大于等于1s
如果都没配置，则使用默认值
如果配置设置不合理，njet会自动设置两个超时时间符合上面三个规则
```

#### 3.22.1.3 **通用配置：**

```
Bash
stream {
        server {
                listen 238.255.253.254:5555 udp;
                gossip zone=test:1m heartbeat_timeout=20s nodeclean_timeout=30s;
        }
}
```

#### 3.22.1.4 **节点配置：**

```
Bash

cluster_name helper;           #同一个集群配置同一个name
node_name node1;               #不通节点设置不同name
```

### 3.23.2 **集群**app sticky

#### 3.23.2.1 **功能描述**

特定app 客户端和server 端 sticky 路由一致性算法

##### 3.23.2.1.1 **指令描述**

```
Bash
app_sticky： 该功能配置指令
zone={name}:{size} : 共享内存配置名称以及大小
cookie:{name}  : 通过指定name的cookie值进行session路由 
header:{name}  : 通过指定name的header值进行session路由
其中： cookie和header二选一
ttl: {session_timeout} :session 过期时间，默认600s， 如果配置，必须>=1s，需配置在cookie 或者header之后
```

##### 3.23.2.1.2 **配置示例**

需要在NJet主配置文件 加载该模块：

load_module modules/njt_app_sticky_module.so;

```
Bash
#示例
...
load_module modules/njt_app_sticky_module.so;
...
stream {
        server {
                listen 238.255.253.254:5555 udp;
                gossip zone=test:1m heartbeat_timeout=20s nodeclean_timeout=30s;
        }
}
    
 http{   
     upstream back{
                app_sticky zone=app:4m cookie:route;
                server 127.0.0.1:8008;           #real server
                server 127.0.0.1:8009;           #real server
     }
      upstream back2{
                app_sticky zone=app:4m header:testheader ttl:600s;
                server 127.0.0.1:8006;           #real server
                server 127.0.0.1:8007;           #real server
     }
     server {
        listen       8186 ;
        server_name  localhost;
        location / {
          proxy_pass http://back;
        }
     }
     server {
        listen       9186 ;
        server_name  localhost;
        location / {
          proxy_pass http://back;
        }
     }
}
```

### 3.23.3 **集群**limit_conn

#### 3.23.3.1 **功能描述**

能够对NJet 集群实现 connection连接数限制

实现可认为是limit_conn_zone 以及limit_conn指令的集合体，两个指令合在一起的功能，也就是说zone的配置和连接数的设置在同一个指令完成

限制功能与limit conn功能一致

#### 3.23.3.2 **指令描述**

##### 3.23.3.2.1 **cluster_limit_conn_dry_run**

•     功能: 如果配置为on，实际不限制，但是会仍然记录请求数； 如果配置为off，则会受limit_conn限制

![img](https://gitee.com/gebona/picture/raw/master/202307101619914.jpg)

​                                                                                                     **点击图片可查看完整电子表格**

##### 3.23.3.2.2 **cluster_limit_conn_log_level**

•     功能: 达到限制时日志打印级别

![img](https://gitee.com/gebona/picture/raw/master/202307101620206.jpg)

​                                                                                                     **点击图片可查看完整电子表格**

##### 3.23.3.2.3 **cluster_limit_conn_status**

•     功能: 达到限制后返回的状态码，默认是503

![img](https://gitee.com/gebona/picture/raw/master/202307101620036.jpg)

​                                                                                                     **点击图片可查看完整电子表格**

##### 3.23.3.2.4 **cluster_limit_conn** 

•     功能：并发请求连接速限制

![img](https://gitee.com/gebona/picture/raw/master/202307101622522.jpg)

​                                                                                                   **点击图片可查看完整电子表格**

#### 3.23.3.3 **配置示例**

```
需要在NJet主配置文件 加载该模块：
load_module  modules/njt_http_cluster_limit_conn_module.so;
```

```
JSON

...
load_module  modules/njt_http_cluster_limit_conn_module.so;
...
stream {
        server {
                listen 238.255.253.254:5555 udp;
                gossip zone=test:1m heartbeat_timeout=20s nodeclean_timeout=30s;
        }
}
http {
   #http层
   cluster_limit_conn $binary_remote_addr zone=cluster_http:1m conn=10;
   upstream back{
                server 127.0.0.1:8008;         
                server 127.0.0.1:8009; 
   }
   
   server {
        #server层
        cluster_limit_conn $binary_remote_addr zone=cluster_server_20:1m conn=20;
        listen 8186; 
        location / {
                proxy_pass http://back;
        }
   }

   server {
        listen 9186;     
        location / { 
                #location层
                cluster_limit_conn $binary_remote_addr zone=cluster_location_40:1m conn=40;
                proxy_pass http://back;
        }
   }
}
```

### 3.23.4 **集群**limit_req

#### 3.23.4.1 **功能描述**

能够对OpenNJet 集群实现 http请求速率限制

实现可认为是limit_req_zone 以及limit_req指令的集合体，两个指令合在一起的功能，也就是说zone的配置和连接数的设置在同一个指令完成

限制功能与limit req功能一致

#### 3.23.4.2 **指令描述**

##### 3.23.4.2.1 **cluster_limit_req_dry_run**

•     功能: 如果配置为on，实际不限制，但是会仍然记录请求数； 如果配置为off，则会受limit_req限制

![img](https://gitee.com/gebona/picture/raw/master/202307101625409.jpg)

​                                                                                                  **点击图片可查看完整电子表格**

##### 3.23.4.2.2 **cluster_limit_req_log_level**

•     功能: 达到限制时日志打印级别

![img](https://gitee.com/gebona/picture/raw/master/202307101626862.jpg)

​                                                                                              **点击图片可查看完整电子表格**

##### 3.22.4.2.3 **cluster_limit_req_status**

•     功能: 达到限制后返回的状态码，默认是503

![img](https://gitee.com/gebona/picture/raw/master/202307101627886.jpg)

​                                                                                                 **点击图片可查看完整电子表格**

##### 3.23.4.2.4 **cluster_limit_req** 

•     功能：http请求速率限制

![img](https://gitee.com/gebona/picture/raw/master/202307101628759.jpg)

​                                                                                                   **点击图片可查看完整电子表格**

#### 3.23.4.3 **配置示例**

```
需要在njet主配置文件 加载该模块：
load_module  modules/njt_http_cluster_limit_req_module.so;
```

```
JSON

...
load_module  modules/njt_http_cluster_limit_req_module.so;
...


stream {
        server {
                listen 238.255.253.254:5555 udp;
                gossip zone=test:1m heartbeat_timeout=20s nodeclean_timeout=30s;
        }
}

 http {
   #http层
   cluster_limit_req $binary_remote_addr zone=cluster_http_10:10m rate=10;
   upstream back{
                server 127.0.0.1:8008;         
                server 127.0.0.1:8009; 
   }
   server {
        #server层
        cluster_limit_req $binary_remote_addr zone=cluster_server_25:10m rate=25;
        listen 8186; 
        location / {
                proxy_pass http://back;
        }
   }

   server {
        listen 9186;     
        location / { 
                #location层
                cluster_limit_req $binary_remote_addr zone=cluster_location_66:10m rate=66;
                proxy_pass http://back;
        }
   }
}
```

## 3.24故障注入

#### 3.24.1.1功能描述

HTTP1和HTTP2故障注入，支持中止(abort)来自下游服务的Http请求，和/或延迟(delay)代理请求,一个故障规则必须具有延迟或中止或两者兼有。在将HTTP请求转发到路由中指定的目的地时，可以注入一个或多个故障。故障注入策略应用于客户端HTTP流量：

延迟规范用于在请求转发路径中注入延迟。

中止规范用于提前中止一个请求，并返回一个预先指定的错误码。

注意：当客户端启用故障时，将不会启用超时或重试。

该模块支持ACL控制。

#### 3.24.1.2指令描述

故障注入主要有三种类型，同一个location下，只允许配置一种：

 Delay:  只注入延迟故障，延迟时间到后，再发起对upstream的连接请求

 Abort：只注入中止故障，收到客户端请求，直接将注入的状态码返回给客户端，结束请求

 Delay+Abort：同时注入延迟故障和abort故障，先进行延迟，延迟时间到后，直接返回abort设置的状态码给客户端，结束请求，不再发起对upstream的连接请求

##### 3.24.1.2.1**fault_inject** 

| Syntax:  | fault_inject type={type}  delay_duration={duration} status_code={code} delay_percentage={pct}  abort_percentage={pct}; |
| -------- | ------------------------------------------------------------ |
| Default: | -                                                            |
| Context: | location                                                     |

- 参数介绍：

	| 参数                      | 类型   | 必填                           | 说明                                                         |
	| ------------------------- | ------ | ------------------------------ | ------------------------------------------------------------ |
	| **type={type}**           | string | 是                             | delay、abort、delay_abort 三者之一delay：延迟注入类型abort：中止注入类型delay_abort: 延迟+中止注入类型 |
	| delay_duration={duration} | string | 是（delay和delay_abort 必填）  | delay时间， 1h/1m/1s/1ms， 必须>=1ms                         |
	| status_code={code}        | uint   | 是 （abort和delay_abort 必填） | code设置注入返回码， [200, 600]                              |
	| delay_percentage={pct}    | uint   | 可选，默认100 （100%）         | 设置注入delay请求的百分比，默认是100%，  范围： [1, 100], eg: 1表示1%， 10表示10% |
	| abort_percentage={pct}    | uint   | 可选，默认100 （100%）         | 设置注入abort请求的百分比，默认是100%，  范围： [1, 100], eg: 1表示1%， 10表示10% |

	###### 3.24.1.2.1.1Abort

	- 功能: 按照设置的百分比概率，对命中的请求直接返回指定的status_code

	| Syntax:  | fault_inject type=abort  status_code={code} abort_percentage={pct}; |
	| -------- | ------------------------------------------------------------ |
	| Default: | -                                                            |
	| Context: | location                                                     |

	参数介绍：

	| 参数                   | 类型 | 必填                   | 说明                                                         |
	| ---------------------- | ---- | ---------------------- | ------------------------------------------------------------ |
	| status_code={code}     | uint | 是                     | code设置注入返回码， [200, 600]                              |
	| abort_percentage={pct} | uint | 可选，默认100 （100%） | 设置注入abort请求的百分比，默认是100%，  范围： [1, 100], eg: 1表示1%， 10表示10% |

	###### 3.24.1.2.1.2Delay

	- 功能: 按照设置的百分比概率，对命中的请求按照设定的duration时间延迟发起对上游upstream的请求

	| Syntax:  | fault_inject type=delay  delay_duration={duration} delay_percentage={pct}; |
	| -------- | ------------------------------------------------------------ |
	| Default: | -                                                            |
	| Context: | location                                                     |

	参数介绍：

	| 参数                      | 类型   | 必填                   | 说明                                                         |
	| ------------------------- | ------ | ---------------------- | ------------------------------------------------------------ |
	| delay_duration={duration} | string | 是                     | delay时间， 1h/1m/1s/1ms， 必须>=1ms                         |
	| delay_percentage={pct}    | uint   | 可选，默认100 （100%） | 设置注入delay请求的百分比，默认是100%，  范围： [1, 100], eg: 1表示1%， 10表示10% |

######               3.24.1.2.1.3Delay + Abort

- 功能: 按照设置的百分比概率，对命中的请求按照设定的duration时间延迟发起对上游upstream的请求

| Syntax:  | fault_inject type=delay_abort  delay_duration={duration} status_code={code} delay_percentage={pct} abort_percentage={pct} ; |
| -------- | ------------------------------------------------------------ |
| Default: | -                                                            |
| Context: | location                                                     |

参数介绍：

| 参数                      | 类型   | 必填                   | 说明                                                         |
| ------------------------- | ------ | ---------------------- | ------------------------------------------------------------ |
| delay_duration={duration} | string | 是                     | delay时间， 1h/1m/1s/1ms， 必须>=1ms                         |
| status_code={code}        | uint   | 是                     | code设置注入返回码， [200, 600]                              |
| delay_percentage={pct}    | uint   | 可选，默认100 （100%） | 设置注入delay请求的百分比，默认是100，  范围： [1, 100], eg: 1表示1%， 100表示100% |
| abort_percentage={pct}    | uint   | 可选，默认100 （100%） | 设置注入abort请求的百分比，默认是100，  范围： [1, 100], eg: 1表示1%， 100表示100% |

#### 3.24.1.3配置示例

##### 3.24.1.3.2Abort

```
     server {
        listen       80 ;
        server_name  localhost;

        location / {
          fault_inject type=abort  status_code=405 abort_percentage=100;
          proxy_next_upstream_tries 0;      #关闭重试
          proxy_next_upstream_timeout 0;    #关闭超时
          proxy_pass http://back/;
        }
     }
```

##### 3.24.1.3.3Delay

```
     server {
        listen       80 ssl;
        server_name  localhost;

        location / {
          fault_inject type=delay  delay_duration=10s delay_percentage=100;
          proxy_next_upstream_tries 0;      #关闭重试
          proxy_next_upstream_timeout 0;    #关闭超时
          proxy_pass http://back;
        }
     }
```

##### 3.24.1.3.4Delay + Abort

```
     server {
        listen       81 ssl http2;
        server_name  192.168.40.136;
        ssl_certificate                 /mycert/www.tmlake.xn--com-7m0aa+3.pem;
        ssl_certificate_key             mycert/www.tmlake.xn--com-7m0aa+3-key.pem;

        location / {
          fault_inject type=delay_abort  delay_duration=10s status_code=405 delay_percentage=20 abort_percentage=80;
          proxy_next_upstream_tries 0;      #关闭重试
          proxy_next_upstream_timeout 0;    #关闭超时
          proxy_pass http://back;
        }
     }
```

## 3.25动态故障注入

#### 3.25.1.1功能描述

可以通过声明式api对故障注入进行动态配置。该模块支持ACL控制，详见章节3.26

#### 3.25.1.2参考配置

```
helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;

load_module modules/njt_http_dyn_fault_inject_module.so;

worker_processes  auto;

cluster_name helper;
node_name node1;

error_log  logs/error.log info;
pid        logs/njet.pid;

events {
    worker_connections  1024;
}

http {
    
    dyn_kv_conf conf/iot-work.conf;
    include       mime.types;
    default_type  application/octet-stream;
    
    access_log  logs/access.log;
    
    upstream backend1 { 
         
         zone backend1 128k;
         
         server 192.168.40.150:5678;

    }

    server {
    
        listen 0.0.0.0:5555;
        server_name localhost;

         location / {
            
             proxy_pass http://backend1;


        location /test_faultinjection {

            # fault_inject type=delay delay_duration=5s status_code=404 delay_percentage=100 abort_percentage=100;
            proxy_next_upstream_tries 0;      #关闭重试
            proxy_next_upstream_timeout 0;    #关闭超时
            proxy_pass http://backend1;
            
            location /test_faultinjection/test_test_faultinjection {
                
                proxy_next_upstream_tries 0;      #关闭重试
                proxy_next_upstream_timeout 0;    #关闭超时
                proxy_pass http://backend1;
            }
        }

    }

}
```

```
 server {
        listen       8081;
        location /config {
            config_api;
            limit_except GET {
               auth_basic "NGINX plus API";
               auth_basic_user_file /etc/njet/htpasswd;
            }
        }
  }
```

#### 3.25.1.3调用样例

##### 3.25.1.3.1动态配置delay类型故障注入

```
curl -X PUT http://127.0.0.1:8081/config/2/config/http_dyn_fault_inject -d'{
    "servers": [
        {
            "listens": [
                "0.0.0.0:5555"
            ],
            "serverNames": [
                "localhost"
            ],
            "locations": [
                {
                    "location": "/test_faultinjection",
                    "fault_inject_type": "delay",
                    "delay_percentage": 100,
                    "abort_percentage": 100,
                    "status_code": 200,
                    "delay_duration": "5s"
                 }
            ]
        }
    ]
}
```

```
{
    "code": 0,
    "msg": "success."
}
```

##### 3.25.1.3.2动态配置abort类型故障注入

```
curl -X PUT http://127.0.0.1:8081/config/2/config/http_dyn_fault_inject -d'{
    "servers": [
        {
            "listens": [
                "0.0.0.0:5555"
            ],
            "serverNames": [
                "localhost"
            ],
            "locations": [
                {
                    "location": "/test_faultinjection",
                    "fault_inject_type": "abort",
                    "delay_percentage": 100,
                    "abort_percentage": 100,
                    "status_code": 405,
                    "delay_duration": "5s"
                 }
            ]
        }
    ]
}
```

```
{
    "code": 0,
    "msg": "success."
}
```

##### 3.25.1.3.3动态配置delay+abort类型故障注入

```
curl -X PUT http://127.0.0.1:8081/config/2/config/http_dyn_fault_inject -d'{
    "servers": [
        {
            "listens": [
                "0.0.0.0:5555"
            ],
            "serverNames": [
                "localhost"
            ],
            "locations": [
                {
                    "location": "/test_faultinjection",
                    "fault_inject_type": "delay_abort",
                    "delay_percentage": 100,
                    "abort_percentage": 100,
                    "status_code": 405,
                    "delay_duration": "5s"
                 }
            ]
        }
    ]
}
```

```
{
    "code": 0,
    "msg": "success."
}
```

##### 3.25.1.3.4动态获取当前的故障注入配置

```
curl -X GET http://127.0.0.1:8081/config/2/config/http_dyn_fault_inject | jq
```

```
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:5555"
            ],
            "serverNames": [
                "localhost"
            ],
            "locations": [
            {
                    "location": "/",
                    "fault_inject_type": "none",
                    "delay_percentage": 100,
                    "abort_percentage": 100,
                    "status_code": 200,
                    "delay_duration": ""
            },
                {
                    "location": "/test_faultinjection",
                    "fault_inject_type": "delay_abort",
                    "delay_percentage": 100,
                    "abort_percentage": 100,
                    "status_code": 405,
                    "delay_duration": "5s",
                    "locations": [
                        {
                            "location": "/test_faultinjection/test_test_faultinjection",
                            "fault_inject_type": "none",
                            "delay_percentage": 100,
                            "abort_percentage": 100,
                            "status_code": 200,
                            "delay_duration": ""
                        }
                    ]
                }
            ]
        }
    ]
}
```

##### 3.25.1.3.4动态获取当前的故障注入配置

```
curl -X GET http://127.0.0.1:8081/config/2/config/http_dyn_fault_inject | jq

```

```
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:5555"
            ],
            "serverNames": [
                "localhost"
            ],
            "locations": [
            {
                    "location": "/",
                    "fault_inject_type": "none",
                    "delay_percentage": 100,
                    "abort_percentage": 100,
                    "status_code": 200,
                    "delay_duration": ""
            },
                {
                    "location": "/test_faultinjection",
                    "fault_inject_type": "delay_abort",
                    "delay_percentage": 100,
                    "abort_percentage": 100,
                    "status_code": 405,
                    "delay_duration": "5s",
                    "locations": [
                        {
                            "location": "/test_faultinjection/test_test_faultinjection",
                            "fault_inject_type": "none",
                            "delay_percentage": 100,
                            "abort_percentage": 100,
                            "status_code": 200,
                            "delay_duration": ""
                        }
                    ]
                }
            ]
        }
    ]
}
```

## 3.26api接口ACL控制

### 3.26.1功能描述

ACL是Access Control List（访问控制列表）的缩写，它是一种用于访问控制的技术.控制面的每个模块提供了可选的配置指令limit_except,通过limit_except 指令，针对http 方法，进行鉴权.以下以kv 模块为例，其他模块类似。

### 3.26.2参考配置（ctrl.conf)

```
location /kv {
                        dyn_sendmsg_kv;
                        limit_except GET {
               auth_basic "NGINX plus API";
               auth_basic_user_file /home/njet/api_acl/htpasswd;
                        }

                }


```

当调用/kv路径时，如果非GET 请求，要进行用户认证。

**生成用户名，密码：**

- 安装：yum install httpd-tools -y
- 生成密码：htpasswd -c  -d  /home/njet/api_acl/htpasswd  test(用户名)

### 3.26.3调用样例

#### 3.26.3.1设置键值 

ctrl.conf 配置文件中配置了acl 后，访问kv 模块时需要用如下命令：

```
curl --user 用户名:密码 -X 'POST'   'http://192.168.40.158:8088/kv'   -H 'accept: */*'   -H 'Content-Type: application/json'   -d '{
  "key": "test_key",
  "value": "test msg"
}'

```

#### 3.26.3.2获取键值对

```
curl -X 'GET'   'http://192.168.40.158:8088/kv?key=test_key00'
```

![WeChat975f62e807b42023ff3a85113793172a](https://gitee.com/gebona/picture/raw/master/202308281058495.png)

## 3.27动态http ssl证书

### 3.27.1动态服务端ssl数据面配置说明

依赖模块

```
njt_helper_broker_module
njt_http_kv_module.so
njt_dyn_ssl_module.so
```

主要配置示例

```
注意配置文件中需要修改so路径，log路径，替换ssl证书
```

```
load_module modules/njt_http_kv_module.so;
load_module modules/njt_dyn_ssl_module.so;

helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;

worker_processes  1;
events {
    worker_connections  1024;
}
cluster_name helper;
node_name node1;


http {
    access_log logs/access.log combined;
    dyn_kv_conf conf/iot-work.cf;
    upstream demo {
       zone demo 128k;
        server 192.168.40.141:8080;
        keepalive 10240;
    }
    server {
        listen 443 ssl;

        #开启国密功能
        ssl_ntls     on;

        #国际 ECC 证书（可选）
        ssl_certificate                 certs/server.crt;
        ssl_certificate_key             certs/server.key;

        #ssl_certificate            certs/SS.crt certs/SE.crt;
        #ssl_certificate_key        certs/SS.key  certs/SE.key;
        #国密套件
        #ssl_ciphers "ECC-SM2-SM4-CBC-SM3:ECDHE-SM2-WITH-SM4-SM3:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:ECDHE-RSA-AES128-SHA256:!aNULL:!eNULL:!RC4:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS";

        #ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

        default_type            text/plain;

        add_header  "Content-Type" "text/html;charset=utf-8";

        location / {
            return 200 "njet ntls test OK, ssl_protocol is $ssl_protocol (NTLSv1.1 表示国密，其他表示国际)";
        }
    }
}
```

### 3.27.2动态服务端ssl控制面配置说明

```
注意配置文件中需要修改so路径，log路径，替换ssl证书
```

```
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_http_ssl_api_module.so;         #加载ssl api

events {
    worker_connections  1024;
}
error_log         logs/error_ctrl.log info;

http {
    dyn_sendmsg_conf  conf/iot-ctrl.conf;
    access_log        logs/access_ctrl.log combined;

    include           mime.types;

    server {
        listen       8081;
        

        location /ssl {
            dyn_ssl_api;         #开启ssl动态配置
        }
  }

}


cluster_name helper;
node_name node1;
```

该模块支持ACL控制，配置参考

```
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_http_ssl_api_module.so;         #加载ssl api

events {
    worker_connections  1024;
}
error_log         logs/error_ctrl.log info;

http {
    dyn_sendmsg_conf  conf/iot-ctrl.conf;
    access_log        logs/access_ctrl.log combined;

    include           mime.types;

    server {
        listen       8081;
      

        location /ssl {
            dyn_ssl_api;
            limit_except GET {
                auth_basic "NGINX plus API";
                auth_basic_user_file /etc/njet/htpasswd;
          }
        }

  }

}


cluster_name helper;
node_name node1;
```

### 3.27.3调用样例

#### 3.27.3.1查询http server ssl 当前配置

请求

```
curl -X GET http://127.0.0.1:8081/ssl
```

返回值

```
< HTTP/1.1 200 OK
< Server: njet/1.23.1
< Date: Sat, 06 May 2023 06:52:52 GMT
< Content-Type: application/json
< Content-Length: 674
< Connection: keep-alive

{
    "servers": [
        {
            "listens": [
                "0.0.0.0:443"
            ],
            "serverNames": [
                "localhost"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "certs/server.crt",
                    "certificateKey": "certs/server.key"
                }
            ]
        }
    ]
}
```

#### 3.27.3.2新增http server ssl国密证书

前提需要静态配置文件配置指令，ssl_ntls on;

```
curl -X PUT  http://127.0.0.1:8081/ssl 
Content-Type: application/json

{
    "listens": [
        "0.0.0.0:443"
    ],
    "serverNames": [
        "localhost"
    ],
    "type": "add",
    "cert_info": {
        "cert_type": "ntls",
        "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+GMAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABOgF2jiXSu+sWssR6wDiiozJXTep2gZgDHdkYMn8JRYu3r/JnD79hdaj\r\nys4KMyYecZ7vxx7Za8XDCHqLtMCYUaejgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQrE10lVWLeOLx0wUJ0qvw9DAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgDAMAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANMXEYW4FZrKpQxedELM7xn461/lHePexG7U7qRJeTdjAiEAuISM\r\nWZunhb0P1gUnMoSjBwKLIr2f7YpmO7xxUO63qdU=\r\n-----END CERTIFICATE-----\r\n",
        "certificateKey": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQgrGLqbyNS5aTuc7Y6\r\nlH3lspDv+ipwL+JEVw6eJnp3CPSgCgYIKoEcz1UBgi2hRANCAAToBdo4l0rvrFrL\r\nEesA4oqMyV03qdoGYAx3ZGDJ/CUWLt6/yZw+/YXWo8rOCjMmHnGe78ce2WvFwwh6\r\ni7TAmFGn\r\n-----END PRIVATE KEY-----\r\n",
        "certificateEnc": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+5MAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABJ434ZA5OJuDfYmPhGuYqcEnth0qwTrr0Nr8/e81X10F+AwBCwt/7iI/\r\nVr8Wus3KpmidSryPfQUrtU2QaY600tOjgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQpWdTiQuwA1LLaCKQHutzKjAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgA4MAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANqpcb/4ZIUVTePBNvJDHb6OPyl3Elr6Z9Cig5DQ97EaAiEA58Lx\r\nLM7OuX/4nxSS+n8LyxToV6uRGOdQKPyAsaOhTHU=\r\n-----END CERTIFICATE-----\r\n",
        "certificateKeyEnc": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQg2zdpRZvEtA5XRXbc\r\nHc/AI3DFLwFAF6s0mUysPE2ZSIOgCgYIKoEcz1UBgi2hRANCAASeN+GQOTibg32J\r\nj4RrmKnBJ7YdKsE669Da/P3vNV9dBfgMAQsLf+4iP1a/FrrNyqZonUq8j30FK7VN\r\nkGmOtNLT\r\n-----END PRIVATE KEY-----\r\n"
    }
}
```

使用GET请求查询http server ssl 当前配置

```
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:443"
            ],
            "serverNames": [
                "localhost"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "certs/server.crt",
                    "certificateKey": "certs/server.key"
                },
                {
                    "cert_type": "ntls",
                    "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIICJjCCAcygAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAcwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYYxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRowGAYDVQQDDBFzZXJ2\r\nZXIgc2lnbiAoU00yKTBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABKWvwarEtpHK\r\nckfCaWTcqhzO8e3z02Q2+NOyUr94WgvuQFuJjdLMoHCDxDuMrhXaxUXhXE/rZsG7\r\nZ4XWyd7K/n+jGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgbAMAoGCCqBHM9VAYN1\r\nA0gAMEUCIQDXyNeZN0iqy2cxDYytWtWLayYdSmdnuPg8VOhXUWWb+QIgEwj6Y2xg\r\n38xlatqML8aO89O4movtgOxPZG2YYTaDon8=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKey": "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIPunrzGM/F+w/64DaLg5RLn2ZUoqYEzwsgxUkeKWF6jmoAoGCCqBHM9V\r\nAYItoUQDQgAEpa/BqsS2kcpyR8JpZNyqHM7x7fPTZDb407JSv3haC+5AW4mN0syg\r\ncIPEO4yuFdrFReFcT+tmwbtnhdbJ3sr+fw==\r\n-----END EC PRIVATE KEY-----\r\n",
                    "certificateEnc": "data:-----BEGIN CERTIFICATE-----\r\nMIICJTCCAcugAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAgwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYUxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRkwFwYDVQQDDBBzZXJ2\r\nZXIgZW5jIChTTTIpMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEdGUnMh317JO6\r\nePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaAwMC16wze1n3JjSIqIMf7ielE\r\nePXi4G1dsKMaMBgwCQYDVR0TBAIwADALBgNVHQ8EBAMCAzgwCgYIKoEcz1UBg3UD\r\nSAAwRQIhANC8B9M4Mo0GDWDgSeJ5M4wlF0QgWMlY29GYB1/bGxq6AiAb4IlVTwnZ\r\nV0/PP3zuNjJjp7blkvsaCGsXtb9pk2rQYw==\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKeyEnc": "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIJH/qG2bKhybB1uUSnQbKiNueeLneQVioS+0kLMygYCVoAoGCCqBHM9V\r\nAYItoUQDQgAEdGUnMh317JO6ePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaA\r\nwMC16wze1n3JjSIqIMf7ielEePXi4G1dsA==\r\n-----END EC PRIVATE KEY-----\r\n"
                }
            ]
        }
    ]
}
```

#### 3.27.3.3新增http server ssl当前证书

```
curl -X PUT http://127.0.0.1:8443/ssl
Content-Type: application/json

{
    "listens": [
        "0.0.0.0:443"
    ],
    "serverNames": [
        "aaaa"
    ],
    "type": "add",
    "cert_info": {
         "cert_type": "regular",
         "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIIEITCCAwmgAwIBAgIUK6l+HK4KR/hLosx2XqNTaHLUkO0wDQYJKoZIhvcNAQEL\r\nBQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl\r\naUppbmcxDjAMBgNVBAoTBW5naW54MQ8wDQYDVQQLEwZ0bWxha2UxETAPBgNVBAMT\r\nCG5naW54LWNhMCAXDTIyMDgwOTA5NDEwMFoYDzIxMjIwNzE2MDk0MTAwWjBiMQsw\r\nCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpSmluZzEQMA4GA1UEBxMHQmVpSmluZzEO\r\nMAwGA1UEChMFbmdpbngxDzANBgNVBAsTBnRtbGFrZTEOMAwGA1UEAxMFbmdpbngw\r\nggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC0hs5E8ZZT+ZU/DvvIRrSL\r\nokk6OE5EA4URo6zawqguKRlTc1USgfMdZFwR2znCxjFcsk7W/FSWqA5k5zyePYL8\r\n/0lhn3i/GQGczfa9dMZZEtN9EMD6TPOiDI/CaWAFc3U7qE+SYckUHLu/lMX8LshR\r\ndNkGecq41Ck5Sftm3xwy4cmpcCIyMpNumHrjJyxkp3S/QGewgZRQypntG9R34EKj\r\nQNFnovsn39TeeG+DBjS5m1+roth0JJ6DbemB47hy/uaEwAQpobN1etx20wCwxRO7\r\n7Huuh4kmWXjUol0nvzXM3ZZX2QnHn4jIg/ALS7O6p86z+m4vEHK4hRfWf52xGjMv\r\nAgMBAAGjgckwgcYwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMB\r\nBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBQz6z5AIFRrP+KD1gBt\r\nW6j5NEMzwDAfBgNVHSMEGDAWgBQ64+C++PsR6pDZoQ7EORozSJ8d9DBHBgNVHREE\r\nQDA+ggkxOTIuMTY4LiqCDiouY2x1c3RlcjEuY29tgg0qLmNsdXN0ZXIuY29tggwq\r\nLnRtbGFrZS5jb22HBH8AAAEwDQYJKoZIhvcNAQELBQADggEBADouKA9fFflNAs2u\r\nu7yj4cGNG+sC8r1wDWyGiXGDzvaB7C/sPTZiCgh8hI5gxzOSC+Se51kNFYymN9g4\r\nBPfB//SZ+CxO60BOt5eFF4PxIW7+d8gGFJr7Yx55QGA9DrvGi/LEiwjV3bQaezjN\r\noXUkfFgyKv79AfNFShkUPiHCklnfyqcCF+GEKxNC1q4wyClUt2EGtwfiO4YR85s0\r\n7jQ2BKJOsw3LLd8pZSvOXYu/I9DoO5w3gM5TgjnJlQPq+bmgqJ7bfWbSpFo70cdF\r\nFLfE9MLpwhDjVQEJ6DOmZqU1dVdNtwLrUrm0/8xkkmac0/8BaEYyPy1SNMGJTMGE\r\nW8psxto=\r\n-----END CERTIFICATE-----\r\n",
         "certificateKey": "data:-----BEGIN RSA PRIVATE KEY-----\r\nMIIEogIBAAKCAQEAtIbORPGWU/mVPw77yEa0i6JJOjhORAOFEaOs2sKoLikZU3NV\r\nEoHzHWRcEds5wsYxXLJO1vxUlqgOZOc8nj2C/P9JYZ94vxkBnM32vXTGWRLTfRDA\r\n+kzzogyPwmlgBXN1O6hPkmHJFBy7v5TF/C7IUXTZBnnKuNQpOUn7Zt8cMuHJqXAi\r\nMjKTbph64ycsZKd0v0BnsIGUUMqZ7RvUd+BCo0DRZ6L7J9/U3nhvgwY0uZtfq6LY\r\ndCSeg23pgeO4cv7mhMAEKaGzdXrcdtMAsMUTu+x7roeJJll41KJdJ781zN2WV9kJ\r\nx5+IyIPwC0uzuqfOs/puLxByuIUX1n+dsRozLwIDAQABAoIBAFh+VJLbUnOrvwuA\r\nTtBoSIzCat8NRuB0UUDKWSuLjGHEZ9POj39ZEFHyJmfibTgba4sjJR6h5t1LWHMC\r\nH2b6hEF86v3d7JTQr0esdy18FtcHMYD3O4H3Qt7HBZmpihZh+K/b29XH9YfUZfyN\r\n81ehnzS+8LwJ6+QarHKW35QX/ny5+pCw+G97cjdE7sClZnryd1KseWubAXDLSgmQ\r\nJcjXMMkwjtd2x/w23ZT97WMEcToL1wMw4hF/tHtu9u3UHMhzLHgrfrkW4QmACbLF\r\n8MJR94wzscJrUEviz5fV+3ZTmK+lujGmO/KkOpNAPGf8QAeBfa/qyBTQCQgCdxRo\r\n8hrDT4ECgYEA5vLsDDWiAROvvPaRkCv0y0UWkdgyBxv2T6pjDJZzMxb0yKZ48E6l\r\nh0PI9bfn0KbI1ZZviiZg20oZ/Keo2wLYa+W00rZhWljWUh0Ivd/5vTPHBmpsXl6+\r\nRK49aNVkfqlleherhfrYUkHKKDeg4lKVY3fwAlAQeg/y4Ds/8k3HUuECgYEAyBu8\r\ns7KuO7sAaJVj+QFfRrR6u2eJD48HPB4UoeDYMaKAuPTiM5/AUjH/YdD4E2lGAyJJ\r\nJVN413yu3Dk9UIAivgqCROepAcnnZ1BwE2qI/gfr2GD95o8JQSphze0pC/6WS/jG\r\niBR+j7MGWIkHJHEIUjOrf8eqtVLAywjxz9rPWA8CgYBTkuzgrjfl893Qn9mlNoLr\r\nXCECvh28fN3xjlMxpvAhONl0EuoI7CzyehEq+lYlJ3Xd9QaAE8tRD8u/plxwhOMU\r\niJea+OzZ6PQF2wPi0j5pvWb0Z2a378kiyXrniPFI9LwIJrCnV1MY0T36t8a8n+33\r\nhNuRuq97vHHDuy003fiXgQKBgGKzM6cauc+iU/hBvzbBk4HnYSXwUm1HKdVgLOMP\r\naPNKaN1RhATchdrE6GcR0FqasTq4fYWYn2ECEalz3idHnFtKCaj87qKAOM//n9gj\r\n0wAhXhWy+WjwIitvQSB2Gqnc37sHML1MBoTQU4/1vn0d93G8JJn5HN0kvQ0oE0Vn\r\ncp/HAoGAGCxpM24rHBK0HmlABZd8ZfLW/sW4Y7bMsQmrOVuPZcTWcO5HzkkiGzTc\r\n8DCDDeGdfJUC9NaM0DnVuMwp8/f5uZv4dTS7Hzf8Ie5vR9oQx/z73iakiKUIpDQc\r\n0/sUJSKijOQRT1xZQmkw5+QClmbjLLS3ZXn9kMt0bbxZRPkcV2A=\r\n-----END RSA PRIVATE KEY-----\r\n"
    }
}
```

查询更新结果

```
{
    "servers": [
        {
            "listens": [
                "0.0.0.0:443"
            ],
            "serverNames": [
                "aaaa"
            ],
            "certificates": [
                {
                    "cert_type": "regular",
                    "certificate": "certs/server.crt",
                    "certificateKey": "certs/server.key"
                },
                {
                    "cert_type": "ntls",
                    "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+GMAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABOgF2jiXSu+sWssR6wDiiozJXTep2gZgDHdkYMn8JRYu3r/JnD79hdaj\r\nys4KMyYecZ7vxx7Za8XDCHqLtMCYUaejgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQrE10lVWLeOLx0wUJ0qvw9DAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgDAMAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANMXEYW4FZrKpQxedELM7xn461/lHePexG7U7qRJeTdjAiEAuISM\r\nWZunhb0P1gUnMoSjBwKLIr2f7YpmO7xxUO63qdU=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKey": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQgrGLqbyNS5aTuc7Y6\r\nlH3lspDv+ipwL+JEVw6eJnp3CPSgCgYIKoEcz1UBgi2hRANCAAToBdo4l0rvrFrL\r\nEesA4oqMyV03qdoGYAx3ZGDJ/CUWLt6/yZw+/YXWo8rOCjMmHnGe78ce2WvFwwh6\r\ni7TAmFGn\r\n-----END PRIVATE KEY-----\r\n",
                    "certificateEnc": "data:-----BEGIN CERTIFICATE-----\r\nMIICWTCCAfygAwIBAgIGAYYFmA+5MAwGCCqBHM9VAYN1BQAwSzELMAkGA1UEBhMC\r\nQ04xDjAMBgNVBAoTBUdNU1NMMRAwDgYDVQQLEwdQS0kvU00yMRowGAYDVQQDExFN\r\naWRkbGVDQSBmb3IgVGVzdDAiGA8yMDIzMDEzMDE2MDAwMFoYDzIwMjQwMTMwMTYw\r\nMDAwWjB2MQswCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpamluZzERMA8GA1UEBxMI\r\nQmVpamluZzExCzAJBgNVBAoTAmFhMQwwCgYDVQQLEwNhYWExDTALBgNVBAMTBGFh\r\nYWExGDAWBgkqhkiG9w0BCQEWCWFhQGFhLmNvbTBZMBMGByqGSM49AgEGCCqBHM9V\r\nAYItA0IABJ434ZA5OJuDfYmPhGuYqcEnth0qwTrr0Nr8/e81X10F+AwBCwt/7iI/\r\nVr8Wus3KpmidSryPfQUrtU2QaY600tOjgZowgZcwGwYDVR0jBBQwEoAQ+X9VtCeU\r\nM2KmVspvzF0a/zAZBgNVHQ4EEgQQpWdTiQuwA1LLaCKQHutzKjAPBgNVHREECDAG\r\nggRhYWFhMDEGCCsGAQUFBwEBBCUwIzAhBggrBgEFBQcwAYYVaHR0cHM6Ly9vY3Nw\r\nLmdtc3NsLmNuMAkGA1UdEwQCMAAwDgYDVR0PAQH/BAQDAgA4MAwGCCqBHM9VAYN1\r\nBQADSQAwRgIhANqpcb/4ZIUVTePBNvJDHb6OPyl3Elr6Z9Cig5DQ97EaAiEA58Lx\r\nLM7OuX/4nxSS+n8LyxToV6uRGOdQKPyAsaOhTHU=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKeyEnc": "data:-----BEGIN PRIVATE KEY-----\r\nMIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQg2zdpRZvEtA5XRXbc\r\nHc/AI3DFLwFAF6s0mUysPE2ZSIOgCgYIKoEcz1UBgi2hRANCAASeN+GQOTibg32J\r\nj4RrmKnBJ7YdKsE669Da/P3vNV9dBfgMAQsLf+4iP1a/FrrNyqZonUq8j30FK7VN\r\nkGmOtNLT\r\n-----END PRIVATE KEY-----\r\n"
                },
                {
                    "cert_type": "regular",
                    "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIIEITCCAwmgAwIBAgIUK6l+HK4KR/hLosx2XqNTaHLUkO0wDQYJKoZIhvcNAQEL\r\nBQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl\r\naUppbmcxDjAMBgNVBAoTBW5naW54MQ8wDQYDVQQLEwZ0bWxha2UxETAPBgNVBAMT\r\nCG5naW54LWNhMCAXDTIyMDgwOTA5NDEwMFoYDzIxMjIwNzE2MDk0MTAwWjBiMQsw\r\nCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpSmluZzEQMA4GA1UEBxMHQmVpSmluZzEO\r\nMAwGA1UEChMFbmdpbngxDzANBgNVBAsTBnRtbGFrZTEOMAwGA1UEAxMFbmdpbngw\r\nggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC0hs5E8ZZT+ZU/DvvIRrSL\r\nokk6OE5EA4URo6zawqguKRlTc1USgfMdZFwR2znCxjFcsk7W/FSWqA5k5zyePYL8\r\n/0lhn3i/GQGczfa9dMZZEtN9EMD6TPOiDI/CaWAFc3U7qE+SYckUHLu/lMX8LshR\r\ndNkGecq41Ck5Sftm3xwy4cmpcCIyMpNumHrjJyxkp3S/QGewgZRQypntG9R34EKj\r\nQNFnovsn39TeeG+DBjS5m1+roth0JJ6DbemB47hy/uaEwAQpobN1etx20wCwxRO7\r\n7Huuh4kmWXjUol0nvzXM3ZZX2QnHn4jIg/ALS7O6p86z+m4vEHK4hRfWf52xGjMv\r\nAgMBAAGjgckwgcYwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMB\r\nBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBQz6z5AIFRrP+KD1gBt\r\nW6j5NEMzwDAfBgNVHSMEGDAWgBQ64+C++PsR6pDZoQ7EORozSJ8d9DBHBgNVHREE\r\nQDA+ggkxOTIuMTY4LiqCDiouY2x1c3RlcjEuY29tgg0qLmNsdXN0ZXIuY29tggwq\r\nLnRtbGFrZS5jb22HBH8AAAEwDQYJKoZIhvcNAQELBQADggEBADouKA9fFflNAs2u\r\nu7yj4cGNG+sC8r1wDWyGiXGDzvaB7C/sPTZiCgh8hI5gxzOSC+Se51kNFYymN9g4\r\nBPfB//SZ+CxO60BOt5eFF4PxIW7+d8gGFJr7Yx55QGA9DrvGi/LEiwjV3bQaezjN\r\noXUkfFgyKv79AfNFShkUPiHCklnfyqcCF+GEKxNC1q4wyClUt2EGtwfiO4YR85s0\r\n7jQ2BKJOsw3LLd8pZSvOXYu/I9DoO5w3gM5TgjnJlQPq+bmgqJ7bfWbSpFo70cdF\r\nFLfE9MLpwhDjVQEJ6DOmZqU1dVdNtwLrUrm0/8xkkmac0/8BaEYyPy1SNMGJTMGE\r\nW8psxto=\r\n-----END CERTIFICATE-----\r\n",
                    "certificateKey": "data:-----BEGIN RSA PRIVATE KEY-----\r\nMIIEogIBAAKCAQEAtIbORPGWU/mVPw77yEa0i6JJOjhORAOFEaOs2sKoLikZU3NV\r\nEoHzHWRcEds5wsYxXLJO1vxUlqgOZOc8nj2C/P9JYZ94vxkBnM32vXTGWRLTfRDA\r\n+kzzogyPwmlgBXN1O6hPkmHJFBy7v5TF/C7IUXTZBnnKuNQpOUn7Zt8cMuHJqXAi\r\nMjKTbph64ycsZKd0v0BnsIGUUMqZ7RvUd+BCo0DRZ6L7J9/U3nhvgwY0uZtfq6LY\r\ndCSeg23pgeO4cv7mhMAEKaGzdXrcdtMAsMUTu+x7roeJJll41KJdJ781zN2WV9kJ\r\nx5+IyIPwC0uzuqfOs/puLxByuIUX1n+dsRozLwIDAQABAoIBAFh+VJLbUnOrvwuA\r\nTtBoSIzCat8NRuB0UUDKWSuLjGHEZ9POj39ZEFHyJmfibTgba4sjJR6h5t1LWHMC\r\nH2b6hEF86v3d7JTQr0esdy18FtcHMYD3O4H3Qt7HBZmpihZh+K/b29XH9YfUZfyN\r\n81ehnzS+8LwJ6+QarHKW35QX/ny5+pCw+G97cjdE7sClZnryd1KseWubAXDLSgmQ\r\nJcjXMMkwjtd2x/w23ZT97WMEcToL1wMw4hF/tHtu9u3UHMhzLHgrfrkW4QmACbLF\r\n8MJR94wzscJrUEviz5fV+3ZTmK+lujGmO/KkOpNAPGf8QAeBfa/qyBTQCQgCdxRo\r\n8hrDT4ECgYEA5vLsDDWiAROvvPaRkCv0y0UWkdgyBxv2T6pjDJZzMxb0yKZ48E6l\r\nh0PI9bfn0KbI1ZZviiZg20oZ/Keo2wLYa+W00rZhWljWUh0Ivd/5vTPHBmpsXl6+\r\nRK49aNVkfqlleherhfrYUkHKKDeg4lKVY3fwAlAQeg/y4Ds/8k3HUuECgYEAyBu8\r\ns7KuO7sAaJVj+QFfRrR6u2eJD48HPB4UoeDYMaKAuPTiM5/AUjH/YdD4E2lGAyJJ\r\nJVN413yu3Dk9UIAivgqCROepAcnnZ1BwE2qI/gfr2GD95o8JQSphze0pC/6WS/jG\r\niBR+j7MGWIkHJHEIUjOrf8eqtVLAywjxz9rPWA8CgYBTkuzgrjfl893Qn9mlNoLr\r\nXCECvh28fN3xjlMxpvAhONl0EuoI7CzyehEq+lYlJ3Xd9QaAE8tRD8u/plxwhOMU\r\niJea+OzZ6PQF2wPi0j5pvWb0Z2a378kiyXrniPFI9LwIJrCnV1MY0T36t8a8n+33\r\nhNuRuq97vHHDuy003fiXgQKBgGKzM6cauc+iU/hBvzbBk4HnYSXwUm1HKdVgLOMP\r\naPNKaN1RhATchdrE6GcR0FqasTq4fYWYn2ECEalz3idHnFtKCaj87qKAOM//n9gj\r\n0wAhXhWy+WjwIitvQSB2Gqnc37sHML1MBoTQU4/1vn0d93G8JJn5HN0kvQ0oE0Vn\r\ncp/HAoGAGCxpM24rHBK0HmlABZd8ZfLW/sW4Y7bMsQmrOVuPZcTWcO5HzkkiGzTc\r\n8DCDDeGdfJUC9NaM0DnVuMwp8/f5uZv4dTS7Hzf8Ie5vR9oQx/z73iakiKUIpDQc\r\n0/sUJSKijOQRT1xZQmkw5+QClmbjLLS3ZXn9kMt0bbxZRPkcV2A=\r\n-----END RSA PRIVATE KEY-----\r\n"
                }
            ]
        },
    ]
}
```

#### 3.27.3.4删除http server ssl证书

删除只会在reload后生效并且有效果

```
curl -X PUT http://127.0.0.1:8443/ssl
Content-Type: application/json

{
    "listens": [
        "0.0.0.0:443"
    ],
    "serverNames": [
        "aaaa"
    ],
    "type": "del",
    "cert_info": {
         "cert_type": "regular",
         "certificate": "data:-----BEGIN CERTIFICATE-----\r\nMIIEITCCAwmgAwIBAgIUK6l+HK4KR/hLosx2XqNTaHLUkO0wDQYJKoZIhvcNAQEL\r\nBQAwZTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaUppbmcxEDAOBgNVBAcTB0Jl\r\naUppbmcxDjAMBgNVBAoTBW5naW54MQ8wDQYDVQQLEwZ0bWxha2UxETAPBgNVBAMT\r\nCG5naW54LWNhMCAXDTIyMDgwOTA5NDEwMFoYDzIxMjIwNzE2MDk0MTAwWjBiMQsw\r\nCQYDVQQGEwJDTjEQMA4GA1UECBMHQmVpSmluZzEQMA4GA1UEBxMHQmVpSmluZzEO\r\nMAwGA1UEChMFbmdpbngxDzANBgNVBAsTBnRtbGFrZTEOMAwGA1UEAxMFbmdpbngw\r\nggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC0hs5E8ZZT+ZU/DvvIRrSL\r\nokk6OE5EA4URo6zawqguKRlTc1USgfMdZFwR2znCxjFcsk7W/FSWqA5k5zyePYL8\r\n/0lhn3i/GQGczfa9dMZZEtN9EMD6TPOiDI/CaWAFc3U7qE+SYckUHLu/lMX8LshR\r\ndNkGecq41Ck5Sftm3xwy4cmpcCIyMpNumHrjJyxkp3S/QGewgZRQypntG9R34EKj\r\nQNFnovsn39TeeG+DBjS5m1+roth0JJ6DbemB47hy/uaEwAQpobN1etx20wCwxRO7\r\n7Huuh4kmWXjUol0nvzXM3ZZX2QnHn4jIg/ALS7O6p86z+m4vEHK4hRfWf52xGjMv\r\nAgMBAAGjgckwgcYwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMB\r\nBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBQz6z5AIFRrP+KD1gBt\r\nW6j5NEMzwDAfBgNVHSMEGDAWgBQ64+C++PsR6pDZoQ7EORozSJ8d9DBHBgNVHREE\r\nQDA+ggkxOTIuMTY4LiqCDiouY2x1c3RlcjEuY29tgg0qLmNsdXN0ZXIuY29tggwq\r\nLnRtbGFrZS5jb22HBH8AAAEwDQYJKoZIhvcNAQELBQADggEBADouKA9fFflNAs2u\r\nu7yj4cGNG+sC8r1wDWyGiXGDzvaB7C/sPTZiCgh8hI5gxzOSC+Se51kNFYymN9g4\r\nBPfB//SZ+CxO60BOt5eFF4PxIW7+d8gGFJr7Yx55QGA9DrvGi/LEiwjV3bQaezjN\r\noXUkfFgyKv79AfNFShkUPiHCklnfyqcCF+GEKxNC1q4wyClUt2EGtwfiO4YR85s0\r\n7jQ2BKJOsw3LLd8pZSvOXYu/I9DoO5w3gM5TgjnJlQPq+bmgqJ7bfWbSpFo70cdF\r\nFLfE9MLpwhDjVQEJ6DOmZqU1dVdNtwLrUrm0/8xkkmac0/8BaEYyPy1SNMGJTMGE\r\nW8psxto=\r\n-----END CERTIFICATE-----\r\n",
         "certificateKey": "data:-----BEGIN RSA PRIVATE KEY-----\r\nMIIEogIBAAKCAQEAtIbORPGWU/mVPw77yEa0i6JJOjhORAOFEaOs2sKoLikZU3NV\r\nEoHzHWRcEds5wsYxXLJO1vxUlqgOZOc8nj2C/P9JYZ94vxkBnM32vXTGWRLTfRDA\r\n+kzzogyPwmlgBXN1O6hPkmHJFBy7v5TF/C7IUXTZBnnKuNQpOUn7Zt8cMuHJqXAi\r\nMjKTbph64ycsZKd0v0BnsIGUUMqZ7RvUd+BCo0DRZ6L7J9/U3nhvgwY0uZtfq6LY\r\ndCSeg23pgeO4cv7mhMAEKaGzdXrcdtMAsMUTu+x7roeJJll41KJdJ781zN2WV9kJ\r\nx5+IyIPwC0uzuqfOs/puLxByuIUX1n+dsRozLwIDAQABAoIBAFh+VJLbUnOrvwuA\r\nTtBoSIzCat8NRuB0UUDKWSuLjGHEZ9POj39ZEFHyJmfibTgba4sjJR6h5t1LWHMC\r\nH2b6hEF86v3d7JTQr0esdy18FtcHMYD3O4H3Qt7HBZmpihZh+K/b29XH9YfUZfyN\r\n81ehnzS+8LwJ6+QarHKW35QX/ny5+pCw+G97cjdE7sClZnryd1KseWubAXDLSgmQ\r\nJcjXMMkwjtd2x/w23ZT97WMEcToL1wMw4hF/tHtu9u3UHMhzLHgrfrkW4QmACbLF\r\n8MJR94wzscJrUEviz5fV+3ZTmK+lujGmO/KkOpNAPGf8QAeBfa/qyBTQCQgCdxRo\r\n8hrDT4ECgYEA5vLsDDWiAROvvPaRkCv0y0UWkdgyBxv2T6pjDJZzMxb0yKZ48E6l\r\nh0PI9bfn0KbI1ZZviiZg20oZ/Keo2wLYa+W00rZhWljWUh0Ivd/5vTPHBmpsXl6+\r\nRK49aNVkfqlleherhfrYUkHKKDeg4lKVY3fwAlAQeg/y4Ds/8k3HUuECgYEAyBu8\r\ns7KuO7sAaJVj+QFfRrR6u2eJD48HPB4UoeDYMaKAuPTiM5/AUjH/YdD4E2lGAyJJ\r\nJVN413yu3Dk9UIAivgqCROepAcnnZ1BwE2qI/gfr2GD95o8JQSphze0pC/6WS/jG\r\niBR+j7MGWIkHJHEIUjOrf8eqtVLAywjxz9rPWA8CgYBTkuzgrjfl893Qn9mlNoLr\r\nXCECvh28fN3xjlMxpvAhONl0EuoI7CzyehEq+lYlJ3Xd9QaAE8tRD8u/plxwhOMU\r\niJea+OzZ6PQF2wPi0j5pvWb0Z2a378kiyXrniPFI9LwIJrCnV1MY0T36t8a8n+33\r\nhNuRuq97vHHDuy003fiXgQKBgGKzM6cauc+iU/hBvzbBk4HnYSXwUm1HKdVgLOMP\r\naPNKaN1RhATchdrE6GcR0FqasTq4fYWYn2ECEalz3idHnFtKCaj87qKAOM//n9gj\r\n0wAhXhWy+WjwIitvQSB2Gqnc37sHML1MBoTQU4/1vn0d93G8JJn5HN0kvQ0oE0Vn\r\ncp/HAoGAGCxpM24rHBK0HmlABZd8ZfLW/sW4Y7bMsQmrOVuPZcTWcO5HzkkiGzTc\r\n8DCDDeGdfJUC9NaM0DnVuMwp8/f5uZv4dTS7Hzf8Ie5vR9oQx/z73iakiKUIpDQc\r\n0/sUJSKijOQRT1xZQmkw5+QClmbjLLS3ZXn9kMt0bbxZRPkcV2A=\r\n-----END RSA PRIVATE KEY-----\r\n"
    }
}
```

## 3.28HTTP3 协议支持

### 3.28.1HTTP3 简介

HTTP3 相关协议主要包括 QUIC 协议([RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000), [RFC 9001](https://datatracker.ietf.org/doc/html/rfc9001), [RFC 9002](https://datatracker.ietf.org/doc/html/rfc9002) [RFC 9221](https://datatracker.ietf.org/doc/html/rfc9221) [RFC 8899](https://datatracker.ietf.org/doc/html/rfc8899)). 具体的HTTP/3 ([RFC 9114](https://datatracker.ietf.org/doc/html/rfc9114)), 以及QPACK ([RFC 9204](https://datatracker.ietf.org/doc/html/rfc9204)). 这些协议在阿里巴巴的[xquic项目](https://github.com/alibaba/xquic)的doc下有翻译成中文的协议，不过版本不是最新的。还可以参考[深入剖析HTTP3协议](https://zhuanlan.zhihu.com/p/431672713)。

目前在nginx1.25.0中，已经支持http3/quic协议了，不过NINGX宣称目前对HTTP3的支持还是实验性的。

对比之前的quic分支，去除了server端push的功能，原因是浏览器（chrome）中去除了对这个功能的支持。此外还去除了stream的quic支持，开发者说可能会考虑在以后再加回来，之前开发主要是调试quic协议。

QUIC(Quick UDP Internet Connection)是谷歌推出的一套基于UDP的传输协议，它实现了TCP + HTTPS + HTTP/2的功能，目的是保证可靠性的同时降低网络延迟。因为UDP是一个简单传输协议，基于UDP可以摆脱TCP传输确认、重传慢启动等因素，建立安全连接只需要一的个往返时间，它还实现了HTTP/2多路复用、头部压缩等功能。

众所周知UDP比TCP传输速度快，TCP是可靠协议，但是代价是双方确认数据而衍生的一系列消耗。其次TCP是系统内核实现的，如果升级TCP协议，就得让用户升级系统，这个的门槛比较高，而QUIC在UDP基础上由客户端自由发挥，只要有服务器能对接就可以。<img src="https://gitee.com/gebona/picture/raw/master/202308281111420.png" alt="截屏2023-08-28 11.10.58" style="zoom:50%;" />

### 3.28.2HTTP3 配置示例

```

helper broker modules/njt_helper_broker_module.so
conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so
conf/ctrl.conf;

load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 


load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

cluster_name helper;
node_name node1;

worker_processes auto;   

error_log  logs/error.log info;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    
    keepalive_timeout  65;

   
    server {
        server_name  localhost;

        # for better compatibility we recommend
        # using the same port number for QUIC and TCP
        listen 8443 quic  reuseport; # QUIC
        listen 8443 ssl;             # TCP
        ssl_certificate     /home/njet/test_http3/9521264_www.tmlake.com.pem;
        ssl_certificate_key /home/njet/test_http3/9521264_www.tmlake.com.key;
        #　ssl_protocols       TLSv1.1;
        ssl_protocols       TLSv1.3;

            location / {
                # advertise that QUIC is available on the configured port
                add_header Alt-Svc 'h3=":$server_port"; ma=86400';
                    root   html;
                    index  index.html index.htm;           
            }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

     }

}

```

### 3.28.3测试HTTP3功能

在centos7下的http3测试工具可从Gitee（https://gitee.com/njet-rd/http3-tools）下载 

#### 3.28.3.1基于curl_http3 测试HTTP3 

```
curl-http3 -vs -D/dev/stdout -o/dev/null --http3 https://quic.nginx.org
* processing: https://quic.nginx.org
*   Trying 35.214.218.230:443...
*  CAfile: /etc/pki/tls/certs/ca-bundle.crt
*  CApath: none
*   Trying 35.214.218.230:443...
......
> Accept: */*
>
< HTTP/3 200
HTTP/3 200
......
```

#### 3.28.3.2基于hey进行压测 

```
hey -n 10 -c 5 -h3  https://quic.nginx.org
Summary:
  Total:        5.8759 secs
  Slowest:      3.9820 secs
  Fastest:      0.5170 secs  
  Average:      1.7755 secs  
  Requests/sec: 1.7019  
  Total data:   25190 bytes  
  Size/request: 2519 bytes

Response time histogram:
  0.517 [1]     |■■■■■■■■■■■■■  
  0.863 [1]     |■■■■■■■■■■■■■  
  1.210 [1]     |■■■■■■■■■■■■■  
  1.556 [2]     |■■■■■■■■■■■■■■■■■■■■■■■■■■■  
  1.903 [3]     |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■  
  2.250 [0]     |  
  2.596 [0]     |  
  2.943 [0]     |  
  3.289 [1]     |■■■■■■■■■■■■■  
  3.636 [0]     |  
  3.982 [1]     |■■■■■■■■■■■■
......
```

#### 3.28.3.3基于浏览器访问HTTP3 

```
在浏览器输入   https://www.tmlake.com:8443/
```

### 3.29.4http3配置指令

#### 3.29.4.1开启、关闭http3功能。

| Syntax:  | http3 on \| off; |
| -------- | ---------------- |
| Default: | http3 on;        |
| Context: | http, server     |

#### 3.29.4.2开启、关闭对QUIC协议对HTTP/0.9功能的支持。

| Syntax:  | http3_hq on \| off; |
| -------- | ------------------- |
| Default: | http3_hq off;       |
| Context: | http, server        |

#### 3.29.4.3设置在一个Connection内允许的请求stream数目最大值。

| Syntax:  | http3_max_concurrent_streams number; |
| -------- | ------------------------------------ |
| Default: | http3_max_concurrent_streams 128;    |
| Context: | http, server                         |

#### 3.29.4.4设置在读写QUIC streams时的缓冲区大小

| Syntax:  | http3_stream_buffer_size size; |
| -------- | ------------------------------ |
| Default: | http3_stream_buffer_size 64k;  |
| Context: | http, server                   |

#### 3.29.4.5设置在Server端保存客户端Cid数目的最大值 

| Syntax:  | quic_active_connection_id_limit number; |
| -------- | --------------------------------------- |
| Default: | quic_active_connection_id_limit 2;      |
| Context: | http, server                            |

#### 3.29.4.6开启、关闭ebpf支持

本功能只在 Linux 5.7+ 以上内核版本中支持

| Syntax:  | quic_bpf on \| off； |
| -------- | -------------------- |
| Default: | quic_bpf off；       |
| Context: | main                 |

#### 3.29.4.7开启关闭GSO功能

本功能只在支持UDP_SEGMENT功能的Linux（>= 4.18）版本中支持

| Syntax:  | quic_gso on \| off; |
| -------- | ------------------- |
| Default: | quic_gso off;       |
| Context: | http, server        |

#### 3.29.4.8设置host_key文件

设置文件保存在加密状态重置(encrypt stateless reset)及地址验证令牌(address validation tokens)过程中要用到的密钥。缺省情况下，这个密钥会随机产生。

| Syntax:  | quic_host_key file; |
| -------- | ------------------- |
| Default: | ——；                |
| Context: | http, server        |

#### 3.29.4.9开启关闭地址验证[QUIC Address Validation](https://datatracker.ietf.org/doc/html/rfc9000#name-address-validation) 特性

本特性主要用来是防止流量放大攻击。地址验证在连接建立期间和连接迁移期间进行。

| Syntax:  | quic_retry on \| off; |
| -------- | --------------------- |
| Default: | quic_retry off;       |
| Context: | http, server          |

## 3.29 sysguard_cpu 基于cpu使用率动态调整worker数量

### 3.29.1功能描述

在OpenNJet控制面通过动态加载模块so，能够通过配置指令，实现在cpu使用率触发配置阈值的情况下能够动态调整worker数量。NJet cpu使用率低的时候要降低worker数量，cpu使用率高的时候要增加worker数量

cpu使用率：OpenNJet所有worker进程的平均cpu使用率, 数据采集自/proc/{pid}/stat

系统cpu使用率：系统所有cpu核的综合使用率，数据采集自/proc/stat

### 3.29.2参考配置

```
load_module modules/njt_http_sendmsg_module.so;       #依赖该module
load_module modules/njt_ctrl_config_api_module.so;
load_module modules/njt_helper_health_check_module.so;
load_module modules/njt_http_upstream_api_module.so;
load_module modules/njt_http_location_api_module.so;
load_module modules/njt_http_ssl_api_module.so;
load_module modules/njt_doc_module.so;
load_module modules/njt_http_vtsd_module.so;
load_module modules/njt_sysguard_cpu_module.so;       #sysguard_cpu module


user nobody;
sysguard_cpu interval=1 low_threshold=11  high_threshold=20 worker_step=2 min_worker=3 max_worker=5  sys_high_threshold=120;

events {
    worker_connections  1024;
}
error_log         logs/error_ctrl.log info;

http {
    dyn_sendmsg_conf  conf/iot-ctrl.conf;
    access_log        logs/access_ctrl.log combined;

    include           mime.types;

    server {
        listen       8081;
        location /hc {
            health_check_api;
        }

        location /api {
             api write=on;
        }

        location /kv {
            dyn_sendmsg_kv;
        }

        location /config {
            config_api;
        }

        location /ssl {
            dyn_ssl_api;
        }

        location /doc {
            doc_api;
        }
          location /dyn_loc {
           dyn_location_api;
        }

        location /metrics {
            vhost_traffic_status_display;
            vhost_traffic_status_display_format html;
        }
  }

}


cluster_name helper;
node_name node1;
```

| 参数               | 类型 | 默认值              | 最小值 | 最大值 | 描述                                                         |
| ------------------ | ---- | ------------------- | ------ | ------ | ------------------------------------------------------------ |
| interval           | int  | 1 （单位为min分钟） | 1      | -      | 检测时间间隔，每隔interval去检测一次                         |
| low_*threshold*    | int  | 10(表示10%)         | 10     | -      | cpu使用率下限，低于该阈值时，自动减少worker_step个worker, 数量受min_worker限制 |
| high_*threshold*   | int  | 70(表示10%)         | 10     | -      | cpu使用率上限，高于该阈值时，自动增加worker_step个worker, 数量受max_worker限制 |
| worker_step        | int  | 1                   | 1      | -      | 每次增加或者减少得worker数量，worker数量受min_worker和max_worker数量限制 |
| min_worker         | int  | 1                   | 1      | -      | 最少worker数量限制                                           |
| max_worker         | int  | ncpu，系统cpu核数   | 1      | -      | 最多worker数量限制                                           |
| sys_high_threshold | int  | 80(表示80%)         | 10     | -      | 系统整体cpu使用率，超过该值时，不增加worker数量              |

- low_*threshold <= high_threshold*
- min_worker <= max_worker
- min_worker数量、max_worker数量与静态配置worker数量的关系？

​        未触发负载调整worker的时候，实际运行worker数量为初始静态配置数量

- 合理配置情况下，会符合正常逻辑，也就是cpu使用率高了，会增加worker数量，cpu使用率低了，会减少worker数量
- 不合理配置的情况下（比如静态初始配置小于min_worker或者大于max_worker, 也可能通过api修改动态worker数），可能会出现不在min_worker\max_worker的范围内的情况，

- ​      比如当前worker数量已经小于min_worker的时候触发减少worker调整的情况，worker数保持不变；

- ​      比如当前worker数量已经大于max_worker的时候触发增加worker调整的情况，worker数也保持不变；

- ### 3.29.3调用样例

- Wrk 一直压测，增加cpu使用率

- ```
	wrk -t 1 -c 100 -d 1000s -L http://192.168.40.90:8001/
	```

- ```
	# 此处日志是总cpu使用率100%
	2023/09/14 16:45:21 [info] 14016#0:  total cpu usage:100  usr:18459855  nice:2938  sys:5048704 idle:416296648 work:23511497  prev_work:23510134 total:439808145  pre_total:439806782 work-:1363 total-:1363
	
	#此处日志表示一共三个（14095 14096 14004）worker进程
	2023/09/14 16:45:21 [info] 14016#0: get all pids:14095_14096_14004_
	# 进程的cpu使用率
	2023/09/14 16:45:21 [info] 14016#0:  get process:14095 cpu_usage:31 utime:364 stime:276 cutime:0 cstime:0 work:640 pre_work:211 diff_work:429 diff_total:1363
	2023/09/14 16:45:21 [info] 14016#0:  get process:14096 cpu_usage:32 utime:379 stime:277 cutime:0 cstime:0 work:656 pre_work:218 diff_work:438 diff_total:1363
	2023/09/14 16:45:21 [info] 14016#0:  get process:14004 cpu_usage:32 utime:1167 stime:753 cutime:0 cstime:0 work:1920 pre_work:1481 diff_work:439 diff_total:1363
	2023/09/14 16:45:21 [info] 14016#0:  old pids:14095_14096_14004_ new pids:14095_14096_14004_
	
	#平均使用率
	2023/09/14 16:45:21 [info] 14016#0:  average cpu usage:31
	#worker触发规则，调整worker数量， 从3个到5个
	2023/09/14 16:45:21 [info] 14016#0:  adjust worker num from 3 to 5
	
	#此处是变为5个worker后的日志
	2023/09/14 16:45:36 [info] 14016#0:  total cpu usage:54  usr:18460323  nice:2938  sys:5049015 idle:416297294 work:23512276  prev_work:23511497 total:439809570  pre_total:439808145 work-:779 total-:1425
	2023/09/14 16:45:36 [info] 14016#0: get all pids:14095_14096_14215_14004_14216_
	2023/09/14 16:45:36 [info] 14016#0:  get process:14095 cpu_usage:11 utime:464 stime:343 cutime:0 cstime:0 work:807 pre_work:640 diff_work:167 diff_total:1425
	2023/09/14 16:45:36 [info] 14016#0:  get process:14096 cpu_usage:12 utime:485 stime:344 cutime:0 cstime:0 work:829 pre_work:656 diff_work:173 diff_total:1425
	2023/09/14 16:45:36 [info] 14016#0:  get process:14215 cpu_usage:7 utime:56 stime:55 cutime:0 cstime:0 work:111 pre_work:0 diff_work:111 diff_total:1425
	2023/09/14 16:45:36 [info] 14016#0:  get process:14004 cpu_usage:12 utime:1269 stime:822 cutime:0 cstime:0 work:2091 pre_work:1920 diff_work:171 diff_total:1425
	2023/09/14 16:45:36 [info] 14016#0:  get process:14216 cpu_usage:8 utime:60 stime:54 cutime:0 cstime:0 work:114 pre_work:0 diff_work:114 diff_total:1425
	2023/09/14 16:45:36 [info] 14016#0:  old pids:14095_14096_14004_ new pids:14095_14096_14215_14004_14216_
	
	
	#此处随着cpu使用率下降，又重新调整worker个数从5个到3个
	2023/09/14 16:45:36 [info] 14016#0:  average cpu usage:10
	2023/09/14 16:45:36 [info] 14016#0:  adjust worker num from 5 to 3
	2023/09/14 16:45:51 [info] 14016#0:  total cpu usage:1  usr:18460332  nice:2938  sys:5049031 idle:416298764 work:23512301  prev_work:23512276 total:439811065  pre_total:439809570 work-:25 total-:1495
	2023/09/14 16:45:51 [info] 14016#0: get all pids:14095_14096_14215_
	2023/09/14 16:45:51 [info] 14016#0:  get process:14095 cpu_usage:0 utime:465 stime:344 cutime:0 cstime:0 work:809 pre_work:807 diff_work:2 diff_total:1495
	2023/09/14 16:45:51 [info] 14016#0:  get process:14096 cpu_usage:0 utime:485 stime:345 cutime:0 cstime:0 work:830 pre_work:829 diff_work:1 diff_total:1495
	2023/09/14 16:45:51 [info] 14016#0:  get process:14215 cpu_usage:0 utime:57 stime:56 cutime:0 cstime:0 work:113 pre_work:111 diff_work:2 diff_total:1495
	2023/09/14 16:45:51 [info] 14016#0:  old pids:14095_14096_14215_14004_14216_ new pids:14095_14096_14215_
	2023/09/14 16:45:51 [info] 14016#0:  old pid:14004 need remove
	2023/09/14 16:45:51 [info] 14016#0:  old pid:14004 remove success
	2023/09/14 16:45:51 [info] 14016#0:  old pid:14216 need remove
	2023/09/14 16:45:51 [info] 14016#0:  old pid:14216 remove success
	2023/09/14 16:45:51 [info] 14016#0:  average cpu usage:0
	
	```

- ## 3.30 OpenNJet 向ADC注册

- ### 3.30.1功能描述

- #### 3.30.1.1 背景及需求

- 为支持部署更多的OpeNnjet 实例并且能够实现水平扩展，需要在和前端4层LB 协调的基础上，满足以下需求：

- 1.前端LB设备仅仅负责4层的流量转发

- 2.OpenNJet可根据业务规模做水平扩容

- 3.OpenNJet通过前端设备提供的API接口向前端设备进行注册

- 4.外部client 通过域名访问不同的服务

- #### 3.30.1.2 实现方案

- ![6308628e-a950-45da-9b6c-09905e616dbb](https://gitee.com/gebona/picture/raw/master/202310131640703.png)

- 流程说明

- 1. ADC创建应用池Pool1, 监听IP1:port1(应用负载部分创建虚拟服务器和应用池)
	2. ADC设置内部DNS服务器（全局负载部分设置侦听器）
	3. ADC设置域名（app1.api.exm.com）和应用池（Pool1）绑定关系（全局负载部分创建虚拟服务器和应用池）
	4. 内部DNS要注册到实际的外部DNS
	5. OpenNJet启动或者重启后（可以是原生的OpenNJet实例也可能是通过KIC启动的OpenNJet服务实例），调用API接口向应用池Pool1注册member（OpenNJet实际的ip和端口）
	6. client访问app1.api.exm.com, 通过外部dns，转发到内部dns，通过内部dns解析得到实际的应用池Pool1的IP1:port1，作为解析结果，返回给client
	7. client通过IP1:port1（host：app1.api.exm.com）访问，由ADC设备通过4层协议转发到实际的OpenNJet的ip和端口，并且proxy到实际的业务app1

- #### 3.30.1.3 测试设备

- https://192.168.40.181:10443

- username:admin

- password:YKADC_test!@#

- ### 3.30.2 参考配置

- | **Syntax**  | register config={config} server={server} port={port}  location={location}； |
	| ----------- | ------------------------------------------------------------ |
	| **Default** | register config=conf/register.json server=127.0.0.1 port=8081  location=/adc; |
	| **Context** | http                                                         |

参数说明：

| 参数     | 类型   | 默认值             | 说明               |
| -------- | ------ | ------------------ | ------------------ |
| config   | string | conf/register.json | 注册信息           |
| server   | string | 127.0.0.1          | 控制面ip           |
| port     | int    | 8081               | 控制面端口         |
| location | string | /adc               | 控制面注册location |

njet.conf 主配置不需要任何配置

```Bash
helper broker modules/njt_helper_broker_module.so
conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so
conf/ctrl.conf;

load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 

load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;

cluster_name helper;
node_name node1;
worker_processes auto;   

error_log logs/error.log info;
pid logs/njet.pid;   
events {
        worker_connections 1024;
}
http {
        dyn_kv_conf conf/iot-work.conf;
        include mime.types; 
        include conf.d/*.conf;
 upstream backend {
                server localhost:8002;
                server localhost:8003;
        }
  server {
                listen 8001;
                server_name localhost;

                location / {
                proxy_pass http://backend;
                          
                }
        }

server {
                listen 8002;
                server_name localhost;

                location / {
                        
                          return 200 "80028002";
                }
        }
 server {
                listen 8003;
                server_name localhost;

                location / {
                        
                          return 200 "80038003";
                }
        }

}        

   
```

ctrl.conf

```

load_module modules/njt_http_sendmsg_module.so; 
load_module modules/njt_ctrl_config_api_module.so; 
load_module modules/njt_helper_health_check_module.so; 
load_module modules/njt_http_upstream_api_module.so; 
load_module modules/njt_http_location_api_module.so; 
load_module modules/njt_doc_module.so; 
load_module modules/njt_http_vtsd_module.so; 

load_module modules/njt_http_register_module.so;
load_module modules/njt_http_lua_module.so;

cluster_name helper; 
node_name node1; 
error_log logs/error-ctrl.log info; 
events { 
        worker_connections 1024; 
}

http { 
        dyn_sendmsg_conf conf/iot-ctrl.conf; 
        config_req_pool_size 1000; 
        access_log logs/access_ctrl.log combined; 
        register config=conf/register.json server=127.0.0.1 port=8088  location=/adc;
        include mime.types; 
      
        include mime.types; 
        lua_package_path "/home/njet/test_adc/work/etc/njet/lualib/lib/?.lua;$prefix/scripts/?.lua;;"; 
        server { 
                listen 8088; 
    
    
                location /kv { 
                        dyn_sendmsg_kv; 
                }
                location /config { 
                        config_api; 
                }
                location /hc { 
                        health_check_api; 
                }
                location /api { 
                        api write=on; 
                }
                location /doc { 
                        doc_api; 
                } 
                location /dyn_loc { 
                        dyn_location_api; 
                }
                location /metrics { 
      vhost_traffic_status_display; 
                        vhost_traffic_status_display_format html;
                } 
   
           location /adc {
            content_by_lua_file objs/scripts/http_register.lua;
        }
        } 
}
```

其中register.json格式如下：

```
{
    "type":"ADC",
    "username":"admin",
    "password":"YKADC_test!@#",
    "address":"192.168.40.136:80",
    "login_api": "https://192.168.40.180:10443/adc/v3.0/login",
    "pool_register_api" : "https://192.168.40.180:10443/adc/v3.0/slb/pool/9ae62b69-a98e-42c0-9b8b-30a53f9e5139/rserver/"
}
```

注册信息字段说明：

| 字段              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| type              | 设备类型，目前支持ADC设备                                    |
| username          | 设备登录用户名                                               |
| password          | 设备登录密码                                                 |
| address           | 注册的OpenNJet实例地址                                       |
| login_api         | 登录api（URL中的IP和端口为ADC设备的地址）                    |
| pool_register_api | 应用池注册api，路径里使用的时应用池的uuid，可以通过通过浏览器在ADC页面按F12查看相关http请求的响应等方式得到对应应用池的uuid |

### 3.30.3 ADC 配置步骤

##### 3.30.3.1 ADC设备安装完只有管理口ip，需要配置业务口ip， 步骤如下：

管理口ip 安装完默认是10.10.254.254， 访问其web页面： https://10.10.254.254:10443

 通过网络->vlan管理-> 设置，然后添加一个主ip，配置一个业务口ip，需要与管理口ip是不同网段，此处设置为192.168.40.180， web配置设置为on， 此后可以通过该ip也可以访问其web界面，地址: https://192.168.40.180:10443

![07e7585f-579e-48f5-8784-4cb9701b7be3](https://gitee.com/gebona/picture/raw/master/202310131646380.png)

#### 3.30.3.2配置侦听ip（也就是内部dns设置）

全局负载本身就是dns， 此处用业务口ip（192.168.40.180）作为dns ip使用（理论上随便设置一个虚拟ip即可）

![113ad3b4-b4cb-4beb-becb-c6abeee8d9f3](https://gitee.com/gebona/picture/raw/master/202310131647657.png)

#### 3.30.3.3配置域名和公网IP

比如此处想注册一个app1.njet.com的域名，公网ip地址假设为192.168.40.222:5000

先在全局负载创建应用池（设置公网ip地址192.168.40.222:5000）， 

![f0429491-05e5-4567-861c-032749a7b656](https://gitee.com/gebona/picture/raw/master/202310131649912.png)

然后在设置处添加一个成员，ip设置为公网ip地址192.168.40.222:5000

![2da24ef3-be41-4381-9457-20d495bc7998](https://gitee.com/gebona/picture/raw/master/202310131649699.png)

然后在全局负载虚拟服务器添加一个虚拟服务器（设置域名 app1.njet.com），引用上面创建的应用池完成域名和ip的关联

![10265880-20e0-4fc9-9ef1-bcbbaddd1b5a](https://gitee.com/gebona/picture/raw/master/202310131650523.png)

#### 3.30.3.4 配置公网ip和njet应用池

应用负载添加应用池(可以通过浏览器F12查看相关http请求的响应得到对应应用池的uuid信息，此uuid作为OpenNJet注册时使用)

比如此处想为域名app1.njet.com 的服务设置一个OpenNjet应用池（取名叫opennjet_pool），以后OpenNJet注册的地址会添加到此应用池，作为该应用池的一个member成员

![748208d7-defa-4b18-ad30-ace61d524928](https://gitee.com/gebona/picture/raw/master/202310131651475.png)

我们之后OpenNJet完成注册应用池，在设置处就会看到我们注册的地址，可以看到我们OpenNJet注册了三个地址

![img](https://gitee.com/gebona/picture/raw/master/202310131651396)

应用负载添加虚拟服务器，设置公网ip地址192.168.40.222:5000，应用池引用上面创建的应用池opennjet_pool

![img](https://gitee.com/gebona/picture/raw/master/202310131651617.(null))

经过上面的步骤就完成了所有的配置，可以通过在自己的机器，设置dns为侦听器地址（192.168.40.180），然后通过curl 去访问OpenNJet 服务：比如，

curl http://app1.njet.com:5000/

app1.njet.com 为我们上面配置的OpenNjet服务的域名，5000端口为公网ip地址的端口，通过上面的访问，就会将请求转发到我们注册的OpenNJet服务

### 3.30.4 调用样例

将访问机器的dns设置为ADC设备设置的dns（此处测试dns地址为192.168.40.180）

![7a591a28-32d3-445e-a17e-0259dcee5b35](https://gitee.com/gebona/picture/raw/master/202310131652356.png)

然后curl访问域名，实际可以访问到真正的OpenNJet服务(域名+公网port， 此处8888为在ADC设备上配置的公网端口)， 会实际访问8002和8003（默认轮询算法）

![dc15382b-ea39-426c-8a59-ef8e2365ac1b](https://gitee.com/gebona/picture/raw/master/202310131653400.png)

## 3.31 Proxy_protocol V2功能

代理协议(Proxy protocol)，是HAProxy的作者Willy Tarreau于2010年开发和设计的一个Internet协议，通过为tcp添加一个很小的包头信息，来方便的传递客户端信息（协议栈、源IP、目的IP、源端口、目的端口等)，在网络情况复杂又需要获取用户真实IP时非常有用。

代理协议分为V1和V2两个版本，V1是人类易读的，V2是二进制格式的，并且支持tlv 功能。

**v1介绍 :**

Proxy protocol V1的格式如下:

PROXY 协议栈 源IP 目的IP 源端口 目的端口rn

例如：

PROXY TCP4 192.168.0.1 192.168.0.11 56324 443\r\n    GET / HTTP/1.1\r\n    Host: 192.168.0.11\r\n    \r\n

**v2 介绍 :** 

相比V1，v2利用二进制格式以实现更高的解析效率，并可以增加特定的扩展属性（TLV）

![](https://gitee.com/gebona/picture/raw/master/202312291016039.png)

![](https://gitee.com/gebona/picture/raw/master/202312291016975.png)

在标准的地址信息后，如果有额外的数据，则该数据是TLV数组。每个tlv结构包括类型（type），length，value。 其中如下的类型数值已经被标准化，应用应该遵循该type的含义，不应用作他途。

```
        #define PP2_TYPE_ALPN           0x01
        #define PP2_TYPE_AUTHORITY      0x02
        #define PP2_TYPE_CRC32C         0x03
        #define PP2_TYPE_NOOP           0x04
        #define PP2_TYPE_UNIQUE_ID      0x05
        #define PP2_TYPE_SSL            0x20
        #define PP2_SUBTYPE_SSL_VERSION 0x21
        #define PP2_SUBTYPE_SSL_CN      0x22
        #define PP2_SUBTYPE_SSL_CIPHER  0x23
        #define PP2_SUBTYPE_SSL_SIG_ALG 0x24
        #define PP2_SUBTYPE_SSL_KEY_ALG 0x25
        #define PP2_TYPE_NETNS          0x30
```

常见应用：

- 传递client的real ip（尤其是非http类的应用，http类可以用x-forward-realip等header传递）
- 代理协议版本2 支持额外的TLV 字段。他可以被前端ssl协议卸载器转发客户端证书信息转发到后端非http 协议的后端服务器。VerneMQ MQTT 代理就是示例之一，它可以利用代理协议版本2，获取客户端的证书的详细信息，进行身份授权。
- 用于在K8s环境中，ingress 对非Http 协议的ssl 终止。
- GCP/amazon/arure 利用特定TLV 实现负载均衡  

### 3.31.2 配置说明

|                   | 必填 | 配置说明                                                     |
| ----------------- | ---- | ------------------------------------------------------------ |
| proxy_protocol    | 是   | 开启proxy_protocol                                           |
| proxy_pp2         | 是   | 开启proxy_protocol v2。  on 开启， off 关闭                  |
| proxy_pp2_set_tlv | 否   | proxy_pp2_set_tlv  key  value 格式。key  是 16 进制（两位，要避免和标准的冲突）。  value 可以是常量，变量。例如：proxy_pp2_set_tlv  0x31    test; 获取：$proxy_protocol_tlv_0x31  前缀变量。注意：包含前面的源ip，目标ip，源端口，目标端口，以及tlv 值，总长度不能超过 4096 |

```
http {
    dyn_kv_conf conf/iot-work.conf;
    include       mime.types;
    default_type  application/octet-stream;
    
    access_log  logs/access.log;

    variables_hash_max_size  2048;
    
    sendfile        on;
    keepalive_timeout  65;

   server {
        listen 5555 proxy_protocol;
        server_name server-5555;
        location /test {     #return 返回结果
          return 200 "src:$proxy_protocol_addr:$proxy_protocol_port,dst:$proxy_protocol_server_addr:$proxy_protocol_server_port,tlv01:$proxy_protocol_tlv_0x31,tlv02:$proxy_protocol_tlv_0x41";
        }
}

stream {
      upstream backend1 {
         zone backend1_zone 128k;
         server 127.0.0.1:22222;
    }

     server {
        listen 22222 ;
        listen [::1]:22222
        proxy_protocol on;  #开启protocol 功能
        proxy_pp2  on;      #开启protocol V2功能
        njtmesh_dest on;
        set $test 123;
        set $test2 11111111111111111112;
        proxy_pp2_set_tlv  0x31    $test;   #设置tlv 字段名，以及值
        proxy_pp2_set_tlv  0x41    $test2;  #设置tlv 字段名，以及值
        proxy_pass 127.0.0.1:5555;
     }
}
```

### 3.31.3 调用样例

#### 3.31.3.1 IPv4访问stream模块22222端口

```
curl 127.0.0.1:22222/test

src:127.0.0.1:48304,dst:127.0.0.1:22222,tlv01:123,tlv02:11111111111111111112
```

#### 3.31.3.2 IPv6访问stream模块22222端口

```
curl -g -6 [::1]:22222/test

src:::1:48312,dst:::1:22222,tlv01:123,tlv02:11111111111111111112
```

#### 3.31.3.3 NJetSidecar 应用v2传递原始目的地址

Ingress sidecar配置

```
stream {
 
     server {
        listen 15006  ;
        njtmesh_dest on;    #参考下面该指令的备注 
        proxy_protocol on;  #开启protocol 功能
        proxy_pp2  on;      #开启protocol V2功能 
        
        proxy_pp2_set_tlv  0x41    $njtmesh_port;  #设置tlv 字段名，以及值
        proxy_pass 127.0.0.1:22222;
     }
}
```

Ingress policy的配置

```


http {
 
   server {
        listen 5555 proxy_protocol;
        server_name server-5555;
        location /{ 
          #todo: 其他的业务配置指令
          proxy_pass 127.0.0.1:$proxy_protocol_tlv_0x41;
        }
}
```

注：#  njtmesh_dest NJet自定义指令，是用于获取firewall规则设置的原始目标信息的指令，开启后，可以通过$njtmesh_port 获取原始的目标端口，同样$njtmesh_ip，可以获得原始的目的地址。

## 3.32 动态 Map 模块

### 3.32.1 功能描述

OpenNJet 配置文件中可以通过Map 指令，检测已有的变量，根据匹配到的值，映射到自定义变量，并对自定义变量赋值。该动态模块可以对已经配置的Map映射规则进行更改

#### 3.32.1.1 当前实现的一些限制

目前的实现只能修改静态配置文件中已有的映射变量中的映射关系，还未实现动态添加新的映射变量。动态添加新的map 变量需要后续开发支持，现阶段不支持。

isVolatile , hostnames 目前也只是用来显示静态配置中这两个flag 的设置，目前这个阶段不要进行修改。

下文提到的 变更json 报文中， keyFrom， keyTo 的值需要是已有的 map 配置中定义的值，才能更新成功。

### 3.32.2配置说明

该动态模块，使用 /config/2/config api 接口 进行动态值的修改，因此需要配置 helper ctl 模块， helper broker 模块， ctl 模块中需要加载 config 模块， 数据面中需要加载 njt_http_dyn_map_module.so

njet.conf 中需要包含：

```
load_module /usr/lib/njet/modules/njt_http_dyn_map_module.so;
helper ctrl /usr/lib/njet/modules/njt_helper_ctrl_module.so njet_ctrl.conf;
helper broker /usr/lib/njet/modules/njt_helper_broker_module.so;
```

ctrl.conf 中需包含：

```
load_module /usr/lib/njet/modules/njt_http_sendmsg_module.so;
load_module /usr/lib/njet/modules/njt_ctrl_config_api_module.so; 
。。。
http {
    server {    
        location /config {
            config_api;
        }
    }
}
```

```
http {
    include mime.types;
   map $arg_a $testv {
        default    00;
        aa         11;
        bb         22;
        cc         33;
        dd         44;
       }

   server {
     server_name loaclhost;
      listen 8002;

       location / {
             return 200 "map test,mapped value is ${testv} \n";

      }

    }
}

```

### 3.32.3 调用样例

#### 3.32.3.1 查询

```
 curl -X 'GET'   'http://192.168.40.158:8088/config/2/config/http_dyn_map'  |jq
```

![9c3bb53b-a091-445c-bbf7-41b3f84b007e](https://gitee.com/gebona/picture/raw/master/202310131700979.png)

```
curl -k https://127.0.0.1:18080/?a=aa

curl -k https://127.0.0.1:18080/?a=bb

curl -k https://127.0.0.1:18080/?a=
```

![f85db3df-7f7c-4e9f-8bc9-80e5ef524fb9](https://gitee.com/gebona/picture/raw/master/202310131700548.png)

#### 3.32.3.2 修改

```
curl -X 'PUT' \
  'http://192.168.40.158:8088/config/2/config/http_dyn_map' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "maps": [
    {
      "keyFrom": "$arg_a",
      "keyTo": "$testv",
      "values": [
        {
          "valueFrom": "default",
          "valueTo": "success"
        },
        {
          "valueFrom": "aa",
          "valueTo": "1111"
        },
        {
          "valueFrom": "bb",
          "valueTo": "2222"
        }
       
      ],
      "isVolatile": false,
      "hostnames": false
    }
  ]
}'
```

![](https://gitee.com/gebona/picture/raw/master/202312291014737.png)

## 3.33 HA/MA配置信息同步

### 3.33.1 功能描述

主备节点能够实现动态配置的同步。主节点通过动态配置接口（声明式api或者命令式api）动态更新配置，然后backup节点能够及时同步这些配置。主节点宕机，backup节点动态更新配置，再主节点重新起来后，也能够及时同步更新的配置。最终保证主备节点都能够保证彼此配置的最新同步。

### 3.33.2 配置说明

主节点： 192.168.40.158

broker进程配置如下

```
#配置主节点监听端口和ip
listener 1883 192.168.40.158

#配置本地socket地址，用于本地worker进程通信
listener 0 /home/njet/test_hamma/data/mosquitto.sock

log_dest file logs/mosquitto.log
log_type debug
log_type information
log_type error
log_type warning
log_type notice

allow_anonymous true
persistence true
autosave_on_changes true
autosave_interval 1
persistence_location /home/njet/test_hamma/data/
```

backup 节点：192.168.40.157

broker进程配置如下：

```
listener 8000 192.168.40.157

listener 0 /home/limin/test_hamma/data/mosquitto.sock
connection bridge-backup
restart_timeout   3
address 192.168.40.158:8000

topic /dyn/# both 0
topic /ins/# both 0

log_dest file logs/mosquitto.log
log_type debug
log_type information
log_type error
log_type warning
log_type notice

allow_anonymous true
persistence true
autosave_on_changes true
autosave_interval 1
persistence_location /home/limin/test_hamma/data/
```

### 3.33.3调用样例

#### 3.33.3.1**声明式API-主节点更新消息，backup节点同步消息**

主节点修改配置信息：

![](https://gitee.com/gebona/picture/raw/master/202312291020665.png)

backup 节点查看配置信息：

![7dbf52ee-5323-4eff-8847-8609a31a210e](../../../../../7dbf52ee-5323-4eff-8847-8609a31a210e.png)

#### 3.33.3.2**命令式**API-主节点更新消息，backup同步消息

主节点更新消息动态增加location：

![8a17e05c-7a76-4aa7-bd35-75f2c06180a7](../../../../../8a17e05c-7a76-4aa7-bd35-75f2c06180a7.png)

backup节点查看是否同步location 配置信息以及增加的location 数量

![8e89d0d8-989e-413f-a00b-1f79d84e7517](../../../../../8e89d0d8-989e-413f-a00b-1f79d84e7517.png)

## 3.34 range内核端口数据转发模块

### 3.34.1 功能描述

在很多时候，比如流量劫持、ftp被动模式代理等功能需要能够支持流量端口转发。比如需要将10000到11000端口范围的所有流量都统一转到12000端口上，然后在12000端口上接收所有的报文进行后续处理。

Privilege Agent 进程将以 root 用户运行，并监听 /worker_a/# 主题， 所有动态配置的消息先发往该进程，处理后，由该进程中的 kv 模块将需持久化的消费发往  /dyn/${module} 主题

### 3.34.2 配置说明

#### njet.conf （range+ftp)

```

helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;
load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_range_module.so; 
load_module modules/njt_http_dyn_range_module.so;  
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_location_module.so; 
load_module modules/njt_http_dyn_bwlist_module.so; 

load_module modules/njt_dyn_ssl_module.so; 
load_module modules/njt_http_vtsc_module.so; 

cluster_name helper; 
node_name node1;

worker_processes auto; 
#user root; 

error_log logs/error.log info; 
pid logs/njet.pid; 

events {
        worker_connections 1024; 
  use epoll;
}
#配置两条range指令
range type=tcp src_ports=11000:12000 dst_port=13000;
range type=tcp src_ports=10000 dst_port=14000;

#默认路径为/usr/sbin/iptables， 如果系统路径不一致需要调用该指令设置下路径
range iptables_path=/usr/sbin/iptables;

http { 
        dyn_kv_conf conf/iot-work.conf; 
        include mime.types; 
        include conf.d/*.conf;
        
        server {
        listen       8100;
        server_name  localhost;

        location / {
           # root   html;
            #index  index.html index.htm;
             return 200 "8100";

        }
  
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
        
}


```

ctrl.conf

```
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_ctrl_config_api_module.so;
load_module modules/njt_helper_health_check_module.so;
load_module modules/njt_http_upstream_api_module.so;
load_module modules/njt_http_location_api_module.so;
load_module modules/njt_doc_module.so;
load_module modules/njt_http_vtsd_module.so;
load_module modules/njt_http_range_api_module.so;
cluster_name helper;
node_name node1;
error_log logs/error-ctrl.log info;
events {
        worker_connections 1024;
}

http {
        dyn_sendmsg_conf conf/iot-ctrl.conf;
        access_log logs/access_ctrl.log combined;
        include mime.types;

        server {
                listen 8081;
                location /kv {
                        dyn_sendmsg_kv;
                }
                location /config {
                        config_api;
                }
                location /hc {
                        health_check_api;
                }
                location /api {
                        api write=on;
                }
                location /doc {
                        doc_api;
                }
                location /dyn_loc {
                        dyn_location_api;
                }
                location /metrics {
                        vhost_traffic_status_display;
                        vhost_traffic_status_display_format html;
                }
                location /range {
                         dyn_range_api;      #配置range api开关
                }
       }
}
~   
```

该功能的实现依赖于iptables， 而普通用户一般没有iptables命令的使用权限，所以如果普通用户启动njet，那么就需要普通用户有使用iptables命令的权限,需要在root用户下，执行如下命令：

```
 sudo /usr/sbin/setcap cap_dac_override,cap_dac_read_search,cap_net_bind_service,cap_net_admin,cap_net_raw,cap_setuid=eip  sbin/njet
```

### 3.34.3 调用样例

#### 3.34.3.1 启动OpenNJet前

查询系统iptables 规则

![chaxun](https://gitee.com/gebona/picture/raw/master/202312291027569.png)

#### 3.34.3.2 启动OpenNJet后

重新查询iptables规则，发现两条规则生效

![35d291ed-cf85-48ae-83ce-546701a6a563](https://gitee.com/gebona/picture/raw/master/202312291028474.png)

#### 3.34.3.3 Reload重启后

重新查询，发现两条规则还在

![fb373191-b32e-4550-907f-606e907d7174 (1)](https://gitee.com/gebona/picture/raw/master/202312291030615.png)

#### 3.34.3.4 停止OpenNJet后

重新查询，两条规则删除

![fea3fe96-6094-403e-bd36-7d339dfca4cb](https://gitee.com/gebona/picture/raw/master/202312291031363.png)

### 3.34.4 命令式api 动态配置端口转发规则

#### 3.34.4.1 需求：

能够通过api接口查询全量的range 配置规则

能够通过api增加或者删除一条range配置规则

根据type、src_ports、dst_port三个字段来确定一条规则

#### 3.34.4.2 配置

ctrl控制面配置：

```
 ...
 load_module modules/njt_http_range_api_module.so;   #load range api module
 
 ...
    server {
        listen       8081;
        
        location /range {
            dyn_range_api;      #配置range api开关
        }
    }
...
```

njet.conf 配置

```
load_module modules/njt_range_module.so;           #range 模块
load_module modules/njt_http_dyn_range_module.so;  #range 动态修改模块，依赖于上面的range模块

user nobody;

events {
    worker_connections  1024;
}
error_log         logs/error_privilege.log info;


#默认路径为/usr/sbin/iptables， 如果系统路径不一致需要调用该指令设置下路径
range iptables_path=/usr/sbin/iptables;

#range 静态配置
range type=tcp src_ports=11000:12000 dst_port=10001;
range type=tcp src_ports=10000 dst_port=14000;

http {
    access_log        logs/access_privilege.log combined;


    include           mime.types;

    server {
       。。。

   }

}


cluster_name helper;
node_name node1;
```

#### 3.34.4.3 动态API

查询：

GET  [http://192.168.40.136:8081/](http://192.168.40.136:8081/range)[range](http://192.168.40.136:8081/range)

```
{
  "ranges": [
    {
      "type": "tcp",
      "src_ports": "11000:12000",
      "dst_port": 10001
    },
    {
      "type": "tcp",
      "src_ports": "10000",
      "dst_port": 14000
    }
  ]
}
```

PUT  http://192.168.40.136:8081/range

```
#添加一条规则
{
    "action": "add",
    "type": "tcp",
    "src_ports": "11000:13000",
    "dst_port": 10001
}

#删除一条规则
{
    "action": "del",
    "type": "tcp",
    "src_ports": "11000:13000",
    "dst_port": 10001
}

```

| 参数      | 类型   | 必填 | 默认值 | 说明                                                         |
| --------- | ------ | ---- | ------ | ------------------------------------------------------------ |
| action    | string | 是   | -      | [add\|del]                                                   |
| type      | string | 是   | -      | [tcp\|udp], 指明是tcp还是udp数据                             |
| src_ports | string | 是   | -      | 支持单端口或者端口范围,格式如下：端口范围（冒号分隔）：     11000:12000一个端口:       11000 |
| dst_port  | int    | 是   | -      | 目标端口                                                     |

return：

```
{
    "code": 0,         #0 表示成功，   非0 表示失败
    "msg": "success"
}
```

错误码

| code | msg               | 描述                           |      |
| ---- | ----------------- | ------------------------------ | ---- |
| 0    | success           | 成功                           |      |
| 2    | -                 | 内存分配失败相关的一些错误信息 |      |
| 4    | rule is not found | 删除一个不存在的规则           |      |
| 4    | rule has exist    | 添加一个已经存在的规则         |      |
| 4    | 其他错误          |                                |      |

#### 3.34.4.4 swagger访问

可以通过doc模块提供的swagger界面操作

http://192.168.40.136:8081/doc/swagger/

#### 3.34.4.5 测试

通过swagger页面进行API测试

初始状态get 查询配置

![f1e344ea-c70e-4c54-bd82-8037ac31c8c6](https://gitee.com/gebona/picture/raw/master/202312291037212.png)

查询iptables

![64272893-62c4-4fde-a991-55fe78f018b9](https://gitee.com/gebona/picture/raw/master/202312291039742.png)

通过swagger页面添加一个规则

![64aa401b-3cbb-41f6-8060-9976362e2ee9](https://gitee.com/gebona/picture/raw/master/202312291039739.png)

再次 查询配置和iptables

![64aa401b-3cbb-41f6-8060-9976362e2ee9](https://gitee.com/gebona/picture/raw/master/202312291040501.png)

![862e02d2-30ea-4641-ac3c-8495dab37364](https://gitee.com/gebona/picture/raw/master/202312291044770.png)

reload后再次查询，动态添加的配置和iptables都存在刚才动态添加的配置

![cf7df66c-c383-40fb-93b7-2e566b7f4fdb](https://gitee.com/gebona/picture/raw/master/202312291053384.png)

Stop OpenNJet后，iptables规则消失

![a122629d-06c2-4c43-8e04-f7965df438f7](../../../../../a122629d-06c2-4c43-8e04-f7965df438f7.png)

## 3.35 OpenNJet支持FTP/FTPS反向代理

### 3.35.1 功能描述

文件传输协议（File Transfer Protocol，FTP），基于该协议FTP客户端与服务端可以实现共享文件、上传文件、下载、删除文件。FTP服务器端可以同时提供给多人共享使用。

FTP服务是Client/Server（简称C/S）模式，基于FTP协议实现FTP文件对外共享及传输的软件称之为FTP服务器源端，客户端程序基于FTP协议，则称之为FTP客户端，FTP客户端可以向FTP服务器上传、下载文件。

FTP上传和下载文件需要有两个tcp连接：

一个是控制连接（port:21），控制连接用于在两个主机之间传输控制信息，如口令，用户标识，存放、获取文件等命令

一个是数据连接(port:20)。数据连接用于实际发送一个文件,发送完文件之后数据连接会关闭

关于数据连接的建立实际又有两种模式：主动模式和被动模式

#### 3.35.1.1主动模式 Port（服务端连接客户端）

客户端开启一个端口N（>1023）向服务端的21端口，建立连接，同时开启一个N+1端口监听，告诉服务端，我监听的是N+1端口，服务端接到请求之后，用自己的20端口连接到客户端的N+1端口，进行传输

#### 3.35.1.2 被动模式 Passive（客户端连接服务端）

客户端同时随机开启两个端口（比如1024，1025），一个端口（1024）跟服务端的21端口建立连接。服务端接到请求之后，随机会开启一个端口（1027）并告诉客户端我开启的是1027端口，客户端用另一个端口（1025）与服务端的（1027）端口进行连接，传输数据

#### 3.35.1.3 **隐式FTPS**

FTP服务器要求FTP客户必须初始化SSL握手过程并和FTP服务器之间建立安全的加密控制连接, 加密控制连接建立之后FTP命令才能够被送到FTP服务器. 如果FTP客户不支持SSL功能,或它和服务器之间没有建立安全的加密控制连接,FTP服务器将不对来自FTP客户的命令做出任何反应

#### 3.35.1.4 功能支持

- 支持明文的FTP反向代理
- 支持隐式FTPS模式（客户端和服务端先建立加密连接，再开始发送命令，数据连接也需要加密） 

### 3.35.2 配置说明

#### 3.35.2.1 系统依赖

```
需要系统上先执行如下命令：
modprobe nf_conntrack_ipv4
modprobe nf_conntrack_ipv6
```

#### 3.35.2.2配置示例

该功能的实现需要依赖range模块（可参考3.34小节）提供的端口流量转发功能

##### 3.35.2.2.1 明文FTP模式

OpenNJet 配置

```
helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;

load_module modules/njt_http_location_module.so;

load_module modules/njt_range_module.so;           #依赖range模块
load_module modules/njt_http_dyn_range_module.so;  #动态range模块，如果不需要动态修改，可不加载这个模块
load_module modules/njt_stream_ftp_proxy_module.so;     #加载ftp代理模块

user  root;
worker_processes  1;

cluster_name helper;
node_name node1;

error_log  logs/error.log info;
pid        logs/njet.pid;

events {
    worker_connections  1024;
}

#此处配置range规则，range模块会使用系统的iptables规则
range type=tcp src_ports=11000:12000 dst_port=10001; 
#默认路径为/usr/sbin/iptables， 如果系统路径不一致需要调用该指令设置下路径
range iptables_path=/usr/sbin/iptables;

stream {

        upstream ctl_upstream {
                hash $remote_addr consistent;
                server 192.168.40.91:21;
                
        }

        server {
                listen       21;
                ftp_ctrl  zone=ftp_zone:10M proxy_ip=192.168.40.136 min_port=11000 max_port=12000;
                proxy_pass  ctl_upstream;
                proxy_timeout 2000s;  #ftp连接默认10分钟会断开，如果传输大文件超过10分钟时，配置该指令
        }

        server {
                #不能再配置proxy_pass指令
                listen       10001;
                njtmesh_dest on;   #此指令必须配置
                ftp_data   zone=ftp_zone;
        }
}
```

vsftpd服务端建议配置：

```
#配置为ftp 被动模式
pasv_enable=YES

#配置被动模式数据端口范围
pasv_max_port=60000
pasv_min_port=60100

#这个配置是允许user_list文件里配置的用户可以访问
userlist_deny=no
```

##### 3.35.2.2.2 隐式FTPS模式

OpenNJet配置

```
helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;

load_module modules/njt_http_location_module.so;

load_module modules/njt_range_module.so;           #依赖range模块
load_module modules/njt_http_dyn_range_module.so;  #动态range模块，如果不需要动态修改，可不加载这个模块
load_module modules/njt_stream_ftp_proxy_module.so;     #加载ftp代理模块

user  root;
worker_processes  1;

cluster_name helper;
node_name node1;

error_log  logs/error.log info;
pid        logs/njet.pid;

events {
    worker_connections  1024;
}

#此处配置range规则，range模块会使用系统的iptables规则
range type=tcp src_ports=11000:12000 dst_port=10001; 
#默认路径为/usr/sbin/iptables， 如果系统路径不一致需要调用该指令设置下路径
range iptables_path=/usr/sbin/iptables;

stream {

        upstream ctl_upstream {
                hash $remote_addr consistent;
                server 192.168.40.91:990;
        }


        server {
                listen       21 ssl;
                
                ftp_ctrl  zone=ftp_zone:10M proxy_ip=192.168.40.136 min_port=11000 max_port=12000;

                ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
                ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
                ssl_certificate     /etc/vsftpd/.sslkey/vsftpd.pem;
                ssl_certificate_key     /etc/vsftpd/.sslkey/vsftpd.pem;

                proxy_ssl on;
                proxy_pass  ctl_upstream;
                proxy_timeout 2000s;  #ftp连接默认10分钟会断开，如果传输大文件超过10分钟时，配置该指令
        }

        server {
                listen       10001 ssl;
                njtmesh_dest on;         #一定要配置该指令，能够获取真实端口
                
                ftp_data   zone=ftp_zone;
                ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
                ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
                ssl_certificate     /etc/vsftpd/.sslkey/vsftpd.pem;
                ssl_certificate_key     /etc/vsftpd/.sslkey/vsftpd.pem;
                proxy_ssl on;
        }
}

```

vsftpd 配置示例

```
#配置服务端为隐式ftp
implicit_ssl=YES

#开启ssl
ssl_enable=YES
ssl_sslv2=YES
ssl_sslv3=YES
ssl_tlsv1=YES

#控制连接与数据连接不共用ssl session， 一定要配置
require_ssl_reuse=NO
#配置ssl证书
rsa_cert_file=/etc/vsftpd/.sslkey/vsftpd.pem
#一般隐式ftp 端口会只用990端口，如果不配置，vsftpd默认还是21，
#有的ftp服务可能会默认为990，此处明确指定端口为990
listen_port=990

#配置ftp server 日志文件
dual_log_enable=YES
vsftpd_log_file=/var/log/vsftpd.log
xferlog_file=/var/log/xferlog

#配置为ftp 被动模式
pasv_enable=YES

#配置被动模式数据端口范围，这个端口范围跟ftp代理的端口范围没有必然关系
pasv_max_port=60000
pasv_min_port=60100

#这个配置是允许user_list文件里配置的用户可以访问
userlist_deny=no
```

### 3.35.3 FTP服务器和客户端搭建

#### 3.35.3.1 FTP 服务端安装

##### 3.35.3.1.1 安装vsftpd

```
yum -y install vsftpd
systemctl start|stop|restart vsftpd
```

##### 3.35.3.1.2 Root  权限登录

修改vsftpd.conf文件：sudo vim /etc/vsftpd.conf，将 #write_enable=YES 前面的注释#去掉

```
sudo vim /etc/ftpusers  更改/etc/ftpusers，注释掉root用户
systemctl stop vsftpd
systemctl start vsftpd
```

#### 3.35.3.2FTP 客户端安装

```
yum -y install ftp
```

```
#连接 IP 端口
ftp 192.168.1.100 8887
#输入名称
ftp>name
#密码
ftp>password
#进入目录
ftp>cd /data/ftp
#下载本地
ftp>get test.txt#文件上传ftp>put myfile /data/ftp/
```

## 3.36 copilot:snmp

[SNMP](https://so.csdn.net/so/search?q=SNMP&spm=1001.2101.3001.7020)="Simple Network Management Protocol"="简单网络管理协议"，[应用层](https://so.csdn.net/so/search?q=应用层&spm=1001.2101.3001.7020)协议，传输层一般采用UDP.  

### 3.36.1 功能描述

SNMP 协议使网络管理员能够使用各种机制（包括电子邮件、警报和陷阱）从远程位置配置和监控他们的网络。

### 3.36.2 配置说明

conf/njet.conf 需要加载 snmp copilot

```
helper snmp modules/njt_helper_snmp_module.so modules/snmp.lua;
```

conf/snmp.conf 

```

-------------------------------------------------------------------------------
-- SmithSNMP Configuration File
-------------------------------------------------------------------------------

protocol = 'snmp'
port = 2222
metrics_url = "http://localhost:8081/metrics/format/json"
trap_server_ip = '127.0.0.1'
--trap_server_ip = '192.168.40.157'
trap_server_port = 162

mib_module_path = lvm.prefix..'modules/mibs'

--mib_module_path = './modules/mibs'
communities = {
  { community = 'public', views = { ["."] = 'ro' } },
  { community = 'private', views = { ["."] = 'rw' } },
}

users = {
  { user = 'roNoAuthUser', views = { ["."] = 'ro' } },
  { user = 'rwNoAuthUser', views = { ["."] = 'rw' } },
  { user = 'roAuthUser', auth_mode = "md5", auth_phrase = "roAuthUser", views = { ["."] = 'ro' } },
  { user = 'rwAuthUser', auth_mode = "md5", auth_phrase = "rwAuthUser", views = { ["."] = 'rw' } },
  { user = 'roAuthPrivUser', auth_mode = "md5", auth_phrase = "roAuthPrivUser", encrypt_mode = "aes", encrypt_phrase = "roAuthPrivUser", views = { ["."] = 'ro' } },
  { user = 'rwAuthPrivUser', auth_mode = "md5", auth_phrase = "rwAuthPrivUser", encrypt_mode = "aes", encrypt_phrase = "rwAuthPrivUser", views = { ["."] = 'rw' } },
}

mib_modules = {
    [1]= { ["oid"] ="1.3.6.1.6.3.1.1.4" , ["m"] = 'snmptrap' },
    [2]= { ["oid"] ="1.3.6.1.4.1.99999.2.1" , ["m"] = 'njet_sysinfo' },
    [3]= { ["oid"] ="1.3.6.1.4.1.99999.2.2" , ["m"] = 'njet_conninfo' },
    [4]= { ["oid"] ="1.3.6.1.4.1.99999.2.3" , ["m"] = 'njet_serverinfo' },
}


```

指标输出及mib

指标：

| 对象名                 | 类型                         | OID                     | 描述                                                         |
| ---------------------- | ---------------------------- | ----------------------- | ------------------------------------------------------------ |
| njetVersion            | OCTET STRING (SIZE(0..128))  | 1.3.6.1.4.1.99999.2.1.1 | NJet's version                                               |
| njetUptime             | OCTET STRING (SIZE(0..1024)) | 1.3.6.1.4.1.99999.2.1.2 | NJet's uptime                                                |
| njetHostname           | OCTET STRING (SIZE(0..512))  | 1.3.6.1.4.1.99999.2.1.3 | Host name                                                    |
| njetConnActive         | Gauge32                      | 1.3.6.1.4.1.99999.2.2.1 | Number of Open Connections                                   |
| njetConnReading        | Gauge32                      | 1.3.6.1.4.1.99999.2.2.2 | Number of Connections that NJet reads request headers        |
| njetConnWriting        | Gauge32                      | 1.3.6.1.4.1.99999.2.2.3 | Number of Connections that NJet reads request bodies, processes requests, or writes responses to a client |
| njetConnWaiting        | Gauge32                      | 1.3.6.1.4.1.99999.2.2.4 | Number of Keep-Alive connections                             |
| njetConnAccepted       | Counter32                    | 1.3.6.1.4.1.99999.2.2.5 | All accepted connections                                     |
| njetConnHandled        | Counter32                    | 1.3.6.1.4.1.99999.2.2.6 | All handled connections                                      |
| njetConnRequest        | Counter32                    | 1.3.6.1.4.1.99999.2.2.7 | Total number of handled requests                             |
| njetServerRequestCount | Counter32                    | 1.3.6.1.4.1.99999.2.3.1 | Request Counter of NJet                                      |
| njetServerInBytes      | Counter32                    | 1.3.6.1.4.1.99999.2.3.2 | In Bytes  Counter of NJet                                    |
| njetServerOutBytes     | Counter32                    | 1.3.6.1.4.1.99999.2.3.3 | Out Bytes  Counter of NJet                                   |
| njetServerRespCount1xx | Counter32                    | 1.3.6.1.4.1.99999.2.3.4 | Response 1xx Counter of NJet                                 |
| njetServerRespCount2xx | Counter32                    | 1.3.6.1.4.1.99999.2.3.5 | Response 2xx Counter of NJet                                 |
| njetServerRespCount3xx | Counter32                    | 1.3.6.1.4.1.99999.2.3.6 | Response 3xx Counter of NJet                                 |
| njetServerRespCount4xx | Counter32                    | 1.3.6.1.4.1.99999.2.3.7 | Response 4xx Counter of NJet                                 |
| njetServerRespCount5xx | Counter32                    | 1.3.6.1.4.1.99999.2.3.8 | Response 5xx Counter of NJet                                 |

MIB 定义

```
YunKe-ADC-MIB DEFINITIONS ::= BEGIN
IMPORTS
    OBJECT-TYPE, NOTIFICATION-TYPE, MODULE-IDENTITY,
    Integer32, enterprises, Counter32, Counter64, Gauge32
        FROM SNMPv2-SMI

    TEXTUAL-CONVENTION, TimeInterval, DisplayString
        FROM SNMPv2-TC;

YunKe-ADC MODULE-IDENTITY
    LAST-UPDATED "201711241500Z"
    ORGANIZATION "ADC"
    CONTACT-INFO ""
    DESCRIPTION  "YunKe-ADC mib."

        REVISION "202311160000Z"
        DESCRIPTION
                "Add NJet"
 ::= { enterprises 99999 }

-- ==========================================
-- Set TMLake-NJet mibs
-- ==========================================

NJet        OBJECT IDENTIFIER ::= { YunKe-ADC 2 }

--Group Definition
njetSystemInfo        OBJECT IDENTIFIER ::= {NJet 1 }

njetConnInfo        OBJECT IDENTIFIER ::= {NJet 2 }

njetServerInfo        OBJECT IDENTIFIER ::= {NJet 3 }

--=======================================
-- njetSystemInfo
--=======================================
njetVersion OBJECT-TYPE
        SYNTAX                OCTET STRING (SIZE(0..64))
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "The version of NJet"
        ::= {njetSystemInfo 1 }

njetUptime OBJECT-TYPE
        SYNTAX                OCTET STRING (SIZE(0..1024))
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "The up time of NJet"
        ::= {njetSystemInfo 2 }

njetHostname OBJECT-TYPE
        SYNTAX                OCTET STRING (SIZE(0..512))
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "The hostname of system"
        ::= {njetSystemInfo 3 }

--=======================================
-- njetConnInfo
--=======================================
njetConnActive OBJECT-TYPE
        SYNTAX                Gauge32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Number of Open Connection"
        ::= {njetConnInfo 1 }

njetConnReading OBJECT-TYPE
        SYNTAX                Gauge32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Number of Connections that NJet reads request headers"
        ::= {njetConnInfo 2 }

njetConnWriting OBJECT-TYPE
        SYNTAX                Gauge32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Number of Connections that NJet reads request bodies, processes requests, or writes responses to a client"
        ::= {njetConnInfo 3 }

njetConnWaiting OBJECT-TYPE
        SYNTAX                Gauge32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Number of Keep-Alive connections"
        ::= {njetConnInfo 4 }

njetConnAccepted OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "All accepted connections"
        ::= {njetConnInfo 5 }

njetConnHandled OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "All handled connections"
        ::= {njetConnInfo 6 }

njetConnRequest OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Total number of handled requests"
        ::= {njetConnInfo 7 }

--=======================================
-- njetServerInfo
--=======================================

njetServerRequestCount OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Request Counter"
        ::= {njetServerInfo 1 }

njetServerInBytes OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Server InBytes"
        ::= {njetServerInfo 2 }

njetServerOutBytes OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "Server OutBytes"
        ::= {njetServerInfo 3 }

njetServerRespCount1xx OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "1xx response Count"
        ::= {njetServerInfo 4 }

njetServerRespCount2xx OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "2xx response Count"
        ::= {njetServerInfo 5 }

njetServerRespCount3xx OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "3xx response Count"
        ::= {njetServerInfo 6 }

njetServerRespCount4xx OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "4xx response Count"
        ::= {njetServerInfo 7 }

njetServerRespCount5xx OBJECT-TYPE
        SYNTAX                Counter32
        MAX-ACCESS        read-only
        STATUS                current
        DESCRIPTION        "5xx response Count"
        ::= {njetServerInfo 8 }

END
```

### 3.36.3 调用样例

#### 3.36.3.1 GET 方式查询指标

使用 snmpwalk 命令进行查询

```Bash
snmpwalk -v 2c -c public localhost:2000 1.3.6.1.4.1.99999.2.1
```

![896a6c91-5232-455b-9842-9cb715b888f2](https://gitee.com/gebona/picture/raw/master/202312291102174.png)

#### 3.36.3.2Trap  功能测试

使用 snmptrapd 进行trap 报文接收的测试

创建 snmptrapd.conf 配置文件，输入内容：`authCommunity log,execute,net public`

然后启动snmptrapd ：

```
 sudo snmptrapd -C -c ./snmptrapd.conf  -df -Lo
```

设置上报的时间间隔，目前用 1.3.6.1.6.3.1.1.4.42 这个oid 表示。使用 snmpset 命令进行设置，如设置时间间隔为2秒：

```
snmpset -v2c -c private ``localhost:2000`` .1.3.6.1.6.3.1.1.4.42.0 t 200
```

设置的时间间隔 大于0 时，将触发trap 方式的指标上报，之前启动的snmptrapd 是直接打印接受的报文到控制台， 因此控制台中将显示 njet 上报的信息。

![13096f1b-e156-42f2-9fce-71a846a9ce04](https://gitee.com/gebona/picture/raw/master/202312291103567.png)

## 3.37 动态VS

### 3.37.1 功能描述

在现有监听的port的基础上，支持对server模块的动态添加、删除，可以对支持在server中添加的指令进行便捷的配置。

还可以在动态server基础上添加动态location实现location级别指令功能的动态添加及使用。

### 3.37.2配置说明

#### 3.37.2.1 njet.conf （数据面配置）

```
helper broker modules/njt_helper_broker_module.so conf/mqtt.conf;
helper ctrl modules/njt_helper_ctrl_module.so conf/ctrl.conf;
load_module modules/njt_http_split_clients_2_module.so;
load_module modules/njt_agent_dynlog_module.so;
load_module modules/njt_http_dyn_bwlist_module.so;
load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;
load_module modules/njt_http_dyn_server_module.so;  #配置动态VS 模块
load_module modules/njt_http_location_module.so;  #location验证

user  root root;

cluster_name helper;
node_name node-u01;

error_log  logs/error.log info;
pid        logs/njet.pid;

events {
    worker_connections  1024;
}


http {
    dyn_kv_conf conf/iot-work.conf;
    include       mime.types;
    default_type  application/octet-stream;

    access_log  logs/access.log;

    vhost_traffic_status_zone;
    vhost_traffic_status_filter_by_set_key $request_uri "$realip_remote_addr to $server_name";
    variables_hash_max_size  2048;

    sendfile        on;
    keepalive_timeout  65;
    
    upstream backend1 {
    
         zone backend1_zone 128k;
         server 127.0.0.1:5800;
         
    }

   server {
        listen 5555;
        server_name test-server;
        
        location / {
          alias html;
        }
   }

   server {
        
        listen 443 ssl;
        server_name dev.test.com;
        
        ssl_reject_handshake off;
        ssl_ntls     off;
      
        ssl_certificate       certs/rsa.dev.test.com.crt.pem;
        ssl_certificate_key    certs/rsa.dev.test.com.key.pem;

        ssl_ciphers     RSA+AES128:RSA+AES256:RSA+3DES:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:EECDH+AES256:EECDH+3DES:!MD5;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_prefer_server_ciphers  on;

        location / {
            charset utf-8;
            default_type text/html;
            return 200 "dev.test.com 443 test ok";
        }
        
    }


}
```

#### 3.37.2.2 ctrl.conf （控制面配置）

```JSON
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_ctrl_config_api_module.so;
load_module modules/njt_http_upstream_api_module.so;
load_module modules/njt_http_location_api_module.so;
load_module modules/njt_doc_module.so;
load_module modules/njt_http_vtsd_module.so;
load_module modules/njt_http_dyn_server_api_module.so; #配置动态VS api模块
load_module modules/njt_http_lua_module.so;

events {
    worker_connections  1024;
}
error_log         logs/error_ctrl.log debug;

http {
    dyn_sendmsg_conf  conf/iot-ctrl.conf;
    access_log        logs/access_ctrl.log combined;

    include           mime.types;

    server {
        listen       8081;
        keepalive_timeout 0;
        
        location /hc {
            health_check_api;
        }

        location /api {
             api write=on;
        }

        location /kv {
            dyn_sendmsg_kv;
        }

        location /config {
            config_api;
        }

        location /doc {
            doc_api;
        }
        location /dyn_loc {
           dyn_location_api;
        }
        location /dyn_srv {                                                            
           dyn_server_api;    #配置动态VS api指令                                        
        }                                                                              

        location /metrics {
            vhost_traffic_status_display;
            vhost_traffic_status_display_format html;
        }
  }

}


cluster_name helper;
node_name node1;
```

### 3.37.3 API说明

新增VS API

| 配置项        | 必填 | 配置说明                                                     |
| ------------- | ---- | ------------------------------------------------------------ |
| type          | 是   | “add”  添加 VS                                               |
| addr_port     | 是   | 添加的主机的，port 端口。  例如："192.168.40.203：8000"， 或 “0.0.0.0:8000” |
| listen_option | 否   | 监听的参数。 例如:ssl,proxy_protocol。ssl会根据监听的端口进行自适应（例如 静态文件中listen 443 ssl;添加443的VS时，该server会自动加上ssl字段。） |
| server_name   | 是   | 主机的server_name, 例如："cluster.tmlake.com"                |
| server_body   | 是   | server_body  server 块内的指令集，每条指令用分号分隔。server_body内容可以为空。 |

删除VS API

| 配置项      | 必填 | 配置说明                                                     |
| ----------- | ---- | ------------------------------------------------------------ |
| type        | 是   | “del”  删除VS                                                |
| addr_port   | 是   | 添加的主机的，port 端口。  例如："192.168.40.203：8000"， 或 “0.0.0.0:8000” |
| server_name | 是   | 主机的server_name, 例如："cluster.tmlake.com"                |

### 3.37.4 调用样例

#### 3.37.4.1 新增VS

通过POST方法新增VS

```
curl -v -X POST http://127.0.0.1:8081/dyn_srv -d '{
        "type": "add",
        "addr_port": "0.0.0.0:5555",
        "server_name": "test.server.com",
        "server_body": "return 200 \"test ok\";"
}'
```

返回值

```
* Trying 127.0.0.1:8081...
* Connected to 127.0.0.1 (127.0.0.1) port 8081 (#0)
> POST /dyn_srv HTTP/1.1
> Host: 127.0.0.1:8081
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> Content-Length: 154
> Content-Type: application/x-www-form-urlencoded
> 
< HTTP/1.1 200 OK
< Server: njet/1.2.3
< Date: Thu, 14 Dec 2023 08:50:42 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

#### 3.37.4.2 新增ssl的VS

#### 3.37.4.3 删除VS

通过PUT方法删除VS

```HTTP
curl -v -X PUT http://127.0.0.1:8081/dyn_srv -d '{
        "type": "del",
        "addr_port": "0.0.0.0:5555",
        "server_name": "test.server.com"
}'
```

返回值

```
* Trying 127.0.0.1:8081...
* Connected to 127.0.0.1 (127.0.0.1) port 8081 (#0)
> PUT /dyn_srv HTTP/1.1
> Host: 127.0.0.1:8081
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> Content-Length: 104
> Content-Type: application/x-www-form-urlencoded
> 
< HTTP/1.1 200 OK
< Server: njet/1.2.3
< Date: Thu, 14 Dec 2023 09:48:19 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

#### 3.37.4.4 在动态VS上添加动态location

通过POST方法添加VS

```
curl -v -X POST http://127.0.0.1:8081/dyn_srv -d '{
        "type": "add",
        "addr_port": "0.0.0.0:5555",
        "server_name": "test.server.com",
        "server_body": "return 200 \"test ok\";"
}'
```

返回值

```
* Trying 127.0.0.1:8081...
* Connected to 127.0.0.1 (127.0.0.1) port 8081 (#0)
> PUT /dyn_srv HTTP/1.1
> Host: 127.0.0.1:8081
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> Content-Length: 104
> Content-Type: application/x-www-form-urlencoded
> 
< HTTP/1.1 200 OK
< Server: njet/1.2.3
< Date: Thu, 14 Dec 2023 09:48:19 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

再通过POST方法添加动态location

```
curl -v -X POST http://127.0.0.1:8081/dyn_loc -d '{
  "type": "add",
  "addr_port": "0.0.0.0:5555",
  "server_name": "test.server.com",
  "locations": [
    {
      "location_rule": "",
      "location_name": "/",
      "location_body": "",
      "proxy_pass": "http://backend1"
    }
  ]
}'
```

返回值

```HTTP
* Trying 127.0.0.1:8081...
* Connected to 127.0.0.1 (127.0.0.1) port 8081 (#0)
> POST /dyn_loc HTTP/1.1
> Host: 127.0.0.1:8081
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> Content-Length: 240
> Content-Type: application/x-www-form-urlencoded
> 
< HTTP/1.1 200 OK
< Server: njet/1.2.3
< Date: Fri, 15 Dec 2023 06:41:53 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

#### 3.37.4.5 在动态VS中配置RSA证书

通过POST方法添加VS

```
curl -v -X POST http://127.0.0.1:8081/dyn_srv -d '{
        "type": "add",
        "addr_port": "0.0.0.0:443",
        "listen_option": "ssl",
        "server_name": "dev.test.com",
        "server_body": "ssl_certificate  certs/rsa.dev.test.com.crt.pem;
                        ssl_certificate_key  certs/rsa.dev.test.com.key.pem;
         
                        ssl_ciphers     RSA+AES128:RSA+AES256:RSA+3DES:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:EECDH+AES256:EECDH+3DES:!MD5;
                        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
                        ssl_prefer_server_ciphers  on;
                        return 200 rsa;"
}'
```

返回值

```
* Trying 127.0.0.1:8081...
* Connected to 127.0.0.1 (127.0.0.1) port 8081 (#0)
> POST /dyn_srv HTTP/1.1
> Host: 127.0.0.1:8081
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> Content-Length: 489
> Content-Type: application/x-www-form-urlencoded
> 
< HTTP/1.1 200 OK
< Server: njet/1.2.3
< Date: Mon, 18 Dec 2023 06:33:14 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

#### 3.37.4.6 在动态VS中添加国密证书

通过POST方法添加VS

```
curl -v -X POST http://127.0.0.1:8081/dyn_srv -d '{
        "type": "add",
        "addr_port": "0.0.0.0:443",
        "listen_option": "ssl",
        "server_name": "dev.test.com",
        "server_body": "ssl_certificate  certs/sm2.dev.test.com.enc.crt.pem certs/sm2.dev.test.com.sig.crt.pem;
                        ssl_certificate_key  certs/sm2.dev.test.com.enc.key.pem certs/sm2.dev.test.com.sig.key.pem;
                        ssl_ntls on;
                        ssl_ciphers     RSA+AES128:RSA+AES256:RSA+3DES:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:EECDH+AES256:EECDH+3DES:!MD5;
                        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
                        ssl_prefer_server_ciphers  on;
                        return 200 guomi;"
}'
```

返回值

```
* Trying 127.0.0.1:8081...
* Connected to 127.0.0.1 (127.0.0.1) port 8081 (#0)
> POST /dyn_srv HTTP/1.1
> Host: 127.0.0.1:8081
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> Content-Length: 489
> Content-Type: application/x-www-form-urlencoded
> 
< HTTP/1.1 200 OK
< Server: njet/1.2.3
< Date: Mon, 18 Dec 2023 06:33:14 GMT
< Content-Type: application/json
< Content-Length: 27
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
{"code":0,"msg":"success."}
```

## 3.38 cache 应用加速

### 3.38.1 功能描述

通常为了节省带宽、以及能够快速获取资源，在中间代理服务器上，通常会配置缓存。缓存机制的基本原理是将 Web 资源（如 HTML、CSS、JavaScript、图像等）保存在客户端或中间代理服务器上，以便在后续请求中直接使用该缓存副本，而不必重新获取资源。当客户端或代理服务器收到对资源的请求时，它们首先检查缓存，如果存在有效的缓存副本，就可以直接返回缓存的副本，从而避免了请求的发送和服务器端的处理过程。

但是上述的缓存机制仍然存在一定的问题，就是第一次访问资源是没有缓存的，所以肯定都是要跟服务器通信，然后去下载资源，如果带宽有限而且资源很大的情况（比如视频文件等），客户端就会长时间处于下载阶段，效率低下。针对这种情况，我们需要实现cache应用加速功能。

ache应用加速，由web管理员提前通过下发配置到代理服务器，由代理服务器提前下载资源并进行缓存，这样当客户端首次访问的时候也能够直接从缓存中获取资源，避免等待。

### 3.38.2 配置说明

njet.conf

```
http {
        dyn_kv_conf conf/iot-work.conf;
        include mime.types;
        include conf.d/*.conf;
        proxy_cache_path  /home/njet/test_cache/cache  levels=1:2 keys_zone=cache_quick:10m purger=on max_size=20g inactive=1d;
        map $request_method  $purge_method{
          PURGE 1;
          default 0;
       }
        variables_hash_max_size 2048;
 server {
            listen       80;

            proxy_cache_valid any 1d;
            expires      1d;

       }

server {
            listen       443 ssl;

            proxy_cache_valid any 1h;
            ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
            ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
            ssl_certificate     /etc/vsftpd/vsftpd.pem;
            ssl_certificate_key     /etc/vsftpd/vsftpd.pem;
       }
}
```

ctrl.conf

```
http {
        dyn_sendmsg_conf conf/iot-ctrl.conf;
        access_log logs/access_ctrl.log combined;
        include mime.types;

        server {
                listen 8081;
                location /kv {
                        dyn_sendmsg_kv;
                }
                location /config {
                        config_api;
                }
                location /hc {
                        health_check_api;
                }
                location /api {
                        api write=on;
                }
                location /doc {
                        doc_api;
                }
                location /dyn_loc {
                        dyn_location_api;
                }
                location /metrics {
                        vhost_traffic_status_display;
                        vhost_traffic_status_display_format html;
                }
              location /cache {
                    cache_quick_api;
          }
        }
}
```

在157 服务器上配置后端server:

njet.conf

```
http {
        dyn_kv_conf conf/iot-work.conf;
        include mime.types;
        include conf.d/*.conf;

        server {


        listen       8001;


        location /aa.log {
           root   /home/limin/test_cache/download;
           add_header Content-Disposition: "attachment";
           add_header Content-Type application/octet-stream;
           autoindex off; # 开启目录浏览功能
           autoindex_exact_size off; # 关闭详细文件大小统计，让文件大小显示MB，GB单位，默认为b
           autoindex_localtime on; # 开启以服务器本地时区显示文件修改日期

        }
    }

                server {


        listen       8003;


        location /cc.log {
           root   /home/limin/test_cache/download;
           add_header Content-Disposition: "attachment";
           add_header Content-Type application/octet-stream;
           autoindex off; # 开启目录浏览功能
           autoindex_exact_size off; # 关闭详细文件大小统计，让文件大小显示MB，GB单位，默认为b
           autoindex_localtime on; # 开启以服务器本地时区显示文件修改日期

        }
    }

        server {


             listen       8002 ssl;
           #  server_name  localhost;

            ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
            ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
            ssl_certificate     /etc/vsftpd/.sslkey/vsftpd.pem;
            ssl_certificate_key     /etc/vsftpd/.sslkey/vsftpd.pem;
        location /bb.log {
           root   /home/limin/test_cache/download;

           add_header Content-Disposition: "attachment";
           add_header Content-Type application/octet-stream;
           autoindex off; # 开启目录浏览功能
           autoindex_exact_size off; # 关闭详细文件大小统计，让文件大小显示MB，GB单位，默认为b
           autoindex_localtime on; # 开启以服务器本地时区显示文件修改日期

        }
         location /aa.log {
           root   /home/limin/test_cache/download;

           add_header Content-Disposition: "attachment";
           add_header Content-Type application/octet-stream;
           autoindex off; # 开启目录浏览功能
           autoindex_exact_size off; # 关闭详细文件大小统计，让文件大小显示MB，GB单位，默认为b
           autoindex_localtime on; # 开启以服务器本地时区显示文件修改日期

        }
   }
}                                  
```

### 3.38.3 调用样例

#### 3.38.3.1 添加cache 配置

```
curl -X 'PUT' \
  'http://192.168.40.158:8081/cache' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "type": "add",
  "location_name": "/aa.log",
  "backend_server":"https://192.168.40.157:8002"    
}'
```

#### 3.38.3.2 删除cache 配置

```
curl -X 'PUT' \
  'http://192.168.40.158:8081/cache' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "type": "del",
  "location_name": "/aa.log",
  "backend_server":"https://192.168.40.157:8002"    
}'
```

#### 3.38.3.3 查询cache 配置查询cache 配置

```
curl -X 'GET' 'http://192.168.40.158:8081/cache' |jq
```

## 3.39 ModSecurity

### 3.39.1 功能描述

ModSecurity是一个入侵侦测与防护引擎，它主要是用于Web 应用程序，所以也被称为Web应用程序防火墙（WAF）。 ModSecurity的功能是增强Web application的安全性和保护Web application以避免遭受来自已知与未知的攻击。

### 3.39.2 配置指令说明

| **syntax****:** *modsecurity on \| off* |
| --------------------------------------- |
| **context:** *http, server, location*   |
| **default:** *off*                      |
| **description:** 开启或关闭 Modsecurity |

| **syntax****:** *modsecurity_rules_file <path to rules file>* |
| ------------------------------------------------------------ |
| **context:** *http, server, location*                        |
| **default:** *no*                                            |
| **description:** 指定modsecurity 配置文件                    |

| **syntax:**  *modsecurity_rules_remote <key> <**URL* *to rules>* |
| ------------------------------------------------------------ |
| **context:** *http, server, location*                        |
| **default:** no                                              |
| **description:** 使用远程的rule规则                          |

| **syntax****:**  *modsecurity_rules <modsecurity rule>*    |
| ---------------------------------------------------------- |
| **context:** *http, server, location*                      |
| **default:** no                                            |
| **description:** 直接在njet 配置文件中配置modsecurity 规则 |

| **syntax****:**  *modsecurity_transaction_id string*         |
| ------------------------------------------------------------ |
| **context:** *http, server, location*                        |
| **default:** no                                              |
| **description:** 通过njet设置会话id , 而不是在library 库中生成会话 id |

### 3.39.3 配置文件说明

RPM 包安装后，conf 下的modsec目录附带了一些基本的规则文件，目录结构如下：

conf

└── modsec

​    ├── crs3

​    │  ├── crs-setup.conf               （owasp 规则集基础配置）

​    │  └── rules                            （owasp 规则文件，从owasp官方下载）

​    │     └── *.conf                         

​    ├── custom_rules.conf               （自定义规则）

​    ├── main.conf                           （主配置文件）

​    └── modsecurity.conf                 （modsecurity 基础配置）

核心配置项：

| 配置文件         | 配置项        | 说明                                                         |
| ---------------- | ------------- | ------------------------------------------------------------ |
| modsecurity.conf | SecRuleEngine | On  规则引擎开启Off  不处理规则DetectionOnly   处理规则但从不执行任何破坏性操作（阻止，拒绝，删除，允许，代理和重定向） |

OWASP 规则集

RPM包中自带的OWASP 规则集为目前的稳定版本v3.3.5，OWASP 规则集官网下载地址：

https://github.com/coreruleset/coreruleset

Paranoia Level

conf/modsec/crs3/crs-setup.conf 文件中有一个 id:90000 的规则，设置了 paranoid level （1 - 4）, paranoia 级别越高，规则越偏执。默认值为偏执级别 1，其中规则较为理智，很少出现误报。

运行测试用例时，某些用例的 paranoid level 需要配置为2，3,或者4 。可以根据需要调整这个 paranoid level 的配置。

官网说明文档如下：

https://coreruleset.org/docs/concepts/paranoia_levels/

### 3.39.4 调用样例

#### 3.39.4.1 在njet配置文件中开启的ModSecurity

njet.conf配置

```
load_module modules/njt_http_modsecurity_module.so;

http {
   ...
   server {
        modsecurity on;
        modsecurity_rules_file modsec/main.conf; #指定modsecurity的配置文件
        ...
   }
}
```

测试效果：

通过curl 命令，在header, args 或body中构造可能用于攻击的报文，检查服务端是否检测到可疑行为。例如：

```
 curl -H "Referer: www.github.com<script><img><iframe>" localhost:5555/modsec
```

返回值

```
*Trying 127.0.0.1:5555...
* Connected to 127.0.0.1 (127.0.0.1) port 5555 (#0)
> GET /modsec HTTP/1.1
> Host: 127.0.0.1:5555
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> Referer: www.github.com<script><img><iframe>
> 
< HTTP/1.1 403 Forbidden
< Server: njet/2.0.0
< Date: Thu, 28 Dec 2023 06:35:54 GMT
< Content-Type: text/html
< Content-Length: 151
< Connection: keep-alive
< 
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>njet/2.0.0</center>
</body>
</html>
* Connection #0 to host 127.0.0.1 left intact
```

## 3.40 动态ModSecurity

#### 3.40.1.1 功能描述

目前支持对location 动态开启及关闭 modsecurity 检测功能， 通过动态配置框架，使用控制面URL  /config/2/config/http_dyn_mosecurity 进行动态设置。动态配置功能 njt_http_dyn_modsecurity_module.so 做为单独的模块，需要动态配置功能需要加载该模块, 并且njt_http_dyn_modsecurity_module.so  需要在 njt_http_modsecurity_module.so之后加载。

#### 3.40.1.2 配置说明

```
load_module modules/njt_http_modsecurity_module.so;
load_module modules/njt_http_dyn_modsecurity_module.so;

http {
   ...
   server {
        modsecurity on;
        modsecurity_rules_file modsec/main.conf; #指定modsecurity的配置文件
        ...
   }
}
```

#### 3.40.1.3 调用样例

#### 3.40.1.4 通过GET接口获取当前的modsecurity开关

使用curl请求

```
 curl -X GET http://127.0.0.1:8081/config/2/config/http_dyn_modsecurity
```

返回值

```Bash
{
  "servers": [
    {
      "listens": [
        "0.0.0.0:5555"
      ],
      "serverNames": [
        "localhost"
      ],
      "locations": [
        {
          "location": "/",
          "modsecurity": {
            "enable": true
          }
        },
        {
          "location": "/test_modsec",
          "modsecurity": {
            "enable": true
          }
        }
      ]
    }
  ]
}
```

#### 3.40.1.5 通过PUT接口更改modsecurity设置

使用curl请求

```
 curl -X PUT http://127.0.0.1:8081/config/2/config/http_dyn_modsecurity -d'
 {
  "servers": [
    {
      "listens": [
        "0.0.0.0:5555"
      ],
      "serverNames": [
        "localhost"
      ],
      "locations": [
        {
          "location": "/",
          "modsecurity": {
            "enable": false
          }
        },
        {
          "location": "/test_modsec",
          "modsecurity": {
            "enable": false
          }
        }
      ]
    }
  ]
}
```

返回值

```Plain
{"code":0,"msg":"success."}
```

## 3.41 Privilege Agent 进程

### 3.41.1 功能描述

Privilege Agent 进程将以 root 用户运行，并监听 /worker_a/# 主题， 所有动态配置的消息先发往该进程，处理后，由该进程中的 kv 模块将需持久化的消费发往  /dyn/${module} 主题

### 3.41.2 配置说明

```
语法:          privileged_agent flag;
默认值:        on;   
允许配置位置:   main
```

默认将启动 privilege agent 进程，需要关闭该进程时才需要配置。

njet 可执行文件需要设置 cap_setuid, 否则privilege agent 进程会启动失败。

```JSON
sudo /usr/sbin/setcap cap_dac_override,cap_dac_read_search,cap_net_bind_service,cap_net_admin,cap_net_raw,cap_setuid=eip /etc/njet/sbin/njet
```

### 3.41.3 调用样例

Privilege 进程是 root 用户启动。

![a6fc022d-41b1-40f4-bf68-7afd09590c1c](https://gitee.com/gebona/picture/raw/master/202312291120562.png)

## 3.42 Proxy_ssl_alpn

### 3.42.1功能描述

​      stream，http 作为 tls代理，反向代理到后端server 时，某些应用需要在tls 握手时，设置tls 的alpn 字段。

### 3.42.2配置说明

客户端配置，该配置在stream 和 https  模块生效。在发送*Client Hello  时，携带。*

|                | 必填 | 配置说明                                                     |
| -------------- | ---- | ------------------------------------------------------------ |
| proxy_ssl_alpn | 是   | 设置客户端发送的 alpn 类型。多个以空格分隔。 例如：proxy_ssl_alpn   njet   h2  h3 stream;  设置 4 个字段。http，stream 用法一样。结合ssl_preread on 指令。 通过变量 $ssl_preread_alpn_protocols, 选择不同的proxy_pass 路径。location， server |

| Syntax:  | proxy_ssl_alpn protocol ...;  |
| -------- | ----------------------------- |
| Default: | —                             |
| Context: | stream,http, server, location |

This directive appeared in version 1.7.8.

#### 3.42.2.1 njet.conf（数据面配置）

```
http {
    dyn_kv_conf conf/iot-work.conf;
    include       mime.types;
    default_type  application/octet-stream;

    access_log  logs/access.log;

    sendfile        on;
    keepalive_timeout  65;

     server {
        listen 5555;
        location = /test1 {
          proxy_ssl_certificate      certs/client.pem;
          proxy_ssl_certificate_key  certs/client.key;
          proxy_ssl_alpn  test1;
          keepalive_timeout 0;
          proxy_pass https://127.0.0.1:5443/;  
        }
        location = /test2 {
          proxy_ssl_certificate     certs/client.pem;
          proxy_ssl_certificate_key certs/client.key;
          proxy_ssl_alpn  test2;
          keepalive_timeout 0;
          proxy_pass https://127.0.0.1:5443/; 
        }
        location = /default {
          proxy_ssl_certificate     certs/client.pem;
          proxy_ssl_certificate_key certs/client.key;
          keepalive_timeout 0;
          proxy_pass https://127.0.0.1:5443/; 
        }


    }


}
stream {

      map $ssl_preread_alpn_protocols $ssl_server {
         default 127.0.0.1:5902;
         test1  127.0.0.1:5900;
         test2 127.0.0.1:5901;
    }


      server {
          listen  7777;
          proxy_ssl on;
          proxy_ssl_certificate     certs/client.pem;
          proxy_ssl_certificate_key certs/client.key;
          proxy_ssl_alpn  test1;   
          proxy_pass  127.0.0.1:5443;  
        }
      server {
          listen  8888;
          proxy_ssl on;
          proxy_ssl_certificate     certs/client.pem;
          proxy_ssl_certificate_key certs/client.key;
          proxy_ssl_alpn  test2;    
          proxy_pass  127.0.0.1:5443;
        }
      server {
          listen  9999;
          proxy_ssl on;
          proxy_ssl_certificate     certs/client.pem;
          proxy_ssl_certificate_key certs/client.key;
          proxy_pass  127.0.0.1:5443;
        }
      server {
          listen  5900 ssl;
          ssl_certificate     certs/server.pem;
          ssl_certificate_key certs/server.key;
          return  "5900 test1 ok";
        }
      server {
          listen  5901 ssl;
          ssl_certificate     certs/server.pem;
          ssl_certificate_key certs/server.key;
          return  "5901 njet ok";
        }
     server {
          listen  5902 ssl;
          ssl_certificate     certs/server.pem;
          ssl_certificate_key certs/server.key;
          return  "5902 default ok";
        }
     server {
         listen 5443;
         ssl_preread on;
         proxy_pass $ssl_server;
     }
}
```

### 3.42.3 调用样例

#### 3.42.3.1 访问http模块下的uri

访问/test1

```
curl -v --http0.9  http://127.0.0.1:5555/test1
*   Trying 127.0.0.1:5555...
* Connected to 127.0.0.1 (127.0.0.1) port 5555 (#0)
> GET /test1 HTTP/1.1
> Host: 127.0.0.1:5555
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> 
* Closing connection 0
5900 test1 ok
```

访问/test2

```
curl -v --http0.9  http://127.0.0.1:5555/test2
*   Trying 127.0.0.1:5555...
* Connected to 127.0.0.1 (127.0.0.1) port 5555 (#0)
> GET /test2 HTTP/1.1
> Host: 127.0.0.1:5555
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> 
* Closing connection 0
5901 test2 ok
```

访问/default

```
curl -v --http0.9  http://127.0.0.1:5555/default
*   Trying 127.0.0.1:5555...
* Connected to 127.0.0.1 (127.0.0.1) port 5555 (#0)
> GET /default HTTP/1.1
> Host: 127.0.0.1:5555
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> 
* Closing connection 0
5902 default ok
```

#### 3.42.3.2 访问stream模块下的uri

访问7777

```
curl -v --http0.9 http://127.0.0.1:7777/
*   Trying 127.0.0.1:7777...
* Connected to 127.0.0.1 (127.0.0.1) port 7777 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:7777
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> 
* Recv failure: Connection reset by peer
* Closing connection 0
5900 test1 ok
```

访问8888

```
curl -v --http0.9 http://127.0.0.1:8888/
*   Trying 127.0.0.1:8888...
* Connected to 127.0.0.1 (127.0.0.1) port 8888 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:8888
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> 
* Recv failure: Connection reset by peer
* Closing connection 0
5901 test2 ok
```

访问9999

```
curl -v --http0.9 http://127.0.0.1:9999/
*   Trying 127.0.0.1:9999...
* Connected to 127.0.0.1 (127.0.0.1) port 9999 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:9999
> User-Agent: curl/8.1.0-DEV
> Accept: */*
> 
* Recv failure: Connection reset by peer
* Closing connection 0
5902 default ok
```



# 4.标准配置文件

## 4.1 **配置前目录准备**

1. 在任意目录下创建目录，用作存储njet项目
2. 进入当前创建的目录并创建 modules 、conf 、 data 、sbin 4个目录
3. 将njet的zip包上传至njet项目目录，并解压
4. 将解压后的ojbs目录中的所有so文件复制到modules; 将objs目录中的njet文件复制到sbin目录
5. 将本目录4.2-4.7的文件创建并保存到conf目录
6. 进入sbin目录，使用 ./njet -p 项目目录 -c 项目目录/conf/njet.conf 启动项目

## 4.2 **njet.conf** **（数据面）**

```
Bash
worker_processes auto;

cluster_name njet;
node_name node1;

error_log logs/error.log error;

helper ctrl modules/njt_helper_ctrl_module.so conf/njet_ctrl.conf;
helper broker modules/njt_helper_broker_module.so;

load_module modules/njt_http_split_clients_2_module.so;  
load_module modules/njt_agent_dynlog_module.so;  
load_module modules/njt_http_dyn_bwlist_module.so; 
load_module modules/njt_dyn_ssl_module.so;
load_module modules/njt_http_vtsc_module.so;
load_module modules/njt_http_location_module.so;
#load_module modules/njt_http_lua_module.so;

events {
    worker_connections  1024;
}

http {
    include mime.types;
    access_log off;
    vhost_traffic_status_zone;
    server {
        listen       8080;
        location / {
           root html;
        }
    }
}
```



## 4.3 **njet_ctrl.conf** **（控制面）**

```
load_module modules/njt_http_sendmsg_module.so;
load_module modules/njt_ctrl_config_api_module.so; 
load_module modules/njt_helper_health_check_module.so;
load_module modules/njt_http_upstream_api_module.so; 
load_module modules/njt_http_location_api_module.so;
load_module modules/njt_doc_module.so;
load_module modules/njt_http_vtsd_module.so;
load_module modules/njt_http_ssl_api_module.so;       

error_log logs/error-ctrl.log info;

events {
    worker_connections  1024;
}

http {
    include mime.types;
    server {
        listen       8081;
        
       
        
        location / {
            return 200 "njet control panel\n";
        }
        location /hc {
            health_check_api;
        }
        
        location /api {
             api write=on;
        }
        
        location /kv {
            dyn_sendmsg_kv;
        }
        
        location /config {
            config_api;
        }
        
        location /doc {
            doc_api;
            limit_except PUT{
               auth_basic "NGINX plus API";
               auth_basic_user_file /etc/nginx/htpasswd;
            }
        }
          location /dyn_loc {
           dyn_location_api;
        }
        location /ssl {
            dyn_ssl_api;  
            limit_except POST {
               auth_basic "NGINX plus API";
               auth_basic_user_file /etc/nginx/htpasswd;
            }       
        }
        location /metrics {
            vhost_traffic_status_display;
            vhost_traffic_status_display_format html;
        }
    }
}
```



## 4.4 **mime.types**

```
Bash

types {
    text/html                                        html htm shtml;
    text/css                                         css;
    text/xml                                         xml;
    image/gif                                        gif;
    image/jpeg                                       jpeg jpg;
    application/javascript                           js;
    application/atom+xml                             atom;
    application/rss+xml                              rss;

    text/mathml                                      mml;
    text/plain                                       txt;
    text/vnd.sun.j2me.app-descriptor                 jad;
    text/vnd.wap.wml                                 wml;
    text/x-component                                 htc;

    image/avif                                       avif;
    image/png                                        png;
    image/svg+xml                                    svg svgz;
    image/tiff                                       tif tiff;
    image/vnd.wap.wbmp                               wbmp;
    image/webp                                       webp;
    image/x-icon                                     ico;
    image/x-jng                                      jng;
    image/x-ms-bmp                                   bmp;

    font/woff                                        woff;
    font/woff2                                       woff2;

    application/java-archive                         jar war ear;
    application/json                                 json;
    application/mac-binhex40                         hqx;
    application/msword                               doc;
    application/pdf                                  pdf;
    application/postscript                           ps eps ai;
    application/rtf                                  rtf;
    application/vnd.apple.mpegurl                    m3u8;
    application/vnd.google-earth.kml+xml             kml;
    application/vnd.google-earth.kmz                 kmz;
    application/vnd.ms-excel                         xls;
    application/vnd.ms-fontobject                    eot;
    application/vnd.ms-powerpoint                    ppt;
    application/vnd.oasis.opendocument.graphics      odg;
    application/vnd.oasis.opendocument.presentation  odp;
    application/vnd.oasis.opendocument.spreadsheet   ods;
    application/vnd.oasis.opendocument.text          odt;
    application/vnd.openxmlformats-officedocument.presentationml.presentation
                                                     pptx;
    application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
                                                     xlsx;
    application/vnd.openxmlformats-officedocument.wordprocessingml.document
                                                     docx;
    application/vnd.wap.wmlc                         wmlc;
    application/wasm                                 wasm;
    application/x-7z-compressed                      7z;
    application/x-cocoa                              cco;
    application/x-java-archive-diff                  jardiff;
    application/x-java-jnlp-file                     jnlp;
    application/x-makeself                           run;
    application/x-perl                               pl pm;
    application/x-pilot                              prc pdb;
    application/x-rar-compressed                     rar;
    application/x-redhat-package-manager             rpm;
    application/x-sea                                sea;
    application/x-shockwave-flash                    swf;
    application/x-stuffit                            sit;
    application/x-tcl                                tcl tk;
    application/x-x509-ca-cert                       der pem crt;
    application/x-xpinstall                          xpi;
    application/xhtml+xml                            xhtml;
    application/xspf+xml                             xspf;
    application/zip                                  zip;

    application/octet-stream                         bin exe dll;
    application/octet-stream                         deb;
    application/octet-stream                         dmg;
    application/octet-stream                         iso img;
    application/octet-stream                         msi msp msm;

    audio/midi                                       mid midi kar;
    audio/mpeg                                       mp3;
    audio/ogg                                        ogg;
    audio/x-m4a                                      m4a;
    audio/x-realaudio                                ra;

    video/3gpp                                       3gpp 3gp;
    video/mp2t                                       ts;
    video/mp4                                        mp4;
    video/mpeg                                       mpeg mpg;
    video/quicktime                                  mov;
    video/webm                                       webm;
    video/x-flv                                      flv;
    video/x-m4v                                      m4v;
    video/x-mng                                      mng;
    video/x-ms-asf                                   asx asf;
    video/x-ms-wmv                                   wmv;
    video/x-msvideo                                  avi;
}
```



## 4.5 **otel.conf**

```
Bash
exporter = "otlp"
processor = "batch"

[exporters.otlp]
#collector server address
# Alternatively the OTEL_EXPORTER_OTLP_ENDPOINT environment variable can also be used.
host = "192.168.40.136"
port = 4317

[processors.batch]
max_queue_size = 2048
schedule_delay_millis = 5000
max_export_batch_size = 512

[service]
# Can also be set by the OTEL_SERVICE_NAME environment variable.
name = "njet-proxy" # Opentelemetry resource name

[sampler]
name = "AlwaysOn" # Also: AlwaysOff, TraceIdRatioBased
ratio = 0.1
parent_based = false
```



## 4.6 **opentelemetry_module.conf**

```
Bash
NjetModuleEnabled OFF;
NjetModuleOtelSpanExporter otlp;
NjetModuleOtelExporterEndpoint 192.168.40.136:4317;  #改成自己的ip，端口无需更改
NjetModuleServiceName DemoService;
NjetModuleServiceNamespace DemoServiceNamespace;
NjetModuleServiceInstanceId DemoInstanceId;
NjetModuleResolveBackends ON;           #固定值，不做修改
NjetModuleTraceAsInfo ON;          
```



## 4.7 **opentelemetry_sdk_log4cxx.xml**

```
log 配置文件 （opentelemetry_sdk_log4cxx.xml）
同其他 NJet 配置文件路径一致
```

```
XML
<?xml version="1.0" encoding="UTF-8" ?>
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/" debug="false">

<appender name="main" class="org.apache.log4j.ConsoleAppender">
 <layout class="org.apache.log4j.PatternLayout">
  <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS z} %-5p %X{pid}[%t] [%c{2}] %m%n" />
  <param name="HeaderPattern" value="Opentelemetry Webserver %X{version} %X{pid}%n" />
 </layout>
</appender>

<appender name="api" class="org.apache.log4j.ConsoleAppender">
 <layout class="org.apache.log4j.PatternLayout">
  <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS z} %-5p %X{pid} [%t] [%c{2}] %m%n" />
  <param name="HeaderPattern" value="Opentelemetry Webserver %X{version} %X{pid}%n" />
 </layout>
</appender>

<appender name="api_user" class="org.apache.log4j.ConsoleAppender">
 <layout class="org.apache.log4j.PatternLayout">
  <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS z} %-5p %X{pid} [%t] [%c{2}] %m%n" />
  <param name="HeaderPattern" value="Opentelemetry Webserver %X{version} %X{pid}%n" />
 </layout>
</appender>

<logger name="api" additivity="false">
    <level value="info"/>
    <appender-ref ref="api"/>
</logger>

<logger name="api_user" additivity="false">
    <level value="info"/>
    <appender-ref ref="api_user"/>
</logger>

<root>
  <priority value="info" />
  <appender-ref ref="main"/>
</root>

</log4j:configuration>
```



## 4.8 **重要：部署所需额外配置**

提测单里除了有流水线出的包  +  还要有如下三个文件

```
1）将njt_otel_module.so、njt_agent_dyn_otel_module.so 放到与其他动态模块同目录，即放置在modules目录下。
2）dependlib.tar 解压后， 启动njet时将该dependlib目录加入到LD_LIBRARY_PATH 环境变量里
eg:    export LD_LIBRARY_PATH=`pwd`/dependlib:$LD_LIBRARY_PATH
vi /etc/profile 
source /etc/profile
```

**[dependlib.tar]**

**[njt_otel_module.so]**

**[njt_agent_dyn_otel_module.so]**



# 5.**常见问题**

a.   reload后，需要等待3秒左右，动态查询接口，才可用。

![image-20230619150243394](https://gitee.com/gebona/picture/raw/master/202306191502054.png)

b.   访问不存在的接口，会报500的错误。

![image-20230619150259118](https://gitee.com/gebona/picture/raw/master/202306191503353.png)

c.   ctrl.conf中自定义了接口location，curl命令可以正常使用，但是doc gui无法使用。

njt_ctrl标准配置文件：需要包含如下location：

/api 提供upstream member维护

/config 提供声明式api

/kv 提供kv维护接口

/dyn_loc 提供动态location配置接口

/hc 主动健康检查接口

/metrics 指标输出接口

/doc/swagger openapi接口

/doc/gui 简单界面维护



# 6.**附录**

**健康检查API说明**

```
openapi: 3.0.3
info:
  title: njet-api
  description: njet-api
  version: 1.0.0
servers:
  - url: 'http://192.168.40.140/api/1'
paths:
  /hc/:
    get:
      tags:
        - Health Check
      summary: Returns a list of health check
      description: Returns a list of health check.
      operationId: getHealthCheckUpstreams
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HealthCheck'
        '500':
          description: 服务器内部错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'

  /hc/{typeName}/{upstreamName}:
    parameters:
      - name: typeName
        type: string
        in: path
        description: 健康检查的类型.
        required: true
        example: HTTP
        enum:
          - HTTP

      - name: upstreamName
        type: string
        in: path
        description: upstream 名称.
        required: true
        example: demo
    get:
      tags:
        - Health Check
      summary: 查询健康检查详细配置
      description: 查询健康检查详细配置
      operationId: getHealthCheckConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HealthCheckConf'
        '404':
          description: upstream 或者 type 未找到
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
        '500':
          description: 服务器内部错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'

    post:
      tags:
        - Health Check
      summary: 创建健康检查
      description: 创建健康检查
      operationId: postHealthCheck
      parameters:
        - in: body
          name: postHealthCheck
          description: 创建新的健康检查
          required: true
          schema:
            $ref: '#/components/schemas/HealthCheckConf'
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
        '400':
          description: 请求body 错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
        '404':
          description: 健康检查类型或upstream 未找到
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
        '409':
          description: 健康检查已经定义过
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
        '500':
          description: 服务器内部错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'

    delete:
      tags:
        - Health Check
      summary: 删除健康检查
      description: 删除健康检查
      operationId: deleteHealthCheck
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
        '404':
          description: 健康检查不存在
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'

        ###DEFINITIONS
components:
  schemas:
    HealthCheckItem:
      type: object
      properties:
        upstream:
          type: string
          description: name of upstream.
        type:
          type: string
          description: type of health check.
    HealthCheck:
      title: healthCheck
      description: health Check 简易信息
      type: array
      items:
        $ref: '#/components/schemas/HealthCheckItem'

    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
        msg:
          type: string

    HealthCheckConf:
      title: healthCheckConf
      description: health Check 完整配置
      type: object
      properties:
        interval:
          type: string
          description: 健康检查间隔
          default: 5s
          example: 5s
        jitter:
          type: string
          description: 健康检查间隔
          default: 0s
          example: 1s
        timeout:
          type: string
          description: 健康检查超时时间
          default: 5s
          example: 5s
        port:
          type: number
          description: 健康检查的端口
          example: 80
          minimum: 1
          maximum: 65535
        passes:
          type: number
          description: 健康检查转变为健康，需要检查连续通过的次数
          minimum: 1
          default: 1
        fails:
          type: number
          description: 健康检查转变为不健康，需要健康检查连续不通过的次数
          minimum: 1
          default: 1
        http:
          type: object
          description: HTTP upstream 健康检查相关配置
          properties:
            grpcService:
              type: string
              description: grpc健康检查请求的 service 名称
              default: demo
            grpcStatus:
              type: number
              default: 13
              description: grpc健康检查标记为通过的状态码
            uri:
              type: string
              description: http 健康检查请请求的uri
              default: /
            header:
              type: array
              description: http 健康检查校验的header
              items:
                type: string
                description:
            body:
              type: string
              description: http 健康检查检查body的内容是否满足表达式
            status:
              type: string
              description: http 健康检查检查返回状态码是否满足
        ssl:
          type: object
          description: ssl客户端相关配置
          properties:
            enable:
              type: boolean
              description: 是否开启https 健康检查
              default: false
            protocols:
              type: string
              description: 指定健康检查使用的ssl协议版本，支持的版本：[SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3]
              default: TLSv1 TLSv1.1 TLSv1.2
            ciphers:
              type: string
              description: 对健康检查请求开启 ciphers，ciphers以openssl 标准格式配置。"openssl ciphers" 指令可以查看ciphers列表
              default: DEFAULT
            name:
              type: string
              description: 设置健康检查请求时携带的Host header
              default: upstream name
            serverName:
              type: boolean
              description: 启用或禁用通过TLS服务器名称指示扩展（SNI，RFC 6066）传递服务器名称。
              default: false
            verify:
              type: boolean
              description: 启用或禁用对连接的HTTPS服务器证书的验证。(需配置trustedCertificate)
              default: false
            verifyDepth:
              type: number
              description: HTTPS服务器证书链中设置验证深度。
              default: 1
              minimum: 1
            trustedCertificate:
              type: string
              description: 指定一个具有PEM格式的受信任CA证书，用于验证指定HTTPS服务器的证书。
            crl:
              type: string
              description: 以PEM格式指定具有撤销证书（CRL）的文件，用于验证代理HTTPS服务器的证书。
            certificate:
              type: string
              description: 指定一个带有PEM格式证书的文件，用于对HTTPS服务器进行身份验证(客户端证书)。
            certificateKey:
              type: string
              description: 指定一个带有PEM格式的密钥的文件，用于对HTTPS服务器进行身份验证(客户端证书key)。
```



## 6.1 动态access log配置API说明

```
openapi: 3.0.3
info:
  title: dyn config api
  description: 动态config 开关
  version: 1.0.0

servers:
  - url: '/config/2'
paths:
  /config/:
    get:
      tags:
        - dyn config module
      summary: Returns a list of allow dynamic conf module
      description: Returns a list of allow dynamic conf module
      operationId: getDynConfModuleList
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModuleList'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


  /config/http_log:
    put:
      tags:
        - dyn log config
      summary: set module conf
      description: set module conf
      operationId: setDynModuleConf
      parameters:
        - in: body
          name: access log conf
          description: access log 配置清单
          required: true
          schema:
            $ref: '#/components/schemas/MainConf'
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
    get:
      tags:
        - dyn log config
      summary: return dynamic module conf
      description: get module conf
      operationId: getDynModuleConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'




        ###DEFINITIONS
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    ModuleList:
      title: module list
      description: 支持动态配置的模块列表
      type: array
      items:
        type: string

    MainConf:
      title: main Conf
      description: 动态access log main 级别配置
      type: object
      properties:
        servers:
          type: array
          items:
            $ref: '#/components/schemas/ServerConf'
        accessLogFormats:
          type: array
          items:
            $ref: '#/components/schemas/FormatConf'

    FormatConf:
      title: FormatConf
      description: 动态access format 配置
      type: object
      properties:
        name:
          type: string
          description: access log format 名称
          example: test
        format:
          type: string
          description: access log 格式字符串
          example: $remote_addr - $remote_user [$time_local] \"$request\" $status $body_bytes_sent \"$http_referer\" \"$http_user_agent\"
        escape:
          type: string
          description:  access log 转义格式
          example: default


    ServerConf:
      title:     ServerConf
      description: 动态access log server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string
        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        locations:
          $ref: '#/components/schemas/LocationLogConf'
    AccessLogConf:
      title:     AccessLogConf
      description: access log 配置
      type: object
      properties:
        path:
          type: string
          description: access log 写入的位置
        formatName:
          type: string
          description: access log 格式名称


    LocationLogConf:
      title: LocationLogConf
      description: location 级别 access log 配置开关信息
      type: object
      properties:
        location:
          type: string
          description: location 名称 .
          example: /demo
        accessLogOn:
          type: boolean
          description: access log 开关状态 .
          example: false
        accessLogs:
          type: array
          items:
            $ref: '#/components/schemas/AccessLogConf'
        locations:
          $ref: '#/components/schemas/LocationLogConf'
```



## 6.2 动态telemetry配置API说明

```
openapi: 3.0.3
info:
  title: dyn config api
  description: 动态config 开关
  version: 1.0.0

servers:
  - url: '/config/2'
paths:
  /config/:
    get:
      tags:
        - dyn config module
      summary: Returns a list of allow dynamic conf module
      description: Returns a list of allow dynamic conf module
      operationId: getDynConfModuleList
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModuleList'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


  /config/otel:
    put:
      tags:
        - dyn telemetry config
      summary: set module conf
      description: set module conf
      operationId: setDynModuleConf
      requestBody:
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
          required: true
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
    get:
      tags:
        - dyn telemetry config
      summary: return dynamic module conf
      description: get module conf
      operationId: getDynModuleConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'




        ###DEFINITIONS
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    ModuleList:
      title: module list
      description: 支持动态配置的模块列表
      type: array
      items:
        type: string

    MainConf:
      title: main Conf
      description: 动态telemetry main 级别配置
      type: object
      properties:
        servers:
          type: array
          items:
            $ref: '#/components/schemas/ServerConf'

    ServerConf:
      title:     ServerConf
      description: 动态telemetry server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string

        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        locations:
          type: array
          description: locations 列表
          items:
            $ref: '#/components/schemas/LocationLogConf'


    LocationLogConf:
      title: LocationLogConf
      description: location 级别 telemetry 配置开关信息
      type: object
      properties:
        location:
          type: string
          description: location 名称 .
          example: /demo
        opentelemetry:
          type: boolean
          description: telemetry 开关状态 .
          example: false
        locations:
          type: array
          description: 嵌套 locations 列表
          items:
            $ref: '#/components/schemas/LocationLogConf'
```



## 6.3 **动态**telemetry - webserver **配置**API说明

```
openapi: 3.0.3
info:
  title: dyn config api
  description: 动态config 开关
  version: 1.0.0

servers:
  - url: '/config/2'
paths:
  /config/:
    get:
      tags:
        - dyn config module
      summary: Returns a list of allow dynamic conf module
      description: Returns a list of allow dynamic conf module
      operationId: getDynConfModuleList
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModuleList'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


  /config/otel_webserver:
    put:
      tags:
        - dyn telemetry webserver config
      summary: set module conf
      description: set module conf
      operationId: setDynModuleConf
      requestBody:
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
          required: true
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
    get:
      tags:
        - dyn telemetry webserver config
      summary: return dynamic module conf
      description: get module conf
      operationId: getDynModuleConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'




        ###DEFINITIONS
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    ModuleList:
      title: module list
      description: 支持动态配置的模块列表
      type: array
      items:
        type: string

    MainConf:
      title: main Conf
      description: 动态telemetry webserver main 级别配置
      type: object
      properties:
        servers:
          type: array
          items:
            $ref: '#/components/schemas/ServerConf'

    ServerConf:
      title:     ServerConf
      description: 动态telemetry webserver server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string

        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        locations:
          type: array
          description: locations 列表
          items:
            $ref: '#/components/schemas/LocationLogConf'


    LocationLogConf:
      title: LocationLogConf
      description: location 级别 telemetry webserver 配置开关信息
      type: object
      properties:
        location:
          type: string
          description: location 名称 .
          example: /demo
        NjetModuleEnabled:
          type: boolean
          description: telemetry webserver 开关状态 .
          example: false
        locations:
          type: array
          description: 嵌套 locations 列表
          items:
            $ref: '#/components/schemas/LocationLogConf'
```



## 6.4 **动态**vts**配置**API**说明**

```
openapi: 3.0.3
info:
  title: dyn config api
  description: 动态config 开关
  version: 1.0.0

servers:
  - url: '/config/1'
paths:
  /config/:
    get:
      tags:
        - dyn config module
      summary: Returns a list of allow dynamic conf module
      description: Returns a list of allow dynamic conf module
      operationId: getDynConfModuleList
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModuleList'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


  /config/http_vts:
    put:
      tags:
        - dyn vts config
      summary: set module conf
      description: set module conf
      operationId: setDynModuleConf
      parameters:
        - in: body
          name: vts conf
          description: vts 配置清单
          required: true
          schema:
            $ref: '#/components/schemas/MainConf'
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
    get:
      tags:
        - dyn vts config
      summary: return dynamic module conf
      description: get module conf
      operationId: getDynModuleConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'




        ###DEFINITIONS
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    ModuleList:
      title: module list
      description: 支持动态配置的模块列表
      type: array
      items:
        type: string

    MainConf:
      title: main Conf
      description: 动态vts main 级别配置
      type: object
      properties:
        vhost_traffic_status_filter_by_set_key:
          type: string
          description: vts filter set .
          example: \"$request_uri\" \"$server_name\"
        servers:
          type: array
          items:
            $ref: '#/components/schemas/ServerConf'

    ServerConf:
      title:     ServerConf
      description: 动态vts server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string

        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        locations:
          $ref: '#/components/schemas/LocationConf'


    LocationConf:
      title: LocationConf
      description: location 级别 vts 配置开关信息
      type: object
      properties:
        location:
          type: string
          description: location 名称 .
          example: /demo
        vhost_traffic_status:
          type: boolean
          description: vts 开关状态 .
          example: false
        locations:
          $ref: '#/components/schemas/LocationConf'
```



## 6.5 **动态**split_clients2配置API说明

```
openapi: 3.0.3
info:
  title: dyn config api
  description: 动态config 开关
  version: 1.0.0

servers:
  - url: '/config/1'
paths:
  /config/:
    get:
      tags:
        - dyn config module
      summary: Returns a list of allow dynamic conf module
      description: Returns a list of allow dynamic conf module
      operationId: getDynConfModuleList
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModuleList'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


  /config/http_split_clients_2:
    put:
      tags:
        - dyn split clients 2 config
      summary: set module conf
      description: set module conf
      operationId: setDynModuleConf
      requestBody:
          content:
            application/json:
              schema:
                  $ref: '#/components/schemas/SplitConf'
          required: true
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
    get:
      tags:
        - dyn split clients 2 config 
      summary: return dynamic module conf
      description: get module conf
      operationId: getDynModuleConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SplitConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    ModuleList:
      title: module list
      description: 支持动态配置的模块列表
      type: array
      items:
        type: string

    SplitConf:
      title: Split Clients 2 Conf
      description: 动态Split Clients 2 配置
      type: object
      properties:
        http:
          type: object 
          properties:
            split_clients_2: 
              type: object 
              properties:
                ${backend_name1}: 
                  type: integer
                  example: 20
                  minimum: 0
                  maximum: 100
                ${backend_name2}: 
                  type: integer
                  example: 80
                  minimum: 0
                  maximum: 100
```



## 6.6 动态黑白名单配置API说明

```
openapi: 3.0.3
info:
  title: dyn config api
  description: 动态config 开关
  version: 1.0.0

servers:
  - url: '/config/1'
paths:
  /config/:
    get:
      tags:
        - dyn config module
      summary: Returns a list of allow dynamic conf module
      description: Returns a list of allow dynamic conf module
      operationId: getDynConfModuleList
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModuleList'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


  /config/http_dyn_bwlist:
    put:
      tags:
        - dyn black white list config
      summary: set module conf
      description: set module conf
      operationId: setDynModuleConf
      requestBody:
          content:
            application/json:
              schema:
                  $ref: '#/components/schemas/MainConf'
          required: true
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
    get:
      tags:
        - dyn black white list config
      summary: return dynamic module conf
      description: get module conf
      operationId: getDynModuleConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    ModuleList:
      title: module list
      description: 支持动态配置的模块列表
      type: array
      items:
        type: string

    MainConf:
      title: main Conf
      description: 动态 main 级别配置
      type: object
      properties:
        servers:
          type: array
          items:
            $ref: '#/components/schemas/ServerConf'

    ServerConf:
      title:     ServerConf
      description: 动态 server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string

        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        locations:
          $ref: '#/components/schemas/LocationConf'


    LocationConf:
      title: LocationConf
      description: location 级别 vts 配置开关信息
      type: object
      properties:
        location:
          type: string
          description: location 名称 .
          example: /demo
        accessIpv4:
          type: object
          description: access 黑白名单
          properties:
            rule: 
              type: string
              enum: [allow, deny]
            addr: 
               type: string
               example: "192.168.1.0"
            mask: 
               type: string
               example: "255.255.255.0"
        locations:
          $ref: '#/components/schemas/LocationConf' 
```



## 6.7 **动态**http server ssl**配置**API**说明**

```
openapi: 3.0.3
info:
  title: dyn server ssl api
  description: 动态server ssl配置
  version: 1.0.0

servers:
  - url: '/'
paths:
  /ssl:
    get:
      tags:
        - dyn http server ssl config
      summary: return http server ssl config
      description: get http server ssl config
      operationId: getDynHttpServerSsl
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'

    put:
      tags:
        - dyn http server ssl config
      summary: set http server ssl conf
      description: set http server ssl conf
      operationId: setDynHttpServerSsl
      requestBody:
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ServerConf'
          required: true
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


        ###DEFINITIONS
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    MainConf:
      title: main Conf
      description: 动态http server ssl main 级别配置
      type: object
      properties:
        servers:
          type: array
          items:
            $ref: '#/components/schemas/GetServerConf'


    GetServerConf:
      title:     GetServerConf
      description: 动态 http server ssl server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string
        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        certificates:
          type: array
          items:
            $ref: '#/components/schemas/CertGroup'
    CertGroup:
      title: CertGroupConf
      description: 证书组配置信息
      type: object
      properties:
        cert_type:
          type: string
          description: 证书类型 regular:标准证书   ntls:国密证书 国密需要都配置，其他证书不配置enc相关字段
          example: ntls
        certificate:
          type: string
          description: cert证书文件内容base64(注意需要配置'data:')
          example: "data:-----BEGIN CERTIFICATE-----\r\nMIICJjCCAcygAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAcwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYYxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRowGAYDVQQDDBFzZXJ2\r\nZXIgc2lnbiAoU00yKTBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABKWvwarEtpHK\r\nckfCaWTcqhzO8e3z02Q2+NOyUr94WgvuQFuJjdLMoHCDxDuMrhXaxUXhXE/rZsG7\r\nZ4XWyd7K/n+jGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgbAMAoGCCqBHM9VAYN1\r\nA0gAMEUCIQDXyNeZN0iqy2cxDYytWtWLayYdSmdnuPg8VOhXUWWb+QIgEwj6Y2xg\r\n38xlatqML8aO89O4movtgOxPZG2YYTaDon8=\r\n-----END CERTIFICATE-----\r\n"
        certificateKey:
          type: string
          description: 证书key文件内容base64(注意需要配置'data:')
          example: "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIPunrzGM/F+w/64DaLg5RLn2ZUoqYEzwsgxUkeKWF6jmoAoGCCqBHM9V\r\nAYItoUQDQgAEpa/BqsS2kcpyR8JpZNyqHM7x7fPTZDb407JSv3haC+5AW4mN0syg\r\ncIPEO4yuFdrFReFcT+tmwbtnhdbJ3sr+fw==\r\n-----END EC PRIVATE KEY-----\r\n"
        certificateEnc:
          type: string
          description: cert证书文件enc内容base64(注意需要配置'data:')
          example: "data:-----BEGIN CERTIFICATE-----\r\nMIICJTCCAcugAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAgwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYUxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRkwFwYDVQQDDBBzZXJ2\r\nZXIgZW5jIChTTTIpMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEdGUnMh317JO6\r\nePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaAwMC16wze1n3JjSIqIMf7ielE\r\nePXi4G1dsKMaMBgwCQYDVR0TBAIwADALBgNVHQ8EBAMCAzgwCgYIKoEcz1UBg3UD\r\nSAAwRQIhANC8B9M4Mo0GDWDgSeJ5M4wlF0QgWMlY29GYB1/bGxq6AiAb4IlVTwnZ\r\nV0/PP3zuNjJjp7blkvsaCGsXtb9pk2rQYw==\r\n-----END CERTIFICATE-----\r\n"
        certificateKeyEnc:
          type: string
          description: 证书key文件enc内容base64(注意需要配置'data:')
          example: "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIJH/qG2bKhybB1uUSnQbKiNueeLneQVioS+0kLMygYCVoAoGCCqBHM9V\r\nAYItoUQDQgAEdGUnMh317JO6ePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaA\r\nwMC16wze1n3JjSIqIMf7ielEePXi4G1dsA==\r\n-----END EC PRIVATE KEY-----\r\n"


    ServerConf:
      title:     ServerConf
      description: 动态 http server ssl server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string
        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        type:
          type: string
          description: add or del, 添加或者删除动态证书
          example: add
        cert_info:
          type: object
          description: 证书配置信息
          properties:
            cert_type:
              type: string
              description: 证书类型 regular:标准证书   ntls:国密证书 国密需要都配置，其他证书不配置enc相关字段
              example: ntls
            certificate:
              type: string
              description: cert证书文件内容base64(注意需要配置'data:')
              example: "data:-----BEGIN CERTIFICATE-----\r\nMIICJjCCAcygAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAcwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYYxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRowGAYDVQQDDBFzZXJ2\r\nZXIgc2lnbiAoU00yKTBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABKWvwarEtpHK\r\nckfCaWTcqhzO8e3z02Q2+NOyUr94WgvuQFuJjdLMoHCDxDuMrhXaxUXhXE/rZsG7\r\nZ4XWyd7K/n+jGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgbAMAoGCCqBHM9VAYN1\r\nA0gAMEUCIQDXyNeZN0iqy2cxDYytWtWLayYdSmdnuPg8VOhXUWWb+QIgEwj6Y2xg\r\n38xlatqML8aO89O4movtgOxPZG2YYTaDon8=\r\n-----END CERTIFICATE-----\r\n"
            certificateKey:
              type: string
              description: 证书key文件内容base64(注意需要配置'data:')
              example: "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIPunrzGM/F+w/64DaLg5RLn2ZUoqYEzwsgxUkeKWF6jmoAoGCCqBHM9V\r\nAYItoUQDQgAEpa/BqsS2kcpyR8JpZNyqHM7x7fPTZDb407JSv3haC+5AW4mN0syg\r\ncIPEO4yuFdrFReFcT+tmwbtnhdbJ3sr+fw==\r\n-----END EC PRIVATE KEY-----\r\n"
            certificateEnc:
              type: string
              description: cert证书文件enc内容base64(注意需要配置'data:')
              example: "data:-----BEGIN CERTIFICATE-----\r\nMIICJTCCAcugAwIBAgIUWzTw8i+wQJSx7s4Rv9IX/RjzcAgwCgYIKoEcz1UBg3Uw\r\ngYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl\r\nMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM\r\nU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTIyMTIwNjA3\r\nMjI1NVoXDTI3MDExNDA3MjI1NVowgYUxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC\r\nSjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu\r\nb2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRkwFwYDVQQDDBBzZXJ2\r\nZXIgZW5jIChTTTIpMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEdGUnMh317JO6\r\nePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaAwMC16wze1n3JjSIqIMf7ielE\r\nePXi4G1dsKMaMBgwCQYDVR0TBAIwADALBgNVHQ8EBAMCAzgwCgYIKoEcz1UBg3UD\r\nSAAwRQIhANC8B9M4Mo0GDWDgSeJ5M4wlF0QgWMlY29GYB1/bGxq6AiAb4IlVTwnZ\r\nV0/PP3zuNjJjp7blkvsaCGsXtb9pk2rQYw==\r\n-----END CERTIFICATE-----\r\n"
            certificateKeyEnc:
              type: string
              description: 证书key文件enc内容base64(注意需要配置'data:')
              example: "data:-----BEGIN EC PARAMETERS-----\r\nBggqgRzPVQGCLQ==\r\n-----END EC PARAMETERS-----\r\n-----BEGIN EC PRIVATE KEY-----\r\nMHcCAQEEIJH/qG2bKhybB1uUSnQbKiNueeLneQVioS+0kLMygYCVoAoGCCqBHM9V\r\nAYItoUQDQgAEdGUnMh317JO6ePSKfs4f4xfQXReq/GWQKqL1vBnla3F22OSbNsaA\r\nwMC16wze1n3JjSIqIMf7ielEePXi4G1dsA==\r\n-----END EC PRIVATE KEY-----\r\n"
```



## 6.8 **动态upstream**配置**API**说明

```
swagger: '2.0'
info:
  version: '7.0'
  title: NJET REST API
  description: NJET REST
    [API](https://njet.org/en/docs/http/njt_http_api_module.html)
    provides access to NJET status information,
    on-the-fly configuration of upstream servers and
    key-value pairs management for
    [http](https://njet.org/en/docs/http/njt_http_keyval_module.html) and
    [stream](https://njet.org/en/docs/stream/njt_stream_keyval_module.html).
basePath: /api/7
tags:
  - name: HTTP Upstreams
  - name: Stream Upstreams
  - name: Method GET
  - name: Method POST
  - name: Method PATCH
  - name: Method DELETE
schemes:
  - http
  - https
paths:
  /http/upstreams/:
    get:
      tags:
        - HTTP Upstreams
        - Method GET
      summary: Return status of all HTTP upstream server groups
      description: Returns status of each HTTP upstream server group
        and its servers.
      operationId: getHttpUpstreams
      produces:
        - application/json
      parameters:
        - name: fields
          in: query
          type: string
          description: Limits which fields of upstream server groups will be output.
            If the “<literal>fields</literal>” value is empty,
            only names of upstreams will be output.
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamMap'
        '404':
          description: Unknown version (*UnknownVersion*)
          schema:
            $ref: '#/definitions/NjetError'
  '/http/upstreams/{httpUpstreamName}/':
    parameters:
    - name: httpUpstreamName
      in: path
      description: The name of an HTTP upstream server group.
      required: true
      type: string
    get:
      tags:
        - HTTP Upstreams
        - Method GET
      summary: Return status of an HTTP upstream server group
      description: Returns status of a particular HTTP upstream server group
        and its servers.
      operationId: getHttpUpstreamName
      produces:
        - application/json
      parameters:
        - name: fields
          in: query
          type: string
          description: Limits which fields of the upstream server group will be output.
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetHTTPUpstream'
        '400':
          description: Upstream is static (*UpstreamStatic*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
    delete:
      tags:
        - HTTP Upstreams
        - Method DELETE
      summary: Reset statistics of an HTTP upstream server group
      description: Resets the statistics for each upstream server
        in an upstream server group and queue statistics.
      operationId: deleteHttpUpstreamStat
      produces:
        - application/json
      responses:
        '204':
          description: Success
        '400':
          description: Upstream is static (*UpstreamStatic*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
  '/http/upstreams/{httpUpstreamName}/servers/':
    parameters:
      - name: httpUpstreamName
        in: path
        description: The name of an upstream server group.
        required: true
        type: string
    get:
      tags:
        - HTTP Upstreams
        - Method GET
      summary: Return configuration of all servers in an HTTP upstream server group
      description: Returns configuration of each server
        in a particular HTTP upstream server group.
      operationId: getHttpUpstreamServers
      produces:
        - application/json
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamConfServerMap'
        '400':
          description: Upstream is static (*UpstreamStatic*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
    post:
      tags:
        - HTTP Upstreams
        - Method POST
      summary: Add a server to an HTTP upstream server group
      description: Adds a new server to an HTTP upstream server group.
        Server parameters are specified in the JSON format.
      operationId: postHttpUpstreamServer
      produces:
        - application/json
      parameters:
        - in: body
          name: postHttpUpstreamServer
          description: Address of a new server and other optional parameters
            in the JSON format.
            The "ID", "backup", and "service" parameters
            cannot be changed.
          required: true
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamConfServer'
      responses:
        '201':
          description: Created
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamConfServer'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid "parameter" value (*UpstreamConfFormatError*),
            missing "server" argument (*UpstreamConfFormatError*),
            unknown parameter "name" (*UpstreamConfFormatError*),
            nested object or list (*UpstreamConfFormatError*),
            "error" while parsing (*UpstreamBadAddress*),
            service upstream "host" may not have port (*UpstreamBadAddress*),
            service upstream "host" requires domain name (*UpstreamBadAddress*),
            invalid "weight" (*UpstreamBadWeight*),
            invalid "max_conns" (*UpstreamBadMaxConns*),
            invalid "max_fails" (*UpstreamBadMaxFails*),
            invalid "fail_timeout" (*UpstreamBadFailTimeout*),
            invalid "slow_start" (*UpstreamBadSlowStart*),
            reading request body failed *BodyReadError*),
            route is too long (*UpstreamBadRoute*),
            "service" is empty (*UpstreamBadService*),
            no resolver defined to resolve (*UpstreamConfNoResolver*),
            upstream "name" has no backup (*UpstreamNoBackup*),
            upstream "name" memory exhausted (*UpstreamOutOfMemory*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
        '409':
          description: Entry exists (*EntryExists*)
          schema:
            $ref: '#/definitions/NjetError'
        '415':
          description: JSON error (*JsonError*)
          schema:
            $ref: '#/definitions/NjetError'
  '/http/upstreams/{httpUpstreamName}/servers/{httpUpstreamServerId}':
    parameters:
      - name: httpUpstreamName
        in: path
        description: The name of the upstream server group.
        required: true
        type: string
      - name: httpUpstreamServerId
        in: path
        description: The ID of the server.
        required: true
        type: string
    get:
      tags:
        - HTTP Upstreams
        - Method GET
      summary: Return configuration of a server in an HTTP upstream server group
      description: Returns configuration of a particular server
        in the HTTP upstream server group.
      operationId: getHttpUpstreamPeer
      produces:
        - application/json
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamConfServer'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid server ID (*UpstreamBadServerId*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Server with ID "id" does not exist (*UpstreamServerNotFound*),
            unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
    patch:
      tags:
        - HTTP Upstreams
        - Method PATCH
      summary: Modify a server in an HTTP upstream server group
      description: Modifies settings of a particular server
        in an HTTP upstream server group.
        Server parameters are specified in the JSON format.
      operationId: patchHttpUpstreamPeer
      produces:
        - application/json
      parameters:
        - in: body
          name: patchHttpUpstreamServer
          description: Server parameters, specified in the JSON format.
            The "ID", "backup", and "service" parameters
            cannot be changed.
          required: true
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamConfServer'
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamConfServer'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid "parameter" value (*UpstreamConfFormatError*),
            unknown parameter "name" (*UpstreamConfFormatError*),
            nested object or list (*UpstreamConfFormatError*),
            "error" while parsing (*UpstreamBadAddress*),
            invalid "server" argument (*UpstreamBadAddress*),
            invalid server ID (*UpstreamBadServerId*),
            invalid "weight" (*UpstreamBadWeight*),
            invalid "max_conns" (*UpstreamBadMaxConns*),
            invalid "max_fails" (*UpstreamBadMaxFails*),
            invalid "fail_timeout" (*UpstreamBadFailTimeout*),
            invalid "slow_start" (*UpstreamBadSlowStart*),
            reading request body failed *BodyReadError*),
            route is too long (*UpstreamBadRoute*),
            "service" is empty (*UpstreamBadService*),
            server "ID" address is immutable (*UpstreamServerImmutable*),
            server "ID" weight is immutable (*UpstreamServerWeightImmutable*),
            upstream "name" memory exhausted (*UpstreamOutOfMemory*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Server with ID "id" does not exist (*UpstreamServerNotFound*),
            unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
        '415':
          description: JSON error (*JsonError*)
          schema:
            $ref: '#/definitions/NjetError'
    delete:
      tags:
        - HTTP Upstreams
        - Method DELETE
      summary: Remove a server from an HTTP upstream server group
      description: Removes a server from an HTTP upstream server group.
      operationId: deleteHttpUpstreamServer
      produces:
        - application/json
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetHTTPUpstreamConfServerMap'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid server ID (*UpstreamBadServerId*),
            server "id" not removable (*UpstreamServerImmutable*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Server with ID "id" does not exist (*UpstreamServerNotFound*),
            unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
  /stream/upstreams/:
    get:
      tags:
        - Stream Upstreams
        - Method GET
      summary: Return status of all stream upstream server groups
      description: Returns status of each stream upstream server group
        and its servers.
      operationId: getStreamUpstreams
      produces:
        - application/json
      parameters:
        - name: fields
          in: query
          type: string
          description: Limits which fields of upstream server groups will be output.
            If the “<literal>fields</literal>” value is empty,
            only names of upstreams will be output.
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetStreamUpstreamMap'
        '404':
          description: Unknown version (*UnknownVersion*)
          schema:
            $ref: '#/definitions/NjetError'
  '/stream/upstreams/{streamUpstreamName}/':
    parameters:
    - name: streamUpstreamName
      in: path
      description: The name of a stream upstream server group.
      required: true
      type: string
    get:
      tags:
        - Stream Upstreams
        - Method GET
      summary: Return status of a stream upstream server group
      description: Returns status of a particular stream upstream server group
        and its servers.
      operationId: getStreamUpstream
      produces:
        - application/json
      parameters:
        - name: fields
          in: query
          type: string
          description: Limits which fields of the upstream server group will be output.
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetStreamUpstream'
        '400':
          description: Upstream is static (*UpstreamStatic*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
    delete:
      tags:
        - Stream Upstreams
        - Method DELETE
      summary: Reset statistics of a stream upstream server group
      description: Resets the statistics for each upstream server
        in an upstream server group.
      operationId: deleteStreamUpstreamStat
      produces:
        - application/json
      responses:
        '204':
          description: Success
        '400':
          description: Upstream is static (*UpstreamStatic*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
  '/stream/upstreams/{streamUpstreamName}/servers/':
    parameters:
      - name: streamUpstreamName
        in: path
        description: The name of an upstream server group.
        required: true
        type: string
    get:
      tags:
        - Stream Upstreams
        - Method GET
      summary: Return configuration of all servers in a stream upstream server group
      description: Returns configuration of each server
        in a particular stream upstream server group.
      operationId: getStreamUpstreamServers
      produces:
        - application/json
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetStreamUpstreamConfServerMap'
        '400':
          description: Upstream is static (*UpstreamStatic*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
    post:
      tags:
        - Stream Upstreams
        - Method POST
      summary: Add a server to a stream upstream server group
      description: Adds a new server to a stream upstream server group.
        Server parameters are specified in the JSON format.
      operationId: postStreamUpstreamServer
      produces:
        - application/json
      parameters:
        - in: body
          name: postStreamUpstreamServer
          description: Address of a new server and other optional parameters
            in the JSON format.
            The "ID", "backup", and "service" parameters
            cannot be changed.
          required: true
          schema:
            $ref: '#/definitions/NjetStreamUpstreamConfServer'
      responses:
        '201':
          description: Created
          schema:
            $ref: '#/definitions/NjetStreamUpstreamConfServer'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid "parameter" value (*UpstreamConfFormatError*),
            missing "server" argument (*UpstreamConfFormatError*),
            unknown parameter "name" (*UpstreamConfFormatError*),
            nested object or list (*UpstreamConfFormatError*),
            "error" while parsing (*UpstreamBadAddress*),
            no port in server "host" (*UpstreamBadAddress*),
            service upstream "host" may not have port (*UpstreamBadAddress*),
            service upstream "host" requires domain name (*UpstreamBadAddress*),
            invalid "weight" (*UpstreamBadWeight*),
            invalid "max_conns" (*UpstreamBadMaxConns*),
            invalid "max_fails" (*UpstreamBadMaxFails*),
            invalid "fail_timeout" (*UpstreamBadFailTimeout*),
            invalid "slow_start" (*UpstreamBadSlowStart*),
            "service" is empty (*UpstreamBadService*),
            no resolver defined to resolve (*UpstreamConfNoResolver*),
            upstream "name" has no backup (*UpstreamNoBackup*),
            upstream "name" memory exhausted (*UpstreamOutOfMemory*),
            reading request body failed *BodyReadError*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
        '409':
          description: Entry exists (*EntryExists*)
          schema:
            $ref: '#/definitions/NjetError'
        '415':
          description: JSON error (*JsonError*)
          schema:
            $ref: '#/definitions/NjetError'
  '/stream/upstreams/{streamUpstreamName}/servers/{streamUpstreamServerId}':
    parameters:
      - name: streamUpstreamName
        in: path
        description: The name of the upstream server group.
        required: true
        type: string
      - name: streamUpstreamServerId
        in: path
        description: The ID of the server.
        required: true
        type: string
    get:
      tags:
        - Stream Upstreams
        - Method GET
      summary: Return configuration of a server in a stream upstream server group
      description: Returns configuration of a particular server
        in the stream upstream server group.
      operationId: getStreamUpstreamServer
      produces:
        - application/json
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetStreamUpstreamConfServer'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid server ID (*UpstreamBadServerId*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*),
            server with ID "id" does not exist (*UpstreamServerNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
    patch:
      tags:
        - Stream Upstreams
        - Method PATCH
      summary: Modify a server in a stream upstream server group
      description: Modifies settings of a particular server
        in a stream upstream server group.
        Server parameters are specified in the JSON format.
      operationId: patchStreamUpstreamServer
      produces:
        - application/json
      parameters:
        - in: body
          name: patchStreamUpstreamServer
          description: Server parameters, specified in the JSON format.
            The "ID", "backup", and "service" parameters
            cannot be changed.
          required: true
          schema:
            $ref: '#/definitions/NjetStreamUpstreamConfServer'
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetStreamUpstreamConfServer'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid "parameter" value (*UpstreamConfFormatError*),
            unknown parameter "name" (*UpstreamConfFormatError*),
            nested object or list (*UpstreamConfFormatError*),
            "error" while parsing (*UpstreamBadAddress*),
            invalid "server" argument (*UpstreamBadAddress*),
            no port in server "host" (*UpstreamBadAddress*),
            invalid server ID (*UpstreamBadServerId*),
            invalid "weight" (*UpstreamBadWeight*),
            invalid "max_conns" (*UpstreamBadMaxConns*),
            invalid "max_fails" (*UpstreamBadMaxFails*),
            invalid "fail_timeout" (*UpstreamBadFailTimeout*),
            invalid "slow_start" (*UpstreamBadSlowStart*),
            reading request body failed *BodyReadError*),
            "service" is empty (*UpstreamBadService*),
            server "ID" address is immutable (*UpstreamServerImmutable*),
            server "ID" weight is immutable (*UpstreamServerWeightImmutable*),
            upstream "name" memory exhausted (*UpstreamOutOfMemory*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Server with ID "id" does not exist (*UpstreamServerNotFound*),
            unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
        '415':
          description: JSON error (*JsonError*)
          schema:
            $ref: '#/definitions/NjetError'
    delete:
      tags:
        - Stream Upstreams
        - Method DELETE
      summary: Remove a server from a stream upstream server group
      description: Removes a server from a stream server group.
      operationId: deleteStreamUpstreamServer
      produces:
        - application/json
      responses:
        '200':
          description: Success
          schema:
            $ref: '#/definitions/NjetStreamUpstreamConfServerMap'
        '400':
          description: |
            Upstream is static (*UpstreamStatic*),
            invalid server ID (*UpstreamBadServerId*),
            server "id" not removable (*UpstreamServerImmutable*)
          schema:
            $ref: '#/definitions/NjetError'
        '404':
          description: |
            Server with ID "id" does not exist (*UpstreamServerNotFound*),
            unknown version (*UnknownVersion*),
            upstream not found (*UpstreamNotFound*)
          schema:
            $ref: '#/definitions/NjetError'
        '405':
          description: Method disabled (*MethodDisabled*)
          schema:
            $ref: '#/definitions/NjetError'
###
###DEFINITIONS
###
definitions:
  ArrayOfStrings:
    title: Array
    description: |
      An array of strings.
    type: array
    items:
      type: string
  NjetHTTPServerZonesMap:
    title: HTTP Server Zones
    description: |
      Status data for all HTTP
      <a href="https://njet.org/en/docs/http/njt_http_api_module.html#status_zone">status zones</a>.
    type: object
    additionalProperties:
      $ref: '#/definitions/NjetHTTPServerZone'
    example:
      site1:
        processing: 2
        requests: 736395
        responses:
          1xx: 0
          2xx: 727290
          3xx: 4614
          4xx: 934
          5xx: 1535
          codes:
            200: 727270
            301: 4614
            404: 930
            503: 1535
          total: 734373
        discarded: 2020
        received: 180157219
        sent: 20183175459
        ssl:
          handshakes: 65432
          handshakes_failed: 421
          session_reuses: 4645
          no_common_protocol: 4
          no_common_cipher: 2
          handshake_timeout: 0
          peer_rejected_cert: 0
          verify_failures:
            no_cert: 0
            expired_cert: 2
            revoked_cert: 1
            hostname_mismatch: 2
            other: 1
      site2:
        processing: 1
        requests: 185307
        responses:
          1xx: 0
          2xx: 112674
          3xx: 45383
          4xx: 2504
          5xx: 4419
          codes:
            200: 112674
            301: 45383
            404: 2504
            503: 4419
          total: 164980
        discarded: 20326
        received: 51575327
        sent: 2983241510
        ssl:
          handshakes: 104303
          handshakes_failed: 1421
          session_reuses: 54645
          no_common_protocol: 4
          no_common_cipher: 2
          handshake_timeout: 0
          peer_rejected_cert: 0
          verify_failures:
            no_cert: 0
            expired_cert: 2
            revoked_cert: 1
            hostname_mismatch: 2
            other: 1
  NjetHTTPServerZone:
    title: HTTP Server Zone
    type: object
    properties:
      processing:
        type: integer
        description: The number of client requests
          that are currently being processed.
      requests:
        type: integer
        description: The total number of client requests received from clients.
      responses:
        description: The total number of responses sent to clients, the
          number of responses with status codes
          “<code>1xx</code>”, “<code>2xx</code>”, “<code>3xx</code>”,
          “<code>4xx</code>”, and “<code>5xx</code>”, and
          the number of responses per each status code.
        type: object
        readOnly: true
        properties:
          1xx:
            type: integer
            description: The number of responses with “<code>1xx</code>” status codes.
            readOnly: true
          2xx:
            type: integer
            description: The number of responses with “<code>2xx</code>” status codes.
            readOnly: true
          3xx:
           type: integer
           description: The number of responses with “<code>3xx</code>” status codes.
           readOnly: true
          4xx:
            type: integer
            description: The number of responses with “<code>4xx</code>” status codes.
            readOnly: true
          5xx:
            type: integer
            description: The number of responses with “<code>5xx</code>” status codes.
            readOnly: true
          codes:
            type: object
            description: The number of responses per each status code.
            readOnly: true
            properties:
              codeNumber:
                type: integer
                description: The number of responses with this particular status code.
                readOnly: true
          total:
            type: integer
            description: The total number of responses sent to clients.
            readOnly: true
      discarded:
        type: integer
        description: The total number of
          requests completed without sending a response.
      received:
        type: integer
        description: The total number of bytes received from clients.
      sent:
        type: integer
        description: The total number of bytes sent to clients.
      ssl:
        type: object
        readOnly: true
        properties:
          handshakes:
            type: integer
            description: The total number of successful SSL handshakes.
            readOnly: true
          handshakes_failed:
            type: integer
            description: The total number of failed SSL handshakes.
            readOnly: true
          session_reuses:
            type: integer
            description: The total number of session reuses during SSL handshake.
            readOnly: true
          no_common_protocol:
            type: integer
            description: The number of SSL handshakes failed
              because of no common protocol.
          no_common_cipher:
            type: integer
            description: The number of SSL handshakes failed
              because of no shared cipher.
          handshake_timeout:
            type: integer
            description: The number of SSL handshakes failed
              because of a timeout.
          peer_rejected_cert:
            type: integer
            description: The number of failed SSL handshakes
              when njet presented the certificate to the client
              but it was rejected with a corresponding alert message.
          verify_failures:
            type: object
            description: SSL certificate verification errors
            properties:
              no_cert:
                type: integer
                description: A client did not provide the required certificate.
              expired_cert:
                type: integer
                description: An expired or not yet valid certificate
                  was presented by a client.
              revoked_cert:
                type: integer
                description: A revoked certificate was presented by a client.
              other:
                type: integer
                description: Other SSL certificate verification errors.
    example:
      processing: 1
      requests: 706690
      responses:
        1xx: 0
        2xx: 699482
        3xx: 4522
        4xx: 907
        5xx: 266
        codes:
          200: 699482
          301: 4522
          404: 907
          503: 266
        total: 705177
      discarded: 1513
      received: 172711587
      sent: 19415530115
      ssl:
        handshakes: 104303
        handshakes_failed: 1421
        session_reuses: 54645
        no_common_protocol: 4
        no_common_cipher: 2
        handshake_timeout: 0
        peer_rejected_cert: 0
        verify_failures:
          no_cert: 0
          expired_cert: 2
          revoked_cert: 1
          other: 1
  NjetHTTPUpstreamMap:
    title: HTTP Upstreams
    description: |
      Status information of all HTTP
      <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#zone">dynamically configurable</a>
      <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#upstream">groups</a>.
    type: object
    additionalProperties:
      $ref: '#/definitions/NjetHTTPUpstream'
    example:
      trac-backend:
        peers:
          - id: 0
            server: 10.0.0.1:8088
            name: 10.0.0.1:8088
            backup: false
            weight: 5
            state: up
            active: 0
            ssl:
              handshakes: 620311
              handshakes_failed: 3432
              session_reuses: 36442
              no_common_protocol: 4
              handshake_timeout: 0
              peer_rejected_cert: 0
              verify_failures:
                expired_cert: 2
                revoked_cert: 1
                hostname_mismatch: 2
                other: 1
            requests: 667231
            header_time: 20
            response_time: 36
            responses:
              1xx: 0
              2xx: 666310
              3xx: 0
              4xx: 915
              5xx: 6
              codes:
                200: 666310
                404: 915
                503: 6
              total: 667231
            sent: 251946292
            received: 19222475454
            fails: 0
            unavail: 0
            health_checks:
              checks: 26214
              fails: 0
              unhealthy: 0
              last_passed: true
            downtime: 0
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
          - id: 1
            server: 10.0.0.1:8089
            name: 10.0.0.1:8089
            backup: true
            weight: 1
            state: unhealthy
            active: 0
            requests: 0
            responses:
              1xx: 0
              2xx: 0
              3xx: 0
              4xx: 0
              5xx: 0
              codes: {}
              total: 0
            sent: 0
            received: 0
            fails: 0
            unavail: 0
            health_checks:
              checks: 26284
              fails: 26284
              unhealthy: 1
              last_passed: false
            downtime: 262925617
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
        keepalive: 0
        zombies: 0
        zone: trac-backend
      hg-backend:
        peers:
          - id: 0
            server: 10.0.0.1:8088
            name: 10.0.0.1:8088
            backup: false
            weight: 5
            state: up
            active: 0
            ssl:
              handshakes: 620311
              handshakes_failed: 3432
              session_reuses: 36442
              no_common_protocol: 4
              handshake_timeout: 0
              peer_rejected_cert: 0
              verify_failures:
                expired_cert: 2
                revoked_cert: 1
                hostname_mismatch: 2
                other: 1
            requests: 667231
            header_time: 20
            response_time: 36
            responses:
              1xx: 0
              2xx: 666310
              3xx: 0
              4xx: 915
              5xx: 6
              codes:
                200: 666310
                404: 915
                503: 6
              total: 667231
            sent: 251946292
            received: 19222475454
            fails: 0
            unavail: 0
            health_checks:
              checks: 26214
              fails: 0
              unhealthy: 0
              last_passed: true
            downtime: 0
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
          - id: 1
            server: 10.0.0.1:8089
            name: 10.0.0.1:8089
            backup: true
            weight: 1
            state: unhealthy
            active: 0
            requests: 0
            responses:
              1xx: 0
              2xx: 0
              3xx: 0
              4xx: 0
              5xx: 0
              codes: {}
              total: 0
            sent: 0
            received: 0
            fails: 0
            unavail: 0
            health_checks:
              checks: 26284
              fails: 26284
              unhealthy: 1
              last_passed: false
            downtime: 262925617
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
        keepalive: 0
        zombies: 0
        zone: hg-backend
  NjetHTTPUpstream:
    title: HTTP Upstream
    type: object
    properties:
      peers:
        $ref: '#/definitions/NjetHTTPUpstreamPeerMap'
      keepalive:
        type: integer
        description: The current number of idle
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#keepalive">keepalive</a>
          connections.
      zombies:
        type: integer
        description: The current number of servers removed
          from the group but still processing active client requests.
      zone:
        type: string
        description: The name of the shared memory
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#zone">zone</a>
          that keeps the group’s configuration and run-time state.
      queue:
        type: object
        description: >
          For the requests
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#queue">queue</a>,
          the following data are provided:
        properties:
          size:
            type: integer
            description: The current number of requests in the queue.
          max_size:
            type: integer
            description: The maximum number of requests that can be in the queue
              at the same time.
          overflows:
            type: integer
            description: The total number of requests rejected due to the queue overflow.
    example:
      upstream_backend:
        peers:
          - id: 0
            server: 10.0.0.1:8088
            name: 10.0.0.1:8088
            backup: false
            weight: 5
            state: up
            active: 0
            ssl:
              handshakes: 620311
              handshakes_failed: 3432
              session_reuses: 36442
              no_common_protocol: 4
              handshake_timeout: 0
              peer_rejected_cert: 0
              verify_failures:
                expired_cert: 2
                revoked_cert: 1
                hostname_mismatch: 2
                other: 1
            max_conns: 20
            requests: 667231
            header_time: 20
            response_time: 36
            responses:
              1xx: 0
              2xx: 666310
              3xx: 0
              4xx: 915
              5xx: 6
              codes:
                200: 666310
                404: 915
                503: 6
              total: 667231
            sent: 251946292
            received: 19222475454
            fails: 0
            unavail: 0
            health_checks:
              checks: 26214
              fails: 0
              unhealthy: 0
              last_passed: true
            downtime: 0
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
          - id: 1
            server: 10.0.0.1:8089
            name: 10.0.0.1:8089
            backup: true
            weight: 1
            state: unhealthy
            active: 0
            max_conns: 20
            requests: 0
            responses:
              1xx: 0
              2xx: 0
              3xx: 0
              4xx: 0
              5xx: 0
              codes: {}
              total: 0
            sent: 0
            received: 0
            fails: 0
            unavail: 0
            health_checks:
              checks: 26284
              fails: 26284
              unhealthy: 1
              last_passed: false
            downtime: 262925617
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
        keepalive: 0
        zombies: 0
        zone: upstream_backend
  NjetHTTPUpstreamPeerMap:
    title: HTTP Upstream Servers
    description: |
      An array of HTTP
      <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#upstream">upstream servers</a>.
    type: array
    items:
      $ref: '#/definitions/NjetHTTPUpstreamPeer'
  NjetHTTPUpstreamPeer:
    title: HTTP Upstream Server
    type: object
    properties:
      id:
        type: integer
        description: The ID of the server.
        readOnly: true
      server:
        type: string
        description: An  <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#server">address</a>
          of the server.
      service:
        type: string
        description: The
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#service">service</a>
          parameter value of the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#server">server</a>
          directive.
      name:
        type: string
        description: The name of the server specified in the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#server">server</a>
          directive.
        readOnly: true
      backup:
        type: boolean
        description: A boolean value indicating whether the server is a
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#backup">backup</a>
          server.
      weight:
        type: integer
        description: <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#weight">Weight</a>
          of the server.
      state:
        type: string
        enum:
          - up
          - draining
          - down
          - unavail
          - checking
          - unhealthy
        description: Current state, which may be one of
          “<code>up</code>”, “<code>draining</code>”, “<code>down</code>”,
          “<code>unavail</code>”, “<code>checking</code>”,
          and “<code>unhealthy</code>”.
      active:
        type: integer
        description: The current number of active connections.
        readOnly: true
      ssl:
        type: object
        readOnly: true
        properties:
          handshakes:
            type: integer
            description: The total number of successful SSL handshakes.
            readOnly: true
          handshakes_failed:
            type: integer
            description: The total number of failed SSL handshakes.
            readOnly: true
          session_reuses:
            type: integer
            description: The total number of session reuses during SSL handshake.
            readOnly: true
          no_common_protocol:
            type: integer
            description: The number of SSL handshakes failed
              because of no common protocol.
          handshake_timeout:
            type: integer
            description: The number of SSL handshakes failed
              because of a timeout.
          peer_rejected_cert:
            type: integer
            description: The number of failed SSL handshakes
              when njet presented the certificate to the upstream server
              but it was rejected with a corresponding alert message.
          verify_failures:
            type: object
            description: SSL certificate verification errors
            properties:
              expired_cert:
                type: integer
                description: An expired or not yet valid certificate
                  was presented by an upstream server.
              revoked_cert:
                type: integer
                description: A revoked certificate was presented by an upstream server.
              hostname_mismatch:
                type: integer
                description: Server's certificate doesn't match the hostname.
              other:
                type: integer
                description: Other SSL certificate verification errors.
      max_conns:
        type: integer
        description: The
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#max_conns">max_conns</a>
          limit for the server.
      requests:
        type: integer
        description: The total number of client requests forwarded to this server.
        readOnly: true
      responses:
        type: object
        readOnly: true
        properties:
          1xx:
            type: integer
            description: The number of responses with “<code>1xx</code>” status codes.
            readOnly: true
          2xx:
            type: integer
            description: The number of responses with “<code>2xx</code>” status codes.
            readOnly: true
          3xx:
           type: integer
           description: The number of responses with “<code>3xx</code>” status codes.
           readOnly: true
          4xx:
            type: integer
            description: The number of responses with “<code>4xx</code>” status codes.
            readOnly: true
          5xx:
            type: integer
            description: The number of responses with “<code>5xx</code>” status codes.
            readOnly: true
          codes:
            type: object
            description: The number of responses per each status code.
            readOnly: true
            properties:
              codeNumber:
                type: integer
                description: The number of responses with this particular status code.
                readOnly: true
          total:
            type: integer
            description: The total number of responses obtained from this server.
            readOnly: true
      sent:
        type: integer
        description: The total number of bytes sent to this server.
        readOnly: true
      received:
        type: integer
        description: The total number of bytes received from this server.
        readOnly: true
      fails:
        type: integer
        description: The total number of unsuccessful attempts
          to communicate with the server.
        readOnly: true
      unavail:
        type: integer
        description: How many times the server became unavailable for client requests
          (state “<code>unavail</code>”) due to the number of unsuccessful
          attempts reaching the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#max_fails">max_fails</a>
          threshold.
        readOnly: true
      health_checks:
        type: object
        readOnly: true
        properties:
          checks:
            type: integer
            description: The total number of
              <a href="https://njet.org/en/docs/http/njt_http_upstream_hc_module.html#health_check">health check</a>
              requests made.
          fails:
            type: integer
            description: The number of failed health checks.
          unhealthy:
            type: integer
            description: How many times the server became unhealthy
              (state “<code>unhealthy</code>”).
          last_passed:
            type: boolean
            description: Boolean indicating if the last health check request was successful
              and passed
              <a href="https://njet.org/en/docs/http/njt_http_upstream_hc_module.html#match">tests</a>.
      downtime:
        type: integer
        readOnly: true
        description: Total time the server was in the “<code>unavail</code>”,
          “<code>checking</code>”, and “<code>unhealthy</code>” states.
      downstart:
        type: string
        format: date-time
        readOnly: true
        description: The time when the server became
          “<code>unavail</code>”, “<code>checking</code>”,
          or “<code>unhealthy</code>”,
          in the ISO 8601 format with millisecond resolution.
      selected:
        type: string
        format: date-time
        readOnly: true
        description: The time when the server was last selected to process a request,
          in the ISO 8601 format with millisecond resolution.
      header_time:
        type: integer
        readOnly: true
        description: The average time to get the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#var_upstream_header_time">response header</a>
          from the server.
      response_time:
        type: integer
        readOnly: true
        description: The average time to get the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#var_upstream_response_time">full response</a>
          from the server.
  NjetHTTPUpstreamConfServerMap:
    title: HTTP Upstream Servers
    description: An array of HTTP upstream servers for dynamic configuration.
    type: array
    items:
      $ref: '#/definitions/NjetHTTPUpstreamConfServer'
    example:
      - id: 0
        server: 10.0.0.1:8088
        weight: 1
        max_conns: 0
        max_fails: 0
        fail_timeout: 10s
        slow_start: 10s
        route: ''
        backup: false
        down: false
      - id: 1
        server: 10.0.0.1:8089
        weight: 4
        max_conns: 0
        max_fails: 0
        fail_timeout: 10s
        slow_start: 10s
        route: ''
        backup: true
        down: true
  NjetHTTPUpstreamConfServer:
    title: HTTP Upstream Server
    description: |
      Dynamically configurable parameters of an HTTP upstream
      <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#server">server</a>:
    type: object
    properties:
      id:
        type: integer
        description: The ID of the HTTP upstream server.
          The ID is assigned automatically and cannot be changed.
        readOnly: true
      server:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#address">address</a>
          parameter of the HTTP upstream server.
          When adding a server, it is possible to specify it as a domain name.
          In this case, changes of the IP addresses
          that correspond to a domain name will be monitored and automatically
          applied to the upstream configuration
          without the need of restarting njet.
          This requires the
          <a href="https://njet.org/en/docs/http/njt_http_core_module.html#resolver">resolver</a>
          directive in the “<code>http</code>” block.
          See also the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#resolve">resolve</a>
          parameter of the HTTP upstream server.
      service:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#service">service</a>
          parameter of the HTTP upstream server.
          This parameter cannot be changed.
        readOnly: true
      weight:
        type: integer
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#weight">weight</a>
          parameter of the HTTP upstream server.
      max_conns:
        type: integer
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#max_conns">max_conns</a>
          parameter of the HTTP upstream server.
      max_fails:
        type: integer
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#max_fails">max_fails</a>
          parameter of the HTTP upstream server.
      fail_timeout:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#fail_timeout">fail_timeout</a>
          parameter of the HTTP upstream server.
      slow_start:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#slow_start">slow_start</a>
          parameter of the HTTP upstream server.
      route:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#route">route</a>
          parameter of the HTTP upstream server.
      backup:
        type: boolean
        description: When <code>true</code>, adds a
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#backup">backup</a>
          server.
          This parameter cannot be changed.
        readOnly: true
      down:
        type: boolean
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#down">down</a>
          parameter of the HTTP upstream server.
      drain:
        type: boolean
        description: Same as the
          <a href="https://njet.org/en/docs/http/njt_http_upstream_module.html#drain">drain</a>
          parameter of the HTTP upstream server.
      parent:
        type: string
        description: Parent server ID of the resolved server.
          The ID is assigned automatically and cannot be changed.
        readOnly: true
      host:
        type: string
        description: Hostname of the resolved server.
          The hostname is assigned automatically and cannot be changed.
        readOnly: true
    example:
      id: 1
      server: 10.0.0.1:8089
      weight: 4
      max_conns: 0
      max_fails: 0
      fail_timeout: 10s
      slow_start: 10s
      route: ''
      backup: true
      down: true
  NjetStreamServerZonesMap:
    title: Stream Server Zones
    description: |
      Status information for all stream
      <a href="https://njet.org/en/docs/http/njt_http_api_module.html#status_zone">status zones</a>.
    type: object
    additionalProperties:
      $ref: '#/definitions/NjetStreamServerZone'
    example:
      mysql-frontend:
        processing: 2
        connections: 270925
        sessions:
          2xx: 155564
          4xx: 0
          5xx: 0
          total: 270925
        discarded: 0
        received: 28988975
        sent: 3879346317
        ssl:
          handshakes: 76455
          handshakes_failed: 432
          session_reuses: 28770
          no_common_protocol: 4
          no_common_cipher: 2
          handshake_timeout: 0
          peer_rejected_cert: 0
          verify_failures:
            no_cert: 0
            expired_cert: 2
            revoked_cert: 1
            other: 1
      dns:
        processing: 1
        connections: 155569
        sessions:
          2xx: 155564
          4xx: 0
          5xx: 0
          total: 155569
        discarded: 0
        received: 4200363
        sent: 20489184
        ssl:
          handshakes: 2040
          handshakes_failed: 23
          session_reuses: 65
          no_common_protocol: 4
          no_common_cipher: 2
          handshake_timeout: 0
          peer_rejected_cert: 0
          verify_failures:
            no_cert: 0
            expired_cert: 2
            revoked_cert: 1
            other: 1
  NjetStreamServerZone:
    title: Stream Server Zone
    type: object
    properties:
      processing:
        type: integer
        description: The number of client connections
          that are currently being processed.
      connections:
        type: integer
        description: The total number of connections accepted from clients.
      sessions:
        type: object
        description: The total number of completed sessions,
          and the number of sessions completed with status codes
          “<code>2xx</code>”, “<code>4xx</code>”, or “<code>5xx</code>”.
        properties:
          2xx:
            type: integer
            description: The total number of sessions completed with
              <a href="https://njet.org/en/docs/stream/njt_stream_core_module.html#var_status">status codes</a>
              “<code>2xx</code>”.
          4xx:
            type: integer
            description: The total number of sessions completed with
              <a href="https://njet.org/en/docs/stream/njt_stream_core_module.html#var_status">status codes</a>
              “<code>4xx</code>”.
          5xx:
            type: integer
            description: The total number of sessions completed with
              <a href="https://njet.org/en/docs/stream/njt_stream_core_module.html#var_status">status codes</a>
              “<code>5xx</code>”.
          total:
            type: integer
            description: The total number of completed client sessions.
      discarded:
        type: integer
        description: The total number of
          connections completed without creating a session.
      received:
        type: integer
        description: The total number of bytes received from clients.
      sent:
        type: integer
        description: The total number of bytes sent to clients.
      ssl:
        type: object
        readOnly: true
        properties:
          handshakes:
            type: integer
            description: The total number of successful SSL handshakes.
            readOnly: true
          handshakes_failed:
            type: integer
            description: The total number of failed SSL handshakes.
            readOnly: true
          session_reuses:
            type: integer
            description: The total number of session reuses during SSL handshake.
            readOnly: true
          no_common_protocol:
            type: integer
            description: The number of SSL handshakes failed
              because of no common protocol.
          no_common_cipher:
            type: integer
            description: The number of SSL handshakes failed
              because of no shared cipher.
          handshake_timeout:
            type: integer
            description: The number of SSL handshakes failed
              because of a timeout.
          peer_rejected_cert:
            type: integer
            description: The number of failed SSL handshakes
              when njet presented the certificate to the client
              but it was rejected with a corresponding alert message.
          verify_failures:
            type: object
            description: SSL certificate verification errors
            properties:
              no_cert:
                type: integer
                description: A client did not provide the required certificate.
              expired_cert:
                type: integer
                description: An expired or not yet valid certificate
                  was presented by a client.
              revoked_cert:
                type: integer
                description: A revoked certificate was presented by a client.
              other:
                type: integer
                description: Other SSL certificate verification errors.
    example:
      dns:
        processing: 1
        connections: 155569
        sessions:
          2xx: 155564
          4xx: 0
          5xx: 0
          total: 155569
        discarded: 0
        received: 4200363
        sent: 20489184
        ssl:
          handshakes: 76455
          handshakes_failed: 432
          session_reuses: 28770
          no_common_protocol: 4
          no_common_cipher: 2
          handshake_timeout: 0
          peer_rejected_cert: 0
          verify_failures:
            no_cert: 0
            expired_cert: 2
            revoked_cert: 1
            other: 1
  NjetStreamUpstreamMap:
    title: Stream Upstreams
    description: Status information of stream upstream server groups.
    type: object
    additionalProperties:
      $ref: '#/definitions/NjetStreamUpstream'
    example:
      mysql_backends:
        peers:
          - id: 0
            server: 10.0.0.1:12345
            name: 10.0.0.1:12345
            backup: false
            weight: 5
            state: up
            active: 0
            ssl:
              handshakes: 1045
              handshakes_failed: 89
              session_reuses: 321
              no_common_protocol: 4
              handshake_timeout: 0
              peer_rejected_cert: 0
              verify_failures:
                expired_cert: 2
                revoked_cert: 1
                hostname_mismatch: 2
                other: 1
            max_conns: 30
            connecions: 1231
            sent: 251946292
            received: 19222475454
            fails: 0
            unavail: 0
            health_checks:
              checks: 26214
              fails: 0
              unhealthy: 0
              last_passed: true
            downtime: 0
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
          - id: 1
            server: 10.0.0.1:12346
            name: 10.0.0.1:12346
            backup: true
            weight: 1
            state: unhealthy
            active: 0
            max_conns: 30
            connections: 0
            sent: 0
            received: 0
            fails: 0
            unavail: 0
            health_checks:
              checks: 26284
              fails: 26284
              unhealthy: 1
              last_passed: false
            downtime: 262925617
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
        zombies: 0
        zone: mysql_backends
      dns:
        peers:
          - id: 0
            server: 10.0.0.1:12347
            name: 10.0.0.1:12347
            backup: false
            weight: 5
            state: up
            active: 0
            ssl:
              handshakes: 5268
              handshakes_failed: 121
              session_reuses: 854
              no_common_protocol: 4
              handshake_timeout: 0
              peer_rejected_cert: 0
              verify_failures:
                expired_cert: 2
                revoked_cert: 1
                hostname_mismatch: 2
                other: 1
            max_conns: 30
            connections: 667231
            sent: 251946292
            received: 19222475454
            fails: 0
            unavail: 0
            health_checks:
              checks: 26214
              fails: 0
              unhealthy: 0
              last_passed: true
            downtime: 0
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
          - id: 1
            server: 10.0.0.1:12348
            name: 10.0.0.1:12348
            backup: true
            weight: 1
            state: unhealthy
            active: 0
            connections: 0
            max_conns: 30
            sent: 0
            received: 0
            fails: 0
            unavail: 0
            health_checks:
              checks: 26284
              fails: 26284
              unhealthy: 1
              last_passed: false
            downtime: 262925617
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
        zombies: 0
        zone: dns
  NjetStreamUpstream:
    title: Stream Upstream
    type: object
    properties:
      peers:
        $ref: '#/definitions/NjetStreamUpstreamPeerMap'
      zombies:
        type: integer
        description: The current number of servers removed from the group
          but still processing active client connections.
      zone:
        type: string
        description: The name of the shared memory
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#zone">zone</a>
          that keeps the group’s configuration and run-time state.
    example:
      dns:
        peers:
          - id: 0
            server: 10.0.0.1:12347
            name: 10.0.0.1:12347
            backup: false
            weight: 5
            state: up
            active: 0
            ssl:
              handshakes: 200
              handshakes_failed: 4
              session_reuses: 189
              no_common_protocol: 4
              handshake_timeout: 0
              peer_rejected_cert: 0
              verify_failures:
                expired_cert: 2
                revoked_cert: 1
                hostname_mismatch: 2
                other: 1
            max_conns: 50
            connections: 667231
            sent: 251946292
            received: 19222475454
            fails: 0
            unavail: 0
            health_checks:
              checks: 26214
              fails: 0
              unhealthy: 0
              last_passed: true
            downtime: 0
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
          - id: 1
            server: 10.0.0.1:12348
            name: 10.0.0.1:12348
            backup: true
            weight: 1
            state: unhealthy
            active: 0
            max_conns: 50
            connections: 0
            sent: 0
            received: 0
            fails: 0
            unavail: 0
            health_checks:
              checks: 26284
              fails: 26284
              unhealthy: 1
              last_passed: false
            downtime: 262925617
            downstart: 2022-06-28T11:09:21.602Z
            selected: 2022-06-28T15:01:25Z
        zombies: 0
        zone: dns
  NjetStreamUpstreamPeerMap:
    title: Stream Upstream Servers
    description: Array of stream upstream servers.
    type: array
    items:
      $ref: '#/definitions/NjetStreamUpstreamPeer'
  NjetStreamUpstreamPeer:
    title: Stream Upstream Server
    type: object
    properties:
      id:
        type: integer
        description: The ID of the server.
        readOnly: true
      server:
        type: string
        description: An
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#server">address</a>
          of the server.
      service:
        type: string
        description: The
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#service">service</a>
          parameter value of the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#server">server</a>
          directive.
      name:
        type: string
        format: hostname
        description: The name of the server specified in the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#server">server</a>
          directive.
        readOnly: true
      backup:
        type: boolean
        description: A boolean value indicating whether the server is a
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#backup">backup</a>
          server.
      weight:
        type: integer
        description: <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#weight">Weight</a>
          of the server.
      state:
        type: string
        readOnly: true
        enum:
          - up
          - down
          - unavail
          - checking
          - unhealthy
        description: Current state, which may be one of
          “<code>up</code>”, “<code>down</code>”, “<code>unavail</code>”,
          “<code>checking</code>”, or “<code>unhealthy</code>”.
      active:
        type: integer
        description: The current number of connections.
        readOnly: true
      ssl:
        type: object
        readOnly: true
        properties:
          handshakes:
            type: integer
            description: The total number of successful SSL handshakes.
            readOnly: true
          handshakes_failed:
            type: integer
            description: The total number of failed SSL handshakes.
            readOnly: true
          session_reuses:
            type: integer
            description: The total number of session reuses during SSL handshake.
            readOnly: true
          no_common_protocol:
            type: integer
            description: The number of SSL handshakes failed
              because of no common protocol.
          handshake_timeout:
            type: integer
            description: The number of SSL handshakes failed
              because of a timeout.
          peer_rejected_cert:
            type: integer
            description: The number of failed SSL handshakes
              when njet presented the certificate to the upstream server
              but it was rejected with a corresponding alert message.
          verify_failures:
            type: object
            description: SSL certificate verification errors
            properties:
              expired_cert:
                type: integer
                description: An expired or not yet valid certificate
                  was presented by an upstream server.
              revoked_cert:
                type: integer
                description: A revoked certificate was presented by an upstream server.
              hostname_mismatch:
                type: integer
                description: Server's certificate doesn't match the hostname.
              other:
                type: integer
                description: Other SSL certificate verification errors.
      max_conns:
        type: integer
        description: The
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#max_conns">max_conns</a>
          limit for the server.
      connections:
        type: integer
        description: The total number of client connections forwarded to this server.
        readOnly: true
      connect_time:
        type: integer
        description: The average time to connect to the upstream server.
        readOnly: true
      first_byte_time:
        type: integer
        description: The average time to receive the first byte of data.
        readOnly: true
      response_time:
        type: integer
        description: The average time to receive the last byte of data.
        readOnly: true
      sent:
        type: integer
        description: The total number of bytes sent to this server.
        readOnly: true
      received:
        type: integer
        description: The total number of bytes received from this server.
        readOnly: true
      fails:
        type: integer
        description: The total number of unsuccessful attempts
          to communicate with the server.
        readOnly: true
      unavail:
        type: integer
        description: How many times the server became unavailable for client connections
          (state “<code>unavail</code>”) due to the number of unsuccessful
          attempts reaching the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#max_fails">max_fails</a>
          threshold.
        readOnly: true
      health_checks:
        type: object
        readOnly: true
        properties:
          checks:
            type: integer
            description: The total number of
              <a href="https://njet.org/en/docs/stream/njt_stream_upstream_hc_module.html#health_check">health check</a>
              requests made.
            readOnly: true
          fails:
            type: integer
            description: The number of failed health checks.
            readOnly: true
          unhealthy:
            type: integer
            description: How many times the server became unhealthy
              (state “<code>unhealthy</code>”).
            readOnly: true
          last_passed:
            type: boolean
            description: Boolean indicating whether the last health check request
              was successful and passed
              <a href="https://njet.org/en/docs/stream/njt_stream_upstream_hc_module.html#match">tests</a>.
            readOnly: true
      downtime:
        type: integer
        description: Total time the server was in the
          “<code>unavail</code>”, “<code>checking</code>”,
          and “<code>unhealthy</code>” states.
        readOnly: true
      downstart:
        type: string
        format: date-time
        description: The time when the server became
          “<code>unavail</code>”, “<code>checking</code>”,
          or “<code>unhealthy</code>”,
          in the ISO 8601 format with millisecond resolution.
        readOnly: true
      selected:
        type: string
        format: date-time
        description: The time when the server was last selected
          to process a connection,
          in the ISO 8601 format with millisecond resolution.
        readOnly: true
  NjetStreamUpstreamConfServerMap:
    title: Stream Upstream Servers
    description: |
      An array of stream upstream servers for dynamic configuration.
    type: array
    items:
      $ref: '#/definitions/NjetStreamUpstreamConfServer'
    example:
      - id: 0
        server: 10.0.0.1:12348
        weight: 1
        max_conns: 0
        max_fails: 1
        fail_timeout: 10s
        slow_start: 0
        backup: false
        down: false
      - id: 1
        server: 10.0.0.1:12349
        weight: 1
        max_conns: 0
        max_fails: 1
        fail_timeout: 10s
        slow_start: 0
        backup: false
        down: false
  NjetStreamUpstreamConfServer:
    title: Stream Upstream Server
    description: |
      Dynamically configurable parameters of a stream upstream
      <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#server">server</a>:
    type: object
    properties:
      id:
        type: integer
        description: The ID of the stream upstream server.
          The ID is assigned automatically and cannot be changed.
        readOnly: true
      server:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#server">address</a>
          parameter of the stream upstream server.
          When adding a server, it is possible to specify it as a domain name.
          In this case, changes of the IP addresses
          that correspond to a domain name will be monitored and automatically
          applied to the upstream configuration
          without the need of restarting njet.
          This requires the
          <a href="https://njet.org/en/docs/stream/njt_stream_core_module.html#resolver">resolver</a>
          directive in the “<code>stream</code>” block.
          See also the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#resolve">resolve</a>
          parameter of the stream upstream server.
      service:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#service">service</a>
          parameter of the stream upstream server.
          This parameter cannot be changed.
        readOnly: true
      weight:
        type: integer
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#weight">weight</a>
          parameter of the stream upstream server.
      max_conns:
        type: integer
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#max_conns">max_conns</a>
          parameter of the stream upstream server.
      max_fails:
        type: integer
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#max_fails">max_fails</a>
          parameter of the stream upstream server.
      fail_timeout:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#fail_timeout">fail_timeout</a>
          parameter of the stream upstream server.
      slow_start:
        type: string
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#slow_start">slow_start</a>
          parameter of the stream upstream server.
      backup:
        type: boolean
        description: When <code>true</code>, adds a
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#backup">backup</a>
          server.
          This parameter cannot be changed.
        readOnly: true
      down:
        type: boolean
        description: Same as the
          <a href="https://njet.org/en/docs/stream/njt_stream_upstream_module.html#down">down</a>
          parameter of the stream upstream server.
      parent:
        type: string
        description: Parent server ID of the resolved server.
          The ID is assigned automatically and cannot be changed.
        readOnly: true
      host:
        type: string
        description: Hostname of the resolved server.
          The hostname is assigned automatically and cannot be changed.
        readOnly: true
    example:
      id: 0
      server: 10.0.0.1:12348
      weight: 1
      max_conns: 0
      max_fails: 1
      fail_timeout: 10s
      slow_start: 0
      backup: false
      down: false
  NjetError:
    title: Error
    description: |
      njet error object.
    type: object
    properties:
      error:
        type: object
        properties:
          status:
            type: integer
            description: HTTP error code.
          text:
            type: string
            description: Error description.
          code:
            type: string
            description: Internal njet error code.
      request_id:
        type: string
        description: The ID of the request, equals the value of the
          <a href="https://njet.org/en/docs/http/njt_http_core_module.html#var_request_id">$request_id</a>
          variable.
      href:
        type: string
        description: Link to reference documentation.
```



## 6.9 **动态**location**配置**API说明

```
openapi: 3.0.3
info:
  title: njet-api
  description: njet-api
  version: 1.0.0
servers:
  - url: '/'
paths:
  /dyn_loc:
    put:
      tags:
        - dyn_loc
      summary: Delete a dynamic location from server
      description: Delete a dynamic location from server.
      requestBody:
          content:
            application/json:
              schema:
                  $ref: '#/components/schemas/del_location'
          required: true
      responses:
        '200':
          description: result info
    post:
      tags:
        - dyn_loc
      summary: Add a dynamic location to server
      description: Add a dynamic location to server.
      requestBody:
          content:
            application/json:
              schema:
                  $ref: '#/components/schemas/add_location'
          required: true
      responses:
       '200':
          description: result info
###DEFINITIONS
components:
  schemas:
    del_location:
      title: dyn_loc
      description: delete dynamic location from server
      type: object
      properties:
        type:
          type: string
          description: fix value "del".
          example: del
        addr_port:
          type: string
          description: addr and port of server.
          example: 192.168.40.203:8081
        server_name:
          type: string
          description:  server name .
          example: cluster.tmlake.com
        location_rule:
          type: string
          description:  Regular Expression.
          example: =
        location_name:
          type: string
          description:  the name of location.
          example: /clb
    add_location:
      title: dyn_loc
      description: add dynamic location to server
      type: object
      properties:
        type:
          type: string
          example: add
        addr_port:
          type: string
          description: addr and port of server.
          example: 192.168.40.203:8081
        server_name:
          type: string
          description:  server name .
          example: cluster.tmlake.com
        locations:
          type: array
          example:
           - location_rule: ""
             location_name: /
             location_body: api write=off;
             proxy_pass:  $dest_upstream;
           - location_rule: ""
             location_name: /test3
             location_body: "set $test 1;"
             proxy_pass:  $dest_upstream;
```



## 6.10 **动态sendmsg模块kv API说明**

```
openapi: 3.0.3
info:
  title: kv api
  description: 维护kv
  version: 1.0.0

servers:
  - url: '/'
paths:
  /kv:
    post:
      tags:
        - sendmsg kv
      summary: kvstore key set
      requestBody:
          content:
            application/json:
              schema:
                  $ref: '#/components/schemas/kvdata'
          required: true
      responses:
        '200':
          description: Success
        '500':
          description: Internal Server Error
    get:
      tags:
        - sendmsg kv
      summary: get value from kvstore 
      parameters:
        - in: query
          name: key
          schema:
            type: string
            example: 'test_key'
      responses:
        '200':
          description: Success
          content:
            text/plain:
              schema:
                type: string
                example: 'test msg'
        '404':
          description: Not Found
        '500':
          description: Internal Server Error
components:
  schemas:
    kvdata:
      title: kv post data
      type: object
      properties:
        key:
          type: string 
          example: 'test_key'
        value: 
          type: string 
          example: 'test msg' 
```



## 6.11 **动态**limit**配置**API**说明**

```
openapi: 3.0.3
info:
  title: dyn config api
  description: 动态limit限流
  version: 1.0.0

servers:
  - url: '/config/2'
paths:
  /config/:
    get:
      tags:
        - dyn config module
      summary: Returns a list of allow dynamic conf module
      description: Returns a list of allow dynamic conf module
      operationId: getDynConfModuleList
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModuleList'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'


  /config/http_dyn_limit:
    put:
      tags:
        - dyn limit config
      summary: set module conf
      description: set module conf
      operationId: setDynModuleConf
      requestBody:
          content:
            application/json:
              schema:
                  $ref: '#/components/schemas/MainConf'
          required: true
      responses:
        '204':
          description: Success
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
    get:
      tags:
        - dyn limit config
      summary: return dynamic module conf
      description: get module conf
      operationId: getDynModuleConf
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MainConf'
        '500':
          description: server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommonMsg'
components:
  schemas:
    CommonMsg:
      title: commonMsg
      description: 公共提示信息
      type: object
      properties:
        code:
          type: string
          description: 状态码
        msg:
          type: string
          description: 提示信息

    ModuleList:
      title: module list
      description: 支持动态配置的模块列表
      type: array
      items:
        type: string

    MainConf:
      title: main Conf
      description: 动态 main 级别配置
      type: object
      properties:
        servers:
          type: array
          items:
            $ref: '#/components/schemas/ServerConf'

        limit_rps:
          type: array
          items:
            $ref: '#/components/schemas/LimitRpsConf'

    LimitRpsConf:
      title:     LimitRpsConf
      description: 动态 limit rps配置
      type: object
      properties:
        zone:
          type: string
          description: zone 名称 .
          example: conn_zone_name

        rate:
          type: string
          description: rate 配置 .
          example: 20r/s

    ServerConf:
      title:     ServerConf
      description: 动态 server 级别配置
      type: object
      properties:
        listens:
          type: array
          description: server listen 端口列表
          items:
            type: string

        serverNames:
          type: array
          description: server Name 列表.
          items:
            type: string
        locations:
          type: array
          description: locations 列表
          items:
            $ref: '#/components/schemas/LocationConf'


    LocationConf:
      title: LocationConf
      description: location 级别 limit 配置信息
      type: object
      properties:
        location:
          type: string
          description: location 名称 .
          example: /demo
        limit_rate:
          type: string
          description: limit_rate 速率 .
          example: 1k
        limit_rate_after:
          type: string
          description: 超过该值后开始限速 .
          example: 5k
        limit_conns_scope:
          type: string
          description: 连接限制数来源， up_share表示来自于上一层， location表示当前location的连接限制数
          example: up_share
        limit_conns:
          type: array
          items:
            type: object
            description: tcp连接数限制
            properties:
              zone: 
                type: string
                example: perip
              conn: 
                type: int
                example: 10
        limit_conn_dry_run:
          type: string
          description: dry_run 开关 .
          enum: [on, off]
        limit_conn_log_level:
          type: string
          description: 超过连接限制后日志打印级别 .
          enum: [info, notice, warn, error]
        limit_conn_status:
          type: int
          description:  超过连接限制后返回码,范围[400, 599], 包含400和599.
          example: 503
        limit_reqs_scope:
          type: string
          description: 连接限制数来源， up_share表示来自于上一层， location表示当前location的连接限制数
          example: up_share
        limit_reqs:
          type: array
          items:
            type: object
            description: http请求数限制, burst以及delay,  delay可为正整数或者nodelay
            properties:
              zone: 
                type: string
                example: req_perip
              burst: 
                type: int
                example: 10
              delay: 
                type: string
                example: nodelay
        limit_req_dry_run:
          type: string
          description: dry_run 开关 .
          enum: [on, off]
        limit_req_log_level:
          type: string
          description: 超过连接限制后日志打印级别 .
          enum: [info, notice, warn, error]
        limit_req_status:
          type: int
          description:  超过连接限制后返回码,范围[400, 599], 包含400和599.
          example: 503
        locations:
          type: array
          description: locations 列表
          items:
            $ref: '#/components/schemas/LocationConf'
```



## 6.12 动态VS配置API说明

```
openapi: 3.0.3
info:
  title: njet-api
  description: njet-api
  version: 1.0.0
servers:

  - url: '/'
	paths:
	  /dyn_srv:
	put:
	  tags:
	   - dyn_srv
		summary: Delete a dynamic vs from server
		  description: Delete a dynamic vs from server.
		  requestBody:
		content:
		  application/json:
		    schema:
		        $ref: '#/components/schemas/del_vs'
		required: true
		  responses:
		    '200':
		description: result info
		post:
		  tags:
		   - dyn_srv
			summary: Add a dynamic vs to server
			  description: Add a dynamic vs to server.
			  requestBody:
			content:
			  application/json:
			    schema:
			$ref: '#/components/schemas/add_vs'
			required: true
			  responses:
			   '200':
			description: result info
			###DEFINITIONS
			components:
			  schemas:
			del_vs:
			  title: dyn_srv
			  description: delete dynamic vs from server
			  type: object
			  properties:
			    type:
			type: string
			description: fix value "del".
			example: del
			    addr_port:
			type: string
			description: addr and port of server.
			example: 0.0.0.0:90
			    server_name:
			type: string
			description:  server name .
			example: server-90
			add_vs:
			  title: dyn_srv
			  description: add dynamic vs to server
			  type: object
			  properties:
			    type:
			type: string
			example: add
			    addr_port:
			type: string
			description: addr and port of server.
			example: 0.0.0.0:90
			    listen_option:
			type: string
			description: parameter of listen
			example: ""
			    server_name:
			type: string
			description:  server name .
			example: server-90
			    server_body:
			type: string
			description:  content of server.
			example: "return 200 server-90"
```

