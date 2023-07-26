---
title: 通过VSCode调试docker容器里的程序（多种方法）
date: 2023-07-19 16:34:37
tags:
categories:
- docker
- vscode
---

# 背景
原来总是通过pycharm连接服务器跑代码测试，太过臃肿，索性使用宇宙第一编辑器vscode来跑测试，一是方便查看修改文件，二是启动速度快。

# 通过vscode的remote-container插件

只需要在应用商店搜索该插件即可，方法很简单，与remote-ssh插件基本一样。
在主机上面运行过docker之后(docker run ...，可以用--name指定名词)，remote-container插件即可检索到这个容器，右键容器，再找到对应的选项，即可连接进去容器内部。



# (常用)通过ssh远程调试容器内部(支持额外安装的docker)

基于原来的镜像，然后再加一层，该层主要是安装gdb、openssh-server，然后开启sshd服务，暴露出来22端口。

步骤：

1. 拉取镜像并创建容器

   ```shell
   $ docker pull paddlecloud/paddlenlp:develop-gpu-cuda11.2-cudnn8-latest
   
   # 将卡加入容器中，并打开ssh端口
   $ docker run --name alon_paddle --runtime=nvidia -v /home/alon/workspace/paddle:/mnt -p 8888:8888 -p 1028:22 -it paddlecloud/paddlenlp:develop-gpu-cuda11.2-cudnn8-latest /bin/bash
   ```

2. 在容器中安装ssh并设置root密码

   ```shell
   # 安装openssh: 
   $ apt-get update；apt-get install openssh-server
   
   # 设置sshroot登录：
   $ echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
   
   # 重启ssh服务：
   $ service ssh restart
   
   # 设置root密码：
   $ passwd root
   ```

3. 通过vscode链接docker容器环境

   ```yaml
   Host docker_paddle
       HostName ***.*.**.***
       User root
       Port 1028
   ```

   **remote-ssh连接的时候是root及其密码（2中设置的root密码）登录**

4. (选)将容器保存为新的镜像并创建新的容器

   ```shell
   # docker commit [OPTIONS] [当前容器ID] [新的镜像名称:新的镜像标签]
   
   # [OPTIONS]说明：
   
   # [-a]：提交的镜像作者。
   
   # [-m]：提交时的说明文字。
   
   # [-p]：在commit时，将容器暂停。
   
   $ docker commit -a "name" -m "新增了openssh" ab7da476849b paddlecloud/paddlenlp:develop-gpu-cuda11.2-cudnn8-latest
   
   # 创建新容器：
   $ docker run --name alon_paddle --runtime=nvidia -v /home/alon/workspace/paddle:/mnt -p 8888:8888 -p 1028:22 -it paddlecloud/paddlenlp:develop-gpu-cuda11.2-cudnn8-latest /bin/bash
   
   3）启动ssh服务
   $ /etc/init.d/ssh restart 或 service ssh restart
   ```

   

# 通过vscode的pipeTransport功能(略)

配置launch.json

``` json
"pipeTransport": {
    "pipeCwd": "${workspaceFolder}",
    "pipeProgram": "docker",
    "pipeArgs": [
        "exec",
        "-i",
        "hello",
        "sh",
        "-c"
    ],
    "debuggerPath": "/path/to/pythonDebug"
},
```

