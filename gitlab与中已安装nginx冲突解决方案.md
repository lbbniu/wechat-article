## 原来的程序配置 
	OneinStack安装
	/root/oneinstack/config/nginx.conf     
	根目录在这/data/wwwroot/
## gitlab的配置在  
		
	/var/opt/gitlab/nginx/conf/nginx.conf
	/var/opt/gitlab/nginx/conf/gitlab-http.conf          
	gitlab设置端口的地方是这/etc/gitlab/gitlab.rb

##nginx安装方式
https://oneinstack.com/
##gitlab安装方式
http://www.cnblogs.com/yangliheng/p/5760185.html


###/opt/gitlab/service/nginx/run 文件
	
	#源内容
	#!/bin/sh
	exec 2>&1
	
	cd /var/opt/gitlab/nginx
	exec chpst -P /opt/gitlab/embedded/sbin/nginx -p /var/opt/gitlab/nginx
	#新内容
	#!/bin/sh
	exec 2>&1
	
	cd /usr/local/nginx
	exec chpst -P /usr/local/nginx/sbin/nginx -p /usr/local/nginx
		
	
###/var/opt/gitlab/nginx/conf/gitlab-http.conf 文件
	#增加
	log_format gitlab_access '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
	log_format gitlab_ci_access '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
	log_format gitlab_mattermost_access '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';	
	proxy_cache_path proxy_cache keys_zone=gitlab:10m max_size=1g levels=1:2;
    proxy_cache gitlab;
    #以上配置来自 /var/opt/gitlab/nginx/conf/nginx.conf 文件中，去除与/usr/local/nginx/conf/nginx.conf 文件中的冲突选项
    #nginx文件修后可以使用 nginx -t 来检测配置文件是否正确
    #使用nginx -s reload 进行重启加载配置文件
  	
  	
### /usr/local/nginx/conf/nginx.conf文件
	#配置文件开头加入
	daemon off;	 #nginx不以守护进程方式运行
	#在http配置段增加
	include /var/opt/gitlab/nginx/conf/gitlab-http.conf;


###修改目录权限
	cd /var/opt/gitlab && chgrp www gitlab-workhorse
	cd /var/opt/gitlab/nginx chown -R www:www uwsgi_temp scgi_temp proxy_temp proxy_cache fastcgi_temp client_body_temp
	cd /usr/local/nginx chown -R www:www uwsgi_temp scgi_temp proxy_temp proxy_cache fastcgi_temp client_body_temp

###干掉nginx gitlab 会自动启动新的nginx进程
	killall nginx 

###gitlab邮箱配置
	#/etc/gitlab/gitlab.rb 文件
	gitlab_rails['smtp_enable'] = true
	gitlab_rails['smtp_address'] = "smtp.qq.com"
	gitlab_rails['smtp_port'] = 25
	#gitlab_rails['smtp_port'] = 465
	gitlab_rails['smtp_user_name'] = "lbbniu@qq.com"
	gitlab_rails['smtp_password'] = "123132151456"
	gitlab_rails['smtp_domain'] = "smtp.qq.com"
	gitlab_rails['smtp_authentication'] = "plain"
	gitlab_rails['smtp_enable_starttls_auto'] = true
	gitlab_rails['gitlab_email_from']='lbbniu@qq.com'
	user['git_user_email'] = "lbbniu@qq.com"
	
###重启所有服务
	#为了保险期间，先停止后启动，或者直接重启服务
	gitlab-ctl stop	 #停止服务
	gitlab-ctl start #启动服务
	gitlab-ctl restart #重启
	gitlab-ctl tail #查看是否有错误
	
