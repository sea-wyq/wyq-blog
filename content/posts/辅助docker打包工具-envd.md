---
title: "辅助构建docker镜像工具-envd"
date: 2022-07-21
categories:
- docker
tags:
- envd
thumbnailImagePosition: right
thumbnailImage: img/main/th-14.jpeg
---

特性：
- 通过编写python代码来构建docekr和开发环境的设置。
- 内置jupyter/Vscode。


<!--more-->

{{< toc >}}


# 安装要求

- Docker (20.10.0 或以上)
```shell
# 在Ubuntu上安装Docker
$ sudo apt-get update

$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

$ sudo mkdir -p /etc/apt/keyrings

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt-get update

$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

# 安装
```SHELL
$ pip install --pre --upgrade envd
$ envd bootstrap
```
> 您可以在运行时添加--dockerhub-mirror或-m标记envd bootstrap，为 docker.io 注册表配置镜像：  
> envd bootstrap --dockerhub-mirror https://docker.mirrors.sjtug.sjtu.edu.cn

# 编写build.envd文件

```python
def build():
    mirror_config()
    base(language="python", image="artifacts.iflytek.com/docker-private/atp/aiges-gpu:10.1-3.9.13-ubuntu1804-v1.0.0")   #加载基础镜像
    install.python_packages(name = [      # 基础镜像需要提供pip
        "torch==1.10",
        "torchvision",
        "openmim",
    ])
    install.system_packages(name = [
        "libgl1-mesa-glx"                 
    ])
    run(commands=[
    "mim install mmcv-full",            # 通过第三方工具来进行安装
    "mim install mmdet==2.24.0",
    ])


def mirror_config():
    config.apt_source(source="""
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
    """)
    config.pip_index(url = "https://pypi.tuna.tsinghua.edu.cn/simple")
```

# 构建环境镜像
``` shell
$ envd build                         # 当前路径构建
$ envd build --path examples/mnist   # 指定路径构建
```

# 构建容器
```shell
$ docker run -ti --entrypoint bash 镜像名
```