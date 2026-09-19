本章基于 Linux 系统 Ubuntu 22.04 安装，docker 版本 version 29.6.2

## 1、基本安装

上传并解压tgz包至 /software/ 目录下

![image-20260827103532051](install.assets/image-20260827103532051.png)

```bash
# 将二进制文件移动到可执行路径下的目录中，例如 /usr/bin/
sudo cp docker/* /usr/bin/

# 启动 docker 守护进程
sudo dockerd &

# 运行测试
docker run hello-world
```

## 2、加入到标准服务管理systemd

```bash
# 创建docker systemd 单元服务文件
touch /etc/systemd/system/docker.service
vim /etc/systemd/system/docker.service

# 填入以下内容
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# 关键：这里的路径要和你放置 dockerd 的路径一致，例如 /usr/bin/dockerd
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

让 systemd 识别并启用服务

```bash
# 重新加载所有服务单元，让 systemd 识别 docker.service
sudo systemctl daemon-reload

# 启用服务，这会在系统开机时自动启动 Docker
sudo systemctl enable docker

# 立即启动 Docker 服务
sudo systemctl start docker

# 验证服务状态
sudo systemctl status docker
```

正常会显示如下内容

```bash
● docker.service - Docker Application Container Engine
     Loaded: loaded (/etc/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2026-08-27 18:44:31 CST; 7s ago
       Docs: https://docs.docker.com
   Main PID: 1902 (dockerd)
      Tasks: 23 (limit: 4540)
     Memory: 119.2M
        CPU: 1.306s
     CGroup: /system.slice/docker.service
             ├─1902 /usr/bin/dockerd
             └─1911 containerd --config /var/run/docker/containerd/containerd.toml
...
...
...
```

## 3、配置国内镜像源

创建并编辑Docker引擎配置文件

```bash
vim /etc/docker/daemon.json
```

填入如下内容。

```bahs
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me"
  ]
}
```

重新启动 docker 服务。

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

验证配置是否生效，运行以下命令，检查输出信息中是否包含你刚刚配置的镜像源地址。

```bash
docker info | grep "Registry Mirrors" -A 5
```

输出以下内容表示配置成功

```txt
...
...
 Registry Mirrors:
  https://docker.m.daocloud.io/
  https://docker.1ms.run/
  https://docker.xuanyuan.me/
 Live Restore Enabled: false
 Product License: Community Engine
...
...
```

