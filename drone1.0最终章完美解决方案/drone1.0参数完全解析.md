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