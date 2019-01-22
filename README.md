# dockerNginx
how to generate a container of nginx for Reverse Proxy. For  H3C VDI CloudClass.

# git CMD for project
    …or create a new repository on the command line
    echo "# dockerNginx" >> README.md
    git init
    git add README.md
    git commit -m "first commit"
    git remote add origin https://github.com/wh136/dockerNginx.git
    git push -u origin master
    …or push an existing repository from the command line
    git remote add origin https://github.com/wh136/dockerNginx.git
    git push -u origin master
# STATUS 出现Exited (255) 1h ago复现
    由nginx镜像运行一个容器的时候没有加 --restart=always 由于主机状态不稳定导致容器停止，并且没有日志提示
    CONTAINER ID        IMAGE          COMMAND                  CREATED             STATUS                   PORTS     NAMES
    adda5b6e256a        nginx          "nginx -g 'daemon of…"   40 minutes ago      Exited (255) 1h ago      80/tcp    nginxPpt  
## 1. 首先在主机上生成一个nginx容器
    docker run -it --name=nginxWiki -v /dockerNginx/conf.d:/etc/nginx/conf.d -d nginx
## 2. 然后重启主机,可以看到Exited (255) 错误码255出现
    docker ps
    CONTAINER ID        IMAGE          COMMAND                  CREATED             STATUS                   PORTS     NAMES
    adda5b6e256a        nginx          "nginx -g 'daemon of…"   40 minutes ago      Exited (255) 1h ago      80/tcp    nginxPpt  
# 生成一个Nginx容器的正确操作
    把Nginx日志挂载到宿主机 
    宿主机反向代理配置文件 /dockerNginxConf/conf
    宿主机日志挂载点/root/nginxLogs
    
    docker run -it --restart=always --name=nginxReverseProxy -p 80:80 -v /dockerNginx/conf:/etc/nginx/conf.d -v /root/nginxLogs:/var/log/nginx -d nginx
    docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
    e6badc25c692        nginx               "nginx -g 'daemon of…"   3 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp   nginxReverseProxy



