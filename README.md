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
    
    
    
