安装

http://nginx.org/en/linux_packages.html#RHEL-CentOS



sudo yum install yum-utils



编辑/etc/yum.repos.d/nginx.repo，内容如下：

[nginx-stable]

name=nginx stable repo

baseurl=http://nginx.org/packages/centos/$releasever/$basearch/

gpgcheck=1

enabled=1

gpgkey=https://nginx.org/keys/nginx_signing.key

module_hotfixes=true



[nginx-mainline]

name=nginx mainline repo

baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/

gpgcheck=1

enabled=0

gpgkey=https://nginx.org/keys/nginx_signing.key

module_hotfixes=true



sudo yum install nginx



常用命令

nginx：启动 Nginx

nginx -s stop：立刻停止 Nginx 服务

nginx -s reload：重新加载配置文件

nginx -s quit：平滑停止 Nginx 服务

nginx -t：测试配置文件是否正确

nginx -v：显示 Nginx 版本信息

nginx -V：显示 Nginx 版本信息、编译器和配置参数的信息



配置文件（负载均衡）

[root@localhost nginx]# cat /etc/nginx/nginx.conf



user  nginx;

worker_processes  1;



error_log  /var/log/nginx/error.log warn;

pid        /var/run/nginx.pid;





events {

​    worker_connections  1024;

}





http {

​    include       /etc/nginx/mime.types;

​    default_type  application/octet-stream;



​    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '

​                      '$status $body_bytes_sent "$http_referer" '

​                      '"$http_user_agent" "$http_x_forwarded_for"';



​    access_log  /var/log/nginx/access.log  main;



​    sendfile        on;

​    \#tcp_nopush     on;



​    keepalive_timeout  65;



​    \#gzip  on;



​    include /etc/nginx/conf.d/*.conf;

   

​    upstream api.blog.com {

​        server 192.168.1.108:8001;

​        server 192.168.1.108:8002;

​    }

 

​    server {

​        listen  8081;

​        server_name  api.blog.com;

​        

​        location / {

​            proxy_pass http://api.blog.com/;

​        }

​    

​    }

}