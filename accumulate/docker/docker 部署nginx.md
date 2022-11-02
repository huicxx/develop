```shell
docker run --name nginx82 -p 82:80 -d -v D:\docker_repo\nginx2\conf\nginx.conf:/etc/nginx/nginx.conf -v D:\docker_repo\nginx2\html:/usr/share/nginx/html -v D:\docker_repo\nginx2\conf\conf.d:/home/nginx/conf/conf.d -v D:\docker_repo\nginx2\ngx_healthcheck_module:/opt/ngx_healthcheck_module -v D:\docker_repo\nginx2\nginx:/opt/nginx nginx:1.19.10-alpine
```