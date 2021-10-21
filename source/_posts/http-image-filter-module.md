---
title: http_image_filter_module
date: 2021-10-21 15:02:41
tags: ['nginx']
cover: https://s3.bmp.ovh/imgs/2021/10/52fd491754edacb4.png
---

#### 在Windows下编译nginx，并添加图片压缩模块http_image_filter_module

nginx做图片服务非常简单，但是在window下给nginx添加模块，需要重新编译，记录一下步骤以及参考的文章

环境 nginx版本1.16  Windows10 64位

##### 1. 搭建编译环境

* 安装[VS2015](https://visualstudio.microsoft.com/zh-hans/vs/older-downloads/)
* nginx需要的依赖

>[openssl-1.0.1u](https://www.openssl.org/source/old/1.0.1/openssl-1.0.1u.tar.gz)
>[pcre-8.42](https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz)
>[zlib-1.2.11](https://sourceforge.net/projects/libpng/files/zlib/1.2.11/zlib-1.2.11.tar.gz/download?use_mirror=kent&download=)

##### 2. 编译libgd

[libgd](https://github.com/libgd/libgd/tree/master/windows)这里需要将libgd编译为一个libgd.dll 步骤可参考其中的md，当中需要的一些依赖也有给出

- Open the VS2015 x86 or x64 Native Tools Command Prompt.

这里用到了vs2015来编译，比较坑爹的地方就是在当时（2021-04）编译的时候仓库里的**Makefile.vc**文件加入了avif的项目，但是文件里面缺缺少相关的，拉取了之前的版本才得以解决。

需要说明的是，libgd.dll是可以在网络上找到的，没有亲自编译的必要（~~目前只在CSDN上看到有，链接就不放了~~）

将编译得到的的libgd.dll放到path里面

#### 3. 编译nginx

将三个依赖库放到nginx源码下的build/lib下面

生成makefile文件

~~~sh
auto/configure --with-cc=cl --builddir=build --with-cc-opt="-DFD_SETSIZE=1024 -I build/lib/gd/include" --with-ld-opt="build/lib/gd/lib/libgd.lib"  --with-debug --prefix= --conf-path=conf/nginx.conf  --pid-path=logs/nginx.pid --http-log-path=logs/access.log --error-log-path=logs/error.log --sbin-path=nginx.exe --http-client-body-temp-path=temp/client_body_temp --http-proxy-temp-path=temp/proxy_temp --http-fastcgi-temp-path=temp/fastcgi_temp --http-scgi-temp-path=temp/scgi_temp --http-uwsgi-temp-path=temp/uwsgi_temp  --with-pcre=build/lib/pcre-8.42 --with-zlib=build/lib/zlib-1.2.11 --with-select_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_stub_status_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module --with-http_slice_module --with-mail --with-stream --with-openssl=build/lib/openssl-1.0.1u --with-openssl-opt=no-asm --with-http_ssl_module --with-mail_ssl_module --with-stream_ssl_module --with-http_image_filter_module
~~~

这里比较容易报错的就是 checking for xxx ... not found 

打开makefile同级目录下的 /auto/configure 然后



~~~sh
#!/bin/sh

# Copyright (C) Igor Sysoev
# Copyright (C) Nginx, Inc.


LC_ALL=C
export LC_ALL

. auto/options
. auto/init
. auto/sources

test -d $NGX_OBJS || mkdir -p $NGX_OBJS

echo > $NGX_AUTO_HEADERS_H
echo > $NGX_AUTOCONF_ERR

echo "#define NGX_CONFIGURE \"$NGX_CONFIGURE\"" > $NGX_AUTO_CONFIG_H


if [ $NGX_DEBUG = YES ]; then
    have=NGX_DEBUG . auto/have
fi


if test -z "$NGX_PLATFORM"; then
    echo "checking for OS"

    NGX_SYSTEM=`uname -s 2>/dev/null`
    NGX_RELEASE=`uname -r 2>/dev/null`
    NGX_MACHINE=`uname -m 2>/dev/null`

    echo " + $NGX_SYSTEM $NGX_RELEASE $NGX_MACHINE"

    NGX_PLATFORM="$NGX_SYSTEM:$NGX_RELEASE:$NGX_MACHINE";

    case "$NGX_SYSTEM" in
        MINGW32_* | MINGW64_* | MSYS_*)
            NGX_PLATFORM=win32
        ;;
    esac

else
    echo "building for $NGX_PLATFORM"
    NGX_SYSTEM=$NGX_PLATFORM
fi

. auto/cc/conf

if [ "$NGX_PLATFORM" != win32 ]; then
    . auto/headers
fi

. auto/os/conf

if [ "$NGX_PLATFORM" != win32 ]; then
    . auto/unix
fi

. auto/threads
. auto/modules
. auto/lib/conf

case ".$NGX_PREFIX" in
    .)
        NGX_PREFIX=${NGX_PREFIX:-/usr/local/nginx}
        have=NGX_PREFIX value="\"$NGX_PREFIX/\"" . auto/define
    ;;

    .!)
        NGX_PREFIX=
    ;;

    *)
        have=NGX_PREFIX value="\"$NGX_PREFIX/\"" . auto/define
    ;;
esac

if [ ".$NGX_CONF_PREFIX" != "." ]; then
    have=NGX_CONF_PREFIX value="\"$NGX_CONF_PREFIX/\"" . auto/define
fi

have=NGX_SBIN_PATH value="\"$NGX_SBIN_PATH\"" . auto/define
have=NGX_CONF_PATH value="\"$NGX_CONF_PATH\"" . auto/define
have=NGX_PID_PATH value="\"$NGX_PID_PATH\"" . auto/define
have=NGX_LOCK_PATH value="\"$NGX_LOCK_PATH\"" . auto/define
have=NGX_ERROR_LOG_PATH value="\"$NGX_ERROR_LOG_PATH\"" . auto/define

have=NGX_HTTP_LOG_PATH value="\"$NGX_HTTP_LOG_PATH\"" . auto/define
have=NGX_HTTP_CLIENT_TEMP_PATH value="\"$NGX_HTTP_CLIENT_TEMP_PATH\""
. auto/define
have=NGX_HTTP_PROXY_TEMP_PATH value="\"$NGX_HTTP_PROXY_TEMP_PATH\""
. auto/define
have=NGX_HTTP_FASTCGI_TEMP_PATH value="\"$NGX_HTTP_FASTCGI_TEMP_PATH\""
. auto/define
have=NGX_HTTP_UWSGI_TEMP_PATH value="\"$NGX_HTTP_UWSGI_TEMP_PATH\""
. auto/define
have=NGX_HTTP_SCGI_TEMP_PATH value="\"$NGX_HTTP_SCGI_TEMP_PATH\""
. auto/define

. auto/make
. auto/lib/make
. auto/install

# STUB
. auto/stubs

have=NGX_USER value="\"$NGX_USER\"" . auto/define
have=NGX_GROUP value="\"$NGX_GROUP\"" . auto/define

if [ ".$NGX_BUILD" != "." ]; then
    have=NGX_BUILD value="\"$NGX_BUILD\"" . auto/define
fi

. auto/summary
~~~

仅供参考

#### 4. 使用vs2015 本机工具命令提示符执行makefile

~~~sh
nmake /f build\Makefile
~~~

一切正常的话就能生成nginx.exe文件，其他文件可以使用nginx官网的文件，只需要替换nginx.exe即可

能正常启动之后在ngixn.conf里还有一些复杂的配置

__nginx.conf__

~~~sh

#user  nobody;
worker_processes 1;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;
events {
     worker_connections 1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  logs/access.log  main;
     sendfile  on;
    #tcp_nopush     on;
    #keepalive_timeout  0;
     keepalive_timeout 65;
    #gzip  on;
    server {
        listen       20202;
        server_name  localhost 192.168.2.3 127.0.0.1 remoteIp;
        #access_log  logs/host.access.log  main;
		# 设置图片目录
        location /safety/ {
            root       c:;   # 路径“H:/www”下必须含有一个“imgs”目录
            autoindex  on;       # 如果为on，那么就发送请求地址对应的服务端路径下的文件列表给客户端
        }
		location / {
            root   html;
            index  index.html index.htm;
        }
		
        # 图片剪裁 crop
        # 按比例缩放 resize
        location ~* /imgs_thumb/?(.*)$ {
			charset 'utf-8';
            set $filename $1;
            set $img_w     $arg_width;
            set $img_h     $arg_height;
			#image_filter resize $img_w $img_h;
			rewrite  ^.+imgs_thumb/?(.*)$ /$1 break;
            proxy_pass  http://remoteIp:20203;    #node api server 即需要代理的IP地址
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			image_filter resize $img_w $img_h;
			image_filter_buffer 30M;
        }
       
        # 设置视频目录
        #location /videos {
        #    root       E:/www;
        #    autoindex  on;
        #}
		#location ~* \.(jpg|gif|png)$ {  
        #   image_filter resize 100 100;  
        #}  
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
	server {
		listen        20203;
		server_name  localhost 127.0.0.1 192.168.2.3 remoteIp;
		root   "c:/img";
		location / {
			index index.php index.html;
			error_page 400 /error/400.html;
			error_page 403 /error/403.html;
			error_page 404 /error/404.html;
			error_page 500 /error/500.html;
			error_page 501 /error/501.html;
			error_page 502 /error/502.html;
			error_page 503 /error/503.html;
			error_page 504 /error/504.html;
			error_page 505 /error/505.html;
			error_page 506 /error/506.html;
			error_page 507 /error/507.html;
			error_page 509 /error/509.html;
			error_page 510 /error/510.html;
			autoindex  off;
		}
		location ~ \.php(.*)$ {
			fastcgi_pass   127.0.0.1:9000;
			fastcgi_index  index.php;
			fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
			fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
			fastcgi_param  PATH_INFO  $fastcgi_path_info;
			fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
			include        fastcgi_params;
		}
	}
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
~~~

配置参考上面的配置remoreIp换成你服务器的ip，端口自选。

其中用到了一个转发，直接访问图片压缩的地址的时候发现，中文转换为url编码，然后url编码直接用到了文件地址，导致报错。再加一层转发才得以解决。



~~仅作记录，个中细节已难以回忆，大部分内容参考~~[这里](https://blog.csdn.net/qq_24054301/article/details/112131057)

[nginx官方中文文档中的内容](http://shouce.jb51.net/nginx-doc/Text/4.15_imagefilter.html)
