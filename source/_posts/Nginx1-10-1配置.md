---
title: Nginx1.10.1配置
categories: Study
tags:
  - http
  - nginx
abbrlink: 990947c
date: 2018-10-26 10:14:53
---

nginx1.10.1版本配置信息详解<!--more-->


```
#普通配置
#==性能配置


#运行用户
user nobody;
#pid文件
pid logs/nginx.pid;
#==worker进程数，通常设置等同于CPU数量，auto为自动检测
worker_processes auto;
#==worker进程打开最大文件数，可CPU*10000设置
worker_rlimit_nofile 100000;
#全局错误日志
error_log  logs/error.log;


#events模块中包含nginx中所有处理连接的设置
events {
    #==worker进程同时打开的最大连接数，可CPU*2048设置
    worker_connections 2048;
    #==告诉nginx收到一个新链接通知后接受尽可能多的链接
    multi_accept on;
    #==设置用于复用客户端线程的轮训方法
    use epoll;
}


#http模块控制着nginx http处理的所有核心特性
http {
    #打开或关闭错误页面中的nginx版本号deng
    server_tokens on;
    #!server_tag on;
    #!server_info on;
    #==优化磁盘IO设置，指定nginx是否调用sendfile函数来输出文件，普通应用设为on，下载等磁盘IO高的应用，可设为off
    sendfile on;
    #==设置nginx在一个数据包里发送所有头文件，而不是一个接一个的发送
    tcp_nopush on;
    #==设置nginx不要缓存数据，而是一段一段的发送，
    #==当需要及时发送数据时，就应该给应用设置这个属性，这样发送一小块数据信息时就不能立即得到返回值
    tcp_nodelay on;

    #==设置nginx是否存储访问日志，关闭这个可以让读取磁盘IO操作更快
    access_log on;
    #设置nginx只记录严重错误
    #error_log logs/error.log crit;

    #==给客户端分配keep-alive链接超时时间
    keepalive_timeout 30;

    #设置用户保存各种key的共享内存的参数，5m指的是5兆
    limit_conn_zone $binary_remote_addr zone=addr:5m;
    #为给定的key设置最大的连接数，这里的key是addr，设定的值是100，就是说允许每一个IP地址最多同时打开100个连接
    limit_conn addr 100;

    #include指在当前文件中包含另一个文件内容
    include mime.types;
    #设置文件使用默认的mine-type
    default_type text/html;
    #设置默认字符集
    charset UTF-8;

    #==设置nginx采用gzip压缩的形式发送数据，减少发送数据量，但会增加请求处理时间及CPU处理时间，需要权衡
    gzip on;
    #==加vary给代理服务器使用，针对有的浏览器支持压缩，有个不支持，根据客户端的HTTP头来判断是否需要压缩
    gzip_vary on;
    #nginx在压缩资源之前，先查找是否有预先gzip处理过的资源
    #!gzip_static on;
    #为指定的客户端禁用gzip功能
    gzip_disable "MSIE[1-6]\.";
    #允许或禁止压缩基于请求和相应的响应流，any代表压缩所有请求
    gzip_proxied any;
    #==设置对数据启用压缩的最少字节数，如果请求小于10240字节则不压缩，会影响请求速度
    gzip_min_length 10240;
    #==设置数据压缩等级，1-9之间，9最慢压缩比最大
    gzip_comp_level 2;
    #设置需要压缩的数据格式
    gzip_types text/plain text/css text/xml text/javascript  application/json application/x-javascript application/xml application/xml+rss; 

    #==开发缓存的同时也指定了缓存文件的最大数量，20s如果文件没有请求则删除缓存
    open_file_cache max=100000 inactive=20s;
    #==指多长时间检查一次缓存的有效信息
    open_file_cache_valid 60s;
    #==文件缓存最小的访问次数，只有访问超过5次的才会被缓存
    open_file_cache_min_uses 5;
    #当搜索一个文件时是否缓存错误信息
    open_file_cache_errors on;

    #==允许客户端请求的最大单文件字节数
    client_max_body_size 8m;
    #==冲区代理缓冲用户端请求的最大字节数
    client_header_buffer_size 32k;

    #
    proxy_redirect off;
    #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #==nginx跟后端服务器连接超时时间(代理连接超时)
    proxy_connect_timeout 60;
    #==连接成功后，后端服务器响应时间(代理接收超时)
    proxy_read_timeout 120;
    #==后端服务器数据回传时间(代理发送超时)
    proxy_send_timeout 20;
    #==设置代理服务器（nginx）保存用户头信息的缓冲区大小
    proxy_buffer_size 32k;
    #==proxy_buffers缓冲区，网页平均在32k以下的设置
    proxy_buffers 4 128k;
    #==高负荷下缓冲大小（proxy_buffers*2）
    proxy_busy_buffers_size 256k;
    #==设定缓存文件夹大小，大于这个值，将从upstream服务器传
    proxy_temp_file_write_size 256k;
    #==1G内存缓冲空间，3天不用删除，最大磁盘缓冲空间2G
    proxy_cache_path /home/cache levels=1:2 keys_zone=cache_one:1024m inactive=3d max_size=2g;


    #设定负载均衡服务器列表
    upstream nginx.test.com{
        #后端服务器访问规则
        #ip_hash;
        #weight参数表示权重值，权值越高被分配到的几率越大
        #server 10.11.12.116:80 weight=5;
        #PC_Local
        server 10.11.12.116:80;
        #PC_Server
        server 10.11.12.112:80;
        #Notebook
        #server 10.11.12.106:80;
    }

    #虚拟主机设定模块（挂载多个站点，只需要配置多个server及upstream节点即可）
    server {
        #监听80端口
        listen 80;
        #定义使用nginx.test.com访问
        server_name nginx.test.com;
        #设定本虚拟主机的访问日志
        access_log logs/nginx.test.com.access.log;
        
        #默认请求
        # 语法规则：location [=|~|~*|^~] /uri/ {...} 先匹配普通location，在匹配正则location
        # = 开头表示精确匹配
        # ^~ 开头表示uri以某个常规字符串开头，理解为匹配url路径即可，无需考虑编解码
        # ~ 开头表示区分大小写的正则匹配
        # ~* 开头表示不区分大小写的正则匹配
        # !~ 开头表示区分大小写的不匹配的正则
        # !~* 开头表示不区分大小写的不匹配的正则
        # / 通用匹配，任何请求都会被匹配到
        location / {
            #定义服务器的默认网站根目录位置
            root html;
            #定义首页索引文件的名称
            index index.html index.htm;
            #定义后端负载服务器组
            proxy_pass http://nginx.test.com;
        }

        #定义错误提示页面
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root html;
        }
        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/{
            root /var/www/virtual/htdocs;
            #过期时间1天
            expires 1d;
            #关闭媒体文件日志
            access_log off;
            log_not_found off;
        }
        #设定查看Nginx状态的地址
        location /NginxStatus {
            #!stub_status on; #无此关键字
            access_log off;
            auth_basic "NginxStatus";
            auth_basic_user_file conf/htpasswd;
        }
        #禁止访问的文件.htxxx
        location ~ /\.ht {
            deny all;
        }
    }
    #网站较多的情况下ngxin又不会请求瓶颈可以考虑挂多个站点，并把虚拟主机配置单独放在一个文件内，引入进来
    #include website.conf;
}


```