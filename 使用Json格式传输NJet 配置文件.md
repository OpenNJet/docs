# 使用Json格式传输NJet 配置文件

当对外提供API时，Njet  的配置文件可以转换成JSON格式进行传输 。规则如下：

- 块类型的配置转换成 json 对象, 块配置的名称转换成json 对象的名称
- 块内的配置项转换成json 对象的属性
- 可重复的元素映射成对象数组 , 元素的名称映射到对象的 "_key" 字段
- "_script" 做为特殊的对象值，复杂的代码逻辑块配置，可以使用 "_script" 保持
- 对应的值与json类型的映射关系:
  - on/off 映射为 json的bool 类型
  - string 映射为json的string类型
  - number 映射为json的number类型
  - 空值 映射为 json的空字符串



# NJet 配置样例：

```
worker_processes  24; 
events { 
    worker_connections  10240; 
} 
http { 
    include       mime.types; 
    default_type  application/octet-stream; 
    sendfile        on; 
    access_log off; 
    keepalive_timeout  120s 120s; 
    keepalive_requests 200000; 
    upstream web { 
       server 192.168.0.230:28080; 
       keepalive 400; 
    } 
    server { 
        listen       80; 
        server_name  localhost; 
        location / { 
            proxy_pass http://web; 
            proxy_http_version 1.1; 
            proxy_set_header Connection ""; 
        } 
      
    } 
}
```

# JSON 配置

```
{    
    "worker_processes": 24,
    "events": {
        "worker_connections": 10240
    },
    "http": {
        "include": "mime.types",
        "default_type": "application/octet-stream",
        "keepalive_timeout": "120s 120s",
        "keepalive_requests": 200000,
        "sendfile": true,
        "access_log": false,
        "upstream": [{
            "_key": "web",
            "server": ["192.168.0.230:28080"],
            "keepalive": 400
        }],
        "server": [{
            "_key": "",
            "listen": 80,
            "server_name": "localhost",
            "location": [{
                "_key": "/",
                "proxy_pass": "http://web",
                "proxy_http_version": 1.1,
                "proxy_set_header": "Connection \"\""
            }]
        }]
    }
}
```

