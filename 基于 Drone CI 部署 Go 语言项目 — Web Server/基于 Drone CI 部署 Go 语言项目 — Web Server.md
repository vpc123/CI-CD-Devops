## 基于 Drone CI 部署 Go 语言项目 — Web Server

### 简介

这里以一个 Golang 项目(Go-Web-App)为例，基于 Drone CI 服务完成其自动化部署工作流的配置和演示：主要步骤包含：


    clone => test => build => image => deploy => notify



包含开发的完成流程说明：

- 开发项目代码, 包括源码文件，.drone.yml 文件、Dockerfile 文件等配置文件和一系列执行脚本(*.sh)
- 提交代码：push、pull request、tag 到代码仓库(Bitbucket Server), 通过 Webhook 触发 Drone 的 Pipeline
- Drone 开始 Pipeline 的执行
	- 	Clone 代码至容器(Drone 默认执行)
	- 	执行单元测试
	- 	编译代码，构建生成可执行程序
	- 	构建 Docker 镜像，并发布到 Docker Registry
	- 	打包资源文件、执行脚本、配置文件等到远程测试服务器
	- 	部署应用容器到测试环境
	- 	结果通知



##### Note: Drone 引入了 pipeline 的概念，上述整个构建过程由多个 stage 组成，每一个 stage 都是在一个 Docker 容器内执行:

- 各 stage 间可以通过共享宿主机的磁盘目录, 实现 build 阶段的数据共享和缓存基础数据, 实现加速下次 build 的目标
- 各 stage 也可以共享宿主机的 docker 环境，实现共享宿主机的 docker image, 不用每次 build 都重新拉取 base image，减少 build 时间
- 可以并发运行。多个 build 可以并发运行，单机并发数量由服务器 cpu 数决定

### Drone Pipeline: 单元测试(test)



    workspace:
	base: /go
	path: src/repo/goweb-app
	
	clone:
	  git:
	    image: plugins/git
	    depth: 50
	    tags: true
	
	pipeline:
	
	  ping: 
	    image: mongo:latest
	    commands:
	      - sleep 3
	      - mongo --host mgo --eval "{ping:1}"
	
	services:  # Dependent service
	  mgo:
	    image: mongo:latest
	
	branches:  # Enable Drone CICD on: master & release/* branches
	  include: [ master, release/* ]



### Drone Pipeline: 发布 Docker 镜像(image)

这里 Docker 镜像是发布到阿里云镜像仓库，将其打包为 Docker Image 发布了，此时的 .drone.yml 配置如下：


    workspace:
	base: /go
	path: src/repo/goweb-app
	
	clone:
	  git:
	    image: plugins/git
	    depth: 50
	    tags: true
	
	pipeline:
	
	  ping: 
	    image: mongo:latest
	    commands:
	      - sleep 3
	      - mongo --host mgo --eval "{ping:1}"
	
	  image:  
	    image: plugins/docker
	    repo: registry.cn-hangzhou.aliyuncs.com/vpc123/six
	    registry: registry.cn-hangzhou.aliyuncs.com
	    secrets: [ docker_username, docker_password ]
	    context: ./
	    dockerfile: ./Dockerfile
	    default_tags: true
	    when:
	      event: push
	services:  
	  mgo:
	    image: mongo:latest
	
	branches:  
	  include: [ master, release/* ]



### Drone Pipeline: 上传静态资源和部署脚本(scp)

    workspace:
	base: /go
	path: src/repo/goweb-app
	
	clone:
	  git:
	    image: plugins/git
	    depth: 50
	    tags: true
	
	pipeline:
	
	  ping: 
	    image: mongo:latest
	    commands:
	      - sleep 3
	      - mongo --host mgo --eval "{ping:1}"
	
	  image:  
	    image: plugins/docker
	    repo: registry.cn-hangzhou.aliyuncs.com/vpc123/six
	    registry: registry.cn-hangzhou.aliyuncs.com
	    secrets: [ docker_username, docker_password ]
	    context: ./
	    dockerfile: ./Dockerfile
	    default_tags: true
	    when:
	      event: push
	
	
	  scp:
	    image: appleboy/drone-scp
	    group: build
	    host:
	      - 192.168.131.30
	    username: root
	    port: 22
	    secrets: [ ssh_password ]
	    target: /home/workspace/ 
	    source: ./release.tar.gz              
	    when:
	      event: push
	

	services:
	  mgo:
	    image: mongo:latest
	
	branches:
	  include: [ master, release/* ]


### Drone Pipeline: 服务部署(deploy)



    workspace:
	base: /go
	path: src/repo/goweb-app
	
	clone:
	  git:
	    image: plugins/git
	    depth: 50
	    tags: true
	
	pipeline:
	
	  ping: 
	    image: mongo:latest
	    commands:
	      - sleep 3
	      - mongo --host mgo --eval "{ping:1}"
	
	  image:  
	    image: plugins/docker
	    repo: registry.cn-hangzhou.aliyuncs.com/vpc123/six
	    registry: registry.cn-hangzhou.aliyuncs.com
	    secrets: [ docker_username, docker_password ]
	    context: ./
	    dockerfile: ./Dockerfile
	    default_tags: true
	    when:
	      event: push
	
	
	  scp:
	    image: appleboy/drone-scp
	    group: build
	    host:
	      - 192.168.131.30
	    username: root
	    port: 22
	    secrets: [ ssh_password ]
	    target: /home/workspace/ 
	    source: ./release.tar.gz              
	    when:
	      event: push
	
	  deploy: 
	    image: appleboy/drone-ssh
	    host:
	      - 192.168.131.30
	    username: root
	    port: 22
	    secrets: [ ssh_password ]
	    command_timeout: 60
	    script:
	      - cd /home/workspace/
	      - pwd
	      - ls -l
	      - mkdir liuyushenghaoshuai
	      - echo "我就是这么骄傲!"
	    when:
	      event: push

	services:
	  mgo:
	    image: mongo:latest
	
	branches:
	  include: [ master, release/* ]












### Drone Pipeline: 结果通知(notify:hipchat)

    workspace:
	base: /go
	path: src/repo/goweb-app
	
	clone:
	  git:
	    image: plugins/git
	    depth: 50
	    tags: true
	
	pipeline:
	
	  ping: 
	    image: mongo:latest
	    commands:
	      - sleep 3
	      - mongo --host mgo --eval "{ping:1}"
	
	  image:  
	    image: plugins/docker
	    repo: registry.cn-hangzhou.aliyuncs.com/vpc123/six
	    registry: registry.cn-hangzhou.aliyuncs.com
	    secrets: [ docker_username, docker_password ]
	    context: ./
	    dockerfile: ./Dockerfile
	    default_tags: true
	    when:
	      event: push
	
	
	  scp:
	    image: appleboy/drone-scp
	    group: build
	    host:
	      - 192.168.131.30
	    username: root
	    port: 22
	    secrets: [ ssh_password ]
	    target: /home/workspace/ 
	    source: ./release.tar.gz              
	    when:
	      event: push
	
	  deploy: 
	    image: appleboy/drone-ssh
	    host:
	      - 192.168.131.30
	    username: root
	    port: 22
	    secrets: [ ssh_password ]
	    command_timeout: 60
	    script:
	      - cd /home/workspace/
	      - pwd
	      - ls -l
	      - mkdir liuyushenghaoshuai
	      - echo "我就是这么骄傲!"
	    when:
	      event: push
	
	#    +  hipchat: # Notify
	#    +    image: jmccann/drone-hipchat
	#    +    url: https://hipchat.your_company_domain.com
	#    +    room: 19
	#    +    auth_token: tWVtsTFzsbkwSJO79JHcyb11TR8dLyEYAlicHviB
	#    +    from: Drone
	#    +    when:
	#    +      status: [ success, failure ]
	#    +    template: >
	#        <strong>{{ uppercasefirst build.status }}</strong> <a href=\"{{ system.link_url }}/{{     repo.owner }}/{{ repo.name }}/{{ build.number }}\">{{ repo.owner }}/{{ repo.name }}#{{     truncate build.commit 8 }}</a> ({{ build.branch }}) by {{ build.author }} in {{ duration     build.started_at build.finished_at }} </br> - {{ build.message }}
	    
	
	
	services:
	  mgo:
	    image: mongo:latest
	
	branches:
	  include: [ master, release/* ]



###### 总结：

至此，我们已经完全完成了drone0.8的所有测试以及验证工作，包括项目的部署，具体流程将会从以下几个方面开始阐述，从项目构建，测试，流水线，镜像上传，项目部署以及最后消息推送等一系列的事件。