```shell
docker run -d --restart=always --hostname my-rabbit --name rabbit -e RABBITMQ_NODENAME=my-rabbit -p 15672:15672 -p 5673:5672 rabbitmq
```

页面管理需要进入镜像中开启，开启命令： rabbitmq-plugins enable rabbitmq_management

管理页面的用户名和密码都是guest