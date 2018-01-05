# linux的基础配置
(小编的linux系统是CentOS7.2的) <br>  
[1.修改主机名称](#修改主机名称) <br>  
[2.配置iptables](#配置iptables) <br>   
[3.安装jdk](#安装jdk)  <br>  
[4.安装tomcat](#安装tomcat) 


## 修改主机名称
   临时生效修改：使用命令行修改 hostname 主机名(可自定义)，重新登录 SHELL 生效，服务器重启后失效。

#### centOS 7以上的linux系统（包含7）
   使用命令hostnamectl set-hostname 主机名 来修改，修改完毕后重新 SHELL 登录即可。

#### centOS 7以下的linux系统 
   1.以根用户登录，或者登录后切换到根用户，然后在提示符下输入hostname命令，可以看出当前系统的主机名为localhost.localdomain。
      
   2.更改/etc/sysconfig下的network文件，在提示符下输入vim /etc/sysconfig/network，然后将HOSTNAME后面的值改为想要设置的主机名。
      
   3.更改/etc下的hosts文件，在提示符下输入vim /etc/hosts，然后将localhost.localdomain改为想要设置的主机名。
      
   4.在提示符下输入reboot命令，重新启动服务器。
      
   5.重启完成后用hostname命令查询系统主机名，可以看出系统主机名已经变更为你设定的名字。
 
 ## 配置iptables
   在centOS 7 以上的系统中默认使用的是firewall,centOS 7以下的系统使用的是iptables 
   firewall和iptables的区别：firewall是centos7里面的新的防火墙命令，它底层还是使用 iptables 对内核命令动态通信包过滤的，简单理解就是firewall是centos7下管理iptables的新命令。
   
   ##### 1.先检查是否安装了iptables  
     service iptables status  
   ##### 2.安装iptables  
     yum install -y iptables  
   ##### 3.升级iptables  
     yum update iptables   
   ##### 4.安装iptables-services  
     yum install iptables-services
   
 #### 禁用/停止自带的firewalld服务 (CentOS 7 以上的系统)
   
   ##### 停止firewalld服务  
     systemctl stop firewalld  
   ##### 禁用firewalld服务  
     systemctl mask firewalld 
 #### 设置现有规则
      #查看iptables现有规则  
      iptables -L -n  
      
      #先允许所有,不然有可能会杯具  
      iptables -P INPUT ACCEPT  
      
      #清空所有默认规则  
      iptables -F  
      
      #清空所有自定义规则  
      iptables -X  
      
      #所有计数器归0  
      iptables -Z  
      
      #允许来自于lo接口的数据包(本地访问)  
      iptables -A INPUT -i lo -j ACCEPT  
      
      #开放22端口  
      iptables -A INPUT -p tcp --dport 22 -j ACCEPT  
      
      #开放21端口(FTP)  
      iptables -A INPUT -p tcp --dport 21 -j ACCEPT  
      
      #开放80端口(HTTP)  
      iptables -A INPUT -p tcp --dport 80 -j ACCEPT  
      
      #开放443端口(HTTPS)  
      iptables -A INPUT -p tcp --dport 443 -j ACCEPT  
      
      #允许ping  
      iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT  
      
      #允许接受本机请求之后的返回数据 RELATED,是为FTP设置的  
      iptables -A INPUT -m state --state  RELATED,ESTABLISHED -j ACCEPT  
      
      #其他入站一律丢弃  
      iptables -P INPUT DROP  
      
      #所有出站一律绿灯  
      iptables -P OUTPUT ACCEPT  
      
      #所有转发一律丢弃  
      iptables -P FORWARD DROP  
#### 其他规则设定(选用)
      #如果要添加内网ip信任（接受其所有TCP请求）  
      iptables -A INPUT -p tcp -s (你的IP) -j ACCEPT  
      
      #过滤所有非以上规则的请求  
      iptables -P INPUT DROP  
      
      #要封停一个IP，使用下面这条命令：  
      iptables -I INPUT -s ***.***.***.*** -j DROP  
      
      #要解封一个IP，使用下面这条命令:  
      iptables -D INPUT -s ***.***.***.*** -j DROP  
#### 保存规则设定
      #保存上述规则  
      service iptables save  
     
#### 开启iptables服务
      #注册iptables服务,相当于以前的chkconfig iptables on  
      systemctl enable iptables.service  
      
      #开启服务  
      systemctl start iptables.service  
      
      #查看状态  
      systemctl status iptables.service  
      
#### 以下为完整设置脚本(选看)
      #!/bin/sh  
      iptables -P INPUT ACCEPT  
      iptables -F  
      iptables -X  
      iptables -Z  
      iptables -A INPUT -i lo -j ACCEPT  
      iptables -A INPUT -p tcp --dport 22 -j ACCEPT  
      iptables -A INPUT -p tcp --dport 21 -j ACCEPT  
      iptables -A INPUT -p tcp --dport 80 -j ACCEPT  
      iptables -A INPUT -p tcp --dport 443 -j ACCEPT  
      iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT  
      iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT  
      iptables -P INPUT DROP  
      iptables -P OUTPUT ACCEPT  
      iptables -P FORWARD DROP  
      service iptables save  
      systemctl restart iptables.service 
      
 #### iptables添加或者删除端口
       #直接修改文件 
       vim /etc/sysconfig/iptables 
       
       #重启防火墙 
       systemctl restart iptables.service
 ## 安装jdk
 (小编这里安装的是JDK 1.8) 
      
      ##方法一：手动解压JDK的压缩包，然后设置环境变量
      1.在/usr/local目录下创建一个软件存放目录
       [root@localhost ~]# mkdir /usr/local/java
       [root@localhost ~]# cd /usr/local/java
       
      2.下载jdk,然后解压
      直接到到官网下载需要的jak包:http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
      下载成功后将jdk文件上传到刚刚创建的目录 /usr/local/java
      
      解压刚才上传成功的tar包
      [root@localhost java]# tar -zxvf jdk-8u151-linux-x64.tar.gz
      
      3.将解压的文件移动到/usr/local/目录下并且重命名为jdk1.8
       cp -r /usr/local/java/jdk1.8.0_151 /usr/local/jdk1.8
      
      4.设置环境变量
      [root@localhost ~]# vim /etc/profile

      #将下列的命令复制到profile文件中

      export JAVA_HOME=/usr/local/jdk1.8
      export JRE_HOME=$JAVA_HOME/jre
      CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
      PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
      export JAVA_HOME JRE_HOME CLASS_PATH PATH

      5.保存修改设置 
       [root@localhost ~]# source /etc/profile

      6.检查安装是否成功
       [root@localhost ~]# java -version
      
      
      ##方法二：用yum安装JDK
      1.查看yum库中都有哪些jdk版本(暂时只发现了openjdk)   openjdk源代码不完整，不建议安装
      [root@localhost ~]# yum search java|grep jdk

      2.选择版本,进行安装 (这里我选择的是1.8)
      [root@localhost ~]# yum install java-1.8.0-openjdk.x86_64
      //安装完之后，默认的安装目录是在: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64

      3.将安装的jdk移动到/usr/local/目录下并且重命名为jdk1.8
      cp -r /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/ /usr/local/jdk1.8

      4.设置环境变量
      [root@localhost ~]# vim /etc/profile

      #将下列的命令复制到profile文件中

      export JAVA_HOME=/usr/local/jdk1.8
      export JRE_HOME=$JAVA_HOME/jre
      CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
      PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
      export JAVA_HOME JRE_HOME CLASS_PATH PATH

      5.保存修改设置 
       [root@localhost ~]# source /etc/profile

      6.检查安装是否成功
       [root@localhost ~]# java -version

       打印如下信息
       openjdk version "1.8.0_151"
       OpenJDK Runtime Environment (build 1.8.0_151-b12)
       OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)


    
 
 
