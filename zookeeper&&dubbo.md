# Linux下的zookeeper+dubbo的配置

## 安装zookeeper
### 1.首先肯定是从官网下载相应的tar包，并解压
    网址：http://zookeeper.apache.org/releases.html#download
    
    解压：tar -zxvf zookeeper-3.4.6.tar.gz
### 2.添加环境变量

    export ZOOKEEPER_HOME=/home/grid/zookeeper-3.4.6
    
    export PATH=$PATH:$ZOOKEEPER_HOME/bin
    
### 3.修改配置文件

    cd  zookeeper-3.4.6/conf ，将zoo_sample.cfg复制为zoo.cfg，并修改zoo.cfg的内容如下：
    命令：cp /usr/local/zoo/zookeeper-3.4.6/conf/zoo_sample.cfg /usr/local/zoo/zookeeper-3.4.6/conf/zoo.cfg
    
    其他文件都可以删掉，只保留下列的几项就可以了
    
    #Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。
    tickTime=2000 
    
    #集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。
    initLimit=10 
    
    #集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）
    syncLimit=5 
    
    #Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。
    dataDir=/usr/local/zoo/zookeeper1/data 
    
    #Zookeeper保存日志文件的目录。
    dataLogDir=/usr/local/zoo/zookeeper1/logs 
    
    #客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
    clientPort=3210 
    
    #服务器名称与地址：集群信息（服务器编号，服务器地址，LF通信端口，选举端口）
    其中N表示服务器编号，YYY表示服务器的IP地址，A为LF通信端口，表示该服务器与集群中的leader交换的信息的端口。
    B为选举端口，表示选举新leader时服务器间相互通信的端口（当leader挂掉时，其余服务器会相互通信，选择出新的leader）。
    一般来说，集群中每个服务器的A端口都是一样，每个服务器的B端口也是一样。但是当所采用的为伪集群时，IP地址都一样，
    只能时A端口和B端口不一样。
    
    下列就是一个伪集群 
    server.1=127.0.0.1:3211:3212
    server.2=127.0.0.1:3221:3232
    server.3=127.0.0.1:3231:3232
    
### 4.创建myid 
    在dataDir对应的路径文件夹下（我这里是zkdata文件夹），创建一个myid的文件，并在该文件中输入对应集群各节点的id ，
    我这里myid内容为1，这个是与server.1=127.0.0.1:3211:3212对应的。各个节点分别输入对应的id值即可。 
    照例：节点2 对应的myid里面的内容就是2
    
### 5.配置伪集群 
    将之前配置好的zookeeper1复制两份zookeeper2，zookeeper3在同一路径下 
    
    修改zoo.cfg 文件以及data文件夹下面的myid文件
    
### 6.启动伪集群

    这个三个服务按照顺序逐步打开
    /usr/local/zoo/zookeeper1/bin/zkServer.sh start
    /usr/local/zoo/zookeeper2/bin/zkServer.sh start
    /usr/local/zoo/zookeeper3/bin/zkServer.sh start
    
    查看zookeeper的运行状态
    /usr/local/zoo/zookeeper1/bin/zkServer.sh status
    /usr/local/zoo/zookeeper2/bin/zkServer.sh status
    /usr/local/zoo/zookeeper3/bin/zkServer.sh status
    
    会发现这三个有一个是leader,两个是follower,一个是主导者，两个是追随者，到这里zookeeper伪集群就搭建完毕了。
   
 ### 7.注意事项
   这个集群必须按照奇数部署并且不能少于三个，因为这个和算法有关.在部署集群的时候，
   必须所有服务都打开之后再去查看状态，不然再没有全部打开之前，查看状态永远都是没有成功的状态显示，小编当时被坑了好久。
   
   端口问题：建议先关闭防火墙再起启动服务。或者就是先把防火墙的端口配置好，再去启动服务，不然会一直报端口错误的问题。
 
 ## 安装dubbo
 
    
    


