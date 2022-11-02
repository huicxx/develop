```shell
docker run -d -p 9000:9000 -p 9090:9090 --name minio --restart=always -e "MINIO_ACCESS_KEY=minioadmin" -e "MINIO_SECRET_KEY=minioadmin" -v D:/docker_repo/minio:/data minio/minio server /data --console-address ":9090" -address ":9000"
```

页面：[http://localhost:9090/login](http://localhost:9090/login)