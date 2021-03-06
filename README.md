# dockerNginx
H3C VDI CloudClass  外网网站问题定位

## 1. 问题描述：位于紫光云主机的Nginx容器间隔大约2~5天的时间会自动挂掉，且无nginx日志可以查看.
## 问题复现
## 1.1 首先在主机上生成一个nginx容器
    docker run -it --name=nginxWiki -v /dockerNginx/conf.d:/etc/nginx/conf.d -d nginx
## 1.2 然后重启主机,可以看到Exited (255) 错误码255出现
    docker ps
    CONTAINER ID        IMAGE          COMMAND                  CREATED             STATUS                   PORTS     NAMES
    adda5b6e256a        nginx          "nginx -g 'daemon of…"   40 minutes ago      Exited (255) 1h ago      80/tcp    nginxPpt 
## 1.3 结论：由于主机状态不稳定导致容器停止，且容器没有自动重启。
## 1.4 解决方法：由nginx镜像运行一个容器的时候添加 --restart=always 并且把Nginx日志挂载到宿主机
# 2.1 生成一个Nginx容器的正确操作
    把Nginx日志挂载到宿主机 
    宿主机反向代理配置文件 /dockerNginxConf/conf
    宿主机日志挂载点/root/nginxLogs
    
    docker run -it --restart=always --name=nginxReverseProxy -p 80:80 -v /dockerNginx/conf:/etc/nginx/conf.d -v /root/nginxLogs:/var/log/nginx -d nginx
    docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
    e6badc25c692        nginx               "nginx -g 'daemon of…"   3 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp   nginxReverseProxy


## 3. 修改主机登陆端口和密码的方法

## 4. docker Minio设置方法
    
## 5. ppt网站Nginx手动编译，（容器化方式） 
    首先下载Nginx
    http://nginx.org/en/download.html
    解压并替换nginx.conf（根据需要编写nginx.conf） 在conf目录下可以找到
    (1)安装Nginx依赖 
        GCC 编译环境
        PCRE(Perl Compatible Regular Expressions) nginx 的 http 模块使用 pcre 来解析正则表达式
        zlib  nginx 使用 zlib 对 http 包的内容进行 gzip
        openssl 支持 https
    yum install -y gcc gcc-c++ make libtool
    yum install -y pcre pcre-devel
    yum install -y zlib zlib-devel
    yum install -y openssl openssl-devel
    yum install gd-devel
    
    
    进入解压目录输入
    ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_image_filter_module 
    参数说明:
    --prefix 用于指定nginx编译后的安装目录
    --add-module 为添加的第三方模块，此次添加了fdfs的nginx模块
    -with..._module 表示启用的nginx模块，如此处启用了http_ssl_module模块
    ./nginx -t 
    make && make install
    (2)
    ln -s /opt/demo/nginx/sbin/nginx /usr/bin/nginx
    nginx start(如需开机自启，可在/etc/rc.d/rc.local 文件中添此命令)
    –prefix 
    指定部署根目录，默认是/usr/local/nginx.此设置会更改其他配置目录的相对路径 
    –sbin-path 
    可执行文件的路径，默认为/sbin/nginx 
    --sbin-path=path  设置nginx可执行文件的名字，默认是prefix/sbin/nginx。
    –conf-path 
    配置文件的路径，默认为/conf/nginx.conf 
    –pid-path 
    pid文件的存放路径，默认存放在/logs/nginx.pid，是一个存放nginx的master进程ID的纯文本文件，刚安装的时候不会生成，nginx启动的时候会自动生成。 
    –http-log-path 
    access日志存放位置，每个http的请求在结束的时候都会访问的日志。 
    –with-ld-opt 
    加入第三方链接时需要的参数。编译之后nginx最终的可执行二进制文件是由编译后的目标文件和一些第三方的库链接生成的。如果想要将某个库链接到nginx中，就需要指定–with-ld-opt=目标库名-目标库路径 
    –with-debug 
    将nginx需要打印debug调试级别日志的代码编译进nginx，这样才可以通过修改配置文件将调试日志打印出来，便于定位服务问题 

    https://stackoverflow.com/questions/40574866/docker-nginx-ngx-http-image-filter-module
    
## 定制 Docker镜像
    参考资料 
    https://zhang.ge/5126.html
    开始操作
    $ docker pull nginx
    $ docker run -it --name="vdi-onlineshare" -d nginx  /bin/bash
    docker run --help 查看详细参数信息
    -it :　进行交互式终端操作  -i	--interactive  -t, --tty=false    Allocate a pseudo-TTY
    在Unix术语, 终端(terminal)=tty=文本的输入输出环境, 控制台(console)=物理终端, shell=命令行解释器
    -d :　--daemon 守护进程模式，等同于 -d=true,容器将会在后台运行，不然执行一次命令后，退出后，便是exit状态了。
    --name : 容器启动后的名字，默认不指定，将会随机产生一个名字。或者使用 -name="containers_name" 
    $ docker exec -ti vdi-onlineshare /bin/bash
    root@4a0e60861950:/# apt-get update
    root@4a0e60861950:/# apt-get install vim
    root@4a0e60861950:/# exit
    $ docker commit vdi-onlineshare vdi-nginx:v1.0
    关于vdi-nginx 1.0
        apt-get update
        apt-get vim
        修改/etc/nginx/nginx.conf 增加 load_module /etc/nginx/modules/ngx_http_image_filter_module.so;
        
    $ docker login
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    vdi-nginx           v1.0                382ae0f24237        2 hours ago         156MB
    nginx               latest              f09fe80eb0e7        11 days ago         109MB
    $ docker tag 382ae0f24237 wh136/vdi-nginx:v1.0
    root@ppt:~# docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    vdi-nginx           v1.0                382ae0f24237        2 hours ago         156MB
    wh136/vdi-nginx     v1.0                382ae0f24237        2 hours ago         156MB
    nginx               latest              f09fe80eb0e7        12 days ago         109MB

    docker push 注册用户名/镜像名
    $ docker push wh136/vdi-nginx:v1.0
### 基于ubuntu定制 golang revel的镜像
    docker pull ubuntu 16.04
    vim Dockerfile 参考ppt项目的Dockerfile
    docker build -t="revel:v1" .
    -t  revel:v1  给新构建的镜像取名为revel， 并设定版本为 v1.0 。
    
    
    
### 云学堂全国大屏遭受redis攻击报错定位
    redis被攻击之后无法登录大屏
    docker logs -f xxx 查看docker 日志
    1:M 17 Mar 01:58:55.036 * Background saving started by pid 5210
    5210:C 17 Mar 01:58:55.036 # Failed opening the RDB file root (in server root dir /etc/crontabs) for saving: Permission denied
    1:M 17 Mar 01:58:55.136 # Background saving error
    CONFIG GET 命令用于取得运行中的 Redis 服务器的配置参数(configuration parameters)
    redis> CONFIG GET *
    1) "dir"
    2) "/var/lib/redis"
    3) "dbfilename"
    4) "dump.rdb"
    5) "requirepass"
    6) (nil)
    7) "masterauth"
    8) (nil)
    9) "maxmemory"
    10) "0"
    11) "maxmemory-policy"
    12) "volatile-lru"
    13) "maxmemory-samples"
    14) "3"
    15) "timeout"
    16) "0"
    17) "appendonly"
    18) "no"
    # ...
    49) "loglevel"
    50) "verbose"
    执行
    redis-cli config get dir
    发现被攻击之后的redis为
    /etc/crontabs # redis-cli config get dir
    1) "dir"
    2) "/etc/crontabs"
    删除改该容器，重新生成redis容器，
    再次执行
    redis-cli config get dir
    /data # redis-cli config get dir
    1) "dir"
    2) "/data"
    通过查找redis攻击发现了该文章中攻击redis的原理
    新的攻击通过在内存中设置一个恶意的键值对并将其作为文件保存在强制服务器执行文件的/etc/crontabs文件夹中
    https://www.cnblogs.com/hacker520/p/9140222.html
    
    vi /etc/crontabs/root
    # do daily/weekly/monthly maintenance
    # min   hour    day     month   weekday command
    */15    *       *       *       *       run-parts /etc/periodic/15min
    0       *       *       *       *       run-parts /etc/periodic/hourly
    0       2       *       *       *       run-parts /etc/periodic/daily
    0       3       *       *       6       run-parts /etc/periodic/weekly
    0       5       1       *       *       run-parts /etc/periodic/monthly
    （疑问，为什么防火墙已经开启了，但是依然能链接到redis ???）
    root@yxt-web:~# redis-cli -h 111.166.23.99 -p 6379
    111.166.23.99:6379> 
    
    附：用RDB方式攻击redis的方法 （需要翻墙才能看到该文章）：
    RDB方式备份数据库的文件名默认为dump.rdb，此文件名可以通过客户端交互动态设置dbfilename来更改，造成可以写任意文件.
    https://www.00theway.org/2017/03/27/redis_exp/
    执行命令反弹shell(写计划任务时会覆盖原来存在的用户计划任务).写文件之前先获取dir和dbfilename的值，以便恢复redis配置，将改动降到最低，避免被发现。
    
    http://www.zuidaima.com/share/2643146230549504.htm
    //只允许127.0.0.1访问6379
    iptables -A INPUT -s 127.0.0.1 -p tcp --dport 6379 -j ACCEPT
    //其他ip访问全部拒绝
    iptables -A INPUT -p TCP --dport 6379 -j REJECT
    
    生活最主要的还是感受，坚持是一种刻意的练习，不断寻找缺点突破缺点的过程，而不是重复做某件事情。
    


    
    
    
    
    
