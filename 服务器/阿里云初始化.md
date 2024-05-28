## Tomcat初始化
1. 下载上传解压JDK和tomcat的linux包：
	- 上传：`scp /Users/huangfyy/Downloads/jdk-21_linux-x64_bin.tar.gz root@localhost: /usr/local/src/java/`
	- 解压：`tar -zxvf apache-tomcat-9.0.89.tar.gz`
2. 配置环境变量：
	- `vim /etc/profile`
	- `export JAVA_HOME=/usr/local/src/java/jdk-21.0.3`
	- `export PATH=$PATH:$JAVA_HOME/bi`
3. tomcat demo：`/usr/local/src/tomcat/apache-tomcat-9.0.89/webapps`
	1. 复制`/webapps/ROOT/WEB-INF`到自定义文件夹下。
	2. 新建index.html文件，当前文件夹内容为：`WEB-INF  index.html`。
	3. `http://121.40.107.7:8080/自定义文件夹名称/` 即可以访问该index.html。
## 域名备案和解析
1. 域名解析：
	https://help.aliyun.com/zh/dns/novice-guide?spm=5176.swas-next_server-detail.help.dexternal.71144ad85b1Mg8