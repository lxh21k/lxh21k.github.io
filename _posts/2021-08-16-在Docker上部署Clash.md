---
title: �在Docker上部署Clash
date: 2021-08-16 16:00
---

## 优点： 

- 后台运行
- 开机自启
- 无需下载可执行文件了，只需要config.yaml

## 部署方法：

[clash as a daemon · Dreamacro/clash Wiki](https://github.com/Dreamacro/clash/wiki/clash-as-a-daemon#docker)

### 1. 安装Docker和Docker compose

[Install Docker Engine](https://docs.docker.com/engine/install/)

安装完成后启动docker

```bash
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

### 2. 创建目录保存 `docker-compose.yaml` 和 `config.yaml`

docker-compose.yaml 用来配置 docker compose

```json
version: '3'
services:
  clash:
    # ghcr.io/dreamacro/clash
    # ghcr.io/dreamacro/clash-premium
    # dreamacro/clash
    # dreamacro/clash-premium
    image: dreamacro/clash-premium:latest
    container_name: clash
    volumes:
      - ./config.yaml:/root/.config/clash/config.yaml
      # - ./ui:/ui # dashboard volume
    #ports:
      # - "7890:7890"
      # - "7891:7891"
      # - "8080:8080" # external controller (Restful API)
    restart: unless-stopped
    network_mode: host # or "host" on Linux
```

由于机场提供的 `config.yaml` 包含clash-premium才支持的规则，因此 `image` 设为 `dreamacro/clash-premium` 仓库。

在Linux系统下 `network_mode` 要设成 `host` ，此时需要注释掉所有 `ports` ，否则docker会报错：

```bash
ERROR: for clash  "host" network_mode is incompatible with port_bindings
```

下载好 `config.yaml` 后即可在此目录下执行：

```bash
$ sudo docker-compose up -d #启动clash
```

也可以通过这条命令查看log：

```bash
$ sudo docker-compose logs
```

终止Clash：

```bash
$ sudo docker-compose stop
```

Docker的一些基本命令：

- `sudo docker images` 查看所有下载的镜像
- `sudo docker ps` 查看当前运行的所有容器

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99752638-eed8-4cb6-99f4-7df9d5dd565a/Untitled.png](https://www.notion.so/Docker-clash-492887f7a5924757b887857b38a7716f#50b2bdcf722a46ee8da743af0ce2ae82)

### 3. 更新 `config.yaml`

见 [Manjaro Installation Log
](https://lxh21k.github.io/2020/12/Manjaro-Installation-Log).