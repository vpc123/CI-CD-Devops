## Drone 快速搭建教程

###### 此教程基于gogs配套搭建使用！
gogs本地快速搭建，使其本地正常运行一个本地的github，提供无人机正常的进行CI/CD的项目可以在企业集群中正常运行起来。

项目代码如下：

在同级目录下存在一个Gogs目录：

    Drone# cat docker-compose.yml
# 
    version: '2'
     
    services:
      drone-server:
    image: drone/drone:0.8
    ports:
      - 8000:8000
      - 9000:9000
    volumes:
      - /tmp/drone:/var/lib/drone/
    environment:
      - DRONE_OPEN=true
      - DRONE_HOST=http://10.100.100.32:9000
      - DRONE_GOGS=true
      - DRONE_GOGS_URL=http://10.100.100.32:3000
      - DRONE_SECRET=test
      drone-agent:
    image: drone/agent:0.8
    command: agent
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - drone-server
    environment:
      - DRONE_SERVER=10.100.100.32:9000
      - DRONE_SECRET=test

# 
### 正常运行项目文件命令：
    #docker-compose docker-compose.yml 

通过这种方式我们可以使得drone项目正常运行，注意开放的映射端口。镜像版本等问题。
之后通过web使用访问控制，进行数据库连接配置操作。

默认使用postgrsql数据库，如果需要更换数据仓库操作，提前进行docker-compose.yml文件参数的配置。

验证运行结果：
![](https://i.imgur.com/Ol6CjF2.png)

我们可以观察到系统中已经存在gogs正常运行的镜像文件。并且可以访问到映射端口。
之后使用系统配置的镜像参数，进行用户名，密码登录操作。用户名，还有密码同gogs是一样的。
![](https://i.imgur.com/q3fDxwK.png)


![](https://i.imgur.com/xDUPnmZ.png)


![](https://i.imgur.com/sToJDSf.png)

到此为止，我们已经顺利绑定gogs还有无人机。
下一步就是还要我们一旦提交代码，便会自动化的打包构建的工作。
下一节我们就会开始顺利进行打包提交的编译构建过程。