# docker-compose-php

#### 介绍
Docker化PHP开发环境

该环境集成了公司PHP研发的常用组件，包含：
- Nginx 1.24
- PHP 8.1 `包含 Swoole、Redis、Mongo、AMQP、Soap 等扩展及 Git、composer、vim 等常用工具`
- Mysql 8.0
- MongoDB 6.0
- Elasticsearch 8.7
- RabbitMQ 3.11
- Redis 7.0
- Consul

#### 目录结构
- app `程序代码目录，映射php容器中的/data/www`
- data `数据库等持久化文件保存目录，删除容器不丢数据`
- logs `日志文件映射目录`
- services `docker-compose编排文件、配置文件等`


#### 安装教程
1. 修改宿主机docker配置文件`daemon.json`(修改了默认网段、国内源、DNS服务器)
```JSON
{
  "bip": "172.16.10.1/24",
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "debug": false,
  "default-address-pools": [
    {
      "base": "10.10.0.0/16",
      "size": 24
    }
  ],
  "dns": [
    "114.114.114.114"
  ],
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "insecure-registries": [],
  "registry-mirrors": [
    "https://1v2h7h6m.mirror.aliyuncs.com",
    "https://registry.docker-cn.com"
  ]
}
```

2. 命令行进入`./services`目录，执行 `docker-compose build`，成功后执行`docker-compose up -d`即可

#### 其他问题
##### ES无法启动
需要对vm.max_map_count进行配置，解放内存限制
Docker Desktop for Mac
```bash
docker-machine ssh
sudo sysctl -w vm.max_map_count=262144
```

Docker Desktop for Windows WSL2-backend
```shell
wsl -d docker-desktop
sysctl -w vm.max_map_count=262144
```

Linux
```bash
# 修改配置文件
grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=262144
# 启动配置
sysctl -w vm.max_map_count=262144
```

##### ARM 处理器中 Docker 无法拉取 Mysql 镜像
将 `./mysql/Dockerfile` 的基础镜像更换成 `mysql/mysql-server`