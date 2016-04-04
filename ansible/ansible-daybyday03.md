## 四. 从实例学习playbook

###  场景二 : 尝试用ansible更新部署tomcat工程
----------------------------
#### 1. 环境说明

    环境说明:
    ansible版本:2.1.0
    os=ubuntu14.04 LTS
    tomcat 7.0.68
    端口号: 80 443
    servers:仍然是两台docker容器
    192.168.99.100:10022
    192.168.99.100:10023
    tomcat更新包RELEAE服务器: 192.168.99.104:/home/deploy/test/YYYYMMDD/test.war

##### 仍然希望使用docker完成测试(省时省事啊)

    docker 基础镜像名字为: sshd:dockerfile
    这是我自己的镜像仓库中得镜像
    这个镜像只是再unbutu系统镜像的基础上, 开放了ssh服务, 这样比较方便进行测试和研究
    
##### 2. 梳理更新思路

    a. 备份原有工程到本地~/backup目录下, 备份包名称为test_YYYYMMDDHHMM.tar.gz, 包含整个tomcat 目录, 排除temp等目录
       备份配置文件到git
    b. 传包到本地~/deploy目录下, ~/deploy/test/YYYYMMDD/test.war
    c. 停止tomcat
    d. 将包从 ~/deploy/test/YYYYMMDD/test.war, 传到tomcat得 webapps/ROOT下
    e. 解压test.war, unzip test.war
    f. 从git拉回配置文件
    g. 重新启动tomcat, 并观察启动日志是否正常
    h. 测试工程是否正常启动, curl -i localhost:8080 , 应返回200, 或使用特定的业务url测试
    
##### 3. step by step 