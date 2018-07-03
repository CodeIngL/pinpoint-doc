## 代理HTTP头部

- Pinpoint配置
- 代理配置
	- Apache HTTP服务器
	- Nginx
	- 应用

### 使用HTTP头部进行代理监控 ###

它用于了解代理和WAS之间的`elapsed`时间。

![](http://naver.github.io/pinpoint/images/proxy-http-header-overview.png)

#### Pinpoint配置 ####

pinpoint.config

	profiler.proxy.http.header.enable=true

#### 代理配置 ####

#### Apache HTTP服务器 ####

	- http://httpd.apache.org/docs/2.4/en/mod/mod_headers.html

添加HTTP头部。

	Pinpoint-ProxyApache: t=991424704447256 D=3775428 i=51 b=49

e.g.

httpd.conf

	<IfModule mod_jk.c>
	...
	RequestHeader set Pinpoint-ProxyApache "%t %D %i %b"
	...
	</IfModule>

％t是必需的值。

### Nginx ###

	http://nginx.org/en/docs/http/ngx_http_core_module.html
	http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header

添加HTTP头部。

	Pinpoint-ProxyNginx: t=1504248328.423 D=0.123

e.g.

nginx.conf

	...
	  server {
	        listen       9080;
	        server_name  localhost;
	
	        location / {
	            ...
	            set $pinpoint_proxy_header "t=$msec D=$request_time";
	            proxy_set_header Pinpoint-ProxyNginx $pinpoint_proxy_header;
	        }
	  }
	...

or

	http {
	...
	
	    proxy_set_header Pinpoint-ProxyNginx t=$msec;
	
	...
	}


**t=$msec是必需的值**

#### 应用 ####

自`epoch`和应用程序信息以来的毫秒数。

添加HTTP头部。

	Pinpoint-ProxyApp: t=1502861340 app=foo-bar

**t=epoch是必需的值。**