## Drone&gogs使用教程！CI/CD实践。 ##

接着上次的部署进行后续的执行任务。请提前安装部署好gogs还有Drone，之后开始进行Ci/Cd实践工作。

首先我们在gogs也就是自己搭建的github上开始进行新建一个自己想去创建的项目。
![](https://i.imgur.com/AH16ojK.png)

我们创建一个项目名为gogs的项目，之后拉取到本地。
![](https://i.imgur.com/IGW1J2W.png)

![](https://i.imgur.com/Q8TrGKJ.png)


在本地项目根目录下，我们开始进行项目的.deploy.yml文件的书写提交操作。

![](https://i.imgur.com/8A3cevO.png)

![](https://i.imgur.com/nNckb34.png)



我们来看下.deploy.yml的内容吧！
  pipeline:
  build:
    image: centos:6.9
    commands:
      - yum install -y gcc gcc-c++ automake autoconf
      - echo "hello world!!!"
之后push到项目中去。



![](https://i.imgur.com/hU4bKD3.png)

![](https://i.imgur.com/o4166Ue.png)

![](https://i.imgur.com/GI5Y4dj.png)

![](https://i.imgur.com/nnI4M2G.png)

![](https://i.imgur.com/rvGLpds.png)

我们已经正常运行并且实现了快速的CI/CD小型项目落地。
给自己点一个赞吧！
