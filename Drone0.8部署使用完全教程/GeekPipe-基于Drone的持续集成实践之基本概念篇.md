## DevOps | Drone 的基本概念


- Drone 一个原生支持 Docker 的开源 CI 系统，基于 Go 编写。
- Drone 的核心概念
- Drone 的运作原理(架构)
- Drone 的基本术语



### Drone 的基本概念

Drone 是一个基于 Docker 容器技术的可扩展的持续集成引擎，用于自动化测试与构建，甚至发布。每个构建都在一个临时的Docker容器中执行，使开发人员能够完全控制其构建环境并保证隔离。开发者只需在项目中包含 .drone.yml 文件，将代码推送到 git 仓库，Drone 就能够自动化的进行编译、测试、发布。


#### 阅读要求

- 熟悉 Docker 以及 Docker Compose
- 熟悉 Git 基本命令
- 对 CI/CD 有一定了解


#### Drone 的基本原理

Drone 的部署分为 Server(Drone-Server) 和 Agent(Drone-agent):

- Server端：负责后台管理界面以及调度
- Agent端：负责具体的任务执行,比如配置文件 .drone.yml 里的 parsing 发生在 drone-server 端，而具体的步骤则在 drone-agent 端里执行。也就意味着 environment/build_args 关键字是不影响配置文件.drone.yml 本身环境变量的解析替换。

目前Drone 支持多种代码托管服务，几乎涵盖市面上主流的代码仓库，如Github, GitLab, Gogs, Gitea、Bitbucket Server等，且在drone-server 里预设了对应托管服务的 API，Drone 的很多功能比如拉取 git repo list/add webhook to repo 都是通过这些 API 完成的。另外 Drone的账户体系依赖于托管服务的账户系统， 并不存在维护账户这个概念。例如我们使用Gogs作为代码托管服务，我们在登录 Drone 时，实际上是 Drone 把用户名密码传给了 Gogs. 因此，激活某个 Repository （仓库） 的构建(为Repository 添加webhook) 能否成功取决于该账号在 Gogs 里是不是该 Repository 的管理员。




#### Drone 的基本术语

##### Webhooks

Drone 通过 OAuth 认证或账号密码登陆代码仓库后，获得完整的控制权。在 Drone 的 web 后台管理页面激活 Repo 后，Drone 调用代码仓库的 API 给 Repository 增加一个 webhook, 当 Repository 出发相应事件(push, tag, pull request) 时，代码仓库发起 http 请求回调 drone 触发构建。

Webhooks 由代码仓库发送，用于触发 pipeline. 代码仓库会在下面 3 中情况下，自动发送 Webhook 请求到 Drone:

- 代码被 push 到 Repo
- 新建一个合并请求 (pull request)
- 新建一个 tag


##### 跳过提交

在本地用 git 提交代码的时候可以通过添加 [CI SKIP](大小写不敏感) 到提交信息(commit message) 中来让 Drone 跳过某个提交。

    $ git commit -m "updated README [CI SKIP]"

##### 包含/跳过分支 Branches

Drone 可以通过参数 branches 忽略某个分支上的提交。branches 支持单个变量，数组，通配符，还支持自选项 exclude (不包含分支)

- 跳过不是 master 分支的提交：

-

    pipeline:
	  build:
	    image: golang
	    commands:
	      - go build 
	      - go test
	branches: [master, develop]

- 跳过匹配外的目标分支提交-多个目标匹配：

- 

    pipeline:
	  build:
	    image: golang
	    commands:
	      - go build 
	      - go test
	branches: [master, develop]

- 提交匹配包含的目标分支：

- 

    pipeline:
	  build:
	    image: golang
	    commands:
	      - go build
	      - go test
	branches:
	  include: [ master, feature/* ]


- 忽略目标分支的提交：

- 

    pipeline:
	  build:
	    image: golang
	    commands:
	      - go build
	      - go test
	branches:
	  exclude: [ develop, feature/* ]

##### Workspace

Workspace 定义了所有工作流步骤共享的容器空间和目录。各个阶段通过各个阶段共享 volume 和工作路径，避免了复制。默认的工作区的目录和仓库的 URL 相匹配。

    /drone/src/gogs/test/hello-world

- 可以使用 YAML 文件里的 workspace 来自定义工作区：

- 

    workspace:
	  base: /go
	  path: src/gogs/test/hello-world
	    
	pipeline:
	  build:
	    image: golang:latest
	    commands:
	      - go get
	      - go test



##### base 属性

base 属性定义了所有工作流步骤共享的基础容器空间。基础容器空间保证了代码、依赖和编译的二进制文件能够在各个步骤间持久化和共享。


    workspace:
	  base: /go
	  path: src/gogs/test/hello-world
	pipeline:
	  deps:
	    image: golang:latest
	    commands:
	      - go get
	      - go test
	  build:
	    image: node:latest
	    commands:
	      - go build

上面的步骤将和下面的 docker 命令类似：

    $ docker volume create my-named-volume
    $ docker run --volume=my-named-volume:/go golang:latest
    $ docker run --volume=my-named-volume:/go node:latest

##### path 属性

path 属性定义了构建的工作目录。这是代码被克隆到的目录，也将是每一个构建步骤的默认工作目录。这个路径必须是基于 base 路径的相对路径。

    workspace:
	  base: /go
	  path: src/gogs/test/hello-world


相当于：

    $ git clone https://gogs/test/hello-world /go/src/gogs/test/hello-world

###### Note: hello-world 目录下即为源代码。

##### cloning

Drone 允许在工作流中手动配置克隆步骤。如果没有定义具体的克隆步骤，Drone 会自动配置。

    clone:
	  git:
	    image: pugins/git
	pipeline:
	  build:
	    image:golang
	    commands:
	      - go build
	      - go test 

depth 可修改克隆深度，当 depth 为 1 时，只克隆当前版本，不克隆当前版本以外的提交记录，如果该参数未指定，默认所有版本提交都克隆：

    clone:
	  git:
	    image: plugins/git
	    depth: 50

通过 image 指定克隆插件时，可以使用 plugins/ 下的预定义插件，也可以使用自定义插件：

    clone:
	  git:
	    image: test/custom-git-plugin


    

##### Pipelines

Drone 的核心概念，pipelines 定义了工作流(或叫流水线)，包含代码构建，代码测试和代码部署等一系列步骤。drone-agent 以自上而下的顺序依次执行 pipeline 关键字里声明的步骤。如果一个步骤返回了非 0 的退出代码，工作流将立即停止并返回一个错误状态。

例如，定义了两个工作流，名字为 frontend 和 backend:

    pipeline:
	  backend:
	    image: golang
	    commands:
	      - go build 
	      - go test
	  frontent:
	    image: node
	    commands:
	      - npm install
	      - npm run test
	      - npm run build


##### 构建步骤

构建步骤是工作流在 Docker 容器中执行的命令。这些命令将工作区(Workspace) 作为工作路径。

    pipeline:
	  backend:
	    image: golang
	    commands:
	+     - go build 
	+     - go test

上面的 commands 将会被转换成简单的 Shell 脚本，大致如下：

    #!/bin/sh
	set -e 
	go build 
	go test

上面的脚本在之后会被作为 Docker 的入口被执行，如下所示：


    $ docker run --entrypoint=build.sh golang


###### Note: 只有构建步骤可以定义命令(commands), 不能使用插件(plugins) 或者服务(services) 来定义命令。


##### 并行执行 group

Drone 支持在同一个示例上并行执行多个步骤。使用 group 属性来配置并行步骤，这将让工作流执行者 (pipeline runner) 并行执行指定的命令。

并行执行配置实例：

    pipeline:
	  backend:
	    group: build
	      image: golang
	      commands:
	        - go build
	        - go test
	  frontend:
	    group: build
	      image: node
	      commands:
	       - npm install 
	       - npm run test
	       - npm run build 
	  publish:
	    image: plugins/docker
	    repo: test/hello-world


上面的例子中，frontend 和 backend 将并行执行。在这两组任务完成之前，publish 步骤将不会执行。


##### 条件执行 when

Drone 允许有条件地执行步骤。下面的例子限制了允许使用 Slack 插件的 Git 分支为 master:

    pipeline:
	  slack:
	    image: plugins/slack
	    channel: dev
	    when:
	     branch: master
    

##### 故障执行 when + status

Drone 使用容器退出代码来决定一个构建的成功或失败。非 0 退出代码(Non-zero exit codes) 使构建失败，同时立即退出当前工作流。

如果需要在构建失败时，执行特定的工作流步骤。可以使用状态条件限制(status constraint) 来修改构建失败时的默认行为和执行步骤。

    pipeline:
	  slack:
	    image: plugins/slack
	    channel: dev
	    when:
	     status: [success, failure]

    

##### Services

services 模块定义服务容器，用类似 docker-compose 的语法，提供服务作为 CI 过程中的支撑。下面的配置文件组建了数据库服务器和缓存服务容器：


    pipeline:
	  build:
	    image: golang:test
	    commands:
	      - go build 
	      - go test
	services:
	  db-server:
	    image: mysql
	  cache-server
	    image: redis

服务容器可以使用 YAML 配置文件中定义的名称来访问。在上面的例子中，mysql 服务被指定为 db-server, 可以使用 db-server:3306 来访问。



##### 配置

服务容器可以通过环境变量来自定义用户名，密码和端口。具体支持的环境变量需要参考各个服务的官方镜像文档，如 MySQL 数据库部分的配置如下：

    services:
	  database:
	    image: mysql
	    environment:
	      - MYSQL_DATABASE=test
	      - MYSQL_ALLOW_EMPTY_PASSWORD=yes



##### 初始化

服务容器需要一定时间来初始化，然后才能被访问。如果刚开始无法连接到一个服务，可以先等待几秒钟或者添加一个退让 (backoff) 机制。

    pipeline: 
	  test:
	    image: golang
	    commands:
	      - sleep 15
	      - go get
	      - go test
	services:
	  database:
	    image:mysql

##### Plugins

Plugins 也是 Drone 的核心概念，Plugins 是执行预定义任务的容器，它们在工作流中被配置为步骤 (steps)。Drone 除了 build 命令之外的功能基本都通过插件完成，例如部署代码，发布构建结果，FTP(SSH)上传, 发送通知以及部署其他更多的功能。建议精读官方用于构建镜像的 docker plugin 的文档：http://plugins.drone.io/drone-plugins/drone-docker , 另外，git 是 Drone 默认的插件，不需要配置。

Docker 和 Slack 插件工作流示例：

    pipeline:
	  build:
	    image: golang
	    commands:
	      - go build
	      - go test
	  publish:
	    image: plugins/docker
	    repo: foo/bar
	    tags: latest
	  
	  notify:
	    image: plugins/slack
	    channel: dev

##### 插件隔离

插件在 Docker 容器中执行，它们与工作流中的其他步骤相互隔离。注意，插件挂载和共享当前工作区，因此它将可以访问对应的源代码。



##### 插件市场

插件被打包和发布为 Docker 容器，它们在概念上和软件库类似(比如 npm), 可以在社区中发布和共享。官方插件市场：http://plugins.drone.io/

##### Deployments

Deployments 参数可以触发 Drone 来部署项目。部署项目在工作流中的事件类型 (event type) 是 deployment。可以使用事件类型 (event type) 或者目标环境变量 (target environment) 来限制执行的部署。

    pipeline:
	  build:
	    image: golang
	    commands:
	      - go build
	      - go test
	  publish:
	    image: plugins/docker
	    registry: registry.heroku.com
	    repo: registry.heroku.com/my-staging-app/web
	    when:
	      event: deployment
	      environment: staging
	  publish_to_prod:
	    image: plugins/docker
	    registry: registry.heroku.com
	    repo: registry.heroku.com/my-production-ap/web
	    when:
	      event: deployment
	      environment: production

上面的例子展示了如何配置工作流指定在特定的目标环境中部署项目，比如 production 生产环境。

##### 触发部署

Drone 也支持使用命令行从已有的构建结果中触发部署。这个行为与使用构建结果(promoting builds) 的概念类似。

    $ drone deploy <_repo_> <_build_> <_environment_>

在准生产环境（staging environment）中使用某一序号的构建结果。

    $ drone deploy test/hello-world 24 staging

在生产环境（production environment）中使用某一序号的构建结果。

    $ drone deploy test/hello-world 24 production




##### atrix Builds 矩阵构建

Drone 支持整合矩阵构建。Drone 为配置文件矩阵中的每个组合各执行一个独立的构建任务。矩阵构建允许您使用多个配置来构建和测试一个提交(commit)。它更适用于开源项目和 libs 维护者。可以为同一个环境变量声明多组值，笛卡儿积后进行交叉构建。Matrix Builds 可以在 .drone.yml 文件里注入环境变量。

矩阵定义示例：

    matrix:
	  GO_VERSION:
	    - 1.4
	    - 1.3
	  REDIS_VERSION:
	    - 2.6
	    - 2.8 
	    - 3.0


只包含特定组合的矩阵示例：

    matrix:
	  include:
	    - GO_VERSION: 1.4
	      REDIS_VERSION: 2.8
	    - GO_VERSION: 1.5
	      REDIS_VERSION: 2.8
	    - GO_VERSION: 1.6
	      REDIS_VERSION: 3.0

##### 变量插值

矩阵变量可以使用 ${VARIABLE} 的语法来进行插值。


    pipeline:
	  build:
	    image: golang:${GO_VERSION}
	    commands:
	      - go get
	      - go build
	      - go test
	services:
	  database:
	    image: ${DATABASE}
	matrix:
	  GO_VERSION:
	    - 1.4
	    - 1.3
	  DATABASE:
	    - mysql:5.5
	    - mysql:6.5
	    - mariadb:10.1


插入矩阵变量后的 Yaml 文件示例：

    pipeline:
	  build:
	-   image: golang:${GO_VERSION}
	   image: golang:1.4
	    commands:
	      - go get
	      - go build
	      - go test
	   environment:
	     - GO_VERSION=1.4
	     - DATABASE=mysql:5.5
	services:
	  database:
	-   image: ${DATABASE}


基于 Docker 镜像标签（Docker image tag）的矩阵构建


    pipeline:
	  build:
	    image: golang:${TAG}
	    commands:
	      - go build
	      - go test
	matrix:
	  TAG:
	    - 1.7
	    - 1.8
	    - latest

基于 Docker 镜像的矩阵构建

    pipeline:
	  build:
	    image: ${IMAGE}
	    commands:
	      - go build
	      - go test
	matrix:
	  IMAGE:
	    - golang:1.7
	    - golang:1.8
	    - golang:latest

示例文件：.drone.yml


    workspace:
	  base: /go
	  path: src/gogs.finogeeks.com/baa-cicd
	pipeline:
	  
	  build:
	    image: golang:latest
	    commands:
	      - go build -o baa-cicd
	  
	  publish:
	    image: plugins/docker
	    registry: registry.finogeeks.com
	    repo: registry.finogeeks.com/test/baa-cicd
	    tags: latest
	    secrets: [ docker_username, docker_password ]
	    insecure: true
	  
	  notify:
	    image: plugins/slack
	    webhook: https://hooks.slack.com/services/xxx/xxx/xxx
	    channel: dev
	    template: >
	      {{#success build.status}}
	        build {{build.number}} succeeded. Good job.
	      {{else}}
	        build {{build.number}} failed. Fix me please.
	      {{/success}}


上面示例中，pipeline 中的 build 部分执行项目的构建，publish 部分将项目的 Docker 镜像发布到 registry, notify 部分发送 slack 消息



##### Environment

环境变量可以按照生效位置 (drone-server/drone-agent) 分为两大类，具体到多个使用场景：


drone.yml 预设的变量，共有两处：

- 这些变量可以在 .drone.yml 文件中直接以 ${XX_XX_XX} 的格式使用，并在 parsing 阶段被替换为具体值。这个行为发生在 drone-server 里，属于构建周期的最早期阶段。 它们分别是: drone 为使用者预设的环境变量，详细变量列表请参考: http://docs.drone.io/environment-reference/
- matrix builds 声明的变量数组。 这两类环境变量暴露出的信息很有限，matrix builds 只能是常量。drone的预设变量里只有 DRONE_COMMIT_MESSAGE 可控。drone 会对其前后插入双引号，末尾加换行符，内容里的特殊字符加转移等方式保证 .drone.yml 文件的安全性和防注入。

CI 容器端的变量

这些变量是作用在 drone-agent 端。包括:

- pipeline 部分的 environment 关键字，类似 docker-compose. 影响 CI 执行容器的运行时环境变量。等效于在容器里执行 export XX=XXX
- build_args 关键字，影响 Dockerfile 文件本身的环境变量。在 docker build 过程中生效。等效于 Dockerfile 里声明 ARG 关键字，也等效于构建命令 docker build –build-arg. 常用场景主要是 http_proxy 变量。 另外 build_args(ARG) 引申出的话题属于 docker 本身的范畴，需要对 Dockerfile 的 ARG 关键字和 ENV 关键字要有所区分。它们大致上无太大差别，唯一的区别是变量的生效周期，前者在 docker build 执行期间。而后者还会 persist 到产物 image 里。


至此，完。

links1:https://zhezh09.github.io/post/tech/cloud/devops/20180925-drone-01-basic/
links2:https://developer.finogeeks.com/topic/11/geekpipe-%E5%9F%BA%E4%BA%8Edrone%E7%9A%84%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E5%AE%9E%E8%B7%B5%E4%B9%8B%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%E7%AF%87