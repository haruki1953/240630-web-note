[Nginx_day03](资料/黑马nginx/day03/笔记/Nginx_day03.md)

## Rewrite功能配置
实现URL的重写

注意：Nginx服务器的Rewrite功能的实现依赖于PCRE的支持，因此在编译安装Nginx服务器之前，需要安装PCRE库。Nginx使用的是ngx_http_rewrite_module模块来解析和处理Rewrite功能的相关配置。

Rewrite的相关命令
```
set指令
if指令
break指令
return指令
rewrite指令
rewrite_log指令
```

Rewrite的应用场景
```
域名跳转
域名镜像
独立域名
目录自动添加"/"
合并目录
防盗链的实现
```

### Rewrite的相关指令
#### set指令
该指令用来设置一个新的变量。

| 语法   | set $variable value; |
| ------ | -------------------- |
| 默认值 | —                    |
| 位置   | server、location、if |

variable：变量的名称，该变量名称要用"$"作为变量的第一个字符，且不要与Nginx服务器预设的全局变量同名。

value：变量的值，可以是字符串、其他变量或者变量的组合等。
![](assets/Pasted%20image%2020240706123057.png)


#### Rewrite常用全局变量

| 变量               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
|`$args             `| `变量中存放了请求URL中的请求参数。比如http://192.168.200.133/server?arg1=value1&args2=value2中的"arg1=value1&arg2=value2"，功能和$query_string一样` |
|`$http_user_agent  `| 变量存储的是用户访问服务的代理信息(如果通过浏览器访问，记录的是浏览器的相关版本信息) |
|`$host             `| 变量存储的是访问服务器的server_name值                        |
|`$document_uri     `| `变量存储的是当前访问地址的URI。比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server"，功能和$uri一样` |
|`$document_root    `| 变量存储的是当前请求对应location的root值，如果未设置，默认指向Nginx自带html目录所在位置 |
|`$content_length   `| 变量存储的是请求头中的Content-Length的值                     |
|`$content_type     `| 变量存储的是请求头中的Content-Type的值                       |
|`$http_cookie      `| `变量存储的是客户端的cookie信息，可以通过add_header Set-Cookie 'cookieName=cookieValue'来添加cookie数据` |
|`$limit_rate       `| 变量中存储的是Nginx服务器对网络连接速率的限制，也就是Nginx配置中对limit_rate指令设置的值，默认是0，不限制。 |
|`$remote_addr      `| 变量中存储的是客户端的IP地址                                 |
|`$remote_port      `| 变量中存储了客户端与服务端建立连接的端口号                   |
|`$remote_user      `| 变量中存储了客户端的用户名，需要有认证模块才能获取           |
|`$scheme           `| 变量中存储了访问协议                                         |
|`$server_addr      `| 变量中存储了服务端的地址                                     |
|`$server_name      `| 变量中存储了客户端请求到达的服务器的名称                     |
|`$server_port      `| 变量中存储了客户端请求到达服务器的端口号                     |
|`$server_protocol  `| 变量中存储了客户端请求协议的版本，比如"HTTP/1.1"             |
|`$request_body_file`| 变量中存储了发给后端服务器的本地文件资源的名称               |
|`$request_method   `| 变量中存储了客户端的请求方式，比如"GET","POST"等             |
|`$request_filename `| 变量中存储了当前请求的资源文件的路径名                       |
|`$request_uri      `| `变量中存储了当前请求的URI，并且携带请求参数，比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server?id=10&name=zhangsan"` |

上述参数还可以在日志文件中使用，这个就要用到前面我们介绍的`log_format`指令
```
log_format main '$remote_addr - $request - $status-$request_uri  $http_user_agent';

access_log logs/access.log main;
```


#### if指令
该指令用来支持条件判断，并根据条件判断结果选择不同的Nginx配置。

| 语法   | if  (condition){...} |
| ------ | -------------------- |
| 默认值 | —                    |
| 位置   | server、location     |

condition为判定条件，可以支持以下写法：
1. 变量名。如果变量名对应的值为空字符串或"0"，if都判断为false,其他条件为true。
```
if ($param){
	
}
```

2. 使用"="和"!="比较变量和字符串是否相等，满足条件为true，不满足为false
```
if ($request_method = POST){
	return 405;
}
```

注意：此处和Java不太一样的地方是字符串不需要添加引号，并且等号和不等号前后到需要加空格。

3. 使用正则表达式对变量进行匹配，匹配成功返回true，否则返回false。变量与正则表达式之间使用`"~","~*","!~","!~*"`来连接。
```
   "~"代表匹配正则表达式过程中区分大小写，
   "~*"代表匹配正则表达式过程中不区分大小写
   "!~"和"!~*"刚好和上面取相反值，如果匹配上返回false,匹配不上返回true

if ($http_user_agent ~ MSIE){
	#$http_user_agent的值中是否包含MSIE字符串，如果包含返回true
}
```
注意：正则表达式字符串一般不需要加引号，但是如果字符串中包含"}"或者是";"等字符时，就需要把引号加上。

4. 判断请求的文件是否存在使用"-f"和"!-f",
```
root html
if (-f $request_filename){
	#判断请求的文件是否存在
}
if (!-f $request_filename){
	#判断请求的文件是否不存在
}
```

5. 判断请求的目录是否存在，使用"-d"和"!-d"

6. 判断请求的目录或者文件是否存在，使用"-e"和"!-e"

7. 判断请求的文件是否可执行，使用"-x"和"!-x"


#### break指令
该指令用于中断当前相同作用域中的其他Nginx配置。与该指令处于同一作用域的Nginx配置中，位于它前面的指令配置生效，位于后面的指令配置无效。并且break还有另外一个功能就是终止当前的匹配并把当前的URI在本location进行重定向访问处理。

| 语法   | break;               |
| ------ | -------------------- |
| 默认值 | —                    |
| 位置   | server、location、if |

例子:
```
location /testbreak{
	default_type text/plain;
	set $username TOM;
	if ($args){
		set $username JERRY;
        break;
		set $username ROSE;
	}
	add_header username $username;
	return 200 $username;
}
```
带参数的情况下，`set $username ROSE` 将不会执行，而 `add_header username $username` 还会被执行。
但是 `return 200 $username` 将失效，会返回301重定向至/testbreak

好奇怪，还是少用吧


#### return指令
该指令用于完成对请求的处理，直接向客户端返回。在return后的所有Nginx配置都是无效的。

| 语法   | return code [text];<br/>return code URL;<br/>return URL; |
| ------ | -------------------------------------------------------- |
| 默认值 | —                                                        |
| 位置   | server、location、if                                     |

code:为返回给客户端的HTTP状态代理。可以返回的状态代码为0~999的任意HTTP状态代理

text:为返回给客户端的响应体内容，支持变量的使用

URL:为返回给客户端的URL地址

```
location /testreturn {
	return 200 success;
}

location /testreturn {
	return https://www.baidu.com; // 302重定向到百度
}

location /testreturn {
	return 302 /home; // 重定向至/home
}

location /testreturn {
	return 302 www.baidu.com;//不允许这么写
}
```


#### rewrite指令
该指令通过正则表达式的使用来改变URI。可以同时存在一个或者多个指令，按照顺序依次对URL进行匹配和处理。

| 语法   | rewrite regex replacement [flag]; |
| ------ | --------------------------------- |
| 默认值 | —                                 |
| 位置   | server、location、if              |

regex:用来匹配URI的正则表达式

replacement：匹配成功后，用于替换URI中被截取内容的字符串。如果该字符串是以 "http://" 或者 "https://" 开头的，则不会继续向下对URI进行其他处理，而是直接返回重写后的URI给客户端。

```
location /rewrite {
	rewrite ^/rewrite/url\w*$ https://www.baidu.com;
	rewrite ^/rewrite/(test)\w*$ /$1;
	rewrite ^/rewrite/demo\w*$ /demo;
}
location /test{
	default_type text/plain;
	return 200 test_success;
}
location /demo{
	default_type text/plain;
	return 200 demo_success;
}
```

flag：用来设置rewrite对URI的处理行为，可选值有如下：
- **last**：终止继续在本location块中处理接收到的URI，并将此处重写的URI作为一个新的URI，使用各location块进行处理。该标志将重写后的URI重写在server块中执行，为重写后的URI提供了转入到其他location块的机会。（默认）
```
location rewrite {
	rewrite ^/rewrite/(test)\w*$ /$1 last;
	rewrite ^/rewrite/(demo)\w*$ /$1 last;
}
location /test{
	default_type text/plain;
	return 200 test_success;
}
location /demo{
	default_type text/plain;
	return 200 demo_success;
}
```
访问 `http://192.168.200.133:8081/rewrite/testabc`,能正确访问


- **break**：将此处重写的URI作为一个新的URI，在本块中继续进行处理。该标志将重写后的地址在当前的location块中执行，不会将新的URI转向其他的location块。
```
location /rewrite {
    #/test   /usr/local/nginx/html/test
    #重写break之后不会跳转到其他location，而是根据root返回静态资源
	rewrite ^/rewrite/(test)\w*$ /$1 break;
	rewrite ^/rewrite/(demo)\w*$ /$1 break;
}
location /test{
	default_type text/plain;
	return 200 test_success;
}
location /demo{
	default_type text/plain;
	return 200 demo_success;
}


```
访问 `http://192.168.200.133:8081/rewrite/demoabc`,页面报404错误，因为重写后会尝试访问`/usr/local/nginx/html/demo`，文件或目录不存在


- **redirect**：将重写后的URI返回给客户端，状态码为302，指明是临时重定向URI,主要用在replacement变量不是以"http://" 或者 "https://" 开头的情况。
```
location rewrite {
	rewrite ^/rewrite/(test)\w*$ /$1 redirect;
	rewrite ^/rewrite/(demo)\w*$ /$1 redirect;
}
location /test{
	default_type text/plain;
	return 200 test_success;
}
location /demo{
	default_type text/plain;
	return 200 demo_success;
}
```
访问`http://192.168.200.133:8081/rewrite/testabc`请求会被临时重定向，浏览器地址也会发生改变

- **permanent**：将重写后的URI返回给客户端，状态码为301，指明是永久重定向URI,主要用在replacement变量不是以 "http://" 或者 "https://" 开头的情况。
```
location rewrite {
	rewrite ^/rewrite/(test)\w*$ /$1 permanent;
	rewrite ^/rewrite/(demo)\w*$ /$1 permanent;
}
location /test{
	default_type text/plain;
	return 200 test_success;
}
location /demo{
	default_type text/plain;
	return 200 demo_success;
}
```
访问`http://192.168.200.133:8081/rewrite/testabc`请求会被永久重定向，浏览器地址也会发生改变


#### rewrite_log指令
该指令配置是否开启URL重写日志的输出功能。

| 语法   | rewrite_log on\|off;       |
| ------ | -------------------------- |
| 默认值 | rewrite_log off;           |
| 位置   | http、server、location、if |

开启后，URL重写的相关日志将以notice级别输出到error_log指令配置的日志文件汇总。
```
rewrite_log on;
error_log  logs/error.log notice;
```


### Rewrite的案例
#### 域名跳转
- 准备三个域名：
```
vim /etc/hosts
```
```
127.0.0.1   www.itcast.cn
127.0.0.1   www.itheima.cn
127.0.0.1   www.itheima.com
```

- 通过Nginx实现访问www.itcast.cn
```
server {
	listen 80;
	server_name www.itcast.cn;
	location /{
		default_type text/html;
		return 200 '<h1>welcome to itcast</h1>';
	}
}
```

通过Rewrite完成将www.ithema.com和www.itheima.cn的请求跳转到www.itcast.com
```
server {
	listen 80;
	server_name www.itheima.com www.itheima.cn;
	rewrite ^/ http://www.itcast.cn;
}
```

问题描述:如何在域名跳转的过程中携带请求的URI？
修改配置信息
```
server {
	listen 80;
	server_name www.itheima.com www.itheima.cn;
	rewrite ^(.*) http://www.itcast.cn$1；
}
```


#### 合并目录
搜索引擎优化(SEO)是一种利用搜索引擎的搜索规则来提高目的网站在有关搜索引擎内排名的方式。我们在创建自己的站点时，可以通过很多中方式来有效的提供搜索引擎优化的程度。其中有一项就包含URL的目录层级一般不要超过三层，否则的话不利于搜索引擎的搜索也给客户端的输入带来了负担，但是将所有的文件放在一个目录下又会导致文件资源管理混乱并且访问文件的速度也会随着文件增多而慢下来，这两个问题是相互矛盾的，那么使用rewrite如何解决上述问题?

举例，网站中有一个资源文件的访问路径时 /server/11/22/33/44/20.html,也就是说20.html存在于第5级目录下，如果想要访问该资源文件，客户端的URL地址就要写成 `http://192.168.200.133/server/11/22/33/44/20.html`,
```
server {
	listen 8083;
	server_name localhost;
	location /server{
		root html;
	}
}
```

但是这个是非常不利于SEO搜索引擎优化的，同时客户端也不好记.使用rewrite我们可以进行如下配置:
```
server {
	listen 8083;
	server_name localhost;
	location /server{
		rewrite ^/server-([0-9]+)-([0-9]+)-([0-9]+)-([0-9]+)\.html$ /server/$1/$2/$3/$4/$5.html last;
	}
}
```
这样的话，客户端只需要输入http://www.web.name/server-11-22-33-44-20.html就可以访问到20.html页面了。这里也充分利用了rewrite指令支持正则表达式的特性。


#### 防盗链
防盗链之前我们已经介绍过了相关的知识，在rewrite中的防盗链和之前将的原理其实都是一样的，只不过通过rewrite可以将防盗链的功能进行完善下，当出现防盗链的情况，我们可以使用rewrite将请求转发到自定义的一张图片和页面，给用户比较好的提示信息。下面我们就通过根据文件类型实现防盗链的一个配置实例:
```
location /images {
    root html;
    valid_referers none blocked www.baidu.com;
    if ($invalid_referer){
        #return 403;
        rewrite ^/    /images/forbidden.png break;
    }
}

```


