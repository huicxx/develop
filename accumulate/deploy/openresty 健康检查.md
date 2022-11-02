## lua-resty-upstream-healthcheck说明

health_check运行在init_worker阶段，也是一个定时任务。

Openresty在1.9.3.2版本或者以上的版本已经集成了lua-resty-upstream-healthcheck模块

## 使用方式

1、安装Openresty ，略。

2、修改nginx.conf文件

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    lua_package_path "/path/to/lua-resty-upstream-healthcheck/lib/?.lua;;";

# sample upstream block:
    upstream tomcat_upstream {
        server 127.0.0.1:81;
        server 127.0.0.1:82;
    }

upstream nginx_upstream {
        server 127.0.0.1:83;
        server 127.0.0.1:84;
    }

## 指定共享内存
lua_shared_dict healthcheck 1m;

## 在worker初始化过程中，启动定时器，进行后端结点的检查
init_worker_by_lua_block {
   local hc = require "resty.upstream.healthcheck"
   local ok, err = hc.spawn_checker {
   -- shm 表示共享内存区的名称，
   shm = "healthcheck",
   -- type 表示健康检查的类型， HTTP or TCP （目前只支持http）
   type = "http",    
   -- upstream 指定 upstream 配置的名称   
   upstream = "nginx_upstream",
   -- 如果是http类型，指定健康检查发送的请求的URL
   http_req = "GET /index.html HTTP/1.0\r\nHost: tomcat\r\n\r\n",
   -- 请求间隔时间，默认是 1 秒。最小值为 2毫秒
   interval = 3000,
   -- 请求的超时时间。 默认值为：1000 毫秒
   timeout = 5000,
   -- 失败多少次后，将节点标记为down。 默认值为 5
   fall = 3, 
   -- 成功多少次后，将节点标记为up。默认值为 2
   rise = 2,
   -- 返回的http状态码，表示应用正常
   valid_statuses = {200, 302},
   -- 并发度， 默认值为 1
   concurrency = 2,
   }
   
   local ok, err = hc.spawn_checker {
   -- shm 表示共享内存区的名称，
   shm = "healthcheck",
   -- type 表示健康检查的类型， HTTP or TCP （目前只支持http）
   type = "http",    
   -- upstream 指定 upstream 配置的名称   
   upstream = "tomcat_upstream",
   -- 如果是http类型，指定健康检查发送的请求的URL
   http_req = "GET /index.html HTTP/1.0\r\nHost: tomcat\r\n\r\n",
   -- 请求间隔时间，默认是 1 秒。最小值为 2毫秒
   interval = 3000,
   -- 请求的超时时间。 默认值为：1000 毫秒
   timeout = 5000,
   -- 失败多少次后，将节点标记为down。 默认值为 5
   fall = 3, 
   -- 成功多少次后，将节点标记为up。默认值为 2
   rise = 2,
   -- 返回的http状态码，表示应用正常
   valid_statuses = {200, 302},
   -- 并发度， 默认值为 1
   concurrency = 2,
   }
 
   if not ok then
   ngx.log(ngx.ERR, "=======> failed to spawn health checker: ", err)
   return
   end
}

server {
        
listen       80;
        server_name  localhost;

location / {
            proxy_pass   http://tomcat_upstream;
        }

location /nginx_upstream {
            proxy_pass   http://nginx_upstream;
        }

        # status page for all the peers:
        location = /status {
            access_log off;
            allow 127.0.0.1;
            deny all;

            default_type text/plain;
            content_by_lua_block {
                local hc = require "resty.upstream.healthcheck"
                ngx.say("Nginx Worker PID: ", ngx.worker.pid())
                ngx.print(hc.status_page())
            }
        }

# status page for all the peers (prometheus format):
        location = /metrics {
            access_log off;
            default_type text/plain;
            content_by_lua_block {
                local hc = require "resty.upstream.healthcheck"
                st , err = hc.prometheus_status_page()
                if not st then
                    ngx.say(err)
                    return
                end
                ngx.print(st)
            }
        }
    } 
server {
        
listen       8080;
        server_name  localhost;

location / {
            proxy_pass   http://nginx_upstream;
        }
 
}
}


```

需要说明：目前代理了两个web页面，需要设置两个hc.spawn_checker方法，并且在方法内部upstream中对应指定的upstream配置，其他参数按实际需要配置使用。

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101202742.png)

3、测试，启动openresty 和4个web应用

openresty查看负载均衡服务状态，配置在80端口中，路径为：http://localhost/status
![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101202848.png)

个人测试使用docker启动了4个nginx，模仿4个web服务。

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101202901.png)

访问页面如下：
![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101202917.png)

openresty 中80代理的是 tomcat_upstream 对应的是81和82的web应用；8080 代理的是nginx_upstream， 对应的是83和84的web应用；
访问 http://localhost/ 

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101202937.png)

访问 http://localhost:8080/ 

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101202943.png)

情况1，暂停81和83的web应用，查看状态

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101203011.png)
![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101203017.png)

情况2，暂停82和83的web应用，查看状态

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101203031.png)

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101203039.png)

