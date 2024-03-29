---
title: docker安装nacos
author: Yu Mengchi
categories:
  - Java 
tags:
  - Java
---
  
1. 拉取镜像
m1版mac还要区分镜像版本，我用的是zhusaidong/nacos-server-m1:2.0.3
2. 本地mysql创建nacos数据库
3. 在nacos数据库中执行脚本
https://github.com/alibaba/nacos/blob/master/config/src/main/resources/META-INF/nacos-db.sql
4. 因为我没有映射容器目录到宿主机目录，所以直接在容器的/home/nacos/conf/application.properties文件中修改配置，主要是数据库的配置
# spring
server.servlet.contextPath=${SERVER_SERVLET_CONTEXTPATH:/nacos}
server.contextPath=/nacos
server.port=${NACOS_APPLICATION_PORT:8848}
spring.datasource.platform=${SPRING_DATASOURCE_PLATFORM:mysql}
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
db.num=${MYSQL_DATABASE_NUM:1}
db.url.0=jdbc:mysql://${MYSQL_SERVICE_HOST:172.20.10.5}:${MYSQL_SERVICE_PORT:3306}/${MYSQL_SERVICE_DB_NAME:nacos}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=10000&socketTimeout=30000&autoReconnect=true&useSSL=false}
#db.url.1=jdbc:mysql://${MYSQL_SERVICE_HOST}:${MYSQL_SERVICE_PORT:3306}/${MYSQL_SERVICE_DB_NAME}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false}
db.user=${MYSQL_SERVICE_USER:root}
db.password=${MYSQL_SERVICE_PASSWORD:jinm89889}
### The auth system to use, currently only 'nacos' is supported:
nacos.core.auth.system.type=${NACOS_AUTH_SYSTEM_TYPE:nacos}
5. 要注意mysql的root用户是否能被其他ip访问，如果不能需要设置mysql权限，使root用户能被其他ip访问
https://blog.csdn.net/Mrzhang567/article/details/122489796
6. 再注意mysql是否会报错 Public Key Retrieval is not allowed。解决决方法是
设置一个配置为true
https://blog.csdn.net/u013360850/article/details/80373604
7. 最后重启docker内的nacos。

启动nacos时用下面命令
docker run --name nacos -d -p 8848:8848 --privileged=true --restart=always -e JVM_XMS=256m -e JVM_XMX=256m -e MODE=standalone -e PREFER_HOST_MODE=nacos  zhusaidong/nacos-server-m1:2.0.3

### docker 部署nginx

1. 拉取nginx镜像
docker pull nginx:latest
2. 启动
3. 修改配置文件/etc/nginx/conf.d/default.conf,主要是/api/的配置，会将对localhost/8080/api/的请求转发到172.20.10.5:8090，172.20.10.5:8090是本地服务的端口
4. 将前端压缩文件放在/usr/share/nginx/html/目录下，启动nginx后，访问localhost/8080/api/会访问后端服务

server {
listen       80;
listen  [::]:80;
server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    location /api/ {
        proxy_pass http://172.20.10.5:8090/;
      }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
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


