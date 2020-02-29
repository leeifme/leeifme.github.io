---
title: 搭建 Airflow 任务调度环境
date: 2019-08-25 15:15:15
draft: false
hideToc: false
enableToc: true
enableTocContent: false
tocLevels: ["h2", "h3", "h4"]
author: 睡沙发の沙皮狗
authorEmoji: 👻
tags:
- 今日知识点
series:
-
categories:
-
image:
---

## 前期准备

### 关闭防火墙和 selinux


```sh
systemctl stop firewalld
systemctl disable firewalld
setenforce 0 
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
```

### 部署 Python 3.X

-   依赖包安装

    Python 安装会依赖一些环境，为了避免在安装过程中因为缺少依赖而产生不必要的麻烦，需要先执行以下命令，确保安装好以下依赖包：

    ```bash
    yum -y install zlib zlib-devel
    yum -y install bzip2 bzip2-devel
    yum -y install ncurses ncurses-devel
    yum -y install readline readline-devel
    yum -y install openssl openssl-devel
    yum -y install openssl-static
    yum -y install xz lzma xz-devel
    yum -y install sqlite sqlite-devel
    yum -y install gdbm gdbm-devel
    yum -y install tk tk-devel
    yum -y install libffi-devel # 只有 3.7 才会用到这个包，如果不安装这个包的话，在 make 阶段会出现如下的报错：ModuleNotFoundError: No module named '_ctypes'
    yum install gcc
    yum install wget
    
    # 安装 pip，因为 CentOs 是没有 pip 的
    #运行这个命令添加epel扩展源 
    yum -y install epel-release 
    #安装pip 
    yum install python-pip
    ```

-   Python 源码下载

    可以前往 https://www.python.org/ftp/python/ 查看 Python 各个版本

    ```sh
    wget https://www.python.org/ftp/python/3.7.4/Python-3.7.4.tgz
    ```

知识点补充：如果不知道 `configure, make, make install` 三个命令作用，[点这里查看](https://www.cnblogs.com/tinywan/p/7230039.html)

**安装 Python**

-   解压缩

    ```sh
    tar -zxvf Python-3.7.0.tgz
    ```

-   进入解压后的目录，依次执行下面命令进行手动编译

    ```sh
    ./configure prefix=/usr/local/python3 # 指定目录
    make && make install
    ```

    如果最后没提示出错，就代表正确安装了

-   添加软链接

    ```sh
    # 做好原始数据的备份
    mv /usr/bin/python /usr/bin/python2.backup 
    mv /usr/bin/pip /usr/bin/pip2.backup
    
    # 设置软连接
    ln -s /usr/local/python3/bin/python3.7 /usr/bin/python
    ln -s /usr/local/python3/bin/pip3.7 /usr/bin/pip
    
    # 检测
    python -V
    pip -V
    ```

-   保证系统可用

    更改 yum 配置，因为其要用到 python2 才能执行，否则会导致 yum 不能正常使用（不管安装 python3 的那个版本，都必须要做的）

    ```yaml
    # 把 #! /usr/bin/python 修改为 #! /usr/bin/python2 
    vi /usr/bin/yum 
    vi /usr/libexec/urlgrabber-ext-down 
    ```

### 部署 PostgreSQL

-   安装 PostgreSQL 仓库

    ```sh
    $ yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    ```

-   安装客户端

    ```sh
    $ yum install postgresql10
    ```

-   安装服务端

    ```sh
    $ yum install postgresql10-server
    ```

-   加入开机启动项

    ```sh
    $ systemctl enable postgresql-10.service
    ```

-   初始化数据库

    ```sh
    $ /usr/pgsql-10/bin/postgresql-10-setup initdb
    ```

-   启动数据库服务

    ```sh
    $ systemctl start postgresql-10
    ```

-   检查运行状态

    ```sh
    $ systemctl status postgresql-10
    ```

-   设置用户密码

    ```sh
    $ su - postgres
    Last login: Mon Aug  5 11:09:14 CST 2019 on pts/0
    # 默认用户postgres
    -bash-4.2$ psql
    psql (10.9)
    Type "help" for help.
    
    postgres=# \password
    # 设置密码为 '******'
    Enter new password:
    Enter it again:
    # 保存退出
    postgres=# \q
    ```

-   修改监听地址

    `vi /var/lib/pgsql/10/data/postgresql.conf `

    ```sh
    listen_addresses = '*'
    #listen_addresses = 'localhost'
    ```

-   修改客户端认证方式

    `vi /var/lib/pgsql/10/data/pg_hba.conf`

    ```sh
    # replication privilege.
    #local   replication     all                                     peer
    #host    replication     all             127.0.0.1/32            ident
    #host    replication     all             ::1/128                 ident
    host    all     all             0.0.0.0/0                 md5
    ```

-   重启服务

    ```sh
    $ systemctl restart postgresql-10
    ```

-   连接测试

    ```sh
    $ psql -h 192.168.1.2 -U postgres
    ```

### 部署 RabbitMQ

**安装 Erlang**

RabbitMQ 是使用 Erlang 开发的，所以需要首先安装 Erlang，本文安装其最新版本

-   添加 repo 文件：

    ```sh
    $ vi /etc/yum.repos.d/rabbitmq_erlang.repo
    ```

    文件内容：

    ```sh
    [rabbitmq_erlang]
    name=rabbitmq_erlang
    baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/$basearch
    repo_gpgcheck=1
    gpgcheck=0
    enabled=1
    gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
    sslverify=1
    sslcacert=/etc/pki/tls/certs/ca-bundle.crt
    metadata_expire=300
    
    [rabbitmq_erlang-source]
    name=rabbitmq_erlang-source
    baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/SRPMS
    repo_gpgcheck=1
    gpgcheck=0
    enabled=1
    gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
    sslverify=1
    sslcacert=/etc/pki/tls/certs/ca-bundle.crt
    metadata_expire=300
    ```

-   安装：

    ```sh
    $ yum -y install erlang socat
    ```

**安装 RabbitMQ**

-   下载 RPM 包：

    ```sh
    $ wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.17/rabbitmq-server-3.7.17-1.el7.noarch.rpm
    ```

-   导入 GPG key：

    ```sh
    $ rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
    ```

-   安装 RabbitMQ：

    ```sh
    $ rpm -Uvh rabbitmq-server-3.7.17-1.el7.noarch.rpm
    ```

-   启动 RabbitMQ：

    ```sh
    $ systemctl start rabbitmq-server
    ```

-   查看 RabbitMQ 运行状态

    ```sh
    $ systemctl status rabbitmq-server
    ```

-   将 RabbitMQ 加入开机自启动：

    ```sh
    $ systemctl enable rabbitmq-server
    ```

**RabbitMQ 配置**

-   启用 RabbitMQ 网页管理插件

    ```sh
    $ rabbitmq-plugins enable rabbitmq_management
    ```

-   创建管理员用户并授权：

    ```sh
    $ rabbitmqctl add_user admin 你的密码
    $ rabbitmqctl set_user_tags admin administrator
    $ rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
    ```

在浏览器访问 [http://IP:15672](http://ip:15672/) 即可进入到 RabbitMQ 网页管理页面

## 部署 Airflow

### 通过 pip 安装 airflow 脚手架

安装之前需要设置一下临时环境变量 `SLUGIFY_USES_TEXT_UNIDECODE` ，不然，会导致安装失败，命令如下：

```sh
export SLUGIFY_USES_TEXT_UNIDECODE=yes
```

-   安装 airflow 脚手架:

    ```sh
    sudo pip install apache-airflow===1.10.4
    ```

    airflow 会被安装到 python3 下的 site-packages 目录下，完整目录为:`${PYTHON_HOME}/lib/python3.6/site-packages/airflow`，我的 airflow 目录如下所示：

    ```sh
    [root@n71 airflow]# pwd
    /usr/local/python3/lib/python3.7/site-packages/airflow
    ```

-   添加环境变量

    `vi ~/.bash_profile`

    ```sh
    # User specific environment and startup programs
    AIRFLOW_HOME=/home/airflow
    SITE_AIRFLOW_HOME=/usr/local/python3/lib/python3.7/site-packages/airflow
    PYTHON3=/usr/local/python3
    
    PATH=$PATH:$HOME/bin:$SITE_AIRFLOW_HOME/bin:$PYTHON3/bin
    
    export AIRFLOW_HOME
    export SITE_AIRFLOW_HOME
    export PATH
    export C_FORCE_ROOT=true 
    ```

    使修改后的使环境变量生效：`sudo source /etc/profile`

### Airflow 相关操作

-   执行 `airflow` 命令做初始化操作

    ```sh
    airflow
    ```

    airflow 会在刚刚的 `AIRFLOW_HOME` 目录下生成一些文件。当然，执行该命令时可能会报一些错误，可以不用理会！生成的文件列表如下所示：

    ```sh
    [root@n71 airflow]# ls
    airflow.cfg  logs  unittests.cfg
    ```

-   为 `airflow` 安装依赖模块

    ```sh
    pip install 'apache-airflow[postgres,celery,hive,rabbitmq]'
    ```

    airflow 的包依赖安装均可采用该方式进行安装，具体可参考 [airflow 官方文档](https://link.juejin.im/?target=https%3A%2F%2Fairflow.apache.org%2Finstallation.html)

### **遇到的坑以及解决方案**

-   **为 `airflow` 安装 [rabbitmq] 依赖模块报错**

    具体可以参看这个 `repositories` https://github.com/celery/librabbitmq

    **解决方案：**

    ```sh
    git clone https://github.com/celery/librabbitmq.git
    
    cd librabbitmq
    
    checkout 1.6
    
    yum install install autoconf automake libtool
    
    make install
    
    # 在安装依赖模块
    pip install 'apache-airflow[rabbitmq]'
    ```

-   **启动 webserver 组件时报错**

    错位如下：

    ```sh
    Error: 'python:airflow.www.gunicorn_config' doesn‘t exist
    ```

    或者是：

    ```sh
    FileNotFoundError: [Errno 2] No such file or directory: 'gunicorn': 'gunicorn'
    ```

    **解决方案：**

    ```sh
    # 1 :  添加软连接
    ln -s /usr/local/python3/bin/gunicorn /usr/bin/gunicorn
    
    # 2  :  添加 python 环境变量
    PYTHON3=/usr/local/python3
    PATH=$PATH:$HOME/bin:$SITE_AIRFLOW_HOME/bin:$PYTHON3/bin
    ```

-   **`airflow worker` 角色不能使用 root 启动**

    ==原因：不能用根用户启动的根本原因，在于 airflow 的 worker 直接用的 celery，而 celery 源码中有参数默认不能使用 ROOT 启动，否则将报错：==

    ```sh
        C_FORCE_ROOT = os.environ.get('C_FORCE_ROOT', False)
    
        ROOT_DISALLOWED = """\
        Running a worker with superuser privileges when the
        worker accepts messages serialized with pickle is a very bad idea!
    
        If you really want to continue then you have to set the C_FORCE_ROOT
        environment variable (but please think about this before you do).
    
        User information: uid={uid} euid={euid} gid={gid} egid={egid}
        """
    
        ROOT_DISCOURAGED = """\
        You're running the worker with superuser privileges: this is
        absolutely not recommended!
    
        Please specify a different user using the --uid option.
    
        User information: uid={uid} euid={euid} gid={gid} egid={egid}
        """
    ```

    **解决方案：**

    ```sh
    # 设置环境变量,  强制celery worker运行采用root模式
     export C_FORCE_ROOT=True
    ```

-   **`airflow remote worker log hostname` 问题**

    当 worker 节点不是跟 webserver 部署在同一台机器的时候，有时从 webserver 查看该 worker 节点日志，出现如下错误：

    ```go
    *** Log file isn't local.
    *** Fetching here: http://n73:8793/log/.../1.log
    *** Failed to fetch log file from worker. HTTPConnectionPool(host='kaimanas.serveriai.lt', port=8793): Max retries exceeded with url: /log/.../1.log (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f64da2fab38>: Failed to establish a new connection: [Errno 111] Connection refused',))
    ```

    n73 (或是其他) 不是 webserver 节点所在的 hostname

    **解决方案：**

    配置 worker 节点的 `/etc/hosts` 的 hostname 映射，把 worker 节点的 ip 映射为本机的 hostname，如下：

    ```sh
    192.168.50.71 n71
    192.168.50.72 n72
    192.168.50.73 n73
    ```

## 配置 airflow.cfg

-   修改 Executor 为 CeleryExecutor

    ```
    executor = CeleryExecutor
    ```

-   指定元数据库（metestore)

    ```go
    sql_alchemy_conn = postgresql+psycopg2://postgres:postgres@192.168.50.73:5432/airflow
    ```

-   设置中间人（broker)

    ```go
    broker_url = amqp://admin:datatom.com@192.168.50.73:5672/
    ```

-   设定结果存储后端 backend

    ```go
    celery_result_backend = db+postgres://postgres:postgres@192.168.50.73:5432/airflow
    result_backend = db+postgres://stork:stork@192.168.50.73:14103/airflow
    ```

-   置 dags 初始化后为暂停状态(启动状态)

    ```go
    dags_are_paused_at_creation = True(False)
    ```

-   不引用实例脚本

    ```go
    load_examples = False
    ```

-   当定义的 dag 文件过多的时候，airflow 的 scheduler 节点运行效率缓慢

    ```go
    [scheduler]
    # The scheduler can run multiple threads in parallel to schedule dags.
    # This defines how many threads will run.
    #默认是2这里改为50
    max_threads = 30
    
    worker_concurrency = 5
    worker_max_tasks_per_child = 10
    ```

-   修改检测新 DAG 间隔，如果 scheduler 检测 DAG 过于频繁，会导致 CPU 负载非常高。而默认 scheduler 检测时间为 0，也就是没有时间间隔

    ```go
    min_file_process_interval = 5
    ```

-   定期刷新 DAG 定义目录中的文件列表，默认300s

    ```go
    dag_dir_list_interval = 10
    ```

-   Airflow 的 DAG 并行度控制

    `dag_concurrency`：表示一个 DAG，在同一时间点最大可以运行多少个 Task
    `max_active_runs_per_dag`：表示一个 DAG，在同一时间点最多可以被运行几个

    ```go
    # 默认值都为16
    dag_concurrency = 16
    max_active_runs_per_dag = 16
    ```

    