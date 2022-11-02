**使用的是hametan/datax-web:2.1.2镜像**

注：

1、需要先创建datax-admin.log文件， 访问地址及数据库导入参考 datax-web的官方2.1.2版本

2、本镜像不带mysql数据库，需要的自己另拉镜像或连接已有mysql

3、PGSQL支持update，必须设置参数 "writeMode": "update (主键ID)",不设置有可能会导致PGSQL同步失败。

4、使用update时目标标要有主键（PGSQL我测试是这样，其它数据库未做测试，不然会有脏数据问题。）

5、本镜像已经将容器时区改为了ShangHai

dockerfile 来源 [https://github.com/chaozhangpower/datax-web-docker](https://github.com/chaozhangpower/datax-web-docker)

pgsql更新支持修改基于教程：[https://blog.csdn.net/u013308496/article/details/106876691/](https://blog.csdn.net/u013308496/article/details/106876691/)

#### 多executer部署

##### 部署admin

--javascripttypescriptbashsqljsonhtmlcssccppjavarubypythongorustmarkdown

```shell
docker run -d --name datax-admin --restart=always -p 2020:2020  -v /data/datax-admin.log:/tmp/datax-admin.log hametan/datax-web:2.1.2 java -jar datax-admin-2.1.2.jar --PORT=2020 --MYSQL_SERVICE_HOST=xxxx --MYSQL_SERVICE_PORT=3306 --MYSQL_USER=xxx --MYSQL_PASSWORD=xxx --DB_DATABASE=xxx
```

- PORT web访问端口
- MYSQL_SERVICE_HOST mysql的ip
- MYSQL_SERVICE_PORT mysql端口
- MYSQL_USER mysql 用户名
- MYSQL_PASSWORD mysql用户的密码
- DB_DATABASE 对应的数据库

##### 部署executor

1、修改executor所在服务器的ip，由于在docker上运行，程序默认获取的是docker中的ip，与真正的ip不一致 。

``` yaml
# web port
server:
  port: ${PORT:}

# log config
logging:
  config: classpath:logback.xml
  path: ${data.path:/home/}/applogs/executor/jobhandler

datax:
  job:
    admin:
      ### datax admin address list, such as "http://address" or "http://address01,http://address02"
#      addresses: http://172.10.0.2:2020
      addresses: ${ADDRESSES:}
    executor:
      appname: datax-executor
      #修改成服务器的ip
      ip:
      port: ${executor.port:9999}
      ### job log path
      logpath: ${data.path:/home}/applogs/executor/jobhandler
      ### job log retention days
      logretentiondays: 30
    ### job, access token
    accessToken:

  executor:
    jsonpath: ${json.path:/opt/datax-web-2.1.2/modules/datax-executor/json}

  pypath: ${python.path:/opt/datax/bin/datax.py}
```

- 修改配置项 datax.job.executor.ip ，值改成真正的服务器ip

docker 启动命令：

```
docker run -d --name datax-executor --restart=always -p 2021:2021 -p 9999:9999 -v D:\docker_repo\datax_web\application.yml:/opt/application.yml -v D:\docker_repo\datax_web\executorlogs\:/home/applogs/executor/jobhandler/ hametan/datax-web:2.1.2 java -jar datax-executor-2.1.2.jar --PORT=2021 --ADDRESSES=http://192.168.100.88:2020
```

-   PORT web端口
- ADDRESSES admin的访问地址


### 单机部署

**使用的是hametan/datax-web:2.1.2镜像**

```shell

docker network create --driver bridge --subnet 172.20.1.0/16 my-network

docker run -d --name datax-admin --restart=always -p 2020:2020 --net my-network --ip 172.20.1.61 -v /data/datax-admin.log:/tmp/datax-admin.log hametan/datax-web:2.1.2 java -jar datax-admin-2.1.2.jar --PORT=2020 --MYSQL_SERVICE_HOST=xxxx --MYSQL_SERVICE_PORT=3306 --MYSQL_USER=xxx --MYSQL_PASSWORD=xxx --DB_DATABASE=xxx

docker run -d --name datax-executor --restart=always -p 2021:2021 --net my-network --ip 172.20.1.62 -v /data/executorlogs/:/home/applogs/executor/jobhandler/ hametan/datax-web:2.1.2 java -jar datax-executor-2.1.2.jar --PORT=2021 --ADDRESSES=http://172.20.1.61:2020
```