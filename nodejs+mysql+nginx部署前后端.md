# 一、云服系统

CentOS 7.6 64位

# 二、安装node

1、安装node

```shell
sudo yum install epel-release sudo yum install nodejs -y
```

2、没有npm则安装npm

```shell
sudo yum install nodejs -y
```

3、检查安装是否完成

```shell
node -v
npm -v
```

# 三、安装mysql

1.mysql下载
①进入mysql官方网站：https://www.mysql.com/ 点击进入DOWNLOADS下载页面

![image-20240228141304502](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240228141304502.png)

②进入下载页面后，找到mysql社区版本MySQL Community (GPL) Downloads点击进入

![image-20240228141344145](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240228141344145.png)

③进入社区版页面后，点击MySQL Community Server

![image-20240228141418672](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240228141418672.png)

④进入后，点击Archives进入mysql版本归档页面

![image-20240228141514589](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240228141514589.png)

⑤进入mysql版本归档页面后选择你所需要的mysql版本、操作系统及系统版本，然后点击download即可下载

![image-20240228141650610](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240228141650610.png)

2.mysql安装规范
　　MySQL安装方式：推荐使用二进制安装（其他安装方式：源码编译安装、yum、rpm）

　　MySQL运行用户：mysql:mysql注意该用户是虚拟用户，只是用于mysql进程运行使用，不允许登录、不创建家目录

　　MySQL目录规范：

　　下载目录/server/tools

　　系统目录/opt/mysql/mysql-xx.xx

　　软连接ln-s/opt/mysql/mysql-xx.xx /usr/local/mysql

　　数据目录/data/mysql/mysql+port/{data,logs}

　　配置文件/data/mysql/mysql+port/my+port.cnf

3.二进制安装MySQL5.7.26（该方式使用于在linux系统下安装MySQL5.7和MySQL8.0的各个小版本）

创建MySQL虚拟用户

```
useradd -s /sbin/nologin -M mysql #创建用户命令
id mysql #查看是否创建成功
```

![image-20240228142224822](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240228142224822.png)

创建目录

```
mkdir -p /server/tools
mkdir -p /opt/mysql
mkdir -p /data/mysql/mysql3306/{data,logs}
cd /server/tools #进入到该目录
```

将下载的文件上传到上面tools目录（也可以复制下载链接之后执行wget命令）

```
rz #上传mysql二进制文件
ll #查看文件是否上传成功
yum install -y lrzsz #如没有rz命令，可通过yum安装
```

解压二进制包　　

```
tar xf mysql-5.7.44-linux-glibc2.12-x86_64.tar.gz #tar xf '下载的文件名'
```

将软件部分移动到指定目录

```
mv mysql-5.7.44-linux-glibc2.12-x86_64  /opt/mysql/mysql-5.7.44
```

创建软连接

```
ln -s /opt/mysql/mysql-5.7.44/ /usr/local/mysql
```

删除mariadb(避免与MySQL冲突)　

```
rpm -e --nodeps mariadb-libs
```

配置文件整理（该配置参数只用于测试环境，不可在生产中使用。配置参数影响着MySQL数据库的性能及安全，慎重！！！）(在/data/mysql/mysql3306/生成my3306.cnf文件并写入下面内容)

```
[client]
default-character-set=utf8mb4
port= 3306
socket= /tmp/mysql.sock
[mysqld]
server-id = 2020021325
user = mysql
port = 3306
basedir = /usr/local/mysql
datadir = /data/mysql/mysql3306/data
socket = /data/mysql/mysql3306/mysql.sock
pid-file = /data/mysql/mysql3306/mysql.pid
event_scheduler = 0
lower_case_table_names = 1
character-set-server = utf8mb4
transaction-isolation = REPEATABLE-READ
#skip_name_resolve 
max_connect_errors = 100000
max_connections = 2048 
skip-external-locking
max_allowed_packet = 1G
sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER"
wait_timeout = 3600
interactive_timeout = 3600
explicit_defaults_for_timestamp = 1 
secure_file_priv = /tmp
slow_query_log=1
slow_query_log_file =/data/mysql/mysql3306/logs/mysql-slow.log
long_query_time = 0.3
log-error=/data/mysql/mysql3306/logs/mysql-error-log.err
log_error_verbosity = 3
log_queries_not_using_indexes = 1
log_slow_admin_statements = 1
log_slow_slave_statements = 1
log_throttle_queries_not_using_indexes = 10
expire_logs_days = 7
min_examined_row_limit = 100
log_timestamps=system 
plugin_dir=/usr/local/mysql/lib/plugin
plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
[mysqld_safe]
log-error=/data/mysql/mysql3306/logs/mysql-error-log.err
pid-file=/data/mysql/mysql3306/mysql.pid
```

安装MySQL依赖包　　

```
yum install libaio-devel -y
yum install numactl -y
```

更改MySQL相关目录的用户组

```
chown -R mysql:mysql /data/*
```

初始化数据库

```
/usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql3306/my3306.cnf --initialize-insecure --user=mysql --basedir=/usr/local/mysql  --datadir=/data/mysql/mysql3306/data
```

（MySQL8.0 、MySQL5.7都是通过mysqld进行初始化数据）

--initialize-insecure ：表示不给默认root用户创建密码，可以空密码登陆

加入环境变量

```
vim /etc/profile #vim编辑

export PATH="/usr/local/mysql/bin:$PATH"



source /etc/profile #执行脚本文件，使得环境变量生效
```

启动MySQL　　

```
/usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql3306/my3306.cnf  &
```

查看是否启动成功　

```
netstat -lntup |grep mysql #如有指定的mysql进程，就表示mysql启动成功
```

连接mysql
#初始化时没有给root用户指定密码，所以可以空密码连接

```
mysql -uroot -p -S /data/mysql/mysql3306/mysql.sock
```

为消除每次登录时要连接mysql.sock，建立一下软连接

```
ln -s /data/mysql/mysql3306/mysql.sock /tmp/mysql.sock
```

使用如下代码进入mysql root即可

```
mysql -uroot
```

重启mysql

```
#进程中已经有mysql进程
#先kill掉再重启
#查看是否有mysql进程
ps -ef|grep mysqld
kill -9 进程号
#重启
/usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql3306/my3306.cnf  &
```



# 四、安装nginx

nginx下载官网查看最新稳定版

[nginx: download](https://nginx.org/en/download.html)

![image-20240228150340183](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240228150340183.png)

1、下载后将安装包上传到/usr/local目录下

2、wget直接下载

```
cd /usr/local
wget http://nginx.org/download/nginx-1.24.0.tar.gz
```

解压

```
tar -zxvf nginx-1.24.0.tar.gz
```

安装前准备
安装前先安装nginx所需要的依赖库，如果缺少依赖库，可能会安装失败

```
yum install gcc-c++ -y
yum install pcre
yum install pcre-devel -y
yum install zlib -y
yum install zlib-devel -y
yum install openssl -y
yum install openssl-devel -y
```

进入nginx-1.24.0目录 ，并执行以下配置命令

```
cd nginx-1.24.0
./configure     #执行configure文件
```

​    configure操作会检测当前系统环境，以确保能成功安装nginx，如果出错，请检查上述安装前依赖包是否已经安装。

执行make安装

注意：下面2步会将nginx安装到/usr/local/nginx目录下，所以请勿占用nginx目录名

```
#make
make install
```

配置nginx开机启动
切换到/lib/systemd/system/目录，创建nginx.service文件

```
cd /lib/systemd/system/
vim nginx.service
```

添加如下内容

```
[Unit]

Description=nginx

After=network.target

[Service]

Type=forking

ExecStart=/usr/local/nginx/sbin/nginx

ExecReload=/usr/local/nginx/sbin/nginx reload

ExecStop=/usr/local/nginx/sbin/nginx quit

PrivateTmp=true

[Install]

WantedBy=multi-user.target
```

退出并保存文件，执行如下命令，让nginx开机启动

```
systemctl start nginx.service
```

```
#启动nginx主进程

 systemctl start nginx.service

#结束nginx

 systemctl stop nginx.service

#重启nginx

 systemctl restart nginx.service
```

检验是否安装成功

​    在浏览器访问服务器公网ip

安装后nginx一些路径如下

```
nginx位置：/usr/local
nginx网站根目录：/usr/local/nginx/html
nginx配置文件：/usr/local/nginx/conf/nginx.conf
```

配置nginx全局可用

```
vim /etc/profile #vim编辑

export PATH=/usr/local/nginx/sbin:$PATH



source /etc/profile #执行脚本文件，使得环境变量生效
```

nginx命令子进程

```
重启nginx服务
nginx -s reload
停止nginx服务
nginx -s stop
查看nginx进程 
ps -ef|grep nginx 
关闭nginx进程 
kill -9 nginx
```

# 五、部署前端项目

将打包好的dist文件夹上传到云服务器中，建议将dist文件夹上传到nginx位置：/usr/local/nginx

修改nginx配置文件：/usr/local/nginx/conf/nginx.conf

```

user  root;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   dist;
            index  index.html index.htm;
	        try_files $uri $uri/ /index.html;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

```
难点：配置时一直遇到cors跨域问题，但是服务端配置了cors库，一直没弄懂为啥，
此时发现nginx界面刷新后会404，搜索后为下面结果，在添加try_files $uri $uri/ /index.html;后，解决刷新404问题，此时跨域问题也同时消失！
```

![image-20240301153438992](C:\Users\22297\AppData\Roaming\Typora\typora-user-images\image-20240301153438992.png)

重启nginx服务即可

```
nginx -s reload
```

访问公网ip即可进入前端项目

# 六、部署node项目

建立文件夹存放node项目

```
mkdir -p /usr/local/nginx/node
```

将node项目上传到服务器

```
/usr/local/nginx/node #node项目地址
```

由于云服务器ssh连接在关闭窗口时会断开，并且node项目启动后无法进行其他操作，因此安装screen后台运行node项目

```
yum install screen -y
```

screen后台运行node项目（退出窗口时按ctrl+a+d）

```
#创建后台窗口node
screen -S node
#进入node项目地址
cd /usr/local/nginx/node
#安装所需库
npm install
#启动node项目
node app.js
```

```
#关闭所有screen后台窗口
screen -ls | grep [session_id] | cut -d. -f1 | awk '{print $1}' | xargs kill
#显示所有窗口
screen -ls
#进入特定窗口
screen -R 窗口名
#关闭特定窗口
screen -X -S 窗口名 quit
```

# 七、部署mysql

建立文件夹存放sql文件，并上传sql文件

```
mkdir -p /usr/local/nginx/sql
```

进入之前创建的root用户

```
mysql -uroot
```

导入上传的sql文件

```
#先创建一个空数据库(此处创建的数据库名任意）
create database btfall;
#查看数据库中的数据库
show databases;
#选中该数据库
use btfall
source /usr/local/nginx/sql/file.sql;
#下面代码即可查看导入是否成功
use btfall
show tables;
#退出mysql
quit
```

导出数据库

```
mysqldump -u root -p 数据库名 > /usr/local/nginx/sql/file.sql
#即可在/usr/local/nginx/sql目录看到导出的file.sql文件
```

创建新角色并授权

```
create user 'huang'@'localhost' identified by 'huang';
grant all on btfall.* to 'huang'@'localhost';
```

