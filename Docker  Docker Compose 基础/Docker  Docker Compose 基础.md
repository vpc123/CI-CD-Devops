### Centos下安装docker-compose的方法

1  安装epel的源

> ```
> $yum -y install epel-release
> ```

2   执行成功之后，再次执行yum -y install python-pip

> ```
> $yum install python-pip
> ```

3   对安装好的pip进行升级 pip install --upgrade pip(可以不进行) 

> ```
> $pip install --upgrade pip
> ```

4   安装Docker-Compose

> ```
> $pip install docker-compose
> ```

5   检查docker-compose的版本信息

> ```
> $docker-compose -version
> ```



总结:docker-compose的安装部署，在进行项目部署使用中，针对drone的项目实践实例化项目的功能特性，我们快速的根据原型产品进行项目的实践，将drone进行容器化，最好不要上k8s因为开始项目的实践在k8s集群表现不是特别友好，我又不想使用jenkins哪种过分消耗系统资源的设施。
