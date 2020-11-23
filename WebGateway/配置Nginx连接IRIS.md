参考文档：
HealthShare 2019.2 > Selected IRIS for Health Topics > Web Gateway Configuration Guide > Using Web Applications with a Remote Web Server
https://docs.intersystems.com/healthconnect20191/csp/docbook/Doc.View.cls?KEY=GCGI_ux#GCGI_ux_nginx





Web Gateway的安装：
1	解压缩Web Gateway安装包
2	参考下面的log进行Web Gateway的安装，其中标黄的地方是指定Nginx要服务的ISC应用服务器的参数配置（请参考拓扑图）
[root@localhost install]# pwd
/software/WebGateway-2019.2.0.107.0-lnxrhx64/install
[root@localhost install]# ./GatewayInstall
Starting Web Gateway installation procedure.
 
Please select WebServer type. Choose "None" if you want to configure
your WebServer manually.
1) Apache
2) None
WebServer type? [2]
 
Please enter destination directory for Web Gateway files [/opt/webgateway]:
 
Please enter hostname of your InterSystems IRIS server [localhost]: 172.16.203.136
 
Please enter superserver port number for your InterSystems IRIS server [1972]: 51773
 
Please enter InterSystems IRIS configuration name [IRIS]: UCR
 
Please enter directory for static CSP content [/opt/webgateway/ucr]:
 
Do you want to create directory /opt/webgateway/ucr [Y]:
 
Web server configuration will not be changed.
------------------------------------------------------------------
InterSystems IRIS configuration name: UCR
InterSystems IRIS server address: 172.16.203.136
InterSystems IRIS server port number: 51773
Web Gateway installation directory: /opt/webgateway
------------------------------------------------------------------
 
Do you want to continue and perform the installation [Y]:
 
    * You need to restart your Apache server before any
      configuration changes will take effect.
 
 
Web Gateway configuration completed!
•	
3	将CSPx.so 和 CSPxSys.so 两个文件拷贝到 Web Gateway的安装路径/bin下，/opt/webgateway/bin/






Nginx的安装：
1	解压缩Nginx-1.16.1安装包，解压路径为 /software/nginx-1.16.1
2	在/software/nginx-1.16.1路径下创建csp目录
# mkdir csp

3	将cspapi.h 和 ngx_http_csp_module_sa.c 拷贝到csp路径下
root@localhost csp]# ll
total 112
-r-xr-xr-x. 1 root root  5101 Feb 11 11:20 cspapi.h
-r-xr-xr-x. 1 root root 99431 Feb 11 11:20 ngx_http_csp_module_sa.c

4	在/software/nginx-1.16.1/csp 路径下创建一个文件为 config
# touch config

5	在config文件里添加如下内容
ngx_addon_name=ngx_http_csp_module_sa
HTTP_MODULES="$HTTP_MODULES ngx_http_csp_module_sa"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_csp_module_sa.c"
CORE_LIBS="$CORE_LIBS -ldl"

6	回到/software/nginx-1.16.1 路径下，对Nginx进行./configure, 添加对于csp module的支持。比如可以运行如下的命令，定义将Nginx安装到 /opt/nginx 路径下，同时，添加对于http_ssl和csp module的支持（除csp module外，其它需要添加的Nginx module请根据需要自行添加）：
# ./configure --prefix=/opt/nginx --with-http_ssl_module --add-module=/software/nginx-1.16.1/csp

在./configure的log中，可以看到csp的module添加成功：
configuring additional modules
adding module in /software/nginx-1.16.1/csp
 + ngx_http_csp_module_sa was configured

7	在/software/nginx-1.16.1 路径下对Nginx进行make
•	# make
•	
8	在/software/nginx-1.16.1 路径下运行make install，安装 Nginx
•	# make install
•	
Nginx的配置：
1	Nginx安装的路径为 /opt/nginx/
2	在 /opt/nginx/html路径下创建csp文件夹
•	#mkdir csp
•	
3	将ISC应用服务器的Instance-install-dir/csp/broker 路径下的cspbroker.js和cspxmlhttp.js这两个文件拷贝到 /opt/nginx/html/csp/broker/路径下
4	将ISC应用服务器的 Instance-install-dir/csp/sys 路径下的所有文件，拷贝到/opt/nginx/html/csp/sys 路径下
5	编辑Nginx 的配置文件 /opt/nginx/config/nginx.conf，添加对于csp的支持
5.1	在http{ 下，添加下面的内容：
•	#Adding CSP Module Path
•	
•	CSPModulePath /opt/webgateway/bin/;
•	
5.2	在server{ 下，添加下面的内容：
•	#add CSP Path
•	
•	location /csp {
•	    CSP on;
•	#CSPFileTypes csp cls zen cxw;
•	    CSPFileTypes *;
•	}
•	
6	启动Nginx
•	# /opt/nginx/sbin/nginx
•	
6.	附录二  nginx.conf sample
Location：/opt/nginx/conf/nginx.conf

#user  nobody;
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
    #
    #Adding CSP Module Path
    #
    CSPModulePath /opt/webgateway/bin/;


    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

	#add CSP Path
	#
	location /csp {
	    CSP on;
	#CSPFileTypes csp cls zen cxw;
	    CSPFileTypes *;
	}s

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

