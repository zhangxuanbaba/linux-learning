#### 一、准备工作

##### 1.1. 下载软件: http://sourceforge.net/projects/fastdfs/files/

##### 1.2. 安装gcc。命令:
		yum install make cmake gcc gcc-c++

#### 二、安装libfastcommon

##### 2.1 上传libfastcommon-master.zip到/usr/local/software下

##### 2.2 进行解压libfastcommon-master.zip: 

		cd /usr/local/software
		mkdir -p /usr/local/fast/
		yum install -y unzip zip
		unzip libfastcommon-master.zip -d /usr/local/fast/

##### 2.3 进入目录:

		cd /usr/local/fast/libfastcommon-master/

##### 2.4 进行编译和安装:

		# 编译
		./make.sh
		# 安装
		./make.sh install

安装信息如下:
我们的libfastcommon默认安装到了/usr/lib64/这个位置(如下: )。

		[root@jsyfdev001 libfastcommon-master]# ./make.sh install
		mkdir -p /usr/lib64
		install -m 755 libfastcommon.so /usr/lib64
		mkdir -p /usr/include/fastcommon
		install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_bits.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h process_ctrl.h fast_mblock.h connection_pool.h /usr/include/fastcommon

##### 2.5 进行软件创建。FastDFS主程序设置的目录为/usr/local/lib/，所以我们需要创建/ usr/lib64/下的一些核心执行程序的软连接文件

		mkdir /usr/local/lib/
		# 建立软链
		ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
		ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so 
		ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so 
		ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so

#### 三、安装FastDFS

##### 3.1 进入到software下

		cd /usr/local/software

解压FastDFS_v5.05.tar.gz文件 
命令如下:

		cd /usr/local/software
		tar -zxvf FastDFS_v5.05.tar.gz -C /usr/local/fast/

##### 3.2 安装编译

		cd /usr/local/fast/FastDFS/
		# 编译命令:
		./make.sh
		# 安装命令:
		./make.sh install

 执行./make.sh install 安装情况如下:

		root@jsyfdev001 FastDFS]# ./make.sh install
		mkdir -p /usr/bin
		mkdir -p /etc/fdfs
		cp -f fdfs_trackerd /usr/bin
		if [ ! -f /etc/fdfs/tracker.conf.sample ]; then cp -f ../conf/tracker.conf /etc/fdfs/tracker.conf.sample; fi
		mkdir -p /usr/bin
		mkdir -p /etc/fdfs
		cp -f fdfs_storaged  /usr/bin
		if [ ! -f /etc/fdfs/storage.conf.sample ]; then cp -f ../conf/storage.conf /etc/fdfs/storage.conf.sample; fi
		mkdir -p /usr/bin
		mkdir -p /etc/fdfs
		mkdir -p /usr/lib64
		cp -f fdfs_monitor fdfs_test fdfs_test1 fdfs_crc32 fdfs_upload_file fdfs_download_file fdfs_delete_file fdfs_file_info fdfs_appender_test fdfs_appender_test1 fdfs_append_file fdfs_upload_appender /usr/bin
		if [ 0 -eq 1 ]; then cp -f libfdfsclient.a /usr/lib64; fi
		if [ 1 -eq 1 ]; then cp -f libfdfsclient.so /usr/lib64; fi
		mkdir -p /usr/include/fastdfs
		cp -f ../common/fdfs_define.h ../common/fdfs_global.h ../common/mime_file_parser.h ../common/fdfs_http_shared.h ../tracker/tracker_types.h ../tracker/tracker_proto.h ../tracker/fdfs_shared_func.h ../storage/trunk_mgr/trunk_shared.h tracker_client.h storage_client.h storage_client1.h client_func.h client_global.h fdfs_client.h /usr/include/fastdfs

##### 3.3 采用默认安装方式脚本文件说明:

##### 3.3.1、服务脚本在:

/etc/init.d/fdfs_storaged <br/>
/etc/init.d/fdfs_trackerd

##### 3.3.2、配置文件在:

/etc/fdfs/client.conf.sample  <br/>
/etc/fdfs/storage.conf.sample  <br/>
/etc/fdfs/tracker.conf.sample

##### 3.3.3、命令行工具在/usr/bin/目录下

查询Fdfs_*的一些列执行脚本的命令如下

		cd /usr/bin/ && ls | grep fdfs

可以在控制台看到有以下输出内容

		[root@jsyfdev001 FastDFS]# cd /usr/bin/ && ls | grep fdfs
		fdfs_appender_test
		fdfs_appender_test1
		fdfs_append_file
		fdfs_crc32
		fdfs_delete_file
		fdfs_download_file
		fdfs_file_info
		fdfs_monitor
		fdfs_storaged
		fdfs_test
		fdfs_test1
		fdfs_trackerd
		fdfs_upload_appender
		fdfs_upload_file

##### 3.3.4 因为FastDFS服务脚本设置的bin目录为/usr/local/bin/下,但是实际我们安装在了/usr/bin/下面。所以我们需要修改FastDFS配置文件中的路径，也就是需要这个配置文件:
编辑storaged命令:

		vim /etc/init.d/fdfs_storaged

进行全局替换命令(注意: 替换结束时,会提示有7处修改)

		:%s+/usr/local/bin+/usr/bin

编辑trackerd命令:

		im /etc/init.d/fdfs_trackerd

进行全局替换命令

		:%s+/usr/local/bin+/usr/bin

#### 四、配置跟踪器

##### 4.1进入fdfs目录

		cd /etc/fdfs/

目录配置跟踪器文件，把tracker.conf.samp le文件进行cope一份:去修改tracker.conf文件

		cp -a tracker.conf.sample  tracker.conf

查询copy文件是否成功

		ll

ll之后控制如输出接口如下:

		[root@jsyfdev001 fdfs]# ll
		total 28
		-rw-r--r-- 1 root root 1461 Oct 22 20:54 client.conf.sample
		-rw-r--r-- 1 root root 7829 Oct 22 20:54 storage.conf.sample
		-rw-r--r-- 1 root root 7102 Oct 22 20:54 tracker.conf
		-rw-r--r-- 1 root root 7102 Oct 22 20:54 tracker.conf.sample

##### 4.2 修改tracker.conf文件

命令:

		vim /etc/fdfs/tracker.conf

把base_path替换成我们自己的文件存放目录

		base_path=/fastdfs/tracker

##### 注意:

对于tracker.conf配置文件参数解释可以找官方文档 <br/>
地址为: http://bbs.chinaunix.net/thread-1941456-1-1.html

##### 4.3 最后我们一定要创建之前定义好的目录, 也就是base_path = /fastdfs/tracker的目录

命令:

		mkdir -p /fastdfs/tracker

##### 4.4 添加防火墙规则:

		vim /etc/sysconfig/iptables

		# 添加入站规则 注意: -A INPUT -j REJECT --reject-with icmp-host-prohibited行之止添加
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j 

注意：这里使用80和8080端口为例。

一般添加到“-A INPUT -p tcp -m state –state NEW -m tcp–dport 22 -j ACCEPT”行的上面或者下面
切记不要添加到最后一行，否则防火墙重启后不生效。

		[root@jsyfdev001 fdfs]# vim /etc/sysconfig/iptables
		# sample configuration for iptables service
		# you can edit this manually or use system-config-firewall
		# please do not ask us to add additional ports/services to this default configuration
		*filter
		:INPUT ACCEPT [0:0]
		:FORWARD ACCEPT [0:0]
		:OUTPUT ACCEPT [0:0]
		-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
		-A INPUT -p icmp -j ACCEPT
		-A INPUT -i lo -j ACCEPT
		-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT

		# FastDFS开放接口
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT

		#重启防火墙使配置生效
		#systemctl restart iptables.service
		#设置防火墙开机启动
		#systemctl enable iptables.service


		# - --------------------以下为拒绝接口-------------------
		-A INPUT -j REJECT --reject-with icmp-host-prohibited
		-A FORWARD -j REJECT --reject-with icmp-host-prohibited

		COMMIT

添加防火墙之后, 重启防火墙

		service iptables restart 
		# 或者以下命令(根据不同防火墙重启)
		systemctl restart iptables.service

##### 4.5 启动跟踪器

查询tracker 注意: 在此处是没有任何文件的

		cd /fastdfs/tracker/ && ls -l

启动tracker命令: 初次启动成功后会在/fastdbf/tracker/ 目录下创建 data、logs俩个目录

		/etc/init.d/fdfs_trackerd start 

查询tracker 注意: 在此处会出现data,logs两个目录

		cd /fastdfs/tracker/ && ls -l

注意: 楼主在此处运行fdfs_trackerd start没有启动, 如下(没有遇到请忽略)

		[root@jsyfdev001 tracker]# /etc/init.d/fdfs_trackerd start
		Starting fdfs_trackerd (via systemctl):  Warning: fdfs_trackerd.service changed on disk. Run 'systemctl daemon-reload' to reload units.
																   [  OK  ]

##### 错误,请执行下面命令(没有遇到请忽略)

		systemctl daemon-reload

查看进程命令:

		ps -ef | grep fdfs

停止tracker命令:

		/etc/init.d/fdfs_trackerd stop

##### 4.6 可以设置开机启动跟踪器:

		vim /etc/rc.d/rc.local

加入配置:/etc/init.d/fdfs_trackerd start

#### 五、配置FastDFS存储

##### 5.1 复制storage文件一份 

		# 进入fdfs文件目录:
		cd /etc/fdfs/
		# copy storage文件一份 
		cp storage.conf.sample storage.conf

##### 5.2 修改storage.conf文件

命令:

		vim /etc/fdfs/storage.conf

修改内容:

		base_path=/fastdfs/storage
		store_path0=/fastdfs/storage
		tracker_server=ip:22122 #注意填写ip
		http.server_port=8888

##### 5.3 创建存储目录:

		mkdir -p /fastdfs/storage

##### 5.4 打开防火墙:

注意: 此处和步骤4.4除了开放端口不同,基本相同,详情请参考4.4

		vim /etc/sysconfig/iptables

		# 添加防火墙规则
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 23000 -j ACCEPT

		service iptables restart
		# 或者以下命令(根据不同防火墙重启)
		systemctl restart iptables.service

##### 5.5 启动存储(storage)

命令:

		# 启动 初次启动成功后会在/fastdfs/storage/ 目录下创建 data、logs俩个目录

ls -l /fastdfs/storage/

		**注意:** 初次启动时, data目录创建较慢,请耐心等待
		/etc/init.d/fdfs_storaged start 
		#关闭
		/etc/init.d/fdfs_storaged stop

##### 5.6 查看FastDFS storage 是否启动成功

		#命令
		ps -ef | grep fdfs

并且我们进入到/fastdfs/storage/data/文件夹下会看到一些目录文件(256*256)

		cd /fastdfs/storage/data/ && ls

到此为止我们的FastDFS环境已经搭建完成!

------------

#### 六、测试环境

##### 6.1 我们先使用命令上传一个文件。注意:是在tracker(跟踪器)中上传。 首先我们在跟踪器里copy一份client.conf文件。

		cd /etc/fdfs/
		cp client.conf.sample client.conf

##### 6.2 编辑client.conf文件

		vim /etc/fdfs/client.conf

修改内容:

		base_path=/fastdfs/tracker
		tracker_server=ip:22122 # 注意这里填写ip

##### 6.3 我们找到命令的脚本位置，并且使用命令，进行文件的上传:

		[root@jsyfdev001 fdfs]# cd /usr/bin/

		[root@jsyfdev001 bin]# ls | grep fdfs
		fdfs_appender_test
		fdfs_appender_test1
		fdfs_append_file
		fdfs_crc32
		fdfs_delete_file
		fdfs_download_file
		fdfs_file_info
		fdfs_monitor
		fdfs_storaged
		fdfs_test
		fdfs_test1
		fdfs_trackerd
		fdfs_upload_appender
		fdfs_upload_file

##### 6.4 使用命令fdfs_upload_file进行上传操作:

首先，我们先看一下存储器，进入到data下，在进入00文件夹 下，发现00文件夹下还有一堆文件夹，然后继续进入00文件夹下，最终我们所 进入的文件夹为:

		[root@jsyfdev001 00]# ls -l /fastdfs/storage/data/00/00
		total 0

然后，我们进行上传操作，比如把之前的/usr/local/software/文件夹下的某一个 文件上传到FastDFS系统中去，在跟踪器中上传文件 
命令如下:

		[root@jsyfdev001 local]# /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/software/FastDFS_v5.05.tar.gz
		group1/M00/00/00/L12w41nulh-AUqfMAAVFOL7FJU4.tar.gz

最后我们发现，命令执行完毕后，返回一个group1/M00/00/00/…的ID，其实就 是返回当前所上传的文件在存储器中的哪一个组、哪一个目 录位置，所以我们查看存储器中的/fastdfs/storage/data/00/00文件夹位置，发现 已经存在了刚才上传的文件，到此为止，我们的测试上传文件已经OK了。如下 :

		[root@jsyfdev001 00]# ls -l /fastdfs/storage/data/00/00
		total 1132
		-rw-r--r-- 1 root root 345400 Oct 22 23:38 L12w41nsu2-AHA7xAAVFOL7FJU4.tar.gz

------------

#### 七、FastDFS与Nginx整合

##### 7.1 首先机器里必须先安装nginx

##### 7.2 然后我们在存储节点上安装fastdfs-nginx- module_v1.16.tar.gz包进行整合。

		# 目录命令:
		[root@jsyfdev001 ~]# cd /usr/local/software/

		# 解压命令:
		[root@jsyfdev001 ~]# tar -zxvf /usr/local/software/fastdfs-nginx-module_v1.16.tar.gz -C /usr/local/fast/

##### 7.3 进入目录:

		[root@jsyfdev001 ~]# cd /usr/local/fast/fastdfs-nginx-module/src

##### 7.4 编辑配置文件config

命令:
修改前: 把config文件中第4行的最后一个local文件层次去掉 把
/usr/include/fastdfs 修改为 /usr/local/include/fastdfs
/usr/local/include/fastcommon/ 修改为 /usr/include/fastcommon/

		[root@jsyfdev001 ~]# vim /usr/local/fast/fastdfs-nginx-module/src/config
		ngx_addon_name=ngx_http_fastdfs_module
		HTTP_MODULES="$HTTP_MODULES ngx_http_fastdfs_module"
		NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_fastdfs_module.c"
		CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
		CORE_LIBS="$CORE_LIBS -L/usr/local/lib -lfastcommon -lfdfsclient"
		CFLAGS="$CFLAGS -D_FILE_OFFSET_BITS=64 -DFDFS_OUTPUT_CHUNK_SIZE='256*1024' -DFDFS_MOD_CONF_FILENAME='\"/etc/fdfs/mod_fastdfs.conf\"'"

修改后:

		[root@jsyfdev001 ~]# vim /usr/local/fast/fastdfs-nginx-module/src/config
		ngx_addon_name=ngx_http_fastdfs_module
		HTTP_MODULES="$HTTP_MODULES ngx_http_fastdfs_module"
		NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_fastdfs_module.c"
		CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
		CORE_LIBS="$CORE_LIBS -L/usr/local/lib -lfastcommon -lfdfsclient"
		CFLAGS="$CFLAGS -D_FILE_OFFSET_BITS=64 -DFDFS_OUTPUT_CHUNK_SIZE='256*1024' -DFDFS_MOD_CONF_FILENAME='\"/etc/fdfs/mod_fastdfs.conf\"'"


##### 7.5 FastDFS与nginx进行集成

首先把7.1步骤之前的nginx进行删除

目录命令:

		[root@jsyfdev001 nginx-1.6.2]# cd /usr/local/

删除命令:

		[root@jsyfdev001 local]# rm -rf nginx

进入到nginx目录命令:

		[root@jsyfdev001 local]# cd /usr/local/nginx-1.6.2

		[root@jsyfdev001 nginx-1.6.2]# ./configure --add-module=/usr/local/fast/fastdfs-nginx-module/src/

重新安装编辑

		[root@jsyfdev001 nginx-1.6.2]# make && make install

##### 7.6 复制fastdfs-ngin-module中的配置文件，到/etc/fdfs目录中，如图所示:

copy命令:

		[root@jsyfdev001 src]# cp /usr/local/fast/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/

##### 7.7 进行修改 /etc/fdfs/ 目录下，我们刚刚copy过来的mod_fastdfs.conf 文件。

		[root@jsyfdev001 src]# vim /etc/fdfs/mod_fastdfs.conf

		# 修改内容:比如连接超时时间、跟踪器路径配置、url的group配置
		connect_timeout=10
		tracker_server=ip:22122 # 注意这里填写本机的ip地址
		url_have_group_name = true
		store_path0=/fastdfs/storage

##### 7.8 复制FastDFS里的2个文件，到/etc/fdfs目录中，如图所示 
目录命令:

		[root@jsyfdev001 src]# cd /usr/local/fast/FastDFS/conf/

Copy命令:

		[root@jsyfdev001 conf]# cp http.conf mime.types /etc/fdfs/

##### 7.9 创建一个软连接，在/fastdfs/storage文件存储目录下创建软连接，将其链接到实际存放数据的目录。

		[root@jsyfdev001 conf]# ln -s /fastdfs/storage/data/ /fastdfs/storage/data/M00

##### 7.10 修改Nginx配置文件

命令:

		[root@jsyfdev001 local]# sudo vim /usr/local/nginx/conf/nginx.conf

修改配置内容如下图所示:

		修改内容为:
		listen 8888;
		server_name localhost;
		location ~/group([0-9])/M00 {
			#alias /fastdfs/storage/data;
			ngx_fastdfs_module; 
		}

注意: nginx里的端口要和第五步配置FastDFS存储中的storage.conf文件配置一致， 也就是(http.server_port=8888)

最后检查防火墙，然后我们启动nginx服务

启动命令:

		[root@jsyfdev001 00]# sudo ps -ef | grep nginx

找到我们6.4步骤中上传的附件

		[root@jsyfdev001 00]# ls -l /fastdfs/storage/data/00/00
		total 1132
		-rw-r--r-- 1 root root 345400 Oct 22 23:38 L12w41nsu2-AHA7xAAVFOL7FJU4.tar.gz

现在我们使用这个ID用浏览器访问地址: 
http://ip:8888/group1/M00/00/00/L12w41nsu2-AHA7xAAVFOL7FJU4.tar.gz 
我们就可以下载这个文件啦!


注意: 我们在使用FastDFS的时候，需要正常关机，不要使用kill -9 强杀FastDFS进程，不然会在文件上传时出现丢数据的情况。 到此，我们的FastDFS与Nginx整合完毕!!

##### 8:启动停止服务步骤如下:
启动命令:

		# 启动tracker命令:
		/etc/init.d/fdfs_trackerd start
		# 查看进程命令:
		ps -el | grep fdfs
		# 启动storage命令:
		/etc/init.d/fdfs_storaged start
		# 查看进程命令:
		ps -el | grep fdfs
		# 启动nginx命令:
		/usr/local/nginx/sbin/nginx

停止命令:

		# 停止tracker命令:
		/etc/init.d/fdfs_trackerd stop 
		# 关闭storage命令:
		/etc/init.d/fdfs_storaged stop 
		# 关闭nginx命令:
		/usr/local/nginx/sbin/nginx -s stop

------------

启动命令：

		tracker server : /usr/local/bin/fdfs_trackerd /etc/fdfs/tracker.conf
		storaged server: /usr/local/bin/fdfs_storaged /etc/fdfs/storage.conf
		一般使用fastdfs还需要使用到它的映射nginx
		启动命令
		nginx :/usr/local/nginx2/sbin/nginx


#### FastDFS问题记录:

##### 问题一:
楼主把阿里云服务器从固定6M宽带升级为按流量收费20M后出现不能上传文件的错误

##### 错误信息如下:
tracker中log日志显示错误信息如下:

		file: tracker_relationship.c, line: 383, selecting leader...

storage中log日志显示错误信息如下:

		INFO - file: tracker_client_thread.c, line: 310, successfully connect to tracker server xxx:22122, as a tracker client, my ip is 192.168.xx.xx
		ERROR - file: tracker_proto.c, line: 48, server: xxx:22122, response status 2 != 0

##### 解决方法如下:

首先, 用netstat命令检查服务的 LISTEN 是否为: ESTABLISHED, 如果不为ESTABLISHED说明服务不能用

命令如下:

		netstat -an | grep 22122

参考状态为:

		CLOSED：无连接是活动的或正在进行
		LISTEN：服务器在等待进入呼叫
		SYN_RECV：一个连接请求已经到达，等待确认
		SYN_SENT：应用已经开始，打开一个连接
		ESTABLISHED：正常数据传输状态
		FIN_WAIT1：应用说它已经完成
		FIN_WAIT2：另一边已同意释放
		ITMED_WAIT：等待所有分组死掉
		CLOSING：两边同时尝试关闭
		TIME_WAIT：另一边已初始化一个释放
		LAST_ACK：等待所有分组死掉

其次,查看集群同步情况

		fdfs_monitor /etc/fdfs/client.conf

查看集群中的状态是否为ACTIVE 如果不是并且不是你想要的便可以删除了

删除命令如下:

		fdfs_monitor /etc/fdfs/client.conf delete group1 10.2.x.x

详细也可参考博客：https://blog.csdn.net/zhou_andy/article/details/54922732

重启后(见上 8),再执行上传操作

		[root@jsyfdev001 local]# /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/software/FastDFS_v5.05.tar.gz
		# 如果出现下方提示, 便说明已经成功
		group1/M00/00/00/L12w41nulh-AUqfMAAVFOL7FJU4.tar.gz

 #### 问题二: 上传文件不成功, 提示response status 28 != 0 

		DEBUG - file: tracker_proto.c, line: 48, server: xxxx:22122, response status 28 != 0
		# 或者
		fastdfs getStoreStorage fail, errno code: 28

出现原因:空间不足
tracker.conf的配置项reserved_storage_space的值默认为4GB，而当前环境下剩余空间已不足4GB。(Fastdfs5.0之上, 显示的为百分比 默认为10%)

解决方法如下:
可先在tracker.conf中修改reserved_storage_space的大小, 后升级服务器硬盘

		 reserved storage space for system or other applications.
		# if the free(available) space of any stoarge server in 
		# a group <= reserved_storage_space, 
		# no file can be uploaded to this group.
		# bytes unit can be one of follows:
		### G or g for gigabyte(GB)
		### M or m for megabyte(MB)
		### K or k for kilobyte(KB)
		### no unit for byte(B)
		### XX.XX% as ratio such as reserved_storage_space = 10%
		reserved_storage_space = 10%


