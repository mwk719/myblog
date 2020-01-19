---
layout: post
title: nginx.conf相关配置
date: 2018-09-03
Author: minweikai
tags: [nginx]
comments: true
---

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	#types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;
	gzip_disable "msie6";

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##
	
	server {
		listen 8001;
		server_name localhost 127.0.0.1;
        #允许通过请求后缀
		location ~.*\.(gif|jpg|jpeg|png|html|css|js|ico|ttf|woff|xlsx|mp3|pcm|mp4|php|py|xls|xlsx|doc|docx|txt)$ {
        expires 24h;
            root /home/gas/html/;#指定图片存放路径
            index index.html index.htm index.php;  # 首页匹配

        }
	}
	
	server {
		listen 80;
		server_name localhost 127.0.0.1;
		location ~.*\.(gif|jpg|jpeg|png|html|css|js|ico|ttf|woff|xlsx|mp3|pcm|mp4|php|py|xls|xlsx|doc|docx|txt)$ {
        expires 24h;
            root /home/gas/html/;#指定图片存放路径
            index index.html index.htm index.php;  # 首页匹配

        }
		#反向代理端口，访问 127.0.0.1/api/
		location /api/ {
			proxy_pass http://172.18.196.230:8077/;
			client_max_body_size 10m;
		}
		
        #反向代理端口，访问 127.0.0.1/hle/
		location /hle/ {
             proxy_pass http://172.18.196.230:8067/;
			client_max_body_size 10m;
        }

        #访问输密码验证
        location / {
            proxy_pass http://172.18.196.230:5601$request_uri;
            client_max_body_size 10m;
            auth_basic "登陆验证";
            auth_basic_user_file /etc/nginx/htpasswd;   #/etc/nginx/htpasswd是密码文件，路径自定义
        }


	}

	include /etc/nginx/conf.d/*.conf;
}

```

