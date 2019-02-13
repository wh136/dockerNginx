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


    
