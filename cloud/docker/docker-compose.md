# 一、概念及安装

## 1，用途？

Docker Compose 是一款用于定义和运行多容器应用的工具。 它是解锁高效开发与部署体验的关键。Compose简化了对整个应用栈的控制，使得在单一YAML配置文件中轻松管理服务、网络和卷。然后，只需一个命令，你就能创建并启动所有服务 从你的配置文件里。Compose 工作涵盖所有环境——生产、预发布、开发、测试等 以及CI工作流程。它还包含管理应用整个生命周期的命令：

- 启动、停止和重建服务
- 查看运行服务状态
- 流式传输运行服务的日志输出
- 在服务上运行一次性命令

## 2，compose工作原理

使用 Docker Compose 时，你使用一个称为 [Compose 文件](https://docs.docker.com/compose/intro/compose-application-model/#the-compose-file)的 YAML 配置文件来配置应用的服务，然后用 [Compose CLI](https://docs.docker.com/compose/intro/compose-application-model/#cli) 创建并启动所有服务。Compose文件的默认路径是（优先）或放置在工作目录中的路径。 Compose 还支持并向后兼容早期版本。 如果两个文件都存在，Compose 更倾向于使用规范文件。`compose.yaml``compose.yml``docker-compose.yaml``docker-compose.yml``compose.yaml`、

## 3，docker CLI

Docker CLI 允许您通过命令及其子命令与 Docker Compose 应用交互。如果你用的是 Docker 桌面，默认包含 Docker Compose CLI。

关键命令：

```bash
# 启动文件中定义的所有服务
docker compose up
# 停止并移除正在运行的服务
docker compose down
# 如果你想监控运行容器的输出和调试问题，可以通过以下方式查看日志：
docker compose logs
# 列出所有服务及其当前状态：
docker compose ps
```

## 4，安装在Linux中

手动安装Plugin插件。

```bash
$ DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}

$ mkdir -p $DOCKER_CONFIG/cli-plugins

$ curl -SL https://github.com/docker/compose/releases/download/v5.5.0/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
# 对二进制文件应用可执行权限
$ chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose

# 测试安装
$ docker compose version

# 在用户的家目录下
root@dev:~# tree .docker/
.docker/
└── cli-plugins
    └── docker-compose
```

# 二、快速入门

项目结构

./
├── app.py
├── compose.yaml
├── Dockerfile
└── requirements.txt

## 1，搭建项目

我们本次的工作目录：/root/testDir/compose-demo，在这个目录下创建并编辑文件 app.py

```bash
## 粘入以下内容
import os
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(
    host=os.getenv("REDIS_HOST", "redis"),
    port=int(os.getenv("REDIS_PORT", "6379")),
)

@app.route("/")
def hello():
    count = cache.incr("hits")
    return f"Hello from Docker! I have been seen {count} time(s).\n"
```

应用通过环境变量读取 Redis 连接信息，默认设置合理，开箱即用。

创建并编辑文件 `requirements.txt`

```bash
flask
redis
```

创建并编辑文件：`Dockerfile`

```bash
# syntax=docker/dockerfile:1

# Build an image with the Python 3.12 image
FROM python:3.12-alpine

# Set the working directory to `/code`
WORKDIR /code

# Set environment variables used by the `flask` command
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0

# Install `gcc` and other dependencies
RUN apk add --no-cache gcc musl-dev linux-headers

# Copy `requirements.txt`
COPY requirements.txt .

# Install the Python dependencies
RUN pip install -r requirements.txt

# Copy the current directory `.` in the project to the workdir `.` in the image
COPY . .

EXPOSE 5000

# Set the default command for the container to `flask run --debug`
CMD ["flask", "run", "--debug"]
```

创建并编辑文件：.env

```bash
# 主机端口映射容器端口 5000，配置于 compose.yaml 文件中
APP_PORT=8000
REDIS_HOST=redis
REDIS_PORT=6379
#######################################
Docker 在构建映像时会把项目目录里的所有内容都发送到守护进程。 没有 ，这包括你的文件（可能包含秘密）和 任何缓存的Python字节码。排除它们能让构建速度快，避免无意中避免 将敏感值烘焙到图像图层中。
```

在不编辑YAML的情况下更改环境间的数值
避免将密钥提交到版本控制
跨多项服务的重用值

创建并编辑文件：.dockerignore

```bash
.env
*.pyc
__pycache__
redis-data
```

## 2，定义并启动您的服务

在您的项目目录中进行

```bash
vim compose.yaml

## 粘贴以下内容 ##
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
      
  redis:
    image: redis:alpine
```

该 compose 文件定义了两种服务：

1）该服务使用由当前目录中构建的镜像。他会将主机上的端口映射到Flash默认监听的容器端口。web Dockerfile 8000 5000

2）该服务使用 Docker Hub 注册表拉取的公开 Redis 映像。redis。有关该文件的更多信息请参考：https://docs.docker.com/compose/intro/compose-application-model/

我们本次实验当下载alpine镜像时发生了DNS无法解析的错误，后通过如下方式解决：

```bash
# 编辑 Docker Engine 配置文件
vim /etc/docker/daemon.json

# 添加一段DNS解析服务器（最终文件内容如下）
{
  "dns": [
    "8.8.8.8",
    "114.114.114.114"
  ],
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me"
  ]
}

# 重新启动 docker 服务
systemctl restart docker

# 验证容器中的 DNS
docker run --rm alpine cat /etc/resolv.conf
```

启动您的应用：

```bash
docker compose up [--build]

## 选项 --build 的说明
不加 --build：	直接启动现有容器。如果镜像不存在，则自动构建；如果镜像已存在，则直接使用，无论代码是否更改。

加上 --build：先强制重新构建所有服务的镜像（忽略已有缓存），然后再启动容器。确保运行的容器使用的是最新的代码和依赖。
```

只需一个命令，你就能从配置文件创建并启动所有服务。Compose 构建你的网页镜像，拉取 Redis 镜像，启动两个容器。

![image-20260829164250992](docker-compose.assets/image-20260829164250992.png)

我们访问虚拟机 8000 端口

![image-20260829171503201](docker-compose.assets/image-20260829171503201.png)

# 三、基础进阶

在继续之前先停止您的应用

```bash
docker compose down
```

## 1，通过健康检查解决启动竞态问题

更新：`compose.yaml`

```bash
services:
	web:
		build:
		ports:
			- "${APP_PORT}:5000"
		environment:
			- REDIS_HOST=${REDIS_HOST}
			- REDIS_HOST=${REDIS_HOST}
		depends_on:
			redis:
				condition: service_healthy
			
	redis:
		image: redis:alpine
		healthcheck:
			test: ["CMD","redis-cli","ping"]
			interval: 5s
			timeout: 3s
			retries: 5
			start_period: 10s
```

![image-20260901114030062](docker-compose.assets/image-20260901114030062.png)

```bash
...
...
Container compose-demo-redis-1 Healthy
...
...
```

## 2，启动 Compose Watch以获取实时更新

没有 Compose Watch，每次代码更改都需要停止堆栈、重建映像和重启容器。Compose Watch 通过自动同步修改文件到运行中的容器，消除了这种循环。

更新 compose.yaml文件

```bash
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
    depends_on:
      redis:
        condition: service_healthy
    develop:
      watch:
        - action: sync+restart
          path: .
          #这个路径路径同步于Dockerfile文件中的 WORKDIR
          target: /code
        - action: rebuild
          path: requirements.txt

  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s
```

再开一个窗口修改app.py的内容

```bash
......
return f"Hello from Compose Watch! I have been seen {count} time(s).\n"
...
```

直接刷新网页发现资源已自动更新，无需在重新 up 一下。

## 3，持久化带有命名卷的数据

每次你停止并重新开始堆栈，访问计数器都会重置为零。Redis 数据 它生活在容器内，因此当容器被移除时它会消失。一个有名的 Volume 通过将数据存储在主机上，超出容器生命周期来解决这个问题。

更新 `compose.yaml`

```yaml
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
    depends_on:
      redis:
        condition: service_healthy
    develop:
      watch:
        - action: sync+restart
          path: .
          target: /code
        - action: rebuild
          path: requirements.txt

  redis:
    image: redis:alpine
    volumes:
    # redis-data 一份由 docker 管理的 命名卷，/data Redis容器里的数据目录Redis 默认会把持久化文件写到这里，比如 dump.rdb。
    # redis-data:/data：把命名卷挂到 Redis 容器的 /data。
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s
# 顶层 volumes: redis-data:：向 Compose 注册这个命名卷。不存在就创建，存在就复用。
volumes:
  redis-data:
```

所以容器删了没关系，数据在 `redis-data` 这个卷里。下次再启动 Redis，还挂同一个卷，数据就回来了。

实际卷名命名规范通常是：`项目名_redis-data`

可以用如下命令查看：

```bash
root@dev:~/testDir/compose-demo# docker volume ls
DRIVER    VOLUME NAME
local     compose-demo_redis-data
```

```bash
docker compose up --watch
```

连续刷新10次页面。

![image-20260919220156654](docker-compose.assets/image-20260919220156654.png)

```bash
# 移除容器，但不删除命名卷
docker compose down
```

停用后重新启动可以发现 redis 缓存的数据没有丢失。

```bash
docker compose down -v
# v 是 volumes（卷）的缩写
```

额外删除 Compose 文件里定义的命名卷和匿名卷，下次再启动，会创建一个新的空卷，计数器归零。

再次启动计数器归1

![image-20260919222245172](docker-compose.assets/image-20260919222245172.png)

