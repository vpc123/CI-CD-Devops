## drone1.0完全新的解析

     image: docker:dind
     volumes:
       - /etc/docker/daemon.json:/etc/docker/daemon.json
     privileged: true
     command: --storage-driver=overlay2


参数解析说明：

    volumes:
      - /etc/docker/daemon.json:/etc/docker/daemon.json

将自己的加速器放到容器中进行镜像下载加速

    privileged: true   #使得docker容器具有完全root权限

指定存储驱动的类型

    command: --storage-driver=overlay2


#### 我们在拉取镜像时做一个分析


我们部署的drone集群镜像时采用的方式，是启动gogs，drone-server，和drone-agent还有一个docker：bind容器进行容器的容器中镜像下载和启动我们本地的镜像不是一个概念，还有针对目前国内访问dockerHub那种神奇的网络速度，何止迷人。所以尽可以能使用的国内镜像仓库最好使用阿里云的镜像仓库做一个封装。

并且保证容器化的docker可用的，并且把自己的镜像加速器文件整合到容器镜像中，使得容器化的docker具备完全的root的执行权限。


    kind: pipeline
    name: default
    steps:
    - name: info
      image: registry.cn-hangzhou.aliyuncs.com/baz/busybox:1.0
      commands:
      - echo "liuyusheng hao ke ai !"
      

