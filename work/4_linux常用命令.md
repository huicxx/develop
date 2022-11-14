
```shell
## ------------端口和进程------------
# 查看端口占用
netstat -tunlp |grep 端口号
lsof -i:端口号

echo $(netstat -nlp | grep :8081 | awk '{print $7}' | awk -F "/" '{ print $1 }')

# 查看线程数
ps -Lf 21496 |wc -l

## ------------文件大小------------
du -h --max-depth=1 /home/ljdw/MinIO
# 查看文件挂载
df -h
# 查看inode占用
df -ih


## ------------过滤文件内容------------
grep -A 1000 "[INFO]2022-08-16 00:01:54"
grep  -A 10 'Ice.TimeoutException' log.log
tail -f log | grep xxx | grep yyy

tail -f xxx.log             ----实时刷新最新日志
tail -100f xxx.log      --------实时刷新最新的100行日志
tail -100f xxx.log | grep [关键字]     -------查找最新的一百行中与关键字匹配的行
tail -100f xxx.log | grep '2019-10-29 16:4[0-9]'    ------查找最新的100行中时间范围在2019-10-29 16:40-2019-10-29 16:49范围中的行
tail -1000f xxx.log | grep -A 5 [关键字] ----------查看最新的1000行中与关键字匹配的行加上匹配行后的5行

sed -n '/2022-09-01 15:28/,/2022-09-01 16:00/p' log.log > nohub.out

## ------------压缩和解压------------
tar -czf xxx.tar.gz *.jpg
tar -xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz

## ------------文件操作----------
find / -name "jdk1.8.0_202.tar.gz" 


#远程拷贝
scp -r  -P 18586 /home/ljdw/openresty/nginx/html/ DeployUser@172.31.2.190:/home/ljdw/openresty/nginx/

## ------------防火墙------------
systemctl start firewalld
systemctl stop firewalld
systemctl status firewalld
#添加指定需要开放的端口：
firewall-cmd --add-port=9998/tcp --permanent
#重载入添加的端口：
za36987
#查询指定端口是否开启成功：
firewall-cmd --query-port=6379/tcp

#查看当前系统打开的所有端口

firewall-cmd --zone=public --list-ports

```
