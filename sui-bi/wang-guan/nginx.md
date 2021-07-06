# Nginx

例子：将http协议的请求自动重定向为https协议请求

```text
http {
    server {
        listen       443 ssl;
        # 当请求为 http 协议时通过状态码 497 触发错误重定向到 https 协议端口
        error_page 497 https://<外部地址>:<外部端口>$request_uri;
        ...
    }
}
```

例子：配置4层tcp代理并开启客户端IP透传

```text
stream {
    ...
    upstream web_dev {
        server 10.14.31.161:443;
        server 10.14.31.162:443;
        server 10.14.31.163:443;
    }
    server {
        listen 443;
        proxy_pass web_dev;
        # 开启透传功能，用于将连接信息从请求连接的源传递到请求连接到的目标
        # 需要通过 `nginx -V 2>&1 | grep -- 'stream_realip_module'` 命令检查 stream_realip_module 模块是否启用
        proxy_protocol on;
    }
    ...
}
```

{% hint style="info" %}
开启 proxy\_protocol 前需要通过 `nginx -V 2>&1 | grep -- 'stream_realip_module'` 命令检查 stream\_realip\_module 模块是否启用
{% endhint %}

例子：配置7层http代理并开启客户端IP透传

```text
upstream gateway {
    server 10.14.31.161:443;
    server 10.14.31.162:443;
    server 10.14.31.163:443;
}

http {
    server {
        # 启用 proxy_protocol
        listen       443 ssl proxy_protocol;
        
        # set_real_ip_from 设置下游负载均衡器或代理的CIDR
        set_real_ip_from 0.0.0.0/0;
        
        # 从 proxy_protocol 中获取客户端原始地址，做为 $remote_addr 参数
        real_ip_header proxy_protocol;
        
        # 设置客户端请求头部的 Host 用于域名校验
        proxy_set_header Host      $host;
        
        # 启用 proxy_protocol 后，可通过 $proxy_protocol_addr 参数获取客户端真实地址
        proxy_set_header X-Real-IP $proxy_protocol_addr;
        proxy_set_header X-Forwarded-For $proxy_protocol_addr;
      
        location /api/v1/ {
             proxy_pass http://gateway/;
        }
    }
}
```

{% hint style="info" %}
开启 proxy\_protocol 前需要通过 `nginx -V 2>&1 | grep -- 'http_realip_module'` 命令检查 http\_realip\_module 模块是否启用
{% endhint %}

