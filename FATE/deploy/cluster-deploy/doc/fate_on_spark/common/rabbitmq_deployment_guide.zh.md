## part1. 环境准备

[Erlang 官方下载地址](https://www.erlang.org/downloads)  [RabbitMQ 官方下载地址](https://www.rabbitmq.com/download.html)

```bash
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.10.7/rabbitmq-server-generic-unix-3.10.7.tar.xz
wget https://github.com/erlang/otp/releases/download/OTP-25.0.3/otp_src_25.0.3.tar.gz
```

## part2. 安装Erlang

* 下载Erlang源码(otp_src_19.3.tar.gz)，并解压至/data/projects/fate/common

  ```bash
  tar -zxvf otp_src_25.0.3.tar.gz -C /data/projects/fate/common
  ```

* 配置ERL_TOP

  ```bash
  cd /data/projects/fate/common/otp_src_25.0.3
  export ERL_TOP=`pwd`
  ```

* 编译

  使用以下命令编译:

  ```bash
  ./configure --prefix=/data/projects/fate/common/erlang
  make
  make install
   ```

  如果出现 **No curses library functions found**报错，则需要安装ncuress，先下载ncurses-6.0.tar.gz

  ```bash
  sudo dnf -y install ncurses-devel
  # 或者
  tar -zxvf ncurses-6.0.tar.gz
  cd ncurses-6.0
  ./configure --with-shared --without-debug --without-ada --enable-overwrite  
  make
  make install (如果报Permission denied，则需要root权限执行)
  ```

* 设置环境变量

  编译完成后，设置 ERL_HOME。编辑 /etc/profile 文件，增加以下内容：

  ```bash
  cat >> /etc/profile << EOF
  export ERL_HOME=/data/projects/fate/common/erlang
  export PATH=$PATH:/data/projects/fate/common/erlang/bin
  EOF
  ```

* 验证

  执行命令: erl, 可以进入Erlang环境，则安装成功；

## part3. 安装RabbitMQ

* **下载RabbitMq Server安装包，并解压至/data/projects/fate/common**

  ```bash
  tar xz -d rabbitmq-server-generic-unix-3.6.15.tar.xz
  tar xvf rabbitmq-server-generic-unix-3.6.15.tar
  ```

* **启动单机RabbitMQ，生成cookie**

  ```bash
  cd /data/projects/fate/common/rabbitmq_server-3.6.15 && ./sbin/rabbitmq-server -detached
  ```

* **修改权限**

把cookie文件的权限设置为400：

  ```bash
  chmod -R 400 .erlang.cookie
  ```

* **集群部署：**

1. 多台机器都需要按照上面方式做一遍；
2. 同步cookie文件：
3. 按照以上方式安装的话，cookie位于/home/app/.erlang.cookie
4. 将集群中的任意一台机器上的cookie文件拷贝到其他机器上进行替换

集群启动：

+ 以mq1为基

  停mq2、mq3

   ```
   sbin/rabbitmqctl stop
   ```

+ 启动mq2、mq3

   ```
   sbin/rabbitmq-server -detached
   ```

+ 停mq2、mq3应用

   ```bash
   sbin/rabbitmqctl stop_app
   ```

+ 将mq2、mq3加到mq1中

  在mq2、mq3上执行

  ```bash
  sbin/rabbitmqctl join_cluster rabbit@mq1
  ```

+ 启动mq2、mq3应用

## part4. rabbitmq配置

+ 确认集群状态

  ```bash
  rabbitmqctl cluster_status
  ```

+ 启动federation服务（enable/disable）:

  ```bash
  rabbitmq-plugins enable rabbitmq_management
  rabbitmq-plugins enable rabbitmq_federation
  rabbitmq-plugins enable rabbitmq_federation_management  
  ```

+ 添加用户

  ```bash
  rabbitmqctl add_user fate fate
  ```

+ 添加角色：

  ```bash
  rabbitmqctl set_user_tags fate administrator
  ```

+ 设置权限：

  ```bash
  rabbitmqctl set_permissions -p / fate ".*" ".*" ".*" 
  ```