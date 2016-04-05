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
    i. how to 回滚? 如何判断?
    
##### 3. step by step 

3.1 官网最佳实践playbook目录树

    production                # inventory file for production servers 关于生产环境服务器的清单文件
    stage                     # inventory file for stage environment 关于 stage 环境的清单文件
    
    group_vars/
       group1                 # here we assign variables to particular groups 这里我们给特定的组赋值
       group2                 # ""
    host_vars/
       hostname1              # if systems need specific variables, put them here 如果系统需要特定的变量,把它们放置在这里.
       hostname2              # ""
    
    library/                  # if any custom modules, put them here (optional) 如果有自定义的模块,放在这里(可选)
    filter_plugins/           # if any custom filter plugins, put them here (optional) 如果有自定义的过滤插件,放在这里(可选)
    
    site.yml                  # master playbook 主 playbook
    webservers.yml            # playbook for webserver tier Web 服务器的 playbook
    dbservers.yml             # playbook for dbserver tier 数据库服务器的 playbook
    
    roles/
        common/               # this hierarchy represents a "role" 这里的结构代表了一个 "role"
            tasks/            #
                main.yml      #  <-- tasks file can include smaller files if warranted
            handlers/         #
                main.yml      #  <-- handlers file
            templates/        #  <-- files for use with the template resource
                ntp.conf.j2   #  <------- templates end in .j2
            files/            #
                bar.txt       #  <-- files for use with the copy resource
                foo.sh        #  <-- script files for use with the script resource
            vars/             #
                main.yml      #  <-- variables associated with this role
            defaults/         #
                main.yml      #  <-- default lower priority variables for this role
            meta/             #
                main.yml      #  <-- role dependencies
    
        webtier/              # same kind of structure as "common" was above, done for the webtier role
        monitoring/           # ""
        fooapp/               # ""

3.2 依照最佳实践建立我们自己的目录树

    $ mkdir -p {production,stage,group_vars,host_vars,library,filter_plugins,roles/{common/{tasks,handlers,templates,files,vars,defaults,meta},monitoring,fooapp}}
    
    $ tree
    .
    |____filter_plugins
    |____group_vars
    |____host_vars
    |____library
    |____production
    |____roles
    | |____common
    | | |____defaults
    | | |____files
    | | |____handlers
    | | |____meta
    | | |____tasks
    | | |____templates
    | | |____vars
    | |____fooapp
    | |____monitoring
    |____stage

3.3 tesrt

    afalfj'df
    阿达来激发了fj
    
    
    
    