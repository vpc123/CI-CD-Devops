## Drone演示完美的CI/CD全流程

### 目的:我们此次借助Drone完美的演示通过gogs和Drone进行ci/cd的流程过程。

演示环境：通过前面的教程构建好Drone1.0的应用开发测试环境，之后通过.drone.yml的书写规则进行合理的流程判断规则，我们通过执行CI/CD流水线开始让我们的整个测试流程完美的跑起来。这只是截止到镜像测试和镜像的推送发布还没有涉及到自动部署的阶段，不过在这个时候我们就是可以实现一套完善的自动化测试开发平台的基本道路了。

### 部署Drone的开发测试环境

参考网址：
https://github.com/vpc123/CI-CD-Devops/tree/master/drone1.0%E6%9C%80%E7%BB%88%E7%AB%A0%E5%AE%8C%E7%BE%8E%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88


### 关联gogs代码库和drone

自动关联的，配置好gogs之后，drone会自动发现到gogs中的数据，开启代码提交自动激活功能，每当代码发生提交动作时，便会自动打包测试镜像。


### 书写测试代码.drone.yml 和Dockerfile

在同级目录下example已经存在正常提交时的实例脚本文件，根据文件字段的不同作用进行相应替换和修改。

#### .drone.yml 文件解析


    kind: pipeline  #指定文件类型，流水线类型文件
    name: docker    #文件名docker
    
    steps:
    - name: publish   #文件作用
      image: plugins/docker:17.12  #文件运行过程中需要的编译镜像，留意不同版本的drone所需要的版本也不完全相同
      settings:
	    repo: registry.cn-hangzhou.aliyuncs.com/vpc123/first  #设置打制和提交代码的位置，这里提交镜像到阿里云，如果是官方镜像存储仓库可以不写具体位置
	    registry: registry.cn-hangzhou.aliyuncs.com   #官网仓库可以默认不写，但是其他共有仓库或者harbor私有仓库必须要注明
	    auto_tag: true     #tag名称，自动打制tag
	    dockerfile: Dockerfile   #如果需要制作新的镜像必须要有一个Dockerfile文件，里面进行制作新镜像的具体步骤
	    username:
	      from_secret: docker_username   #仓库名的用户名称
	    password:
	      from_secret: docker_password   #仓库名的用户密码
	  when:                              #流水线的触发流程时间，当满足这些条件时，便会自动发生镜像打制的触发事件
	    event:
	    - push
	    - tag


#### Dockerfile 文件解析

    FROM registry.cn-hangzhou.aliyuncs.com/ubuntu/ubuntu:hdpOK
    
    LABEL maintainer="Bo-Yi Wu <appleboy.tw@gmail.com>"
    
    ENV FLUTTER_HOME ${HOME}/flutter
    ENV FLUTTER_VERSION 1.0.0-stable
    
    RUN apt-get update \
      && apt-get install -y libglu1-mesa git curl unzip wget xz-utils lib32stdc++6 \
      && apt-get clean
    
    
    
    WORKDIR /

###### 标准的镜像制作流程，根据项目的实际需要进行制作发布。

#### 总结：这次完全把镜像打制完成推送成功，耗费了不少的时间和精力，但是也学会和总结了不少的心得。
#### 针对这次流水线项目的上线过程我觉得十分开心，私有镜像仓库的区别还有整个过程自己没有放弃的心态，都令我获益颇多，Drone的流水线我是从2018年10月份就开始琢磨，直到2019年的1月20号才真正完全理解透彻这其中蕴含的哲学思想。也希望今后碰巧看到这篇指导文章的你可以少走些弯路。



参考链接：

重点参考：

-  重点：https://github.com/appleboy

- 1  https://blog.csdn.net/kikajack/article/details/80593751
 
- 2  https://blog.csdn.net/github_35614077/article/details/83028832