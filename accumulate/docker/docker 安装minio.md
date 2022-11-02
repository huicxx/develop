旧版的镜像：


```
docker pull minio/minio:RELEASE.2021-06-17T00-10-46Z
```


```shell
docker run -p 9000:9000 --name minio -d --restart=always \
  -e "MINIO_ROOT_USER=minio" \
  -e "MINIO_ROOT_PASSWORD=minio123456" \
  -v /usr/local/minio/data:/data \
  -v /usr/local/minio/config:/root/.minio \
  minio/minio:RELEASE.2021-06-17T00-10-46Z server /data
```

页面：[http://localhost:9090/login](http://localhost:9090/login)