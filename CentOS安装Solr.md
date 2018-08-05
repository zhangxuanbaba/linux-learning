## CentOS安装Solr

solr安装
环境准备:

jdk 1.8

楼主安装目录为:

		/usr/local/jdk1.8/

tomcat8 
PS: 最后是8.5版本之后 8.5之下版本 Apache已停止维护 
楼主安装目录为:

		/usr/local/solr-tomcat/

solr-7.2.1 目前最新版本(2018-01-15)

一: 下载
创建下载目录 并进入到下载目录

		mkdir -p /usr/local/software/ && cd /usr/local/software/

solr 下载地址

		wget http://archive.apache.org/dist/lucene/solr/7.2.1/solr-7.2.1.zip

解压目录

		unzip /usr/local/software/solr-7.2.1.zip 

把Solr的项目拷贝到solr-tomcat的webapps下

		cp -R /usr/local/software/solr-7.2.1/server/solr-webapp/webapp/ /usr/local/solr-tomcat/webapps/

注意: 路径为: solr-7.2.1/server/solr-webapp/webapp,而不是: solr-7.2.1/server/solr-webapp/ 否则Tomcat不能成功启动

为Solr更名: 把webapp更改为solr

		mv /usr/local/solr-tomcat/webapps/webapp /usr/local/solr-tomcat/webapps/solr

二: 调整项目

拷贝solr-7.2.1/server/lib/ext/*下的jar包到 solr-tomcat/webapps/solr/WEB-INF/lib/下

		cp -R /usr/local/software/solr-7.2.1/server/lib/ext/* /usr/local/solr-tomcat/webapps/solr/WEB-INF/lib/

拷贝solr-7.2.1/dist/solr-dataimporthandler-7.2.1.jar 包到 solr-tomcat/webapps/solr/WEB-INF/lib/下

		cp -R /usr/local/software/solr-7.2.1/dist/solr-dataimporthandler-7.2.1.jar /usr/local/solr-tomcat/webapps/solr/WEB-INF/lib/

拷贝solr-7.2.1/dist/solr-dataimporthandler-extras-7.2.1.jar 包到 solr-tomcat/webapps/solr/WEB-INF/lib/下

		cp -R /usr/local/software/solr-7.2.1/dist/solr-dataimporthandler-extras-7.2.1.jar /usr/local/solr-tomcat/webapps/solr/WEB-INF/lib/


拷贝: metrics开头的5个文件 <br/>
名称如下: <br/>
metrics-core-3.2.2.jar <br/>
metrics-ganglia-3.2.2.jar <br/>
metrics-graphite-3.2.2.jar <br/>
metrics-jetty9-3.2.2.jar <br/>
metrics-jvm-3.2.2.jar

		cp -r /usr/local/software/solr-7.2.1/server/lib/metrics*.* /usr/local/solr-tomcat/webapps/solr/WEB-INF/lib/

拷贝solr-7.2.1/server下的solr 到/usr/local/solrHome 作为solr的主目录

		cp -R /usr/local/software/solr-7.2.1/server/solr/* /usr/local/solrHome

查看拷贝成功的目录

		[root@jsyfdev001 solrHome]# ls -al /usr/local/solrHome
		total 24
		drwxr-xr-x   3 root root 4096 Jan 15 18:28 .
		drwxr-xr-x. 35 root root 4096 Jan 15 18:23 ..
		drwxr-xr-x   4 root root 4096 Jan 15 18:28 configsets
		-rw-r--r--   1 root root 3095 Jan 15 18:28 README.txt
		-rw-r--r--   1 root root 2170 Jan 15 18:28 solr.xml
		-rw-r--r--   1 root root 1006 Jan 15 18:28 zoo.cfg

修改WEB-INF/web.xml的配置 把/put/your/solr/home/here指向到/usr/local/solrHome

		sudo vim /usr/local/solr-tomcat/webapps/solr/WEB-INF/web.xml 

自带的Solr配置

		 <!--
			<env-entry>
			   <env-entry-name>solr/home</env-entry-name>
			   <env-entry-value>/put/your/solr/home/here</env-entry-value>
			   <env-entry-type>java.lang.String</env-entry-type>
			</env-entry>
		   -->

修改后的Solr配置

		  <env-entry>
			 <env-entry-name>solr/home</env-entry-name>
			 <!--注意: 此处做了修改Solr的主目录 指向到/usr/local/solrHome-->
			  <env-entry-value>/usr/local/solrHome</env-entry-value>
			  <env-entry-type>java.lang.String</env-entry-type>
		   </env-entry>

创建classes 用来存放日志文件

		mkdir -p /usr/local/solr-tomcat/webapps/solr/WEB-INF/classes 

拷贝solr-7.2.1/server/resources/log4j.properties 到solr-tomcat/webapps/solr/WEB-INF/classes下

		cp /usr/local/software/solr-7.2.1/server/resources/log4j.properties /usr/local/solr-tomcat/webapps/solr/WEB-INF/classes/

三: 开放端口

		sudo vim /etc/sysconfig/iptables

添加开放端口:

		# solr-tomcat端口 注意: 35080为你的Tomcat端口
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 35080 -j ACCEPT

注意: 如果你的服务器是阿里云需要在阿里云上添加开放端口

		// TODO 启动solr-tomcat

启动后打开网址出错:

http://ip:35080/solr/index.html 
出错信息为:
1. 权限控制

HTTP Status 403 - Access to the requested resource has been denied

		解决方法如下:
		注释web.xml中权限控制配置
		sudo vim /usr/local/solr-tomcat/webapps/solr/WEB-INF/web.xml 

在web.xml的最后面 把下面的代码注释掉

修改前:

		 <security-constraint>
			<web-resource-collection>
			  <web-resource-name>Disable TRACE</web-resource-name>
			  <url-pattern>/</url-pattern>
			  <http-method>TRACE</http-method>
			</web-resource-collection>
			<auth-constraint/>
		  </security-constraint>
		  <security-constraint>
			<web-resource-collection>
			  <web-resource-name>Enable everything but TRACE</web-resource-name>
			  <url-pattern>/</url-pattern>
			  <http-method-omission>TRACE</http-method-omission>
			</web-resource-collection>
		  </security-constraint>

修改后:

		 <!--
		  <security-constraint>
			<web-resource-collection>
			  <web-resource-name>Disable TRACE</web-resource-name>
			  <url-pattern>/</url-pattern>
			  <http-method>TRACE</http-method>
			</web-resource-collection>
			<auth-constraint/>
		  </security-constraint>
		  <security-constraint>
			<web-resource-collection>
			  <web-resource-name>Enable everything but TRACE</web-resource-name>
			  <url-pattern>/</url-pattern>
			  <http-method-omission>TRACE</http-method-omission>
			</web-resource-collection>
		  </security-constraint>
		  -->

404页面

		HTTP Status 404 - Servlet LoadAdminUI is not available

解决方法如下:

		检查以上jar是否全部放到solr/WEB-INF/lib中
		检查是否拷贝solr-7.2.1/server下的solr 到/usr/local/solrHome 作为solr的主目录

OK, 至此Solr搭建完毕!

------------

使用Solr

创建Core Admin

![](https://img-blog.csdn.net/20180117094422962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGFud3VxaWEwMzcw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注: 此处相当于建造一个关系型数据库的表, 用来存放数据的

当按照图片上步骤执行到第六步时, 会出现以下错误

错误的大概意思为:
在我们点击Add core之后Solr会自动帮我们生成的一个文件(路径为: /usr/local/solrHome/new_core/conf/)中没有找到solrconfig.xml

		org.apache.solr.common.SolrException:org.apache.solr.common.SolrException: Could not load conf for core new_core: Error loading solr config from /usr/local/solrHome/new_core/conf/solrconfig.xml

解决方法如下:

		#创建Solr的core admin(上图第一步骤的时候)
		mkdir -p  /usr/local/solrHome/new_core

		# 删除new_core下所有的文件, 防止添加在上图第一步的时候 创建多余的文件
		rm -rf /usr/local/solrHome/new_core/

拷贝_default下的conf目录

		cp -R /usr/local/software/solr-7.2.1/server/solr/configsets/_default/* /usr/local/solrHome/new_core/


		#重新启动Solr-tomcat 再次执行上图的步骤 便不全报错
		# 关闭Solr-tomcat
		/usr/local/solr-tomcat/bin/shutdown.sh
		# 启动Tomcat
		/usr/local/solr-tomcat/bin/startup.sh
