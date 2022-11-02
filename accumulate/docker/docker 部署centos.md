```shell
docker run -d --name centos7 -p 8010:80 -v D:\docker_repo\nginx\ngx_healthcheck_module:/opt/ngx_healthcheck_module -v D:\docker_repo\nginx\nginx:/opt/nginx centos:centos7 /bin/bash -c "while true; do echo hello world; sleep 1; done"
```