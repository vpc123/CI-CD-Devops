## gogs本地github搭建教程: ##
gogs本地快速搭建，使其本地正常运行一个本地的github，提供无人机正常的进行CI/CD的项目可以在企业集群中正常运行起来。

项目代码如下：

在同级目录下存在一个Gogs目录：

    Gogs# cat docker-compose.yml
# 


    
      version: '2'
    services:
    postgres:
      image: postgres:9.5
      restart: always
      ports:
       - "5432:5432"
      environment:
       - "POSTGRES_USER=admin"
       - "POSTGRES_PASSWORD=123456"
       - "POSTGRES_DB=gogs"
      networks:
       - gogs
    gogs:
      image: gogs/gogs:0.11
      restart: always
      ports:
       - "10022:22"
       - "3000:3000"
      links:
       - postgres
      environment:
       - "RUN_CROND=true"
      networks:
       - gogs
      depends_on:
       - postgres

    networks:
    gogs:
      driver: bridge

### 正常运行项目文件命令：
    #docker-compose docker-compose.yml 

通过这种方式我们可以使得gogs项目正常运行，注意开放的映射端口。镜像版本等问题。
之后通过web使用访问控制，进行数据库连接配置操作。

默认使用postgrsql数据库，如果需要更换数据仓库操作，提前进行docker-compose.yml文件参数的配置。

验证运行结果：
![](https://i.imgur.com/h6uP49z.png)

我们可以观察到系统中已经存在gogs正常运行的镜像文件。并且可以访问到映射端口。
之后使用系统配置的镜像参数，进行用户名，密码登录操作。
![](https://i.imgur.com/IXISJ27.png)


登入系统可以观察到的界面信息：
![](https://i.imgur.com/fULrijz.png)



使用方式和github一致：同时git命令行的用法一致。
2018/9/18  开始愉快的生活吧！