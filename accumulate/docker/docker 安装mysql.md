docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7

##主从

[https://blog.csdn.net/xizhen2791/article/details/123660049](https://blog.csdn.net/xizhen2791/article/details/123660049)

docker run -p 3307:3306 --name mysql-master -v D:\docker\mysql\master\log:/var/log/mysql -v D:\docker\mysql\master\data:/var/lib/mysql -v D:\docker\mysql\master\conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

docker run -p 3308:3306 --name mysql-slave -v D:\docker\mysql\slave\log:/var/log/mysql -v D:\docker\mysql\slave\data:/var/lib/mysql -v D:\docker\mysql\slave\conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

windows 文件映射需要授权

chmod 644 /etc/mysql/my.cnf




