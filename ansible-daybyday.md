# ansible 日积月累

## 一. ansible 安装

##### 源代码安装


    $ git clone git://github.com/ansible/ansible.git --recursive
    $ cd ./ansible
    
#### 使用Bash的时候

    $ source ./hacking/env-setup
    
#### 如果没有安装pip需要安装pip后再继续

    $ sudo easy_install pip
    
#### 然后安装响应的依赖包

    $ sudo pip install paramiko PyYAML Jinja2 httplib2 six
    
#### 安装ansible

    $ cd ansible
    $ sudo python setup.py install 

#### Thats so easy, 安装就是这么简单


## 二. 搭建测试环境

#### 1. 先弄两个被控端环境, 我使用docker容器, 在我mac本机拉起两个容器, 开放两个端口, 模拟两台主机, 具体关于docker容器不在这里讲述

    $ docker run -d --name bserver -p 10022:22 sshd:dockerfile /run.sh -D
    $ docker run -d --name bserver -p 10023:22 sshd:dockerfile /run.sh -D
    
#### 测试下免密登陆之前是否配好了, 192.168.99.100 是我docker容器虚拟机宿主机的IP

    $ ssh -p 10022 root@192.168.99.100
    $ ssh -p 10023 root@192.168.99.100
    
#### 2. 创建ansible的hosts文件, 我们按照默认的来,放在 /etc/ansible/下

    $ sudo mkdir -p /etc/ansible
    $ cd /etc/ansible
    $ sudo vi /etc/ansible/hosts
    
#### 增加如下内容到/etc/ansible/hosts文件

    192.168.99.100:10022
    192.168.99.100:10023

#### 3. 测试一下这俩用容器弄的虚拟环境ok不

    $ ansible all -m ping -u root

    192.168.99.100 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
    }
    
#### "pong"的一下通了哈

    ## PS:这里可以看出docker对于标准化测试那是真的非常方便, 上面这个搭建过程, 不考虑docker镜像的处理, 总共也不超过5分钟

## 三. 开玩

#### 1. 为验证效果, 先分别打开两个终端窗口,分别登陆到两台 docker 容器

    $ ssh -p 10022 root@192.168.99.100
    $ ssh -p 10023 root@192.168.99.100
    
#### 2. 然后我们尝试向主机列表里的主机批量发送命令, 当然要用helloworld 啦

    $ ansible all -a "touch /root/helloworld.txt " -u root
    
#### 在两台docker容器上分别 ls, 发现 10022 这个台机器创建了文件, 但是10023这台机器没创建啊

    $ ls
    helloworld.txt
    
    $ ls
    
#### 这是为什么呢, 经过研究, 发现如果hosts里面列表按前面的编排, ansible好像是认为操作的时同一台机器
#### 翻看ansible的相关文档, 发现还可以这样定义主机(/etc/ansible/hosts)
    
    $ cat /etc/ansible/hosts
    
    [my-ubuntu]
    192.168.99.104

    [docker-servers]
    docker1 ansible_ssh_port=10022 ansible_ssh_host=192.168.99.100
    docker2 ansible_ssh_port=10023 ansible_ssh_host=192.168.99.100
    #192.168.99.100:10023
    #192.168.99.100:10023
    
#### 这样编排完之后, 我们再次执行下命令试试

    $ ansible docker-servers -m ping -u root
    
    docker2 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
    }
    docker1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
    }
    
##### 两条返回成了, 再往服务器发送指令试试

    $ ansible docker-server -a "rm /root/helloworld.txt" -u root
    
    docker2 | SUCCESS | rc=0 >>


    docker1 | SUCCESS | rc=0 >>
    
    
    $ ansible docker-server -a "touch /root/helloworld.txt" -u root

    docker1 | SUCCESS | rc=0 >>


    docker2 | SUCCESS | rc=0 >>
    
##### 现在进入到两个docker 容器中, 分别ls, 发现 helloworld.txt 在两台主机上都已经创建了

    $ ls
    helloworld.txt
    
    $ ls
    helloworld.txt
    
