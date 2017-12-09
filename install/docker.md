
# 拉取镜像

```
docker pull rabbitmq:management
```
也可以使用国内地址，速度更快
```
docker pull daocloud.io/library/rabbitmq:3.5.3-management

# 重命名
docker tag <IMAGE ID> rabbitmq:management
# 删除命名之前
docker rmi daocloud.io/library/rabbitmq:3.5.3-management
```

# 启动

```
# 后台运行(-d)
# 暴露 5672(amqp Protocol) 和 15672(RabbitMQ Management) 两个端口
# 容器名是 rabbit (--name rabbit)
# 环境变量设置用户名密码
docker run -d -p 5672:5672 -p 15672:15672 --name rabbit -e RABBITMQ_DEFAULT_USER=kail  -e RABBITMQ_DEFAULT_PASS=1234 rabbitmq:management
```

# 访问 Web UI

地址 ： http://localhost:15672/


# 其他

- [rabbitmq dockerfile lib](https://github.com/docker-library/rabbitmq/)
- https://hub.docker.com/_/rabbitmq/