<table style="height: 1656px;"><colgroup><col width="233" /><col width="543" /></colgroup>
<tbody>
<tr style="height: 18px;">
<td style="height: 18px; width: 705px; text-align: center;" colspan="2"><strong>继承 nginx-1.23.1 所有功能， 并且100%兼容nginx</strong></td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 705px; text-align: center;" colspan="2"><strong>OpenNJet 功能特性</strong></td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 212.094px;" rowspan="2">Copilot框架</td>
<td style="height: 18px; width: 486.906px;">支持动态加载不同的外部copilot模块</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持外部模块异常退出的自动重启</td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 212.094px;" rowspan="2">KV模块</td>
<td style="height: 18px; width: 486.906px;">支持键值的查询及设置</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持键值的持久化</td>
</tr>
<tr style="height: 18px;">
<td style="height: 54px; width: 212.094px;" rowspan="3">动态配置框架</td>
<td style="height: 18px; width: 486.906px;">支持控制平面的消息发送</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持RPC消息、组播消息</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持消息持久化</td>
</tr>
<tr style="height: 18px;">
<td style="height: 54px; width: 212.094px;" rowspan="3">Cache-purge</td>
<td style="height: 18px; width: 486.906px;">支持缓存清理</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持按指定前缀清理缓存</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">开启分片后修改源文件不会造成下载失败</td>
</tr>
<tr style="height: 18px;">
<td style="height: 144px; width: 212.094px;" rowspan="8">health_check</td>
<td style="height: 18px; width: 486.906px;">支持单独在helper进程开启健康检查，不影响数据面业务</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行时动态开启或关闭健康检查功能</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持校验返回http code</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持校验返回http header</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持校验返回http body</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持https健康检查</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持强制健康检查，以及持久化功能</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持国密https健康检查</td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 212.094px;" rowspan="2">Split-clients-2</td>
<td style="height: 18px; width: 486.906px;">支持蓝绿发布</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行时动态调整流量比例</td>
</tr>
<tr style="height: 18px;">
<td style="height: 54px; width: 212.094px;" rowspan="3">黑白名单</td>
<td style="height: 18px; width: 486.906px;">支持黑名单方式进行访问IP的限制</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持白名单方式进行访问IP的限制</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行时动态设置IPv4的黑白名单列表</td>
</tr>
<tr style="height: 18px;">
<td style="height: 54px; width: 212.094px;" rowspan="3">doc模块</td>
<td style="height: 18px; width: 486.906px;">支持location 级别通过doc_api 指令配置，实现对swagger、gui页面的访问</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持通过swagger 页面实现对各功能opentapi的访问</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持通过gui页面实现对动态模块配置修改的能力</td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 212.094px;" rowspan="2">telemetry（外部编译模块）</td>
<td style="height: 18px; width: 486.906px;">支持http请求在不同server间的服务追踪</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持动态开关控制调用链的生成</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">正向代理（支持http/https）</td>
<td style="height: 18px; width: 486.906px;">实现了HTTP CONNECT 方法支持http/https正向代理访问</td>
</tr>
<tr style="height: 54px;">
<td style="height: 126px; width: 212.094px;" rowspan="5">vts模块</td>
<td style="height: 54px; width: 486.906px;">支持server的request、response、traffic、cache信息的统计，其中server的response可以按照response code进行分类统计，分类统计使用的response code为1xx、2xx、3xx、4xx、5xx</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持upstream和cache信息的统计</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持通过内嵌的html页面进行统计信息的展示</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持通过Prometheus、grafana进行统计信息的展示</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持动态配置server的location统计开关，支持动态配置server的filter key</td>
</tr>
<tr style="height: 18px;">
<td style="height: 72px; width: 212.094px;" rowspan="4">国密支持</td>
<td style="height: 18px; width: 486.906px;">支持server中使用国密</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持反向代理中使用国密</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持国密双证证书</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">动态（国密）证书更新</td>
</tr>
<tr style="height: 18px;">
<td style="height: 108px; width: 212.094px;" rowspan="6">动态access log</td>
<td style="height: 18px; width: 486.906px;">支持运行中动态关闭access log功能</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行中动态修改写入的日志文件</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行中切换syslog服务器</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行中切换写入文件的变量</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行时增加日志format</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持运行时修改日志format</td>
</tr>
<tr style="height: 18px;">
<td style="height: 54px; width: 212.094px;" rowspan="3">声明式API</td>
<td style="height: 18px; width: 486.906px;">支持感知声明式模块注册</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持查询声明式模块查询</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持PUT方式更新声明式配置</td>
</tr>
<tr style="height: 18px;">
<td style="height: 72px; width: 212.094px;" rowspan="4">边车支持</td>
<td style="height: 18px; width: 486.906px;">支持流量劫持，兼容istio 规则</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持协议识别</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持代理http1.1</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持 istio 的双向认证(service-to-service mTLS)</td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 212.094px;" rowspan="2">动态location 支持</td>
<td style="height: 18px; width: 486.906px;">支持通过api 向vs 添加 location</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持通过api 从vs中删除已经添加的location</td>
</tr>
<tr style="height: 18px;">
<td style="height: 90px; width: 212.094px;" rowspan="4">动态upstream api 支持</td>
<td style="height: 18px; width: 486.906px;">支持通过api，对http 或stream 中的upstream 信息进行查询</td>
</tr>
<tr style="height: 36px;">
<td style="height: 36px; width: 486.906px;">支持通过api，对http 或stream 中的upstream 的server 进行， 添加，修改，删除</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持通过api，对http 或stream 中的upstream 的统计信息进行重置</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持post 添加的upstream server 持久化或非持久化</td>
</tr>
<tr style="height: 36px;">
<td style="height: 54px; width: 212.094px;" rowspan="2">动态域名upstream server</td>
<td style="height: 36px; width: 486.906px;">支持静态配置upstream server 域名的reslove 属性，定时解析域名，根据域名对应的ip 增减结果，同步更新到upstream server 列表中</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持通过upstream api post接口，添加server 域名，并定时解析域名</td>
</tr>
<tr style="height: 18px;">
<td style="height: 54px; width: 212.094px;" rowspan="3">Http 会话保持支持</td>
<td style="height: 18px; width: 486.906px;">支持cookie 会话保持</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持route 会话保持</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持lear 会话保持</td>
</tr>
<tr style="height: 36px;">
<td style="height: 36px; width: 212.094px;">负载均衡</td>
<td style="height: 36px; width: 486.906px;">slow_start 慢启动功能,针对轮询算法，实现server新增或故障转正常后，业务的流量在指定时间，缓慢增长</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">动态SSL证书</td>
<td style="height: 18px; width: 486.906px;">动态ssl证书功能由声明式api改为命令式api</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">动态配额功能</td>
<td style="height: 18px; width: 486.906px;">配额（limit conn、limit rps、limit rate）功能声明式 API 动态化配置</td>
</tr>
<tr style="height: 54px;">
<td style="height: 54px; width: 212.094px;">组播集群</td>
<td style="height: 54px; width: 486.906px;">组播gossip构建集群功能 基于组播实现app_stickey 会话同步功能 基于组播实现limit connection集群连接数限制功能 基于组播实现limit rps集群请求数限制功能</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">动态worker数量调整</td>
<td style="height: 18px; width: 486.906px;">基于API动态调整worker进程数量 基于cpu使用率动态调整worker数量</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">HA Copilot</td>
<td style="height: 18px; width: 486.906px;">提供HA Copilot模块，NJet多个实例间支持VIP的配置</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">动态virtual server</td>
<td style="height: 18px; width: 486.906px;">动态添加VS 功能</td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 212.094px;" rowspan="2">FTP反向代理（被动模式）</td>
<td style="height: 18px; width: 486.906px;">支持纯明文ftp代理</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持隐式ftps代理</td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 212.094px;" rowspan="2">流量劫持</td>
<td style="height: 18px; width: 486.906px;">支持tcp流量劫持</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 486.906px;">支持动态api配置流量劫持</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">SNMP</td>
<td style="height: 18px; width: 486.906px;">通过SNMP接口输出指标</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">cache应用加速</td>
<td style="height: 18px; width: 486.906px;">支持通过动态api接口添加删除</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">设备注册</td>
<td style="height: 18px; width: 486.906px;">支持ADC设备注册</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">集群配置同步功能</td>
<td style="height: 18px; width: 486.906px;">HA/MA集群配置同步功能</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">沙箱功能</td>
<td style="height: 18px; width: 486.906px;">实现配置沙箱功能</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">alpn</td>
<td style="height: 18px; width: 486.906px;">proxy_ssl_alpn 设置alpn 值</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">KIC</td>
<td style="height: 18px; width: 486.906px;">OpenNJet KIC（详细参考KIC release 说明）</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">ModSecurity</td>
<td style="height: 18px; width: 486.906px;">集成ModSecurityV3，支持location 动态开启及关闭modsecurity</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">动态map</td>
<td style="height: 18px; width: 486.906px;">Stream动态Map设置</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 212.094px;">其他</td>
<td style="height: 18px; width: 486.906px;">Proxy protocol V2协议，支持设置TLV 字段值</td>
</tr>
</tbody>
</table>