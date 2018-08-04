#### 一、安装Nginx：

###### 1 : wget下载

		mkdir -p /usr/local/software/
		cd /usr/local/software/
		wget http://nginx.org/download/nginx-1.6.2.tar.gz

###### 2 : 解压文件, 进行安装：

		tar -zxvf /usr/local/software/nginx-1.6.2.tar.gz -C ../

###### 3 : 下载锁需要的依赖库文件：

		yum install pcre
		yum install pcre-devel
		yum install zlib
		yum install zlib-devel
		#不装opensll运行./configure时,可能会出现 "./configure: error: the HTTP cache module requires md5 functions from OpenSSL library."的错误信息
		yum -y install openssl openssl-devel

###### 4 : 进行configure配置：

		mkdir -p /usr/local/nginx
		cd /usr/local/nginx-1.6.2 && ./configure --prefix=/usr/local/nginx

查看是否报错

###### 5 : 编译安装

		make && make install

###### 6 : 启动Nginx：

到conf目录下： 看到如下4个目录

		cd /usr/local/nginx

		…conf 配置文件
		…html 网页文件
		…logs 日志文件
		…sbin 主要二进制程序

Nginx启动,关闭,重启命令：

		#查看nginx进行
		sudo ps -ef | grep nginx
		#开启
		/usr/local/nginx/sbin/nginx
		#关闭
		sudo killall nginx
		#重启
		sudo killall nginx
		/usr/local/nginx/sbin/nginx

成功：查看是否启动

		netstat -ano | grep 80

失败：可能为80端口被占用等。
最终：
浏览器访问地址：http://ip:80 （看到欢迎页面即可）

#### 二、使用Nginx：简单与单台Tomcat整合

 首先找到nginx.conf文件：vim /usr/local/nginx/conf/nginx.conf

		server {
		  listen  80;
		       server_name  localhost:80;
		       # 8080为Tomcat端口, 这里不做多介绍
		    location / {
		        proxy_pass http://localhost:8080；
		    }
		    //...others
		}

#### 三、详细使用（nginx就是去配置其文件而已），如下所示：
      #开启进程数 <=CPU数
      worker_processes 1;

      #错误日志保存位置
      #error_log logs/error.log;
      #error_log logs/error.log notice;
      #error_log logs/error.log info;

      #进程号保存文件
      #pid logs/nginx.pid;

      #等待事件
      events {
          #每个进程最大连接数（最大连接=连接数x进程数）
          #每个worker允许同时产生多少个链接，默认1024
          worker_connections 1024;
      }


      http {
          #文件扩展名与文件类型映射表
          include mime.types;
          #默认文件类型
          default_type application/octet-stream;
          #日志文件输出格式 这个位置相于全局设置
          #log_format main '$remote_addr - $remote_user [$time_local] "$request" '
          # '$status $body_bytes_sent "$http_referer" '
          # '"$http_user_agent" "$http_x_forwarded_for"';
          #请求日志保存位置
          #access_log logs/access.log main;
          #打开发送文件
          sendfile on;
          #tcp_nopush on;
          #连接超时时间
          #keepalive_timeout 0;
          keepalive_timeout 65;
          #打开gzip压缩
          #gzip on;
          #设定请求缓冲
          client_header_buffer_size 1k;
          large_client_header_buffers 4 4k;
          #设定负载均衡的服务器列表 (可配置集群)
          upstream myproject1 { 
              #weigth参数表示权值，权值越高被分配到的几率越大
              #max_fails 当有#max_fails个请求失败，就表示后端的服务器不可用，默认为1，将其设置为0可以关闭检查
              #fail_timeout 在以后的#fail_timeout时间内nginx不会再把请求发往已检查出标记为不可用的服务器
              #这里指定多个源服务器，ip:端口,80端口的话可写可不写
              server 192.168.1.78:8080 weight=5 max_fails=2 fail_timeout=600s;
              server 192.168.1.222:8080 weight=3 max_fails=2 fail_timeout=600s;
          }
          upstream myproject2 {
              server 192.168.1.78:8201 weight=5 max_fails=2 fail_timeout=600s;
          }

      #第一个虚拟主机
      server {
          #监听IP端口
          listen 80;
          #主机名
          server_name localhost;
          #设置字符集
          #charset koi8-r;
          #本虚拟server的访问日志 相当于局部变量
          #access_log logs/host.access.log main;
          #对本server"/"启用负载均衡
          location / {
              #root /root; #定义服务器的默认网站根目录位置
              #index index.php index.html index.htm; #定义首页索引文件的名称
              proxy_pass http://myproject; #请求转向myproject定义的服务器列表
              # 根据不同域名, 跳转到不同链接
              # 如: www.a.com域名 说跳转到myproject1的服务器列表(上面upstream中有配置)
              if ($http_host ~* "^www\.a\.com$") {
                 proxy_pass http://myproject1;
              }
              # 301重定向 页面永久性转移
              # 注意: if后面必须有空格 =号前后也必须有空格
              # 把 aa.top 301转向为 www.aa.com网站
              # 可以使用HTTP状态检测 链接为: http://tool.chinaz.com/pagestatus/ 返回状态码为301 即301重定向成功
              if ($host = 'aa.top' ) {
                 rewrite ^/(.*)$ http://www.aa.com/$1 permanent;
              }

              #if ($http_host ~* "^www\.b\.com\.cn$") {
              #   proxy_pass http://myproject2;
              #}


              #以下是一些反向代理的配置可删除.
              # proxy_redirect off;
              # proxy_set_header Host $host;
              # proxy_set_header X-Real-IP $remote_addr;
              # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              # client_max_body_size 10m; #允许客户端请求的最大单文件字节数
              # client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，
              # proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
              # proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
              # proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
              # proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
              # proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
              # proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
              # proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传
          }
          location /upload {
              alias e:/upload;
          }
          #设定查看Nginx状态的地址
          location /NginxStatus {
              stub_status on;
              access_log off;
              #allow 192.168.0.3;
              #deny all;
              #auth_basic "NginxStatus";
              #auth_basic_user_file conf/htpasswd;
          }
          #error_page 404 /404.html;
          # redirect server error pages to the static page /50x.html  
          # 定义错误提示页面
          error_page 500 502 503 504 /50x.html;
          location = /50x.html {
              root html;
          }
          # proxy the PHP scripts to Apache listening on 127.0.0.1:80
          #
          #location ~ \.php$ {
          # proxy_pass http://127.0.0.1;
          #}
          # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
          #
          #location ~ \.php$ {
          # root html;
          # fastcgi_pass 127.0.0.1:9000;
          # fastcgi_index index.php;
          # fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
          # include fastcgi_params;
          #}
          # deny access to .htaccess files, if Apache's document root
          # concurs with nginx's one
          #
          #location ~ /\.ht {
          # deny all;
          #}
      }

      server {
             listen 8201;
             charset UTF-8;
             #access_log  logs/cms-background.access.log  main;
             server_name localhost;
                location / {
                root  /data/www/project/cms/cms-background/dist;
                # 添加vue-route的跳转设置
                try_files $uri $uri/ @router;
                index  index.html index.htm;
            }
            location @router {
                rewrite ^.*$ /index.html last;
            }
        }

          # another virtual host using mix of IP-, name-, and port-based configuration
          #
          #server {
              #多监听
              # listen 8000;
              #主机名
              # listen somename:8080;
              # server_name somename alias another.alias;

              # location / {
                  #WEB文件路径
                  # root html;
                  #默认首页
                  # index index.html index.htm;
              # }
          #}

          # HTTPS server HTTPS SSL加密服务器
          #
          #server {
              # listen 443;
              # server_name localhost;

              # ssl on;
              # ssl_certificate cert.pem;
              # ssl_certificate_key cert.key;

              # ssl_session_timeout 5m;

              # ssl_protocols SSLv2 SSLv3 TLSv1;
              # ssl_ciphers HIGH:!aNULL:!MD5;
              # ssl_prefer_server_ciphers on;

              # location / {
                  # root html;
                  # index index.html index.htm;
              # }
          #}
      }

#### 四．配置tomcat集群负载均衡（这里我们之间配置2个tomcat到nginx里，然后进行测试）

#### 五．其他配置信息文件说明
参考博客1：http://blog.csdn.net/wave_1102/article/details/44475093 <br/>
参考博客2：http://blog.csdn.net/shimiso/article/details/8690897 <br/>
参考博客3：https://blog.csdn.net/hanwuqia0370/article/details/78151934 <br/>

#### 六: HTTP状态检测

检测http状态工具: 站长之家 <br/>
http状态码大全: 百度百科 <br/>
