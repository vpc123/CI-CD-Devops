## drone 1.0 新的定时任务界面&&构建任务支持重启

时间：2018/12/18

作者：流火夏梦

###### drone 1.0 的定时任务是一个不错的功能，早期的版本是必须使用cron 表达式的 最近发布的版本支持通过配置就可以了，很方便，只是目前比较简单的，支持小时、 天、周、月、年的模式

##### 环境准备

- docker-compose 文件


- 启动&&配置 

###### 项目使用gogs 进行git 管理，首先需要配置gogs，添加用户，创建简单的项目， 项目drone配置文件

    kind: pipeline
    name: default
    steps:
    - name: info
      image: busybox
      commands:
      - echo "appdemo"


- 集成drone&&测试

- cron 效果 

- 步骤重启

说明，当时这个步骤只支持部分任务的，重新运行，对于已经运行完成之后，因为基于容器的共享存储已经删除，除非合适共享


参考链接：https://www.cnblogs.com/rongfengliang/p/9993246.html