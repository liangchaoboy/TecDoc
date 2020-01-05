#### 基本命令

- nginx -h

    ```
    root@iZuf65j82tplbqtm6akqvnZ:/etc/nginx# nginx -h
    nginx version: nginx/1.14.0 (Ubuntu)
    Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

    Options:
    -?,-h         : this help
    -v            : show version and exit
    -V            : show version and configure options then exit
    -t            : test configuration and exit
    -T            : test configuration, dump it and exit
    -q            : suppress non-error messages during configuration testing
    -s signal     : send signal to a master process: stop, quit, reopen, reload
    -p prefix     : set prefix path (default: /usr/share/nginx/)
    -c filename   : set configuration file (default: /etc/nginx/nginx.conf)
    -g directives : set global directives out of configuration file
    ```

- nginx -V #查看版本号及支持的模块

    ```
    root@iZuf65j82tplbqtm6akqvnZ:/etc/nginx# nginx -V
    nginx version: nginx/1.14.0 (Ubuntu)
    built with OpenSSL 1.1.1  11 Sep 2018
    TLS SNI support enabled
    configure arguments: --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-DUghaW/nginx-1.14.0=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-mail=dynamic --with-mail_ssl_module
    ```

 - 检测指定配置文件的语法格式(-c 是指定配置文件路径)

    ```
    root@iZuf65j82tplbqtm6akqvnZ:/etc/nginx# nginx -t -c /etc/nginx/nginx.conf
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```
- 指定配置文件启动 Nginx ：nginx -c /etc/nginx/nginx.conf

```
root@iZuf65j82tplbqtm6akqvnZ:/var/log/nginx# ps -ef | grep nginx
root     16431 16052  0 02:30 pts/0    00:00:00 grep --color=auto nginx
root@iZuf65j82tplbqtm6akqvnZ:/etc/nginx# nginx -c /etc/nginx/nginx.conf
root@iZuf65j82tplbqtm6akqvnZ:/etc/nginx# ps -ef | grep nginx
root     16438     1  0 02:31 ?        00:00:00 nginx: master process nginx -c /etc/nginx/nginx.conf
www-data 16439 16438  0 02:31 ?        00:00:00 nginx: worker process
www-data 16440 16438  0 02:31 ?        00:00:00 nginx: worker process
root     16442 16052  0 02:31 pts/0    00:00:00 grep --color=auto nginx
```

- nginx -s reload 不宕机, 平滑地加载配置文件;在升级/修改配置的时候，平滑过度到新配置
- nginx -s stop 立即停止 nginx 服务
- nginx -s quit 平滑地停止 nginx 服务
- nginx -s reopen 重新开始记录 Nginx 日志
- 重载配置文件（不影响当前请求）#下面的命令实际上是无效的，需要注意

    `nginx -s reload -c /etc/nginx/nginx.conf`   

- tcp 负载均衡配置模板（四层负载均衡从 1.9 版本开始默认支持）

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}
stream {
	upstream qiniu {
		least_conn; #最少链接数请求
		server 101.89.125.227:80;
		server 101.89.125.237:80;
	}
	upstream test {
		server 125.77.154.41:80;
	}
	upstream test2 {
		server 222.186.137.252:443;
	}
	server {
		listen 6379;
		proxy_timeout 20s;
		proxy_pass qiniu;
	}
	server {
		listen 80;
		proxy_pass test;
	}
	server {
		listen 443;
		proxy_pass test2;
	}

	log_format proxy '$remote_addr [$time_local]'
                '$protocol $status $bytes_sent $bytes_received'
                '$session_time "$upstream_addr" '
                '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';
	access_log /var/log/nginx/tcp-access.log proxy;
	error_log  /var/log/nginx/tcp-error.log;

}
```


- http 负载均衡配置模板

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}
http {
	upstream qniu {
		server 218.93.204.41:80 weight=3;
		server 120.223.240.224:80 weight=1;
	}

	server {
		listen 80;
		server_name v5.qutoutiao.net v4.qutoutiao.net;
        location / {
                proxy_pass http://qniu;
		proxy_set_header Host $host;
        	}
	location ~ /\.ts {
		deny all;
		}
	}
	log_format proxy '$remote_addr [$time_local]'
                '$status $bytes_sent'
                '"$upstream_addr" ';
	access_log /var/log/nginx/access.log proxy;
	error_log /var/log/nginx/error.log;

}
```
- 查看当前使用的配置文件, nginx -t 会显示配置文件路径

```
root@iZuf65j82tplbqtm6akqvnZ:/etc/nginx# ps -ef | grep nginx
root     16235     1  0 01:55 ?        00:00:00 nginx: master process nginx -c /etc/nginx/nginx_tcp.conf
www-data 16236 16235  0 01:55 ?        00:00:00 nginx: worker process
www-data 16237 16235  0 01:55 ?        00:00:00 nginx: worker process
root     16272 16052  0 02:01 pts/0    00:00:00 grep --color=auto nginx
```


- log_format 参数

```
参数                      说明                                         示例
$remote_addr             客户端地址                                    211.28.65.253
$remote_user             客户端用户名称                                --
$time_local              访问时间和时区                                18/Jul/2012:17:00:01 +0800
$request                 请求的URI和HTTP协议                           "GET /article-10000.html HTTP/1.1"
$http_host               请求地址，即浏览器中你输入的地址（IP或域名）     www.wang.com 192.168.100.100
$status                  HTTP请求状态                                  200
$upstream_status         upstream状态                                  200
$body_bytes_sent         发送给客户端文件内容大小                        1547
$http_referer            url跳转来源                                   https://www.baidu.com/
$http_user_agent         用户终端浏览器等信息                           "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SV1; GTB7.0; .NET4.0C;
$ssl_protocol            SSL协议版本                                   TLSv1
$ssl_cipher              交换数据中的算法                               RC4-SHA
$upstream_addr           后台upstream的地址，即真正提供服务的主机地址     10.10.10.100:80
$request_time            整个请求的总时间                               0.205
$upstream_response_time  请求过程中，upstream响应时间                    0.002
```


> https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/

> https://www.nginx.com/resources/wiki/start/topics/examples/loadbalanceexample/

> http://www.freeoa.net/osuport/servap/nginx-proxy-stream_3356.html

> http://nginx.org/en/docs/stream/ngx_stream_core_module.html

> https://www.cnblogs.com/lvgg/p/6140584.html

