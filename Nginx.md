# Nginx

​	*Nginx* (engine x) 是一个高性能的HTTP和反向代理服务，也是一个IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。 

​	`https://baike.baidu.com/item/nginx/3817705?fr=aladdin#8`

### 命令

​	启动： `start nginx`

​	停止：	`nginx -s stop` `nginx -s quit`

​	重载配置：	`nginx -s reload`

### 负载均衡

#### 实现

​	开启两个tomcat服务

​	编辑%nginx_home/conf/nginx.conf

```shell
#在 ‘#gzip on;’ 后面添加
upstream localhost{	#向上请求流初始化
    server localhost:8080 weight=1;	#该服务访问权重为1
    server localhost:8081 weight=1;
}
```

```shell
#在http元素内添加 include vhost/*.conf
http{
    include vhost/*.conf;	#加载该目录下所有配置，相对于该配置文件目录
    ......
}
```

​	新建vhost，在vhost下新建任意.conf文件，编辑

```shell
server{	#表示一个虚拟主机，服务器
    listen 80;	#监听端口
    server_name www.lov.com;	#服务名
    location / {	#根路径代理
        proxy_pass http://localhost;	#代理路径
        proxy_connect_timeout 500ms;
    }
}
```

​	编辑c:/windows/system32/drivers/etc/hosts

```shell
127.0.0.1 www.lov.com	#将该域名解析指向本机
127.0.0.1 localhost
```

​	启动nginx 或 重载配置		`nginx -s reload`	

### 静态代理

​	减少中间件（tomcat）的IO，提高性能；对于动态处理交由服务器，静态资源访问交由nginx转发获取

#### 实现

​	在vhost下新建.conf文件

```shell
server{
	listen 80;
	server_name img.lov.com;
	root D:/img/;	#root路径
	location / {
		index index.html;
	}
}
```

​	修改系统hosts

```shell
127.0.0.1 img.lov.com
```

​	启动nginx 或 重载配置		`nginx -s reload`	

![](D:\Git_REP\Nginx\conf.png)

### 总结

**负载均衡与静态代理的问题：**

```text
1、因为代理的都是不同的服务器，而对于应用，用户登录后的session或默认存在对应服务器，而对于静态资源或其他服务器访问时，在不同机器上会导致获取不到对应session

	（统一认证解决）所以对于这种问题，可以增加一个服务器专门存储session，当对不同用户要获取权限时直接从在服务器中获取session
```

```text
2、单点登录不是统一认证，只是对于单点登录可以用统一认证解决
```





​	