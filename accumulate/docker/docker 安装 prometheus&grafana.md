### server

```shell
docker run -d --name prometheus --restart=always -p 19090:9090 -v /home/ljdw/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /home/ljdw/prometheus/data:/prometheus/data -v /home/ljdw/prometheus/rule:/etc/prometheus/rule prom/prometheus
```

### alertmanager

```shell
docker run -d --name alertmanager --restart=always -p 19093:9093 -v /home/ljdw/prometheus/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml prom/alertmanager
```

### exporter

```shell
docker run -d --name node-exporter --restart=always -p 19100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" prom/node-exporter

docker run -d --name redis_exporter --restart=always -p 19101:9121 oliver006/redis_exporter --redis.addr=172.0.0.1:6379 --redis.password=za36987

docker run -d --name rabbitmq-export --restart=always -v /home/ljdw/prometheus/rabbit-export:/etc/rabbitmq -p 19102:9419 kbudde/rabbitmq-exporter -config-file /etc/rabbitmq/config.json

docker run -d --name prometheus-dingtalk --restart=always -v /home/ljdw/prometheus/dingtalk/config.yml:/etc/prometheus-webhook-dingtalk/config.yml -v /home/ljdw/prometheus/dingtalk/template.tmpl:/etc/prometheus-webhook-dingtalk/template.tmpl -p 19094:8060 timonwong/prometheus-webhook-dingtalk
```

# grafana

```shell
docker run -d --name=grafana --restart=always -p 13000:3000 grafana/grafana
```

