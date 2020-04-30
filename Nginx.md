> 全局块：boss_process 和 worker_process数量
>
> events块： 配置nginx & 用户网络连接
>
> http块： 反向代理，动静分离，缓存，日志...
>
> + http全局
> + server块：具体配置虚拟主机的代理（每个主机对应一个server块）
>     + server
>     + location 匹配

``` properties
# 命令
./nginx
# 向nginx进程发送信号
./nginx -s stop | reload | quit | reopen
# 指定配置文件启动
./nginx -c configLocation

############ 反向代理配置

# server1
sever {
	listen	80;
	server_name	localhost;	
	
	location / {
        # 反向代理路径
        proxy_pass	http://192.168.1.104:8080
	}
}

# server2
server {
	...
	# 正则匹配 ~
	location ~ /dev/ {
		proxy_pass	...
	}
	location ~ /prod/ {
		proxy_pass	...
	}
}

############ http块 负载均衡配置
# 轮询 权重 hash fair
# 集群列表
upstream balanceserver {
	# 1 权重
	# server localhost:8081	weight=5;
	# server localhost:8082	weight=10;
	
	# 2 ip_hash
	ip_hash;
	server localhost:8081	weight=5;
	server localhost:8082	weight=10;
	
	# 3 fair 响应时间
}
server {
	location / {
		# banlanceserver
		proxy_pass	http://balanceserver;
	}
}

############ 动静请求分离
# static -> html css img
# dynamic -> tomcat

# 方式一 -> 静态资源与动态资源独立部署到不同服务器
location /img/ {
	root /img/;
	# 列出当前文件夹内容
	autoindex	on;
}
location /css/ {
	root /css/;
}
# 方式二 -> 动静资源发布到一台服务器，通过配置区分
```